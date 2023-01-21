---  
layout: post  
title: "[iOS] - GCD 3. Queues and Threads"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: 

---  

# 3. Queues & Threads

이 글에서는 다음의 내용을 볼 것이다.

* Thread 가 무엇인지
* Queue 가 무엇인지
* QoS 가 무엇인지
* Queue 를 생성하여 작업을 dispatch 하는 방법

# Threads

**실행 중인 프로세스가 작업들을 시스템이 가진 자원에 따라 나눈 것** 

iOS 앱은 여러 thread 를 사용해서 여러 작업을 수행하는 프로세스다. 

* Thread 는 기기 CPU 에 core 가 있는 한 많이 생성할 수 있다.
* Core, thread 가 많을 수록 앱의 속도는 더 빨라질 수 있다.
* OS 가 thread 를 언제 생성하고 없애야 하는지를 판단하는데 많은 수치들을 고려하고 있고, 높은 추상화를 사용해서 thread 를 생성하는 과정을 관리하고 있기 때문에 개발자가 명시적으로 할 필요가 없다. 

앱의 작업을 여러 thread 로 나눴을 때의 장점은 다음과 같다.

* 빠른 실행 : 작업을 thread 에서 실행하면 병행적으로 수행하는 것이 가능해지기 때문에 순차적으로 실행하는 것보다 작업을 빠르게 마칠 수 있다. 
* 응답성 (Responsiveness) : 사용자에게 보여지는 작업 (e.g. UI 업데이트) 들만 main UI thread 에서 수행하면 사용자 입장에서는 다른 thread 에서 수행되는 작업 때문에 앱이 느려진다거나 일시적으로 멈추는 것을 눈치채지 못한다.
* 자원 소비 최적화 : Thread 는 os 가 최적화했다.

# Dispatch queues

Queue 는 하나 이상의 thread 를 사용해서 작업한다. Queue 를 생성하면 OS 는 하나 이상의 thread 를 생성해서 queue 에 할당한다. 만약 이미 있는 thread 를 사용할 수 있을 때는 재사용하고, 그렇지 않은 경우에는 새로 만든다.

Dispatch queue 에는 다음 두 종류의 queue 가 있다.

1. Serial
2. Concurrent

`DispatchQueue` 로 queue 를 생성할 수 있다. 두 종류의 queue 를 어떻게 생성하는 지 보기 전에 `main` queue, `global` queue 에 대해 먼저 볼 것이다.

## Main queue

**UI 작업을 수행하는 serial queue.** 

* 앱이 시작하면 자동으로 생성됨
* `DispatchQueue.main` 클래스 변수로 접근할 수 있음
* **UI 작업만 동기적으로 수행해야 한다.** UI 작업이 아닌 이상 main queue 에서 절대로 동기적으로 수행하면 안되는데, 그렇지 않으면 UI 를 멈추게 돼서 앱의 성능을 떨어뜨릴 수 있기 때문이다.

## Global queue

****

Concurrent queue 가 필요하지만 직접 관리하고 싶지 않을 때 `global` 클래스 메서드를 통해 미리 정해진 global queue 중 하나를 가져와서 사용해도 된다. > Global queue 는 항상 concurrent 하고 FIFO 다.


```swift
let queue = Dispatch.global(qos: .userInteractive)
```


## Queue 생성

### 1. Serial

Serial queue 는 하나의 thread 를 갖고 한 번에 하나의 작업만 수행할 수 있다. Serial queue 내의 각 작업은 다음 작업을 시작하기 전에 무조건 완료되어야 한다.

```swift
let label = "com.y00jin"
let queue = DispatchQueue(label: label)
```

`label` 인자에는 식별을 위해 필요한 고유한 값을 넣는다. UUID 를 사용해서 고유함을 보장할 수도 있는데, 디버깅 할 때 `label` 확인하기 때문에 의미 있는 값을 부여하기 위해 reverse-DNS 스타일 이름을 쓰는 것이 좋다.

### 2. Concurrent

Concurrent queue 를 생성하기 위해서는 `attribute` 에 `.concurrent` 를 다음과 같이 전달하면 된다.

```swift
let label = "com.y00jin"
let queue = DispatchQueue(label: label, attributes: .concurrent)
```

Concurrent queue 는 굉장히 흔히 사용하기 때문에 **QoS (Quality of service)**에 따라 6개의 다른 global queue 가 제공되고 있다. 
 
## Quality of service

**작업의 우선순위/중요도**

**Concurrent dispatch queue 를 사용할 때, queue 에 전달된 작업이 얼마나 중요한지를 iOS 에 알려서 자원들을 요구하는 작업들의 우선순위를 정할 수 있게 해야 한다. 높은 우선순위의 작업이 낮은 우선 순위의 작업보다 더 많은 시스템 자원과 에너지를 소모해서 더 빠르게 수행된다.**

Apple 은 6 종류의 qos 를 제공하고 있다. 다음은 qos 우선순위가 중요한 것부터 나열한 것이다. 

1. .userInteractive : 사용자가 직접적으로 상호작용하는 작업. UI 의 반응성을 빠르게 유지하기 위한 작업에 사용된다. 만약 작업이 빠르게 수행되지 않는다면 앱이 멈춘 것처럼 보이기 때문에 이 우선순위를 갖는 queue 에 전달된 작업들은 즉각적으로 실행 후 완료된다. (e.g. UI 업데이트, 애니메이션)
2. .userInitiated : 사용자가 UI 를 통해 즉각 시작하려는 작업이지만 비동기적으로 수행될 수 있는 작업 / 사용자가 직접 요청한 작업이지만 결과에 UI 가 필요하지 않은 작업. 이 queue 에 전달된 작업들은 완료되기까지 대략 몇 초가 소요된다. (e.g. 로컬 데이터베이스에서 문서를 열거나 읽기 위해 사용자가 버튼을 클릭했을 때 수행하는 작업, 트위터에서 아래로 드래그해서 목록을 새로고침하는 작업 / 계산 앱에서 팁 재계산하는 작업)
3. *.default* : 명시적으로 사용하지 말아야 한다.
4. .utility : 로딩 인디케이터가 필요할 수 있는 길게 수행해야 하는 작업. 시스템은 반응성과 성능을 에너지 효율성과 조율해서 작업을 수행한다. 작업을 수행하는데 몇 분이 소요될 수 있다. (e.g. I/O, 네트워킹)
5. .background : 사용자가 모르는 작업. 사용자와 상호작용이 필요하지 않고 수행하는 시간이 제한적이지 않다. OS 는 속도보다 에너지 효율성에 초점을 맞춘다. 작업을 완료하는데 많은 시간이 소요될 수 있다. (e.g. Prefetching, 데이터베이스 관리, 원격 서버와 동기화, 백업) 
6. *.unspecified* : 시스템이 qos 를 추측하게 된다. qos 를 사용하지 않는 레거시 API 를 지원하기 위해 존재한다. `.default` 와 마찬가지로 명시적으로 사용하지 말아야 한다.

### QoS 추리

Concurrent dispatch queue 는 생성자를 통해 qos 를 지정할 수 있다.

```swift
let queue = DispatchQueue(label: label, qos: .userInitiated, attributes: .concurrent)
```

* Main thread 의 qos 는 `.userInitiated` 
* queue 의 qos 보다 더 높은 qos 를 가진 작업을 queue 에 할당하면 queue 의 qos, queue 에 넣어진 작업들도 우선순위가 같이 높아지게 된다.

## Queue 에 작업 추가

작업 : 실행해야 하는 블럭

Dispatch queue 는 queue 에 작업을 추가할 수 있는 다음 두 가지 메서드를 가지고 있다.

* `sync`
* `async`

### 예시

만약 앱이 시작하면, 서버와 연결해서 앱의 상태를 업데이트해야 할 수 있다. 이는 userInitiated 작업이 아닌데, 즉각적으로 수행될 필요가 없고 네트워킹 I/O 에 의존하는 작업이기 때문이다. 따라서 이 작업을 global utility queue 에 전달해야 한다.

```swift
DispatchQueue.global(qos: .utility).async { [weak self] in
	guard let self = self else { return }

	// 작업 수행
	// UI 를 업데이트하기 위해 main queue 로 전환해야 함
	DispatchQueue.main.async {
		self.textLabel.text = "UI 업데이트!"
	}
}
```

위 코드의 핵심은 두 가지 부분이 있다.

1. `[weak self]` : `DispatchQueue` 의 클로저도 일반적인 클로저와 마찬가지로 클로저의 획득된 (captured) 값을 잘 관리 해줘야 한다. GCD `async` 클로저에서 `self` 를 강한 참조를 하게 되면 순환 참조로 인해 메모리 누수가 발생할 수 있다. 
2. UI 업데이트 작업 : UI 를 업데이트 하는 작업을 background queue 안에서 `main` queue 에 전달했다. **UI 를 업데이트 하는 작업은 무조건 main queue 에서 수행되어야 한다. ** `main` queue 에 dispatch 할 수 있는 가능한 많은 작업들을 해서 적은 양의 코드만 main thread 에 전달하고, UI 를 반응성 있게 하는 것이 좋다. 

작업을 동기적으로 (synchronously) 하게 전달할 때는 특히 더 주의해야 한다. 만약 작업을 현재 사용하는 queue 에 동기적으로 전달하게 되면 현재 queue 를 block 하고, 작업은 현재 queue 의 자원을 사용하려고 하기 때문에 deadlock 이 발생한다. Main thread 에서 `sync` 를 호출하게 되면 main thread 를 block 하고 deadlock 이 발생할 수도 있기 때문에 호출하면 안된다.

또 다른 예시로 `Data` 메서드를 사용해서 이미지를 다운 받는 작업을 아래와 같이 구현했다.

```swift
DispatchQueue.global(qos: .utility).async { [weak self] in
	guard let self = self else { return }
	
	let url = self.urls[index]
	
	guard let data = try? Data(contentsOf: url),
		  let iamge = UIImage(data: data) else {
		return  
	}
}
```

위 코드에서 `Data(contentsOf: url)` 은 synchronous 하게 실행되지만, global queue 에서 실행되기 때문에 별도의 thread 에서 수행되고, UI 는 영향 받지 않게 된다. 

## DispatchWorkItem

`DispatchWorkItem` 은 실행할 작업 코드를 담고 있는 객체를 제공해서 이 객체를 queue 에 전달할 수 있다.

* 이름 없는 클로저를 사용해서 작업 전달
 
```swift
let queue = DispatchQueue(label: "xyz")
queue.async {
	print("여기에서 작업을 수행합니다.")
}
```

* DispatchWorkItem 사용

```swift
let queue = DispatchQueue(label: "xyz")
let workItem = DispatchWorkItem {
	print("여기에서 작업을 수행합니다.")
}
queue.async(execute: workItem)
```

### 취소

`DispatchWorkItem` 을 사용하면 실행 전, 도중, 후에 작업을 취소할 수 있다. Work Item 에 `cancel()` 메서드를 호출하면 다음 동작 중 하나가 수행된다.

1. 작업이 queue 에서 실행되지 않았을 경우 queue 에서 제거된다.
2. 작업이 실행중인 경우 `isCancelled` 프로퍼티가 `true` 로 바뀐다.

### 의존성 설정

`DispatchWorkItem` 의 `notify(queue:execute:)` 메서드를 사용해서 현재 work item 이 완료됐을 때 수행되어야 할  다른 `DispatchWorkItem` 과 수행할 queue 를 지정할 수 있다.

```swift
let queue = DispatchQueue(label: "xyz")
let firstWorkItem = DispatchWorkItem { }
let secondUIWorkItem = DispatchWorkItem { }

 firstWorkItem.notify(queue: DispatchQueue.main, execute: secondUIWorkItem)
 queue.async(execute: firstWorkItem)
```






* 참고
* https://stackoverflow.com/questions/40176485/whats-the-difference-between-userinitiated-and-userinteractive-in-gcd
* https://developer.apple.com/library/archive/documentation/Performance/Conceptual/EnergyGuide-iOS/PrioritizeWorkWithQoS.html
