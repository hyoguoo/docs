---
layout: editorial
---

# Doc Comment

> 공개된 API 요소에는 항상 문서화 주석을 작성하라

공개된 API를 올바로 문서화하기 위해선 아래의 요건들을 충족해야 한다.

- 공개된 모든 클래스, 인터페이스, 메서드, 필드 선언에 문서화 주석을 달아야 한다.
- 직렬화 가능 클래스라면 직렬화 형태에 관해서도 문서화해야 한다.
- 스레드 안전하든 안하든 스레드 안전 수준을 명시해야 한다.
- 유지보수까지 고려한다면 공개되지 않은 클래스, 인터페이스, 생성자, 메서드, 필드에도 간략하게나마 문서화 주석을 달아두는 것이 좋다.
- 기본 생성자에는 문서화 주석을 달 수 없으니 기본 생성자를 사용하면 안된다.
- 첫 번째 문장은 요약 설명으로 간주되기 때문에 대상의 기능을 고유하게 기술해야 한다.

## 메서드 문서화 주석

특히 메서드 문서화 주석은 해당 메서드와 클랑리언트 사이의 규약을 명료하게 기술해야 한다.

- 어떻게(how)가 아닌 무엇(what)을 하는지를 기술
- 클라이언트가 메서드를 호출하기 위한 전제조건(precondition) 기술
    - 일반적으로 `@throws` 태그로 비검사 예외를 선언하여 암시적으로 기술
- 메서드 수행 후 만족해야 하는 사후조건(postcondition)을 기술
- 부시스템의 상태에 영향을 미치는 경우 부작용(side effect)을 기술
- `@param`, `@return` 태그 사용하여 메서드의 매개변수와 반환값 기술
    - 관례상 해당 매개변수가 뜻하는 값이나 반환값의 의미를 간략하게 기술
- `@throws` 태그 사용하여 메서드가 던질 수 있는 모든 예외 기술(검사/비검사 모두)

위 규칙을 지킨 문서화 주석 예시는 아래와 같다.

```java
class Example {
    /**
     * 이 리스트에서 지정한 위치의 원소를 반환한다.
     *
     * <p>This method is <i>not</i> guaranteed to run in constant time. In some
     * implementations it may run in time proportional to the element position.
     * <p>이 메서드는 상수 시간에 수행됨을 보장하지 <i>않는다</i>. 구현에 따라
     * 원소의 위치에 비례한 시간이 걸릴 수도 있다.
     *
     * @param  index 반환할 원소의 인덱스; 0 이상이고 리스트 크기보다 작아야 한다.
     * @return 이 리스트에서 지정한 위치의 원소
     * @throws IndexOutOfBoundsException index 범위를 벗어나면,
     *         ({@code index < 0 || index >= this.size()})이면 발생한다.
     */
    E get(int index);
}
// `{@code ...}` 태그는 코드를 인라인으로 포함시키는 태그로, 코드를 인용하는 것과 같은 효과를 내는데, 코드용 폰트로 렌더링해주면서 HTML 특수문자나 다른 자바독 태그를 무시해준다.
// 비슷한 태그로 `{@literal ...}` 태그가 있는데, 이는 HTML 특수문자를 무시해주는 효과만 있고 코드용 폰트로 렌더링해주지는 않는다.
```

## 그 외에 특수한 문서화 주석

그 외에 제네릭 타입, 열거 타입, 애너테이션 타입을 문서화할 때는 추가적인 주석을 달아야 한다.

- 제네릭 타입 : 모든 타입 매개변수
- 열거 타입 : 모든 상수
- 애너테이션 타입 : 모든 멤버
