---
layout: editorial
---

# Index(인덱스)

> 데이터베이스 쿼리의 성능에서 뺴놓을 수 없는 요소 중 하나

데이터베이스에서 인덱스란, 컬럼들의 값과 해당 레코드가 저장된 주소를 키와 값의 쌍(key-value pair)으로 저장하는 자료구조이다.

- 장점: 빠른 데이터 조회
- 단점: 인덱스를 관리하기 위한 추가적인 자원이 필요하며, 데이터를 추가하거나 수정, 삭제하는 작업이 느려질 수 있음

실제로 인덱스를 적용하면 생각 이상으로 큰 성능 향상을 가져올 수 있다.

```sql
SELECT {{SAME QUERY ...}}
-- [2023-04-14 02:52:38] 8 rows retrieved starting from 1 in 3 s 892 ms (execution: 3 s 876 ms, fetching: 16 ms)
-- CREATE INDEX ... completed in 29 s 426 ms
SELECT {{SAME QUERY ...}}
-- [2023-04-14 02:53:19] 8 rows retrieved starting from 1 in 73 ms (execution: 55 ms, fetching: 18 ms)
```

## 인덱스의 종류

인덱스는 보통 키(key)라는 말과 혼용해서 사용하는데, 인덱스를 역할별로 구분하면 다음과 같다.

- 기본키(primary key): 테이블에서 특정 레코드를 패표하는 컬럼의 값으로 사용되는 인덱스로, NULL 값을 가질 수 없으면서 중복된 값을 가질 수 없다.
- 보조키(secondary key): 기본키를 제외한 나머지 모든 인덱스

###### 참고자료

- [Real MySQL 8.0 1 - 개발자와 DBA를 위한 MySQL 실전 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Real+MySQL&isbn=9791158392703&cipId=228440237%2C)
