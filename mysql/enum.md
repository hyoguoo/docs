---
layout: editorial
---

# ENUM

테이블의 구조에 나열된 목록 중 하나의 값을 저장할 수 있는 타입으로, 코드화된 값을 관리할 때 사용한다.  
ENUM 타입의 장점으로 코드 값의 의미를 쉽게 파악할 수 있다는 점과, 문자열이 아닌 정수 값으로 저장하기 때문에 저장 공간을 절약할 수 있다는 점이 있다.  
(최대 갯수는 65,535개이며, 255개 미만은 저장 공간으로 1바이트, 255개 이상은 2바이트를 사용)

```mysql
CREATE TABLE tb_enum
(
    fd_enum ENUM ('PROCESSING', 'FAILURE', 'SUCCESS')
);

# 아래 두 쿼리는 동일한 결과를 반환한다.
# 1
SELECT *
FROM tb_enum
WHERE fd_enum = 1;
# 2
SELECT *
FROM tb_enum
WHERE fd_enum = 'PROCESSING';
```

ENUM 타입은 문자열처럼 비교하거나 저장할 수 있으며, 실제로 디스크나 멤모리에 저장하는 값은 값에 매핑된 정숫값을 저장한다.

ENUM 컬럼에 저장되는 값에 새로운 값을 추가해야하는 경우 테이블 구조를 변경해야하는 단점이 존재한다.  
5.6 이전 버전에서는 값을 추가할 때마다 항상 테이블을 리빌드 해야헀지만, 5.6 버전부터는 ENUM의 마지막에 추가되는 경우엔 테이블 구조(메타데이터) 변경만으로 처리할 수 있어 단점이 어느정도 해소되었다.  
(마지막에 추가하는 경우 INSTANT 알고리즘으로 메타데이터만 변경하지만, 순서 중간에 추가하는 경우 COPY 알고리즘에 읽기 잠금까지 필요하다.)

## ENUM 정렬

ENUM 타입은 실제로는 정수 값으로 저장되기 때문에 정렬 시에도 정수 값으로 정렬된다.  
때문에 ENUM 타입을 정렬할 때는 문자열로 정렬하고 싶은 경우 CAST 함수를 사용해야한다.

```mysql
SELECT *
FROM tb_enum
ORDER BY CAST(fd_enum AS CHAR); 
```

###### 참고자료

- [Real MySQL 8.0 2 - 개발자와 DBA를 위한 MySQL 실전 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Real+MySql+8.0&isbn=9791158392727&cipId=228440238%2C)