---  
layout: post  
title: "Swift - Concurrency by Tutorials"  
subtitle: ""  
categories: app
tags: app-swift concurrency
comments: true  
header-img: 
---  
  
> `Concurrency를 통해 앱의 성능을 높이는 방법과 개념에 대해 알아보자.`  

---

이 글은 "Concurrency by Tutorials[링크](https://www.raywenderlich.com/books/concurrency-by-tutorials/v2.0)"라는 책을 읽고 중요한 개념을 정리한 것이다. 
내가 스스로 정리하면 좋을 것 같은 개념적인 부분들만 추려서 올릴 것이고, 책에 포함된 구체적인 실습에 대해서는 작성하지 않을 것이다.

## 1. Introduction

앱의 기능, UI도 중요하지만 가장 중요한 것들 중 하나는 단연 **성능**과 **응답성**일 것이다. 성능과 응답성이 좋을 때는 아무도 
그것에 대해 신경쓰지 않지만, 성능과 응답성이 안 좋아질 때는 사람들이 그 앱을 사용하지 않을 가능성이 크다. 앱이든 뭐든 모든 프로그램에서
성능의 중요성이 크다는 사실을 부인하기는 어려울 것이다.

### Concurrency란?

Concurrency는 앱의 로직에서 특정 부분들이 랜덤한 순서로 같은 시간에 동작해도 데이터 플로우 상 올바르게 동작할 수 있는 가능성을 의미한다. 한마디로
병행성이다.

현대 기기들은 거의 대부분 하나 이상의 CPU를 갖고, 아이폰의 경우 2011년 부터 듀얼 코어였다. 하나 이상의 코어, CPU를 가진다는 것은 같은 시간에 하나 이상의
작업을 처리할 수 있다는 의미다. 이 '작업' 단위로 앱을 논리적인 코드의 '뭉치'로 분리하는 것은 iOS 기기가 이런 나눠진 작업들을 동시에 실행해서 전체적인
성능을 향상시킬 수 있게 하는 것을 가능하게 한다.

### 병행성을 이용해야 하는 이유는?

사용자가 앱에 무엇을 요청해서 이를 기다리게 하지 않게 보장하는 것은 중요하다. 예를 들어 이미지로 이루어진 테이블을 스크롤할 때 버벅거리는 경우, 사용자는 이런 
성능 상에 문제가 있다고 생각하게 될 것이다. 이렇게 렉이 걸릴 때 표시할 이미지 대신에 '작업중' 혹은 '처리중'이라는 표시를 띄워야 할 것이다.

병행성을 사용하면 앱의 전반적인 구조를 더 생각할 수 있게 된다. 단순히 큰 메서드를 작성해서 작업을 처리하기 보다는 더 작은 단위로 나눠 관리하게 좋게 메서드를 
작성할 수 있다.

### 병행성을 어떻게 이용해야 하는가?

이 글의 핵심이 될 것이다. 몇 개의 작업을 같은 시간에 실행하기 위해 앱의 구조를 구상해야 한다. 같은 자원을 수정하는 작업은 thread safe하게 만들지 않는 이상
같은 시간에 실행할 수 없다.

이 책에서는 iOS가 코드를 병행적으로 실행시킬 수 있게 하는 두 방법을 소개한다. 

1. Grand Central Dispatch (GCD)
2. Operation

Operation같은 경우는 GCD를 기반으로 작성되어 더 복잡한 시나리오를 다루는 것이 가능하게 되었다.

대부분의 현대 프로그래밍 언어는 병행성을 위한 몇몇 형태를 제공하며 Swift에서는 클로저를 사용한다. Swift5에서는 더 흔한 async/await를 구현할 계획이 있었지만
다음 릴리즈까지의 명세에서는 지워졌다.

## 2. GCD & Operations

위에서 말했듯이, 앱을 병행성 있게 만들 수 있는 2가지 API가 있다.

1. GCD, Grand Central Dispatch
2. Operation

이 둘 중 하나만 골라 써야 하는 것도 아니고, Operation이 GCD기반으로 작성되었기 때문에 완전히 배타적인 기술도 아니다.

### Grand Central Dispatch

GCD는 C의 `libdispatch`라는 라이브러리를 애플이 구현한 것이다. 자원이 이용가능한지에 따라 병행해서 실행 가능한 작업이나 메서드, 클로저를
큐에 줄세우는 것이 목적이다. 그리고 작업을 실행시킬 수 있는 프로세서 코어에서 작업을 실행시킨다.

GCD의 구현을 보면 쓰레드를 이용하는데, 이를 개발자가 직접 관리하는 것을 걱정하지 않아도 된다. GCD가 관리하는 모든 작업들은 FIFO 큐에 배치된다. 이렇게
큐에 넣어진 작업들은 이후 시스템에 의해 관리되는 쓰레드에서 실행된다. 이떄 어떤 쓰레드에서 작업을 실행할지는 보장할 수 없다.

### Synchronous와 asynchronous task

큐에 배치된 작업들은 동기/비동기적으로 실행될 수 있다. 

* 작업이 동기적으로 실행된다 : 앱은 현재 실행하고 있는 작업이 끝날때까지 기다리고, 다음 작업을 실행하기 위해 대기하고 있을 것이다.
* 작업이 비동기적으로 실행된다 : 현재 실행하고 있는 작업의 결과를 기다리지 않고 다음 작업을 바로 실행한다.

위에서 모든 작업들은 FIFO 큐에 배치된다고 했는데, 큐가 FIFO 기반이어도 내가 작업을 큐에 제공한 순서대로 작업이 끝나는 것은 아니다. 말 그대로
FIFO는 작업이 언제 시작할지를 결정하지, 끝날 때를 결정하는 것은 아니다.

일반적으로 오랜 시간 실행되는 작업을 백그라운드에서 비동기적으로 실행시키고 싶을 것이다. GCD에서는 클로저를 이용해 이를 간단하게 구현했다.

```swift
// Class level variable
let queue = DispatchQueue(label: "someLabel.hello.example") // 1. 디스패치 큐를 생성한다.

// Somewhere in your function
queue.async {
    // 2. Call slow non-UI methods here
    DispatchQueue.main.async {
        // 3. Update the UI here
    } 
}
```

위 코드를 보면

1. 큐를 생성하고
2. 백그라운드 쓰레드에서 비동기적으로 작업을 실행시키고
3. 완료되면 UI를 업데이트하기 위해 메인 쓰레드로 코드를 위임한다.

### Serial, Concurrent queue

두 종류의 큐가 있다. 

* Serial 큐 : 큐와 연관된 **단일 쓰레드**가 있다. 따라서 오로지 한 시간에는 작업을 하나만 실행시킬 수 있다. 
* Concurrent 큐 : 시스템이 리소스를 가진대로 가능한 많은 쓰레드를 활성화시킬 수 있다. 쓰레드는 concurrent 큐에서 생성되며 이용된다.

iOS에 concurrent 큐를 이용한다고 해도 한 시간에 하나 이상의 작업이 실행될거라는 보장은 없다.

### Asynchronous doesn't mean concurrent

작업이 비동기라고 해서 이 작업들이 병행적으로 실행된다는 보장은 없다. 비동기 작업을 Serial큐나 concurrent 큐에 모두 제공할 수 있다. 
동기/비동기로 실행한다는 것은 내가 작업을 실행하고 있는 큐가 다음 작업을 소환하기 전에 해당 작업이 끝날때까지 기다려야 할지 말지를 결정한다.

한편, 큐가 serial/concurrent 하다고 분류하는 것은 큐가 single/mutliple 쓰레드를 갖고 있는지를 결정한다.
예를 들어 serial 큐에 asynchronous한 작업을 세 개 제공하면 serial 큐에서는 쓰레드가 오직 하나만 존재하기 때문에 한 작업이 완전히 끝나야 
다른 작업을 실행시킬 수 있는 것이다.

여기서 헷갈리지 말아야 하는게, 비동기 작업이 serial 큐에 배치되어서 동기적으로 실행된다고 해도, 작업 자체는 비동기 작업이다. 즉 비동기 작업이
serial 큐로 들어가면 이런 serial 큐의 환경 때문에 동기처럼 실행이 되는 것이다. 

### Operations

GCD는 백그라운드에서 한 번 실행하고 마는 일반적인 작업들에 사용하기 좋다. 만약 이미지 수정 기능과 같이 재사용 할 수 있는 기능을 구현하고 싶다면 이 기능을
클래스로 묶는 것이 좋을 수도 있다. `Operation`을 상속해서 이를 해결할 수 있다.

#### Operation subclassing

작업을 실행하는 클로저를 GCD에서 `DispatchQueue`에 제공하는 것과 비슷하게, `Operations`는 `OperationQueue`에 제공될 수 있는 전체가 기능으로
작동하는 클래스다. 이때 작업들은 아래 상태로 존재할 수 있다.

* isReady
* isExecuting
* isCancelled
* isFinished

GCD와는 다르게, 작업은 기본적으로 동기적으로 실행되고, 이를 비동기적으로 실행시키기 위해서는 더 많은 작업이 필요하다. 작업을 직접 실행시킬 수도 있지만, 동기적인
환경 때문에 좋은 생각이 아니다. UI 성능이 영향을 받지 않게 하기 위해 `OperationQueue`에 작업을 던져서 메인 쓰레드에서 작업을 떼내고 싶을 것이다.

### Bonus features

Operations는 작업을 취소하거나, 작업의 상태를 알리고, 비동기적인 작업들을 operation으로 묶고, 여러 작업들 사이의 의존성을 명세하는 것과 같이
작업에 대한 강력한 통제권을 제공한다.

### BlockOperation

종종 operations에 많이 의존하는 앱을 구현하면서 GCD스럽고 더 간단한 클로저가 필요할 수도 있다. 하지만 이때 `DispatchQueue`를 생성하고 싶지 않으면
`BlockOperation` 클래스를 사용할 수 있다.

`BlockOperation`은 `Operation`을 상속하며 default 전역 큐에 있는 하나 이상의 클로저가 병행적으로 실행되는 것을 관리한다. `Operation`을 상속하면 operation을 이용하는
장점도 얻을 수 있다.

### 어떤 것을 써야 하나?

GCD는 단순히 실행하고 잊어버려도 되는 간단한 작업들에 더 간단하게 적용이 가능하다. 반면 Operation은 한 작업을 트래킹하거나 그것을 취소해야 하는 권한을 계속 유지하기 위한
기능들을 제공한다.

좀 더 풀어서 말하면 실행되기만 하면 되는 메서드/코드의 단위를 가지고 작업하고 있다면 GCD를, 데이터나 기능을 캡슐화해야 하는 객체를 가지고 작업하고 있다면 Operations를 
사용하는 것이 좋다. 

## 3. Queues & Threads

### Threads

멀티쓰레딩이라는 말이 있다. 쓰레드는 실행 쓰레드의 짧은 부분이고, 실행되는 프로세스가 시스템에 있는 자원을 어떻게 분리시키는지에 대한 것이다. iOS 앱은 여러 쓰레드를 활성화해서
여러 개의 작업을 실행한다. 기기의 CPU가 여러개라면 여러 쓰레드를 가질 수 있다.

앱을 여러 쓰레드로 나눠서 동작시키면 장점이 있다.

* 빠른 동작 : 쓰레드에 작업을 실행하면, 병행적으로 실행시키는 것이 가능하기 때문에 모든 작업들을 순서대로 ㅣㄹ행하는 것보다 빨리 끝낼 수 있다.
* 반응성 : 만약 메인 UI 쓰레드에서 사용자에게 보여지는 작업만을 실행한다면, 사용자는 다른 쓰레드에서 실행되고 있는 작업 때문에 앱이 느려지거나 멈추는 현상을 겪지 못할 것이다.
* 최적화된 자원 소요 : 쓰레드들은 OS에 최적화되어 있다.

더 많은 코어를 가질수록 더 많은 쓰레드를 가지고, 앱이 더 빨라지게 된다. 

### Dispatch Queues

쓰레드를 가지고 작업하기 위해 `DispatchQueue`를 생성한다. 큐를 생성하면, OS는 큐에 하나 이상의 쓰레드를 생성해 할당했을 가능성이 있다.

만약 존재하는 쓰레드들이 사용 가능하다면 이를 재사용 하겠지만, 아니라면 OS는 필요할 때 이것들을 생성할 것이다. 

Dispatch 큐를 생성하기 위해 아래와 같은 코드를 작성한다.

```swift
let label = "com.somelabel.example"
let queue = DispatchQueue(label: label)
```

일반적으로 생성자 안의 label에 텍스트를 직접 넣지만, 코드의 짧음을 위해 별개의 문장으로 쪼갠다.

`label`은 단순히 식별의 목적을 위한 고유한 값이다. 고유함을 보장하기 위해 UUID를 사용할 수 있지만, 라벨은 내가 디버깅할 때 개발자가 보는 것이며
텍스트를 할당하는 것이 개발에 좋기 때문에 역-DNS 스타일의 이름을 사용하는 것이 가장 좋다.

### Main queue

앱이 처음 실행되었을 때 `main` dispatch 큐는 자동으로 생성된다. 이 큐는 UI에 대한 책임이 있는 단일 큐다. 이건 굉장히 자주 사용되기 때문에, 애플은 
`DispatchQueue.main`으로 이 큐에 접근할 수 있게 클래스 변수로 만들었다. 이 큐에서는 UI와 관련된 작업만 실행시키고, 그렇지 않는 작업들은 절대로 실행시키지 않는다.

기본적으로 DispatchQueue는 serial 큐이기 때문에 concurrent 큐를 생성하려면 아래와 같이 attribute를 설정한다.

```swift
let label = "com.someexample.label"
let queue = DispatchQueue(label: label, attributes: .concurrent)
```

병행 큐는 굉장히 흔하기 때문에 애플은 큐가 꼭 가져야 하는 Quality of service(Qos)에 따라 다른 6개의 전역 큐를 제공한다.

### Quality of service

Concurrent dispatch queue를 사용하면, iOS에게 큐에 보내진 작업들의 중요도를 알려줘서 자원을 요구하는 다른 작업들 사이에서 작업들의 우선순위를
적절히 매길 수 있게 해야  한다. 높은 우선순위의 작업은 낮은 우선 순위의 작업보다 빨리 실행되어야 하고, 더 많은 시스템 자원과 에너지를 써야 한다.

병행 큐가 필요하지만 스스로 관리하고 싶지 않을 때, 미리 정의된 전역 큐 중 하나를 갖고 오기 위해 `DispatchQueue`의 `global`이라는 클래스 메서드를 사용할 수 있다.

```swift
let queue = DispatchQueue.global(qos: . userInteractive)
```

위에서 언급했듯이, 애플은 6개의 qos 클래스를 제공한다.

**.userInteractive**

`.userInteractive` QoS는 사용자가 직접 상호작용하는 작업들을 위해 사용된다. UI를 반응성 있게 하고 빠르게 유지하기 위해 필요한 UI를 업데이트 하는 연산,
애니메이션 등등이 여기 해당된다. 만약 앱이 즉각적으로 사용자의 동작에 반응하지 않으면 멈춘 것처럼 보이기 때문에 이 큐에 들어간 작업들은 시각적으로 즉각 완료되어야 한다.

**.userInitiated**

`.userInitiated` 큐는 사용자가 즉각적으로 실행되어야 하는 작업을 UI에서 떼어놓았지만 비동기적으로 실행될 수 있을 때 사용되어야 한다. 예를 들어, 
문서를 로컬 데이터베이스에서 열거나 읽을 때, 유저가 버튼을 클릭하면 작업을 이 큐에 배치하면 된다. 이 큐에서 실행되는 작업들은 몇 초 내에 완료되어야 한다.

**.utility**

`.utility` 디스패치 큐는 일반적으로 길게 실행되는 연산이나 I/O, 네트워킹이나 연속적으로 데이터를 가졍는 것과 같은 작업 지표를 포함하는 작업을 위해 사용된다. 시스템은
반응성과 성능을 에너지 효율과 저울질한다. 이 큐에 있는 작업은 몇 초에서 몇 분까지 걸리 수 있다.

**.background**

사용자가 직접적으로 알고 있지 않은 작업들은 `.background` 큐를 이용한다. 이 작업들은 사용자 상호작용을 요구하지 않고 시간에 제약이 있지도 않다. 연산에 필요한 데이터를 미리 
가져오는 것, 데이터베이스 유지, 원격 버스 동기화와 백업을 실행할 때 이 큐를 이용하면 된다. OS는 속도 대신에 에너지 효율성에 초점을 맞출 것이다. 이 큐는 오랜 시간 동안, 몇 분 이상의 작업을 위해 사용된다.

**.default and .unspecified**

명시적으로 사용하지 말아야 하는 두 가지 경우가 있다. `.default`는 `.userInitiated`와 `.utility` 사이에 있는 옵션이고 이는 qos argumnet의 기본값이다.
`.unspecified` 옵션은 qos 에서 벗어난 쓰레드를 겨냥한 legacy API를 지원하기 위해 존재한다.

전역 큐는 항상 concurrent하며 FIFOek.

### Inferring QoS

 만약 자신만의 병행 디스패치 큐를 만드려면 아래와 같이 생성자를 통해 qos가 무엇인지 시스템에 알려줄 수 있다.
 
 ```swift
 let queue = DispatchQueue(label: label,
                          qos: .userInitiated,
                          attributes: .concurrent)
 ```

OS는 큐에 제공된 작업들을 봐서 qos를 필요할 때 바꾼다.

만약 큐보다 높은 qos의 작업이 배치된다면 큐의 qos 레벨이 높아질 것이고, 큐에 있는 모든 작업들의 우선순위도 같이 올라간다.

만약 현재 context가 메인 쓰레드라면, 우리가 추론할 수 있는 QoS는 `.userInitiated`다. QoS를 스스로 정의할 수 있지만 더 높은 QoS를 갖업을 추가했다면
큐의 QoS는 이에 맞추기 위해 높아질 것이다.

### Add task to queues

디스패치 큐들은 작업을 큐에 추가하기 위한 동기와 비동기 메서드를 모두 제공한다. 여기서 작업이란 단순히 "실행해야 하는 코드 블록"이다. 
예를 들어 앱이 실행되었을 때 앱의 상태를 업데이트 하기 위해 서버와 연결해야 할 수도 있다. 이는 즉각적으로 실행될 필요가 없고 네트워킹 I/O에 따라
달려있기 때문에 아래와 같이 전역 유틸리티 큐에 전달한다.

```swift
DispatchQueue.global(qos: .utility).async { [weak self] in
    guard let self = self else {return}

    // Perform your work here

    //Switch back to the main queue to update your UI
    DispatchQueue.main.async {
        self.textLabel.text = "New articles available!"
    }
}
```

GCD 비동기 클로저에서 self를 강하게 획득하는 것은 클로저가 완료되면 클로저가 비할당되기 때문에 참조 사이클을 생성하지는 않지만, `self`의 생명 주기는 연장한다.
예를 들어 만약 어느 순간 해제된 뷰 컨트롤러에서 네트워크 요청을 했다면, 클로저는 여전히 호출될 수 있다. 만약 뷰 컨트롤러를 약하게 획득했다면 `nil`이 될 것이지만 강하게 획득했다면
뷰 컨트롤러는 클로저가 작업을 끝낼때까지 살아있을 것이다. 이렇게 약하게/강하게 값을 획득하는 것은 요구사항에 따라 달라질 수 있다.

이 코드에서 UI를 업데이트 하는 부분이 백그라운드 큐에서 dispatch되는 것 내의 main 큐에 어떻게 전다되는지 확인하면,
async 타입의 호출을 다른 것 안에 얺어 호출한다. 이런 경우는 굉장히 흔하다.

UI 업데이트를 메인 큐 외의 다른 큐에서 실행하면 안된다. API 콜백을 사용한ㄴ 큐에 대한 것이 아니라면, 메인 큐에 전달해야 한다.

작업을 dispatch 큐에 동기적으로 전달하는 것을 굉장히 조심해야 한다. 비동기 대신 동기 메서드를 사용한다면 이게 정말 맞는 것인지 다시 생각할 필요가 있다.
만약 현재 큐에 큐를 block 시키는 동기 작업을 제출했다면, 작업은 현재 큐의 자원에 접근할 것이고, 이는 데드락을 초래할 것이다.
비슷하게, main 큐에서 동기 메서드를 호출하면 UI를 업데이트 하는 쓰레드를 멈추게 되어 앱이 멈춰진 것처럼 보인다.

따라서 절대로 메인 쓰레드에서 sync를 호출하면 안된다. 아니면 메인 쓰레드가 멈춰서 데드락에 빠질 수 있다. 

### DispatchWorkItem

익명의 클로저말고 `DispatchQueue`에 전달할 수 있는 다른 방법이 있다. `DispatchWorkItem`은 큐에 전달하고 싶은 코드를 갖는 실제 객체를 제공하는 클래스다.

```swift
let queue = DispatchQueue(label: "xyz")
queue.async {
    print("The block of code ran!")
}
```

위의 코드는 아래의 코드와 똑같이 동작한다.

```swift
let queue = DispatchQueue(label: "xyz")
let workItem = DispatchWorkItem {
    print("The block of code ran!")
}
queue.async(execute: workItem)
```

### Canceling a work item

`DispatchWorkItem`을 사용하고 싶은 이유 중 하나는 작업을 시랭 전/실행 도중에 작업을 멈추고 싶어서일 것이다. `cancel()`을 호출하면 아래의 두 작업 중 하나가 실행된다.

1. 만약 작업이 큐에서 시작되지 않았다면, 지워진다.
2. 만약 작업이 진행중이라면, `isCancelled` 프로퍼티가 `true`로 변한다.

가능하다면 코드 상에서 `isCancelled` 프로퍼티를 주기적으로 확인해서 작업을 취소할 때 적절한 행동을 하도록 한다.

### Poor man's dependencies

`DispatchWorkItem` 클래스는 현재 작업이 완료된 후에 실행되어야 하는 다른 `DispatchWorkItem`을 정의하는데 사용할 수 있는 `notify(queue:execute:)` 메서드를 제공한다.

```swift
let queue = DispatchQueue(label: "xyz")
let backgroundWorkItem = DispatchWorkItem {}
let updateUIWorkItem = DispatchWorkItem {}

backgroundWorkItem.notify(queue: DispatchQueue.main, execute: updateUIWorkItem) //back작업이 끝나면 메인 큐에서 updateUI 작업을 실행시킨다. 
queue.async(execute: backgroundWorkItem)
```

operation의 경우 dependency를 설정하면 설정한 큐들 모두 큐에 추가해줘야 하는데, 여기에서는 단순히 notify 안에서 실행할 큐와 작업을 지정해주면 되나보다.

만약 작업을 취소하거나 의존성을 명시하는 기능이 필요하다면 Operation을 사용하는 부분을 같이 봐도 된다.

### Groups & Semaphores

가끔 큐에 작업들을 그냥 던지는 것 대신에 작업 그룹들을 처리할 필요가 있다. 이것들은 같은 시간에 실행 될 필요는 없지만, 그룹 내의 작업들이 모두 완료되었을 때를
알기 위해 사용한다. 애플은 이를 위해 Dispatch Groups를 제공한다.

### Dispatch Group

`DispatchGroup`이라고 이름이 붙여진 클래스는 작업들의 그룹이 완료되었는지를 추적하고 싶을 때 사용한다.

`DispatchGroup`은 초기화하는 것에서 시작한다. 그룹 하나를 이미 갖고 있고 작업을 이 그룹으로 넣고 싶을 때, 야넴ㅅ초 벼뎓dp qlehdrl aptjemdml argument로 그룹을 전달할 수 있다.

```swift
let group = DispatchGroup()

someQueue.async(group: group) { ... your work ... }
someQueue.async(group: group) { ... more work ... }
someOtherQueue.async(group: group) { ... other work ... }

group.notify(queue: DispatchQueue.main) { [weak self] in 
    self?.textLabel.text = "All jobs have completed"
}
```

위 코드에서 볼 수 있듯이, 그룹은 꼭 하나의 dispatch 큐에 연결되지는 않는다. 하나의 그룹 내의 작업들은 우선순위에 따라 여러 개의 큐에 전달이 가능하다.
`DispatchGroup`은 `notify(queue:)` 메서드를 사용해서 전달된 모든 작업이 끝났음을 알릴 수 있다.

알리는 것 자체는 비동기적이므로, 이전에 전달된 작업들이 아직 다 끝나지 않은 한 notify를 호출하는 것 후에 더 많은 작업을 추가하는 것이 가능하다.

`notify` 메서드가 dispatch 큐를 파라미터로 받는 것을 볼 수 있다. 모든 작업이 완료되면, 내가 제공한 클로저는 파라미터로 받은 디스패치 큐에서 실행될 것이다. 

## 5. Concurrency Problems


Dispatch 큐가 제공하는 장점이 모든 성능 이슈에 대해 만변통치약처럼 작동하지는 않는다. 만약 조심하지 않고 병행성을 구현했을 때는 아래의 문제를 겪을 수 있다.

* Race conditions
* Deadlock
* Priority inversion

### Race conditions

쓰레드는 앱 자체를 포함해서 같은 프로세스, 같은 주소 공간을 공유한다. 이는 쓰레드들은 같은 공유되는 자원에 읽고 쓰는 것을 시도한다는 의미다. 만약 이때 조심하지 않는다면,
많은 쓰레드가 같은 시간에 같은 변수에 쓰려고 하기 때문에 경쟁 조건에빠질 수 있다.

두 쓰레드가 실행중이고, 둘 다 한 변수를 업데이트 한다고 생각하자. 읽는 것과 쓰는 것은 하나의 작업처럼 실행시킬 수 없는 분리된 작업들이다. 
컴퓨터는 한 틱에 하나의 작업을 실행할 수 있게 하는 clock cycle을 기반으로 동작한다.

이때 컴퓨터의 clock cycle과 실제 시계를 혼동하면 안된다. iPhone XS는 2.49 GHz 프로세서를 가지고 있기 때문에 한 초에 2,490,000,000 clock cycle을 갖는다.

경쟁 조건으로 예상하지 못한 결과를 초래 했을 경우 serial 큐를 이용해서 해결할 수 있다. 만약 프로그램이 concurrent하게 접근되어야 하는 변수를 가지고 있다면,
아래와 같이 private 큐로 읽기와 쓰기를 감쌀 수 있다.

```swift
private let threadSafeCountQueue = DispatchQueue(label: "...")
private var _count = 0
public var count: Int {
    get {
        return threadSafeCountQueue.sync {
            _count
        }
    }
    
    set {
        threadSafeCountQueue.sync {
            _count = newValue
        }
    }
}
```

위와 같이 serial 큐를 이용해서 변수에 접근하는 것을 관리하고 하나의 변수에 하나의 쓰레드만 접근하는 것을 보장할 수 있다.

### Thread barrier

가끔, 간단한 변수 수정보다 게터/세터에 더 복잡한 로직을 공유 리소스가 요구하는 경우가 있다. 이때 lock과 세마포어와 관련된 솔루션들이 많이 나오곤 한다.
Locking은 적절히 구현하기 어렵다. 대신 애플의 dispatch barrier를 이용하면 좋다.

병행 큐를 생성하면 같은 시간에 실행시키고 싶은 만큼의 읽기 작업을 실행시킬 수 있다.

만약 변수에 쓰기 작업이 필요한 경우, lock을 시켜서 이전에 제출된 작업들이 완료되었지만, 업데이트가 끝날때까지는 새로운 작업이 추가되지 않게 한다.

Dispatch barrier를 아래와 같이 구현한다.

```swift
private let threadSafeCountQueue = DispatchQueue(label: "...", attributes: .concurrent)
private var _count = 0
public var count: Int {
    get {
        return threadSafeCountQueue.sync {
            return _count
        }
    }
    
    set {
        threadSafeCountQueue.async(flags: .barrier) { [unowned self] in
            self._count = newValue
        }
    }
}
```

Concurrent한 큐를 원한다는 것을 명시했다는 것과, 쓰기 작업이 barrier를 이용해서 구현이 되었다는 것을 확인해라.

Barrier에 접근하게 되면, 큐는 serial인 것처럼 행동하며 barrier 작업만이 실행될수있게 한다. 작업이 끝났다면, barrier task 이후에 전달된 작업들이
병행적으로 실행될 수 있다.

### Deadlock

서로 절대로 완료되지 않는 작업들을 기다리는 것이 데드락이다. 

### Priority Inversion

Priority inversion은 낮은 qos를 가진 큐가 높은 qos를 가진 큐보다 더 높은 시스템 우선순위를 가지게 되었을 때 발생한다.

더 높은 qos를 갖는 큐가 더 낮은 qos를 갖는 큐와 같은 자원을 공유할 때 우선순위 역전이 발생한다. 더 낮은 큐가 객체에 락을 건다면, 더 높은 큔ㄴ 기다려야 한다.
락이 풀릴때까지, 높은 우선순위의 큐는 낮은 우선순위의 작업이 끝날때까지 아무것도 할 수 없다. 

## 6. Operations

Operations는 GCD와 매우 유사하게 동작한다. GCD와 Operations 모두 다른 쓰레드에서 동작해야 하는 코드의 뭉치를 전달할 수 있게 한다. 하지만 operation
은 전달된 작업을 더 강력하게 통제할 수 있다. 

### Reusability

Operation을 사용하고 싶은 이유 중 하나는 재사용성이다. Operation은 실제 Swift 객체이며, 입력값을 넣어 작업을 세팅하거나 helper 메서드를 구현할 수 있다.
그래서 한 작업을 감쌀 수 있고 미래에 종종 실행하고, 한 번 이상 그 작업을 실행할 수 있게 전달하는 것이 쉬워진다.

### Operation States

Operation은 생명 주기를 대표하는 state machine이 있다. 

* 초기화되고 실행할 준비가 된다면, isReady 상태가 된다.
* start 메서드를 실행하면 isExecuting 상태가 될 것이다.
* cancel 메섣를 호출하면, isFinished 상태로 이동하기 전에 isCancelled 상태로이동한다.
* 만약 취소가 된 게 아니라면, isExecuting에서 isFinished 상태로 바로 이동한다.

![image](https://user-images.githubusercontent.com/41438361/121327220-35ab8f80-c94e-11eb-967c-4da708f9ea45.png)

이 상태들은 `Operation` 클래스의 읽기 전용 Boolean 프로퍼티들이다. 작업이 실행되는지 아닌지를 확인하기 위해 작업의 실행 도중 query를 할 수 있다.

`Operation` 클래스에서 모든 상태 전이를 관리하고, 개발자가 직접 영향을 미칠 수 있는 두 가지는 작업을 시작해서 isExecuting 상태로 이동하고, cancel 메서드를 사용해서
isCancelled 상태로 이동하는 경우다.

### BlockOperation

`BlockOperation` 클래스를 이용해서 Operation을 코드 블럭 밖에서 만들 수 있다. 보통 생성자에 클로저를 전달한다.

```swift
let operation = BlockOperation {
    print("2 + 3 = \(2 + 3)")
}
```

`BlockOperation`은 기본 글로벌 큐에 있는 하나 이상의 클로저의 병행적인 실행을 관리한다. Operation은 KVO(Key-Value Observing)알림, 의존성과
Operation이 제공하는 다른 모든 것의 이점을 챙길 수 있다. 

`BlockOperation`은 클로저의 그룹을 관리한다. 모든 클로저가 끝났을 때 스스로 종료되었다고 표시하는 것이 dispatch group과 비슷하게 동작한다.
위 코드는 operation에 단일 클로저를 더하는 것이지만 여러개의 아이템을 더할 수 있다.

`BlockOperation`에 있는 작업들은 병행적으로 실행된다. 만약 이를 순서에 맞게 실행하고 싶다면, 이를 private한 `DispatchQueue`에 전달하거나 우선순위를 설정해야 한다.

`BlockOperation`에 여러개의 클로저를 더하고 싶으면, `addExecutionBlock` 메서드를 호출해서 새로운 클로저에 전달하면 된다. 

```swift
let sentence = "Ray’s courses are the best!"
let wordOperation = BlockOperation()

for word in sentence.split(separator: " ") {
    wordOperation.addExecutionBlock {
        print(word)
      }
}

wordOperation.start()
```

### Subclassing operation

`BlockOperation` 클래스는 간단한 작업에 좋지만 더 복잡한 작업이나 재사용이 가능한 컴포넌트를 실행하고 싶을 때는 `Operation`의 자식 클래스를 만들어야 한다.





ddd
