---
layout: editorial
---

# UPDATE / DELETE

보통 애플리케이션에서는 UPDATE / DELETE 쿼리는 주로 하나의 테이블에 대해 한 건(혹은 소량) 레코드 처리를 위해 사용한다.  
하지만 MySQL 서버에서는 여러 테이블을 조인해서 한 개 이상의 테이블을 대상으로 UPDATE / DELETE 쿼리를 실행할 수 있다.

## UPDATE(DELETE) ... ORDER BY ... LIMIT n

WHERE 조건절에 일치하는 모든 레코드를 처리하는 것이 일반적이지만, ORDER BY + LIMIT 조합을 사용해 상위 n개의 레코드만 처리할 수 있다.  
너무 많은 건의 레코드를 처리하면 트랜잭션을 너무 오래 유지하게 되어 다른 트랜잭션의 성능에 영향을 줄 수 있기 때문에, LIMIT으로 조금씩 나눠서 처리하는 해결 방법으로도 사용한다.

## JOIN UPDATE

두 개 이상의 테이블을 조인해서 조인된 결과 레코드를 대상으로 UPDATE / DELETE 쿼리를 실행하는 기능으로, 주로 아래 두 가지 경우에 사용한다.

1. 조인된 테이블 중에서 특정 테이블의 컬럼 값을 다른 테이블의 컬럼에 업데이트하는 경우
2. 양쪽 테이블에 공통으로 존재하는 레코드만 찾아서 업데이트하는 경우

SELECT 쿼리와 마찬가지로 JOIN 순서에 따라 성능이 달라질 수 있기 때문에, 성능을 고려해서 적절한 JOIN 순서를 선택해야 한다.  
또한 일반적으로 조인되는 모든 테이블에 대해 읽기 참조만 되는 테이블은 읽기 잠금이 걸리고, 컬럼이 변경되는 테이블은 쓰기 잠금이 걸리기 때문에 웹 서비스 환경에선 주의해서 사용해야 한다.

## 여러 레코드 UPDATE

일반적으로 여러 개의 레코드를 하나의 쿼리로 처리하는 경우 동일한 레코드 값으로만 업데이트할 수 있었지만, 8.0 버전부터는 서로 다른 값으로 업데이트할 수 있다.  
아래와 같이 레코드 생성(Row Constructor)을 사용하면 여러 레코드를 하나의 쿼리로 처리할 수 있다.

```sql
UPDATE user_level ul
    INNER JOIN (VALUES ROW (1, 1), ROW (2, 4)) new_user_level (user_id, user_lv)
    ON new_user_level.user_id = ul.user_id
SET ul.user_lv = ul.user_lv + new_user_level.user_lv;
```

`VALUES ROW(...), ROW(...), ...` 문법은 임시 테이블을 생성하는 효과를 낼 수 있는데, 생성된 테이블에 조인하여 업데이트할 레코드를 찾아 업데이트하는 효과를 낼 수 있다.

## JOIN DELETE

기본적으로 `JOIN DELETE` 문장은 일반적인 단일 테이블 DELETE 문장과 다른 문법으로 쿼리를 작성해야 한다.

```sql
-- 3개의 테이블을 조인한 뒤 employees, dept_emp 테이블의 레코드만 삭제
DELETE e, de -- 삭제할 테이블을 지정
FROM employees e,
     dept_emp de,
     departments d
WHERE e.emp_no = de.emp_no
  AND de.dept_no = d.dept_no
  AND d.dept_no = 'd59';
```

SELECT 쿼리와 마찬가지로 JOIN 순서에 따라 성능이 달라질 수 있기 때문에, 성능을 고려해서 적절한 JOIN 순서를 선택해야 한다.

###### 참고자료

- [Real MySQL 8.0 (2권)](https://kobic.net/book/bookInfo/view.do?isbn=9791158392727)
