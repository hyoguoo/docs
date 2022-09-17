https://start.spring.io

- Project : 과거에는 라이브러리를 땡겨서 오고 빌드하는 라이프사이클을 관리해주는 툴, 현재는 Gradle이 강세
  - Maven
  - Gradle

- Spring Boot : 버전 선택

- Project Metadata : 현재 프로젝트에 맞게 설정

- Dependencies : 어떤 라이브러리를 가져올 지 선택
  - Spring Web
  - Thymeleaf

### Spring 생성 시 프로젝트 구조
```
- gradle : gradle 관련 폴더
- src
  - main
    - java : 실제 패키지와 소스파일
    - resources : 실제 자바 코드 파일을 제외한 xml, properties, html 등 java 파일을 제외한 파일들
  - test : 테스트 코드 소스
- build.gradle : gradle build 설정 파일
- gradlew
- gradlew.bat
- settings.gradle
```