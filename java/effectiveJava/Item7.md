# 아이템 7 다 쓴 객체 참조를 해제하라

- 자바는 가비지 컬렉터를 갖춘 언어이기 때문에 메모리를 직접 관리해야 하는 C, C++ 과 같은 엉어에 비해 프로그래머의 삶이 훨씬 평안해진다
- 그러나 메모리 관리에 더 이상 신경 쓰지 않아도 되는 것은 아니다

### 메모리 누수가 일어나는 위치는 어디인가?

```java
public class Stack() {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object o) {
        ensureCapacity();
        elements[size++] = e;
    }
    
    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        return elements[--size];
    }

    /**
     * 원소를 위한 공간을 적어도 하나 이상 확보한다.
     * 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다.
     */
    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    } 
}
```

- 이 스택을 사용하는 프로그램을 오래 실행하다 보면 점차 가비지 컬렉션 활동과 메모리 사용량이 늘어나 결국 성능이 저하된다
- 심한 경우 디스크 페이징이나 OutOfMemoryError를 일으켜 프로그램이 종료되기도 한다
- 메모리 누수는 어디서 일어날까?
    - 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다
    - 이 스택이 그 객체들의 다 쓴 참조를 여전히 가지고 있기 때문이다
        - 다 쓴 참조란 앞으로 다시 쓰지 않을 참조를 뜻한다
        - elements 배열의 ‘활성 영역'밖의 참조들이 여기 해당
        - 활성 영역은 인덱스가 size보다 작은 원소들로 구성
- 가비지 컬렉션 언어에서는 (의도치 않게 객체를 살려두는) 메모리 누수를 찾기가 아주 까다롭다
- 객체 참조 하나를 살려두면 가비지 컬렉터는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체를 회수해가지 못한다

### 해결 방법은?

- 해당 참조를 다 썼을 때 null 처리(참조 해제)하면 된다
- 예시의 스택 클래스에서는 각 원소의 참조가 더 이상 필요 없어지는 시점은 스택에서 꺼내질 때다

```java
// 제대로 구현한 pop 메서드
public Object pop() {
	if (size == 0) {
		throw new EmptyStackException();
	}
	Object result = elements[--size];
	elements[size] = null; // 다 쓴 참조 해제
	return result;
}
```

- 만약 null 처리한 참조를 실수로 사용하려 하면 프로그램은 즉시 NullPointerException 던지며 종료됨
- 프로그램 오류는 가능한 한 조기에 발견하는 게 좋다

### 모든 객체를 일일이 null 처리하지는 않아도 된다

- 객체 참조를 null 처리하는 일은 예외적인 경우여야 한다
- 자기 메모리를 직접 관리하는 클래스일때 null 처리를 해주자
- 위 예시는 배열 안에 속한 원소들은 사용되지만 속하지 못한 배열은 쓰이지 않는다
- 이를 가비지 컬렉터는 알 길이 없기 때문에 메모리 누수가 일어나는 것
- 따라서 프로그래머는 비활성 영역이 되는 순간 null 처리를 해서 더는 쓰지 않을 것임을 알리자

### 캐시 역시 메모리 누수를 일으키는 주범이다

- 객체 참조를 캐시에 넣고 나서 이 사실을 까맣게 잊은 채 그 객체를 다 쓴 뒤로도 한참을 그냥 놔두는 일,,
- 운 좋게 캐시 외부에서 키를 참조하는 동안만 엔트리가 살아 있는 캐시가 필요한 상황이라면 WeakHashMap을 사용해 캐시를 만들자
- 다 쓴 엔트리는 그 즉시 자동으로 제거될 것이다
- 단, WeakHashMap은 이러한 상황에서만 유용하다

### 리스너 혹은 콜백

- 메모리 누수의 세 번째 주범
- 클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면 뭔가 조치해주지 않는 한 콜백은 계속 쌓여감
- 이럴 때 콜백을 약한 참조로 저장하면 가비지 컬렉터가 즉시 수거해간다