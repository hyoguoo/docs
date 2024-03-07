---
layout: editorial
---

# JVM Version Control

## Java 버전 전환 설정

- `~/.zshrc`에 Java 버전 변경 함수 추가

```shell
function javahome() {
  export JAVA_HOME=$(/usr/libexec/java_home -v "${1:-1.8}");
  java -version
}
```

- java version 확인

```shell
$ /usr/libexec/java_home -V
Matching Java Virtual Machines (3):
    18.0.1.1 (arm64) "Oracle Corporation" - "OpenJDK 18.0.1.1" /Users/hyogu/Library/Java/JavaVirtualMachines/openjdk-18.0.1.1/Contents/Home
    17.0.4 (arm64) "Amazon.com Inc." - "Amazon Corretto 17" /Users/hyogu/Library/Java/JavaVirtualMachines/corretto-17.0.4.1/Contents/Home
    11.0.16 (x86_64) "GraalVM Community" - "GraalVM CE 22.2.0" /Users/hyogu/Library/Java/JavaVirtualMachines/graalvm-ce-11/Contents/Home
/Users/hyogu/Library/Java/JavaVirtualMachines/openjdk-18.0.1.1/Contents/Home
```

- 실행

```shell
$ javahome 11
openjdk version "11.0.16" 2022-07-19
OpenJDK Runtime Environment GraalVM CE 22.2.0 (build 11.0.16+8-jvmci-22.2-b06)
OpenJDK 64-Bit Server VM GraalVM CE 22.2.0 (build 11.0.16+8-jvmci-22.2-b06, mixed mode, sharing)
```