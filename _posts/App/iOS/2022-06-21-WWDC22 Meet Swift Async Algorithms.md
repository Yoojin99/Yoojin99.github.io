---  
layout: post  
title: "[iOS] - WWDC22 Meet Swift Async Algorithms"  
subtitle: ""  
categories: app
tags: app-ios wwdc
comments: true  
header-img: 

---  

[Meet Swift Async Algorithms](https://developer.apple.com/videos/play/wwdc2022/110355/?time=15)
https://github.com/apple/swift-async-algorithms
https://docs.swift.org/swift-book/LanguageGuide/Concurrency.html
https://developer.apple.com/videos/play/wwdc2021/10058
https://developer.apple.com/videos/play/wwdc2021/10256

Swift Async Algorithms 패키지는 AsyncSequence를 사용해서 시간에 따라 값을 처리하는 것에 초점을 맞추는 알고리즘 모음이다. 

# AsyncSequence Recap

<img width="780" alt="image" src="https://user-images.githubusercontent.com/41438361/174720589-56bbc577-5e62-4f9c-8138-3545fe9a88bc.png">

AsyncSequence는 비동기적으로 생성된 값들을 표현할 수 있게 해준다. 

* 기본적으로 Sequence와 같다. 즉 Sequence를 어떻게 사용하는지 알면 AsyncSequence를 어떻게 사용할 지 아는 것이다.
* for-await를 사용해서 반복할 수 있다.
* Sequence와는 두 가지 차이가 있다.

1. iterator의 `next` 함수가 비동기적으로 동작해서 Swift concurrency를 사용해서 값들을 전달할 수 있다.
2. Swift 의 throw와 비슷하게 failure가 발생하면 이를 처리할 수 있다.

AsyncSequence를 통해 Sequence에서 사용할 수 있는 도구들을 모두 비동기 버전으로 사용할 수 있다. map, filter, reduce 등이 여기 해당된다.

=> Meet AsyncSequence 세션 (WWDC21)에서 더 확인할 수 있다.

<img width="1010" alt="image" src="https://user-images.githubusercontent.com/41438361/174725445-4cf7063e-416e-459d-82f2-62cb53ec9080.png">

Swift Async Algorithms 패키지는 여기에서 더 나아가 clock을 활용할 수 있게 됐다. Swift Async Algorithms 패키지는 오픈 소스 패키지다.

=> Swift Algorithms pakcage에 대해 더 알아보려면 "MEET THE Swift Algorithms and Collections packages" 세션(WWDC21)에서 확인할 수 있다.

# Multi-input algorithms

여러개의 input AsyncSequence를 다룰 수 있는 알고리즘들이 있다. 이 알고리즘들은 AsyncSequence를 여러 다른 방법으로 합치는 것에 초점을 맞춘다. 하지만 이들은 하나의 공통점이 있는데 **여러 개의 input AsyncSequence를 받아 하나의 output AsyncSequence를 만드는 것이다.*

## Zip

<img width="823" alt="image" src="https://user-images.githubusercontent.com/41438361/174726854-64acf4ef-3020-41c2-b603-c87e7eb6cd5f.png">

Zip 알고리즘은 여러 개의 입력을 받고 이들을 iterate해서 각 요소들을 합쳐 하나의 결과 튜플을 생성하는 알고리즘이다. 비동기 ZIP 알고리즘은 표준 라이브러리의 ZIP과 똑같은데, 각 요소들을 병행적으로 iterate하고 만약 한 곳에서라도 에러가 발생하는 경우 에러를 다시 던진다.

![Jun-21-2022 15-52-17](https://user-images.githubusercontent.com/41438361/174735378-05238159-bbfe-42ff-bf15-0e164edac8d5.gif)

가령 영상 프리뷰를 생성하고 효율적인 저장을 위해 영상을 여러 사이즈로 transcoding하는 작업을 비동기적으로 각각 진행한다고 해보자. Zip을 사용하면 transcoded된 영상과 프리뷰를 같이 서버에 보낼 수 있게 된다.

Zip이 병행적이기 때문에, transcoding이 더 오래 걸릴 수도 있고 프리뷰를 생성하는 것이 더 오래 걸릴 수 있다. Zip은 어떤 종류의 작업이 먼저 처리되어야 한다는 우선순위가 없기 때문에 두 값 중 들어오는 순서는 상관이 없다.
들어오는 순서와는 상관 없이 먼저 도착한 값은 튜플이 완성될 때까지 다른 값을 기다린다.

## Merge

<img width="1270" alt="image" src="https://user-images.githubusercontent.com/41438361/174728973-a97699b4-2b6e-4159-b380-c8eee9a30ebc.png">

가령 메세지를 주고 받을 때 AsyncSequence를 사용한다고 해보자. 그리고 계정이 여러개 있다면, 각 계정이 메세지들의 AsyncStream을 생성할 것인데, 이를 구현할 때는 모든 AsyncSequence인 것처럼 다뤄야 한다. 따라서 AsyncSequence를 합치는 알고리즘이 필요했고,
이를 Merge 알고리즘을 통해 해결할 수 있다.

Merge는 여러 AsynceSequence들을 병행적으로 iterate한다는 점에서는 Zip과 비슷하다. 하지만 **짝이 지어진 튜플을 생성하는 대신에 베이스들(여기에서는 각 작업을 의미하는 것 같다)이 같은 타입의 요소를 가정하고 이 요소들을 합쳐 하나의 AsyncSequence를 만든다.**

![merge](https://user-images.githubusercontent.com/41438361/176350241-324afc47-20a1-469a-a0e2-f82a7644c87f.gif)

Merge는 어떤 작업에서든 첫 번째 요소를 받는 걸로 시작한다. 더 이상 생성되는 값들이 없을 때까지 반복한다. 더 이상 생성되는 값이 없을 때 모든 base AsyncSequence가 nil을 리턴한다. 만약 base 중 하나라도 에러를 띄우면 다른 모든 iteration이 취소된다.

Merge를 통해 메세지들의 AsyncSequence를 받고 이들을 합칠 수 있다. 

# Clock, instant, duration

Zip, Merge와 같이 합치는 알고리즘은 값들이 생성될 때 병행적으로 동작하는데, 시간 자체를 가지고 작업하는게 유용할 때가 있다. Swift 5.7에서는 시간과 관련된 안전하고 일관성 있는 새로운 API `Clock`, `Instant`, `Duration`이 새로 등장했다.

## Clock

<img width="1005" alt="image" src="https://user-images.githubusercontent.com/41438361/176350823-97492265-1eb6-430a-8895-a2bf027e10b0.png">

Clock 프로토콜은 두 primitive를 정의한다.

1. 주어진 시간이 지난 후 wake up하는 방법
2. 현재라는 시간(개념)을 생성하는 방법

### ContinuousClock vs SuspendingClock

* ContinuousClock : 스탑워치처럼 시간을 잴 수 있다. 관측하는 것의 상태와 상관 없이 시간이 흐른다.
* SuspendingClock : 기기가 sleep할 경우 시간을 멈춘다. 

<img width="1574" alt="image" src="https://user-images.githubusercontent.com/41438361/176352175-4324c0d0-d6d0-440d-a8e4-f3aea85bc102.png">

위 코드에서 왼쪽은 `SuspendingClock`, 오른쪽은 `ContinuousClock`을 사용해서 시간을 잰다. 아래에는 재고 있는 시간의 경과를 나타낸다. 만약 기기가 sleep하면(맥북의 경우 노트북을 닫는 경우 등) `SuspendingClock`을 사용한 경우에서 시간을 재는 작업이 멈추게 된다.

<img width="1559" alt="image" src="https://user-images.githubusercontent.com/41438361/176352444-3f3fc95d-8d4a-4a6c-9381-8ce9f0a28548.png">

그래서 기기가 다시 wake up했을 때(맥북의 경우 노트북을 다시 열었을 때 등) `SuspendingClock`을 사용해서 시간을 잰 경우는 기기가 sleep한 시점에서 멈춰있지만 `ContinuousClock`을 사용한 경우는 계속 시간을 재서 더 많은 시간이 경과했을 것이다.


# Algorithms using time


# Collections
