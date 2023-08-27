---
layout: editorial
---

# 제어자

> 클래스, 변수 또는 메서드의 선언부에 함께 사용되어 부가적인 의미를 부여하는 키워드

- 접근 제어자 : `public`, `protected`, `private`, `default`
- 그 외 제어자 : `static`, `final`, `abstract`, `synchronized`, `native`, `transient`, `volatile`, `strictfp`

접근 제어자는 하나의 대상에 대해 하나만 사용할 수 있다.

## static - 클래스의, 공통적인

인스턴스 변수는 각 인스턴스마다 별도의 저장 공간을 가지지만, 클래스 변수는 모든 인스턴스가 공통된 저장 공간을 공유한다.

|  대상   | 의미                                                                             |
|:-----:|:-------------------------------------------------------------------------------|
| 멤버 변수 | - 모든 인스턴스에 공통적으로 사용되는 클래스 변수가 된다<br>- 인스턴스 생성 없이 사용 가능<br>- 클래스가 메모리에 로딩될 때 생성 |
|  메서드  | - 인스턴스 생성 없이 사용 가능<br>- 인스턴스 멤버 직접 사용 불가능                                      |

## final - 마지막의, 변경될 수 없는

|    대상     | 의미                                                      |
|:---------:|:--------------------------------------------------------|
|    클래스    | - 변경될 수 없는 클래스, 확장될 수 없는 클래스가 됨<br>- 다른 클래스의 조상이 될 수 없음 |
|    메서드    | - 변경될 수 없는 메서드<br>- 오버라이딩을 통한 재정의 불가능                   |
| 멤버변수/지역변수 | - 값을 변경할 수 없는 상수가 됨                                     |

### 생성자를 이용한 final 멤버 변수 초기화

final 멤버 변수는 선언과 동시에 초기화 하거나, 생성자를 이용해 초기화 할 수 있다.  
생성자를 이용하여 초기화할 경우 인스턴스마다 다른 값을 가질 수 있다.

```java
class Card {
    final int NUMBER; // 상수
    final String KIND;

    Card(String kind, int num) {
        KIND = kind;
        NUMBER = num;
    }
}
```

## abstract - 추상의, 미완성의

메서드의 선언부만 작성하고 구현부는 작성하지 않은 추상 메서드를 포함한 클래스를 추상 클래스라고 하며, 이 클래스를 상속받는 클래스는 반드시 추상 메서드를 구현해야 한다.  
또한 추상 클래스는 인스턴스를 생성할 수 없다.

| 대상  | 의미                                                                    |
|:---:|:----------------------------------------------------------------------|
| 클래스 | - 추상 메서드를 포함한 클래스<br>- 인스턴스를 생성할 수 없음<br>- 상속을 통해 자손 클래스를 만들어 사용      |
| 메서드 | - 구현부가 없는 추상 메서드<br>- 선언부만 작성하고 구현부는 작성하지 않음<br>- 자손 클래스에서 반드시 구현해야 함 |

```java
abstract class Player {
    abstract void play(int pos); // 추상 메서드

    abstract void stop(); // 추상 메서드

    void go() { // 일반 메서드
        // ...
    }
}

class AudioPlayer extends Player {
    void play(int pos) { /* ... */ }

    void stop() { /* ... */ }
}
```

### 추상 클래스 vs 인터페이스

|         |        추상 클래스        |      인터페이스      |
|:-------:|:--------------------:|:---------------:|
|   상속    |      단일 상속만 가능       |    다중 상속 가능     |
|  접근 제어  |   모든 접근 제어자 사용 가능    | `public`만 사용 가능 |
| 인스턴스 생성 |         불가능          |       불가능       |
|   변수    |        선언 가능         |    상수만 선언 가능    |
| 메서드 선언  | 추상 메서드, 일반 메서드 모두 가능 |  추상 메서드만 선언 가능  |

## 접근 제어자

- 접근 범위

|  접근 제어자   | 같은 클래스 | 같은 패키지 | 자손 클래스 | 그 외의 영역 |
|:---------:|:------:|:------:|:------:|:-------:|
|  public   |   O    |   O    |   O    |    O    |
| protected |   O    |   O    |   O    |         |
|  default  |   O    |   O    |        |         |
|  private  |   O    |        |        |         |

- 접근 제어자의 사용

|  대상   |               접근 제어자                |
|:-----:|:-----------------------------------:|
|  클래스  |           public, default           |
| 멤버변수  | public, protected, default, private |
|  메서드  | public, protected, default, private |
|  생성자  | public, protected, default, private |
| 지역 변수 |                  -                  |

### 생성자 private

생성자의 접근 제어자를 `private`으로 선언하여 클래스의 인스턴스를 외부에서 생성하지 못하도록 제한할 수 있다.  
이 특징을 이용해 아래와 같이 활용할 수 있다.

- 불필요한 객체 생성 제한

`JDK`의 `Arrays` 클래스의 모슨 메서드를 `static`으로 구성하여 객체를 생성하지 않고도 사용할 수 있도록 하였다.  
때문에 불필요하게 객체를 생성할 필요가 없기 때문에 `private` 생성자를 사용하여 객체 생성을 제한하였다.

```java
public class Arrays {
    // Suppresses default constructor, ensuring non-instantiability.
    private Arrays() {
    }

    // ...
}
```

- 싱글톤 패턴

싱글톤 패턴은 인스턴스를 오직 하나만 생성하도록 제한하는 패턴으로 `getInstance()` 이용해 하나의 객체만 생성되도록 한다.

```java
public class Singleton {
    private static Singleton s = new Singleton();

    private Singleton() {
        // ...
    }

    public static Singleton getInstance() {
        if (s == null) {
            s = new Singleton();
        }
        return s;
    }
}
```

###### 참고자료

- [java의 정석](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=Java의+정석&isbn=9788994492032&cipId=200741285%2C)