# 실행 계획 분석

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

실행 순서는 위에서 아래로 순서대로 표시되는데,  
위쪽에 출력된 결과일수록(id 컬럼의 값이 작을수록) 쿼리의 바깥 부분이거나 먼저 접근한 테이블이고,  
아래쪽에 출력된 결과일수록 쿼리의 안쪽(Inner) 부분 또는 나중에 접근한 테이블에 해당한다.

## id 컬럼

하나의 SELECT 문장은 다시 한 개 이상의 Sub Select 문장을 포함할 수 있다.  
위의 처음 예시에서는 조인을 사용하였기 때문에 id 값이 증가하지 않고 1로 표시되지만, 여러 개의 Sub Select 문장이 포함된 쿼리의 경우 id 값이 증가하면서 표시된다.

```mysql
EXPLAIN
SELECT ((SELECT COUNT(*) FROM employees) + (SELECT COUNT(*) FROM salaries)) AS total_count;
```

| id | select_type |    table    | type  |     key     | ref  |  rows  |     Extra      |
|:---|:-----------:|:-----------:|:-----:|:-----------:|:----:|:------:|:--------------:|
| 1  |   PRIMARY   |    NULL     | NULL  |    NULL     | NULL |  NULL  | No tables used |
| 3  |  SUBQUERY   | departments | index | ix_deptname | NULL |   9    |  Using index   |
| 2  |  SUBQUERY   |  employees  | index | ix_hiredate | NULL | 300252 |  Using index   |

## select_type 컬럼

각 단위 SELECT 쿼리가 어떤 타입의 쿼리인지 표시되는 컬럼

### SIMPLE

UNION이나 서브쿼리를 사용하지 않는ㄴ 단순한 SELECT 쿼리

### PRIMARY

UNION이나 서브쿼리를 가지는 SELECT 쿼리의 실행 계획에서 가장 바깥쪽에 있는 단위 쿼리

### UNION

UNION으로 결합하는 단위 SELECT 쿼리 가운데 첫 번째를 제외한 두 번째 이후 단위 SELECT 쿼리

### DEPENDENT UNION

UNION이나 UNION ALL로 집합을 결합하는 쿼리에서 표시  
DEPENDENT는 UNION이나 UNION ALL로 결합된 단위 쿼리가 외부 쿼리에 의해 영향을 받는 것을 의미

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

UNION 결과를 담아두는 임시 테이블을 생성하는 단위 쿼리를 의미, 단위 쿼리가 아니기 때문에 id 값이 NULL로 표시된다.

### SUBQUERY

select_type의 SUBQUERY는 FROM 절 이외에서 사용되는 서브쿼리만을 의미(FROM절에 사용된 서브쿼리는 DERIVED로 표시)

### DEPENDENT SUBQUERY

바깥쪽 SELECT 쿼리에 의해 영향을 받는 서브쿼리, 외부 쿼리가 먼저 수행되야하므로 일반 서브쿼리보다 성능이 떨어질 수 있다.

### DERIVED

FROM 절에서 단위 SELECT 쿼리의 실행 결과로 메모리나 디스크에 임시 테이블을 생성하게 되는 쿼리

### DEPENDENT DERIVED

FROM 절에서 LATERAL 키워드를 사용하여 외부 컬럼을 참조하는 서브쿼리를 의미

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

###### 참고자료

- [Real MySQL 8.0 1 - 개발자와 DBA를 위한 MySQL 실전 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Real+MySQL&isbn=9791158392703&cipId=228440237%2C)