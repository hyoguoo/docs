---
layout: editorial
---

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

아래에 위치할수록 각 트랜잭션 간의 데이터 격리 정도가 높아지며, 동시 처리 성능도 떨어지게된다.  
격리 수준이 높아질수록 MySQL 서버 성능이 떨어지긴 하나, `SERIALIZABLE` 격리 수준이 아니라면 크게 성능에 영향을 주지 않는다.  
일반적인 온라인 서비스 용도의 데이터베이스에서는 `READ COMMITTED`(오라클)/`REPEATABLE READ`(MySQL) 격리 수준을 사용한다.

## READ UNCOMMITTED

각 트랜잭션의 변경 내용이 `COMMIT` / `ROLLBACK` 상관없이 다른 트랜잭션에서 조회할 수 있는 격리 수준이다.

|    트랜잭션 1    |    트랜잭션 2    |
|:------------:|:------------:|
|   `BEGIN`    |              |
| `INSERT OGU` |              |
|              | `SELECT OGU` |
|              |    정상 조회     |
|   `COMMIT`   |              |

만약 트랜잭션 1에서 `COMMIT`이 아닌 `ROLLBACK`을 수행했다면, 데이터가 존재하지 않게 된다.  
하지만 트랜잭션 2에서는 정상적으로 조회를 하게 되어 데이터가 불일치하게 된다.(`DIRTY READ` 발생)

## READ COMMITTED

오라클 DBMS에서 기본으로 사용되는 격리 수준으로, 온라인 서비스에서 가장 많이 사용되는 격리 수준이다.  
이 레벨에서는 `DIRTY READ`가 발생하지 않으며, 데이터를 변경했을 경우 `COMMIT`이 완료된 데이터만 다른 트랜잭션에서 조회할 수 있다.  
만약 조회하려는 데이터가 `COMMIT`이 완료되지 않았다면, 언두 영역에서 백업된 레코드를 조회한다.

|                 트랜잭션 1                  |      DATABASE       |              트랜잭션 2               |
|:---------------------------------------:|:-------------------:|:---------------------------------:|
|                 `BEGIN`                 | no: 59 name: DDUZY  |                                   |
| `UPDATE SET name = 'OGU' WHERE no = 59` |                     |                                   |
|                                         |                     |      `SELECT WHERE no = 59`       |
|                                         | 변경 전 데이터를 언두 로그로 복사 | DDUZY로 조회(언두 로그에 백업 된 이전 데이터를 조회) |
|                `COMMIT`                 |  no: 59 name: OGU   |                                   |

이 격리 수준에서도 한 트랜잭션에서 동일한 데이터를 여러 번 조회하면, 각각의 조회는 동일한 결과를 반환하지 않을 수 있다.(`NON-REPEATABLE READ` 발생)

|                  트랜잭션 1                   |      DATABASE      |              트랜잭션 2              |
|:-----------------------------------------:|:------------------:|:--------------------------------:|
|                                           |                    |             `BEGIN`              |
|                                           |  no: 59 name: OGU  |      `SELECT WHERE no = 59`      |
|                                           |                    |             OGU로 조회              |
|                  `BEGIN`                  |                    |                                  |
| `UPDATE SET name = 'DDUZY' WHERE no = 59` |                    |                                  |
|                 `COMMIT`                  | no: 59 name: DDUZY |                                  |
|                                           |                    | `SELECT WHERE no = 59`(같은 쿼리 실행) |
|                                           |                    |            DDUZY로 조회             |

일반적인 웹 애플리케이션에서는 문제가 없을 수 있으나, 하나의 트랜잭션에서 동일한 데이터를 여러 번 조회하고 변경하는 금전적인 거래와 같은 경우에는 문제가 발생할 수 있다.

## REPEATABLE READ

InnoDB 스토리지 엔진에서 기본으로 사용되는 격리 수준으로, 바이너리 로그를 가진 MySQL에서는 최소 `REPEATABLE READ` 격리 수준을 사용해야 한다.  
MVCC를 사용해 `READ COMMITTED` 격리 수준에서 발생하는 `NON-REPEATABLE READ` 문제를 해결하여 동일한 트랜잭션 내에서 동일한 결과를 보장한다.

### MVCC(Multi Version Concurrency Control)를 이용한 `Non-Repeatable Read` 방지

모든 InnoDB 트랜잭션에는 고유한 트랜잭션 번호(순차적 증가 값)가 부여되며, 언두 영역에 백업든 레코드에는 트랜잭션 번호가 저장된다.  
`REPEATABLE READ`은 언두 영역에 백업된 레코드 중 트랜잭션 번호를 기준으로 레코드를 조회하게 된다.

|                 트랜잭션 12                 |                          DATABASE                           |            트랜잭션 10            |
|:---------------------------------------:|:-----------------------------------------------------------:|:-----------------------------:|
|                                         |                TRX-ID: 6 no: 59 name: DDUZY                 |      `BEGIN(TRX-ID: 10)`      |
|                                         |                                                             |    `SELECT WHERE no = 59`     |
|                                         |                                                             |           DDUZY로 조회           |
|           `BEGIN(TRX-ID: 12)`           |                                                             |                               |
| `UPDATE SET name = 'OGU' WHERE no = 59` |                                                             |                               |
|                                         |                     변경 전 데이터를 언두 로그로 복사                     |                               |
|          `COMMIT(TRX-ID: 12)`           | TRX-ID: 12 no: 59 name: OGU<br>TRX-ID: 6 no: 59 name: DDUZY |                               |
|                                         |                                                             |    `SELECT WHERE no = 59`     |
|                                         |                                                             | 자신의 트랜잭션 번호보다 작은 번호 중 최근 것 조회 |
|                                         |                                                             |           DDUZY로 조회           |

트랜잭션 번호를 이용해 트랜잭션 12에서 데이터가 변경되었지만, 트랜잭션 10은 자신의 번호보다 작은 트랜잭션의 번호를 조회하여 언두 로그에 있는 DDUZY를 조회하게 된다.

### 넥스트 키 락을 이용한 `PHANTOM READ` 방지

위와 같이 잠금 없이 조회하는 경우엔 MVCC를 통해 데이터 무결성을 보장하지만, 베타적 잠금을 걸게 되면 `PHANTOM READ`가 발생할 수 있다.

#### 일반적인 DBMS 기준

일반적인 DBMS에서 베타적 잠금을 걸어 조회하게 되면 `PHANTOM READ`가 발생하여 동일한 쿼리를 실행했을 때 결과가 달라질 수 있다.

|                       트랜잭션 12                        |                           DATABASE                           |              트랜잭션 10               |
|:----------------------------------------------------:|:------------------------------------------------------------:|:----------------------------------:|
|                                                      |                 TRX-ID: 6 no: 59 name: DDUZY                 |          `BEGIN(TRX-10)`           |
|                                                      |                         59인 레코드만 잠금                          | `SELECT WHERE no >= 59 FOR UPDATE` |
|                                                      |                                                              |              DDUZY 조회              |
|                 `BEGIN(TRX-ID: 12)`                  |                                                              |                                    |
| `INSERT INTO t1 VALUES (60, 'OGU')`<br>(대기 없이 바로 실행) |                                                              |                                    |
|                 `COMMIT(TRX-ID: 12)`                 | TRX-ID: 6 no: 59 name: DDUZY<br>TRX-ID: 12 no: 60: name: OGU |                                    |
|                                                      |                                                              | `SELECT WHERE no >= 59 FOR UPDATE` |
|                                                      |                                                              |           DDUZY, OGU 조회            |

id = 59인 레코드에 대해서만 잠금이 걸리고, 트랜잭션 12의 요청은 잠금 없이 즉시 실행이 된다.  
위에서는 MVCC를 통해 트랜잭션 10이 DDUZY만 조회했지만, 베타적 잠금을 통해 조회했기 때문에 테이블의 데이터를 직접 읽어와 DDUZY와 OGU를 모두 조회하게 된다.

#### InnoDB 스토리지 MySQL 기준

하지만 MySQL에서는 넥스트 키 락이 존재하기 때문에 트랜잭션 12의 요청이 잠금을 획득하기 위해 대기하게 되고, 결국에는 트랜잭션 10은 DDUZY만 조회하게 된다.

|               트랜잭션 12               |                           DATABASE                           |              트랜잭션 10               |
|:-----------------------------------:|:------------------------------------------------------------:|:----------------------------------:|
|                                     |                 TRX-ID: 6 no: 59 name: DDUZY                 |          `BEGIN(TRX-10)`           |
|                                     |                 59뿐만 아니라 59 이상의 레코드에 대해서도 잠금                 | `SELECT WHERE no >= 59 FOR UPDATE` |
|                                     |                                                              |              DDUZY 조회              |
|         `BEGIN(TRX-ID: 12)`         |                                                              |                                    |
| `INSERT INTO t1 VALUES (60, 'OGU')` |                                                              |                                    |
|            넥스트 키 락 잠금 대기            |                                                              |                                    |
|                 ...                 |                                                              |      `SELECT WHERE no >= 59`       |
|                 ...                 |                                                              |              DDUZY 조회              |
|                 ...                 |                                                              |          `COMMIT(TRX-10)`          |
|        `COMMIT(TRX-ID: 12)`         | TRX-ID: 6 no: 59 name: DDUZY<br>TRX-ID: 12 no: 60: name: OGU |                                    |

갭락 존재로 일반적으로 MySQL에서는 `Phantom Read`가 발생하지 않지만, 발생 할 수 있는 시나리오는 다음과 같다.

1. 첫 번째 조회는 잠금 없이 조회
2. 두 번째 조회에서 베타적 잠금을 획득하여 조회

하지만 이렇게 조회하는 경우는 일반적으로 없기 때문에 `Phantom Read`가 거의 발생하지 않는다고 볼 수 있다.

## SERIALIZABLE

가장 단순하고 엄격한 격리 수준으로, 그만큼 동시 처리 성능이 다른 격리 수준보다 떨어진다.

- 읽기 작업도 공유 잠금(읽기 잠금)을 획득해야 함
- 잠금이 걸린 경우 다른 트랜잭션에서 해당 레코드를 변경하거나 삭제할 수 없음

하지만 InnoDB 스토리지 엔진에서는 `READ COMMITTED` 격리 수준에서도 `PHANTOM READ` 문제를 해결할 수 있어 해당 격리 수준을 사용하지 않는다.

###### 참고자료

- [Real MySQL 8.0 1 - 개발자와 DBA를 위한 MySQL 실전 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Real+MySQL&isbn=9791158392703&cipId=228440237%2C)
- [망나니개발자 티스토리](https://mangkyu.tistory.com/299)
