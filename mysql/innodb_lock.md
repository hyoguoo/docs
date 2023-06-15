# InnoDB 스토리지 엔진 락

- InnoDB 스토리지 엔진은 레코드 기반의 잠금 기능을 제공
- 잠금 정보가 상당히 작은 크기의 메모리로 관리
- 레코드 락이 페이지락 혹은 테이블 락으로 범위가 커지는 경우(락 에스컬레이션)가 없음

## 레코드 락(Record Lock)

- 레코드 자체만을 잠그는 락으로 InooDB에서는 레코드 자체를 잠그는 것이 아니라 인덱스의 레코드를 잠그는 차이가 존재(자체를 잠금 <--> 인덱스를 잠금에는 큰 차이 존재)
- 인덱스가 하나도 없는 테이블에 대해서도 자동 생성된 클러스터 인덱스를 이용해 레코드 락을 획득

## 갭 락(Gap Lock)

- 레코드와 바로 인접한 레코드 사이의 간격만을 잠그는 락
- 해당 간격에 새로운 레코드가 생성(INSERT)되는 것을 제어하는 역할
- 보통 넥스트 키 락의 일부로 사용되며, 그 자체로는 잘 사용되지 않음

## 넥스트 키 락(Next-Key Lock)

- 레코드 락과 갭 락을 합쳐 놓은 형태의 잠금
- 바이너리 로그에 기록되는 쿼리가 레플리카 서버에서 실행될 때 소스 서버의 결과와 동일하게 나오도록 보장하는 기능

## 자동 증가 락(Auto-Increment Lock)

- MySQL의 `AUTO_INCREMENT` 키워드를 사용해 자동으로 증가하는 컬럼에 대해 잠금을 걸어주는 락
- 각 레코드가 중복되지 않고 저장된 순서대로 증가하는 것을 보장하기 위해 테이블 수준의 잠금을 걸어줌
- 새로운 레코드를 저장하는 `INSERT` 쿼리에서만 사용
- 명시적으로 획득하거나 해제할 수 없음

## 인덱스와 잠금

InnoDB의 잠금과 인덱스는 중요한 연관 관계가 존재하는데, `레코드 락`에서 인덱스를 잠그는 방식으로 처리하게 된다.  
인덱스를 잠그게 되면, 변경할 레코드를 찾기 위해 검색한 인덱스의 레코드를 모두 락이 걸리게 된다.

```sql
-- 테이블 정보
-- TABLE NAME: employees
-- KEY ix_firstname (first_name)
SELECT COUNT(*)
FROM employees; -- 300000
SELECT COUNT(*)
FROM employees
WHERE first_name = 'Kwon'; -- 253
SELECT COUNT(*)
FROM employees
WHERE first_name = 'Kwon'
  AND last_name = 'Ogu'; -- 1

UPDATE employees
SET hire_date = NOW()
WHERE first_name = 'Kwon'
  AND last_name = 'Ogu';
```

- 위의 실행 문에서 `UPDATE` 문장은 단 한 건의 레코드만 변경
- 하지만 이 문장의 조건에서 인덱스를 이용할 수 있는 조건은 `first_name` 컬럼 하나만 존재
- 때문에 `first_name` 컬럼의 인덱스를 잠그게 되고, 이에 따라 `first_name` 컬럼의 값이 `Kwon`인 모든 레코드가 락 생성
- 만약 테이블에 인덱스가 하나도 없었다면 테이블을 풀 스캔하면서 `UPDATE` 작업을 수행하게 되어 모든 30만 건의 레코드가 락이 걸리게 때문에 인덱스 설계가 매우 중요

## 레코드 수준의 잠금 확인 및 해제

테이블 잠금은 잠금의 대상이 테이블 자체이므로 쉽게 문제 파악이 되지만, 레코드 수준의 잠금은 잠금의 대상이 레코드이기 때문에 잠금이 걸려있는지 확인하기가 어렵다.  
이를 확인하기 위해 레코드 잠금과 잠금 대기에 대한 조회를 하는 쿼리를 사용할 수 있다.  
만약 아래와 같은 시나리오를 가정하게 된다면

|                            커넥션1                             |                            커넥션2                            |                                     커넥션3                                     |
|:-----------------------------------------------------------:|:----------------------------------------------------------:|:----------------------------------------------------------------------------:|
|                          `BEGIN;`                           |                                                            |                                                                              |
| `UPDATE employees SET birth_date=NOW() WHERE emp_no=10001;` |                                                            |                                                                              |
|                                                             | `UPDATE employees SET hire_date=NOW() WHERE emp_no=10001;` |                                                                              |
|                                                             |                                                            | `UPDATE employees SET hire_date=NOW(), birth_date=NOW() WHERE emp_no=10001;` |

커넥션 1이 아직 `COMMIT`을 실행하지 않은 상태이므로 해당 레코드의 잠금을 그대로 가지고 있으며, 커넥션 2 / 커넥션 3은 해당 레코드의 잠금을 대기하고 있다.  
(MySQL 8.0 기준) `performance_schema` 테이블을 이용하여 잠금과 대기 순서를 확인할 수 있다.

```sql
SELECT r.trx_id              waiting_trx_id,
       r.trx_mysql_thread_id waiting_thread,
       r.trx_query           waiting_query,
       b.trx_id              blocking_trx_id,
       b.trx_mysql_thread_id blocking_thread,
       b.trx_query           blocking_query
FROM performance_schema.data_lock_waits w
         INNER JOIN information_schema.innodb_trx b
                    ON b.trx_id = w.blocking_engine_transaction_id
         INNER JOIN information_schema.innodb_trx r
                    ON r.trx_id = w.requesting_engine_transaction_id
```

| waiting_trx_id | waiting_thread |     waiting_query     | blocking_trx_id | blocking_thread |    blocking_query     |
|:--------------:|:--------------:|:---------------------:|:---------------:|:---------------:|:---------------------:|
|  0x7f9b1c0003  |       3        | `UPDATE employees...` |  0x7f9b1c0002   |        2        | `UPDATE employees...` |
|  0x7f9b1c0003  |       3        | `UPDATE employees...` |  0x7f9b1c0001   |        1        |         NULL          |
|  0x7f9b1c0002  |       2        | `UPDATE employees...` |  0x7f9b1c0001   |        1        |         NULL          |

위 결과를 보고 아래와 같은 사항을 확인할 수 있다.

- 2번 스레드는 1번 스레드가 락을 풀 때까지 대기
- 3번 스레드는 2번 스레드와 1번 스레드가 락을 풀 때까지 대기

여기서 만약 1번 스레드를 종료(`KILL 1`)하게 되면 2번 스레드와 3번 스레드는 락을 획득할 수 있게 된다.

###### 참고자료

- [Real MySQL 8.0 1 - 개발자와 DBA를 위한 MySQL 실전 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Real+MySQL&isbn=9791158392703&cipId=228440237%2C)