---  
layout: post  
title: "[iOS] - GCD 2. GCD & Operations"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: 

---  

# GCD & Operations

앱을 concurrent하게 만드는 데 사용할 수 있는 두 개의 API가 있다.

1. **Grand Central Dispatch**, **GCD**
2. **Operation**

`Operation` 클래스는 GCD를 기반으로 구현되었다.

이 글에서는 다음의 내용들을 볼 것이다.

* GCD와 Operation이 무엇인지
* Sync (동기)와 Async (비동기)의 차이
* Serial 큐와 Concurrent 큐의 차이

## Grand Central Dispatch

GCD는 **작업들(메서드/클로저)들을 큐에 넣어서 가용 자원에 따라 병행해서 실행할 수 있게 하는 라이브러리다.** `libdispatch` 라이브러리라고도 불리고, Apple이 C 언어로 개발했다. [GCD Github 페이지](https://github.com/apple/swift-corelibs-libdispatch)에 의하면 GCD는 멀티코어 하드웨어 환경에서 코드를 병행적으로 실행할 수 있게 지원하는 라이브러리다. 

즉 GCD는 간단하게 정리하면 **작업을 병행적으로 실행할 수 있게 해주는 라이브러리**다. 

그렇다면 왜 이런 작업을 해주는 라이브러리를 따로 사용하는 것일까? GCD는 내부적으로 구현부에서 thread를 사용한다. GCD를 사용했을 때 개발자로서 편리한 점은 이런 **thread들을 개발자가 직접 관리하지 않아도 되기 때문**이다. GCD를 사용하지 않고 thread를 생성해서 작업을 직접 넣으려면 굉장히 많은 양의 코드를 작성해야 하는데, GCD를 사용한다면 굉장히 적은 양의 코드로 같은 작업을 할 수 있다.

GCD에서 관리하는 모든 작업들은 GCD에서 관리하는 FIFO(First-In, First-Out) 큐에 들어갔다 나오게 된다. 큐에 전송된 작업들은 시스템이 관리하는 thread pool에서 실행되게 된다. 즉 개발자는 작업을 만들어서 GCD 큐에 넣기만 하면 되고, 어떤 thread에서 실행할지는 GCD가 결정한다. 참고로 작업을 실행할 때 어떤 thread가 작업을 실행하는지는 보장되지 않는다. 

> 컴퓨터 공학에서 thread pool이란 컴퓨터 프로그램을 병행적으로 실행하기 위한 소프트웨어 디자인 패턴이다. Thread pool은 여러 개의 스레드들을 관리하고, 스레드들은 작업이 자신에게 할당되기를 기다린다.

즉 작업을 큐에 넣어서 병행적으로 실행시키는데, **작업**을 어떻게 실행시킬지, 어떤 **큐**에 작업을 실행시킬지를 아래와 같이 구분해서 실행할 수 있다.

* 작업 : sync (동기) / async (비동기)
* 큐 : serial / concurrent

## Synchronous and asynchronous tasks

큐에 넣어진 **작업**들은 두 가지 형태로 실행될 수 있다.

* **Synchronous (동기)** : 앱은 현재 실행하고 있는 작업이 끝날 때까지 현재의 run loop를 block 시킨후 기다리고, 끝나면 다음 작업을 실행한다. 
* **Asynchronous (비동기)** : 비동기 작업을 실행하면 작업이 끝날 때까지 기다리지 않고, 작업이 실행되는 동안 다른 작업을 실행할 수 있다. 

> GCD에서 사용하는 큐가 FIFO이지만, 작업이 큐에 넣어진 순서대로 끝나는 것은 아니다. FIFO 순서는 작업이 시작하는 순서만 보장하고, 끝나는 것은 순서를 보장할 수 없다. 

일반적으로는 이미지 다운로드 작업과 같이 **오래 걸리고, UI를 업데이트하지 않는 작업을 백그라운드에서 비동기적으로 실행한다.** 

코드를 대강 보면 아래와 같다. 1. 큐를 생성하고, 2. 백그라운드 thread에서 작업을 비동기적으로 실행하게 전달하고, 3. 작업이 완료되면 main thread에 코드를 위임해서 main thread에서 UI를 업데이트하게 한다.

```swift
// 1. 큐 생성
let queue = DispatchQueue(label: "y00jin")

// 2. 작업을 비동기로 실행시킬게요
queue.async {
    // 오래 걸리는 작업 수행
    
    // 3. main thread에 작업 위임
    DispatchQueue.main.async {
        // UI 업데이트
    }
}
```

## Serial and concurrent queues

앞서서 작업의 실행 형태에 두 가지가 있다고 했는데, 작업을 넣는 **큐**도 두 종류가 있다.

* **Serial** : Serial 큐는 **하나의 thread**만 있어서 한 번에 하나의 작업만 실행할 수 있다.
* **Concurrent** : Concurrent 큐는 **자원이 허용하는 한 많은 thread를 사용**할 수 있기 때문에 필요한 한 많은 thread들이 생성된다. 

> 사진

> Concurrent 큐를 사용해도, 여러 개의 작업이 동시에 실행된다는 보장이 없다. 만약 자원이 매우 한정적인 상황이라면 오로지 하나의 작업만 수행할 수도 있다.

### Asynchronous != Concurrent

Sync/Async 와 Serial/Concurrent를 비슷한 개념으로 혼동하는 경우가 정말 많다. 하지만 이 둘은 **전혀 다른 개념**이다. Async 작업이라 해서 concurrent하게 실행되는 것이 절대 아니다. 

* Sync/Async : **작업을 실행하는 큐가 작업이 끝날때까지 기다려야 하는지, 아니면 다른 작업을 실행할 수 있는지 여부**
* Serial/Concurrent : **작업을 실행할 수 있는 thread가 하나인지, 여러 개가 있는지 여부**

따라서 serial/concurrent 큐에 sync/async 작업을 수행하는 경우의 수 4가지가 나올 수 있다.

1. Serial 큐에 sync 작업 3개 수행
2. Serial 큐에 async 작업 3개 수행
3. Concurrent 큐에 sync 작업 3개 수행
4. Concurrent 큐에 async 작업 3개 수행

## Operation

작업들 간의 의존성을 추가하는 등의 더 복잡한 작업을 수행할 때 `Operation` 클래스를 상속해서 수행할 수 있다. 

GCD에서 `DispatchQueue`에 수행할 클로저를 전달했던 것처럼, Operation에서도 `OperationQueue`에 `Operation`을 전달할 수 있다. 

`Operation`객체는 operation을 실행하기에 안전한지 판단하고, 생애주기동안 operation이 진행되는 것을 클라이언트에게 알리기 위해 내부적으로 상태를 갖는다. `Operation`은 다음의 상태를 가질 수 있다.

* `isReady`
* `isExecuting`
* `isCancelled`
* `isFinished`

### BlockOperation

Opeartion이 추상 클래스이기 때문에, `Operation`을 직접 사용할 수는 없고 시스템에서 정의된 자식 클래스 중 하나인 `BlockOperation`을 사용할 수 있다. `BlockOperation`은 하나 이상의 블럭을 기본 global 큐에서 병행적으로 실행하는 것을 관리한다. 이 클래스를 사용해서 여러 개의 operation 객체를 생성하지 않고 여러 개의 블럭을 동시에 실행시킬 수 있다. 

## 어떤 것을 고를지

* GCD
	* 실행하고 바로 잊어비릴 수 있는 비교적 간단한 작업에 주로 사용한다.
* Operation
	* Operation은 내부적으로 GCD를 사용해서 구현했고, GCD에 비해 작업을 다루는 데 더 많은 기능을 제공한다. (두 작업 간의 의존성 설정 등) 데이터를 캡슐화할 필요가 있는 객체를 사용하거나 더 많은 기능을 사용하고 싶을 때 사용한다.

GCD와 Operation을 비교할 때 많이 나오는 말 중 하나는 GCD는 작업의 취소 기능을 기본적으로 지원하지 않는다는 것인데, GCD에서도 `DispatchWorkItem` 을 사용해서 작업을 쉽게 취소할 수 있다.

* 참고
* https://github.com/apple/swift-corelibs-libdispatch
* https://byby.dev/gcd
* https://en.wikipedia.org/wiki/Thread_pool
* https://stackoverflow.com/questions/7078658/operation-queue-vs-dispatch-queue-for-ios-application
* https://developer.apple.com/documentation/foundation/operation
* https://developer.apple.com/documentation/foundation/blockoperation



