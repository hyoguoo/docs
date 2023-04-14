# 인덱스(Index)

> 데이터베이스 쿼리의 성능에서 뺴놓을 수 없는 요소 중 하나

컬럼들의 값과 해당 레코드가 저장된 주소를 키와 값의 쌍(key-Value pair)으로 삼아 인덱스를 만들어 놓아 빠르게 데이터를 찾을 수 있도록 도와주는 장치  
하지만 인덱스를 사용하면 데이터를 추가하거나 수정, 삭제하는 작업을 할 때마다 인덱스를 수정해야 하므로 해당 작업에서는 성능이 떨어질 수 있다.

- 실제 프로젝트에서 하나의 컬럼에 대한 B-TREE 인덱스 적용 전/후 시간 차이

```sql
SELECT {{SAME QUERY...}}
-- [2023-04-14 02:52:38] 8 rows retrieved starting from 1 in 3 s 892 ms (execution: 3 s 876 ms, fetching: 16 ms)
-- CREATE INDEX ... completed in 29 s 426 ms
SELECT {{SAME QUERY...}}
-- [2023-04-14 02:53:19] 8 rows retrieved starting from 1 in 73 ms (execution: 55 ms, fetching: 18 ms)
```

###### 출처

- [Real MySQL 8.0 1](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=284710853)
