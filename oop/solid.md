# 좋은 객체 지향 설계의 5가지 원칙(SOLID)

클린코드로 유명한 로버트 마틴이 정의한 객체 지향 설계의 5가지 원칙

## SRP: 단일 책임 원칙(Single Responsibility Principle)

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

## OCP: 개방-폐쇄 원칙 (Open-Closed principle)

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

// 좋은 예, 기존 코드를 변경하지 않고 확장 가능
class Test1 {
    private Shape shape;

    public Test1(Shape shape) {
        this.shape = shape;
    }

    public void doSomething() {
        shape.calculateArea();
        // ...
    }
}


// 나쁜 예, 기능을 추가하기 위해선 필드와 조건문을 추가해야하는 변경이 필요
class Test2 {
    private Circle circle = new Circle();
    private Rectangle rectangle = new Rectangle();

    public void doSomething(String shape) {
        if (shape.equals("circle")) {
            circle.calculateArea();
        } else if (shape.equals("rectangle")) {
            rectangle.calculateArea();
        }
        // ...
    }
}
```

위 코드에서 기능을 추가하기 위해서는 Shape 인터페이스를 구현하는 클래스만 추가하면 된다.

## LSP: 리스코프 치환 원칙 (Liskov Substitution Principle)

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

## ISP: 인터페이스 분리 원칙 (Interface segregation principle)

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

## DIP: 의존 역전 원칙 (Dependency inversion principle)

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
