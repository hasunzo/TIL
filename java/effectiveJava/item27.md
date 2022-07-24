# 아이템 27. 비검사 경고를 제거하라

### 제네릭을 사용하기 시작하면 수많은 컴파일러 경고를 보게 될 것이다

- 대부분의 비검사 경고는 쉽게 제거할 수 있다

```java
Set<Lark> exaltation = new HashSet();
```

- 컴파일러가 잘못됐다고 알려줌

```java
Venery.java:4: warning: [unchecked] unchecked conversion
	Set<Lark> exaltation = new HashSet();
required: Set<Lark>
found: HashSet
```

- 컴파일러가 알려준 대로 수정하면 경고가 사라짐
- 자바 7부터 지원하는 다이아몬드 연산자(<>) 로도 해결할 수 있다

```java
Set<Lark> exaltation = new HashSet<>()
```

- 이러면 컴파일러가 올바른 실제 타입 매개변수(이 경우는 Lark)를 추론해준다

### 할 수 있는 한 모든 비검사 경고를 제거하라

- 그 코드는 타입 안정성이 보장된다
- 런타임에 ClassCastException이 발생할 일이 없다

### @SuppressWarnings(”unchecked”) 애너테이션을 달아 경고를 숨기자

- 경고를 제거할 수는 없지만 타입 안전하다고 확신할때!
- 단, 타입 안전함을 검증하지 않은 채 경고를 숨기면 스스로에게 잘못된 보안 인식을 심어주는 꼴
- 그 코드는 경고 없이 컴파일되겠지만,
- 런타임에는 여전히 ClassCastException을 던질 수 있다
- 한편, 안전하다고 검증된 비검사 경고를 숨기지 않고 그대로 두면 진짜 문제를 알리는 새로운 경고가 나와도 눈치채지 못할 수 있음
- @SuppressWarnings(”unchecked”) 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 한다