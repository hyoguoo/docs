---
layout: editorial
---

# Wildcard type

> 한정적 와일드카드를 사용해 API 유연성을 높이라

[item 28](item28.md)에서 언급했듯 매개변수화 타입은 불공변(invariant)인 부분에 대해 의문점을 가질 수 있다.  
List<String>은 List<Object>의 하위 타입이 아닌데, List<String>은 List<Object>가 하는 일을 제대로 수행할 수 없기 때문에 리스코프 치환 원칙에 따르면 불공변인 것이 타당하다.

하지만 이대로 사용하기엔 불편한 점이 많아 매개변수화 타입을 유연하게 사용할 수 있도록 도와주는 방법이 있는데, 바로 한정적 와일드카드 타입(bounded wildcard type)을 사용하는 것이다.
