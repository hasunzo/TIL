# 아이템 8 finalizer와 cleaner 사용을 피하라

### 자바가 제공하는 두 가지 객체 소멸자

- finalizer
    - 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다
- cleaner
    - 자바 9에서는 finalizer를 사용 자제(deprecated) API로 지정하고 cleaner를 대안으로 소개했다
    - cleaner는 finalizer보다는 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다

### finalizer와 cleaner로는 제때 실행되어야 하는 작업은 절대 할 수 없다

- finalizer와 cleaner는 즉시 수행된다는 보장이 없다
- 파일 닫기를 finalizer나 cleaner에 맡기면 중대한 오류를 일으킬 수 있다
    - 시스템이 동시에 열 수 있는 파일 개수에 한계가 있기 때문
- 시스템이 finalizer나 cleaner 실행을 게을리해서 파일을 계속 열어 둔다면 새로운 파일을 열지 못해 프로그램이 실패한다
- finalizer나 cleaner를 얼마나 신속히 수행할지는 전적으로 가비지 컬렉터 알고리즘에 달렸다

### 상태를 영구적으로 수정하는 작업에서는 절대 finalizer나 cleaner에 의존해서는 안 된다

- 데이터베이스 같은 공유 자원의 영구 락(lock) 해제를 finalizer나 cleaner에 맡겨 놓으면 분산 시스템 전체가 서서히 멈출 것

### finalizer와 cleaner는 심각한 성능 문제도 동반한다

- 간단한 AutoCloeable 객체를 생성하고 가비지 컬렉터가 수거하기까지 12ns
- finalizer를 사용하면 550ns가 걸림
- finalizer가 가비지 컬렉터의 효율을 떨어뜨림

### finalizer를 사용한 클래스는 finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수도 있다

- finalizer 공격 원리
    - 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 된다
    - 이 finalizer는 정적 필드에 자신의 참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있다
    - 이렇게 일그러진 객체가 만들어지면 메서드를 호출해 애초에는 허용되지 않았을 작업을 수행하는 건 일도 아니다
    - 객체 생성을 막으려면 생성자에서 예외를 던지는 것만으로 충분하지만 finalizer가 있다면 그렇지도 않다

### final이 아닌 클래스를 finalizer 공격으로부터 방어하려면 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선언하자