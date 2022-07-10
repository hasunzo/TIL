# 아이템 5 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라

- 많은 클래스가 하나 이상의 자원에 의존한다
- 이런 클래스를 정적 유틸리티 클래스로 구현한 모습을 드물지 않게 볼 수 있다

### 정적 유틸리티를 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다

```java
public class SpellChecker {
	private static final Lexicon dictionary = ...;
	
	private SpellChecker() {} // 객체 생성 방지

	public 
}
```

### 싱글턴을 잘못 사용한 예 - 유연하지 않고 테스트하기 어렵다

```java
public class SpellChecker {
	private final Lexicon dictionary = ...;
	
	private SpellChecker(...) {}
	public static SpellChecker INSTANCE = new SpellChecker(...);

	public boolean isValid(String word) { ... }
	public List<String> suggestions(String type) { ... }
}
```

- 두 방식 모두 사전을 단 하나만 사용한다고 가정한다는 점에서 그리 훌륭해 보이지 않다
- 실전에서는 사전이 언어별로 따로 있고 특수 어휘용 사전을 별도로 두기도 한다
- 심지어 테스트용 사전도 필요할 수 있다
- 사전 하나로 이 모든 쓰임에 대응하기란 어려움

### SpellChecker 가 여러 사전을 사용할 수 있도록 만들기

- dictionary 필드에서 final 한정자 제거하고 다른 사전으로 교체하는 메서드 추가
    - 이 방식은 어색하고 오류 내기 쉬움
    - 멀티스레드 환경에서는 쓸 수 없다
- 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다
    - 그럼 어떤 방법이?
        - 클래스(SpellChecker)가 여러 자원 인스턴스를 지원해야 하며,
        - 클라이언트가 원하는 자원을 사용해야 함
    - 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식

```java
// 의존 객체 주입은 유연성과 테스트 용이성을 높여준다
public class SpellChecker {
	private final Lexicon dictionary;
	
	public SpellChecker(Lexicon dictionary) {
		this.dictionary = Objects.requireNonNull(dictionary);
	}

	public boolean isValid(String word) { ... }
	public List<String> suggestions(String typo) { ... }
}
```

- 예에서는 dictionary 라는 딱 하나의 자원만 사용하지만 자원이 몇 개든 의존 관계가 어떻든 잘 작동
- 또한 불변을 보장하여 (같은 자원을 사용하려는) 여러 클라이언트가 의존 객체들을 안심하고 공유할 수 있기도 하다
- 의존 객체 주입은 생성자, 정적 팩터리, 빌더 모두에 똑같이 응용할 수 있다

### 의존 객체 주입 패턴의 쓸만한 변형 → 생성자에 자원 팩터리를 넘겨주기

- 팩터리
    - 호출할 때마다 특정 타입의 인스턴스를 반복해서 만들어주는 객체
    - 팩터리 메서드 패턴을 구현한 것
    - Supplier<T> 인터페이스가 팩터리를 표현한 완벽한 예