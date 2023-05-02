# 통계 정보

MySQL 5.7 버전까지는 테이블과 인덱스에 대한 개괄적인 정보만 가졌고, 테이블 컬럼 값들이 실제 어떻게 분포돼 있는지에 대한 정보 없이 실행 계획을 수립하여 정확도가 떨어졌다.  
이를 개선하기 위해 MySQL 8.0 버전부터는 인덱스되지 않은 컬럼들에 대해서도 데이터 분포도를 수집해서 저장하는 히스토그램(Histogram)을 도입했다.

## 테이블 및 인덱스 통계 정보

비용 기반 최적화에서 가장 중요하게 여기는 것은 통계 정보로, 통계 정보가 없으면 옵티마이저는 제대로 된 실행 계획을 수립할 수 없다.(10억 건인데 풀 테이블 스캔하는 등)  
MySQL 5.6 버전부터 통계 정보의 정확성을 높일 수 있는 방법을 제공하기 시작했다.  
테이블에 대한 통계 정보를 `innodb_index_stats`, `innodb_table_stats` 테이블에서 관리하게 된다.  
테이블을 생성할 때 `STATS_PERSISTENT` 옵션을 설정할 수 있는데, 이 설정값에 따라 테이블 단위로 영구적인 통계 정보를 보관할지 여부를 결정할 수 있다.

```mysql
CREATE TABLE tb_test
(
    fd1 INT,
    fd2 VARCHAR(20),
    PRIMARY KEY (fd1)
)
    ENGINE = InnoDB
    STATS_PERSISTENT = DEFAULT;
# ALTER TABLE로 변경도 가능
```

- `STATS_PERSISTENT = DEFAULT` : 데이터베이스의 `innodb_stats_persistent` 시스템 변수의 값에 따라 통계 정보 보관, 기본적으로 `1`으로 설정되어 있음
- `STATS_PERSISTENT = 1` : 테이블 단위로 영구적인 통계 정보 보관
- `STATS_PERSISTENT = 0` : 데이터베이스의 테이블에 저장하지 않음(MySQL 5.5 이전의 방식대로 관리하는 방법)

## 히스토그램

MySQL 8.0 이전 버전에서는 단순히 인덱스된 컬럼의 유니크한 값의 갯수만 가지고 있어 최적의 실행 계획을 수립하기엔 부족했다.  
8.0 버전부터는 컬럼의 데이터 분포도를 참조할 수 있는 히스토그램(Histogram)이 도입됐다.

### 히스토그램 정보 수집 및 삭제

히스토그램 정보는 컬럼 단위로 관리되는데, 자동이 아닌 `ANALYZE TABLE ... UPDATE HISTOGRAM` 명령으로 수동으로 수집 및 관리된다.

```mysql
ANALYZE TABLE test.TB_WEATHER UPDATE HISTOGRAM ON temperatureMaximum, temperatureMinimum;

SELECT *
FROM COLUMN_STATISTICS;
```

| SCHEMA_NAME | TABLE_NAME | COLUMN_NAME        | HISTOGRAM                                                                                                                                                                                                                                                                         |
|:------------|:-----------|:-------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| test        | TB_WEATHER | temperatureMaximum | {"buckets": \[\[5, 0.0013850415512465374\], \[7, 0.002770083102493075\], ..., "data-type": "int", "null-values": 0.0, "collation-id": 8, "last-updated": "2023-05-02 14:36:31.698758", "sampling-rate": 1.0, "histogram-type": "singleton", "number-of-buckets-specified": 100}   |
| test        | TB_WEATHER | temperatureMinimum | {"buckets": \[\[-3, 0.0013850415512465374\], \[-2, 0.002770083102493075\], ..., "data-type": "int", "null-values": 0.0, "collation-id": 8, "last-updated": "2023-05-02 14:36:31.698865", "sampling-rate": 1.0, "histogram-type": "singleton", "number-of-buckets-specified": 100} |

히스토그램은 버킷 단위로 구분되어 레코드 건수나 컬럼 값의 범위가 관리되며, 히스토그램 타입(histogram-type)은 다음과 같이 두 가지가 지원된다.

- Singleton(싱글톤 히스토그램)
    - 컬럼값 개별로 레코드 건수를 관리하는 히스토그램
    - 컬럼이 가지는 값별로 버킷이 할당
    - 각 버킷이 컬럼의 값과 발생 빈도의 비율 값을 가짐
    - Value-Based 히스토그램 또는 도수 분포라고도 불림
- Equi-Height(높이 균형 히스토그램)
    - 컬럼값의 범위를 균등한 개수로 구분해서 관리하는 히스토그램
    - 개수가 균등한 컬럼값의 범위 별로 하나의 버킷이 할당
    - 각 버킷이 범위 시작 값과 마지막 값, 발생 빈도율과 각 버킷에 포함된 유니크한 값의 갯수
    - Height-Balance 히스토그램라고도 불림

그 외에 필드에 대한 설명은 아래와 같다.

- sampling-rate: 히스토그램 정보를 수집하기 위해 스캔한 페이지의 비율(0.35 = 35%)
- number-of-buckets-specified: 히스토그램을 생성할 때 설정했던 버킷의 개수, 기본 100개의 버킷(최대 1024개 설정 가능하지만 일반적으로 100개면 충분)

### 용도와 효과

실제 데이터는 항상 균등한 분포도를 가지지 않기 때문에 히스토그램 정보 없이 인덱스에 대한 통계 정보만으론 최적의 실행 계획을 수립하기 어렵다.  
이러한 부분을 보완하기 위해 히스토그램 정보를 이용해 인덱스의 통계 정보를 보완한다.

```mysql
EXPLAIN
SELECT *
FROM employees
WHERE first_name = 'Zita'
  AND birth_date BETWEEN '1950-01-01' AND '1960-01-01';
```

| id | select_type | table     | type | key          | rows | filtered |
|:---|:------------|:----------|:-----|:-------------|:-----|:---------|
| 1  | SIMPLE      | employees | ref  | ix_firstname | 224  | 11.11    |

위 결과에서 옵티마이저에서는 `ix_firstname` 인덱스를 사용해 `Zita` 조건에 일치하는 224건을 찾고, 그중에서 11.11%를 birth_date 조건에 해당할 것으로 예측한 것을 알 수 있다.

```mysql
ANALYZE TABLE employees UPDATE HISTOGRAM ON first_name, birth_date;

EXPLAIN
SELECT *
FROM employees
WHERE first_name = 'Zita'
  AND birth_date BETWEEN '1950-01-01' AND '1960-01-01';
```

| id | select_type | table     | type | key          | rows | filtered |
|:---|:------------|:----------|:-----|:-------------|:-----|:---------|
| 1  | SIMPLE      | employees | ref  | ix_firstname | 224  | 60.82    |

히스토그램 정보를 이용해 실제 데이터 분포도를 반영한 통계 정보를 이용해 실행 계획을 더 정확하게 수립할 수 있다.(실제 63.84%)  
이는 JOIN을 할 떄에도 건 수가 적은 테이블을 기준으로 실행 계획을 수립할 수 있게 해주기 때문에 성능 상으로 아주 큰 이점을 가져다 줄 수 있다.

## 코스트 모델(Cost Model)

MySQL 서버가 쿼리를 처리할 때 아래와 같은 작업을 수행하게 된다.

- 디스크로부터 데이터 페이지 읽기
- 메모리(InnoDB 버퍼 풀)로부터 데이터 페이지 읽기
- 인덱스 키 비교
- 레코드 평가
- 메모리 임시 테이블 작업
- 디스크 임시 테이블 작업

위 작업에서 얼마나 필요한지 예측하고 전체 작업 비용을 계산한 결과를 바탕으로 최적의 실행 계획을 찾는데, 필요 단위 작업들의 비용을 코스트 모델이라고 한다.  
이전에는 작업들의 비용을 서버 소스 코드에 상수로 존재하였으나 하드웨어마다 성능이 다르기 때문에 5.7 버전부터는 DBMS가 실행 환경에 맞게 조정할 수 있게 됐다.  
코스트 모델을 조정하는 것은 상당히 어려운 일이기 때문에 기본 값으로도 충분히 좋은 성능을 낼 수 있도록 설계되어 있다.

###### 참고자료

- [Real MySQL 8.0 1 - 개발자와 DBA를 위한 MySQL 실전 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Real+MySQL&isbn=9791158392703&cipId=228440237%2C)