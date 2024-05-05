---
layout: editorial
---

# Basic

## 수평적 규모 확장(Scale Out)

규모 확장은 크게 수직적 규모 확장과 수평적 규모 확장으로 나뉜다.

- 수직적 규모 확장: 단일 서버의 성능을 향상시키는 방법(CPU / RAM / SSD 등 업그레이드
- 수평적 규모 확장: 여러 서버로 분산하여 처리하는 방법

만약 서버로 유입되는 트래픽 양이 적거나 서버의 성능이 충분하다면 수직적 규모 확장이 유리하지만 다음과 같은 문제가 존재한다.

- 하나의 서버로 트래픽을 처리할 수 있는 절대적인 성능적 한계 존재
- SPOF(Single Point of Failure)로 인한 위험성이 존재
- 자동복구 방안이나 다중화 방안이 존재하지 않아 장애 발생 시 복구가 어려움

위와 같은 문제로 인해 보통 대규모 애플리케이션을 설계할 때는 수평적 규모 확장을 고려한다.

## 로드밸런서

로드밸런서는 부하 분산 집합에 속한 웹 서버에 트래픽 부하를 고르게 분산하는 역할 뿐만 아니라 다음과 같은 기능을 제공할 수 있다.

- 자동복구: 로드밸런서는 서버에 장애가 발생하면 해당 서버로의 트래픽을 차단하고, 다른 서버로 트래픽을 분산한다.
    - Amazon Elastic Load Balancer(ELB)는 등록된 대상을 정지적으로 헬스 체크하여 문제가 있는 인스턴스를 트래픽에서 자동으로 제외한다.
- 확장성: 트래픽에 따라 서버를 추가하거나 제거할 수 있다.

## 데이터베이스 다중화

많은 데이터베이스가 다중화를 지원하며, 보통은 마스터-슬레이브 구조로 구성된다.

- master: 쓰기 작업을 처리하는 서버
- slave: 읽기 작업을 처리하는 서버, 보통 읽기 작업의 횟수가 많기 때문에 많은 slave 서버를 두어 부하를 분산한다.

데이터베이스를 다중화하면 다음과 같은 장점을 얻을 수 있다.

- 성능 향상: 읽기 작업을 분산하여 처리하기 때문에 성능 향상
- 안정성: 데이터베이스 일부가 손상되더라도 데이터 보존 가능
- 가용성: 하나의 데이터베이스 서버에 장애가 발생하더라도 다른 서버에 있는 데이터 사용 가능

## 데이터베이스 수평적 확장(=Sharding)

대규모 데이터베이스를 샤드라고 부르는 작은 단위로 분할하는 기술로, 각 샤드는 같은 스키마를 가지고 있지만 서로 다른 데이터를 저장하게 된다.  
데이터가 보관되는 샤드는 샤딩 전략에 따라 결정되며, 샤딩을 도입하게 되면 다음과 같은 문제가 발생할 수 있다.

- 재 샤딩: 하나의 샤드에 저장된 데이터가 너무 많아지거나 분포가 균등하지 않을 경우 샤딩 전략을 재설정 필요
- 유명인사 문제: 특정 샤드에 질의가 집중되어 해당 샤드에 부하가 집중되는 문제
- 조인과 비정규화: 여러 샤드 서버로 쪼개게 되면, 여러 샤드에 걸친 데이터 조인이 힘들어져 비정규화가 필요할 수 있음

## 캐시(Cache)

값 비싼 연산 결과 또는 자주 참조되는 데이터를 메모리에 저장하여 빠르게 접근할 수 있도록 하는 저장소이다.  
애플리케이션의 성능은 데이터베이스 쿼리의 성능에 크게 의존하는데, 캐시를 사용하면 데이터베이스 쿼리를 줄여 성능을 향상시킬 수 있다.  
대표적인 캐시 전략으론 읽기 주도형 캐시 전략(read-through caching strategy)이 존재하며, 다음과 같이 동작한다.

1. 캐시 계층에 데이터 조회 요청
2. 캐시 계층에 데이터가 존재하면 캐시에서 데이터 조회
    1. 캐시 계층에 데이터가 존재하지 않으면 데이터베이스에서 데이터 조회
    2. 데이터베이스에서 조회한 데이터를 캐시 계층에 데이터 저장
3. 데이터 반환

### 캐시 사용 시 고려사항

캐시 적용 시엔 갱신 빈도 / 참조 빈도 / 만료 시간을 고려해야하며, 그 외에도 다음과 같은 사항을 고려할 수 있다.

- 캐시 메모리: 크기가 너무 작으면 데이터가 캐시에서 자주 밀려나 성능 저하 발생 가능(과할당하여 방지 필요)
- 데이터 방출 정책: 캐시가 꽉 찼을 때 데이터가 캐시에서 밀려나는 방법을 결정하는 정책으로, LRU / LFU / FIFO 등이 있다.

## 콘텐츠 전송 네트워크(CDN)

정적 콘텐츠를 전솔하는 데 사용되는 서버 네트워크로, 물리적으로 분산된 서버의 네트워크이다.  
이미지 / 비디오 / CSS ? JavaScript 파일 등을 캐시하여 사용자에게 빠르게 전송할 수 있다.

사용자가 특정 이미지 URL에 접근하는 경우 아래와 같이 동작한다.

1. 사용자가 이미지 URL에 접근
2. CDN 서버에 요청
3. CDN 서버에 해당 이미지가 존재하면 CDN 서버에서 이미지 조회
    1. CDN 서버에 해당 이미지가 존재하지 않으면 원본 서버에 요청
    2. 원본 서버에서 조회한 이미지를 CDN 서버에 저장
4. CDN 서버에서 이미지 반환

## 무상태(Stateless) 웹 계층

웹 계층의 수평적 확장을 위해선 상태 정보를 웹 계층에서 제거해야 한다.  
상태가 필요한 경우엔 RDBMS / 캐시 시스템 / NoSQL 데이터베이스 등을 사용할 수 있다.

만약 상태 정보를 웹 계층에서 저장하게 되면, 특정 클라이언트가 항상 같은 서버로 전송해야 한다.  
로드밸런서의 고정 세션 방법을 사용할 수 있지만 로드밸런서의 부하가 증가하고 서버 확장성이 저하될 수 있다.

## 다중 데이터 센터

여러 데이터 센터를 지원하게 되면, 가용성을 높이고 전 세계 사용자에게 빠른 응답 시간을 제공할 수 있다.

- 지리적 라우팅(geo-routing)을 사용하여 사용자가 가장 가까운 데이터 센터로 연결
- 특정 데이터 센터에 장애 발생 시 장애가 없는 데이터 센터로 트래픽을 전환

이러한 다중 데이터센터 아키텍처를 설계하기 위해선 다음과 같은 사항을 고려해야 한다.

- 트래픽 우회: 올바른 데이터 센터로 트래픽을 보내는 방법 필요
- 데이터 동기화: 데이터 센터 간 데이터 동기화 방법 필요
- 테스트와 배포: 다중 데이터 센터 아키텍처를 테스트하고 배포하는 방법 필요

## 메시지 큐

메시지의 무손실을 보장하는 비동기 통신을 지원하는 컴포넌트로, 메시지의 버퍼 역할을 하면서 비동기적으로 전송한다.  
메시지 큐의 기본적인 동작 방식은 다음과 같다.

1. 생산자 또는 발행자(producer/publisher)라고 불리는 입력 서비스가 메시지를 큐에 발행(publish)
2. 소비사 또는 구독자(consumer/subscriber)라고 불리는 서버가 메시지를 받아 그에 맞는 동작 수행

메시지 큐를 이용하면 서비스 간 / 서버 간 결합을 느슨하게 하여 서비스 간 의존성을 줄일 수 있다.  
결과적으로 규모 확장성을 높이고 서비스의 가용성을 높일 수 있는 아키텍처를 구성하기 좋아진다.

## 로그 / 메트릭 / 자동화

시스템의 규모가 커질수록 로그 / 메트릭 / 자동화의 중요성은 더욱 커진다.

- 로그: 시스템의 상태를 추적 및 문제 해결에 활용(단일 서비스로 모아주는 도구 활용)
- 메트릭: 시스템의 현재 상황 파악 및 사업 전략 수립에 활용
- 자동화: 시스템의 배포 / 관리 / 모니터링을 자동화하여 생산성과 안정성 향상

###### 출처

- [가상 면접 사례로 배우는 대규모 시스템 설계 기초](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=%EA%B0%80%EC%83%81+%EB%A9%B4%EC%A0%91+%EC%82%AC%EB%A1%80%EB%A1%9C+%EB%B0%B0%EC%9A%B0%EB%8A%94+%EB%8C%80%EA%B7%9C%EB%AA%A8&isbn=9788966263240&cipId=228421467%2C)
