---
layout: editorial
---

# 스프링과 객체 지향 프로그래밍

- 객체 지향 프로그래밍은 컴퓨터 프로그램을 명령어의 목록으로 보는 시각에서 벗어나 여러 개의 독립된 단위, 즉 "객체"들의 모임으로 파악하고자 하는 것이다. 각각의 객체는 메시지를 주고받고, 데이터를 처리할 수
  있다.
- 객체 지향 프로그래밍은 프로그램을 유연하고 변경이 용이하게 만들기 때문에 대규모 소프트웨어 개발에 많이 사용된다.

## 객체 지향 특징(다형성)

객체 지향엔 추상화 / 캡슐화 / 상속 / 다형성이라는 특징이 있는데 스프링은 다형성 특징을 살리기 좋은 프레임워크다.

### 다형성

다형성은 `역할`과 `구현`으로 구분하여 실세계와 비유해볼 수 있다. 예를 들어 운전자 - 자동차라는 관계가 있고, 자동차 역할은 K3/아반떼/테슬라 등으로 구현할 수 있다.  
사용자는 자동차의 인터페이스를 알고 있고, 위의 자동차들은 자동차 인터페이스를 통해 만들어졌기 때문에 문제 없이 작동할 수 있다. -> 클라이언트에게 영향을 주지 않고 새로운 기능을 추가적으로 개발할 수 있다.

### 역할과 구현 분리

- 역할과 구현으로 구분하면 단순해지고, 유연해지며 변경도 편리해진다.
- 장점
    - 대상의 인터페이스만 알면 된다.
    - 대상의 내부 구조를 몰라도 된다.
    - 대상의 내부 구조가 변경되어도 영향을 받지 않는다.
    - 대상 자체를 변경해도 영향을 받지 않는다.

### 자바에서의 역할과 구현 분리

- 자바 언어의 다형성을 활용
    - 역할 -> 인터페이스
    - 구현 -> 인터페이스를 구현한 클래스, 구현 객체
- 객체를 설계할 때 역할과 구현을 명확히 분리하여, 역할(인터페이스)를 먼저 부여하고 구현 객체 만들기

### 자바 언어의 다형성

자바 기본 문법인 `오버라이딩`으로 다형성을 구현할 수 있다.  
다형성으로 인터페이스를 구현한 객체를 실행 시점에 유연하게 변경할 수 있으며, 클래스 상속 관계에서도 적용 가능하다.

```java
public interface MemberRepository {
    // ...
}

public class JdbcMemberRepository implements MemberRepository {
    // ...
}

public class MemoryMemberRepository implements MemberRepository {
    // ...
}

public class MemberService {
    //    private MemberRepository memberRepository = new MemoryMemberRepository();
    private MemberRepository memberRepository = new JdbcMemberRepository();
}
```

### 다형성의 본질

- 인터페이스를 구현한 객체 인스턴스를 실행 시점에 유연하게 변경할 수 있다.
- 다형성의 본질을 이해하려면 협력이라는 객체사이의 관계에서 시작해야함
- 클라이언트를 변경하지 않고, 서버의 구현 기능을 유연하게 변경할 수 있다.

### 역할과 구현 분리의 한계

- 인터페이스 자체가 변경될 경우 클라이언트/서버 모두 큰 변경이 발생
- 인터페이스를 안정적으로 잘 설계하는 것이 가장 중요

### 스프링과 객체 지향

- 다형성은 객체 지향의 가장 큰 특징
- 스프링은 다형성을 극대화시키기 좋은 프레임워크
- 제어의 역전(IoC) / 의존관계 주입(DI)은 다형성을 활용해 역할과 구현을 편리하게 다룰 수 있도록 지원

## 좋은 객체 지향 설계의 5가지 원칙(SOLID)

클린코드로 유명한 로버트 마틴이 정의한 객체 지향 설계의 5가지 원칙

### SRP: 단일 책임 원칙(Single Responsibility Principle)

- 하나의 클래스는 하나의 책임(목적)만 가져야 한다.
    - 책임은 클 수도 있고, 작을 수도 있으며 상황에 따라 다르다.
    - 클래스를 변경하는 이유가 하나의 이유여야 한다.
- 변경에 용이하게 설계했다면 단일 책임 원칙을 잘 따른 것

```java
class Order {
    private List<Item> items;

    public void addItem(Item item) {
        // 아이템을 주문에 추가하는 로직
    }

    public double calculateTotal() {
        // 주문의 총 가격을 계산하는 로직
    }
}

class Item {
    private String name;
    private double price;

    // 아이템의 이름과 가격을 설정하는 생성자 및 메서드
}
```

Order 클래스는 주문에 관련된 책임만 가지고 있고, Item 클래스는 아이템에 관련된 책임만 가지고 있다.

### OCP: 개방-폐쇄 원칙 (Open-Closed principle)

- 확장엔 열려있으나, 변경에는 닫혀있어야 한다.
- 새로운 기능을 추가할 때 기존 코드를 변경하지 않고 확장할 수 있도록 설계해야 한다.

```java
interface Shape {
    double calculateArea();
}

class Circle implements Shape {
    private double radius;

    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

class Rectangle implements Shape {
    private double width;
    private double height;

    @Override
    public double calculateArea() {
        return width * height;
    }
}
```

위 코드에서 기능을 추가하기 위해서는 Shape 인터페이스를 구현하는 클래스만 추가하면 된다.

### LSP: 리스코프 치환 원칙 (Liskov Substitution Principle)

- 객체는 정확성을 깨뜨리지 않으면서 상위 클래스의 객체를 하위 클래스로 대체 가능해야 한다.
- 부모 클래스의 규악을 자식 클래스가 다 지켜야 한다는 것을 의미한다.

```java
class Rectangle {
    private int width;
    private int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}

class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        super.setWidth(width);
        super.setHeight(width);
    }

    @Override
    public void setHeight(int height) {
        super.setWidth(height);
        super.setHeight(height);
    }
}

class Example {
    public static void main(String[] args) {
        Rectangle rectangle = new Rectangle();
        rectangle.setWidth(10);
        rectangle.setHeight(20);
        System.out.println(rectangle.getArea()); // 200

        Rectangle square = new Square();
        square.setWidth(10);
        square.setHeight(20);
        System.out.println(square.getArea()); // 400
    }
}
```

위 코드에서 에러가 발생하지 않지만 넓이를 구하는데에 논리적인 차이가 있어 의도하지 않은 결과가 나올 수 있다.

### ISP: 인터페이스 분리 원칙 (Interface segregation principle)

- 여러 기능을 포함한 범용 인터페이스 보단 특정 클라이언트를 위한 인터페이스 여러 개를 생성하는 것이 좋다.
- 인터페이스가 명확해지며 대체 가능성이 높아진다.

```java
interface Printer {
    void print(Document document);
}

interface Scanner {
    void scan(Document document);
}

class AllInOnePrinter implements Printer, Scanner {
    public void print(Document document) {
        // 출력 로직
    }

    public void scan(Document document) {
        // 스캔 로직
    }
}
```

인터페이스를 여러 개로 분리하여 클래스에 필요한 인터페이스만 구현하도록 하면, 필요한 기능만 사용할 수 있고, 대체 가능성도 높아진다.

### DIP: 의존 역전 원칙 (Dependency inversion principle)

- "추상화에 의존해야하고, 구체화에 의존하면 안된다."를 따르는 원칙
- 구현 클래스에 의존하지 않고, 인터페이스에 의존해야 한다.
- 구현체에 의존하게 되면 변경이 어려워지고, 테스트도 어려워진다.

```java
interface PaymentProvider {
    void processPayment(int amount);
}

class KakaoPaymentProvider implements PaymentProvider {
    public void processPayment(int amount) {
        // 카카오페이 결제 로직
    }
}

class OrderService {
    private final PaymentProvider paymentProvider;

    public OrderService(PaymentProvider paymentProvider) {
        this.paymentProvider = paymentProvider;
    }

    public void processOrder(Order order) {
        // 주문 처리 로직
        paymentProvider.processPayment(order.getTotalAmount());
    }
}

class OrderServiceTest {
    void processOrder() {
        OrderService orderService = new OrderService(new KakaoPaymentProvider());
        orderService.processOrder(new Order());
    }
}
```

OrderService 클래스는 PaymentProvider 인터페이스에 의존하고 있기 때문에, KakaoPaymentProvider 클래스를 구현하여 인터페이스의 구현체를 주입받아 사용할 수 있다.

## 정리

객체 지향의 핵심은 다형성이지만, 다형성 하나 만으로는 객체 지향 설계의 원칙을 지킬 수 없으며, 변경 용이성도 매우 떨어지게 된다.  
스프링은 DI 컨테이너를 제공하여 다형성 + OCP/DIP를 가능하게 지원해준다.  
모든 설계에 인터페이스를 부여하는 것이 이상적이지만, 추상화라는 비용(구현체를 봐야하는)이 발생한다.  
그래서 기능 확장 가능성이 없는 경우엔 구체 클래스를 직접 사용하고, 필요할 때 도입하는 겻도 나쁘지 않은 방법이다.

###### 참고자료

- [스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/스프링-핵심-원리-기본편)