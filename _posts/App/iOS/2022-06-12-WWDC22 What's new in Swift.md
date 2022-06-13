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

쓰레드 안전성을 위해서도 마찬가지. 예상치 못한 동작을 하는 concurrency 버그를 예방하는 것이 목적이다. Swift에서 위 코드와 같이 작성하는 것을 막는 이유는 `removeLast()` 메서드가 배열에 숫자를 더한 다음 실행될 지 알 수 없기 때문이다. 즉 Actor와 같은 걸로 접근을 동기화하지 않고 백그라운드 task에서 배열을 수정하는 것이 안전하지 않다는 뜻이다. Actor는 data race를 제거하기 위한 첫 주요 단계였다.

#### Data race safety

<img width="390" alt="image" src="https://user-images.githubusercontent.com/41438361/173265015-d3e1d91c-f5c1-419f-80d7-265038fc3452.png">

새로 개선된 concurrency model을 적용했다. Actor를 하나의 독립된 섬이라고 생각해서 concurrency 영역에서 고립되었다고 생각하면 된다. 만약 다른 쓰레드가 독립된 actor를 통해 저장된 데이터를 처리하고 싶다면 어떻게 될까?

=> Eliminate data races using Swift Concurrency 세션에서 더 자세히 확인할 수 있다.

<img width="583" alt="image" src="https://user-images.githubusercontent.com/41438361/173265215-d1546357-9812-437d-ac24-506edcd93238.png">

Data 안전에서 쓰레드 안전으로 가는 건 Swift 6의 목표라고 한다. 이를 위해 작년의 concurrency model을 앞선 기능으로 향상시켰고, data race가 발생할 가능성을 확인하는 새로운 opt-in safety check다.

### Distributed actors

<img width="573" alt="image" src="https://user-images.githubusercontent.com/41438361/173265437-259acd45-8b09-4974-a601-d9ff90032190.png">

Distributed actors는 위의 actor(섬)이 다른 기기에 위치하고, 이 사이에 네트워크를 두는 것이다. 이를 통해 분산된시스템을 간단히 개발할 수 있다.

<img width="998" alt="image" src="https://user-images.githubusercontent.com/41438361/173265655-f20c703a-ecfd-4e09-9c2d-8c5964d5dcd9.png">

위 코드의 distributed actor는 다른 기기에서 호출될 수 있는 actor. `distributed` 키워드는 원격 기기에 위치할 수 있는 actor에서 호출될 수 있는 함수 앞에 붙일 수 있다.

`endOfRound` 메서드 내에서는 로컬/원격 플레이어의 `makeMove` 함수를 호출하는데 distributed actor 호출은 네트워크 에러 때문에 실패할 수 있다. 만약 실패할 경우 에러를 던지기 때문에 `tray`, `await` 키워드를 actor 밖에서 함수를 호출할 때 붙여야 한다.

<img width="374" alt="image" src="https://user-images.githubusercontent.com/41438361/173265987-becb7c33-e6c8-42c7-8dfb-e23e4693d638.png">

오픈 소스 Distributed Actors : 서버 사이드에 초점을 맞춤. 

=> Meet distributed actors in Swift 세션에서 자세한 내용 확인 가능

### Async algorithms

<img width="431" alt="image" src="https://user-images.githubusercontent.com/41438361/173266130-9458f191-ac05-4d66-a734-6c745fb9dce3.png">
<img width="498" alt="image" src="https://user-images.githubusercontent.com/41438361/173266169-a80694c1-b2c3-4ca8-be8d-b1e9118a4281.png">

AsyncSequence로 작업할 때 사용할 수 있는 새로운 오픈 소스 알고리즘 제공.

=> Meet Swift Async Algorithms 세션

### Concurrency optimizations

<img width="358" alt="image" src="https://user-images.githubusercontent.com/41438361/173266512-084a534c-bd38-483b-b7ef-6e3af65e3bc2.png">

* actor prioritization : actor는 우선순위가 가장 높은 작업을 먼저 실행한다.
* Priority-Inversion avoidance : 중요도가 낮은 작업이 우선순위가 높은 작업을 block 시킬 수 없다.

### Using the Swift concurrency instruments

<img width="1660" alt="image" src="https://user-images.githubusercontent.com/41438361/173266605-2b80931b-48bb-499b-b13d-47f9cf3336b5.png">

Instruments의 Swift Concurrency view에서 성능 이슈를 확인할 수 있다. 

1. 상단 : 동시에 실행되고 있는 task의 수, 특정 시간 이후 생성된 전체 task.
2. 하단 : Task 간의 부모-자식 관계를 나타낸 그래프

=> Visualize and optimize Swift concurrency 세션에서 확읺 가능

# Expressive Swift

## Optional Binding

<img width="1659" alt="image" src="https://user-images.githubusercontent.com/41438361/173267126-35934dd2-ea10-4641-881c-b27aec624ba6.png">

옴셔널 바인딩을 할 때 바인딩 할 이름을 쓰지 않고 이제 변수만 써줘도 된다.

## Closure type inference

<img width="1150" alt="image" src="https://user-images.githubusercontent.com/41438361/173267301-401d33b7-e259-4300-ba9a-1f4b7b3c87dd.png">

Swift는 한 줄 클로저 내의 코드에 따라 어떤 타입이 리턴될지 알 수 있다. `parseLine` 함수는 `MailmapEntry`를 리턴하고 있기 때문에 `entires`가 `MailmapEntry`의 배열이 될 것임을 예상할 수 있다. 위 코드처럼 여러 줄의 복잡한 클로저에서도 똑같이 동작한다. 클로저의 result 타입을 명시하지 않고 do-catch, if-else 등을 사용할 수 있다.

## Permitted pointer conversions

<img width="1682" alt="image" src="https://user-images.githubusercontent.com/41438361/173267744-ac2c7013-1f73-46aa-93e0-1755eb973bdf.png">

Swift는 type과 memory safety를 굉장히 중시한다. 따라서 다릍 포인터 타입 간에 자동으로 변환되는 것을 금지한다. 하지만 C는 어떤 경우에는 허용된다.

<img width="1251" alt="image" src="https://user-images.githubusercontent.com/41438361/173268122-db67e459-abbb-44cf-ac9b-edd9a234beea.png">

그래서 C API를 Swift에서 사용할 때 문제가 될 수 있다. C에서 포인터 타입이 같지 않은 걸 C에서 자동 변환을 통해서 해결하기 위해 디자인했지만, Swift에서는 이를 에러로 처리하기 때문이다. 

<img width="1641" alt="image" src="https://user-images.githubusercontent.com/41438361/173268293-b696d83d-93e2-49f4-a233-5eb9a437442c.png">

Swift에서 한 타입의 포인터를 다른 타입으로서 여기고 접근하는 것이 매우 위험하기 때문에 어떤 작업을 할 지 명시적으로 나타내야 한다. 

*그래서 문제가 뭔데?* : Swift는 type safety를 중시하면서 C 친화적인 코드에 굉장히 쉽게 접근하는 것도 중요하게 생각하기 때문이다. 그래서 이런 C 함수를 사용할 때 쉽게 사용하게 만들고 싶었음.

그래서 Swift는 이제 imported 함수와 메서드 호출에 별도의 규칙이 생겼다. C에서는 되는데 Swift에서는 안되는 포인터 변환을 허용한다.

## String processing

<img width="1707" alt="image" src="https://user-images.githubusercontent.com/41438361/173268978-551e3060-365e-4d07-b5db-66e5e93302c7.png">

문자열을 파싱하는 함수. 복잡하기 때문에 이를 대체.

<img width="1088" alt="image" src="https://user-images.githubusercontent.com/41438361/173276329-a91222ea-8b06-42e7-b31a-db782606ec44.png">

Swift 5.7에서는 regex를 사용할 수 있다.

<img width="1361" alt="image" src="https://user-images.githubusercontent.com/41438361/173276623-b5853cd0-2c5c-407b-b140-5db8f49d97a9.png">

이 regex literal은 바로 이해하기가 어렵다. 이 규칙들을 심볼 대신에 단어로교체한다면 이해하기가 쉬워진다.

<img width="1578" alt="image" src="https://user-images.githubusercontent.com/41438361/173276740-9fd04d54-a52d-47f0-95ba-d24872c2f2bc.png"><img width="690" alt="image" src="https://user-images.githubusercontent.com/41438361/173276756-9625ee47-5d4a-4f60-b98c-6df1e978e6fb.png">

이를 조합해서 SwiftUI와 같은 형태로 표현할 수 있다. RegexBuilder 라이브러리를 통해 이런 SwiftUI 스타일의 언어를 regex에 사용할 수 있다.

=> Meet Swift Regex, Swift Regex: Beyond The Basics 세션에서 더 알아볼 수 있다.

## Generic code clarity

<img width="930" alt="image" src="https://user-images.githubusercontent.com/41438361/173278366-01a2c8d3-6a6f-4e10-bf4a-e337f1fcffa9.png">

프로토콜 MailMap, 이를 채택하는 두 구조체 HasedMailMap, OrderedMailmap.

<img width="1576" alt="image" src="https://user-images.githubusercontent.com/41438361/173278468-8b9bc9f0-834a-44b2-bbc7-cb6a8457784e.png">

Mailmap 프로토콜을 사용하는 두 방법이 있다. 이 두 버전의 차이를 말하기 어려운데, 같은 문법을 사용하고 있기 때문이다. 위 함수의 "Mailmap"과 밑 함수의 "Mailmap"은 다른 의미를 갖는다.

<img width="1671" alt="image" src="https://user-images.githubusercontent.com/41438361/173278673-040fafb1-223d-497f-9005-65890c79b2fd.png">

프로토콜 이름을 inheritance list, generic parameter list, generic conformanve constraint, opaque result type으로 작성하면 **이 프로토콜을 준수하는 인스턴스**라는 뜻이다.

하지만 varialbe type, generic argument, ganeric same-type constraint, function parameter, result type에서 프로토콜 이름을 쓰면 **이 프로토콜을 준수하는 인스턴스를 포함한 박스**라는 뜻이 된다.

<img width="1658" alt="image" src="https://user-images.githubusercontent.com/41438361/173279726-535107f4-1fe4-46ce-a64d-70d68a590bbd.png">

후자의 경우 더 많은 공간을 차지하고, 연산하는데 더 많은 시간이 걸린다. 하지만 표기 자체는 전자 후자 모두 `Mailmap`으로 표시하기 때문에 내가 어떤 걸 사용하고 있는지 구별하기 힘들다. 

### any keyword

<img width="1637" alt="image" src="https://user-images.githubusercontent.com/41438361/173279883-71e40f45-c377-43a1-b426-caf84f177f05.png"><img width="1624" alt="image" src="https://user-images.githubusercontent.com/41438361/173280043-14668c2a-ac0c-4ec0-a3b8-3b9f9f6e9b03.png">

Swift 5.7에서는 이제 conforming type을 포함하는 박스를 사용할 때 `any` 키워드를 붙여야 한다. 이를 명시적으로 붙이지 않으면 에러 메세지가 뜰 것이다.

<img width="1565" alt="image" src="https://user-images.githubusercontent.com/41438361/173280117-b83839ce-caa7-4361-84ea-2aaf40267232.png">

그래서 위에서 봤던 두 타입의 메서드를 위와 같이 쓸 수 있다.

* 1 : Mailmap을 제네릭 타입을 받는다.
* 2 : Mailmap을 any 타입으로 받는다. 

<img width="1283" alt="image" src="https://user-images.githubusercontent.com/41438361/173280254-c20785e9-5fd5-4c2a-830b-4a3d526877cf.png">

에러메세지에서도 어떤 일이 발생하는 지 더 명확하게 표현 가능. 위 코드에서는 any Mailmap을 generic Mailmap 파라미터로 전달하고 있다.  원래는 이때 에러메세지로 "Mailmap cannot conform to itself"라는 역설적인 문장의 형태로 출력됐는데, any의 개념을 통해 문제 상황을 명확하게 표현가능하다.

### Pass to generic arguments

<img width="886" alt="image" src="https://user-images.githubusercontent.com/41438361/173280290-307e0c5d-6dd8-4f29-918a-485f9b5d19e7.png">

문제는 mailmap을 포함한 박스(any)가 Mailmap 프로토콜을 준수하지 않는다는 것이다. 그래서 그림과 같이 큰 박스가 파라미터에 맞지 않고, 내부에 있는 적합한 mailmap을 꺼내 이를 파라미터에 전달해야 한다. 그리고 위와 같이 간단한 상황에서는 이제 Swift가 알아서 박스를 열고 내부의 인스턴스를 꺼내 제네릭 파라미터에 전달하기 때문에 에러 메세지를 거의 보지 않게 된다.

### Supports Self and associated types

<img width="1700" alt="image" src="https://user-images.githubusercontent.com/41438361/173280725-73cb3ae7-cfef-40dd-aebd-5983bea953c8.png">

또한 이전에는 프로토콜은 self 타입을 사용하거나 연관 타입을 가지고 있을 때, Equatable과 같은 프로토콜을 채택할 때 any 타입으로서 사용되지 못했다. 이제 Swift 5.7에서 이런 에러는 뜨지 않는다. 

### Primary associated types

<img width="1586" alt="image" src="https://user-images.githubusercontent.com/41438361/173281029-62c1ecf7-3615-41cc-9bdb-c0ab91dfbbe4.png">

또한 Collection과 같이 복잡한 프로토콜도 any 타입으로서 사용될 수 있다. "primary associated types" 신기능 : element type을 명시할 수 있다.

<img width="1444" alt="image" src="https://user-images.githubusercontent.com/41438361/173281175-2a86bb9e-a3bf-40b5-95a4-6aa6b86aa19f.png">

위 코드에서 Element과 같이 사용자들이 사용하면서 중요하게 여기는 연관 타입이 있다면 프로토콜 이름 뒤에 프로토콜의 주요 연관 타입을 제약할 수 있다.

<img width="1211" alt="image" src="https://user-images.githubusercontent.com/41438361/173281415-bde2e48a-5445-4d58-80af-6d7f88461ee9.png">

위 코드에서 Element과 같이 사용자들이 사용하면서 중요하게 여기는 연관 타입이 있다면 프로토콜 이름 뒤에 프로토콜의 주요 연관 타입을 제약할 수 있다.

기존에 사용되던 AnyCollection은 type-erasing wrapper로 any 타입과 같이 동작하게 하기 위해 작성된 구조체다. 

* AnyCollection : boilerplate code로 가득찬 struct
* any type : 같은 기능을 하는 내장된 언어 기능

### 주의사항

<img width="1306" alt="image" src="https://user-images.githubusercontent.com/41438361/173281821-b1d385e7-4e8a-43b3-9804-d71ede4faa22.png">

Mailmap이 Equatable을 준수한다고 해도 any Mailmap에 == 연산자를 상용할 수 없다. 이유는  두 mailmap이 concrete type을 가지도록 요구되는데 두 개의 any Mailmap을 사용할 때 이게 보장이 되지 않기 때문이다. 

그래서 이런 상황에서는 제약이 따를 수 있기 때문에 대부분의 경우에는 대신에 제네릭을 사용하라고 권고하고 있다.

<img width="1608" alt="image" src="https://user-images.githubusercontent.com/41438361/173282102-fb1a0cc2-748e-4fe6-a1b5-dc24d219c59e.png">

위 코드에서 제네릭을 작성하는 것은 any 버전을 작성하는 것보다 복잡하다. 

<img width="1603" alt="image" src="https://user-images.githubusercontent.com/41438361/173282893-8102ba11-7cb0-4583-94ec-ba0cd88cbb0f.png">

만약 generic parameter가 한 곳에서만 사용된다면 some 키워드를 사용해서 짧게 표현할 수 있다. 

<img width="1608" alt="image" src="https://user-images.githubusercontent.com/41438361/173283024-817babff-195c-4c54-b613-ae1fb39fbd93.png">

그리고 이제 primary associated type을 지원하기 때문에 더 이해하기 쉽게 모든 collection 타입의 mailmap을 받을 수 있게 작성할 수 있다. 

=> Embrace Swift generics, Design protocol interfaces in Swift 세션에서 더 알아볼 수 있다.
