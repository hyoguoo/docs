# Java Virtual Machine

자바의 큰 특징 중 하나는 어느 플랫폼이든 컴파일 된 코드가 플랫폼 독립적이다.  
이럴 수 있는 이유는 컴파일 된 코드를 실행시켜주는 가상 환경인 JVM 존재 때문이다. JVM은 자바를 실행하기 위한 가상 환경이라고 생각하면 되며, 자바로 작성된 애플리케이션은 모두 JVM에서만 실행되기 때문에
실행을 위해서는 반드시 JVM이 필요하다.  
JVM을 거치기 때문에 속도가 느릴 수 있다는 단점을 가지고 있을 수 있지만, 컴파일러와 최적화 기술로 그 속도의 격차를 많이 줄여 그 단점을 완화했다.

|    Java Application    |
|:----------------------:|
| (Windows/Mac/Linux)JVM |
| OS(Windows/Mac/Linux)  |
|   Computer(Hardware)   |

Java Application은 위와 같이 JVM 위에서 실행이 되는데, 해당 OS에 맞는(실행 가능 한) JVM이 있다면, 모든 OS에서도 Application의 변경 없이 실행이 가능하다.

## Java 실행 과정

![img.png](../image/java_excution_process.png)

1. 작성된 Java Source를 Java Compiler를 통해 Java Byte Code로 컴파일
2. 컴파일 된 Byte Code를 JVM의 Class Loader에 전달
3. Class Loader는 Dynamic Loading을 통해 필요한 클래스들을 로딩 및 링크하여 Runtime Data Area(JVM Memory)로 전달
4. Execution Engine이 올라온 Byte Code들을 명령어 단위로 하나씩 가져와서 실행

## JDK & JRE & JVM

![img.png](../image/java_jdk_diagram.png)

### JVM(Java Virtual Machine)

자바 바이트 코드를 실행시키기 위한 가상 머신

### JRE(Java Runtime Environment) = JVM + Library

자바 애플리케이션을 실행하기 위한 도구(필요한 라이브러리 및 필수 파일)가 포함된 실행 환경

### JDK(Java Development Kit)

자바로 개발하기 위한 필요 요소(javac 등)를 포함한 개발 키트

###### 출처

- Java의 정석