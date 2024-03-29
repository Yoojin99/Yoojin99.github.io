---  
layout: post  
title: "[iOS] - Combine 프레임워크 WWDC 2019 정리"  
subtitle: ""  
categories: app
tags: app-ios combine
comments: true  
header-img: 

---  
  
> WWDC 2019에서 소개된 Combine이 무엇이고, 어떻게 사용하는 건지 알아보자.  

---

Combine은 프레임워크로, WWDC 2019에서 처음 소개되었다. WWDC 2019 "Introducing Combine" 영상에 나온 개요를 보면 아래와 같이 나와있다.

<img width="614" alt="image" src="https://user-images.githubusercontent.com/41438361/156953885-f1f4d979-4474-45e9-85c0-e38a8dc65218.png">

시간에 따라 값들을 처리하기 위한 하나의 통합된 선언형 프레임워크라고 한다. Combine을 통해 네트워킹, key value observing, 알림, 그리고 callback을 단순화할 수 있다.
즉 비동기적인 작업을 단순화한 프레임워크라고 생각하면 될 것 같다. 

이 글에서는 먼저 Combine에 대해 대략적으로 알기 위해 앞 부분에 WWDC 2019의 두 영상을 먼저 정리했다. [Introducing Combine](https://developer.apple.com/videos/play/wwdc2019/722/?time=1098)
과 [Combine in Practice](https://developer.apple.com/videos/play/wwdc2019/721)로, 첫 번째 영상에서는 Combine의 등장 배경, Combine의 핵심 개념인 Publisher, Subscriber, Operator에 대해 설명하고 있다. 두 번째 영상에서는 더 구체적인 부분과 실제 어떻게 코드로 사용하는지를 보여준다.

# Introducing Combine (WWDC 2019)

## Combine 등장 계기

위에서 얘기한 WWDC 2019 "Introducing Combine"에서는 Combine이 등장하게 된 배경에 대해 얘기하고 있다. Combine 자체에 대해 중요한 얘기는 아니니
이 부분은 읽지 않아도 된다. >> 후에 Asynchronous programming이라는 별도의 포스트로 옮길 예정이다.

### Asynchronous programming

먼저 Asynchronous programming(비동기적 프로그래밍)에 대해 알아야 한다. Asynchronous programming은 뭘까?

우리는 흔히 API를 호출해서 서버로부터 응답값을 받는다. 하지만 함수를 호출해서 API를 통해 응답값을 받기까지는 시간이 걸린다. 바로 여기서 Asynchronous programming을 통해
**함수를 호출한 시점과, 응답값을 받는 시점 사이의 지연을 적절히 처리**할 수 있다. 

Asynchronous programming이 없다면? API를 호출하고 응답값을 받기까지 지연될 동안 앱은 화면을 로딩하는 데 그만큼 오랜 시간을 쓸 것이다. 화면을 로딩한다는 것은
예를 들어 사용자가 로그인했는데, 데이터베이스로부터 사용자의 데이터를 받기까지 기다리는 것도 될 수 있고, 새로운 화면에 진입했을 때 데이터가 로딩되기까지 
기다리는 경우가 될 수 있다. 한 번 쯤은 다 겪어봤을 테지만 이런 상황을 겪게 되면 앱이 느리다, 서버가 느리다와 같은 부정적인 느낌을 받게 된다.

그래서 Asynchronous programming은 **사용자가 앱을 이용하면서도, 이런 로딩과 관련된 작업들을 백그라운드에서 처리해서 사용자의 경험을 향상**시켜준다.

조금 더 구체적인 예시를 들어보자. Asynchronous programming은 사용자가 앱을 사용하면서 데이터와 관련된 비동기적 처리를 백그라운드에서 처리해서 사용성을 향상시켜준다고 했다.
사용자가 데이터를 생성해서 데이터베이스로 전송하는데는 오랜 시간이 걸릴 수 있다. 입금 폼을 작성한다고 해보자. 입금 폼에 엄청 많은 내용을 입력해서
전송을 눌렀는데 전송이 완료될때까지 다음 화면으로 넘어갈 수 없고 계속 로딩 창만 봐야 한다면 분명 화가 날 것이다. Asynchronous programming을 사용해서, 사용자는 데이터를 데이터베이스에 전송하는
함수가 실행되는 동안 다른 화면으로 이동할 수 있게 된다. 많이 사용하는 인스타그램에서도 asynchronous programming을 볼 수 있다. 사진이 로딩되어서
화면에 뜰 때까지 스크롤을 못하고 가만히 기다리지 않고, 사용자는 다른 화면으로 계속 이동할 수 있다. 

### Asynchronous programming이 동작하는 방법

Asynchronous programming이 비동기 작업을 적절히 처리해서 사용성을 높인다는 것은 알겠고, 어떻게 동작하는 걸까? 이를 알기 가장 쉬운 방법은 Synchronous programming과 비교하는 것이다.

#### Synchronous programming

Synchronous programming은 엄격한 순서를 가진다. 코드가 synchronous 하게 실행된다는 거는 알고리즘의 각 단계따라 순서대로 실행된다는 의미다.
순서대로 실행된다는 거는 이전 단계가 끝날 때까지 다음 단계를 시작하지 못한다는 것이다.

요리 레시피를 예를 들어 생각하면 이해하기 쉽다. 계란 말이를 만들기 위해서는 먼저 달걀을 깨고, 프라이팬에 기름을 붓고, 계란을 프라이팬에 넣고 말아야 한다.
그런데 달걀을 깨기도 전에 계란을 프라이팬에 넣고 마는 것은 불가능하다. 즉 각 단계는 꼭 순차적으로 실행되어야 한다. 

### Asynchronous programming

Asynchronous programming 은 반대로 요리를 예로 든다고 하면 여러 사람이 동시에 한 작업씩 맡아 처리하게 하는 방식이다. 예를 들어 한 사람은 계란을 까고,
한 사람은 기름을 팬에 두르고, 한 사람은 계란만 마는 것이다. 그래서 여러 처리를 동시에 할 수 있다. 

### Asynchronous functions

Asynchronous function은 front end 앱에서 종종 사용되고, 특히 독립적이고 처리할 양이 많은 IO 작업에서 사용된다. Front end 앱들은 asynchronous 함수를 사용해서
앱의 동작 플로우를 향상시킨다.

Backend에서도 많은 작업들을 처리하거나 많은 네트워크 호출을 처리하기 위해 asynchronous function을 사용한다. Backend에서 asynchronous programming은 컴퓨터가
더 많은 작업들을 더 빠르게 처리할 수 있게 해준다. 응답 시간이 정해지지 않았으면서 결과를 처리하는 함수들을 많이 호출한다. 예를 들어 웹 스크래핑을 해서 데이터베이스에 저장한다고 해보자.
이 작업은 어떤 순서로 스크래핑 결과가 디렉토리에 저장되어야 하는지는 상관 없고, 단지 저장할 파일 이름만 필요하다. 이런 경우 asynchronous 하게 처리할 수 있다.

### Common use cases

Asynchronous function을 흔히 API를 호출하기 위해 많이 사용한다. 네트워킹 시간과 응답 시간이 확실하지 않기 때문에 asynchronous function을 사용해서
"웹사이트나 REST API를 통해 데이터를 가져오고, 데이터가 오면 이 데이터를 내 스크립트로 가지고 와라"라는 작업을 처리하는 것이다. Async function은 아래의 목적으로 사용된다.

1. API와 상호작용하기 위해
2. 앱의 UX를 천천히 만들기 위해

1번은 알겠는데, 2번은 무엇일까? UX를 천천히 한다는 것은 사용자의 행동에 딜레이를 거는 것이다. 왜 이런 작업이 필요한가? 그 이유는 컴퓨터는 작업을 굉장히
빠른 시간에 처리할 수 있기 때문에 사용자에게 충격을 줄 수 있기 때문이다. 솔직히 나는 빠르면 빠를수록 좋은게 아닌가 싶지만 디자이너들은 의도적으로 앱의 속도를
늦추기도 한다고 한다. 예를 들어 메세지 앱에서 사용자가 입력 중이면 로딩 표시가 뜬다. 사용자가 바로 메세지를 즉각적으로 보내면 이런 로딩 표시는 필요가 없지만,
메세지를 받는 사람이 이 로딩 표시를 봄으로써 무언가가 일어나고 있음을 인지하고 앱을 사용하기가 더 편해진다는 것이다. 화면에 있는 요소가 애니메이션으로
나타나고 사라지는 것도 asynchronous하게 작동한다. 왜냐하면 나타나거나 사라지는 동작을 시간에 걸려 처리하는데, 이 작업이 끝날 때까지 다른 동작을 못하는 것이 아니라
다른 함수들이 백그라운드에서 처리될 수 있기 때문이다.

### When to use asynchronous functions

Asynchronous도 적재적소가 있다. Asynchronous는 프로그램에 복잡성을 더하고, 코드를 읽기 어렵게 만든다. 주니어 개발자들은 종종 코드가 런타임에
잘 작동하기 위한 보호의 목적으로 async function을 남발하는 경우가 있다. 일반적으로 async function은 사용하는 곳은 아래와 같다.

* 좋음 : 작업이 시간이 걸리고, 많이 반복되는 곳
* 나쁨 : 단순함이 요구되는 곳

---

다시 본론으로 돌아와서, combine이 등장하게 된 계기에 대해 알아보자. WWDC에서 예시로 든 앱을 보자.

<img width="1050" alt="image" src="https://user-images.githubusercontent.com/41438361/156997696-c745ba34-b81e-4094-b900-5f3e6d7854ca.png">

사용자 정보를 입력해서 회원 가입을 해야 하는 폼이다. 여기에서도 굉장히 많은 asynchronous 작업이 일어난다.

1. 사용자 이름을 입력할 때 Target/Action을 사용해서 사용자가 타이핑하고 있음에 대해 알림을 받는다. 추가로 사용자가 타이핑을 잠시 멈춘 후 타이머를 사용해 잠시 기다려서 서버에 네트워킹 요청을 과도하게 보내지 않도록 한다.
2. KVO를 사용해서 이런 asynchronous operation 후에 업데이트 된 내용을 감지한다.
3. URL session request로 요청한 것에 응답값을 받아 UI를 업데이트 한다.

위의 예제를 보면 다양한 asynchronous interface를 사용하고 있다.

<img width="791" alt="image" src="https://user-images.githubusercontent.com/41438361/156998482-b6f77f47-4553-4aee-851d-91846b1c92e6.png">

위에서 봤듯이, Cocoa SDK에는 Target/Action, NotificationCenter와 ad-hoc callback과 같이 많은 asynchronous interface가 있다.
이런 API들은 closure나 completion block을 가지게 된다. 위에 나온 모든 인터페이스들은 중요하면서도 다른 use case를 가디고 있는데,
이 모든 것을 한데 합치려면 꽤 복잡할 수밖에 없다.

그래서 Combine이 등장했다. Combine은 위의 인터페이스들을 완전히 대체하기 보다는 인터페이스들의 공통점을 기반으로 생겨났다.

## Combine Feature

Combine은 맨 처음에도 언급했듯이, **시간에 따라 값들을 처리하는 하나의 통합된 선언형 API**이다. WWDC 2019에서 Combine의 특징으로 네 가지를
말하고 있다. 하나씩 보자.

<img width="588" alt="image" src="https://user-images.githubusercontent.com/41438361/157000926-05279252-5705-4b0b-badc-0a9415f8384d.png">

### 1. Generic

Combine은 Swift로 작성되어 있어서 **Generic과 같은 Swift의 여러 장점들을 사용할 수 있다**. 특히 Generic은 개발자가 작성해야 하는 boilerplate 코드의
양을 줄여준다. 즉 asynchronous behaviour를 제네릭하게 한 번 작성해서 다양한 타입의 asynchronous interface에 적용할 수 있는 것이다.

### 2. Type Safe

Combine은 type safe해서, 런타임이 아닌 **컴파일 시간에 에러를 확인할 수 있다.** 이거는 그냥 Swift로 작성되었기 때문에 1번과도 같이 생각할 수 있는 특징인 것 같다.

### 3. Composition First

Combine을 디자인할 때 주요 포인트는 "composition first"였다고 한다.
이게 무슨 말이냐면, 핵심 개념은 단순해서 이해하기 쉬운데, 이를 합쳐서 각 부분을 합친 것 이상의 무언가를 만들 수 있다는 소리다. 솔직히 무슨 말인지 잘 모르겠지만
핵심 개념을 이해하기 쉬우면서 요소들을 조합해서 더 복잡한 무언가를 쉽게 만들 수 있게 디자인 했다는 소리인 것 같다.

### 4. Request Driven

Combine은 request-driven(request 기반)이어서 앱의 메모리 사용과 성능을 더 잘 관리할 수 있게 해준다고 한다. 이것도 어떻게 가능하게 해준다는 건지는
조금 더 공부해야 알 수 있을 것 같다.

## Key Concepts

Combine의 핵심 개념은 3가지다. 

<img width="701" alt="image" src="https://user-images.githubusercontent.com/41438361/157001250-6e60a8d8-dc90-44b3-9a17-e75fc7db27eb.png">

### Publisher

<img width="1082" alt="image" src="https://user-images.githubusercontent.com/41438361/157002140-edeea059-913b-4f1e-a3dc-e1c5a648cc59.png">

Publisher는 Combine API의 선언적인 부분이다. 얘네는 이름에서 유추할 수 있듯이 뭔가를 발행하는 애들로, **값과 에러가 어떻게 생성되는 지를 정의한다.**
어떻게 생성되는지를 정의한다는 거는 값과 에러를 직접 생성하는 것은 아니라는 의미다.

즉 설명하는 역할을 가지고 있고, 이는 **값 타입**으로 Swift에서는 struct를 사용한다고 할 수 있다.

Publisher는 **Subscriber를 등록하는 것을 허용한다.** 이를 통해 시간에 따라 값들을 받게 된다.

Publisher 프로토콜을 자세히 보자.

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157002438-9aa92ae8-aaf2-4ffd-b980-f5d50d574813.png">

보면 두 개의 associated type을 가지고 있다.

1. Output : 생성하는 값 타입
2. Failure : 생성하는 에러 타입. 다만 publisher가 에러를 생성하는 게 불가능하다면 이 연관 타입에 never를 넣으면 된다.

그리고 Publisher는 하나의 key function을 가지고 있다. `subscribe` 함수는 Subscriber의 input이 Publisher의 Output과 일치해야 하고,
Subscriber의 Failure가 Publisher의 Failure와 일치해야 함을 요구하고 있다. 이는 생각해보면 당연한게 Publisher가 발행한 걸 Subscriber가
받아야 하니 publisher의 output 타입이 subscriber의 input 타입과 같아야 할 것이다.

<img width="1781" alt="image" src="https://user-images.githubusercontent.com/41438361/157003226-ea78d4b8-99c7-448a-93d3-577ae69f54da.png">

위는 NotificationCenter를 위한 새로운 Publisher로, 보면 struct로 되어 있고, Output, Failure 타입이 정의되어 있다. 그래서 핵심은
NotificationCenter를 Combine으로 대체하는 것이 아니라 Combine을 NotificationCenter에 적용한다는 것이다.

### Subscriber

<img width="990" alt="image" src="https://user-images.githubusercontent.com/41438361/157137786-1f590378-7d3b-4f72-83e6-4770a56df580.png">

Subscriber는 Publisher의 짝이라고 할 수 있다. 얘네는 **값과 Publisher가 제한적일 때는 completion을 받는다**고 한다.

Subscribe는 주로 행동하는 주체고 값을 전달 받아 상태를 변경하기 때문에 **reference type을 사용하고, 이는 Swift에서 클래스를 사용**한다는 말이 된다.

Subscriber의 프로토콜을 자세히 보자.

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157138262-c241f6bb-6a50-4e6a-8690-f4add5bfd585.png">

얘네도 Publisher과 똑같이 두 개의 연관 타입 Input, Failure를 가지고 있다. 그리고 Subscriber가 Failure를 받을 수 없다면, Never 타입을 사용하면 된다.

그리고 세 개의 key function이 있다. 

1. subscription 받기. subscription은 Subscriber가 Publisher에서 Subscriber로 이동하는 데이터 플로우를 어떻게 제어하는지를 나타낸다. 
2. Input 받기
3. 연결된 Publisher가 제한적이라면 Completion(Finished/Failure)를 받을 수 있다.

Subscriber도 예시를 보도록 하자.

<img width="1787" alt="image" src="https://user-images.githubusercontent.com/41438361/157138757-2697104f-5f4e-4cf0-8e41-56fcc126ab0d.png">

위는 Assign 클래스로, 보면 객체와 type safe한 keyPath로 초기화 된다. 얘가 하는 일은 input이 들어오면 그 객체의 프로퍼티에 작성하는 것이다.
Swift에서는 단순히 프로퍼티에 값을 쓸 때 에러를 핸들링할 방법이 없기 때문에 Assign의 Failure 타입을 Never로 설정했다.

## How publisher and subscriber fit together

이제 대충 Publisher가 값을 내보내고, Subscriber가 subscription, input, completion을 받는다는 것을 알았다. 그러면 publisher과 subscriber는 어떻게 같이 동작할까?

<img width="1146" alt="image" src="https://user-images.githubusercontent.com/41438361/157139329-45c3aeaf-d485-4d8b-b48e-c010b2c88b41.png">

1. 먼저 Subscriber를 가지고 있는 controller 객체나 다른 무언가가 있을 것이고, 얘는 subscribe를 호출해서 Subscriber를 Publisher에 붙인다.
2. 그러면 Publisher는 Subscriber에게 subscription을 보내는데, 이 subscription으로 Subscriber는 Publisher에게 특정 수나 제한되지 않은 수의 값을 받겠다고 Publisher에 요청을 보낼 것이다.
3. 이제 Publisher는 Subscriber에게 값을 자유롭게 보낼 수 있다.
4. 마지막으로 만약 Publisher가 제한되어 있다면 Completion이나 Error를 보낸다.

그니까 좀 더 쉽게 생각하면 Publisher는 넷플릭스고 나는 subscriber다. 처음에 나는 넷플에 회원가입(subscribe 호출)을 해서 구독자가 된다.
그러면 넷플은 나한테 멤버십 회원권(subscription)을 주고, 나는 이 멤버십 회원권으로 넷플한테 영화를 요청(request 호출)할 수 있다. 그래서
내가 영화를 보여달라고 넷플에 요청을 하면 넷플은 정해진 영화의 수를 보여주는 것이다.(receive)

WWDC에서 보여준 또 다른 예시를 보자.

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157140262-aa94b725-5496-49a7-99e5-b439fb445e20.png">

여기서 하고자 하는 것은 학생들이 졸업할 때 알림을 받아 모델 객체의 값을 업데이트 하는 것이다. 그래서 `graduationPublisher`라는 NotificationCenter Publisher를 만들었고,
`gradeSubscriber`라는 Assign subscriber를 만들었다. 그리고 `subscribe`를 호출해서 이 둘을 연결하려고 하는데, 이러면 타입이 같지 않기 때문에
컴파일 에러가 나게 된다.

<img width="488" alt="image" src="https://user-images.githubusercontent.com/41438361/157140561-5d30a972-6f1a-4a84-9d7f-93eb686ba61e.png">

NotificationCenter는 notification을 만드는데, Assign는 정수 값을 받기를 예상하고 있기 떄문이다. 그래서 위와 같이 중간에서 notification과 integer를
변환할 무언가가 필요한데, 이것이 **Operator**다.

## Operator

Operator는 Publisher 프로토콜을 채택하고 있는 Publisher다. 그래서 얘네도 선언적이며 값 타입이다. 얘네가 하는 거는 값을 변환하거나, 값을 추가하거나
삭제하거나 다른 연산들에 대한 행위를 정의한다.

그리고 upstream이라고 하는 또 다른 publisher를 구독하고, downstream이라고 하는 subscriber에 결과를 전달한다.

Operator의 예시를 보자.

<img width="1785" alt="image" src="https://user-images.githubusercontent.com/41438361/157141011-fbb3f54f-2121-4114-b485-cca0f90cb3b8.png">

Map인데, Map은 연결할 upstream과 어떻게 upstream의 결과를 자신의 output으로 변환하는지에 대한 정보로 초기화 된다. Map은 스스로 Failure를
생성하지 않기 때문에 단순히 upstream의 Failure 타입을 그대로 전달한다. 

그래서 이 MAP을 통해 notification과 integer를 변환할 수 있다.

<img width="481" alt="image" src="https://user-images.githubusercontent.com/41438361/157141553-ee82f6c2-3d3a-417b-afd9-8f6ed316a488.png">

## Publisher-Operator-Subscriber

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157141701-cc0ed5ee-00f9-4155-aeaa-e71899ef3386.png">

위에서 봤던 코드에서 publisher와 subscriber는 그대로 사용하고, `converter`가 추가됐다. 코드를 보면 `converter`는 `graduationPublisher`와
연결되어 있고 클로저를 가지고 있다. 클로저는 notification을 받는데, userInfo에서 "NewGrade"라는 키를 가진 값을 받는다.

그리고 이 키에 값이 있다면 정수로 변환하고, 없거나 정수가 아니라면 default 값 0를 리턴한다. 그래서 중요한 것은 이 `converter`가 notification을 받아
integer로 변환시켜준다는 것이고, 이를 또 subscriber에 연결하는 모든게 연결이 다 잘 된 것이다. 

그런데 위 코드의 문법을 보면 꽤 길고 쓸데없는 단어가 있다고 생각할 수 있는데, 더 **유연한 문법**으로도 작성할 수 있다. 

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157163651-6b48d0bb-07d9-475c-9711-cd3c33e9f78a.png">

**Publisher의 extension**으로 작성된 위의 코드를 보자. Extension으로 작성되었으니 모든 Publisher는 이를 사용할 수 있다. Operator의
이름을 딴 여러 메서드들이 구현되어 있고, 위의 코드에서는 Map의 이름을 딴 메서드가 예시로 나와있다. 

인자를 보면 upstream을 제외하고 Map을 초기화하기 위한 인자들이 있다. Upstream의 경우는 단순히 self를 사용하면 되기 때문에 인자로 가질 필요가 없다.

위의 코드를 보면 어 그냥 Operator 이름을 따서 publisher의 extension으로 집어넣은거 아니야? 라고만 생각이 드는데 WWDC 영상에서는 이것이 asynchronous programming을 
생각하는 관점을 바꿀 것이라고 했다.

## New Syntax

위에서 언급한 publisher의 extension을 사용해서 코드를 다시 작성한 예시를 보자.

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157164149-c92590d6-d1bc-46a8-b5aa-b33bb58b3146.png">

먼저 NotificationCenter Publisher를 생성한다. 그리고 notification을 받으면, 클로저를 사용해서 map을 하고, Merlin의 grade property에 assign한다.

위 코드의 문법을 보면 선형적이고, **단계마다 어떻게 진행될 수 있는 지 알기 쉽기 때문에 이해하기도 쉽다**. WWDC에서는 이런 단계별 문법이 Combine을 사용하는데 핵심이라고 했다.
각 단계는 다음 명령어를 연속적으로 설명하고, 값들은 처음에 Publisher에서 생성되고, 여러 개의 Operator를 거쳐 마지막에는 Subscriber로 들어온다. 

그리고 이런 Operator가 많은데, 이를 **Declarative Operator API**라고 부른다고 한다.

## Declarative Operator API

<img width="1433" alt="image" src="https://user-images.githubusercontent.com/41438361/157164659-67d23dfa-3547-4b82-b0d5-1d177c8a1df8.png">

Declarative Operator API에는 아래와 같은 것들이 있다고 한다.

1. Functional transformations : Map, Filter, Reduce 등등
2. List operations : Publisher의 몇 번째 요소 이런 것들을 알 수 있다 등등
3. Error handling : 에러가 발생하면 이를 default나 다른 값으로 값을 설정한다
4. Thread or queue movement : 무거운 작업을 백그라운드 쓰레드로 옮기거나 UI작업을 메인 쓰레드로 옮긴다
5. Scheduling and time : loop, dispatch queue, timer, timeouts 등등을 포함한다.

그런데 이 많은 operator 들을 어떻게 사용해야 할 지를 생각하면 복잡하게 느껴질 수 있다. 그래서 WWDC 영상에서는 **Combine의 핵심 디자인 원리를 근간으로 하여 생각하라 한다.**

즉 compostion에 초점을 맞추는 것이다.

## Composition

<img width="1060" alt="image" src="https://user-images.githubusercontent.com/41438361/157167259-722febd6-c0e3-4d3a-b145-248ec9df8176.png">

수많은 operator들이 있지만, operator 하나가 할 수 있는 일을 작게 만들어서 각각을 더 이해하기 쉽게 만들었다고 한다. 그래서 operator를 만들 때 이미 존재하는 Swift Collection API 이름에서
영감을 받았다고 한다. 

<img width="974" alt="image" src="https://user-images.githubusercontent.com/41438361/157167544-bf7dbb47-5e18-4955-913b-e6a4973853dc.png">

위의 사분면 그래프를 보자. Swift에서 integer를 synchronous하게 표현하려면 `int`와 같은 걸 사용할 거고, 많은 integer를 synchronous하게 표현하고 싶으면 
integer array를 사용할 것이다.

Combine에서는 이런 개념을 asynchronous한 영역에 매핑했다. 그래서 하나의 값을 표현하고 싶으면 뒤에 보겠지만 `future`를 사용할 거고, 많은 값들을 asynchronous하게
표현해야 한다면 Publisher를 사용한다.

즉 **이미 알고 있는 array를 사용하는 연산과 비슷한 거를 찾아보고 싶다면, Publisher에서 같은 이름의 연산이 있는지를 찾아보면 된다.**

예시를 보자.

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157168104-759fe1ba-6fbb-43b7-9fc2-7f72577594ae.png">

위 코드에서는 key가 존재하지 않거나 저장된 값이 정수가 아니면 default 값으로 0을 선택하고 있다. 하지만 나쁜 값이 들어왔을 때 이를 처리하지 않는 것이 더 좋은 처리 방법인 것 같다.
그래서 이 클로저가 nil을 리턴하게 한 다음, nil 값들은 필터링 해버리는 방법을 생각할 수 있다. Swift 4.1에서는 이 연산을 `compactMap`으로 할 수 있고, Publisher도 같은 연산을 가지고 있다.
매우 비슷한 방식으로 동작한다.

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157168361-24abd861-ecdf-4130-a841-bbbdce53ceff.png">

그래서 코드를 위와 같이 `map` 대신 `compactMap`을 사용하게 바꿀 수 있고, nil를 리턴하면 이를 필터링해버릴 것이다. 

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157168517-73c23e3d-eb6d-4ef3-a2cc-7899082fc796.png">

이런 익숙한 단계별 명령어를 좀 더 사용해서 위와 같이 작성할 수 있다. Filter를 사용해서 특정 값 이상인 값들만 거를 수 있고, prefix를 사용해서 앞의 몇 개의 값만
받을 수 있도록 설정할 수 있다. 이런 연산들은 array에서도 같은 기능을 한다. 위 코드에서는 `prefix(3)`를 통해 3개의 값을 받으면, upstream을 취소하고
downstream에 completion을 보내게 된다. 

## Combining Publishers

Map과 filter는 굉장히 좋은 API인데, 주로 synchronous항 연산들에 사용된다. Combine은 이 때 asynchronous한 영역에서 빛을 발한다.

이런 상황에서 유용한 operator가 있는데, zip과 combineLatest다.

### Zip

예를 들어 지팡이를 발급받기 위해 3가지 비동기 작업이 완료될 때까지 기다려야 하는 앱이 있다고 해보자. 세 비동기 작업이 완료된 후에 활성화 되는 버튼이 있어서 이 버튼을 눌러야 다음 단계로 넘어갈 수 있다. 이런 작업을 zip으로 할 수 있다.

Zip은 여러 upstream input들을 하나의 tuple로 변환한다.

<img width="1189" alt="image" src="https://user-images.githubusercontent.com/41438361/157169220-6be24993-3b21-4fc5-857f-c939b499a20a.png">

예를 들어 첫 번째 publisher가 A를 생성하고, 그리고 두 번째 publisher가 1을 생성하면, 이제 tuple을 생성해서 이 값을 downstream subscriber로
전송할 수 있는 것이다.

<img width="605" alt="image" src="https://user-images.githubusercontent.com/41438361/157169408-d172c8fc-b564-47c2-b4b4-2e323a00f570.png">

그림을 보면 (A, 1)이라는 tuple이 생성되어 downstream에 전송되고 있다.

<img width="1635" alt="image" src="https://user-images.githubusercontent.com/41438361/157169476-2dd814c5-862e-4964-b895-1908caf77811.png">

위에서 얘기한 지팡이를 발급받기 위해 세 비동기 작업을 기다려야 하는 앱이다. 코드를 보면 세 개의 upstream을 가진 zip을 사용해서 세 개의 비동기 작업이
완료되기를 기다리고 있다. 그리고 각 연산은 Boolean 값을 리턴한다. 그래서 이 세 Boolean 값을 tuple로 map을 하고, `isEnabled` 프로퍼티에 할당하는 것이다.

### CombineLatest

이제 지팡이를 발급받았다! 그런데 발급 받고 나서 지팡이를 사용하기 전에 동의해야 하는 이용 약관들이 존재한다. 세 가지 약관을 다 동의해야 넘어갈 수 있는데,
가령 세 약관을 다 동의했다가 아래와 같이 한 약관을 다시 동의하지 않음으로 변경했다고 해보자.

<img width="473" alt="image" src="https://user-images.githubusercontent.com/41438361/157170029-3566cf40-2d7d-4662-a686-ea698f3c0640.png">

이러면 하단의 play 버튼은 활성화 되어 있다가 다시 비활성화되어야 할 것이다. 이런 작업을 combineLatest로 할 수 있다.

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157170160-76d783b2-19bb-48aa-a7c1-27c5e46dc7ba.png">

Zip과 똑같이, 여러 upstream input을 받아 하나의 값으로 만든다. 하지만 Zip과는 다르게, upstream들 중 어떤 것에서만 값이 들어와도 진행할 수 있다는 것이다.
Zip은 모든 upstream에서 값이 들어와야지 진행되었는데, combineLatest는 한 upstream에서 값이 내려오든 여러 upstream에서 값이 내려오든 상관하지 않는 것이다.

이를 위해 각 upstream에서 받았던 가장 최신의 값을 저장하고 있고, 이를 하나의 downstream 값으로 변환할 closure로 구성되어 있다.

<img width="680" alt="image" src="https://user-images.githubusercontent.com/41438361/157170524-870f35b1-42a5-4792-b8da-7c2726cd532e.png">

마찬가지로 예시를 들어서 두 개의 upstream이 있고 각 publisher로부터 A, 1을 받았다고 해보자. 그러면 (A, 1)이라는 tuple을 생성해 downstream으로
보냈을 것이다. 그리고 이후 두 번째 publisher가 2를 내려보내면, 기존에 두 번째 publisher한테 받았던 1을 2로 대체하고, 새로운 (A, 2) 튜플을 만들어 downstream에 보내게 된다.

<img width="657" alt="image" src="https://user-images.githubusercontent.com/41438361/157170703-2e5d9cfd-7d09-422c-ad24-4ef923b134dd.png">

그래서 upstream 중 하나라도 값이 변하게 되면 새로운 이벤트가 발생하게 된다.

<img width="1637" alt="image" src="https://user-images.githubusercontent.com/41438361/157170835-a58ab0e3-a7ff-4c26-af40-986b53e9d06f.png">

다시 돌아가서 이용 약관 동의하는 부분을 보자. 세 개의 upstream을 갖는 CombineLatest를 사용했고, 변할 수 있는 세 개의 boolean 값을 하나의 Boolean 값으로 만들어
play 버튼의 isEnabled 프로퍼티에 할당하고 있다.

## Recommendation

당장 비동기적인 모든 작업들을 combine으로 대체할 수는 없다. 따라서 지금부터 적용할 수 있는 부분을 찾아 Combine을 사용하면 된다.

예를 들어서 NotificationCenter를 사용하고 있고, notification을 받는데 무언가를 할지 말지를 결정해야 한다면 Filter를 사용할 수있다.
그리고 여러 asynchronous operation의 결과가 중요하다면 zip을 사용할 수 있다. 그리고 URL Session을 사용해서 데이터를 받고 이를 JSON Decoder를 사용해서
나만의 객체로 만들어야 한다면 Decode를 사용할 수도 있다.

이 WWDC 영상에서는 Combine의 기초 개념인 Publisher, Subscriber, 그리고 Operator에 대해 전반적으로 살펴봤는데, error handling,
cancelation, scheduler, time, combine을 사용한 design pattern도 같이 보면 좋다. 이는 WWDC의 또 다른 영상인 "Combine in Practice"에서 
자세하게 알 수 있다.

# Combine in Practice (WWDC 2019)

영상 앞부분에는 앞선 WWDC 영상에서 다뤘던 기초적인 내용들을 다시 되짚고 있는데, 이 부분은 생략하도록 하겠다. 그리고 이 영상은 "Combine in Practice"답게
실제 코드를 어떻게 작성할지를 보여주려고 하는 것 같다.

<img width="1700" alt="image" src="https://user-images.githubusercontent.com/41438361/157172425-6840899e-2a3c-4647-a7f1-e0494b82175f.png">
<img width="845" alt="image" src="https://user-images.githubusercontent.com/41438361/157172686-16bde87a-2e10-4be9-a75e-50ca3ce5648d.png">

갑분 마법사와 뭔 낙서냐 하겠지만 우리의 목표는 저 마법사를 도와 앱을 만들 건데, 마술 트릭을 다른 마법사들과 공유할 수 있는 앱을 만드는 것이다.
저 낙서가 마법사가 스케치한 앱의 대략적인 프로토타입이다. 그래서 어떻게 combine을 사용해서 저 낙서의 "Trick Name"을 마술 트릭의 이름으로 채울 수 있을까?

## Deep dive

이제는 코드롤 좀 자세히 볼 것이다.

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157180536-17bc3484-b405-4df6-b743-0f48fd4cc43b.png">

NotificationCenter는 publisher로 notification을 발행한다. 이 함수는 Publisher를 리턴할 건데, Combine에서 중요한 거는 Publisher의
output과 failure 타입이 무엇인지가 중요하다. NotificationCenter Publisher는 notification을 전달하고, 절대 실패하는 일이 없다.
위 사진에서 네모 박스 두개가 있는데, 위에 있는 것은 output 타입, 밑에 있는 것은 failure 타입이 될 것이다.

제일 중요한 것은 Publisher가 전달하는 실제 마법 트릭에 대한 데이터다.

<img width="1783" alt="image" src="https://user-images.githubusercontent.com/41438361/157180996-1c2095f4-9661-4969-957c-78b09321f277.png">

이 데이터가 dictionary 안에 있다고 마법사가 알려줬고, combine의 map 함수를 이용해서 notification을 우리가 원하는 형태로 변환한다.
그리고 이 결과를 보면 output은 data가 될 것이고, 에러는 절대로 생성되지 않는다.

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157181609-16db282a-acdf-4cc8-bdc2-070752c9c67d.png">

그리고 마법사는 데이터는 이미 앱 내에 정의된 타입의 JSON payload가 될 수 있다고 알려줬다. 그래서 Combine에서 데이터를 decode할 수 있는 operator인
tryMap을 사용했다. 이건 map이랑 비슷한데, 에러를 stream에 throw할 수 있는 추가 기능이 있다. 마찬가지로 이 함수의 output은 마법 트릭의 Publisher가 될 것이고
failure는 Swift 에러 프로토콜을 따를 것이다.

그리고 추가로 이렇게 데이터에서 커스텀 타입을 decoding하는 작업은 흔한 작업이기 때문에 이런 걸 해주는 operator가 있다. 바로 Decode다.

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157181858-497f9065-4acd-4b8a-9ea9-ca1949e2b589.png">

## Error handling

<img width="1283" alt="image" src="https://user-images.githubusercontent.com/41438361/157182775-82bc366f-f9a7-44c5-9638-a56bafca7ba7.png">

이제 우리는 fail 할 수 있는 Publisher를 가지고 있다. **Combine에서는 발생 가능한 failure를 적절히 다루는 것은 매우매우 중요하다.**
모든 Publisher와 Subscriber는 생성하거나 허용할 failure의 정확한 타입을 서술할 수 있다. Swift와 마찬가지로, 에러를 처리하는 것이 관행에 따라 처리되지 않게 하기 위해 Combine에 이런 
걸 구축해두었다고 한다. 

많은 타입에서 failure type으로 never를 설정했는데, 이건 그들이 fail할 수 있거나 이 failure가 앞서서 handling 되기를 암시하는 것이다.
Combine은 발생하는 failure에 반응하고 복구하기 위한 많은 operator를 제공하고 있다. 

<img width="1770" alt="image" src="https://user-images.githubusercontent.com/41438361/157183035-522588ba-1d1f-46ba-a78e-3d4caff534e5.png">

코드를 위와 같이 작성해서 failure가 절대로 발생하지 않는다고 해보자. 이러면 Publisher의 failure 타입은 never가 된다. 

<img width="1789" alt="image" src="https://user-images.githubusercontent.com/41438361/157183263-0a226502-9d36-4d5a-884a-e341ffd6b94d.png">

Publisher가 downstream Subscriber와 연결되어 있는데 중간에 assertNoFailure operator가 있다고 해보자. 이 operator는 값을 받는다면
이를 그대로 전달할 것이다. 

<img width="1489" alt="image" src="https://user-images.githubusercontent.com/41438361/157183517-233bd63a-913c-4f60-837b-490502b61e68.png">

하지만 upstream에서 에러가 오면, 프로그램은 멈추게 될 것이다. 이런 상황은 피해야 한다. 

## Failure Handling Operators

이를 방지하기 위해, failure를 다루는 많은 operator가 존재한다. 

<img width="1790" alt="image" src="https://user-images.githubusercontent.com/41438361/157183741-1ad4b3e2-18c1-4e9c-b77f-908cdcb33f4d.png">

* retry를 사용해서 upstream Publisher와의 연결을 재시도 하거나 에러를 다른 타입으로 변환할 수도 있다.
* catch는 upstream Publisher에서 failure가 발생했을 경우 recovery Publisher를 정의하는 클로저를 받아 처리해준다.

### catch

앞선 publisher - assertNoFailure - Subscriber 케이스에서는 에러가 발생하면 upstream 과의 연결은 중단되게 된다. 

<img width="1696" alt="image" src="https://user-images.githubusercontent.com/41438361/157184535-2cb42370-a351-44e2-bf9d-4c4c7ff2ec95.png">

이런 상황에서 assertNoFailure 대신에 catch를 사용한다면 에러가 발생했을 때 upstream connection은 끊기고, recovery closure가 새로운 publisher를 
생성하게 된다. 그리고 이를 subscribe해서 값을 받게 된다.

즉 catch operator는 기존의 publisher를 새로운 걸로 대체해서 에러를 핸들링한다.

<img width="1789" alt="image" src="https://user-images.githubusercontent.com/41438361/157184748-e7ff19e5-e293-4114-ac77-d179bc053431.png">

다시 코드를 보면 다른 operator와 비슷하게 catch를 호출하는데, Publisher를 리턴하는 클로저도 같이 제공해야 한다.

**Combine은 이미 publish하고 싶은 값이 있을 때 사용할 수 있는 특별한 Publisher를 제공하는데, Just다.** Just라 명명된 이유는 그냥
이 값을 publish해라 라는 의미에서 just라 부른다고 한다.

아무튼 이렇게 코드를 작성하면 Publisher는 절대로 fail하지 않게 된다.

## Review

<img width="1087" alt="image" src="https://user-images.githubusercontent.com/41438361/157185346-1a9a2251-d2f9-4be2-88c2-a033b79c7511.png">

지금까지 작성한 코드를 다시 보자. Publisher의 notification으로 시작해서, decode되기를 원하는 데이터까지 계속 mapping 해왔다. 그리고 decode operator를 
사용해서 우리가 임의로 정의한 타입으로 데이터를 다시 변환했다.

하지만 다양한 이유로 decoding이 실패할 수 있기 때문에 이를 고려해서 failure가 발생할 경우 upstream을 대체한다.

하지만 여기서 문제는 우리가 Recovery Publisher로 기존의 upstream을 대체했을 때, 다른 notification을 더 이상 받을 수 없다. 기존의 Upstream과의
연결을 중단해버렸기 때문이다. 우리가 진짜로 원하는 것은 **decode를 해서 이게 실패했을 때 대체제를 사용하고 original upstream과의 연결은 끊지 않는 것이다.*

Combine에서는 이를 flatMap이라는 operator로 해결할 수 있다.

### flatMap

<img width="1075" alt="image" src="https://user-images.githubusercontent.com/41438361/157186474-f4b646f3-ff30-4621-a271-46a58d64f8bd.png">

upstream Publisher에서 값이 오면, 이 값들을 바탕으로 새로운 Publisher를 생성하게 된다. flatMap은 이 중첩된 Publisher를 사용해서 subscribing을
관리할 수 있다. 말이 어려운데, 이를 그림으로 이해해보자.

<img width="1746" alt="image" src="https://user-images.githubusercontent.com/41438361/157187018-57ce6430-57cc-4129-b76b-f93605e14323.png">

1. upstream에서 flatMap operator로 값이 전달된다.
2. flatMap은 이 값을 새로운 Publisher로 변환하기 위한 closure를 호출한다.
  1. Just-decode-catch를 거쳐 값을 downstream으로 내려보낸다.
  2. 만약 위 그림과 같이 decode하는 도중 에러가 발생하면? 에러가 catch에 도달하면 recovery Publisher로 대체된다. 그리고 이 publisher는 flatMap에 리턴되고, 이 publisher가 publish하는 값이 downstream으로 전달되기 때문에 operation이 절대 실패하지 않는다는 것을 보장할 수 있다.

솔직히 그림을 봐도 이해하기가 힘들다.

<img width="1787" alt="image" src="https://user-images.githubusercontent.com/41438361/157187813-1229e5c5-5639-40eb-910b-bb693c52f534.png">

코드를 보자. catch와 함께 우리가 받은 데이터로 새로운 publisher just를 사용한다. `data`는 map Operator에서 받은 데이터로, 이를 decode하고,
catch해서, 새로운 publisher를 flatMap에 리턴한다. 그러면 flatMap은 이 publisher를 subscribe하고, 최종적으로 publisher는 절대로 실패할 수 없는
publisher가 된다.

하여간 flatMap을 사용해서 에러 핸들링을 하는 걸 알아봤고, 원래 하고자 했던 매직 트릭 이름을 publish하기 위해 아래와 같이 코드를 작성한다.

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157188665-c180a546-cf5c-4137-bd78-4279b908c008.png">

`publisher(for:)` operator를 사용해서 새로운 publisher를 만들고, 이 경우에는 string을 publish 할 것이다.

## Scheduled Operators

추가로 강력한 기능을 제공하는 operator가 있다. 현실에서 스케줄링하는 것과 같이, scheduled operator는 언제 어디서 특정 이벤트가 전달되었는지를
파악할 수 있게 해준다.

<img width="1680" alt="image" src="https://user-images.githubusercontent.com/41438361/157188954-f02c0e88-2bd2-4aa2-918f-d788944d170d.png">

Operator들은 본질적으로 RunLoop와 DispatchQueue에 의해 지원되고, 아래의 operator들이 있다.

<img width="1154" alt="image" src="https://user-images.githubusercontent.com/41438361/157189143-f24a5282-993b-49cb-98bf-ca979b342db3.png">

* delay : 미래의 특정 시간이 될 때까지 이벤트를 연기한다
* throttle : 특정 rate보다 더 빠르지 않게 이벤트가 전달되도록 한다.
* receive : downstream이 특정 쓰레드나 큐에 이벤트를 받도록 보장한다

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157189592-493eb78c-c2f4-409c-bc67-36a36f188808.png">

마지막에 `receive` operator를 사용해서 마법 트릭 이름이 항상 메인 큐에 전달되도록 보장할 것이다. 그리고 output과 failure타입이 변하지 않은 것도 확인할 수 있는데,
scheduled operator는 일반적으로 output과 failure 타입을 변경하지 않는다. 

## Review

<img width="1618" alt="image" src="https://user-images.githubusercontent.com/41438361/157189922-cacd2852-e938-4f0f-b0a0-4ebdb1d2d993.png">

지금까지 작성한 코드의 구조를 다시 보자. `publisher(for:)` 를 사용해서 마법 트릭 이름을 추출했고, 마지막으로 `receive(on:)` operator로
메인 쓰레드로 작업을 옮겼다. 그리고 이젠 메인 쓰레드 context에서 UI를 업데이트 할 준비가 된 것이다!

## Publisher

<img width="1330" alt="image" src="https://user-images.githubusercontent.com/41438361/157190395-58ba4f09-6d5d-4ee9-a9ee-d30d4e18c3ae.png">

지금까지 주구장창 Publisher와 그들의 operator에 대해 알아봤다. 단계별로 각 operator를 거쳐 시간에 따라 값들을 생성하는 방법을 제공했다. 
그리고 Publisher가 Just와 같이 값들을 synchronous하게 생성할 수도 있지만, NotificationCenter와 같이 asynchronous하게 생성할 수도 있음을
알았다. 

## Subscriber

이제는 값을 받는 부분에 대해 살펴보자.

<img width="1779" alt="image" src="https://user-images.githubusercontent.com/41438361/157191160-819b55a6-f8ac-463d-b66d-0c63b5b55132.png">

앞선 WWDC 영상에서 봤듯이, Subscriber는 세 개의 함수를 가지고 있다. 이 세 함수는 세 규칙을 내포하고 있다.

<img width="962" alt="image" src="https://user-images.githubusercontent.com/41438361/157194859-c79459a6-06ef-4ce1-84ca-06b9b9418b13.png">

1. subscribe 호출에 대한 응답으로, Publisher는 `receive(subscription:)`을 딱 한 번만 호출한다.
2. Publisher는 Subscriber가 요청한 이후에 0개 이상의 값들을 downstream으로 전달할 수 있다.
3. Publisher는 Publisher가 전송하는 걸 완료했거나 에러가 발생했음을 나타낼 수 있는 하나의 completion을 전송할 수 있다.

그리고 참고로 completion은 optional인데 이는 stream이 무한대로 존재할 수 있을 수도 있기 떄문이다.

<img width="837" alt="image" src="https://user-images.githubusercontent.com/41438361/157195159-3220939a-aa8d-414b-9b44-2564cccbb86b.png">

그리고 Combine에서는 다양한 종류의 Subscriber를 제공한다. 

### Key path assignment

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157195445-72a5b020-84b9-48d9-bf3e-3ab86d3a36d8.png">

Combine에서 가장 간단한 형태의 subscription은 위와 같이 `assign(to:on:)` operator를 사용해서 key path assignment를 사용하는 것이다. 것이다.
이렇게 하면 upstream Publisher에서 온 값은 명시된 key path의 객체에 할당된다. 이는 어떤 publisher를 어떤 프로퍼티에 할당할지 제약이 딱히 없고
자유롭기 때문에 강력하다고 할 수 있다. 

이 operator는 cancellation token을 생성하는데, 이를 사용해서 subscription을 중지시킬 수도 있다.

### Cancellation

<img width="1766" alt="image" src="https://user-images.githubusercontent.com/41438361/157196211-ed175cf6-8e0c-48da-b026-0aed8428c08d.png">

Cancellation은 필요할 때 subscriber가 unsubscribe할 수 있게 해준다.

Cancel을 Combine의 형태로 만든 이유는 Publisher가 이벤트를 전달하기 전에 subscription을 중단하는 것이 도움이 될 때가 있기 때문이다. 특히
subscription과 연관된 자원들을 해제할 때 유용하다. 

그리고 새로운 프로토콜 `Cancellable`이 나왔는데, 얘는 메서드를 보면 알 수 있듯이 취소할 수 있거나 취소되어야 하는 것들을 묘사할 수 있다.
그리고 굉장히 편리한 `AnyCancellable` 이라는 클래스는 deinit시에 자동으로 cancel을 호출해준다. 이를 통해 cancel을 명시적으로 호출하는 횟수를 줄일 수 있다.

<img width="1757" alt="image" src="https://user-images.githubusercontent.com/41438361/157197166-453e4dcc-e389-493e-8eba-0afdbe592d4d.png">

### Sink

<img width="1757" alt="image" src="https://user-images.githubusercontent.com/41438361/157197296-ce019bdc-8919-489b-8d85-783e2c08f3f0.png">

Subscription의 두 번째 형태를 볼 건데, sink operator를 사용하는 것이다. 단순히 클로저를 제공하고 값이 들어오면 클로저가 호출돼서 내가 하고 싶은 
작업들을 할 수 있는 것이다. assign과 마찬가지로, sink는 cancellable를 리턴해서 subscription을 중단할 수 있게 해준다.

### Subject

<img width="1773" alt="image" src="https://user-images.githubusercontent.com/41438361/157197641-62796a9f-e641-4efb-9e46-d2e6b5032fc8.png">

세 번째 형태의 subscription은 약간 복합적인데, Publisher 같기도 하고 subscriber같기도 하다.

수신받는 값들을 multicasting하는 걸 지원하고, 값들을 반드시 전달할 수 있게 해준다. 그리고 이건 이미 존재하는 code base와 함께 동작할 때 중요하다.

<img width="1725" alt="image" src="https://user-images.githubusercontent.com/41438361/157198779-cfa51331-014a-4aeb-a5cb-dfe7525da793.png">

Subject를 통해 여러 개의 downstream Subscriber에 값을 전달할 수 있다. 

<img width="1727" alt="image" src="https://user-images.githubusercontent.com/41438361/157199004-c98ac960-4034-4c03-ab6e-3a53e1e8c1cb.png">

Subject에는 두 종류가 있는데, Passthrough subject와 CurrentValue subject다. Passthrought subject는 값을 저장하고 있지 않아서
subject를 subscribe한 이후의 값만 볼 수 있다. CurrentValue subject는 최근에 받은 값을 저장하고 있어서, 새로운 subscriber가 이를 볼 수 있도록 한다.

<img width="1790" alt="image" src="https://user-images.githubusercontent.com/41438361/157199611-5ab44896-b170-4377-a5ce-3f157fc0160b.png">

먼저 subject를 생성하는데 Output과 failure 타입을 같이 명시해서 생성자를 호출한다. 그리고 upstream publisher에 subscribe를 해서 subscriber로서 동작한다.
그리고 publisher가 operator를 사용하는 것과 같이, `sink`를 사용해서 subscriber 스스로 subscriber를 생성한다. 그리고 값을 직접적으로 전달할 수도 있다.

## Example

처음에는 작은 Publisher에서 시작해서, 여러 변형을 거쳐 우리가 원하는 최종 Publisher를 만들었다. 실제 앱에서 어떻게 사용할 지 예시를 보자.

마법 학교를 가입하기 위해 아래와 같은 폼을 가진 앱이 있다.

<img width="1328" alt="image" src="https://user-images.githubusercontent.com/41438361/157206607-5f290d4a-6bed-4bed-8612-899cf2481880.png">

여기에서 해야 할 일은 두 가지다.

1. 사용자 이름이 유효한지 서버에서 검증을 받아야 한다.
2. 비밀번호 확인을 하는데, 비밀번호-비밀번호 확인란에 입력한 비밀번호가 일치하고 8글자를 넘어야 함을 확인한다.

그리고 이 모든 조건을 충족시켰을 때 UI를 업데이트 해서 버튼을 활성화 시킬 것이다.

그래서 서버에 확인하는 asynchronous 한 동작이 있고, 디바이스 내에서 처리 가능한 synchronous한 작업이 있다. 그리고 이 모두를 조합해야 한다. 여기서 Combine을 사용해보자.

### @Published

<img width="1298" alt="image" src="https://user-images.githubusercontent.com/41438361/157418458-4f936d81-ae7d-4fa2-9dc1-c22544c97117.png">

그 전에 하나 알아야 할 것이 있는데, 바로 이 `@Published`다. Published는 Swfit 5.1 피처로 프로퍼티에 Publisher를 추가한다.

간단한 예시로 어떻게 `@Published`를 사용하는 지 보자.

<img width="1790" alt="image" src="https://user-images.githubusercontent.com/41438361/157419038-5441d48b-f148-4a1b-abab-40bb20331a6c.png">

1. 먼저 `@Published` Published property wrapper를 프로퍼티 앞에 추가한다.
2. `currentPassword`는 "1234"다.
3. `$password`에서처럼 달러 prefix를 붙여서 wrapped value에 접근한다.
4. 그리고 `password`를 "password"롤 설정했을 때, subscriber는 이 변화된 값을 받을 것이다.
5. 그래서 "The published value is 'password'" 가 출력된다.

이제 `@published`를 어떻게 사용하는지 알았으니, 앱에서는 어떻게 사용해야 할 지 보자.

<img width="1023" alt="image" src="https://user-images.githubusercontent.com/41438361/157419992-05dc199a-8e1f-4be6-b158-0fbde8bd7eea.png">

그리고 위에서처럼 비밀번호와 비밀번호 확인란에 입력한 두 값이 같은 때에 확인 절차를 걸쳐야 한다.

<img width="965" alt="image" src="https://user-images.githubusercontent.com/41438361/157420523-53425ece-d244-4df1-81b2-c441e136ddd5.png">

그래서 이 프로퍼티에 Published를 붙여서 두 개의 Publisher를 추가하고, published string은 절대로 fail하는 일이 없다. 그리고 이 두 published 된 것들을 하나의 검증된 비밀번호로 만들고 싶은데, 여기에 CombineLatest operator를 사용할 수 있다. 

<img width="1781" alt="image" src="https://user-images.githubusercontent.com/41438361/157420940-e37e92ff-5f10-4312-9217-33940baba024.png">

위의 코드를 보면 비밀번호와 비밀번호 확인 프로퍼티에 `@Published`를 붙여서 publisher를 추가했다. 그리고 CombineLatest를 사용해서
달러 prefix와 함께 property wrapper에 접근할 수 있고, 이 둘 중 하나라도 값이 바뀌면 신호를 받을 수 있다. 그리고 클로저를 사용해서 비즈니스
로직을 구현하는데, 코드를 보면 두 프로퍼티 모두 다 8자 이상임을 검증하고 있다. 그리고 그렇지 않을 경우에는 nil을 리턴해서, 즉 nil을 signal로
사용해서 작성한 폼이 유효하지 않음을 나타냈다.

그리고 타입을 작성한 코드 부분을 보면 어떤 일이 일어나는지를 쉽게 확인 가능하다. 두 개의 published string을 받고, 이들의 가장 최신의 값들을 
합쳐서 최종적으로 optional string을 만들었음을 확인할 수 있다.

만약에 "password1"이라는 비밀 번호를 허용하지 않겠다는 새로운 요구사항이 생기면 어떻게 할까?

<img width="1763" alt="image" src="https://user-images.githubusercontent.com/41438361/157423449-e7570503-8bcf-42c9-9d65-ba0f254a989f.png">

위 코드를 보면 map을 추가했다. 타입도 바뀌었는데, 두 개의 published string에서 최신의 값들을 받아 이를 통합하고, optional string으로 매핑했다. 하지만 이걸 API 영역으로 넘기고 이게 다른 Publisher를 구성하게끔 하고 싶다. 

<img width="1775" alt="image" src="https://user-images.githubusercontent.com/41438361/157554957-0463d800-b888-4d7c-93ce-39b9c1ab5fd5.png">

이때 `eraseToAnyPublisher`라는 Operator를 사용해서 AnyPublisher를 리턴하고, optional string과 never를 리턴하게 할 수 있다. 여기서
타입이 바뀌지는 않았지만 API 영역에 포함시키길 원하는 정확한 부분을 알리고 안의 구현 디테일을 숨길 수 있다.

<img width="764" alt="image" src="https://user-images.githubusercontent.com/41438361/157555626-94ba5a90-b4fd-467f-996e-495a4b6b728f.png">
<img width="876" alt="image" src="https://user-images.githubusercontent.com/41438361/157555678-37a79cab-056c-4365-a500-651936cae691.png">

그래서 지금까지 한 걸 보면, string 두 개에 각각 Published property wrapper를 사용해서 string Publisher를 추가했다. 그리고 CombineLatest를 사용해서 두 개의 Publisher의 최신 값들을 섞고, 비즈니스 로직을 추가했다. 그리고 map을 사용해서 나쁜 비밀번호를 거르고
최종적으로는 이 작업들이 API영역에 속하고, 이를 다른 것들과 같이 사용하기 위해 eraseToAnyPublisher를 사용했다.

이제 비밀번호와 비밀번호 확인란을 검증하는 첫 번째 Publisher를 만들었다. 다음으로 넘어가자.

<img width="1397" alt="image" src="https://user-images.githubusercontent.com/41438361/157556082-0bbd99b2-a9d5-4c98-918e-87a5500dac90.png">

이제 사용자가 굉장히 빠르게 타이핑하는 값을 가지고 서버에서 검증해서 사용자 이름이 유효한지를 검증할 것이다. 

<img width="934" alt="image" src="https://user-images.githubusercontent.com/41438361/157556446-34655abd-eef4-4886-befa-c10770045b26.png">

그래서 앞서서 했던 것과 같이 string 프로퍼티에 Published를 추가한다. 그리고 여기에서 좀 특별한 것은 사용자 이름이 유효한지를 검증을 교청하는
네트워크 연산이 사용자가 하나의 문자를 타이핑할 때마다 일어나지 않게 하는 것이다. 아니면 우리의 서버에 엄청 많은 요청을 보내게 될 것이다. 그래서 신호를
좀 더 부드럽게 보내야 한다. 이 때 Debounce를 써서 특정 속도보다 빠른 값은 받지 않도록 하는 window를 설정할 수 있게 해준다.

Debounce에 빠른 신호들이 들어왔을 때, 이를 하나의 signal로 완화시킬 수 있다.

<img width="1269" alt="image" src="https://user-images.githubusercontent.com/41438361/157557335-84b4f5b9-a600-47b2-9fd8-2bdc1f00e1a7.png">

추가로, 사용자가 입력한 값이 이전에 입력한 값과 같다면 나중에 입력한 값이 유효한지를 또 검증하기 위해 서버에 요청할 필요가 없다. 예를 들어 사용자가 Merlin을 입력하고, n을 지운다음 다시 n을 입력하면 서버에 다시 요청할 필요가 없다. 여기에 removeDuplicates를 사용할 수 있다. 
이를 통해 같은 값이 계속해서 해당 window에서 계속 전송되는 것을 막을 수 있다. 

<img width="1401" alt="image" src="https://user-images.githubusercontent.com/41438361/157557624-615bf50e-4aa9-47fa-90fa-079bf16325b6.png">

그래서 코드로 위와 같이 작성한다. Published를 사용자 이름 프로퍼티에 추가하고, debounce를 사용해서 신호를 완화시킨다. 그리고 중복되는 요청을 제거한다. 그런데 아직 비동기 작업은 아무것도 안됐고, 단지 신호만 완화했다. 

<img width="1762" alt="image" src="https://user-images.githubusercontent.com/41438361/157558058-d68b3b53-90a3-4679-91f1-921fb6d9e63f.png">

그래서 이를 위해서 앱에 정의된 서버에 사용자 이름이 유효한지 검증을 요청하는 `usernameAvailable`이라는 메서드를 사용할건데, 이를 Publisher로
가져올 것이다. FlatMap을 사용하면 stream에서 값을 가져와서 새로운 Publisher를 리턴한다고 했다. FlatMap 안에서 Future라는 걸 호출하고, 이걸
만들 때 클로저를 제공해야 한다. Promise는 결과(success/failure)를 가지는 클로저라고 생각하면 된다. 그리고 Future 안에서 userNameAvailalbe 함수를 호출하고, 비동기적으로 이게 완료돼서 값을 가져오게 되면 이 경우에 Promise를 success로 채우게 된다. 그리고 값이
유요하지 않으면 nil을 리턴한다.

<img width="964" alt="image" src="https://user-images.githubusercontent.com/41438361/157558663-2907b9d3-0554-45f1-817e-d8f161a2f4ef.png">

여기에서도 어떤 일이 일어났는지를 보면, 처음에 단순한 Publisher로 시작하고, 신호를 완화하고 중복을 제거했다. 그리고 Future를 사용해서 비동기 네트워크 호출을 하는 API를 감쌌고, flatMap을 사용해서 stream을 두 갈래로 나눴다. 그리고 이것들은 API 영역에 속하기 때문에 eraseToAnyPublisher를 사용했다.

<img width="671" alt="image" src="https://user-images.githubusercontent.com/41438361/157559055-a9646ef2-2056-4c2c-b591-86783022f5ce.png">

그래서 이제 두 개의 Publisher가 생겼고, 이 두 신호(하나는 기기내에서 이루어지고 다른 하나는 비동기 네트워크 호출에 의해 생긴다)를 가지고 앱의 UI를
활성화하거나 비활성화하는 것이다.

<img width="1776" alt="image" src="https://user-images.githubusercontent.com/41438361/157559248-b24a5907-7b4f-4684-b7dd-8566c738422c.png">

여기에 CombineLatest operator를 사용한다. 두 Publisher를 가지고, 유효하지 않음을 확인하고 optional tuple/nil을 리턴한다.

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/157559538-1b877379-0dca-4c1c-9c1e-f305d64c0e41.png">

이를 UI에 연결하는 것도 간단하다. 먼저 ViewController의 전체 생명주기에서 subscription을 관리하기 위해 변수를 생성한다. 그리고 map을 사용해
boolean 값을 버튼의 isEnalbed 프로퍼티에 매핑한다.

* 출처
* https://developer.apple.com/videos/play/wwdc2019/722/?time=124
* https://developer.apple.com/videos/play/wwdc2019/721
* https://www.bmc.com/blogs/asynchronous-programming/
