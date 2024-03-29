---
layout: editorial
---

# Exception Documentation

> 메서드가 던지는 모든 예외를 문서화하라

검사 예외는 항상 따로 선언하면서, 각 예외가 발생하는 상황을 자바독의 `@throws` 태그를 사용해 정확히 문서화해야 한다.(공통 상위 클래스 하나로 선언하지 말고, 각각의 예외를 선언해야 한다.)  
예외 문서화에 대한 가이드는 아래와 같다.

1. `main` 메서드(`main` 메서드는 JVM만이 호출하기 때문)를 제외하고서는 `Exception`이나 `Throwable` 같은 공통 상위 클래스를 던지는 것은 좋지 않다.
2. 자바 언어 자체가 요구하는 것은 아니지만 비검사 예외도 검사 예외처럼 문서화하여, 메서드를 성공적으로 수행할 수 있도록 도와주는 게 좋다.
    - public 메서드라면 비검사 예외도 문서화하여 메서드 사용자에게 알려주는 것이 좋다.
    - 나아가 인터페이스 메서드에 선언하여 일반 규약에 속하게 만들어 모든 구현체가 일관되게 동작하도록 할 수도 있다.
3. 메서드가 던질 수 있는 예외를 각각 `@throws` 태그로 문서화하지만, 비검사 예외는 `throws` 목록에 넣지 말아야 한다.
    - 검사/비검사에 따라 사용자가 처리해야 할 일이 달라지기 때문에 그 둘을 확실히 구분해주어야 사용자가 어느 것이 비검사 예외인지 바로 알 수 있다.
4. 한 클래스에 정의된 많은 메서드가 같은 이유로 같은 예외를 던진다면, 메서드가 아닌 클래스 설명에 해당 예외를 문서화하는 것이 좋다.
