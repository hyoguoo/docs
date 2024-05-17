---
layout: editorial
---

# Design News Feed System

## 요구사항

- 모바일 / 웹 모두 지원
- 피드 페이즈에 새로운 스토리 생성 및 조회 기능
- 스토리엔 이미지 혹은 비디오 포함 가능
- 시간 흐름 역순으로 스토리 정렬
- 한 명의 사용자는 최대 5,000명의 팔로워를 가질 수 있음
- DAU 1,000만명

## 기능 정의

위와 같은 요구사항을 만족하기 위해 크게 두 가지 기능을 구현해야 한다.

- 피드 발행: 사용자가 스토리가 포스팅 시 해당 데이터를 캐시와 데이터베이스에 기록
    - 새 포스팅은 팔로워의 뉴스 피드에도 전송
- 뉴스 피드 생성: 모든 친구의 포스팅을 시간 흐름 역순으로 정렬하여 사용자에게 제공

위 기능을 API 단위로 나누어 설계하면 다음과 같다.

- 피드 발행 API `POST /v1/me/feed`
    - body: 포스팅 내용
    - 헤더: 인증을 위한 Authorization 헤더 토큰
- 피드 읽기 API `GET /v1/me/feed`
    - 헤더: 인증을 위한 Authorization 헤더 토큰

## 상세 설계

### 피드 발행

피드 발행 기능을 제공하기 위한 시스템은 대략적으로 아래와 같이 구성할 수 있다.

```
디바이스 ──> 로드밸런서 ──> 웹서버 ──> 포스팅 저장 서비스 ──> 포스팅 캐시 ──> 포스팅 데이터베이스
                            ├── 포스팅 전송 서비스 ──> 사용자 관계 그래프 데이터베이스 
                            └── 알림 서비스      ├─> 사용자 정보 캐시 ──> 사용자 데이터베이스
                                              └─> 메시지 큐 ──> 포스팅 전송 작업 서버 ──> 뉴스 피드 캐시
```

- 웹 서버: HTTP 요청을 내부 서비스로 중계
    - 인증: Authorization 헤더 토큰을 확인하여 사용자 인증
    - 처리율 제한: 유해한 컨텐츠 방지 및 특정 사용자가 올릴 수 있는 포스팅 수 제한
- 포스팅 저장 서비스: 새 포스팅을 데이터베이스와 캐시에 저장
- 포스팅 전송 서비스: 새 포스팅을 해당 사용자의 연관 관계에 있는 모든 사용자에게 전달
    - 뉴스 피드 캐시에 저장해 빠르게 조회 할 수 있도록 함
- 알림 서비스: 팔로워에게 새 포스팅 알림 역할

포스팅 전송 서비스에서 새 포스팅을 해당 사용자의 연관 관계에 있는 모든 사용자에게 전달하는 것을 팬아웃이라고 하는데,  
팬아웃 모델에는 쓰기 시점 팬아웃과 읽기 시점 팬아웃 두 가지 존재한다.

- 쓰기 시점 팬아웃(푸시 모델): 새 포스팅을 저장하는 시점에 뉴스 피드 갱신
    - 뉴스 피드가 실시간으로 갱신되어 사용자가 뉴스 피드 조회 시점에 빠르게 조회 가능
    - 사용자 모두의 뉴스 피드를 갱신하기 때문에 많은 시간 소요 문제 가능성이 존재하며 자주 이용하지 않는 사용자까지 불필요하게 갱신되는 문제가 있음
- 읽기 시점 팬아웃(풀 모델): 뉴스 피드를 조회하는 시점에 뉴스 피드 갱신
    - 자주 이용하지 않는 사용자의 뉴스 피드 갱신을 방지
    - 사용자가 뉴스 피드 조회 시점에 갱신되기 때문에 뉴스 피드 조회 시간이 길어질 수 있음

위 두 가지 방법을 혼합하여 각 장점을 취하도록 설계할 수 있다.

- 대부분의 사용자: 뉴스 피드를 빠르게 조회할 수 있는 것이 중요하므로 쓰기 시점 팬아웃(푸시 모델) 사용
- 팔로워가 아주 많은 사용자: 팔로워로 하여금 해당 사용자의 포스팅을 필요할 때 조회하는 방식으로 읽기 시점 팬아웃(풀 모델) 사용
- 추가적으로 안정 해시를 통해 요청과 데이터를 고르게 분산하여 처리(특정 데이터가 많이 요청되어 부하가 집중되는 것을 방지)

두 방법을 혼합하게 된 팬아웃 서비스는 다음과 같이 동작하게 된다.

1. 사용자 관계 그래프 데이터베이스에서 친구 ID 목록 조회
2. 사용자 정보 캐시에서 친구들 정보 조회
3. 친구 목록과 새 스토리 포스팅 ID를 메시지 큐에 전달
4. 포스팅 전송 작업 서버에서 메시지 큐에서 메시지를 가져와 뉴스 피드 캐시에 저장
    - 뉴스 피드 캐시는 <post_id, user_id> 순서쌍 저장
    - 사용자가 피드 전부를 보는 경우는 드물기 때문에 캐시의 크기를 제한하여 최근 포스팅만 저장
    - 메모리를 절약하기 위해 id만 저장하고, 실제 데이터는 데이터베이스에서 가져옴

### 뉴스 피드 생성 및 조회

사용자가 보는 뉴스 피드를 생성하기 위한 시스템은 대략적으로 아래와 같이 구성할 수 있다.

```
디바이스 <──> 로드밸런서 ──> 웹서버(여러 대) ──> 뉴스 피드 서비스 ──> 뉴스 피드 캐시
        └─> CDN                                       ├─> 사용자 정보 캐시 ──> 사용자 데이터베이스
                                                      └─> 포스팅 캐시 ──> 포스팅 데이터베이스
```


1. 사용자가 뉴스 피드 조회 요청
2. 로드밸런서는 웹 서버 중 하나를 선택하여 요청 전달
3. 뉴스 피드 서비스에서 뉴스 피드 캐시에 저장된 포스팅 ID 목록 조회
4. 뉴스 피드에 표시할 사용자 이름 / 사용자 사진 / 포스팅 컨텐츠 / 이미지 등을 사용자 캐시와 포스팅 캐시에서 조회
5. 조회된 데이터들로 뉴스 피드 생성
6. 생성된 뉴스 피드를 사용자에게 전달

### 캐시

캐시는 뉴스 피드 시스템의 핵심 컴포넌트 중 하나로, 다음과 같은 정보를 저장할 수 있다.

- 뉴스 피드: 뉴스 피드 ID 보관
- 컨텐츠: 포스팅 데이터 보관, 인기 컨텐츠와 일반 컨텐츠를 구분하여 저장
- 소셜 그래프: 팔로워 / 팔로잉 관계 보관
- 행동: 사용자 행동(좋아요, 댓글 등) 보관
- 횟수: 좋아요 / 댓글 수 / 팔로워 수 등 보관
    
###### 참고자료

- [가상 면접 사례로 배우는 대규모 시스템 설계 기초](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=%EA%B0%80%EC%83%81+%EB%A9%B4%EC%A0%91+%EC%82%AC%EB%A1%80%EB%A1%9C+%EB%B0%B0%EC%9A%B0%EB%8A%94+%EB%8C%80%EA%B7%9C%EB%AA%A8&isbn=9788966263240&cipId=228421467%2C)