---
layout: editorial
---

# Analyze Execution Plan(실행 계획 분석)

MySQL 8.0 버전부터는 쿼리의 실행 계획을 `EXPLAIN` 명령으로 확인할 수 있으며, 단순 테이블 형태로 출력할 수 있다.

```mysql
EXPLAIN
SELECT *
FROM employees e
         INNER JOIN salaries s ON s.emp_no = e.emp_no
WHERE e.first_name = 'ABC';
```

| id | select_type | table | partitions | type |    possible_keys     |     key      | key_len |        ref         | rows | filtered | Extra |
|:---|:-----------:|:-----:|:----------:|:----:|:--------------------:|:------------:|:-------:|:------------------:|:----:|:--------:|:-----:|
| 1  |   SIMPLE    |   e   |    NULL    | ref  | PRIMARY,ix_firstname | ix_firstname |   58    |       const        |  1   |  100.00  | NULL  |
| 1  |   SIMPLE    |   e   |    NULL    | ref  |       PRIMARY        |   PRIMARY    |    4    | employees.e.emp_no |  10  |  100.00  | NULL  |

실행 순서는 위에서 아래로 순서대로 표시되며, id 컬럼의 순서에 따라 아래의 규칙을 따른다.

- 위쪽에 출력된 결과일수록(id 컬럼의 값이 작을수록) 쿼리의 바깥 부분이거나 먼저 접근한 테이블
- 아래쪽에 출력된 결과일수록 쿼리의 안쪽(Inner) 부분 또는 나중에 접근한 테이블에 해당

실행 계획 테이블을 살펴보면, 여러 컬럼이 표시되는데 각 컬럼의 의미는 다음과 같다.

## id 컬럼

하나의 SELECT 문장은 다시 한 개 이상의 Sub Select 문장을 포함할 수 있는데,  
위의 처음 예시에서는 조인을 사용하였기 때문에 id 값이 증가하지 않고 1로 표시되지만, 여러 개의 Sub Select 문장이 포함된 쿼리의 경우 id 값이 증가하면서 표시된다.

```mysql
EXPLAIN
SELECT ((SELECT COUNT(*) FROM employees) + (SELECT COUNT(*) FROM departments)) AS total_count;
```

| id | select_type |    table    | type  |     key     | ref  |  rows  |     Extra      |
|:---|:-----------:|:-----------:|:-----:|:-----------:|:----:|:------:|:--------------:|
| 1  |   PRIMARY   |    NULL     | NULL  |    NULL     | NULL |  NULL  | No tables used |
| 3  |  SUBQUERY   | departments | index | ux_deptname | NULL |   9    |  Using index   |
| 2  |  SUBQUERY   |  employees  | index | ix_hiredate | NULL | 300252 |  Using index   |

## select_type 컬럼

각 단위 SELECT 쿼리가 어떤 타입의 쿼리인지 표시되는 컬럼으로, 쿼리의 종류에 따라 다음과 같이 표시된다.

### SIMPLE

UNION이나 서브쿼리를 사용하지 않는 단순한 SELECT 쿼리

### PRIMARY

UNION이나 서브쿼리를 가지는 SELECT 쿼리의 실행 계획에서 가장 바깥쪽에 있는 단위 쿼리

### UNION

UNION으로 결합하는 단위 SELECT 쿼리 가운데 첫 번째를 제외한 두 번째 이후 단위 SELECT 쿼리

### DEPENDENT UNION

UNION이나 UNION ALL로 집합을 결합하는 쿼리,  
DEPENDENT는 UNION이나 UNION ALL로 결합된 단위 쿼리가 외부 쿼리에 의해 영향을 받는 것을 의미한다.

```mysql
EXPLAIN
SELECT *
FROM employees e1
WHERE e1.emp_no IN (SELECT e2.emp_no
                    FROM employees e2
                    WHERE e2.first_name = 'Matt'
                    UNION
                    SELECT e3.emp_no
                    FROM employees e3
                    WHERE e3.last_name = 'Matt');
```

| id   |    select_type     |   table    |  type  |   key   | ref  |  rows  |      Extra      |
|:-----|:------------------:|:----------:|:------:|:-------:|:----:|:------:|:---------------:|
| 1    |      PRIMARY       |     e1     |  ALL   |  NULL   | NULL | 300252 |   Using where   |
| 2    | DEPENDENT SUBQUERY |     e2     | eq_ref | PRIMARY | func |   1    |   Using where   |
| 3    |  DEPENDENT UNION   |     e3     | eq_ref | PRIMARY | func |   1    |   Using where   |
| NULL |    UNION RESULT    | <union2,3> |  ALL   |  NULL   | NULL |  NULL  | Using temporary |

### UNION RESULT

UNION 결과를 담아두는 임시 테이블을 생성하는 단위 쿼리,  
단위 쿼리가 아니기 때문에 id 값이 NULL로 표시된다.

### SUBQUERY

select_type의 SUBQUERY는 FROM 절 이외에서 사용되는 서브쿼리  
(FROM절에 사용된 서브쿼리는 DERIVED로 표시)

### DEPENDENT SUBQUERY

바깥쪽(Outer) SELECT 쿼리에서 정의된 컬럼을 사용해 외부 영향을 받는 서브쿼리,  
외부 쿼리가 먼저 수행되야하므로 일반 서브쿼리보다 성능이 떨어질 때가 많다.

### DERIVED

FROM 절에서 단위 SELECT 쿼리의 실행 결과로 메모리나 디스크에 임시 테이블을 생성하게 되는 쿼리

### DEPENDENT DERIVED

FROM 절에서 LATERAL JOIN을 사용하여 외부 컬럼을 참조하는 서브쿼리

```mysql
SELECT *
FROM employees e
         LEFT JOIN LATERAL
    (SELECT *
     FROM salaries s
     WHERE s.emp_no = e.emp_no
     ORDER BY s.from_date DESC
     LIMIT 2) AS s2
                   ON s2.emp_no = e.emp_no;
```

| id |    select_type    |   table    | type |     key     |     Extra      |
|:---|:-----------------:|:----------:|:----:|:-----------:|:--------------:|
| 1  |      PRIMARY      |     e      | ALL  |    NULL     |      NULL      |
| 1  |      PRIMARY      | <derived2> | ref  | <auto_key0> |      NULL      |
| 2  | DEPENDENT DERIVED |     s      | ref  |   PRIMARY   | Using filesort |

### UNCACHEABLE SUBQUERY

일반적으로 서브쿼리는 이전의 실행 결과를 그대로 사용할 수 있게 캐시에 저장되어 재사용되지만,  
서브쿼리에 포함된 요소에 의해 캐시가 불가능한 SUBQUERY의 경우 UNCACHEABLE SUBQUERY로 표시, 아래의 경우 캐시가 불가능하게 된다.

- 사용자 변수가 서브쿼리에 사용된 경우
- NOT-DETERMINISTIC 속성의 스토어드 루틴이 서브쿼리 내에 사용된 경우
- UUID() / RAND() 같은 결괏값이 매번 달라지는 함수가 서브쿼리 내에 사용된 경우

### UNCACHEABLE UNION

UNCACHEABLE + UNION 특성을 가진 쿼리

### MATERIALIZE

DERIVED와 비슷하게 쿼리의 내용을 임시 테이블로 생성하는 쿼리(동일하지는 않음)

## table 컬럼

테이블의 이름에 별칭이 부여된 경우에는 별칭이 표시되며 `<>`로 둘러싸인 테이블은 임시 테이블을 의미한다.(안의 숫자는 단위 쿼리의 id값)

## type 컬럼

쿼리의 테이블 접근 방식을 의미하며, 인덱스를 사용했는지, 풀 테이블 스캔을 했는지 등을 표시한다.

- system: InnoDB에서는 사용되지 않음
- const: 프라이머리 키나 유니크 키 컬럼을 이용하는 WHERE 조건절을 가지며, 동등(Equal) 검색 조건으로 한 건의 레코드만 읽어오는 경우 사용
- eq_ref: 조인에서 처음 읽은 테이블의 컬럼 값을, 그 다음 읽어야 할 테이블의 프라이머리 키나 유니크 키 컬럼의 동등(Equal) 검색 조건에 사용하는 경우 사용  
  (여러 테이블이 조인되는 쿼리의 실행 계획에서만 표시)
- ref: 인덱스의 종류와 관계 없이 동등(Equal) 조건 검색 시 사용
- fulltext: MySQL 서버의 전문 검색(Full-text Search) 기능을 사용하는 경우 사용
- ref_or_null: ref 접근 방법과 동일하나 NULL 비교가 추가된 형태
- unique_subquery: WHERE에 사용될 수 있는 IN(subquery) 형태의 서브쿼리를 위한 접근 방법, 서브쿼리에서 중복되지 않는 유니크한 값만 반환 시 사용
- index_subquery: WHERE 조건절에 사용될 수 있는 IN(subquery) 형태의 서브쿼리를 위한 접근 방법,  
  서브쿼리에서 중복되는 값이 있을 수 있을 때 사용(인덱스를 이용해 중복된 값 제거)
- range: 인덱스 레인지 스캔 형태의 접근 방법으로, 범위 검색하는 경우 사용
- index_merge: 2개 이상의 인덱스를 이용해 각 검색 결과를 병합해서 접근하는 방법
- index: 인덱스를 처음부터 끝까지 읽는 인덱스 풀 스캔하여 접근하는 방법
- ALL: 테이블의 모든 레코드를 처음부터 끝까지 읽는 풀 테이블 스캔 방식으로 접근하는 방법

## possible_keys 컬럼

최적의 실행 계획을 만들기 위해 후보로 선정했던 접근 방법에서 사용되는 인덱스 목록으로, 사용될 법했던 인덱스 목록(실제로 사용된 인덱스 목록과는 다름)

## key 컬럼

실제로 사용된 인덱스 목록

## key_len 컬럼

다중 컬럼으로 구성된 인덱스에서 쿼리를 처리하기 위해 사용된 인덱스의 길이를 의미(바이트 단위로, CHAR(4) + INTEGER 인덱스를 사용한 경우 4*4 + 4 = 20)

## ref 컬럼

접근 방법이 ref인 경우 참조 조건으로 어떤 값이 제공됐는지 표시(연산이 적용된 경우 func로 표시됨)

## rows 컬럼

반환하는 레코드의 건수가 아니라 쿼리를 처리하기 위해 읽어야 할 레코드의 건수를 의미(예측 값으로 실제와 일치하지 않음)

## filtered 컬럼

쿼리를 통해 반환된 데이터 중에서 필터 조건에 일치하는 데이터의 비율을 의미(예측 값으로 실제와 일치하지 않음, 실제 데이터와 비슷하게 예측할 수록 쿼리 성능 향상)

## Extra 컬럼

주로 내부적인 처리 알고리즘에 대해 여러 가지 정보를 쉼표로 구분해서 표시하며 순서는 상관 없음

###### 참고자료

- [Real MySQL 8.0 1 - 개발자와 DBA를 위한 MySQL 실전 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Real+MySQL&isbn=9791158392703&cipId=228440237%2C)
