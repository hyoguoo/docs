---
layout: editorial
---

# 시큐어 코딩

> 소프트웨어(SW)를 개발함에 있어 개발자의 실수, 논리적 오류 등으로 인해 SW에 내포될 수 있는 보안취약점(vulnerability)을 배제하기 위한 코딩 기법

## 입력 데이터 검증 및 표현

프로그램 입력 값에 대한 검증 누락 또는 부적절한 검증 등으로 SQL Injection, 크로스사이트 스크립트(XSS) 등의 공격 유발 가능

### 1. SQL Injection

사용자로부터 입력된 값을 입력 받은 경우 동적쿼리(Dynamic Query)를 생성하기 때문에 개발자가 의도하지 않은 쿼리가 실행되어 정보유출에 대한 악용 가능성

#### 공격 예시

- 코드와 전달 인자

```typescript
// 인자로 `name"; DROP BOARD; --`; 전달
const sql = `
    SELECT
        *
    FROM
        BOARD
    WHERE
        contents = "${contents}"
`;
```

- 실행되는 쿼리

```sql
SELECT *
FROM BOARD
WHERE contents = "name";
DROP
BOARD; --"
```

#### 안전한 코딩 기법

- 동적 쿼리에 들어오는 값 검증
- Prepared Statement: 쿼리의 문법 처리 과정이 미리 선 수행되고 바인딩 데이터가 할당 되기 때문에 해당 데이터는 SQL 문법적인 의미를 가질 수 없게됨

```typescript
const sql = `
    SELECT
        *
    FROM
        BOARD
    WHERE
        contents = ?
`;

return await this.datasource.query(sql, [
    contents,
]);
```

### 2. 크로스 사이트 스크립트(XSS)

웹 페이지에서 악의적인 스크립트를 포함시켜 사용자 측에서 실행되게 유도하는 방법

#### 안전한 코딩 기법

- 외부 입력 값 또는 출력값에 스크립트가 삽입되지 못하도록 문자열 치환 함수를 사용해 `^ < > * ' / ( )` 등을 `&amp;`등으로 특수문자 코드로 치환하여 사용
- html 태그를 허용하는 대상의 경우 white list를 만들어 해당 태그들만 허용

### 3. 위험한 형식 파일 업로드

서버 측에서 실행될 수 있는 스크립트 파일을 업로드하여 그 파일을 통해 웹을 통해 직접 실행 시켜 공격하는 방법

#### 안전한 코딩 기법

- 특정 파일만 허용하는 white list 설정
- 파일 확장자 및 Content-Type 확인
- 파일 크기 및 파일 개수 제한
- 파일의 실행을 막기 위해 무작위 이름으로 변환 및 해당 파일의 실행 권한 삭제

```typescript
const MEGABYTE = 1024 * 1024;

class File {
    public static isValidImage(image?: Express.Multer.File) {
        this.validateSize(image, 5 * MEGABYTE);
        this.validateImageExtension(image);
    }

    private static validateSize(image: Express.Multer.File, maximumSize: number) {
        if (image.size < 0 || image.size > maximumSize) {
            throw new BadRequestException(
                `이미지 크기가 유효하지 않습니다.(0 ~ ${maximumSize / MEGABYTE}MB)`,
            );
        }
    }

    private static validateImageExtension(image: Express.Multer.File) {
        const validExtensionList = ['jpeg', 'jpg', 'png'];
        const imageExtension = image.mimetype.split('/')[1];

        if (!validExtensionList.includes(imageExtension)) {
            throw new BadRequestException(
                '지원하지 않는 파일 형식입니다.(jpg, jpeg, png)',
            );
        }
    }
}
```

### 4. 경로 조작 및 자원 삽입

검증되지 않은 외부 입력 값을 통해 파일 및 서버 등 시스템 자원에 대한 접근 혹은 식별을 허용할 경우 입력 값을 통해 접근할 경우 생기는 문제

#### 공격 예시

- 변조 전: `http://www.example.com/file/download/?filename=pic.jpg%path=data`
- 변조 후: `http://www.example.com/file/download/?filename=paaword%path=../../etc`

#### 안전한 코딩 기법

- 외부의 입력 값을 식별자로 사용하는 경우 적절한 검증을 거치거나 사전에 정의된 리스트에서 선택되도록 제한
- 외부 입력이 파일명인 경우에는 경로 순회(`/ \ ..`)문자를 제거 하여 사용

```typescript
export class TermsSaveRequest {
    @IsNotEmpty()
    @IsEnum(TermsType)
    termsType: TermsType; // 디렉토리 경로가 될 type을 Enum value로 강제

    @IsNotEmpty()
    @IsDateString()
    revisionDate: string;
}
```

### 5. 운영체제 명령어 삽입

사용자 입력 값에 운영체제 명령어의 일부 또는 전부로 구성되어 실행되는 경우 의도치 않은 시스템 명령어를 유발시킬 수 있는 경우로 시스템 동작 및 운영에 악영향을 미칠 수 있다.

#### 안전한 코딩 기법

- 외부 입력 값이 시스템 명령에 포함되는 경우 `| ; & : > <  \ !` 같은 멀티라인 파이프 리다이렉트 문자 등을 필터링
- 명령어 생성에 필요한 값들을 미리 지정해놓고 외부 입력에 따라 선택하여 사용

### 6. 신뢰하지 않은 URL 주소로 자동접속 연결

사용자로부터 입력되는 값을 외부사이트의 주소로 사용하여 연결하는 서버 프로그램은 피싱(Phishing) 공격 노출 취약성

#### 안전한 코딩기법

- 허용하는 URL을 white list로 관리

```java
class Example {

    public void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String allowURL[] = {"http://url1.com", "http://url2.com", "http://url3.com"};
        String nurl = request.getParameter("nurl");
        try {
            Integer n = Integer.parseInt(nurl);
            if (n >= 0 && n < 3)
                response.sendRedirect(allowURL[n]);
        } catch (NumberFormatException nfe) {
        }
    }
}
```

### 7. 크로스 사이트 요청 위조(Cross-Site Request Forgery)

특정 웹사이트에 대해서 사용자가 인지하지 못한 상황에서 사용자의 의도와는 무관하게 요청하는 공격

1. 공격자가 CSRF 스크립트가 포함된 게시물 등록
2. 사용자가 해당 게시물 조회
3. 사용자의 권한(Auth)으로 CSRF Script 실행
4. 서버는 사용자의 요청으로 인식해 그대로 실행

#### 예시

사용자가 아래와 같은 태그가 포함된 사이트를 열람할 시 아래 태그에 있는 스크립트가 실행되어 비밀번호가 강제로 변경

```html
<img src="https://examplesite.com/user?action=set-pw&pw=1234" , width="0" height="0"/>
```

#### 안전한 코딩 기법

- 정보 수정을 하는 API는 `GET` 메서드를 사용하지 말고 `POST` 사용
- request header에 있는 요청을 한 페이지가 정보가 담긴 referer 속성을 검증하여 차단
- CSRF Token 검증  
  (https://junhyunny.github.io/information/security/spring-boot/spring-security/cross-site-reqeust-forgery/)

```java
// 보통 Host와 Referrer 값이 일치하는 특성을 이용한 방어 코드 
public class ReferrerCheckInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String referer = request.getHeader("Referer");
        String host = request.getHeader("host");
        if (referer == null || !referer.contains(host)) {
            response.sendRedirect("/");
            return false;
        }
        return true;
    }
}
```

### 8. 서버사이드 요청 위조

적절한 검증절차를 거치지 않은 사용자가 입력 값을 서버 간의 요청 안에 악의적인 행위를 넣어 발생 시키는 보안 약점

```
[공격자] <--> [웹 서버] <--> [내부 서버]
# 공격자는 내부 서버에 직접 접근이 불가능한 환경
```

1. 공격자가 서버에 조작한 요청 전송
2. 서버가 해당 요청 처리
3. 수신한 서버가 조작된 요청을 내부 서버에 다시 요청

#### 안전한 코딩 기법

- 사용자의 입력 값을 white list 방식으로 필터링
- 무작위 URL을 받아야 할 경우 반대로 블랙리스트 URL 지정

<참고 : 삽입 코드의 예>

|                 설명                  |                                                               삽입 코드의 예                                                               |
|:-----------------------------------:|:------------------------------------------------------------------------------------------------------------------------------------:|
| 내부망 중요 정보 획득 외부 접근 차단된 admin 페이지 접근 | http://sample_site.com/connect?url=http://192.168.0.45/member/list.json http://sample_site.com/connect?url=http://192.168.0.45/admin |
|        도메인 체크를 우회하여 중요 정보 획득        |                      http://sample_site.com/connect?url=http://sample_site.com:x@192.168.0.45/member/list.json                       |
|        단축 URL을 이용한 Filter 우회        |                                     http://sample_site.com/connect?url=http://bit.ly/sdjk3kjhkl3                                     |
|       도메인을 사설IP로 설정 해 중요정보 획득       |                               http://sample_site.com/connect?url=http://192.168.0.45/member/list.json                                |
|             서버 내 파일 열람              |                                        http://sample_site.com/connect?url=file:///etc/passwd                                         |

### 9. HTTP 응답 분할

- HTTP Request(Response) Message 구조

```http request
HTTP/1.1 403 Forbidden
Server: Apache
Content-Type: test/heal:charset-i8o-8953-1
Date: Wed, 10 Aug 2016 09:23:25 GHT
Keep-Alive: timeout-5, max-1000
Connection: Keep-Alive
Age: 3464
Date: Wed, 10 Aug 2016 09:46:25 GMT
Content-Length: 220

«IDOCTYPE HIML PUBLIC™ T-77IETF//DID HIMI 2.0//EN">
```

|        Start Line        |
|:------------------------:|
|      General Header      |
| Request(Response) Header |
|      Entity Header       |
|     CRLF(Empty Line)     |
|           Body           |

HTTP 요청에 들어 있는 인자 값이 HTTP 응답 헤더에 포함되어 사용자에게 다시 전달될 때 입력값에 CR(CarriageReturn, %0d) + LF(LineFeed, %0a)와 같은 개행 문자가 존재할 경우
HTTP 응답이 두 개 이상으로 분리될 수 있는데, 공격자는 CRLF를 이용해 여러 개의 HTTP 응답을 만들어 해당 응답 메시지에 악의적인 코드를 주입

1. 인젝션 코드가 삽입 된 HTTP 코드 요청

```http request
GET /test?exampleParam=example%0d%0aContentLength:%200%0d%0d%0a%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aContent-Length:%2019%0d%0a%0d%0a<html>Attack</html>
...
```

- 정상적인 HTTP Response

```http request
HTTP/1.1 200 OK
...(Header)
```

- 인젝션 코드로 생성 된 메시지:  
  `%0d%0aContentLength:%200%0d%0d%0a%0aHTTP/1.1%20200%20OK%0d%0aContent-Type:%20text/html%0d%0aContent-Length:%2019%0d%0a%0d%0a<html>Attack</html>`

```http request
ContentsLength: 0

HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 19

<html>Attack</html>
```

2. 서버 측에서 그대로 받아서 처리하여 HTTP 응답을 분할하여 2개가 존재

```http request
HTTP/1.1 200 OK
... 
    # 원래 응답 안에 인젝션 코드 출력
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 19

<html>Attack</html>
... # 원래 응답 값 계속
```

위와 같이 되면 `<html>Attack</html>`이 실행되고 원래 응답 값 일부를 모두 문자열로 인식하여 문자 그대로 화면에 출력하게 된다.

#### 안전한 코딩 기법

- 요청 파라미터 값에 HTTP 응답 헤더를 포함 시킬 경우 CR(\r), LF(\n) 같은 개행 문자 제거
- 외부 입력 값이 헤더, 쿠키, 로그 등에 사용될 경우 항상 개행 문자 검증
- 헤더에 사용되는 예약어 등은 white list로 제한을 권장

```java
class Example {
  ...

    public void replaceCRLF() {
        String contentType = request.getParameter("content-type");
        String filteredContentType = contentType.replaceAll("\r", "").replaceAll("\n", "");
      ...
    }
}
```

## 캡슐화

소프트웨어가 중요한 데이터나 기능성을 불충분하게 캡슐화 하는 경우, 인가된 데이터와 그렇지 않은 데이터를 구분하지 못하게 되어 중요한 정보가 노출되는 경우가 발생한다.  
또한 객체 지향 방법론 중 중요한 개념으로도 사용되며 객체와 필드의 은닉을 통해 외부의 잘못된 사용을 방지하는 것으로 캡슐화가 부적절 혹은 불충분하게 되어 있으면 정보 은닉의 기능을 상실한다.

### 디버그 코드

디버깅 목적으로 삽입 된 코드를 개발 완료된 이후에 제거하지 않은 경우 민감한 정보가 노출 될 수 있다.

#### 안전한 코딩기법

- 배포 전 디버그 코드 삭제
- 환경에 따른 출력 로그 레벨 제한 지정

```typescript
const app = await NestFactory.create(AppModule, {
    logger: process.env.NODE_ENV === 'production'
        ? ['error', 'warn', 'log']
        : ['error', 'warn', 'log', 'verbose', 'debug']
});
```

### private Array(Object) 접근 제한

`private`으로 선언 되었지만 `public`으로 선언 된 메서드를 통해 그대로 반환하면 그 배열의 주소값이 외부에 노출되어 외부에서 배열 수정이 가능해지며,  
또한 `public` 메서드를 통해 `priviate` 선언된 배열에 저장하면 `private` 배열을 외부에서 접근할 수 있게 된다.

#### 안전한 코딩기법

- `private`로 선언된 배열을 `public`을 통해 선언 된 메서드를 통해 반환 하지 않도록 함
- 필요한 경우엔 복제본을 반환 하거나 수정을 제어하는 `public` 메서드를 별도 선언

##### before

```java
class Example {
    private String[] colors;

    public String[] getColors() {
        return colors;
    }

    public void setColors(String[] colors) {
        this.colors = colors
    }
}
```

##### after

```java
class Example {
    private String[] colors;

    public String[] getColors() {
        String[] ret = null;
        if (this.colors != null) {
            ret = new String[colors.length];
            for (int i = 0; i < colors.length; i++) {
                ret[i] = this.colors[i];
            }
            return ret;
        }
    }

    public void setColors(String[] colors) {
        this.colors = new String[colors.length];
        for (int i = 0; i < colors.length; i++) {
            this.colors[i] = colors[i];
        }
    }
}
```

## 그 외

### 1. 보안 기능

인증/접근 제어/기밀성/암호화 등을 올바르게 구현해야하는 것으로 중요한 정보를 암호화 없이 저장하거나 하드코딩 되어 노출 되는 경우가 없어야 한다.

### 2. 코드 오류

형변환 오류/반환/NullPointer 참조와 같이 오류를 발생시키거나, 자원을 필요 이상으로 할당하게 되어 시스템에 부하를 주는 경우가 없어야 한다.

### 3. API 오용

의도된 사용에 반하는 방법으로 API를 사용하거나 보안에 취약한 API의 사용을 자제해야 한다.

### 4. 시간 및 상태

동시 또는 거의 동시 수행을 지원하느 병렬 시스템이나 하나 이상의 프로세스가 동작하는 환경에서 시간 및 상태를 부적절하게 관리하여 생길 수 있는 약점  
Node 기반 프로젝트의 경우 pm2 cluster 모드를 통해 스케줄링을 돌릴 경우 동시에 여러 개의 프로세스가 접근할 수 있다.

### 5. 에러 처리

에러를 처리하지 않거나 에러 정보에 중요 정보가 포함될 때 발생할 수 있는 취약점

- 오류 메시지는 사용자에게 최소한의 정보만 포함
- 예외 상황은 내부적으로 처리
- 민감한 정보를 포함하는 오류 메시지는 출력하지 않도록 미리 정의된 메시지를 제공
- try-catch 와 같은 제어문을 이용한 오류 상황 대응

###### 참고자료

- [한국인터넷진흥원](https://www.kisa.or.kr/2060204/form?postSeq=13&lang_type=KO)
- [행정자치부](https://www.mois.go.kr/frt/bbs/type001/commonSelectBoardArticle.do%3Bjsessionid=fr7QaTyG2gK5o02XJnYETp3havIQ1MGLKMYdWaaEe5me9IOk932SIy2BbP1AM08Z.mopwas54_servlet_engine1?bbsId=BBSMSTR_000000000012&nttId=42152)
- [인포섹 네이버 블로그](https://blog.naver.com/skinfosec2000/220694143144)