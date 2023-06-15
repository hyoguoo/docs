# 타입

> 타입 = 개념, 일상적인 용어인 개념을 공학적으로 대체한 것

## 데이터 타입

실제 메모리에서는 0과 1의 행렬만 존재하기 때문에 메모리 영역에서는 타입을 구분할 수 없다.  
프로그래머가 이러한 메모리 영역을 용도와 행동에 따라 구분하기 위해 타입 시스템(type system)이 생겨났다.

- 데이터가 어떻게 사용되는 것의 정의
- 데이터를 메모리에 어떻게 표현하는지 감춤

그럼으로써 위의 두 가지의 장점을 얻을 수 있게 되었고, 데이터를 사용하는 프로그래머에게 데이터의 의미를 명확하게 전달할 수 있게 되었다.

## 객체와 타입

객체지향 프로그램을 작성할 때 객체를 일종의 데이터처럼 사용하지만, 객체에서 중요한 것은 데이터가 아닌 행동이며 다시 한 번 아래와 같이 정리할 수 있다.

- 객체가 수행하는 행동에 따라 객체의 타입이 결정된다.
- 내부 표현은 외부로부터 감춰진다.
- 객체가 어떤 데이터를 가지고 있는지는 타입을 분류하는데에 영향이 없으며, 어떤 행동을 하는지에 따라 타입을 분류한다.
- 반대로 동일 데이터지만 다른 행동인 경우엔 다른 타입으로 분류하게 된다.

위 원칙들을 통해 객체지향 설계에 중요한 원칙을 이끌어 내게 된다.

- `다형성`: 동일 책임이지만 내부 표현 방식이 다르다는 것은 해당 책임을 처리하는 방식은 다를 수 있다는 것을 의미
- `캡슐화`: 데이터의 내부 표현 방식과 무관하게 행동만을 고려하는 것은 외부에 내부 데이터를 감추는 것을 의미

## 일반화/특수화(generalization/specialization)

타입과 타입 사이에는 일반화/특수화 관계가 존재할 수 있다.

- 특수한 타입(Subtype)
    - 일반적인 개념보다 범위가 더 좁다는 것을 의미
    - 특수한 타입보다 더 적은 수의 행동을 가지게 되는 것
- 일반적인 타입(Supertype)
    - 더 포괄적인 의미를 내포하기 때문에 더 넓은 범위를 의미
    - 특수한 타입이 가진 행동들 중 일부 행동을 가지게 되는 것

## 정적 모델

### 타입의 목적

타입안 동적으로 변하는 객체의 복잡성을 타입이라는 개념을 이용해 추상화하여 정적인 관점에서 표현할 수 있게 된다.  
결국 타입은 추상화를 통해 객체의 복잡성을 다루기 위한 방법이다.

### 동적 모델과 정적 모델

- 동적 모델(dynamic model): 객체가 특정 시점에 구체적으로 어떤 상태를 가지는지 표현하는 것(=스냅샷(snapshot))
- 정적 모델(static model): 객체가 가질 수 있는 모든 상태와 모든 행동을 시간에 독립적으로 표현하는 것(=타입 다이어그램(type diagram))

객체지향 애플리케이션을 설계하고 구현하기 위해서는 객체 관점의 동적 모델과 타입 관점의 정적 모델을 적절히 혼용해야 한다.  
실제 프로그래밍을 할 때 아래와 같이 접근하게 된다.

- 클래스를 작성하는 시점: 시스템을 정적인 관점에서 접근하는 것
- 실제로 애플리케이션이 동작하는 시점: 객체의 동적인 관점에서 접근하는 것

### 클래스

객체지향 프로그래밍 언어에서 정적인 모델은 클래스를 이용해 구현되고 타입을 구현하는 가장 보편적인 방법은 클래스를 이용하는 것이다.  
그렇다고 클래스와 타입은 동일한 것이 아니고 타입은 객체를 분류하기 위해 사용하는 개념일 뿐이며 클래스는 타입을 구현하는 수단 중 하나일 뿐이다.

###### 참고자료

- [객체지향의 사실과 오해: 역할, 책임, 협력 관점에서 본 객체지향](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=객체지향의+사실&isbn=9788998139766&cipId=200539082%2C4626710)