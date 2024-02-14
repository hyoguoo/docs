---
layout: editorial
---

# InnoDB Storage Engine Lock

InnoDB 스토리지 엔진은 다른 엔진과는 다르게 레코드 기반의 잠금 기능을 제공한다는 가장 큰 특징이 있다.  
또한 잠금 정보를 작은 크기의 메모리로 관리하기 때문에 레코드 락이 페이지락/테이블 락으로 확장되는 경우(락 에스컬레이션)가 없다.

## 레코드 락(Record Lock)

테이블 전체가 아닌 레코드 단위로 잠그는 락으로 레코드 자체를 잠그는 것이 아니라 인덱스의 레코드를 잠그는 방식으로 처리한다.  
(인덱스가 하나도 없는 테이블의 경우 자동 생성된 클러스터 인덱스에 대한 레코드 락을 획득하여 처리한다.)

### 레코드 자체 잠금 & 인덱스의 레코드 잠금 차이

레코드 락은 테이블 레코드가 아닌 인덱스를 잠그는 방식으로 처리하게 된다.  
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
- 인덱스를 통해 스캔할 수 없는 상황에는 레코드 조회 시 테이블을 풀 스캔하면서 30만 건의 레코드 전부 락이 걸리게 됨

## 갭 락(Gap Lock)

레코드가 지정된 범위에 해당하는 인덱스 테이블 공간을 대상으로 거는 잠금으로, 다른 데이터가 삽입되는 것을 방지하기 위해 사용된다.

```sql
-- 51 ~ 55 사이의 레코드에 대해 베타적 잠금 획득하는 쿼리
SELECT *
FROM member
WHERE 51 <= age
  AND age <= 55
    FOR
UPDATE;
```

age 52 / 53을 가진 레코드가 있을 때, 위 쿼리가 실행된다면, 실제 존재하는 레코드 뿐만 아니라, 51과 55 사이의 공간에 대해 갭 락이 걸리게 된다.

| age | PK  |        잠금 여부        |
|:---:|:---:|:-------------------:|
| ... | ... |         ...         |
| 50  | 59  |          X          |
|     |     | 갭 락(51 ~ 52 사이의 공간) |
| 52  | 61  |        레코드 락        |
| 53  | 62  |        레코드 락        |
|     |     | 갭 락(53 ~ 55 사이의 공간) |
| 56  | 65  |          X          |

** 갭 락은 보통 넥스트 키 락의 일부로 사용되며, 그 자체 하나만으론 잘 사용되지 않는다.

## 넥스트 키 락(Next-Key Lock)

레코드와 그 다음 레코드 사이의 갭에 대해 잠금을 걸어주는 락으로, 갭 락과 레코드 락을 합친 형태다.

| age | PK  |
|:---:|:---:|
| ... | ... |
| 50  | 59  |
| 52  | 61  |
| 52  | 62  |
| 56  | 65  |
| ... | ... |

```sql
INSERT into member (age)
VALUES (52);
```

위 테이블 상황에서 `age` 컬럼에 대해 `52` 값을 가진 레코드를 삽입을 시도하게 되면 아래 두 가지의 락이 걸리게 된다.

- 레코드 락: `52` 값을 가진 레코드
- 갭 락: `50` ~ `52` 사이의 공간 + `52` ~ `56` 사이의 공간

## 자동 증가 락(Auto-Increment Lock)

자동으로 증가하는 컬럼(`AUTO_INCREMENT`)에 대해 잠금을 걸어주는 락으로 다음과 같은 특징이 있다.

- 내부적으로 테이블 수준의 잠금을 걸어 중복되지 않는 값을 보장
- 명시적으로 획득하거나 해제할 수 없음
- MySQL 8.0에서는 잠금을 걸지 않는 방법을 기본 값으로 사용 중이며 `innodb_autoinc_lock_mode` 시스템 변수를 이용하여 변경 가능

## 레코드 수준의 잠금 확인 및 해제

테이블 잠금은 잠금의 대상이 테이블 자체이므로 쉽게 문제 파악이 되지만, 레코드 수준의 잠금은 걸려있는지 확인하기가 어렵다.

|                            커넥션1                             |                            커넥션2                            |                                     커넥션3                                     |
|:-----------------------------------------------------------:|:----------------------------------------------------------:|:----------------------------------------------------------------------------:|
|                          `BEGIN;`                           |                                                            |                                                                              |
| `UPDATE employees SET birth_date=NOW() WHERE emp_no=10001;` |                                                            |                                                                              |
|                                                             | `UPDATE employees SET hire_date=NOW() WHERE emp_no=10001;` |                                                                              |
|                                                             |                                                            | `UPDATE employees SET hire_date=NOW(), birth_date=NOW() WHERE emp_no=10001;` |

위 시나리오는 커넥션 1이 아직 `COMMIT`을 실행하지 않은 상태이므로 해당 레코드의 잠금을 그대로 가지고 있으며,  
커넥션 2 / 커넥션 3은 해당 레코드의 잠금을 대기하고 있음을 확인할 수 있다.

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

MySQL 8.0 기준 `performance_schema` 테이블을 이용하여 잠금과 대기 순서를 확인할 수 있다.

| waiting_trx_id | waiting_thread |     waiting_query     | blocking_trx_id | blocking_thread |    blocking_query     |
|:--------------:|:--------------:|:---------------------:|:---------------:|:---------------:|:---------------------:|
|  0x7f9b1c0003  |       3        | `UPDATE employees...` |  0x7f9b1c0002   |        2        | `UPDATE employees...` |
|  0x7f9b1c0003  |       3        | `UPDATE employees...` |  0x7f9b1c0001   |        1        |         NULL          |
|  0x7f9b1c0002  |       2        | `UPDATE employees...` |  0x7f9b1c0001   |        1        |         NULL          |

위 결과를 보고 각 스레드가 어떤 쿼리를 실행하고 어떤 스레드의 잠금을 대기하고 있는지 확인할 수 있다.

- 2번 스레드: 1번 스레드의 락 대기
- 3번 스레드: 2번 스레드 + 1번 스레드의 락 대기

###### 참고자료

- [Real MySQL 8.0 1 - 개발자와 DBA를 위한 MySQL 실전 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Real+MySQL&isbn=9791158392703&cipId=228440237%2C)
