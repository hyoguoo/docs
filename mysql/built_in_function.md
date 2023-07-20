---
layout: editorial
---

# 내장 함수

DBMS 종류와 관계없이 기본적인 기능의 SQL 함수는 대부분 동일하게 제공되지만, 이름이나 사용법에 표준이 없어 각 DBMS 마다 호환되지 않는다.  
많은 함수들이 존재하며 그 중에서도 주의해야 할 함수들은 아래와 같다.

## 현재 시간 조회

현재 시간을 조회하는 함수로는 `NOW()` / `SYSDATE()` 가 존재하는데, 같은 기능을 수행하지만 작동 방식에서 차이가 있다.

```mysql
SELECT NOW()     AS now
     , SYSDATE() AS sysdate
     , SLEEP(2)
     , NOW()     AS now_after_sleep
     , SYSDATE() AS sysdate_after_sleep;
# now                   : 2021-08-17 22:38:00
# sysdate               : 2021-08-17 22:38:00
# now_after_sleep       : 2021-08-17 22:38:00
# sysdate_after_sleep   : 2021-08-17 22:38:02
```

NOW() 함수는 하나의 SQL에서 같은 값을 가지지만, SYSDATE() 함수는 SQL 내에서도 호출 시점에 따라 다른 값을 가지게 된다.    
SYSDATE() 함수는 레플리카 서버에서의 안정성이나 인덱스 효율에서 문제가 발생할 수 있기 때문에 NOW() 함수를 사용하는 것이 좋다.

## GROUP_CONCAT - GROUP BY 문자열 결합

GROUP BY 절을 사용하여 레코드들을 연결하는 함수로, 값들을 연결하기 위해 제한적인 메모리 버퍼 공간을 사용한다.  
기본적으로 1024 바이트의 메모리 버퍼를 사용하며, 이를 초과하는 경우에는 `max_group_concat_len` 시스템 변수를 통해 설정할 수 있다.

## 벤치마크

벤치마크 함수는 인수로 받은 표현식을 반복하여 실행하고, 실행 시간을 초 단위로 반환하는 함수로, 특정 SQL의 성능을 측정할 때 사용한다.  
횟수만큼 직접 쿼리를 실행하는 것과는 다르기 때문에(네트워크, 쿼리 파싱 및 최적화 비용 등이 제외됨) 주의해야 하며, 두 개의 동일 기능을 상대적으로 비교할 때 사용하는 것이 좋다.

## 그 외

그 외에 일반적으론 사용하지 않지만 IP 관련 함수 / JSON / Geometry 같은 함수들도 존재한다.

###### 참고자료

- [Real MySQL 8.0 2 - 개발자와 DBA를 위한 MySQL 실전 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Real+MySql+8.0&isbn=9791158392727&cipId=228440238%2C)