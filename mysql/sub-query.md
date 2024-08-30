---
layout: editorial
---

# Sub Query(서브쿼리)

MYSQL 5.6까지는 서브쿼리를 최적으로 실행하지 못 했으나, 8.0부터는 서브쿼리를 최적화하여 실행하게 되어 성능이 향상되었다.  
서브쿼리가 사용되는 위치 별로 최적화 방법이 다르기 때문에, 작성하는 방법에 따라 성능이 달라질 수 있다.

## SELECT 절에서의 서브쿼리

내부적으로 임시 테이블을 만들거나 쿼리를 비효율적으로 실행하게 만들진 않기 때문에 서브쿼리가 적절히 인덱스를 사용한다면 크게 주의할 사항은 없다.  
일반적으로 SELECT 절에서 서브쿼리는 항상 컬럼과 레코드가 하나인 결과를 반환해야 한다.

### JOIN

서브쿼리 말고 조인으로 처리되는 경우가 많은데, 이 경우엔 보통 조인으로 처리할 때 더 빠르기 때문에 가능하면 조인을 사용하는 것이 좋다.

```sql
-- 서브쿼리
SELECT COUNT(CONCAT(e1.first_name,
                    (SELECT e2.first_name FROM employees e2 WHERE e2.emp_no = e1.emp_no))
           )
FROM employees e1;

-- 조인
SELECT COUNT(CONCAT(e1.first_name, e2.first_name))
FROM employees e1,
     employees e2
WHERE e1.emp_no = e2.emp_no;
```

그리고 서브쿼리가 동일하게 여러 번 사용 되는 경우엔 LATERAL JOIN을 사용하면 빠른 성능을 기대할 수 있다.

```sql
-- 서브쿼리
SELECT e.emp_no,
       e.first_name,
       (SELECT s.salary
        FROM salaries s
        WHERE s..emp_no = e.emp_no
        ORDER BY s.from_date DESC
        LIMIT 1) AS salary,
       (SELECT s.from_date
        FROM salaries s
        WHERE s.emp_no = e.emp_no
        ORDER BY s.from_date DESC
        LIMIT 1) AS from_date,
       (SELECT s.to_date
        FROM salaries s
        WHERE s.emp_no = e.emp_no
        ORDER BY s.from_date DESC
        LIMIT 1) AS to_date
FROM employees e
WHERE e.emp_no = 5959;

-- 조인
SELECT e.emp_no,
       e.first_name,
       s2.salary,
       s2.from_date,
       s2.to_date
FROM employees e
         INNER JOIN LATERAL (
    SELECT *
    FROM salaries s
    WHERE s.emp_no = e.emp_no
    ORDER BY s.from_date DESC
    LIMIT 1) s2 ON s2.emp_no = e.emp_no
WHERE e.emp_no = 5959;
```

이렇게 되면 서브쿼리가 3번 사용되어 추가로 3번의 쿼리가 실행되는 것을, 한 번만 실행하게 되어 성능이 향상된다.

## FROM 절에서의 서브쿼리

5.6 이하 버전에서는 FROM 절의 서브쿼리는 항상 결과를 임시 테이블에 저장하는 방식으로 처리되었지만, 이후엔 서브쿼리를 외부 쿼리로 병합하는 최적화를 수행하도록 개선됐다.  
하지만 서브쿼리 내에 아래와 같은 기능이 포함하고 있으면 외부 쿼리로 병합할 수 없다.

- 집합 함수 사용(`COUNT`, `SUM`, `MAX`, `MIN` 등)
- DISTINCT 사용
- GROUP BY 사용
- HAVING 사용
- LIMIT 사용
- UNION 사용
- SELECT 절 내에서 서브쿼리 사용
- 사용자 변수 사용(사용자 변수에 값이 할당되는 경우)

## WHERE 절에서의 서브쿼리

WHERE 절에서의 서브쿼리는 아래 세 가지 형태로 사용하며 동작 방식이 각각 다르다.

### 동등 또는 부등 연산자와 함께 사용

5.5 이전 버전에서는 서브쿼리가 실행될 때마다 외부 쿼리의 레코드 수만큼 서브쿼리가 실행되었지만,
서브 쿼리를 먼저 실행한 후 상수로 변환한 뒤 나머지 쿼리를 실행하는 방식으로 개선되었다.

### IN 연산자와 함께 사용

테이블의 레코드가 다른 테이블의 레코드를 이용한 표현식이나 컬럼과 일치하는지 체크하는 형태를 세미 조인이라고 한다.  
IN절 또한 세미 조인으로 보는데, 5.5 버전까지는 최적화가 되지 않았으나 8.0 버전 부터는 최적화가 많이 개선되었기 때문에, 성능상 큰 문제는 없으니 사용해도 된다.

### NOT IN 연산자와 함께 사용

위와 반대로 안티 세미 조인이라고 하는데, Not-Equal 연산자가 인덱스를 제대로 활용할 수 없듯이 NOT IN 연산자도 최적화할 수 있는 방법이 많이 없기 때문에 사용하지 않는 것이 좋다.

###### 참고자료

- [Real MySQL 8.0 (2권)](https://kobic.net/book/bookInfo/view.do?isbn=9791158392727)
