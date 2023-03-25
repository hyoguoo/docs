# Isolation Level(격리 수준)

> 동시에 실행되는 트랜잭션들이 서로 영향을 미치는 정도

여러 트랜잭션이 동시에 처리될 때 특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있는 정도를 말한다.  
InnoDB 기준 격리 수준은 아래 표와 같이 4가지로 나뉜다.

|      격리 수준       | DIRTY READ | NON-REPEATABLE READ | PHANTOM READ |
|:----------------:|:----------:|:-------------------:|:------------:|
| READ UNCOMMITTED |     O      |          O          |      O       |
|  READ COMMITTED  |     X      |          O          |      O       |
| REPEATABLE READ  |     X      |          X          | O(InnoDB X)  |
|   SERIALIZABLE   |     X      |          X          |      X       |

아래에 위치하는 각 트랜잭션 간의 데이터 격리 정도가 높아지며, 동시 처리 성능도 떨어지게된다.  
격리 수준이 높아질수록 MySQL 서버 성능이 떨어지긴 하나, `SERIALIZABLE` 격리 수준이 아니라면 크게 성능에 영향을 주지 않는다.  
일반적인 온라인 서비스 용도의 데이터베이스에서는 `READ COMMITTED`(오라클)/`REPEATABLE READ`(MySQL) 격리 수준을 사용한다.

## READ UNCOMMITTED

각 트랜잭션에서의 변경 내용이 `COMMIT` / `ROLLBACK` 상관없이 다른 트랜잭션에서 조회할 수 있다.

|    트랜잭션 1    |    트랜잭션 2    |
|:------------:|:------------:|
|   `BEGIN`    |              |
| `INSERT OGU` |              |
|              | `SELECT OGU` |
|              | 정상적으로 결과 반환  |
|   `COMMIT`   |              |

만약 트랜잭션 1에서 `ROLLBACK`을 수행했다면, 트랜잭션 2에서는 정상적으로 조회를 하게 되어 데이터가 불일치하게 된다.(`DIRTY READ` 발생)

## READ COMMITTED

오라클 DBMS에서 기본으로 사용되는 격리 수준으로, 온라인 서비스에서 가장 많이 사용되는 격리 수준이다.  
이 레벨에서는 `DIRTY READ`가 발생하지 않으며, 데이터를 변경했을 경우 `COMMIT`이 완료된 데이터만 다른 트랜잭션에서 조회할 수 있다.  
만약 조회하려는 데이터가 `COMMIT`이 완료되지 않았다면, 언두 영역에서 백업된 레코드를 조회한다.

|                 트랜잭션 1                  |      DATABASE       |             트랜잭션 2              |
|:---------------------------------------:|:-------------------:|:-------------------------------:|
|                 `BEGIN`                 |                     |                                 |
| `UPDATE SET name = 'OGU' WHERE no = 59` |  no: 59 name: GUO   |                                 |
|                                         |                     |     `SELECT WHERE no = 59`      |
|                                         | 변경 전 데이터를 언두 로그로 복사 | GUO로 조회(언두 로그에 백업 된 이전 데이터를 조회) |
|                `COMMIT`                 |      데이터 정상 반영      |                                 |

이 격리 수준에서도 한 트랜잭션에서 동일한 데이터를 여러 번 조회하면, 각각의 조회는 동일한 결과를 반환하지 않을 수 있다.(`NON-REPEATABLE READ` 발생)

|                 트랜잭션 1                  |     DATABASE     |                트랜잭션 2                 |
|:---------------------------------------:|:----------------:|:-------------------------------------:|
|                                         |                  |                `BEGIN`                |
|                                         | no: 59 name: OGU |      `SELECT WHERE name = 'GUO'`      |
|                                         |                  |               데이터 조회 실패               |
|                 `BEGIN`                 |                  |                                       |
| `UPDATE SET name = 'GUO' WHERE no = 59` |                  |                                       |
|                `COMMIT`                 | no: 59 name: GUO |                                       |
|                                         |                  | `SELECT WHERE name = 'GUO'`(같은 쿼리 실행) |
|                                         |                  |               데이터 조회 성공               |

일반적인 웹 프로그램에서는 문제가 없을 수 있으나, 하나의 트랜잭션에서 동일한 데이터를 여러 번 조회하고 변경하는 금전적인 거래와 같은 경우에는 문제가 발생할 수 있다.

## REPEATABLE READ

MySQL InnoDB 스토리지 엔진에서 기본으로 사용되는 격리 수준으로, 바이너리 로그를 가진 MySQL에서는 최소 `REPEATABLE READ` 격리 수준을 사용해야 한다.  
이 레벨에서는 위의 `NON-REPEATABLE READ` 문제가 발생하지 않으며, 트랜잭션 내에서 실행된 `SELECT` 쿼리는 `COMMIT`이 완료되기 전까지는 계속 같은 데이터를 조회하게 된다.

### MVCC(Multi Version Concurrency Control)

MVCC란 InnoDB 스토리지 엔진이 트랜잭션이 `ROLLBACK`될 가능성에 대비해 변경되기 전 레코드를 언두 공간에 백업해두고 실제 레코드 값을 변경하는 방식이다.  
`READ COMMITTED` 격리 수준에서도 MVCC를 이용해 `COMMIT`되기 전의 데이터를 보여주지만  
`REPEATABLE READ`와 `READ COMMITED`의 차이는 언두 영역에 백업된 레코드의 여러 버전 가운데 어떤 버전을 조회할지 결정하는 것에 차이가 있다.  
모든 InnoDB 트랜잭션에는 고유한 트랜잭션 번호(순차적 증가 값)가 부여되며, 언두 영역에 백업든 레코드에는 트랜잭션 번호가 저장된다.  
언두 영역에 백업된 데이터는 InnoDB 엔진이 불필요하다고 판단하는 시점에 오래된 트랜잭션 번호가 부여된 언두 영역 데이터를 삭제한다.

|                 트랜잭션 12                 |                                 DATABASE                                 |            트랜잭션 10            |
|:---------------------------------------:|:------------------------------------------------------------------------:|:-----------------------------:|
|                                         |                        TRX-ID: 6 no: 59 name: GUO                        |      `BEGIN(TRX-ID: 10)`      |
|                                         |                                                                          |    `SELECT WHERE no = 59`     |
|                                         |                                                                          |            GUO로 조회            |
|           `BEGIN(TRX-ID: 12)`           |                                                                          |                               |
| `UPDATE SET name = 'OGU' WHERE no = 59` |                                                                          |                               |
|                                         |                           변경 전 데이터를 언두 로그로 복사                            |                               |
|          `COMMIT(TRX-ID: 12)`           | 테이블<br>TRX-ID: 12 no: 59 name: OGU<br>언두로그<br>TRX-ID: 6 no: 59 name: GUO |                               |
|                                         |                                                                          |    `SELECT WHERE no = 59`     |
|                                         |                                                                          | 자신의 트랜잭션 번호보다 작은 번호 중 최근 것 조회 |
|                                         |                                                                          |            GUO로 조회            |

트랜잭션 번호를 이용해 트랜잭션 12에서 데이터를 변경했지만, 트랜잭션 10은 자신의 번호보다 작은 트랜잭션의 번호만 조회할 수 있어 언두 로그에 있는 GUO를 조회하게 된다.

### SELECT & TRANSACTION

트랜잭션 내에서 실행되는 `SELECT` 쿼리와 없이 실행되는 `SELECT` 쿼리는 `READ COMMITTED` 격리 수준에서는 동일한 결과를 반환한다.  
하지만 `REPEATABLE READ` 격리 수준에서는 위의 예제와 같은 상황에서는 트랜잭션 내에서 실행된 `SELECT` 쿼리는 `COMMIT`이 완료되기 전까지는 계속 같은 데이터를 조회하게 된다.

### PHANTOM READ

`REPEATABLE READ` 격리 수준에서도 `INSERT`를 실행하게 되면 첫 번째 `SELECT` 쿼리와 두 번째 `SELECT` 쿼리의 결과가 다르게 나타난다.  
이렇게 다른 트랜잭션에 의해 레코드 결과가 변경되는 것을 `PHANTOM READ`라고 한다.

|               트랜잭션 12               |                          DATABASE                          |              트랜잭션 10               |
|:-----------------------------------:|:----------------------------------------------------------:|:----------------------------------:|
|                                     |                 TRX-ID: 6 no: 59 name: GUO                 |          `BEGIN(TRX-10)`           |
|                                     |                                                            | `SELECT WHERE no >= 59 FOR UPDATE` |
|                                     |                                                            |               GUO 조회               |
|         `BEGIN(TRX-ID: 12)`         |                                                            |                                    |
| `INSERT INTO t1 VALUES (60, 'OGU')` |                                                            |                                    |
|        `COMMIT(TRX-ID: 12)`         | TRX-ID: 6 no: 59 name: GUO<br>TRX-ID: 10 no: 60: name: OGU |                                    |
|                                     |                                                            | `SELECT WHERE no >= 59 FOR UPDATE` |
|                                     |                                                            |            GUO, OGU 조회             |

## SERIALIZABLE

가장 단순하고 엄격한 격리 수준으로, 그만큼 동시 처리 성능이 다른 격리 수준보다 떨어진다.  
읽기 작업도 공유 잠금(읽기 잠금)을 획득해야 하며, 동시에 다른 트랜잭션은 해당 레코드를 변경하지 못하게 된다.  
이 방식을 사용하면 `PHANTOM READ` 문제도 발생하지 않게 된다.  
하지만 InnoDB 스토리지 엔진에서는 갭 락과 넥스트 키 락을 사용해 `READ COMMITTED` 격리 수준에서도 `PHANTOM READ` 문제를 해결할 수 있기 때문에  
`SERIALIZABLE` 격리 수준은 거의 사용되지 않는다.

###### 출처

- [Real MySQL 8.0 1](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=284710853)