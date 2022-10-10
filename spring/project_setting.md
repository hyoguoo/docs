### Environment
- macOS
- Java 11
- Intellij IDE


### 프로젝트 생성
- Spring Boot Starter Site https://start.spring.io
```
- Project: Gradle Project
- Spring Boot: Recently Stable Version
- Language: Java
- Packaging: Jar
- Java: 11
- Dependencies: Spring Web, Thymeleaf
```

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

### Project JDK 설정
Project Structure(⌘;) -> Project SDK 설정

### Gradle JDK 설정
Intellij Setting - Build, Execution, Deployment - Gradle projects - Build and run
- Build and run using : Intellij IDEA
- Run tests using : Intellij IDEA
Gradle을 통해 실행하게 되면 간혹 느릴 때가 있어, IDEA에서 바로 Java를 실행시키는 것이 훨씬 빠르다.