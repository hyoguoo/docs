---
layout: editorial
---

# Literal Notation(리터럴 표기)

## 문자열

SQL 표준에서 문자열은 항상 홑따옴표(')를 사용해서 표시하지만, MySQL에서는 홑따옴표와 쌍따옴표를 모두 사용할 수 있다.  
만약 문자열 내에 홑따옴표나 쌍다옴표가 포함돼 있을 때 아래와 같이 사용할 수 있다.

```sql
SELECT *
FROM department
WHERE dept_no = 'd''001'; # 표준

SELECT *
FROM department
WHERE dept_no = 'd"001'; # 표준

SELECT *
FROM department
WHERE dept_no = "d'001"; # MySQL에서만 사용 가능

SELECT *
FROM department
WHERE dept_no = "d""001"; # MySQL에서만 사용 가능
```

MySQL에서 쌍따옴표를 문자열 리터럴 표기에 사용하는 것을 방지하려면 SQL_MODE 시스템 변수값에 `ANSI_QUOTES`를 추가하면 된다.

## 숫자

숫자를 SQL에 사용할 때는 따옴표 없이 숫자 값을 입력하면 되는데, MySQL에는 따옴표를 사용하더라도 비교 대상이 숫자 값(컬럼)이라면 숫자로 자동 형변환을 해주게 된다.  
그와 반대 상황도 마찬가지로 숫자 값이라면 문자열로 자동 형변환을 해주게 되는데 성능상 문제가 발생할 수 있기 때문에 가능하면 숫자 값은 숫자로, 문자열은 문자열로 비교하는 것이 좋다.

```sql
SELECT *
FROM tab_test
WHERE number_col = '59'; # '59'라는 값 하나만 숫자로 변환

SELECT *
FROM tab_test
WHERE string_col = 59; # string_col 컬럼의 모든 값이 숫자로 변환, 만약 숫자가 아닌 값이 있다면 에러 발생 가능성 존재
```

위 쿼리에서 모든 문자열 값을 숫자로 변환을 해야하고, 게다가 인덱스가 걸려 있더라도 형변환을 해야하기 때문에 이를 이용하지 못하게 된다.

## 날짜

보통 DBMS에서는 날짜 타입을 비교하거나 INSERT/UPDATE 하기 위해선 문자열을 DATGE 타입으로 변환해야하지만,  
MySQL에서는 정해진 형태의 날짜 포맷으로 표기하면 자동으로 DATE/DATETIME 값으로 변환해주기 때문에 변환 함수를 사용하지 않아도 된다.

## Boolean

`BOOL`/`BOOLEAN` 타입이 존재하지만, `TINYINT(1)` 타입과 동일한 동의어이다.

###### 참고자료

- [Real MySQL 8.0 2 - 개발자와 DBA를 위한 MySQL 실전 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Real+MySql+8.0&isbn=9791158392727&cipId=228440238%2C)
