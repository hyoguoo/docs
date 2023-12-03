---
layout: editorial
---

# Effective Java(이펙티브 자바)

해당 페이지에 존재하는 모든 문서들은
[이펙티브 자바](https://www.nl.go.kr/seoji/contents/S80100000000.do?schM=intgr_detail_view_isbn&page=1&pageUnit=10&schType=simple&schStr=%EC%9D%B4%ED%8E%99%ED%8B%B0%EB%B8%8C+%EC%9E%90%EB%B0%94&isbn=9788966262281&cipId=227313228%2C6952194)
3판을 읽고 정리한 내용입니다.

## Chapter 2. 객체 생성과 파괴

|                   Item                    |                  Title                  |
|:-----------------------------------------:|:---------------------------------------:|
| [Item 1. Static Factory Method](item1.md) |         생성자 대신 정적 팩터리 메서드를 고려하라         |
|        [Item 2. Builder](item2.md)        |         생성자에 매개변수가 많다면 빌더를 고려하라         |
|       [Item 3. Singleton](item3.md)       |     private 생성자나 열거 타입으로 싱글턴임을 보증하라     |
|  [Item 4. Noninstantiability](item4.md)   |     인스턴스화를 막으려거든 private 생성자를 사용하라      |
| [Item 5. Dependency Injection](item5.md)  |      자원을 직접 명시하지 말고 의존 객체 주입을 사용하라      |
|  [Item 6. Unnecessary Objects](item6.md)  |             불필요한 객체 생성을 피하라             |
|    [Item 7. Obsolete Object](item7.md)    |             다 쓴 객체 참조를 해제하라             |
|  [Item 8. Finalizer & Cleaner](item8.md)  |       finalizer와 cleaner 사용을 피하라        |
|  [Item 9. try-with-resources](item9.md)   | try-finally보다는 try-with-resources를 사용하라 |

## Chapter 3. 모든 객체의 공통 메서드

|               Item               |              Title              |
|:--------------------------------:|:-------------------------------:|
|   [Item 10. equals](item10.md)   |     equals는 일반 규약을 지켜 재정의하라     |
|  [Item 11. hashCode](item11.md)  | equals를 재정의하려거든 hashCode도 재정의하라 |
|  [Item 12. toString](item12.md)  |       toString을 항상 재정의하라        |
|   [Item 13. clone](item13.md)    |      clone 재정의는 주의해서 진행하라       |
| [Item 14. Comparable](item14.md) |      Comparable을 구현할지 고려하라      |

## Chapter 4. 클래스와 인터페이스

|                       Item                       |                   Title                   |
|:------------------------------------------------:|:-----------------------------------------:|
|      [Item 15. Access Modifier](item15.md)       |           클래스와 멤버의 접근 권한을 최소화하라           |
|      [Item 16. Accessor Method](item16.md)       | public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라 |
|    [Item 17. Minimize Mutability](item17.md)     |               변경 가능성을 최소화하라               |
|        [Item 18. Composition](item18.md)         |             상속보다는 컴포지션을 사용하라              |
|        [Item 19. Inheritance](item19.md)         |   상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라   |
| [Item 20. Abstract Class & Interface](item20.md) |           추상 클래스보다는 인터페이스를 우선하라           |
|       [Item 21. Default Method](item21.md)       |          인터페이스는 구현하는 쪽을 생각해 설계하라          |
|     [Item 22. Constant Interface](item22.md)     |         인터페이스는 타입을 정의하는 용도로만 사용하라         |
|         [Item 23. Subtyping](item23.md)          |        태그 달린 클래스보다는 클래스 계층구조를 활용하라        |
|        [Item 24. Nested Class](item24.md)        |         멤버 클래스는 되도록 static으로 만들라          |
| [Item 25. Limit File Top-level Class](item25.md) |          톱레벨 클래스는 한 파일에 하나만 담으라           |

## Chapter 5. 제네릭

|                          Item                           |            Title            |
|:-------------------------------------------------------:|:---------------------------:|
|             [Item 26. Raw Type](item26.md)              |        로 타입은 사용하지 말라        |
|         [Item 27. Unchecked Warning](item27.md)         |        비검사 경고를 제거하라         |
|           [Item 28. List vs Array](item28.md)           |       배열보다는 리스트를 사용하라       |
|           [Item 29. Generic Type](item29.md)            |      이왕이면 제네릭 타입으로 만들라      |
|          [Item 30. Generic Method](item30.md)           |      이왕이면 제네릭 메서드로 만들라      |
|           [Item 31. Wildcard Type](item31.md)           | 한정적 와일드카드를 사용해 API 유연성을 높이라 |
|              [Item 32. Varargs](item32.md)              |   제네릭과 가변인수를 함께 쓸 때는 신중하라   |
| [Item 33. Type Safe Heterogeneous Container](item33.md) |     타입 안전 이종 컨테이너를 고려하라     |

## Chapter 6. 열거 타입과 애너테이션

|                  Item                  |              Title               |
|:--------------------------------------:|:--------------------------------:|
|       [Item 34. Enum](item34.md)       |      int 상수 대신 열거 타입을 사용하라       |
|     [Item 35. Ordinal](item35.md)      |   ordinal 메서드 대신 인스턴스 필드를 사용하라   |
|     [Item 36. EnumSet](item36.md)      |      비트 필드 대신 EnumSet을 사용하라      |
|     [Item 37. EnumMap](item37.md)      |   ordinal 인덱싱 대신 EnumMap을 사용하라   |
|  [Item 38. Extended Enum](item38.md)   | 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라 |
|    [Item 39. Annotation](item39.md)    |       명명 패턴보다는 애너테이션을 사용하라       |
|    [Item 40. @Override](item40.md)     |    @Override 애너테이션을 일관되게 사용하라    |
| [Item 41. Marker Interface](item41.md) |  정의하려는 것이 타입이라면 마커 인터페이스를 사용하라   |

## Chapter 7. 람다와 스트림

|                       Item                        |         Title          |
|:-------------------------------------------------:|:----------------------:|
|           [Item 42. Lambda](item42.md)            |   익명 클래스보다는 람다를 사용하라   |
|      [Item 43. Method Reference](item43.md)       |   람다보다는 메서드 참조를 사용하라   |
|    [Item 44. Functional Interface](item44.md)     |   표준 함수형 인터페이스를 사용하라   |
|           [Item 45. Stream](item45.md)            |     스트림은 주의해서 사용하라     |
|  [Item 46. Side-Effect-Free Function](item46.md)  | 스트림에서는 부작용 없는 함수를 사용하라 |
| [Item 47. Return Collection or Stream](item47.md) | 반환 타입으로는 스트림보다 컬렉션이 낫다 |
|       [Item 48. Parallel Stream](item48.md)       |   스트림 병렬화는 주의해서 적용하라   |

## Chapter 8. 메서드

|                    Item                    |            Title             |
|:------------------------------------------:|:----------------------------:|
| [Item 49. Parameter Validation](item49.md) |       매개변수가 유효한지 검사하라        |
|    [Item 50. Defensive Copy](item50.md)    |       적시에 방어적 복사본을 만들라       |
|   [Item 51. Method Signature](item51.md)   |      메서드 시그니처를 신중히 설계하라      |
|     [Item 52. Overloading](item52.md)      |        다중정의는 신중히 사용하라        |
|       [Item 53. Varargs](item53.md)        |        가변인수는 신중히 사용하라        |
|   [Item 54. Empty Collection](item54.md)   |  null이 아닌, 빈 컬렉션이나 배열을 반환하라  |
|       [Item 55. Optional](item55.md)       |        옵셔널 반환은 신중히 하라        |
|     [Item 56. Doc Comment](item56.md)      | 공개된 API 요소에는 항상 문서화 주석을 작성하라 |

## Chapter 9. 일반적인 프로그래밍 원칙

|                            Item                             |              Title              |
|:-----------------------------------------------------------:|:-------------------------------:|
|         [Item 57. Local Variable Scope](item57.md)          |         지역변수의 범위를 최소화하라         |
|             [Item 58. For-each Loop](item58.md)             | 전통적인 for 문보다는 for-each 문을 사용하라  |
|                [Item 59. Library](item59.md)                |         라이브러리를 익히고 사용하라         |
|          [Item 60. Decimal Calculation](item60.md)          | 정확한 답이 필요하다면 float와 double은 피하라 |
| [Item 61. Primitive Type & Boxed Primitive Type](item61.md) |    박싱된 기본 타입보다는 기본 타입을 사용하라     |
|           [Item 62. Avoid String Type](item62.md)           |    다른 타입이 적절하다면 문자열 사용을 피하라     |
|             [Item 63. String Concat](item63.md)             |      문자열 연결은 느리니 주의해서 사용하라      |
|          [Item 64. Interface Reference](item64.md)          |       객체는 인터페이스를 사용해 참조하라       |             
