---
layout: editorial
---

# MySQL

버전이나 스토리지 엔진이 명시되지 않은 경우 MySQL 8.0 / InnoDB 스토리지 엔진을 기준으로 합니다.

* [Architecture(아키텍처)](architecture.md)
* [트랜잭션](transaction.md)
* Lock(잠금)
  * [MySQL 엔진 락](mysql_lock.md)
  * [InnoDB 스토리지 엔진 락](innodb_lock.md)
* [Isolation Level](isolation_level.md)
* [Index](index.md)
  * [B-Tree Index](btree_index.md)
  * [ETC Index](etc_index.md)
* [Optimizer](optimizer.md)
* [데이터 처리](data_processing.md)
* 실행 계획(Execution Plan)
  * [통계 정보](statistics.md)
  * [실행 계획 확인](check_execution_plan.md)
  * [실행 계획 분석](analyze_execution_plan.md)
* 쿼리(Query)
  * [쿼리 작성과 연관된 시스템 변수](query_system_variable.md)
  * [리터럴 표기법](literal_notation.md)
  * [연산자](operator.md)
  * [내장 함수](built_in_function.md)
  * [SELECT](select.md)
  * [SELECT LOCK](select_lock.md)
  * [Sub Query](sub_query.md)
  * [INSERT](insert.md)
  * [UPDATE / DELETE](update_delete.md)
  * [성능 테스트](performance_test.md)
* 데이터 타입
  * [문자열(CHAR / VARCHAR)](char_varchar.md)
  * [숫자](number.md)
  * [날짜 / 시간](date_time.md)
  * [enum](enum.md)
* [Replication](replication.md)
