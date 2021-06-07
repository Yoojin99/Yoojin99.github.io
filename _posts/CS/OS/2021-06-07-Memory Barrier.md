---  
layout: post  
title: "Memory Barrier, 메모리 배리어란?"  
subtitle: ""  
categories: cs
tags: cs-os, memory, barrier
comments: true  
header-img: 

---  
  
> `Memory Barrier가 무엇인지 알아보자.`  

---

## Memory barrier

메모리 베리어는 membar, memory fence, fence instruction으로 알려져 있다. 메모리 배리어는 CPU나 컴파일러에게 barrier 명령문 전 후의 메모리 연산을 순서에 맞게 실행하도록 강제하는 기능이다.
즉 barrier 이전에 나온 연산들이 barrier 이후에 나온 연산보다 먼저 실행이 되는게 보장되어야 하는 것이다.

Memory barrier는 현대 CPU가 성능을 좋게 하기 위해 최적화를 거쳐 순서에 맞지 않게 실행시키는 결과를 초래할 수 있기 때문에 필수적이다. 이런 메모리 연산(load/store)의 재배치는 single 쓰레드로 실행할 때는 알아차리기 어렵지만,
제대로 관리되지 않는 한 병행적으로 작동되는 프로그램과 디바이스 드라이버에서 예상할 수 없는 결과를 초래하기도 한다. 싱글 쓰레드에서는 하드웨어가 명령어 통합성을 강제하기 때문에 싱글 쓰레드에서는 문제가 되지 않는다.

즉 정리하자면 성능을 좋게 만드려고 코드의 순서를 바꿔서 실행시킬 수 있는데, 이런 것을 막고 순서대로 코드가 실행되게 하도록 강제하는 것이다. 아래 간략화 된 코드를 보자. 실제로 동작하는 코드는 아니고, 핵심적인 부분만
남겨놓은 코드다.

```java
static void Thread1() {
  y = 1; // Store y
  r1 = x; // Load x
}

static void Thread2() {
  x = 1; // Store x
  r2 = y; // Load y
}

static void Main(string[] args) {
  while(true) {
    x = y = r1 = r2 = 0; // 반복문의 초기부분마다 모든 변수의 값을 0으로 할당한다.
    
    Task t1 = new Task(Thread1);
    Task t2 = new Task(Thread2);
    
    t1.Start();
    t2.Start();
    
    Task.WaitAll(t1, t2);
    
    if (r1 == 0 & r2 == 0) 
      break;
  }
  System.out.println("While문이 break되었다.");
}
```

메모리 연산인 store과 load를 하는 Thread1, Thread2가 있고, 이를 Main 함수 안에서 실행시킨다. 위의 동작을 봤을 때 우리가 기대하는 것은 초기에 0으로 설정되어 있는 x, y 값이 1로 바뀌고 r1, r2에 이어서
1이 할당되면서 while 문이 계속 반복되어 무한 루프를 도는 것이다. 하지만 실제로 실행해 보면(위의 코드가 실행되는 건 아니다) `While문이 break되었다.` 로그가 찍히는 경우가 생긴다.

어떻게 이런 일이 가능한 것일까? 분명 while문을 빠져 나오는 조건은 r1과 r2가 모두 0일 때였다. 이 반복문을 빠져나왔다는 것은 원래 아래의 순서대로 실행되어야 할 코드가 

```
x = 0 // 반복문 안
y = 0

x = 1 // 쓰레드 안
y = 1

r1 = x // 쓰레드 안
r2 = y
```

실제로는 아래와 같이 실행되는 것이다.

```
x = 0 // 반복문 안
y = 0

r1 = x // 쓰레드 안
r2 = y

// 여기에서 반복문 빠져나옴!

x = 1 // 쓰레드 안
y = 1
```

성능의 최적화를 위해서 코드의 순서를 이렇게 뒤집어서 실행시킬 수 있는데, 싱글 쓰레드에서는 문제가 되지 않지만 멀티 쓰레드에서는 문제가 된다. 따라서 이를 방지하기 위해, 명령어를 순서대로 실행시키기 위해
메모리 배리어를 사용한다. 

```java
static void Thread1() {
  y = 1; // Store y
  
  Thread.MemoryBarrier(); // 메모리 배리어
  
  r1 = x; // Load x
}

static void Thread2() {
  x = 1; // Store x
  
  Thread.MemoryBarrier(); // 메모리 배리어
  
  r2 = y; // Load y
}

static void Main(string[] args) {
  while(true) {
    x = y = r1 = r2 = 0; // 반복문의 초기부분마다 모든 변수의 값을 0으로 할당한다.
    
    Task t1 = new Task(Thread1);
    Task t2 = new Task(Thread2);
    
    t1.Start();
    t2.Start();
    
    Task.WaitAll(t1, t2);
    
    if (r1 == 0 & r2 == 0) 
      break;
  }
  System.out.println("While문이 break되었다.");
}
```

위의 코드와 같이 메모리 배리어를 설정하면 배리어 위의 코드가 배리어 아래의 코드와 순서가 바뀌어서 실행될 수 없게 된다. 이렇게 코드의 실행 순서를 보장해 주는 것이 메모리 배리어의 역할이다. 

### 종류

메모리 배리어는 메모리 read/write가 내가 예상한 대로 실행되게 하는 명령어의 클래스다. 예를 들어 full fence(barrier)는 fence 전의 모든 read/write
는 fence 이후의 read/write 을 실행시키기 전에 모두 실행되어야 한다는 것이다. 메모리 배리어의 종류는 아래와 같다.

1. Full Memory Barrier : read/write 를 둘 다 막는다.
2. Store Memory Barrier : write만 막는다.
3. Load Memory Barrier : read만 막는다.

메모리 배리어는 하드웨어 개념임을 기억해야 한다. 높은 레벨의 언어에서는 mutex와 세마포어를 이용해서 이를 해결한다. 낮은 레벨에서 메모리 배리어를 사용하고 메모리 배리어를 명시적으로 사용하는 것을 구현하는 것은 불필요할 수 있다.
메모리 배리어의 사용은 하드웨어 구조를 자세히 공부한 것이 전제가 되어야 하고, 애플리케이션 코드보다는 디바이스 드라이버에서 많이 사용된다.

참고

* https://ko.wikipedia.org/wiki/%EB%A9%94%EB%AA%A8%EB%A6%AC_%EB%B0%B0%EB%A6%AC%EC%96%B4
* https://en.wikipedia.org/wiki/Memory_barrier
* https://gameproyyj.tistory.com/251?category=878138
* https://stackoverflow.com/questions/286629/what-is-a-memory-fence
