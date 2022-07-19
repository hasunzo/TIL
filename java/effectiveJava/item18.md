# 아이템 18. 상속보다는 컴포지션을 사용하라

### 상속의 단점

- 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다
    - 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다
    - 상위 클래스는 릴리스마다 내부 구현이 달라질 수 있으며 그 여파로 코드 한 줄 건드리지 않은 하위 클래스가 오동작할 수 있다는 말
    - 상위 클래스 설계자가 확장을 충분히 고려하고 문서화도 제대로 해두지 않으면 하위 클래스는 상위 클래스의 변화에 발맞춰 수정돼야만 한다

### 상속을 잘못 사용한 예시

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
    // 추가된 원소의 수
    private int addCount = 0;
    
    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    
    public int getAddCount() {
        return addCount;
    }
}
```

- addAll 메서드는 제대로 작동하지 않는다
- addAll 메서드로 원소 3개를 더한 후 addCount 의 값을 확인해보면
- 3을 예상했지만 6이란 결과값이 나옴
- 왜 그럴까?
    - HashSet의 addAll 메서드가 add 메서드를 사용해 구현되었기 때문
- 이처럼 자신의 다른 부분을 사용하는 자기사용 여부는 해당 클래스의 내부 구현 방식에 해당하며
- 자바 플랫폼 전반적인 정책인지, 그래서 다음 릴리스에서도 유지될지는 알 수 없다

### 재정의가 문제다!

- 새로운 메서드를 추가하면 괜찮지 않을까?
    - 안전한 것은 맞다! 다만,
    - 하위 클래스에 추가한 메서드와 시그니처가 같고 반환 타입은 다르다면?
        - 컴파일 조차 되지 않음
    - 반환 타입 마저 같으면?
        - 재정의한 꼴이니 앞서 야기된 문제가 다시 발생

### 컴포지션(composition)을 사용하자

- 기존 클래스를 확장하는 대신, 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조
- 기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 뜻에서 컴포지션이라 한다

```java
// 래퍼 클래스 - 상속 대신 컴포지션을 사용했다.
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;
    
    public InstrumentedSet(Set<E> s) {
        super(s);
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }
    
    public int getAddCount() {
        return addCount;
    }
}
```

```java
// 재사용할 수 있는 전달 클래스
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear() {
        s.clear();
    }

    public boolean contains(Object o) {
        return s.contains(o);
    }

    public boolean isEmpty() {
        return s.isEmpty();
    }

    public int size() {
        return s.size();
    }

    public Iterator<E> iterator() {
        return s.iterator();
    }

    public boolean add(E e) {
        return s.add(e);
    }

    public boolean remove(Object o) {
        return s.remove(o);
    }

    public boolean containsAll(Collection<?> c) {
        return s.containsAll(c);
    }

    public boolean addAll(Collection<? extends E> c) {
        return s.addAll(c);
    }

    public boolean removeAll(Collection<?> c) {
        return s.removeAll(c);
    }

    public boolean retainAll(Collection<?> c) {
        return s.retainAll(c);
    }

    public Object[] toArray() {
        return s.toArray();
    }

    public <T> T[] toArray(T[] a) {
        return s.toArray(a);
    }

    @Override
    public boolean equals(Object obj) {
        return super.equals(obj);
    }

    @Override
    public int hashCode() {
        return s.hashCode();
    }

    @Override
    public String toString() {
        return s.toString();
    }
}
```

- InstrumentedSet은 HashSet의 모든 기능을 정의한 Set 인터페이스를 활용해 설계되어 견고하고 아주 유연하다
- 구체적으로는 Set 인터페이스를 구현했고 Set의 인스턴스를 인수로 받는 생성자를 하나 제공한다
- 임의의 Set에 계층 기능을 덧씌워 새로운 Set으로 만드는 것이 이 클래스의 핵심이다
- 상속 방식은 구체 클래스 각각을 따로 확장해야 하며 지원하고 싶은 상위 클래스의 생성자에 각각 대응하는 생성자를 별도로 정의해줘야 한다
- 하지만 지금 선보인 컴포지션 방식은 한 번만 구현해두면 어떠한 Set 구현체라도 계측할 수 있으며 기존 생성자들과도 함께 사용할 수 있다

```java
Set<Instant> times = new InstrumentedSet<>(new TreeSet<>(cmp));
Set<E> s = new InstrumentedSet<>(new HashSet<>(INIT_CAPACITY));
```

- InstrumentedSet을 이용하면 대상 Set 인스턴스를 특정 조건하에서만 임시로 계측할 수 있다

```java
static void walk(Set<Dog> dogs) {
	InstrumentedSet<Dog> iDogs = new InstrumentedSet<>(dogs);
	... // 이 메서드에서는 dogs 대신 iDogs를 사용한다.
}
```

- 다른 Set 인스턴스를 감싸고 있다는 뜻에서 InstrumentedSet 같은 클래스를 래퍼 클래스라 하며
- 다른 Set 에 계층 기능을 덧씌운다는 뜻에서 데코레이터 패턴이라고 한다
- 컴포지션과 전달의 조합은 럽은 의미로 위임이라고 부른다
- 단, 엄밀히 따지면 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 위임에 해당한다

### 래퍼 클래스는 콜백 프레임워크와는 어울리지 않는다

- 콜백 프레임워크에서는 자기 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백) 때 사용하도록 한다
- 내부 객체는 자신을 감싸도 있는 래퍼의 존재를 모르니 대신 자신(this)의 참조를 넘기고 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 된다
- 이를 SELF 문제라고 한다
- 전달 메서드가 성능에 주는 영향이나 래퍼 객체가 메로리 사용량에 주는 영향을 걱정하는 사람도 있지만 실전에서는 둘 다 별다른 영향이 없다고 밝혀졌다
- 재사용할 수 있는 전달 클래스를 인터페이스당 하나씩만 만들어두면 원하는 기능을 덧씌우는 전달 클래스들을 아주 손쉽게 구현할 수 있다

### 상속은 반드시 하위 클래스가 상위 클래스의 ‘진짜' 하위 타입인 상황에서만 쓰여야 한다

- 클래스 B가 클래스 A와 is-a 관계일 때만 클래스 A를 상속해야 한다