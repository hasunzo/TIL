## Stack
- 스택이란 한 쪽 끝에서만 자료를 넣고 뺼 수 있는 LIFO(Last In First Out)형식의 자료구조를 뜻한다.
따라서 최근에 스택에 추가한 항목이 가장 먼저 제거될 항목이다.
- 스택은 재귀 알고리즘을 사용하는 경우 유용하다. 
  재귀 알고리즘은 재귀적으로 함수를 호출해야 하는 경우에 임시 데이터를 스택에 넣어주고 
  재귀함수를 빠져나와 퇴각 검색을 할 때는 스택에 넣어 두었던 임시 데이터를 빼주게 된다.
  스택은 이런 일련의 행위를 직관적으로 가능하게 해주며 재귀 알고리즘을 반복적 형태를 통해서 구현할 수 있게 해준다.
- 그 외에도 웹 브라우저 방문기록, 실행취소, 역순 문자열 만들기, 수식의 괄호 연산, 후위 표기법 계산에 유용하다.

```java
class Stack<T> {
    class Node<T> {
        private T data;
        private Node<T> next;
        
        public Node(T data) {
            this.data = data;
        }
    }
    
    private Node<T> top;
    
    public T pop() {
        if (top == null) {
            throw new ExptyStackException();
        }
        
        T item = top.data;
        top = top.next;
        return item;
    }
    
    public void push(T item) {
        Node<T> t = new Node<T>(item);
        t.next = top;
        top = t;
    }
    
    public T peek() {
        if (top == null) {
          throw new ExptyStackException();
        }
        return top.data;
    }
    
    public boolean isEmpty() {
        return top == null;
    }
}
```

## Queue
- 큐는 먼저 집어 넣은 데이터가 먼저 나오는 FIFO(First In First Out)형식의 자료구조를 뜻한다.
- 큐는 데이터가 입력된 순서대로 처리해야 할 필요가 있는 상황에 이용한다.
- 너비 우선 탐색(BFS, Breath-First Search) 구현의 경우 
  처리해야 할 노드의 리스트를 저장하는 용도로 큐를 사용하고 노드를 하나 처리할 때마다 해당 노드와 인접한
  노드들을 큐에 다시 저장하면 노드를 접근한 순서대로 처리할 수 있다.
- 그 외에도 캐시구현, 우선순위가 같은 작업 예약(인쇄 대기열), 선입선출이 필요한 대기열(티켓 카운터), 콜센터 고객 대기시간,
프린터의 출력 처리, 윈도우 시스템의 메시지 처리기, 프로세스 관리에 유용하다.
  
```java
class Queue<T> {
    class Node<T> {
        private T data;
        private Node<T> next;
        
        public Node(T data) {
            this.data = data;
        }
    }
    
    private Node<T> first;
    private Node<T> last;
    
    public void add(T item) {
        Node<T> t = new Node<T>(item);
        
        if (last != null) {
            last.next = t;
        }
        
        last = t;
        if (first == null) {
            first = last;
        }
    }
    
    public T remove() {
        if(first == null) {
            throw new NoSuchElementException();
        }
        
        T data = first.data;
        first = first.next;
        
        if (first == null) {
            last = null;
        }
        
        return data;
    }
    
    public T peek() {
        if (first == null) {
          throw new NoSuchElementException();
        }
        return first.data;
    }
    
    public boolean isEmpty() {
        return first == null;
    }
}
```