---
layout: editorial
---

# Reflection(리플렉션)

> 런타임 시점의 자바 프로그램에서 클래스 / 인터페이스 / 필드 메서드 등의 정보를 동적으로 검사하고 조작할 수 있는 기능

리플렉션을 하면 클래스의 정보를 가져올 수 있고, 문법 및 간단한 사용 방법은 아래와 같다.

```java
class ReflectionExample {

    private final String privateField;

    public ReflectionExample() {
        privateField = "Private field value";
    }

    public static void main(String[] args) throws Exception {
        ReflectionExample obj = new ReflectionExample();

        // 1. 클래스 정보 가져오기
        Class<?> clazz = obj.getClass();
        System.out.println("Class name: " + clazz.getName());

        // 2. 필드 정보 가져오기
        Field[] fields = clazz.getDeclaredFields();
        for (Field field : fields) {
            System.out.println("Field name: " + field.getName());
        }

        // 3. 메서드 정보 가져오기
        Method[] methods = clazz.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println("Method name: " + method.getName());
        }

        // 4. private 필드에 접근하여 값 가져오기
        Field privateField = clazz.getDeclaredField("privateField");
        String fieldValue = (String) privateField.get(obj);
        System.out.println("Private field value: " + fieldValue);

        // 5. private 필드에 접근하여 값 변경하기
        privateField.setAccessible(true);
        privateField.set(obj, "New private field value");
        System.out.println("New private field value: " + obj.privateField);

        // 6. private 메서드 호출하기
        Method privateMethod = clazz.getDeclaredMethod("privateMethod");
        privateMethod.invoke(obj);

        // 7. 정의되지 않은 메서드 호출하기
        try {
            clazz.getDeclaredMethod("undefinedMethod").invoke(obj);
        } catch (NoSuchMethodException e) {
            System.out.println(e); // java.lang.NoSuchMethodException: ReflectionExample.undefinedMethod()
        }
    }

    public void publicMethod() {
        System.out.println("Public method");
    }

    private void privateMethod() {
        System.out.println("Private method");
    }
}
```

1. 클래스 정보 가져오기
    - `Class<?> clazz = obj.getClass();`: ReflectionExample 클래스의 정보를 가져온다.
2. 필드 정보 가져오기
    - `Field[] fields = clazz.getDeclaredFields();`: ReflectionExample 클래스의 모든 필드 정보를 가져온다.
3. 메서드 정보 가져오기
    - `Method[] methods = clazz.getDeclaredMethods();`: ReflectionExample 클래스의 모든 메서드 정보를 가져온다.
4. private 필드에 접근하여 값 가져오기
    - `Field privateField = clazz.getDeclaredField("privateField");`: ReflectionExample 클래스의 privateField 정보를 가져온다.
    - `String fieldValue = (String) privateField.get(obj);`: ReflectionExample 클래스의 privateField 값을 가져온다.
5. private 필드에 접근하여 값 변경하기
    - `privateField.setAccessible(true);`: ReflectionExample 클래스의 privateField에 접근할 수 있도록 설정한다.
    - `privateField.set(obj, "New private field value");`: ReflectionExample 클래스의 privateField 값을 변경한다.
6. private 메서드 호출하기
    - `Method privateMethod = clazz.getDeclaredMethod("privateMethod");`: ReflectionExample 클래스의 privateMethod 정보를 가져온다.
    - `privateMethod.invoke(obj);`: ReflectionExample 클래스의 privateMethod를 호출한다.
7. 정의되지 않은 메서드 호출하기
    - `clazz.getDeclaredMethod("undefinedMethod").invoke(obj);`: ReflectionExample 클래스의 undefinedMethod를 호출한다.
    - 현재 클래스에 존재하지 않기 때문에 NoSuchMethodException이 발생하여 catch 블록이 실행된다.

## 장점

클래스와 인터페이스의 정보를 가져와서 사용할 수 있기 때문에 애플리케이션을 개발할 때 유연하게 할 수 있게 된다.

1. 동적 객체 생성: 런타임 중 동적으로 객체를 생성하여 사용자의 입력이나 외부 파일에 따라 클래스를 동적으로 로딩하여 객체를 생성할 수 있다.
2. 클래스 정보 검사: 클래스의 구조를 검사하고, 필드와 메서드 정보를 가져와서 사용할 수 있다.
3. 기존 동작 변경: 클래스의 private 필드나 메서드에 접근하여 값을 변경하거나 메서드를 호출할 수 있다.
4. 프레임워크 확장: 프레임워크를 사용할 때 리플렉션을 사용하면 프레임워크의 기능을 확장할 수 있다.

## 단점

자유롭게 접근하고 사용할 수 있는 만큼 리스크가 큰 기능이기 때문에 사용할 때 주의해야 한다.

1. 성능 저하: 리플렉션은 런타임 시점에 클래스의 정보를 가져오기 때문에 성능 저하가 발생할 수 있다.
2. 접근 지시자 무시: 리플렉션을 사용하면 private 필드나 메서드에 접근할 수 있기 때문에 접근 지시자를 무시할 수 있다.
3. 컴파일 타임에 확인되지 않는 오류: 리플렉션을 사용하면 컴파일 타임에 확인되지 않는 오류가 발생할 수 있다.

###### 참고자료

- [스프링 핵심 원리 - 고급편](https://www.inflearn.com/course/스프링-핵심-원리-고급편)