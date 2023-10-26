---
layout: editorial
---

# CORS(Cross-Origin Resource Sharing)

웹 애플리케이션에서 same-origin policy(동일 출처 정책)에 따라 제한되는 도메인 간 자원 공유를 허용하기 위한 매커니즘

## Same-Origin Policy(동일 출처 정책)

브라우저에서 보안 상의 이유로, 스크립트를 실행하는 도메인과 리소스를 요청하는 도메인이 동일해야하는 규칙으로,  
위배되는 상황에는 브라우저가 리소스 요청을 차단하게 된다.

## Cross-Origin

cross-origin의 종류는 아래로 정의된다.

- 다른 도메인 (example.com - test.com)
- 다른 하위 도메인 (example.com - store.example.com)
- 다른 포트 (example.com:80 - example.com:90)
- 다른 프로토콜 (https://example.com - http://example.com)

여기서 `https://example.com`과 `https://www.example.com`은 다른 도메인으로 간주되기 때문에 주의해야한다.

### 예시(기준 `http://store.ogufamily.com`)

|                   URL                   |    Result    |   Reason    |
|:---------------------------------------:|:------------:|:-----------:|
| http://store.ogufamily.com/babyogu.html | same origin  | path만 다른경우  |
|   http://store.ogufamily.com/ogu.html   | same origin  | path만 다른경우  |
|       https://store.ogufamily.com       | cross origin | 프로토콜이 다른 경우 |
|      http://store.ogufamily.com:81      | cross origin |  포트가 다른경우   |
|        http://news.ogufamily.com        | cross origin |  도메인이 다른경우  |

## CORS 작동 방식

1. 클라이언트가 웹 페이지에서 JavaScript를 통해 다른 도메인의 리소스(`cross-origin`) 요청
2. 브라우저는 요청을 보낼 때 HTTP 헤더에 Origin 값을 넣어 전송
3. 서버에서 Origin 값을 확인
4. 서버에서 응답의 `Access-Control-Allow-Origin` 헤더를 통해 요청을 허용할 Origin을 전송
5. 브라우저에서 응답의 `Access-Control-Allow-Origin` 헤더를 확인하여 자신의 도메인과 일치하는지 확인

위의 과정에서 일치한다면 성공적으로 리소스를 받아올 수 있고, 일치하지 않는다면 브라우저는 에러가 발생하여 아래의 에러를 확인할 수 있다.

```
*** has sbeen blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

언어 및 프레임워크 별로 설정 방법은 다르지만 서버 쪽에서 Origin 설정을 해주어 간단히 해결할 수 있다.

## Preflight

일종의 예비 요청으로, 단순 요청이 아닐 경우에 브라우저가 자동으로 전송하는 요청이다.  
클라이언트의 요청과 서버의 응답을 확인하여 실제 요청을 전송할지 결정하게 된다.

1. 클라이언트에서 `OPTIONS /resource` 요청을 `Origin` 헤더와 함께 전송
2. 서버에서 `Access-Control-*` 헤더와 함께 응답(성공하는 경우 200 OK)
    - `Access-Control-Allow-Origin` : 허용 `origin`
    - `Access-Control-Allow-Methods` : 허용 `method`
    - `Access-Control-Allow-Headers` : 허용 `header`
    - `Access-Control-Max-Age` : preflight 요청 결과를 캐싱할 시간
3. 클라이언트에서 `Access-Control-Allow-Origin` 헤더를 확인하여 실제 본 요청을 전송할지 결정
4. preflight 유효 시간 동안 캐싱하여 다음 요청부터는 preflight 요청을 생략하고 바로 본 요청을 전송

###### 참고자료

- [MDN](https://developer.mozilla.org/ko/docs/Web/HTTP/CORS)