---
layout: editorial
---

# JDBC(Java Database Connectivity)

자바에서 데이터베이스에 접속할 수 있도록 하는 자바 API로, Java로 데이터베이스에 연결 및 쿼리를 실행하기 위한 인터페이스를 제공한다.

- `java.sql.Connection`: DB 연결 정보 및 연결 관리
- `java.sql.Statement`: SQL 쿼리 실행 및 결과를 받아오는 역할
- `java.sql.ResultSet`: SQL 요청 결과
- JDBC 드라이버: DB 벤더에서 제공하는 라이브러리

위의 세 개의 인터페이스와 JDBC 드라이버를 사용하여 DB에 접근할 수 있다.

## JDBC Flow

1. Get Connection
    - 필요한 연결 정보를 가지고 데이터베이스 연결
    - `DriverManager` 클래스 사용
2. Create Statement
    - `Connection` 으로부터 Statement 객체 생성
3. Configure Statement
    - SQL 쿼리를 실행하기 위해 `Statement` 객체에 Query 설정
4. Execute Statement
    - 구성된 Query 실행
        - `executeQuery()`: SELECT
        - `executeUpdate()`: INSERT, UPDATE, DELETE
5. Handle Warning
    - SQL 쿼리 실행 중 발생한 경고 혹은 예외 처리
6. Return Result
    - SQL 쿼리 실행 결과를 `ResultSet` 객체로 반환
7. Close Statement
    - 사용 완료 후 `Statement.close()` 메서드를 통해 Statement 객체 닫기
8. Close Connection
    - 작업 완료 후 `Connection.close()` 메서드를 통해 Connection 객체 닫기

## JDBC의 사용

JDBC는 여러 문제를 해결해주었지만 오래된 기술이고 사용하는 방법이 복잡하기 때문에, 최근에는 직접 사용하는 것보다는 `SQL Mapper`나 `ORM`을 결합하여 사용하고 있다.

![JDBC에 접근하는 여러 방법](image/jdbc_access.png)

- SQL Mapper
    - 장점
        - SQL 응답 결과를 객체로 편리하게 변환
        - JDBC의 반복 코드를 제거
    - 단점
        - 개발자가 SQL을 직접 작성
    - 대표 기술: 스프링 JdbcTemplate, MyBatis

- ORM
    - 장점
        - 객체와 RDB의 패러다임 불일치 해결
        - 생산성
        - 유지보수
    - 단점
        - 학습비용
        - 잘못 사용하면 성능 이슈
    - JPA는 자바 진영의 ORM 표준 인터페이스
    - 대표 기술: JPA, 하이버네이트, 이클립스링크

###### 참고자료

- [스프링 DB 1편 - 데이터 접근 핵심 원리](https://www.inflearn.com/course/스프링-db-1)