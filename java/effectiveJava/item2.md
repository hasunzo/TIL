### 생성자 혹은 정적 팩터리의 매개변수가 많아지면 생기는 불편함

- 대응하기가 어렵다
- 점층적 생성자 패턴은 필수 매개변수만 받는 생성자, 거기에 매개변수 1개를 받는 생성자… 2개를.. 3개를.. 전부 다 받는.. 으로 늘려가는 패턴인데. 이 경우 매개변수가 많아지면 클라이언트 코드를 작성하거나 읽기가 어려워진다
    - 값의 의미가 무엇이며, 개수를 세어야 한다. 여기에 타입까지 같다면..? 오류를 찾기 힘든 버그로 이어짐
    - 매개변수의 순서가 바뀌어도 알아차리기 쉽지않다

```java
Apple apple = new Apple();
apple.setName("redApple");
apple.setColor("red");
apple.setWeight(40);
```

- 자바빈즈 패턴은 매개변수가 없는 생성자로 객체를 만들고 setter 메서드를 호출해 값을 설정하는 것이다
    - 자바빈즈 패턴에서는 객체 하나를 만들려면 메서드를 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓인다
    - 일관성이 깨지면 버그를 심은 코드와 그 버그 때문에 런타임에 문제를 겪는 코드가 물리적으로 멀리 떨어져 있을 것이므로 디버깅도 만만치 않음
    - 자바빈즈 패턴에서는 클래스를 불변으로 만들 수 없으며 스레드 안전성을 얻으려면 추가 작업이 필요하다

### 결론은 빌더 패턴(Builder pattern)

- 클라이언트는 필요한 객체를 직접 만드는 대신 필수 매개변수만으로 생성자를 호출해 빌더 객체를 얻는다
- 그런 다음 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정한다
- 마지막으로 매개변수가 없는 build 메서드를 호출해 필요한 객체를 얻는다

```java
public class Apple {
    private final String name;
    private final String color;
    private final int weight;
    private final String flavor;
    private final int price;
    
    public static class Builder {
        // 필수 매개변수
        private final String name;
        private final String color;
        
        // 선택 매개변수
        private int weight;
        private String flavor;
        private int price;

        public Builder(String name, String color) {
            this.name = name;
            this.color = color;
        }

        public Builder weight(int val) {
            weight = val;
            return this;
        }

        public Builder flavor(String val) {
            flavor = val;
            return this;
        }

        public Builder price(int val) {
            price = val;
            return this;
        }

        public Apple build() {
            return new Apple(this);
        }
    }

    public Apple(Builder builder) {
        this.name = builder.name;
        this.color = builder.color;
        this.weight = builder.weight;
        this.flavor = builder.flavor;
        this.price = builder.price;
    }
}
```

- Apple 클래스는 불변, 모든 매개변수의 기본값들을 한곳에 모아두었다
- 빌더의 세터 메서드들은 빌더 자신 반환 → 연쇄적으로 호출할 수 있다
- 이런 방식을 플루언트API 혹은 메서드 연쇄(method chaining)라고 한다

```java
Apple apple = new Apple.Builder("prettyApple", "red")
		.weight(100).flavor("달콤").price(1000).build();
```

- 쓰기 쉽다
- 읽기 쉽다

### 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다

- 각 계층의 클래스에 관련 빌더를 멤버로 정의
- 추상 클래스는 추상 빌더를, 구체 클래스(concrete class)는 구체 빌더를 갖게 한다
- 피자의 다양한 종류를 표현하는 계층구조의 루트에 놓은 추상 클래스

```java
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;
    
    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);
        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();

        // 하위 클래스는 이 메서드를 재정의 (overriding) 하여
        // "this"를 반환하도록 해야 한다
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone();
    }
}
```

- Pizza.Builder 클래스는 재귀적 타입 한정을 이용하는 제네릭 타입이다
- 여기에 추상 메서드 self를 더해 하위 클래스에서는 형번환없이 메서드 연쇄를 지원할 수 있다
    - self 타입이 없는 자바를 위한 이러한 우회 방법을 `시뮬레이트한 셀프 타입 관용구`라 한다

```java
// 뉴욕 피자
// size 매개변수가 필수
public class NyPizza extends Pizza {
    public enum Size {SMALL, MEDIUM, LARGE}

    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;

        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}
```

```java
//칼초네 피자
// 소스를 넣을지 선택하는 매개변수 필수
public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // 기본값

        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override
        public Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() {
            return this;
        }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```

- 각 하위 클래스의 빌더가 정의한 build 메서드는 해당하는 구체 하위 클래스를 반환하도록 선언한다
    - NyPizza.Builder ⇒  NyPizza 반환
    - Calzone.Builder ⇒ Calzone 반환
- 공변 반환 타이핑 (covariant return typing)
    - 하위 클래스의 메서드가 상위 클래스의 메서드가 정의한 반환 타입이 아닌
    - 그 하위 타입을 반환하는 기능
- 공변 반환 타이핑을 이용하면 클라이언트가 형변환에 신경 쓰지 않고도 빌더를 사용할 수 있다

```java
NyPizza pizza = new NyPizza.Builder(SMALL)
	.addTopping(SAUSAGE).addTopping(ONION).build();

Calzone calzone = new Calzone.Builder()
	.addTopping(HAM).sauceInside().build();
```

- 생성자로는 누릴 수 없는 사소한 이점으로, 빌더를 이용하면 가변인수 매개변수를 여러 개 사용할 수 있다
- 메서드를 여러 번 호출하도록 하고 호출 때 넘겨진 매개변수들을 하나의 필드로 모을 수도 있다

### 빌더 패턴의 단점

- 객체를 만들려면 빌더부터 만들어야 한다
- 빌더 생성 비용이 크진 않지만 성능에 민감한 상황에서는 문제가 될 수 있다
- 매개변수가 4개 이상은 되어야 값어치를 한다
- but, API는 시간이 지날수록 매개변수가 많아지니 빌더로 시작하는 편이 나을 때가 많다

### 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는 게 더 낫다