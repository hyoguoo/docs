---
layout: editorial
---

# Optimization

> 최적화는 신중히 하라

프로그래밍할 때는 빠르고 좋은 성능도 중요하지만, 견고한 구조를 가진 명확한 좋은 프로그램을 만드는 것도 중요하다.  
만약 좋은 구조로 만들어진 프로그램이라면 좋은게 보통이고, 성능이 나오지 않더라도 아키텍처 자체를 바꾸지 않고도 독립적으로 성능을 개선할 수 있다.  
또한 만약 최적화를 해야한다면 최적화 시도 전후로 성능을 측정하여 적용 여부를 판단해야 한다는 것이다.

## API를 설계가 성능에 미치는 영향

public 타입을 가변으로 만들면(내부 데이터를 변경할 수 있게) 불필요한 방어적 복사를 유발하게 되어 성능 문제로 이어질 수 있다.  
예를 들어 `java.awt.Component`의 `getSize` 메서드는 가변 타입인 `Dimension`을 반환하기 때문에 방어적 복사를 해야한다.  
때문에 `getSize`를 호출할 때마다 새로운 객체를 생성해야 해서 성능 저하로 이어질 수 있는 것이다.

이를 더 올바르게 설계한다면 아래와 같이 해볼 수 있다..

- `Dimension`을 불변 타입으로 만든다.(가장 이상적인 방법)
- `getSize`를 `getWidth`와 `getHeight`로 나누어 `Dimension` 객체의 기본 타입 값들을 따로따로 반환하는 방식으로 변경한다.
