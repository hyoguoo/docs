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

넥스트 키 락은 InnoDB에서 팬텀 리드(Phantom Read)를 방지하기 위해 사용하는 잠금 기법이다.

- 레코드 락과 갭 락을 결합한 형태
- 특정 레코드와 그 앞뒤 간격을 포함하는 범위에 대해 `(prev, current]` 방식으로 잠금 설정

즉, 쿼리에서 참조한 인덱스 레코드뿐만 아니라 그 이전과 다음 레코드와의 갭까지 함께 잠금이 걸리므로, 조회된 범위 내에 새로운 레코드가 삽입되는 것을 차단하는 데 사용된다.

### 예시

| age | PK  |
|:---:|:---:|
| 50  | 101 |
| 52  | 102 |
| 56  | 103 |

`age` 컬럼에 대한 인덱스를 가진 member 테이블이 있고, 다음과 같은 쿼리를 실행하면 InnoDB는 넥스트 키 락을 설정하게 된다.

```sql
SELECT * FROM member WHERE age = 52 FOR UPDATE;
```

위 쿼리는 단일 값(age = 52)만 조회하지만, 실제로는 다음과 같은 넥스트 키 락이 설정된다.

- (50, 52] 범위: 레코드 52와 그 앞 공간
- (52, 56] 범위: 레코드 52 다음 값과의 갭도 포함

이처럼 유니크 하지 않은 인덱스의 단일 값을 조회하더라도 넥스트 키 락은 인덱스 순서상 앞뒤 갭을 모두 포함해서 잠금하게 된다.

|   구분    |    범위    |            설명             |
|:-------:|:--------:|:-------------------------:|
| 넥스트 키 락 | (50, 52] |      레코드 52와 앞의 갭 포함      |
| 넥스트 키 락 | (52, 56] | 레코드 52 이후 인덱스상 다음 레코드와의 갭 |
|  레코드 락  |    52    |     명시적으로 선택된 레코드 자체      |

### 단일 조회 시에도 범위에 대해 넥스트 키 락이 걸리는 이유

락을 이용한 읽기, UPDATE, DELETE와 같은 명령문은 SQL 명령문 처리 시 InnoDB는 정확한 WHERE 조건을 기억하지 않고, 스캔된 인덱스 범위에 대해 잠금을 설정한다.

- 유니크한 인덱스를 사용할 경우: InnoDB는 발견된 인덱스 레코드만 잠금
- 유니크하지 않은 인덱스 or 범위 조건: InnoDB는 스캔된 인덱스 범위에 대해 넥스트 키 락을 설정

## 자동 증가 락(Auto-Increment Lock)

자동으로 증가하는 컬럼(`AUTO_INCREMENT`)에 대해 잠금을 걸어주는 락으로 다음과 같은 특징이 있다.

- 내부적으로 테이블 수준의 잠금을 걸어 중복되지 않는 값을 보장
- 명시적으로 획득하거나 해제할 수 없음
- MySQL 8.0에서는 잠금을 걸지 않는 방법을 기본 값으로 사용 중이며 `innodb_autoinc_lock_mode` 시스템 변수를 이용하여 변경 가능(기본값: 1)

|        값         | AUTO_INCREMENT 값 연속성 | 동시 INSERT 처리 성능 | 락 동작 방식 설명                                                                                          |
|:----------------:|:--------------------:|:---------------:|:----------------------------------------------------------------------------------------------------|
| `0`(TRADITIONAL) |        항상 연속적        |  매우 낮음 (직렬 처리)  | 모든 INSERT에서 테이블 단위 락을 걸고 순차적으로 AUTO_INCREMENT 값을 할당                                                 |
| `1`(CONSECUTIVE) |       대부분 연속적        |       보통        | 삽입 수를 알 수 있는 단일 INSERT는 락을 최소한으로 걸음 / `INSERT ... SELECT` 같은 삽입 수를 알 수 없는 경우 구문이 끝날 때까지 테이블 단위 락 유지 |
| `2`(INTERLEAVED) |      연속성 보장 안 됨      |      매우 높음      | 락을 잡지 않아 빠르게 삽입하나 ID는 비연속적일 수 있음                                                                    |

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

- [Real MySQL 8.0 (1권)](https://kobic.net/book/bookInfo/view.do?isbn=9791158392703)
