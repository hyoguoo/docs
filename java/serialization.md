---
layout: editorial
---

# Serialization

> 객체의 상태를 바이트 스트림으로 변환하여 파일, 데이터베이스, 네트워크 등으로 전송할 수 있도록 하는 기술

직렬화(Serialization)란 객체에 저장된 데이터를 스트림에 쓰기(write)위해 연속적인(serial) 데이터로 변환하는 것을 말한다.

## ObjectOutputStream과 ObjectInputStream

ObjectOutputStream과 ObjectInputStream은 각각 객체의 직렬화와 역직렬화를 처리하는 스트림 클래스로, 기본적인 직렬화/역직렬화 메커니즘을 제공한다.

### ObjectOutputStream

객체를 직렬화하여 출력 스트림에 기록할 수 있도록 해준다. 기본적으로 객체의 모든 필드를 직렬화하지만, transient로 표시된 필드는 제외된다.

- defaultWriteObject():
    - 기본 직렬화 동작을 수행
    - 이 메서드를 호출하지 않으면 기본 필드가 직렬화되지 않음
    - 커스터마이징이 필요할 경우, 이 메서드 호출 후 추가적인 데이터를 기록 가능
- writeObject()
    - 기본 직렬화 동작을 커스터마이징하기 위해 정의할 수 있음
    - 이 메서드를 통해 transient 필드를 포함한 추가적인 데이터를 직접 기록 가능

### ObjectInputStream

직렬화된 객체를 역직렬화하여 입력 스트림에서 객체로 복원할 수 있도록 해준다.

- defaultReadObject()
    - 기본 역직렬화 동작을 수행
    - 이 메서드를 호출하지 않으면 기본 필드가 역직렬화되지 않음
    - 메서드 호출 후, 추가적인 데이터를 읽어들일 수 있음
- readObject()
    - 기본 역직렬화 동작을 커스터마이징하기 위해 정의할 수 있음
    - 이 메서드를 통해 직렬화 과정에서 기록된 추가적인 데이터 복원 가능

## Serializable 인터페이스

Java에서 직렬화를 지원하기 위해 `Serializable` 인터페이스를 구현해야 한다.

- 인터페이스는 특별한 메서드를 정의 할 필요 없으며, 단순히 해당 객체가 직렬화될 수 있음을 나타냄
- Java 직렬화 과정에서 특정 동작을 커스터마이징하기 위해 여러 메서드를 정의할 수 있음

### transient 키워드

transient 키워드는 직렬화 과정에서 특정 필드를 제외하고 싶을 때 사용된다.

- 비밀번호와 같은 민감한 정보나 불필요한 정보에 선언
- transient로 선언된 필드는 기본적으로 직렬화되지 않으며, 역직렬화 시에는 해당 타입의 기본값으로 초기화됨

### 직렬화와 관련된 메서드

직렬화와 관련된 주요 메서드와 각 메서드의 역할은 다음과 같다.

|                       Method                       |          Description           |
|:--------------------------------------------------:|:------------------------------:|
| `private void writeObject(ObjectOutputStream out)` | 직렬화 과정 중 객체 데이터를 직접 쓰는 방법을 정의  |
|  `private void readObject(ObjectInputStream in)`   | 역직렬화 과정 중 객체 데이터를 직접 읽는 방법을 정의 |
|           `private Object readResolve()`           |   역직렬화 후, 반환할 인스턴스를 대체하거나 변경   |
|          `private Object writeReplace()`           |   직렬화 전, 직렬화될 인스턴스를 대체하거나 변경   |
|         `private void readObjectNoData()`          |    직렬화된 데이터 없이 객체를 읽을 때 호출     |

### Serializable 인터페이스 구현 예시

아래는 `Serializable` 인터페이스를 구현한 예시 코드로, 수행되는 순서는 다음과 같다.

1. `writeReplace`: 직렬화 전, 직렬화될 인스턴스를 대체하거나 변경
2. `writeObject`: 직렬화 과정 중 객체 데이터를 직접 쓰는 방법을 정의
3. `readObject`: 역직렬화 과정 중 객체 데이터를 직접 읽는 방법을 정의
4. `readResolve`: 역직렬화 후, 반환할 인스턴스를 대체하거나 변경

```java
class Person implements Serializable {

    private static final long serialVersionUID = 1L;

    private String name;
    private int age;
    private transient String password; // 직렬화에서 제외될 필드

    public Person(String name, int age, String password) {
        this.name = name;
        this.age = age;
        this.password = password;
    }

    // 1.
    private Object writeReplace() throws ObjectStreamException {
        System.out.println("writeReplace called");
        // 직렬화 시 이름을 "Replaced Ogu"으로 변경하여 반환
        return new Person("Replaced " + name, age, password);
    }

    // 2.
    private void writeObject(ObjectOutputStream out) throws IOException {
        System.out.println("writeObject called");
        out.defaultWriteObject(); // 기본 직렬화 동작
        // 비밀번호를 Base64로 인코딩하여 직렬화
        String encodedPassword = Base64.getEncoder().encodeToString(password.getBytes());
        out.writeObject(encodedPassword); // name, age, password 순서로 직렬화
    }

    // 3.
    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        System.out.println("readObject called");
        in.defaultReadObject(); // 기본 역직렬화 동작, name, age 복원
        // 직렬화된 비밀번호를 Base64로 디코딩하여 복원
        String encodedPassword = (String) in.readObject(); // password 복원
        password = new String(Base64.getDecoder().decode(encodedPassword));
    }


    // 4.
    private Object readResolve() throws ObjectStreamException {
        System.out.println("readResolve called");
        return new Person(name, age + 100, password);
    }

    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + ", password='" + password + "'}";
    }
}

class SerializationExample {

    public static void main(String[] args) {
        Person person = new Person("Ogu", 59, "secretPassword");

        // 직렬화
        try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("person.ser"))) {
            out.writeObject(person);
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 역직렬화
        try (ObjectInputStream in = new ObjectInputStream(new FileInputStream("person.ser"))) {
            Person deserializedPerson = (Person) in.readObject();
            System.out.println("Deserialized Person: " + deserializedPerson);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```
