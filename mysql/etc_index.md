# 유니크 인덱스

테이블이나 인덱스에 같은 값이 2개 이상 저장될 수 없는 것을 의미하며, 인덱스라기보다 제약 조건에 가깝다고 볼 수 있다.  
MySQL에서는 인덱스 없이 유니크 제약만 설정할 방법이 없다.  
인덱스 쓰기 작업도 수행되어 비용이 발생하는데, 추가적으로 중복 체크를 수행하기 때문에 성능 저하가 발생한다.  
때문에 프라이머리 키와 유니트 인덱스 동일하게 생성하거나, 꼭 유일성이 보장되어야 하는 컬럼이 아니라면 유니크 인덱스를 사용하지 않는 것이 좋다.

# 외래키

외래키 제약이 설정되면 자동으로 연관되는 테이블의 컬럼에 인덱스가 생성된다.  
InnoDB의 외래키 관리에는 아래와 같은 잠금이 발생한다.

```mysql
CREATE TABLE tb_parent
(
    id INT          NOT NULL,
    fd VARCHAR(100) NOT NULL,
    PRIMARY KEY (id)
);

CREATE TABLE tb_child
(
    id  INT NOT NULL,
    pid INT          DEFAULT NULL,
    fd  VARCHAR(100) DEFAULT NULL,
    PRIMARY KEY (id),
    KEY ix_parentid (pid),
    CONSTRAINT child_ibfk_1 FOREIGN KEY (pid) REFERENCES tb_parent (id) ON DELETE CASCADE
);
```

## 자식 테이블의 변경 대기

|                       커넥션 1                       |                   커넥션 2                   |
|:-------------------------------------------------:|:-----------------------------------------:|
|                     `BEGIN;`                      |                                           |
| `UPDATE tb_parent SET fd='changed-2' WHERE id=2;` |                                           |
|                                                   |                 `BEGIN;`                  |
|                                                   | `UPDATE tb_child SET pid=2 WHERE id=100;` |
|                    `ROLLBACK;`                    |                 커넥션 1 대기                  |
|                                                   |                 `COMMIT;`                 |

1. 1번 커넥션에서 id 2인 레코드에 대해 쓰기 잠금을 획득
2. 2번 커넥션에서 자식 테이블의 외래키 컬럼(pid)을 pid 2로 변경하는 쿼리 실행
3. 부모 테이블의 변경 작업이 완료될 때까지 2번 커넥션은 대기

자식 테이블의 외래 키 컬럼은 부모 테이블의 확인이 필요하므로, 부모 테이블의 해당 레코드가 쓰기 잠금이 걸려 있으면 해당 쓰기 잠금이 해제될 때까지 대기하게 된다.  
하지만 자식 테이블의 외래키(pid)가 아닌 컬럼은 부모 테이블의 변경 여부와 상관없이 변경이 가능하다.

## 부모 테이블의 변경 대기

|                                        커넥션 1                                        |                커넥션 2                |
|:-----------------------------------------------------------------------------------:|:-----------------------------------:|
|                                      `BEGIN;`                                       |                                     |
| `UPDATE tb_child SET fd='changed-100' WHERE id=100;`<br/>부모키 1을 참조하는 자식 테이블 레코드에 접근 |                                     |
|                                                                                     |              `BEGIN;`               |
|                                                                                     | `DELETE FROM tb_parent WHERE id=1;` |
|                                     `ROLLBACK;`                                     |              커넥션 1 대기               |
|                                                                                     |              `COMMIT;`              |

1. 1번 커넥션에서 부모키 1을 참조하는 자식 테이블 레코드에 대해 쓰기 잠금 획득
2. 2번 커넥션에서 부모 테이블의 id 1에 대해 삭제하는 쿼리 실행
3. 자식 테이블의 변경 작업이 완료될 때까지 2번 커넥션은 대기

여기서는 자식 테이블이 생성될 때 정의된 외래키 특성(`ON DELETE CASCADE`)으로 부모 테이블의 레코드가 삭제되면 자식 테이블의 레코드도 함께 삭제 되기 때문에  
자식 테이블의 변경 작업이 완료될 때까지 2번 커넥션은 대기하게 된다.

# 그 외

전문 검색 인덱스 / 멀티 밸류 인덱스 / 클러스터링 인덱스 / R-Tree 인덱스 / 함수 기반 인덱스 등이 존재한다.

###### 참고자료

- [Real MySQL 8.0 1 - 개발자와 DBA를 위한 MySQL 실전 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Real+MySQL&isbn=9791158392703&cipId=228440237%2C)