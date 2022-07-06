# [아이템 1] 생성자 대신 정적 팩터리 메서드를 고려하라

```java
public static Boolean valuOf(boolean b) {
	return b ? Boolean.TRUE : Boolean.FALSE;
}
```

- 해당 코드는 boolean 기본 타입의 박싱 클래스인 Boolean 에서 발췌한 간단한 예.
- 기본 타입인 boolean 값을 받아 Boolean 객체 참조로 변환해 준다
- 클래스는 클라이언트에 public 생성자 대신 정적 팩토리 메서드를 제공할 수 있다

### 정적 팩터리 메서드가 생성자보다 좋은 장점 다섯 가지

### 1. 이름을 가질 수 있다

- 하나의 시그니처로는 하나의 생성자만 만들 수 있고 매개변수들을 다르게 한다고 해도 생성자가 어떤 역할을 하는지 쉽게 알 수 없다.
- 시그니처가 같은 생성자가 여러 개 필요할 것 같으면 정적 팩터리 메서드로 바꾸고 차이점이 잘 드러나는 이름을 지어주자

### 2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다

- 불필요한 객체 생성을 막아 인스턴스를 통제할 수 있게 되면 싱글턴으로 만들 수도 있고 인스턴스화 불가로 만들 수도 있다
- 불변 값 클래스에서 동치인 인스턴스가 단 하나뿐임을 보장할 수 있다

### 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다

- 반환할 객체의 클래스를 자유롭게 선택할 수 있게 하는 ‘엄청난 유연성’
- 이 유연성을 응용하면 API를 만들 때 구현 클래스를 공개하지 않고도 객체를 반환할 수 있어 API를 작게 유지할 수 있다
- 자바 컬렉션 프레임워크
    - 핵심 인터페이스들에 수정 불가나 동기화 등의 기능을 덧붙인 총 45개의 유틸리티 구현체 제공
    - 이 구현체 대부분을 단 하나의 인스턴스화 불가 클래스인 java.util.Collections에서 정적 팩터리 메서드를 통해 얻도록 했다

### 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다

- 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관없다.
- EnumSet 클래스의 예시
    - 원소가 64개 이하면 원소들을 long 변수 하나로 관리하는 RegularEnumSet의 인스턴스
    - 원소가 65개 이상이면 long 배열로 관리하는 JumboEnumSet의 인스턴스
- 클라이언트는 하위 타입 클래스의 존재를 모른다
- 클라이언트는 팩터리가 건네주는 객체가 어느 클래스의 인스턴스인지 알 수 없고 알 필요도 없다

### 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다

- 대표적인 서비스 제공자 프레임워크인 JDBC
- 서비스 제공자 프레임워크에서의 제공자는 서비스의 구현체다
- 이 구현체들을 클라이언트에 제공하는 역할을 프레임워크가 통제하여 클라이언트를 구현체로부터 분리해준다
- 서비스 제공자 프레임워크의 3가지 핵심 컴포넌트
    - 서비스 인터페이스 (service interface)
        - 구현체의 동작을 정의
    - 제공자 등록 API(provider registration API)
        - 제공자가 구현체를 등록할 때 사용
    - 서비스 접근 API(service access API)
        - 클라이언트가 서비스의 인스턴스를 얻을 때 사용
        - ‘유연한 정적 팩터리’의 실체

### 정적 팩터리 메서드의 단점

### 1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다

- 컬렉션 프레임워크의 유틸리티 구현 클래스들은 상속할 수 없다는 이야기
- 이 제약은 상속보다 컴포지션을 사용하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 장점으로 받아들일 수 있음

### 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다

- 생성자처럼 API 설명에 명확히 드러나지 않음 → 사용자가 정적 팩터리 메서드 방식 클래스를 인스턴스화할 방법을 알아내야 함

### 정적 팩터리 메서드에서 사용하는 명명 방식들

- from
    - 매개변수를 하나 받아서 해당 타입의 인스턴스 반환 형변환 메서드
    - `Date d = Date.from(instant);`
- of
    - 여러 매개변수를 받아 적합한 타입의 인스턴스 반환하는 집계 메서드
    - `Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);`
- valueOf
    - from과 of의 더 자세한 버전
    - `BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);`
- instance 혹은 getInstance
    - (매개변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만, 같은 인스턴스임을 보장하진 않음
    - `StackWalker luke = StackWalker.getInstance(options);`
- create 혹은 newInstance
    - instance 혹은 getInstance와 같지만, 매번 새로운 인스턴스를 생성해 반환함을 보장함
    - `Object newArray = Array.newInstance(classObject, arrayLen);`
- getType
    - getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 씀
    - “Type”은 팩터리 메서드가 반환할 객체의 타입
    - `FileStore fs = Files.getFileStore(path)`
- newType
    - newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 씀
    - “Type”은 팩터리 메서드가 반환할 객체의 타입
    - `BufferedReader br = Files.newBufferedReader(path);`
- type
    - getType과 newType의 간결한 버전
    - `List<Complaint> litany = Collections.list(legacyLitany);`