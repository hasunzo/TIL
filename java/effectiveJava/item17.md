# 아이템 17. 변경 가능성을 최소화하라

### 불변 클래스

- 인스턴스의 내부 값을 수정할 수 없는 클래스
- 불변 인스턴스에 간직된 정보는 고정되어 객체가 파괴되는 순간까지 절대 달라지지 않는다
- 자바의 불변 클래스들
    - String
    - 기본 타입의 박싱된 클래스들
    - BigInteger
    - BigDecimal
- 불변 클래스는 가변 클래스보다 설계하고 구현하고 사용하기 쉬우며 오류가 생길 여지도 적고 훨씬 안전하다

### 클래스를 불변으로 만들기 위한 규칙

- 객체의 상태를 변경하는 메서드(변경자)를 제공하지 않는다
- 클래스를 확장할 수 없도록 한다
- 모든 필드를 final로 선언한다
- 모든 필드를 private으로 선언한다
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다

### 불변 복소수 클래스

```java
public final class Complex {
    private final double re;
    private final double im;

    public Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public double realPart() { return re; }

    public double imaginaryPart() {
        return im;
    }

    public Complex plus(Complex c) {
        return new Complex(re + c.re, im + c.im);
    }

    public Complex minus(Complex c) {
        return new Complex(re - c.re, im - c.im);
    }

    public Complex times(Complex c) {
        return new Complex(re * c.re - im * c.im,
                re * c.im + im * c.re);
    }

    public Complex dividedBy(Complex c) {
        double tmp = c.re * c.re + c.im * c.im;
        return new Complex((re * c.re + im * c.im) / tmp,
                (im * c.re - re * c.im) / tmp);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Complex complex = (Complex) o;
        return Double.compare(complex.re, re) == 0
                && Double.compare(complex.im, im) == 0;
    }

    @Override
    public int hashCode() {
        return 31 * Double.hashCode(re) + Double.hashCode(im);
    }

    @Override
    public String toString() {
        return "Complex{" +
                "re=" + re +
                ", im=" + im +
                '}';
    }
}
```

- 이 사칙연산 메서드들이 인스턴스 자신은 수정하지 않고 새로운 Complex 인스턴스를 만들어 반환하는 모습에 주목
    - 이처럼 피연산자에 함수를 적용해 그 결과를 반환하지만, 피연산자 자체는 그대로인 프로그래밍 페턴을 함수형 프로그래밍이라 한다
    - 이와 달리, 절차적 혹은 명령형 프로그래밍에서는 메서드에서 피연산자인 자신을 수정해 자신의 상태가 변하게 된다
- 메서드 이름으로 (add 같은) 동사 대신 (plus 같은) 전치사를 사용한 점에도 주목
    - 이는 해당 메서드가 객체의 값을 변경하지 않는다는 사실을 강조하려는 의도

### 불변 객체는 단순하다

- 불변 객체는 생성된 시점의 상태를 파괴될 때까지 그대로 간직한다
- 모든 생성자가 클래스 불변식을 보장한다면 그 클래스를 사용하는 프로그래머가 다른 노력을 들이지 않더라도 영원히 불변으로 남는다

### 불변 객체는 근본적으로 스레드 안전하여 따로 동기화할 필요 없다

- 불변 객체에 대해서는 어떤 스레드도 다른 스레드에 영향을 줄 수 없으니 불변 객체는 안심하고 공유할 수 있다

### 정적 팩터리 사용

- 불변 클래스는 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩터리를 제공할 수 있다
- 정적 팩터리를 사용하면 메모리 사용량과 가비지 컬렉션 비용이 줄어듬

### 복사 자체가 의미가 없다

- 아무리 복사해봐야 원본과 똑같으니 복사 자체가 의미가 없다
- 따라서 불변 클래스는 clone 메서드나 복사 생성자를 제공하지 않는 게 좋다
- String 클래스의 복사 생성자는 이 사실을 잘 이해하지 못한 자바 초창기 때 만들어진 것.
    - 되도록 사용하지 말자

### 불변 객체는 자유롭게 공유할 수 있음은 물론, 불변 객체끼리는 내부 데이터를 공유할 수 있다

- BigInteger 클래스 내부
    - 값의 부호(sign)
        - int 변수
    - 크기(magnitude)
        - int 배열
    - negate 메서드는 크기가 같고 부호만 반대인 새로운 BigInteger를 생성
    - 이때 배열은 비록 가변이지만 복사하지 않고 원본 인스턴스와 공유해도 된다
    - 그 결과 새로 만든 BigInteger 인스턴스도 원본 인스턴스가 가리키는 내부 배열을 그대로 가리킨다

### 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다

- 값이 바뀌지 않는 구성요소들로 이뤄진 객체라면 그 구조가 아무리 복잡하더라도 불변식을 유지하기 훨씬 수월하기 때문
- 불변 객체는 맵의 키와 집합(Set)의 원소로 쓰기에 안성맞춤이다
- 맵이나 집합은 안에 담긴 값이 바뀌면 불변식이 허물어지는데 불변 객체를 사용하면 그런 걱정은 하지 않아도 된다

### 불변 객체는 그 자체로 실패 원자성을 제공한다

- * 실패원자성 : 메서드에서 예외가 발생한 후에도 그 객체는 여전히 유효한 상태여야 한다는 성질
- 상태가 절대 변하지 않으니 잠깐이라도 불일치 상태에 빠질 가능성이 없다

### 불변 클래스에도 단점은 있다. 값이 다르면 반드시 독립된 객체로 만들어야 한다는 것이다

- 값의 가짓수가 많다면 이들을 모두 만드는 데 큰 비용을 치러야 한다
- 백만 비트짜리 BigInteger 에서 비트 하나를 바꿔야 한다고 했을 때

```java
BigInteger moby = ...;
moby = moby.flipBit(0); // 새로운 BigInteger 인스턴스를 생성하는 flipBit()
```

- flipBit 메서드는 새로운 BigInteger 인스턴스를 생성한다
    - 원본과 단지 한 비트만 다른 백만 비트짜리 인스턴스를 생성..
- 이 연산은 BigInteger의 크기에 비례해 시간과 공간을 잡아먹음
- BigSet도 BigInteger 처럼 임의 길이의 비트 순열을 표현하지만 BigInteger 와는 달리 가변이다
- BigSet 클래스는 원하는 비트 하나만 상수 시간 안에 바꿔주는 메서드를 제공한다

```java
BigSet moby = ...;
moby.flip(0);
```

- 원하는 객체를 완성하기까지의 단계가 많고, 그 중간 단계에서 만들어진 객체들이 모두 버려진다면 성능 문제가 더 불거진다
- 이 문제에 대처하는 방법 두가지
    - 다단계 연산(multistep operation)들을 예측하여 기본 기능으로 제공
        - 이를 제공하면 더 이상 각 단계마다 객체를 생성하지 않아도 된다
        - 불변 객체는 내부적으로 아주 영리한 방식으로 구현할 수 있기 때문
        - BigInteger는 모듈러 지수 같은 다단계 연산 속도를 높여주는 가변 동반 클래스를 package-private으로 두고 있다
        - 이 가변 동반 클래스를 사용하기란 BigInteger를 쓰는 것보다 훨씬 어려움
        - 그래도 우리는 운이 좋다. 그 어려운 부분을 모두 BigInteger 가 대신 처리해주니 말이다
    - 해당 클래스를 public 으로 제공
        - 클라이언트들이 원하는 복잡한 연산들을 정확히 예측할 수 있다면 package-private 의 가변 동반 클래스만으로 충분하나
        - 그렇지 않다면 public으로 제공하는게 최선
        - 대표적인 예가 String
        - String의 가변 동반 클래스는? StringBuilder

### 불변 클래스를 만드는 설계 방법

- 모든 생성자를 private 혹은 package-private 으로 만들고 public 정적 팩터리를 제공하는 방법

```java
public class Complex {
    private final double re;
    private final double im;

    private Complex(double re, double im) {
        this.re = re;
        this.im = im;
    }

    public static Complex valueOf(double re, double im) {
        return new Complex(re, im);
    }
}
```

- 바깥에서 볼 수 없는 package-private 구현 클래스를 원하는 만큼 만들어 활용할 수 있으니 훨씬 유연함
- 패키지 바깥의 클라이언트에서 바라본 이 불변 객체는 사실상 final
- public이나 protected 생성자가 없어서 다른 패키지에서는 이 클래스를 확장하는 게 불가능하기 때문이다
- 정적 팩터리 방식은 다수의 구현 클래스를 활용한 유연성을 제공하고 이에 더해 다음 릴리스에서 객체 캐싱 기능을 추가해 성능을 끌어올릴 수도 있다

### 성능을 위한 불변 클래스 규칙 완화

- 어떤 메서드도 객체의 상태 중 외부에 비치는 값을 변경할 수 없다
    - 어떤 불변 클래스는 계산 비용이 큰 값을 나중에 계산하여 final 이 아닌 필드에 캐시 해놓기도 한다
    - 똑같은 값을 다시 요청하면 캐시해둔 값을 반환하여 계산 비용을 절감하는 것
    - 몇 번을 계산해도 항상 같은 결과가 만들어짐이 보장되기 때문

### 정리

- getter 가 있다고 해서 무조건 setter 를 만들지 말자
- 클래스는 꼭 필요한 경우가 아니라면 불변이어야 한다
- 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자
- 다른 합당한 이유가 없다면 모든 필드는 private final 이어야 한다
- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다