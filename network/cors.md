# CORS(Cross-Origin Resource Sharing)

## CORS(교차 출처 리소스 공유)

브라우저는 same-origin policy(동일 출처 정책)에 의해 cross-origin의 리소스 요청을 차단하는데 cross-origin의 종류는 아래로 정의된다.

- 다른 도메인 (example.com - test.com)
- 다른 하위 도메인 (example.com - store.example.com)
- 다른 포트 (example.com:80 - example.com:90)
- 다른 프로토콜 (https://example.com - http://example.com)

#### 예시(기준 `http://store.ogufamily.com`)

|                    URL                    |    Result    |   Reason    |
|:-----------------------------------------:|:------------:|:-----------:|
|  http://store.ogufamily.com/babyogu.html  | same origin  | path만 다른경우  |
|    http://store.ogufamily.com/ogu.html    | same origin  | path만 다른경우  |
|        https://store.ogufamily.com        | cross origin | 프로토콜이 다른 경우 |
|       http://store.ogufamily.com:81       | cross origin |  포트가 다른경우   |
|         http://news.ogufamily.com         | cross origin |  도메인이 다른경우  |

## CORS 에러

에러가 발생했을 때 아래와 같은 에러 메시지가 표시되는데

```shell
*** has sbeen blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```

언어 및 프레임워크 별로 설정 방법은 다르지만 서버 쪽에서 Origin 설정을 해주어 간단히 해결할 수 있다.

## Preflight

기본적으로 브라우저는 `cross-origin` 요청을 전송하기 전에 `OPTIONS` METHOD로 preflight를 전송하면 Response로 아래의 정보가 넘어오게 된다.

- `Access-Control-Allow-Origin` : 허용 `origin`
- `Access-Control-Allow-Methods` : 허용 `method`

결과를 성공적으로 확인 후 `cross-origin` 요청을 보내서 그 이후 과정을 진행하게 된다. 때문에 `cross-origin` 요청을 보낼 때 마다 preflight 요청을 보낸다고 생각할 수 있는데,  
캐싱을 통해 그 결과를 일정 기간 저장시켜 바로 요청을 가능하도록 한다. 캐싱 시간은 서버 쪽 `cross-origin` 설정 중 `maxAge`에 값을 주어 설정할 수 있다.

###### 참고자료

- [MDN](https://developer.mozilla.org/ko/docs/Web/HTTP/CORS)