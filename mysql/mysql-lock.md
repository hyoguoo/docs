---
layout: editorial
---

# MySQL Engine Lock

MySQL 엔진 레벨의 락은 모든 스토리지 엔진에 영향을 주게 되는 락으로, 아래와 같은 종류가 있다.

## 글로벌 락(Global Lock)

- 락의 범위가 MySQL 서버 전체로(작업 대상 테이블이나 데이터베이스가 아니더라도 포함), 제공하는 잠금 중 가장 범위가 큼
- 한 세션에서 글로벌 락을 획득하면 다른 세션에서 SELECT를 제외한 대부분의 DDL/DML 쿼리가 락이 해제될 때까지 대기
- 모든 테이블에 큰 영향을 미치기 때문에 웹 서비스에서는 사용하지 않는 것이 좋음
- `FLUSH TABLES WITH READ LOCK` 명령으로 획득하며, `UNLOCK TABLES` 명령으로 해제

## 백업 락(Backup Lock)

InnoDB 스토리지 엔진에서는 트랜잭션을 지원하기 때문에 글로벌 락을 사용할 필요가 거의 없어졌으며, 좀 더 가벼운 락인 백업 락을 사용한다.  
특정 세션에서 백업 락을 획득하면 아래의 작업이 제한되지만 일반적인 테이블의 데이터 변경은 허용된다.

- 데이터베이스 및 테이블 등의 변경 및 삭제
- REPAIR TABLE / OPTIMIZE TABLE 명령
- 사용자 관리 및 비밀번호 변경

MySQL 서버를 `소스 - 레플리카`로 구성하는 경우 주로 백업 작업은 레플리카 서버에서 이루어 지는데, 이 때 레플리카 서버에서 백업 락을 사용하여 백업을 수행하고 있다.

## 테이블 락(Table Lock)

개별 테이블 단위로 설정되는 잠금으로, 명시적 또는 묵시적으로 특정 테이블의 락을 획득할 수 있다.

- 명시적 락
    - `LOCK TABLES tbl_name [READ | WRITE]` 명령으로 획득
    - `UNLOCK TABLES` 명령으로 해제
    - 특별한 상황이 아니면 애플리케이션에서 사용할 일이 거의 없음
    - 명시적으로 테이블을 잠그는 작업은 글로벌 락과 동일하게 다른 작업에 큰 영향을 미침
- 묵시적 락
    - (MyISAM / MEMORY 한정)DML 쿼리를 실행하면 발생
    - 하지만 InnoDB 엔진에서는 스토리지 엔진 차원에서 레코드 기반의 잠금을 제공하기 때문에 DML 쿼리로 묵시적 락이 설정되지 않음
    - (모든 엔진 포함)DDL 쿼리로 스키마를 변경하는 경우 묵시적 락이 설정됨

## 네임드 락(Named Lock)

네임드 락은 `GET_LOCK()` 함수를 사용해 임의의 문자열에 대해 락을 설정할 수 있으며,  
잠금의 대상이 테이블/레코드 같은 데이터베이스 특정 객체가 아니라 지정한 문자열(String)에 대한 잠금이다.

```sql
-- lock_name 문자열을 10초 동안 락을 획득
SELECT GET_LOCK('lock_name', 10);

-- lock_name 문자열에 대해 잠금 확인
SELECT IS_FREE_LOCK('lock_name');

-- lock_name 문자열에 대해 잠금 해제
SELECT RELEASE_LOCK('lock_name');
```

트랜잭션이 종료될 때 자동으로 해제 되지 않기 때문에, 잠금 시간이 지나거나 명시적으로 해제해야 한다.

### 응용 - 분산 락

네임드 락은 아래와 같이 명시적으로 락을 설정하고 해제하는 방식으로 분산 락을 구현할 수 있다.

```java
// LockRepository.java
public interface LockRepository extends JpaRepository<Stock, Long> {

    @Query(value = "SELECT GET_LOCK(:key, 3000)", nativeQuery = true)
    void getLock(String key);

    @Query(value = "SELECT RELEASE_LOCK(:key)", nativeQuery = true)
    void releaseLock(String key);
}

// NamedLockService.java
public class NamedLockService {

    private LockRepository lockRepository;
    private StockService stockService;

    @Transactional
    public void decrease(Long id, Long quantity) {
        try {
            // 1. 락 획득, 이미 락이 설정되어 있는 경우 대기하게 됨
            lockRepository.getLock(id.toString());
            // 2. 재고 감소 로직
            stockService.decreaseStock(id, quantity);
        } finally {
            // 3. 락 해제
            lockRepository.releaseLock(key);
        }
    }
}
```

## 메타데이터 락(Metadata Lock)

데이터베이스 객체(테이블이나 뷰 등)의 이름이나 구조를 변경하는 경우에 획득하는 잠금으로,  
명시적으로 획득하거나 해제하는 것이 아닌 테이블의 이름을 변경하는 경우 자동으로 획득된다.

```sql
-- 테이블 이름 변경 1
RENAME TABLE rank TO rank_backup;
-- 테이블 이름 변경 2
RENAME TABLE rank_new TO rank;
```

위와 같이 두 개의 명령문으로 나누어 실행하면, 기존 `rank` 테이블에 접근하려는 다른 세션에서 `Table Not Found` 에러가 발생할 수 있다.

```sql
-- 테이블 이름 변경
RENAME TABLE rank TO rank_backup, rank_new TO rank;
```

하지만 위와 같이 RENAME TABLE 명령문에 두 개의 작업을 한 번에 실행하면  
두 테이블에 대해 전부 락을 획득하게 되어 다른 세션에서 `Table Not Found` 같은 에러가 발생하지 않는다.

### 응용 - 테이블 구조 변경

만약 서비스 유지 중 테이블 구조를 변경해야하는 상황이라면 아래와 같이 할 수 있다.

1. 새로운 구조의 테이블 생성
2. 최근(1시간 직전 또는 하루 전)까지의 데이터를 Primary Key인 id 값을 범위별로 나눠서 여러 개의 스레드로 복사
3. 트랜잭션을 auto commit으로 설정
4. 작업 대상 테이블에 대해 테이블 락을 설정
5. 기존 테이블의 데이터를 새로운 테이블로 복사
6. 모든 데이터 복사 완료 후 테이블 이름 변경
7. 테이블 쓰기 락 해제 및 테이블 삭제

```sql
-- 1. 새로운 구조의 테이블 생성
CREATE TABLE access_log_new
(
    id        BIGINT NOT NULL AUTO_INCREMENT,
    client_ip INT UNSIGNED,
--  ...
    PRIMARY KEY (id)
)

-- 2. 최근(1시간 직전 또는 하루 전) 데이터까지는 Primary Key인 id 값을 범위별로 나눠서 여러 개의 스레드로 복사
INSERT INTO access_log_new
SELECT *
FROM access_log_new
WHERE id BETWEEN 1 AND 1000000;

INSERT INTO access_log_new
SELECT *
FROM access_log_new
WHERE id BETWEEN 1000001 AND 2000000;

INSERT INTO access_log_new
SELECT *
FROM access_log_new
WHERE id BETWEEN 2000001 AND 3000000;
-- ...

-- 3. 트랜잭션을 auto commit으로 설정
SET autocommit = 0;

-- 4. 작업 대상 테이블 2개에 대해 테이블 쓰기 락 설정
LOCK TABLES access_log WRITE, access_log_new WRITE;

-- 5. 기존 테이블의 데이터를 새로운 테이블로 복사
SELECT MAX(id) as max_id
FROM access_log;
INSERT INTO access_log_new
SELECT *
FROM access_log
WHERE id > max_id;
COMMIT;

-- 6. 모든 데이터 복사 완료 후 테이블 이름 변경
RENAME TABLE access_log TO access_log_old, access_log_new TO access_log;

-- 7. 테이블 쓰기 락 해제 및 테이블 삭제
UNLOCK TABLES;
DROP TABLE access_log_old;
```

###### 참고자료

- [Real MySQL 8.0 1 - 개발자와 DBA를 위한 MySQL 실전 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Real+MySQL&isbn=9791158392703&cipId=228440237%2C)
