## Hash Table
- Hash Table 이란 키에 값을 저장하는 데이터 구조이며 키를 통해 값을 조회, 삭제, 수정, 삽입이 가능하다.
해시 테이블에 키가 들어오면 Hash Function을 통해 해시로 변경 후 저장된다.
이 후 해시를 배열의 인덱스로 사용하여 해당 인덱스에 값을 저장한다.
따라서 많은 양의 데이터를 빠르게 검색할 수 있고 저장공간의 효율성이 높아진다.

## Hash Map
- Hash Map 또한 키에 대한 해시 값을 사용하여 값을 저장하고 조회한다.

### 해시 충돌
- 이러한 해싱의 과정에서 해쉬코드의 값이 동일한 경우가 생길 수 있는데 이때 해쉬 충돌이 일어난다.
- 해시 충돌을 해결하기 위한 방식으로는 대표적으로 Separate Chaining과 Open Addressing이 있다. 
  Open Addressing은 데이터를 삽입하려는 해시 버킷이 이미 사용 중인 경우 다른 해시 버킷에 해당 데이터를 삽입하는 방식이며
  Separate Chaining은 각 인덱스에 데이터를 저장하는 Linked List에 대한 포인터를 가지는 방식이다.
- Open Addressing은 연속된 공간에 데이터를 저장하기 때문에 Sperate Chaining 방식에 비해 메모리를 덜 사용하여 캐시 효율이 높다. 
  하지만 배열의 크기가 커질수록 캐시 적중률이 낮아짐에 따라 캐시 효율이라는 Open Addressing의 장점이 사라진다.
- JDK 내부에서는 Separate Channing 방식을 사용한다.

## Hash Table과 Hash Map
- Hash Table과 Hash Map 모두 Map 인터페이스를 구현하고 있기 때문에 제공하는 기능은 같다. 따라서 성능, 기능 비교에 큰 의미가 없다.
다만 Hash Map은 보조 해시 함수를 사용하여 해시 테이블에 비해 해시 충돌이 덜 발생할 수 있어 상대적으로 성능상 이점이 있다.
- Java8 HashMap 보조 해시 함수는 상위 16비트 값을 XOR 연산하는 형태의 보조 해시 함수를 사용한다.
  Java 7에 비해 단순한 형태로 바뀌었는데 이는 Java 8에서 해시 충돌이 많이 발생하게 되면 Linked List 대신 Tree를 사용하므로
  성능 문제가 완화되었고 Java 7까지 사용했던 보조 해시 함수의 효과가 크지 않았기 때문이다.

- [https://bcho.tistory.com/1072](https://bcho.tistory.com/1072)
- [https://d2.naver.com/helloworld/831311](https://d2.naver.com/helloworld/831311)

