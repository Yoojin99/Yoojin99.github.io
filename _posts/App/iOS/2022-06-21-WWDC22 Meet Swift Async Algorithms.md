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

Merge는 여러 AsynceSequence들을 병행적으로 iterate한다는 점에서는 Zip과 비슷하다. 하지만 짝이 지어진 튜플을 생성하는 대신에 각 요소들이 같은 타입의 요소들을 가지고 있어서 이 요소들을 합쳐 하나의 AsyncSequence를 만든다.



# Clock, instant, duration


# Algorithms using time


# Collections
