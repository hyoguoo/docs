# Facade Pattern(퍼사드 패턴)

> 복잡성을 감추기 위해 단순화된 인터페이스를 제공하는 구조적 디자인 패턴

잡한 시스템에 단순화되고 높은 수준의 인터페이스를 제공하여 클라이언트가 시스템과 더 쉽게 상호 작용할 수 있도록 하는 구조적 디자인 패턴이다.

## 퍼사드 패턴의 구성

- Facade: 클라이언트가 하위 시스템에 액세스하는 진입점, 기본 하위 시스템의 복잡성을 추상화하고 숨기는 단순화된 고급 인터페이스를 제공
- 하위 시스템: 복잡한 시스템을 구성하는 개별 구성 요소 또는 클래스

아래 코드는 퍼사드 패턴을 사용하여 복잡한 하위 시스템을 단순화된 인터페이스로 래핑하는 방법을 보여준다.

```java
// 하위 시스템
class SubSystem1 {
    public void doSomething() {
        System.out.println("SubSystem1");
    }
}

class SubSystem2 {
    public void doSomething() {
        System.out.println("SubSystem2");
    }
}

class SubSystem3 {
    public void doSomething() {
        System.out.println("SubSystem3");
    }
}

// 퍼사드
class Facade {
    private SubSystem1 subSystem1;
    private SubSystem2 subSystem2;
    private SubSystem3 subSystem3;

    public Facade() {
        this.subSystem1 = new SubSystem1();
        this.subSystem2 = new SubSystem2();
        this.subSystem3 = new SubSystem3();
    }

    public void doSomething() {
        subSystem1.doSomething();
        subSystem2.doSomething();
        subSystem3.doSomething();
    }
}

class Client {
    Facade facade = new Facade();

    // 퍼사드를 통해 하위 시스템에 접근
    public void doSomething1() {
        facade.doSomething();
        // ...
    }

    public void doSomething2() {
        facade.doSomething();
        // ...
    }
}
```

## 퍼사드 패턴의 이점

- 클라이언트 코드 단순화: 사용하기 쉬운 단일 인터페이스를 제공하여 클라이언트 코드를 단순화(하위 시스템의 복잡성을 은닉)
- 유지보수성 향상: 복잡한 시스템에 대한 변경 사항이 Facade 클래스 내에 포함되어 코드베이스의 유지 보수 용이성 향상

## 파사드 패턴을 이용한 순환 참조 해결

- 순환 참조: 두 개 이상의 클래스가 서로에 대한 참조를 보유하고 있는 상태

순환 참조는 객체 지향 프로그래밍에서 흔히 발생하는 문제로, 순환 참조가 발생하면 코드를 이해하기 어려워지고 유지보수가 어려워진다.  
이 문제는 파사드 패턴을 사용하여 해결할 수 있다.

- 순환 참조가 발생하는 코드

```java
class ClassA {
    private ClassB classB;

    public ClassA() {
        this.classB = new ClassB();
    }

    public void doSomething() {
        // ...
    }

    public void useClassB() {
        classB.doSomething();
    }
}

class ClassB {
    private ClassA classA;

    public ClassB() {
        this.classA = new ClassA();
    }

    public void doSomething() {
        // ...
    }

    public void useClassA() {
        classA.doSomething();
        // ...
    }
}
```

- 퍼사드 패턴을 사용하여 순환 참조 해결

```java
class ClassA {
    private ClassB classB;

    public ClassA(ClassB classB) {
        this.classB = classB;
    }

    public void doSomething() {
        // ...
    }

    public void useClassB() {
        classB.doSomething();
    }
}

class ClassB {
    // ClassA를 필드로 가지고 있지 않음
//    private ClassA classA;

    public ClassB() {
    }

    public void doSomething() {
        // ...
    }

    // ClassA를 사용하는 메서드 삭제
//    public void useClassA() {
//        classA.doSomething();
//        // ...
//    }
}

class Facade {
    private ClassA classA;
    private ClassB classB;

    public Facade() {
        this.classA = new ClassA(this.classB);
        this.classB = new ClassB(this.classA);
    }

    public void useClassAwithClassB() {
        classA.doSomething();
        classB.doSomething();
        // ...
    }
}
```

`ClassB`에서 `ClassA`를 필요로 했던 메서드를 별도의 `Facade` 클래스로 분리하여 `ClassB`가 더이상 `ClassA`를 필요로 하지 않도록 하여 순환 참조를 해결할 수 있다.