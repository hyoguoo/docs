# URI(Uniform Resource Identifier)

URI는 로케이터(locator), 이름(name) 또는 둘 다 추가로 분류될 수 있다.

![img.png](../image/uri_diagram.png)

```
　　　　　　　　　　　　　　　　　　　　ｈｉｅｒａｒｃｈｉｃａｌ　ｐａｒｔ
　　　　　　　　┌───────────────────┴────────────────────┐
　　　　　　　　　　　　　　　　　　　　ａｕｔｈｏｒｉｔｙ　　　　　　　　　　ｐａｔｈ
　　　　　　　　┌───────────────┴──────────────┐┌───┴────┐
　　ａｂｃ：／／ｕｓｅｒｎａｍｅ：ｐａｓｓｗｏｒｄ＠ｅｘａｍｐｌｅ．ｃｏｍ：１２３／ｐａｔｈ／ｄａｔａ？ｋｅｙ＝ｖａｌｕｅ＃ｆｒａｇｉｄ１
　　└┬┘　　　└───────┬───────┘└────┬─────┘└─┬┘　　　　　　　　└───┬────┘└───┬──┘
ｓｃｈｅｍｅ　　ｕｓｅｒ　ｉｎｆｏｒｍａｔｉｏｎ　　　　ｈｏｓｔ　　　  ｐｏｒｔ　　　　　　　　  ｑｕｅｒｙ　　　ｆｒａｇｍｅｎｔ

　　ｕｒｎ：ｅｘａｍｐｌｅ：ｍａｍｍａｌ：ｍｏｎｏｔｒｅｍｅ：ｅｃｈｉｄｎａ
　　└┬┘　└──────────────┬───────────────┘
ｓｃｈｅｍｅ　　　　　　　　　　　　　　ｐａｔｈ
```

## URI/URL/URN 단어 뜻

- Uniform: 리소스 식별하는 통일된 방식
- Resource: 자원, URI로 식별할 수 있는 모든 것(제한 없음)
- Identifier: 다른 항목과 구분하는데 필요한 정보
- Locator: 리소스가 있는 위치를 지정
- Name: 리소스에 이름을 부여

## URL 전체 문법

- scheme://[userinfo@]host[:port][/path][?query][#fragment]
- https://www.google.com:443/search?q=hello&hl=ko

|    문법    |             내용              |                       설명                       |
|:--------:|:---------------------------:|:----------------------------------------------:|
|  scheme  | http, https, ftp, file, ... |               리소스에 접근하기 위한 프로토콜                |
| userinfo |        user:password        |          서버에 접근할 때 사용자 정보(거의 사용하지 않음)          |
|   host   |       www.google.com        |                  호스트명 또는 IP주소                  |
|   port   |        80, 443, 8080        |              접근 포트(일부 port 생략 가능)              |
|   path   |           /search           |                 리소스 경로, 계층적 구조                 |
|  query   |       ?q=hello&hl=ko        | key=value 형태, `?`로 시작 `&`로 구분, query parameter |
| fragment |          #bookmark          |          html 내부 북마크, id(서버에 전송하지 않음)          |

###### 출처

- https://www.inflearn.com/course/http-웹-네트워크