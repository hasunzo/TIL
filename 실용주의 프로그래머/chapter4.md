# 4장. 실용주의 편집증

- 완벽한 소프트웨어는 만들 수 없다.
    - 그들은 자기 자신 역시 믿지 않는다.
    - 자신의 실수에 대비해 방어적으로 코드를 짠다.
- 계약에 따른 설계를 하라
- 일찍 작동을 멈추게 하라
- 단정문을 사용해서 불가능한 상황을 예방하라
- 예외는 예외적인 문제에 사용하라
- 시작한 것은 끝내라

### 계약에 의한 설계

- 첫 번째 방어법
- 계약에 의한 설계
    - 소프트웨어 모듈들의 권리와 책임을 문서화
    - 정확한 프로그램이란 스스로 자신이 하는 일이라고 주장하는 것만큼만 하는 프로그램
    - 이 주장을 문서화하고 검증하는 것이 계약에 의한 설계의 핵심
- 선행조건
    - 루틴이 호출되기 위해 참이어야 하는 것
    - 루틴의 요구사항
- 후행조건
    - 루틴이 자기가 할 것이라고 보장하는 것
- 클래스 불변식
    - 호출자의 입장에서 볼 떄는 이 조건이 언제나 참이라고 클래스 보장함
- DBC를 자체 지원하는 언어는 `선 후행 조건`을 컴파일러와 런타임 시스템에서 자동으로 검사
- 이 경우 모든 코드기반이 각자 계약을 엄수해야 하기 때문에 최고의 이득을 얻을 수 있음
- 문제를 찾고 원인을 밝히기 위해서는 사고가 났을 때 일찍 멈추는 것이 유리함
- 의미론적 불변식
    - 의미론적 불변식을 사용하면 일종의 ‘철학적 계약'인 불변의 요구사항을 표현할 수 있음
    - 신용카드 트랜잭션 스위치 예
        - 어떤 종류의 실패가 발생하든 간에, 에러가 있다면 트랜잭션을 처리하지 않아야지,
        - 트랜잭션을 중복 처리하면 안된다
    - 이런 간단한 법칙이 복잡한 에러 복구 시나리오를 해결하는 데에 큰 도움이 되는 것이 입증됨
    - 고정 불변의 법칙인 요구사항과 새로운 경영진에 의해 얼마든지 바뀔 수 있는 단순한 정책을 혼동하지 말아야 함
    - 불변식은 어떤 것의 바로 그 의미의 중심이 되어야 함
    - 일시적인 정책에 영향을 받으면 안됨
    - 자격이 있는 요구사항을 찾았다면 어떤 문서에든 잘 드러나도록 만들라

> 오류 발생시 소비자의 입장을 우선하라
>

### 죽은 프로그램은 거짓말을 하지 않는다

- 버그를 제거하는 중에 어떤 손상도 입히지 않도록 확실히 해야 한다
- 실용주의 프로그래머는 만약 에러가 있다면 정말로 뭔가 나쁜 일이 생신 것이라고 자신에게 이야기한다
- 망치지 말고 멈추라
    - 가능한 한 빨리 문제를 발견하게 되면, 좀 더 일찍 시스템을 멈출 수 있다는 이득이 있음
    - 방금 불가능한 뭔가가 발생했다는 것을 코드가 발견한다면 프로그램은 더 이상 유효하지 않다고 할 수 있음.
    - 이 시점 이후로 하는 일은 모두 수상쩍음
    - 되도록 빨리 종료해야 할 일

### 단정적 프로그래밍

- 이런 일은 절대 일어날 리 없어
    - 이런 류의 자기기만을 훈련하지 말자. 특히 코딩할 때는
    - 처음 단정 실패를 촉발케 한 잘못된 정보에 다른 코드가 의존하지 않도록 하라
- 확인을 쉽게 하는 방법
- 가정을 적극적으로 검증하는 코드
- 단정 기능을 켜두라
    - 단정문에 관한 한 가지 일반적인 오해
        - 단정은 코드에 과부하를 준다
        - 일단 코드가 테스트되고 선적된 다음에는 단정이 필요 없고,
        - 코드 실행히 빨라지도록 단정을 꺼버려야 한다
        - 단정은 디버깅 도구일 뿐이다.
    - 여기에 존재하는 공공연한 잘못된 가정
        - 테스트가 모든 버그를 발견한다는 가정
        - 낙관주의자들은 여러분의 프로그램이 험한 세상에서 돌아간다는 사실을 잊음
    - 퍼포먼스 문제가 있다 할지라도, 정말 문제가 되는 단정문만 끄도록 하다

### 언제 예외를 사용할까

- 예외는 적절히 사용하지 못하면 득보다 실이 더 많다
- 예외를 모두 처리하다보면 코드가 꽤 지저분해질 수 있다.
- 프로그램의 정상적 로직이 에러 처리에 완전히 묻혀 잘 보이지 않게 될 수 도 있다
- 무엇이 예외적인가
    - 예외에 문제가 있다면 하나는 이걸 언제 사용할지 아는 것
    - 코드가 어떤 파일을 읽으려고 할 때 파일이 존재하지 않는다면 예외가 발생해야 하나?
        - 경우에 따라 다르다
    - 예외를 정상적인 처리 과정의 일부로 사용하는 프로그램은 고전적인 스파게티 코드의 가독성 문제와 관리성 문제를 전부 떠안게 된다
        - 이런 프로그램은 캡슐화 역시 깨트린다
        - 예외 처리를 통해 루틴과 그 호출자들 사이의 결합도가 높아져 버린다
- 에러 처리기는 또 다른 대안이다
    - 에러 처리기는 에러가 감지되었을 때 호출되는 루틴이다

### 리소스 사용의 균형

- 곡예 중에 공을 하나도 떨어뜨리지 않는 방법
- 시작한 것을 끝내라
    - 리소스를 할당하는 루틴이나 객체가 리소스를 해제하는 책임 역시 져야한다는 걸 의미
- 객체와 예외
    - 할당과 해제의 균형은 클래스의 생성자와 소멸자를 생각나게 한다
    - 클래스는 하나의 리소스를 대표하며, 생성자는 그 리소스 타입의 특정 객체를 제공하고, 소멸자는 그것을 현 스코프에서 제거함
- 균형과 예외
    - 예외를 지원하는 언어는 리소스 해제에 복잡한 문제가 있을 수 있다
    - 예외가 던져진 경우, 그 예외 이전에 할당된 모든 것이 깨끗이 청소된다고 어떻게 보장?
- 자바에서 리소스 사용의 균형
    - C++과 달리 자바에서는 게으른 방식의 자동 객체 삭제를 사용
    - 참조가 없는 객체들은 가비지 콜렉션의 후보가 되며, 만약 가비지 콜렉션이 그 객체들을 지우려고 하기만 한다면 객체의 finalize 메서드가 호출될 것
    - 더 이상 대부분 메모리 누수 책임을 지지 않게 되어 개발자에게는 아주 편해진 일
    - but C++ 방식대로 자원을 청소하도록 구현하기는 어려워짐
    - 이것을 보상하기 위한 기능 → finally 절
        - finally 절 안의 코드들은 try 블록 안의 코드가 한 문장이라도 실행되면 반드시 실행되도록 되어 있다
        - 예외가 던져지더라도 상관없음
        - finally 절 안의 코드는 반드시 실행
- 리소스 사용의 균형을 잡을 수 없는 경우
    - 기본적인 리소스 할당 방식이 아예 적절하지 않은 경우도 있음
    - 보통 동적 자료 구조형을 사용하는 프로그램에서..
    - 한 루틴에서 메모리의 일정 영역을 할당한 후 다음 어떤 더 큰 구조에 그것을 연결 한동안 그대로 쓰이는 식
    - 여기에서 메모리 할당에 대한 의미론적인 불변식을 정하는 일이 필요해짐
    - 집합적인 자료구조 안의 자료에 대해 책임을 지는 게 누구인지 정해놓아야 한다
- 균형을 점검하기
    - 실용주의 프로그래머는 자신을 포함해서 아무도 믿지 않기 때문에 정말로 리소스가 적절하게 해제되었는지 실제로 점검하는 코드를 늘 작성하는 것이 언제나 좋다고 우리는 생각함