---
layout: editorial
---

# Naming Convention

> 일반적으로 통용되는 명명 규칙을 따르라

자바 플랫폼의 명명 규칙은 잘 정립되어 있어 일관성 있고 읽기 쉬운 코드를 작성할 수 있게 도와준다.  
패키지/클래스/인터페이스/메서드/필드/타입 변수/상수 등의 이름을 지을 때는 일반적으로 통용되는 명명 규칙들이 존재하며, 이를 따르는 것이 좋다.

## 철자 규칙

철자 규칙은 비교적 통일된 편이며, 대부분의 경우에는 표준 철자를 따르는 것이 좋다.

- 패키지: 각 요소를 `.`으로 구분하여 계층적으로 구성
    - 패키지 이름은 모두 소문자로 작성
    - 각 요소는 일반적으로 8자 이하의 짧은 단어로 구성(긴 경우엔, 약어나 각 단어의 첫 글자만 따서 사용)
    - 많은 기능을 제공하는 경우엔 계층을 나눠 많은 요소로 구성(`java.util.concurrent.atomic` 등)
    - 조직 바깥에서도 사용될 패키지인 경우 일반적으로 회사 도메인 이름을 역순으로 사용
        - `com.google`, `com.google.common`, `com.google.common.collect` 등
- 클래스/인터페이스: 하나 이상의 단어로 이뤄지며, 파스칼 케이스를 따른다.
    - 여러 단어의 첫 글자만 딴 약자나, 널리 통용되는 줄임말(`max`, `min` 등)을 제외하고는 줄여 쓰지 않음
    - 약어의 경우 첫 글자만 대문자인 경우가 많음(`HttpUrl`, `XmlHttpRequest` 등)
- 메서드/필드: 첫 글자를 소문자로 쓴다는 점(캐멀 케이스)만 제외하면 클래스/인터페이스와 동일
    - 상수(`static final`) 필드: 예외로 상수 필드는 모두 대문자로 작성하며, 여러 단어의 조합인 경우 `_`로 구분(`MIN_VALUE`, `NEGATIVE_INFINITY` 등)
- 지역변수: 필드와 비슷한 명명 규칙이 적용되지만, 약어를 사용하는 경우가 많음(`i`, `x`, `y` 등)
- 타입 매개변수: 보통 한 글자로 표현(`T`, `E`, `K`, `V` 등)
    - T: 임의의 타입
    - E: 컬렉션의 요소 타입
    - K/V: 맵의 키/값 타입
    - X: 예외 타입
    - R: 반환 타입
    - 그 외: T, U, V 등의 연속된 알파벳 혹은 T1, T2, T3 등의 숫자를 붙여서 표현

## 문법 규칙

문법 규칙은 철자 규칙과 달리 더 유연하고 논란이 많아, 규칙이 따로 없는 경우가 많다.

|                  종류                  |                        설명                        |                     네이밍 예시                     |
|:------------------------------------:|:------------------------------------------------:|:----------------------------------------------:|
|           객체를 생성할 수 있는 클래스           |                  단수 명사나 명사구 사용                   |           `Thread`, `PriorityQueue`            |
|           객체를 생성할 수 없는 클래스           |                  복수형 명사나 명사구 사용                  |          `Collections`, `Collectors`           |
|               인터페이스 이름               | 클래스와 똑같이 짓거나 -able, -ible, -er 등의 형용사로 끝나는 이름 사용 |      `Runnable`, `Iterable`, `Accessible`      |
|                애너테이션                 |   지배적인 규칙이 없이 대부분은 명사, 동사, 전치사, 형용사 등을 조합하여 사용   | `BindingAnnotation`, `Inject`, `ImplementedBy` |
|             동작을 수행하는 메서드             |                (목적어를 포함한) 동사구 사용                 |             `append`, `drawImage`              |
|          boolean을 반환하는 메서드           |                is, has 등의 접두어 사용                 |       `isEmpty`, `isDigit`, `isEnabled`        |
|           객체의 속성을 반환하는 메서드           |              명사, 명사구 혹은 get 접두어 사용               |         `size`, `hashCode`, `getTime`          |
| 객체의 타입을 바꿔서 다른 타입의 또 다른 객체를 반환하는 메서드 |                   toType 형태 사용                   |             `toArray`, `toString`              |
|        객체의 내용을 다른 뷰로 보여주는 메서드        |                   asType 형태 사용                   |               `asList`, `asMap`                |
|      객체의 값을 기본 타입 값으로 반환하는 메서드       |                 typeValue 형태 사용                  |           `intValue`, `doubleValue`            |
