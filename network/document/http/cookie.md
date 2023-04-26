# 쿠키(Cookie)

> `Client`의 상태를 식별하기 위한 가장 널리 사용 되는 방식

사용자 로그인 세션 관리, 사용자 선호 설정, 광고 정보 트래킹 같은 곳에 사용되며, 항상 `HTTP` 헤더에 포함되어 전송되기 때문에 네트워크의 트래픽이 증가할 수 있다.  
쿠키의 타입은 파기되는 시점에 따라 아래 두 가지로 나뉜다.

- 세션 쿠키(Session Cookie)
    - 사용자가 사이트를 탐색할 때, 관련한 설정과 선호 사항들을 저장하는 임시 쿠키
    - 브라우저를 닫으면 삭제
- 지속 쿠키(Persistent Cookie)
    - 사용자가 주기적으로 방문하는 사이트에 대한 설정 정보나 로그인 이름 등을 저장하는 쿠키
    - 디스크에 저장되어, 브라우저를 닫거나 컴퓨터를 재시작하더라도 유지되는 쿠키

## 쿠키의 생명주기

- `Session Cookie`: 브라우저가 종료되면 쿠키가 삭제됨
- `Persistent Cookie`: 브라우저가 종료되어도 쿠키가 삭제되지 않음
- `Expires`나 `Max-Age`를 지정하지 않으면 세션 쿠키로 생성됨
    - `Expires` : 쿠키의 만료 시간을 지정
    - `Max-Age` : 쿠키의 만료 시간을 초 단위로 지정

## 도메인

모든 사이트에 브라우저 내의 모든 쿠키가 전송된다면 개인정보 문제와 불필요한 데이터를 전송하는 문제가 발생할 수 있기 때문에  
해당 쿠키가 적용될 도메인을 지정하는 속성(`domain=example.com`)을 지정할 수 있다.

- 명시하는 경우
    - `Domain` 속성에 도메인을 지정
    - `Domain` 속성에 지정한 도메인과 하위 도메인에 쿠키가 적용됨(`dev.example.com`에도 적용됨)
    - 해당 쿠키가 적용될 하위 경로를 지정하는 속성으로, 일반적으로 루트로 지정함(`path=/`)
- 명시하지 않는 경우
    - `Domain` 속성에 현재 도메인이 지정됨
    - `Domain` 속성에 지정한 도메인만 쿠키가 적용됨(`dev.example.com`에는 적용되지 않음)

## 보안

쿠키는 브라우저에 저장되기 때문에, 해당 브라우저가 다른 사람에 의해 접근될 경우 쿠키도 함께 유출될 수 있으며, 쿠키는 누구나 쉽게 볼 수 있는 평문 형태로 저장되기 때문에, 쿠키 값이 변조될 위험성이 존재한다.  
이러한 문제를 방지하기 위해 쿠키에 보안을 강화하는 속성을 지정할 수 있다.

- `Secure`
    - 쿠키는 원래 `HTTP`, `HTTPS`를 구분하지 않고 전송
    - `Secure` 속성을 지정하면 `HTTPS`에서만 쿠키가 전송됨
- `HttpOnly`
    - 자바스크립트 코드에서 쿠키에 접근이 가능한데 이를 막기 위해 `HttpOnly` 속성을 지정(XSS 공격 방지)
    - HTTP 전송에만 쿠키를 사용할 수 있게 됨
- `SameSite`
    - `SameSite` 속성을 지정하면 `SameSite` 속성에 지정한 도메인과 같은 도메인에서만 쿠키가 전송됨
    - `SameSite=Strict` : 동일한 도메인에서만 쿠키가 전송됨

###### 참고자료

- [HTTP 완벽 가이드](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=HTTP+완벽+가이드&isbn=9788966261208&cipId=200309770%2C4096969)
- [(그림으로 배우는) http & network basic](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=9788931447897&isbn=9788931447897&cipId=200443691%2C)