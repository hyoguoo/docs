# B-Tree 인덱스

> B-Tree 인덱스는 데이터베이스의 인덱스를 구성하는 가장 기본적인 방법, Binary의 B가 아니라 Balanced의 B

## 구조 및 특성

최상위에 하나의 루트 노드가 존재하고, 그 하위에 자식 노드가 붙어 있는 형태로 아래의 노드로 구성된다.

- Root Node: 최상위 노드
- Branch Node: 중간에 있는 노드
- Leaf Node: 가장 하위에 있는 노드로, 실제 데이터 레코드를 찾아가기 위한 주솟값을 가짐

## 인덱스 키의 추가/삭제/변경/검색

- 인덱스 키 추가
    - B-Tree에 저장될 때 저장될 키 값을 이용해 적절한 위치를 탐색 후 리프 노드에 저장
    - 만약 리프 노드가 꽉 차있다면 리프 노드를 분리하고 새로운 리프 노드를 생성하게 되는데, 이 때 상위 브랜치 노드까지 처리 범위가 넓어짐
    - 비교적 많은 비용이 발생할 수 있으며, 비용의 대부분이 메모리와 CPU에서 발생하는 것이 아닌, 디스크에서 읽고/쓰기 작업에 많은 비용(시간)이 발생
    - InnoDB의 경우 인덱스 키 추가 잡업을 지연시켜 나중에 처리하는 방식을 사용(Primary Key, Unique Index 경우 중복체크가 필요하여 즉시 처리)
- 인덱스 키 삭제
    - 해당 키 값이 저장된 B-Tree 리프 노드를 찾아 삭제 마크만 하면 작업 완료
- 인덱스 키 변경
    - B-Tree의 키 값이 변경되는 경우 리프 노드 위치의 변경 필요하며, 이 때 키 값을 삭제한 후 새로운 키 값을 추가하는 방식으로 처리
    - 위의 삭제 및 추가 과정이 절차적으로 진행
- 인덱스 키 검색
    - 위의 비용들을 감수하고 인덱스를 사용하는 이유 중 하나(빠른 탐색)
    - 루트 노드로부터 시작해 브랜치 노드를 거쳐 최종 리프 노드까지 내려가며 트리 탐색
    - SELECT(조회)에서만 사용되는 것이 아닌 UPDATE/DELETE에서도 인덱스 키 검색을 사용

## 인덱스 사용에 영향을 미치는 요소

인덱스를 구성하는 컬럼의 크기 / 레코드 건수 / 유니크 인덱스 키 값의 개수 등에 따라 변경/조회 성능이 달라질 수 있다.

### 1. 인덱스 키 값의 크기

디스크에 데이터를 저장하는 가장 기본 단위를 페이지 또는 블록이라고 하며, 작업의 최소 단위로 사용된다.  
인덱스도 결국은 페이지 단위로 관리되며, 루트/브랜치/리프 노트를 구분한 기준이 페이지 단위이다.  
B-Tree의 각 노드는 자식 노드를 가지는 갯수는 인덱스 페이지의 크기 + 키 값의 크기에 따라 결정된다.

- 인덱스 페이지 크기: 16KB
- 인덱스 키: 16 Byte
- 자식노드 주소: 12 Byte
- 하나의 인덱스 페이지에 저장할 수 있는 최대 키 값의 개수: 16 * 1024 / (16 + 12) = 585

인덱스의 키가 커지게 되면, 인덱스 페이지에 저장할 수 있는 키 값의 개수가 줄어들게 되고, 이는 인덱스 키 값의 추가/삭제/변경/검색에 대한 비용이 증가하게 된다.

### 2. B-Tree 깊이

B-Tree 인덱스의 깊이는 성능이 중요한 요소 중 하나지만, 직접 제어할 수 있는 요소가 아니다.  
위에서의 인덱스 키 값의 평균 크기가 늘어나면 인덱스의 깊이가 늘어나게 되고, 이는 인덱스 키 값의 추가/삭제/변경/검색에 대한 비용이 증가하게 된다.  
깊이 = 디스크 I/O 횟수로 직결되므로 깊이가 깊어질수록 성능이 저하로 이어지기 때문에, 인덱스 키 값의 평균 크기를 줄이는 것이 성능 향상에 도움이 된다.  
실제로 대용량 데이터베이스에서는 깊이가 5 이상이 되는 경우는 거의 없다.

### 3. 선택도(기수성)

선택도(Selctivity) / 기수성(Cardinality)는 거의 같은 의미로 사용되며, 모든 인덱스 키 값 가운데 유니크한 값의 수를 의미한다.

- 전체 인덱스 키 값: 100개
- 유니크한 값: 10개
- 기수성: 10

중복된 값이 많아지면 기수성이 낮아지게 되고, 이는 인덱스 키 값의 추가/삭제/변경/검색에 대한 비용이 증가하게 된다.

- 예시

```sql
-- 전체 레코드 수: 10000개
-- country + city 중복해서 저장돼 있지 않음
CREATE TABLE tb_test
(
    country VARCHAR(10),
    city    VARCHAR(10),
    INDEX   idx_country (country),
);

-- Case A: Country 컬럼의 유니크한 값의 개수: 10개
-- -- 10개의 국가(country)의 도시(city) 레코드가 존재
-- Case B: Country 컬럼의 유니크한 값의 개수: 1000개
-- -- 1000개의 국가(country)의 도시(city) 레코드가 존재
SELECT *
FROM tb_test
WHERE country = 'Korea'
  AND city = 'Seoul';
```

- 인덱스된 컬럼(country)에 대해서는 전체 레코드의 건수나 유니크한 값의 개수 등에 대한 통계 정보를 가짐
- 전체 레코드 건수를 뉴니크 값 개수로 나누게되면 하나의 키 값으로 검색했을 때 평균적으로 읽어야 하는 레코드 건수를 알 수 있음
- Case A: 평균 1000건 조회
    - 10000 / 10 = 1000건이 일치하는 것으로 예상하게 됨
    - 레코드를 조회하기 위해 평균 999건의 레코드의 불필요한 데이터를 읽음
- Case B: 평균 10건 조회
    - 10000 / 1000 = 10건이 일치하는 것으로 예상하게 됨
    - 레코드를 조회하기 위해 평균 9건의 레코드의 불필요한 데이터를 읽음

### 4. 읽어야 하는 레코드의 건수

- 인덱스를 통해 테이블의 레코드를 읽는 것은 바로 레코드를 읽는 것보다 높은 비용이 발생
- 일반적인 DBMS 옵티마이저에서 인덱스를 통해 레코드 1건을 읽는 것이 테이블에서 직접 레코드 1건을 읽는 것보다 4-5배 정도의 비용 발생
- 때문에 인덱스를 통해 읽어야 할 레코드 건 수가 전체 레코드의 20-25%를 넘어가면 인덱스를 사용하지 않는 것이 더 효율적

###### 출처

- [Real MySQL 8.0 1](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=284710853)