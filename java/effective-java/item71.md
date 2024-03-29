---
layout: editorial
---

# Necessary Checked Exception

> 필요 없는 검사 예외 사용을 피하라

검사 예외는 번거로운 일이지만 제대로 활용하면 API와 프로그램의 질을 높일 수 있다.(예외 처리를 강제함으로써 API 사용자가 예외 상황에서 복구할 수 있도록 도와줌)  
하지만 과하게 사용하면 쓰기 불편한 API를 낳고, 사용하는 클라이언트의 코드 쪽의 부담을 주기 때문에 꼭 필요한 곳에만 검사 예외를 사용하는 것이 좋다.

## 검사 예외를 사용해야하는 경우

아래와 같은 조건을 만족하는 경우엔 검사 예외를 사용하여 에러 처리를 강제하는 것이 좋다.

- API를 올바르게 사용하더라도 발생할 수 있는 예외
- 예외 발생 시 프로그래머가 의미 있는 조치를 취할 수 있는 경우

하지만 만약 위 둘 중 하나라도 해당하지 않으면 검사 예외보다는 비검사 예외를 사용하는 것이 좋다.

## 검사 예외 사용 회피

검사 예외를 던지는 것은 많은 부담을 주기 때문에 최소화 하는 것이 좋다.  
때문에 검사 예외를 회피하는 방법을 알아보자.

### 옵셔널 반환

검사 예외를 회피하는 가장 쉬운 방법은 옵셔널을 반환하는 것이다.  
옵셔널을 반환하면서 예외 대신 빈 옵셔널을 반환하는 방식으로 검사 예외를 회피할 수 있다.(대신 예외 발생 이유에 대해 부가 정보를 제공할 수 없다.)

### 상태 검사 메서드

기존 검사 예외를 던지는 메서드를 아래 두 개의 메서드로 분리하여 검사 예외를 회피할 수 있다.

1. 상태 검사 메서드: 예뢰가 던져질지 여부를 boolean으로 반환
2. 동작 메서드: 상태 검사 메서드를 통해 검사 예외를 회피한 뒤 동작을 수행(검사 예외 대신 비검사 예외를 던질 수 있음)

```java
class Example {
    
    public static void main(String[] args) {
        if (obj.actionPermitted(args)) {
            obj.action(args);
        } else {
            // 예외 상황 대처 코드
        }
    }
}
```
