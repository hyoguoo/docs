# Java Virtual Machine
자바의 큰 특징 중 하나는 어느 플랫폼이든 컴파일 된 코드가 플랫폼 독립적이다.  
이럴 수 있는 이유는 컴파일 된 코드를 실행시켜주는 가상 환경인 JVM 존재 때문이다. 

## Java 실행 과정
![img.png](../image/java_excution_process.png)
1. 작성된 Java Source를 Java Compiler를 통해 Java Byte Code로 컴파일
2. 컴파일 된 Byte Code를 JVM의 Class Loader에 전달
3. Class Loader는 Dynamic Loading을 통해 필요한 클래스들을 로딩 및 링크하여 Runtime Data Area(JVM Memory)로 전달
4. Execution Engine이 올라온 Byte Code들을 명령어 단위로 하나씩 가져와서 실행

