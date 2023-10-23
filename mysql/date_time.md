---
layout: editorial
---

# Date Time

MySQL은 날짜만 / 시간만 혹은 날짜 + 시간 합쳐서 하나의 컬럼에 저장할 수 있도록 여러 가지 타입을 지원한다.

|  데이터 타입   |    저장 공간(Byte)     |
|:---------:|:------------------:|
|   YEAR    |         1          |
|   DATE    |         3          |
|   TIME    | 3 + (밀리초 단위 저장 공간) |
| DATETIME  | 5 + (밀리초 단위 저장 공간) |
| TIMESTAMP | 4 + (밀리초 단위 저장 공간) |

## 밀리초 단위

밀리초 단위로 데이터를 저장하기 위해서는 날짜 타입 뒤 괄호 안에 숫자를 넣어서 밀리초 단위 저장 공간을 지정할 수 있다.(기본값은 0)  
밀리초 단위 저장 공간은 2자리당 1바이트씩 공간이 더 필요하여, MySQL 8.0에서는 마이크로초까지 저장 가능한 `DATETIME(6)` 타입은 (5+3)를 사용한다.

## 타임존

MySQL의 날짜 타입은 컬럼 자체에 타임존 정보가 저장되지 않아 DATETIME / DATE 타입은 현재 DBMS 커넥션의 타임존과 관계없이 입력된 값을 그대로 저장한다.  
하지만 TIMESTAMP는 항상 UTC 타임존으로 저장되므로 타임존이 달라져도 값이 자동으로 보정된다.

```mysql
CREATE TABLE tb_timezone
(
    fd_datetime  DATETIME,
    fd_timestamp TIMESTAMP
);

SET time_zone = 'Asia/Seoul';

INSERT INTO tb_timezone
VALUES (NOW(), NOW());

SELECT *
FROM tb_timezone;
# +---------------------+---------------------+
# | fd_datetime         | fd_timestamp        |
# +---------------------+---------------------+
# | 2023-05-05 05:09:00 | 2023-05-05 05:09:00 |
# +---------------------+---------------------+

SET time_zone = 'America/Los_Angeles';

SELECT *
FROM tb_timezone;
# +---------------------+---------------------+
# | fd_datetime         | fd_timestamp        |
# +---------------------+---------------------+
# | 2023-05-05 05:09:00 | 2023-05-04 13:09:00 |
# +---------------------+---------------------+
```

## 자동 업데이트 설정

MySQL은 날짜 타입에 대해 INSERT / UPDATE 시 자동으로 값을 업데이트할 수 있도록 설정할 수 있다.

```mysql
CREATE TABLE tb_autoupdate
(
    id                   BIGINT NOT NULL AUTO_INCREMENT,
    created_at_datetime  DATETIME  DEFAULT CURRENT_TIMESTAMP,
    created_at_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at_datetime  DATETIME  DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    updated_at_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id)
);
```

MySQL 5.6 이전 버전에서는 TIMESTAMP 타입에 대해서만 자동 업데이트 설정이 가능했지만, 5.6 이후 버전부터는 DATETIME 타입에 대해서도 자동 업데이트 설정이 가능하다.

###### 참고자료

- [Real MySQL 8.0 2 - 개발자와 DBA를 위한 MySQL 실전 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Real+MySql+8.0&isbn=9791158392727&cipId=228440238%2C)