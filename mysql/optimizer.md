---
layout: editorial
---

# Optimizer(옵티마이저)

> 최적의 실행 계획을 수립하는 역할

## 옵티마이저의 종류

데이터베이스 서버에서 쿼리를 처리할 때, 옵티마이저는 크게 두 가지 방식으로 최적화를 수행한다.

- 비용 기반 최적화(Cost-based Optimizer): 대부분의 DBMS가 사용하는 옵티마이저
    - 쿼리 처리를 위한 여러 가지 가능한 방법을 만들고, 각 단위 작업의 비용 정보와 대상 테이블의 예측된 통계 정보를 이용해 실행 계획별 비용 산출
    - 산출된 실행 방법 중 비용이 가장 적은 실행 계획을 선택
- 규칙 기반 최적화(Rule-based Optimizer): 예전 초기 버전의 오라클 DBMS에서 사용
    - 대상 테이블의 레코드 건수나 선택도 등을 고려하지 않고 옵티마이저에 내장된 우선순위에 따라 실행 계획을 수립하는 방식
    - 통계 정보(테이블 레코드 건수나 컬럼값의 분포도)를 고려하지 않고 실행 계획이 수립되어 항상 같은 실행 방법 생성

###### 참고자료

- [Real MySQL 8.0 1 - 개발자와 DBA를 위한 MySQL 실전 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Real+MySQL&isbn=9791158392703&cipId=228440237%2C)
