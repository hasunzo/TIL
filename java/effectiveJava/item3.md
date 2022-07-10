# 아이템 3 private 생성자나 열거 타입으로 싱글턴임을 보장하라

### 싱글턴 (singleton)

- 인스턴스를 오직 하나만 생성할 수 있는 클래스
- 함수와 같은 무상태 (stateless) 객체, 설계상 유일해야 하는 시스템 컴포넌트

### 클래스를 싱글턴으로 만들면 이를 사용하는 클라이언트를 테스트하기가 어려워질 수 있다

- 타입을 인스턴스로 정의 후 그 인터페이스를 구현해서 만든 싱글턴이 아닌경우
- 싱글턴 인스턴스를 가짜 구현으로 대체할 수 없기 때문
    - 생성자를 private 으로 감춰두었기 때문


### 싱글턴을 만드는 방식

- public static final 필드 방식의 싱글턴
- 정적 팩터리 방식의 싱글턴
- ⇒ 두 방식 모두 생성자는 private 으로 감춰두고 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 하나 마련해둔다

### public static final 필드 방식의 싱글턴

```java
public class Elvis {
	public static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }

	public void leaveTheBuilding() { ... }
}
```

- private 생성자는 public static final 필드인 Elvis.INSTANCE를 초기화할 때 딱 한 번만 호출된다
- public 이나 protected 생성자가 없으므로 전체 시스템에서 하나뿐임이 보장된다
- 권한이 있는 클라이언트는 리플렉션 API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다

### 정적 팩터리 방식의 싱글턴

```java
public class Elvis {
	private static final Elvis INSTANCE = new Elvis();
	private Elvis() { ... }
	public static Elvis getInstance() { return INSTANCE; }

	public void leaveTheBuilding() { ... }
}
```

- Elvis.getInstance 는 항상 같은 객체의 참조를 반환

### 두 방식의 장점

- public 필드 방식
    - 해당 클래스가 싱글턴임이 API에 명백히 드러남
    - public static 필드가 final 이니 절대로 다른 객체를 참조할 수 없다
    - 간결함
- 정적 팩터리 방식
    - API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점
    - 유일한 인스턴스를 반환하던 팩터리 메서드가 호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있음
    - 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다
    - 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다

### 싱글턴 클래스 직렬화

- Serializable 을 구현한다고 선언하는 것만으로는 부족
- 모든 인스턴스 필드를 일시적이라고 선언하고 readResolve 메서드를 제공해야 한다

```java
// 싱글턴임을 보장해주는 readResolve 메서드
private Object readResolve() {
	// 진짜 Elvis를 반환하고 가짜 Elvis는 가비지 컬렉터에 맡긴다.
	return INSTANCE;
}
```

### 열거 타입 방식의 싱글턴 - 바람직한 방법

```java
public enum Elvis {
	INSTANCE;

	public void leaveTheBuilding() { ... }
}
```

- public 필드 방식과 비슷하지만 더 간결하고 추가 노력 없이 직렬화가 가능
- 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아줌
- 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법
- 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다