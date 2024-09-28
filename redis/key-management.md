---
layout: editorial
---

# Key Management

Redis에서 키는 데이터를 저장하고 관리하는 중요한 요소로, 키는 기본적으로 문자열로 표현되며, 다양한 명령어와 설정을 통해 관리할 수 있다

### 키의 자동 생성과 삭제

Redis는 키가 존재하지 않는 상태에서 값을 설정하는 명령어(`SET`, `HSET`, `SADD` 등)를 사용하면 자동으로 키가 생성 된다.

- 키가 존재하지 않는 경우: 새로운 키가 생성되고 값이 설정
- 키가 이미 존재하는 경우: 기존 키의 값을 업데이트
- 다른 자료구조로 이미 생성 된 경우: 에러 발생

```bash
SET mykey "value"  # mykey가 없으면 생성
SET mykey "new_value"  # mykey가 있으면 값 업데이트
LPUSH mykey "1" "2"  # 에러 발생
```

리스트의 경우 들어간 아이템이 모두 삭제 되면 해당 키도 자동으로 삭제된다.

```bash
LPUSH mylist "1" "2" "3"
LPOP mylist
LPOP mylist
LPOP mylist  # mylist 키 삭제
````

### 키와 관련된 명령어

|    명령어     |                  기능                   |          예시           |
|:----------:|:-------------------------------------:|:---------------------:|
|   `SET`    |                키와 값 설정                |  `SET mykey "value"`  |
|   `GET`    |                키의 값 조회                |      `GET mykey`      |
|  `EXISTS`  |              키가 존재하는지 확인              |    `EXISTS mykey`     |
|   `KEYS`   |           특정 패턴에 맞는 키 목록 조회           |     `KEYS user:*`     |
|   `SCAN`   |             키 목록을 순회하며 조회             |       `SCAN 0`        |
|   `SORT`   |  list, set, sorted set 데이터를 정렬하여 반환   |     `SORT mylist`     |
|  `RENAME`  |                키 이름 변경                | `RENAME mykey newkey` |
|   `COPY`   |                 키 복사                  |  `COPY mykey newkey`  |
|   `TYPE`   |             키의 데이터 타입 조회              |     `TYPE mykey`      |
|  `OBJECT`  |  키의 메모리 사용량, 데이터 타입, TTL 등의 상세 정보 조회  |    `OBJECT mykey`     |
|   `DEL`    |                 키 삭제                  |      `DEL mykey`      |
| `FLUSHALL` |           레디스에 저장된 모든 키 삭제            |      `FLUSHALL`       |
|  `UNLINK`  | 키를 삭제하지만, 백그라운드에서 삭제 작업을 수행하여 블로킹을 방지 |    `UNLINK mykey`     |
|  `EXPIRE`  |           키의 TTL(만료 시간) 설정            |   `EXPIRE mykey 60`   |
|   `TTL`    |             남은 TTL 시간 조회              |      `TTL mykey`      |
| `PERSIST`  |         TTL을 제거하고 키를 영구적으로 유지         |    `PERSIST mykey`    |

#### `KEYS` vs `SCAN`

두 명령어 모두 특정 패턴에 맞는 키를 조회하는데 사용되지만, 안정성과 성능 면에서 차이가 있다.

- `KEYS`: 모든 키의 정보를 반환하는데, 싱글 스레드인 레디스에선 해당 작업을 수행하는 동안 다른 명령어를 수행 할 수 없어 성능 이슈가 발생할 수 있음
- `SCAN`: 커서를 기반으로 특정 범위의 키만 조회할 수 있어 비교적 안전하게 사용 가능
