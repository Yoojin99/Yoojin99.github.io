---  
layout: post  
title: "[iOS] - WWDC22 What's new in Swift"  
subtitle: ""  
categories: app
tags: app-ios wwdc
comments: true  
header-img: 

---  

# Swift Package Manager

## TOFU 

<img width="1330" alt="image" src="https://user-images.githubusercontent.com/41438361/173230872-e197b5f0-652a-459d-974c-0ff241cf7795.png">

Trust On First Use. 패키지가 처음 다운로드 됐을 때 패키지의 지문을 기록하는 보안 프로토콜. 이후 다운로드 할 때마다 지문을 다시 확인해서 지문이 다를 경우 에러를 발생시킨다.

## Package plugins

=> Meet Swift Package plugins, Create Swift Package plugins 세션에서 더 많은 내용을 확인할 수 있다.

### Command plugins

<img width="1724" alt="image" src="https://user-images.githubusercontent.com/41438361/173231040-574b9f4d-a303-4b08-9d93-12dc8063599d.png">

모든 오픈 소스 도구를 Xcode와 Swift Package Manager에서 사용할 수 있다.

Command plug-ins : 오픈 소스 툴과 Swift Package Manager을 연결지어 준다.

<img width="606" alt="image" src="https://user-images.githubusercontent.com/41438361/173231134-efbe2a9a-a946-4eee-b45b-fd917b73c1f7.png">

Plug-ins은 간단한 Swift 코드. `CommandPlugin` 프로토콜을 만족하는 구조체를 생성해서 plug-in을 정의할 수 있다. 그리고 특정 기능을 하는 함수를 추가하면 된다.

<img width="473" alt="image" src="https://user-images.githubusercontent.com/41438361/173231245-52e2917a-f0ac-4b74-88b7-09865e86f42c.png">

플러그인을 정의하면 Swift PM 커맨드 라인 인터페이스와 Xcode의 메뉴 목록에서 사용할 수 있다.

### Build tool plugins

<img width="809" alt="image" src="https://user-images.githubusercontent.com/41438361/173231319-f75ba26b-99ca-41ee-9501-77d3e37e749f.png">

빌드 중간에 특정 작업을 할 수 있게 해주는 패키지. Build tool plug-in을 구현하면 sandbox에서 실행시킬 수 있는 명령어를 생성한다.

* command plug-in : 어느 때든 실행할 수 있고 패키지 내의 파일을 바꿀 수 있는 명시적인 권한을 가질 수 있다.
* build tool plug-in : 특정 타입의 파일을 처리하거나 코드를 생성하기 위해 사용됨

<img width="1715" alt="image" src="https://user-images.githubusercontent.com/41438361/173231475-1361a885-c897-4a96-a02a-bdd19ade69c8.png">

Build tool plug-in의 패키지 구조. `plugin.Swift`는 패키지 plug-in 타겟을 구현한 Swift 스크립트다. Plug-in은 Swift 실행 가능 파일로 간주된다.

<img width="556" alt="image" src="https://user-images.githubusercontent.com/41438361/173231517-4dd473bf-7505-4adc-a65e-afaa807c485a.png">

빌드 시스템이 어떤 커맨드를 실행하고 어떤 결과를 낼지를 정의하는 빌드 커맨드의 집합과 함께 플러그인을 구현할 수 있다.

## Module aliasing

<img width="469" alt="image" src="https://user-images.githubusercontent.com/41438361/173231642-7d975e6c-8669-4146-a0d4-f6f64a7a88aa.png">

패키지를 확대할 때 모듈 충돌이 일어날 수 있는데, 이는 두 개의 별개의 패키지가 같은 이름의 모듈을 정의할 때 발생한다.

Swift 5.7에서는 module disambiguation을 소개한다.

<img width="490" alt="image" src="https://user-images.githubusercontent.com/41438361/173231716-7024d4f5-5dd3-4a31-ad28-71bc87cbefe1.png">

두 패키지가 Logging 모듈을 정의하고 있기 때문에 충돌한다. Package manifest의 Dependencies 섹션에 moduleAliases 키워드를 추가해서 이를 해결한다.

<img width="311" alt="image" src="https://user-images.githubusercontent.com/41438361/173231728-f0f38a41-13f4-445b-92c6-bbb5128fbbbc.png">

이를 통해 같은 이름을 가진 모듈을 구별하기 위해 다른 이름을 사용할 수 있다.

# Performance improvements

## Build-time improvements

### Swift driver settings

드라이버는 현재 별도의 실행가능파일이 아니라 Xcode 빌드 시스템 안에서 프레임워크로 사용될 수 있다. 이를 통해 병렬 컴퓨팅과 같은 작업이 가능하도록 빌드를 빌드 시스템과 더 가깝게 연결 지을 수 있었다.

<img width="1409" alt="image" src="https://user-images.githubusercontent.com/41438361/173231857-a3cb23ed-b527-4d95-8da7-9eb895f6972a.png">

위 표는 빌드가 얼마나 빨라졌는지 보여주는 표.

=> Demystify parallelization in Xcode builds 세션에서 더 많은 내용을 확인할 수 있다.

### Faster type checking of generics

제네릭 시스템의 핵심 부분을 재구현해서 타입 체커 성능을 향상시켰다.

<img width="575" alt="image" src="https://user-images.githubusercontent.com/41438361/173232000-8b7fc81f-aa24-44e2-924f-0f9fd0f09db3.png">
<img width="595" alt="image" src="https://user-images.githubusercontent.com/41438361/173231990-a036a71c-fc97-40e4-9179-6c74c592c168.png">

기존에는 연관된 프로토콜이 많을 수록 시간과 메모리 사용량이 급수적으로 늘어날 수 있었다. Swift 5.7에서는 기존에 빌드가 17초 걸리던 코드를 이제 1초 미만으로 줄일 수 있다!!

## Runtime improvements

기존에는 매번 앱을 시작할 때마다 iOS에서 프로토콜 확인을 4초 가량 했다. 앱을 실행할 때마다 프로토콜이 매번 연산되어야 했기 때문에 프로토콜을 많이 추가할 수록 실행 시간이 길어졌는데, 이제 캐시된다고 한다.

=> Improve app size and runtime peformance 세션에서 자세한 내용을 확인할 수 있다.

# Concurrency updates

## Improvements to the concurrency model

<img width="464" alt="image" src="https://user-images.githubusercontent.com/41438361/173232355-978af2f5-f7f8-43c6-a6bc-8f9f5e0b362a.png">

Concurrency가 중요하고 핵심적인 기능이기 때문에 iOS 13, macOS Catalina에서 사용할 수 있게 됐다.

## Extensions to the model

<img width="307" alt="image" src="https://user-images.githubusercontent.com/41438361/173232416-a6863692-53ad-489e-adfb-65efba29c582.png">

### Data race avoidance

<img width="629" alt="image" src="https://user-images.githubusercontent.com/41438361/173232486-4e2dc55f-346c-461e-b31e-08a16c3f01b1.png">

Swift에서 메모리 안전은 중요한 기능 중 하나다. 위 코드와 같이 배열을 수정하는 도중 배열 원소의 개수에 접근하는 것이 안전하지 않기 때문에 Swift에서는 이런 작업을 할 수 없다.

<img width="381" alt="image" src="https://user-images.githubusercontent.com/41438361/173232581-f093cd0f-afa8-4c91-831c-b43635c47253.png">
<img width="305" alt="image" src="https://user-images.githubusercontent.com/41438361/173232588-d7d37d47-171e-45bd-a4cd-1faf7a31ab59.png">



### Distributed actors

### Async algorithms

Sdfsdsadfasdf
