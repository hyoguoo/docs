---
layout: editorial
---

# Adapter Pattern(어댑터 패턴)

> 호환되지 않는 인터페이스를 가진 개체가 함께 작동할 수 있도록 하는 구조적 디자인 패턴

이는 호환되지 않는 두 인터페이스 사이의 브리지 역할을 하여 기존 클래스의 소스 코드를 변경하지 않고도 호환되도록 만든다.  
이 패턴은 재사용하거나 새 시스템에 통합하려는 기존 클래스(또는 시스템)가 있지만 해당 인터페이스가 새 시스템에 필요한 것과 일치하지 않을 때 유용하다.

## 어댑터 패턴의 구성

- 대상 인터페이스: 클라이언트 코드가 사용할 인터페이스
- Adaptee: 수정하려는 기존 클래스 또는 시스템
- Adapter: Adaptee가 대상 인터페이스와 호환되도록 변환하는 클래스

```java
// 적용 시키려는 기존 시스템
class OldSystem {
    public double getTemperatureInCelsius() {
        return 59.0;
    }
}

// 새로운 시스템을 위한 인터페이스
interface NewSystemInterface {
    double getTemperatureInCelsius();

    double getTemperatureInFahrenheit();
}

// 기존 시스템을 새로운 시스템에 적용하기 위한 어댑터
class Adapter extends OldSystem implements NewSystemInterface {
    @Override
    public double getTemperatureInCelsius() {
        return super.getTemperatureInCelsius();
    }

    @Override
    public double getTemperatureInFahrenheit() {
        return (getTemperatureInCelsius() * 9 / 5) + 32;
    }
}

class Client {
    public static void main(String[] args) {
        NewSystemInterface newSystem = new Adapter();

        double celsiusTemperature = newSystem.getTemperatureInCelsius();
        double fahrenheitTemperature = newSystem.getTemperatureInFahrenheit();

        System.out.println("섭씨 온도: " + celsiusTemperature);
        System.out.println("화씨 온도: " + fahrenheitTemperature);
    }
}
```

## 어댑터 패턴의 이점

- 재사용성: 새 코드와 호환되지 않는 기존 클래스 재사용 가능
- 관심사항 분리: Adaptee 작동 방식에 대한 세부사항과 클라이언트 코드를 분리
- 유연성: 여러 Adaptee를 단일 대상 인터페이스에 적용할 수 있어 유연성이 향상
- 레거시 시스템 통합: 코드를 변경하지 않고 레거시 시스템을 최신 시스템에 통합할 수 있음

## 어댑터 패턴 사용 사례의 예

- 라이브러리 통합: 애플리케이션의 인터페이스 요구 사항을 준수하지 않는 라이브러리가 있는 경우 가능한 인터페이스를 제공하는 어댑터 생성하여 라이브러리를 사용
- 하드웨어 통합: 하드웨어 장치를 다룰 때 어댑터를 사용하여 하드웨어와 소프트웨어 간 호환성을 제공
- API 버전 호환성: API가 변경되거나 업데이트될 때 이전 버전의 API와의 호환성을 유지하기 위해 어댑터를 사용
- 데이터베이스 드라이버: 어댑터를 사용하면 다양한 데이터베이스 드라이버가 공통 데이터베이스 액세스 인터페이스를 구현하도록 할 수 있음(예: JDBC)

요약하면, 어댑터 패턴은 호환되지 않는 두 인터페이스 사이의 변환기 역할을 하여 두 인터페이스가 원활하게 함께 작동할 수 있도록 한다.