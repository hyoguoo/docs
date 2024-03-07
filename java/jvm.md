---
layout: editorial
---

# Java Virtual Machine

> 자바 애플리케이션이 수행되는 런타임 엔진

|    Java Application    |
|:----------------------:|
| JVM(Windows/Mac/Linux) |
| OS(Windows/Mac/Linux)  |
|   Computer(Hardware)   |

자바의 큰 특징 중 하나는 가상 환경인 JVM의 존재인데, Computer -> OS -> JVM -> Java Application 레이어 형태로 JVM 위에서 실행된다.  
이 존재로 아래와 같은 특징을 가지게 된다.

- JVM 위에서 실행되기 때문에 어느 플랫폼이든 컴파일 된 코드가 실행될 수 있음
- 때문에 실행을 위해서는 반드시 JVM이 필요함
- JVM 거치기 때문에 속도가 느릴 수 있음(최근엔 컴파일러와 최적화 기술로 그 속도의 격차를 많이 줄임)

## Java 실행 과정

![java execution process](image/java-execution-process.png)

- Class Loader: 바이트 코드 로딩 / 검증 / 링킹 등 수행
- Runtime Data Area: 앱 실행을 위해 사용되는 JVM 메모리 영역
- Execution Engine: 메모리 영역에 있는 데이터를 가져와 해당하는 작업 수행

1. 작성된 Java Source를 Java Compiler를 통해 Java Byte Code로 변환
2. 컴파일 된 Byte Code를 JVM의 Class Loader에 전달
3. Class Loader는 Dynamic Loading을 통해 필요한 클래스들을 로딩 및 링크하여 Runtime Data Area(JVM Memory)로 전달
4. Execution Engine이 올라온 Byte Code들을 명령어 단위로 하나씩 가져와서 실행

## JDK & JRE & JVM

![java jdk diagram](image/java-jdk-diagram.png)

- JVM(Java Virtual Machine): 자바 바이트 코드를 실행시키기 위한 가상 머신
- JRE(Java Runtime Environment): 자바 애플리케이션을 실행하기 위한 도구(필요한 라이브러리 및 필수 파일)가 포함된 실행 환경
- JDK(Java Development Kit): 자바로 개발하기 위한 필요 요소(javac 등)를 포함한 개발 키트

## JVM 메모리 구조

JVM 메모리는 크게 5가지 영역으로 나뉘며, 각각의 영역은 다음과 같은 역할을 수행한다.

|   영역   |                    용도                    |           생명 주기            | 스레드 공유 여부 |
|:------:|:----------------------------------------:|:--------------------------:|:---------:|
| Method |    클래스 정보, 클래스(static) 변수, 상수, 메소드 코드    |        JVM 시작 ~ 종료         |     O     |
|  Heap  |             객체 인스턴스, 인스턴스 변수             | `Gabage Collection`에 의해 관리 |     O     |
| Stack  | 쓰레드 별로 런타임에 호출 된 메서드, 지역 변수, 매개 변수, 리턴 값 |          메서드 종료 시          |     X     |

이외에도 쓰레드의 명령어 주소가 저장되는 `PC Register` 자바 외의 언어로 작성된 네이티브 코드가 저장되는 `Native Method Stack`이 존재한다.

###### 참고자료

- [java의 정석](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Java의+정석&isbn=9788994492032&cipId=200741285%2C)
