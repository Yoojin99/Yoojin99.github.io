---  
layout: post  
title: "[iOS] - Embrace Swift generics"  
subtitle: ""  
categories: app
tags: app-ios wwdc
comments: true  
header-img: 

---  

[Embrace Swift generics](https://developer.apple.com/videos/play/wwdc2022/110352)

<img width="1692" alt="image" src="https://user-images.githubusercontent.com/41438361/173763802-247156f8-0001-43a4-bb0f-5ffff2387738.png">

**추상화는 구체적인 디테일에서 개념을 분리한다.**

<img width="862" alt="image" src="https://user-images.githubusercontent.com/41438361/173764248-77527c51-1fcf-4088-84b9-c21f3869cf24.png">

추상화를 코드에 적용하는 방법은 다양한다. 코드를 함수나 지역 변수로 만들어서 여러 번 사용되는 기능이나 값을 재사용하기 편하게 만들 수 있다.

기능을 함수로 추상화하면 디테일은 추상화돼서 없어지고, 추상화를 사용한 코드는 디테일 없이 무엇이 일어나는지에 대한 개념을 표현할 수 있다.

<img width="1388" alt="image" src="https://user-images.githubusercontent.com/41438361/173764615-398286ff-90f4-4567-a2a0-a81006d2d94f.png">

**Swift에서는 concrete type 자체를 추상화할 수 있다.** 같은 개념을 갖고 있지만 디테일이 다른 타입들이 있는 경우 이 concrete type들을 모두 다루기 위해 추상화된 코드를 작성할 수 있다.

# Model with concrete types

<img width="1628" alt="image" src="https://user-images.githubusercontent.com/41438361/173764930-be228a63-583c-471c-b2ec-3797faf4ddf0.png">

* Cow라는 concrete type이 있고, 구조체로 정의됐다. Cow는 `eat`라는 메서드를 있고 인자로 `Hay`를 받는다.
* `Hay`는 `grow`라는 메서드가 있고 `Hay`를 생성하는 작물 `Alfalfa`를 기른다.
* `Alfalfa` 구조체는 `Hay`를 수확하는 메서드가 있다.
* `Farm`은 소에게 밥을 주는 메서드가 있다.

<img width="671" alt="image" src="https://user-images.githubusercontent.com/41438361/173765517-98e84630-405f-450c-a6b4-d463db975e99.png">

더 많은 동물을 추가하고 싶다. `Farm`에서 모든 동물들에게 밥을 주게 하고 싶으면 어떻게 해야 할까?

## overloading

<img width="726" alt="image" src="https://user-images.githubusercontent.com/41438361/173765787-90098d37-7464-46ae-a640-5c1d2332cb00.png">
<img width="631" alt="image" src="https://user-images.githubusercontent.com/41438361/173765874-8175a7ce-1f06-4b80-a6e8-48cc547ead76.png">

`feed` 메서드를 overload해서 각각 다른 타입의 인자를 받게 할 수 있지만, 내부 구현은 굉장히 비슷할 것이기 때문에 동물들이 추가될 때마다 boilerplate 코드가 생겨나게 된다.

이렇게 **반복적으로 구현된 overload들이 있다면 이를 제네릭으로 작성해야 하는 신호라고 볼 수 있다.** 이 구현들이 비슷한 이유는 동물들이 비슷한 기능을 하기 때문이다.

# Identify common capabilities

<img width="560" alt="image" src="https://user-images.githubusercontent.com/41438361/173766589-5f1781ea-ba20-4096-83aa-38976bbabe9c.png">

이제 각 동물 타입에서 공통되는 부분을 정의해야 한다. 각 동물 타입은 모두 먹는다는 동작을 하지만, 각 동물들은 다른 종류의 먹이를 먹기 때문에 구현을 할 때는 각각 다르게 동작하게 해야 한다.

여기에서 하고 싶은 것은 `eat` 메서드를 추상화해서 concrete type에 따라 다르게 동작하게 하고 싶은 것이다.

<img width="1505" alt="image" src="https://user-images.githubusercontent.com/41438361/173767422-7f87989f-407b-4867-95b0-235b5c36347e.png">

**다른 concrete 타입에 따라 다르게 동작하게 하는 추상화된 코드의 능력을 "다형성"이라고 한다.** 다형성은 한 코드가 코드가 어떻게 사용됐는지에 따라 다른 동작을 하게 한다.

<img width="1039" alt="image" src="https://user-images.githubusercontent.com/41438361/173767740-ab63a2a7-92be-469c-993d-92c673c0494f.png">

다형성은 다향한 형태로 구현할 수 있다.

1. Overloading : 인자 타입에 따라 같은 함수 호출이 다른 의미를 지닐 수 있게 한다. "ad-hoc polymorphism"이라고도 불리는데, overloading이 일반적인 해결책이 아니기 때문. 위에서 봤듯이 반복적인 코드를 생성할 수 있다.
2. Subtypes : Supertype에서 동작하는 코드가 런타임에서 코드를 사용하고 있는 subtype에 따라 다르게 동작하게 하는 것이다.
3. Generic : Type 파라미터를 사용해서 다른 타입에서 동작하는한 코드를 작성할 수 있게 하고, concrete 타입 자체가 인자로 사용된다.

## Subtype polymorphism

Subtype 관계를 나타내는 방법은 클래스 계층을 사용하는 것이다.

<img width="1278" alt="image" src="https://user-images.githubusercontent.com/41438361/173768708-9f7f903e-f705-4f16-883a-f43151137350.png">

* `Animal` 클래스를 생성하고, 각 동물 구조체를 클래스 타입으로 바꾼다.
* 각 동물 클래스가 `Animal` 클래스를 상속하고, `eat` 메서드를 override한다.

이를 통해 모든 동물 타입을 나타내는 추상화된 base 클래스 `Animal`이 생겼다. `Animal` 클래스를 호출하는 코드는 subtype polymorphism에 따라 자식클래스에 구현된 내용을 호출할 것이다.

위 코드는 여러 문제가 있다.

1. `Animal`의 `eat` 메서드에 파라미터 타입을 아직 채우지 못했다.
2. 클래스를 사용해서 다른 동물 인스턴스간의 상태가 필요하지 않고 공유되게 하고 싶지 않음에도 불구하고 참조 문법을 썼다.
3. 자식 클래스를 구현할 때 까먹고 base 클래스의 메서드를 override하지 않으면 런타임 에러가 발생한다.
4. **각 동물 subtype이 다른 종류의 음식을 먹는데 이런 의존성은 클래스 계층에서 표현하기 어렵다.**

<img width="1540" alt="image" src="https://user-images.githubusercontent.com/41438361/173771235-e5125030-8b5e-44d3-98f5-22324f48b902.png">

클래스 계층에서 각 동물들이 다른 종류의 음식을 먹는 것을 처리하기 위해서 `Any`를 사용할 수 있다. 하지만 여전히 문제가 있다.

* 오버라이딩 된 메서드에서 추가적인 boilerplate 코드가 생성된다.
* 잘못된 음식 종류를 전달했을 때 런타임에서 에러가 나는 문제가 발생한다.

<img width="1315" alt="image" src="https://user-images.githubusercontent.com/41438361/173771714-f6e225db-4128-4faf-8f5e-3154a27a8842.png">
<img width="1289" alt="image" src="https://user-images.githubusercontent.com/41438361/173772488-f453f554-d6f5-4809-b989-bca4490acc66.png">

동물의 먹이에 type-safe한 방법을 적용하기 위해 타입 파라미터를 받게 할 수 있다. 하지만 이마저도 부자연스럽다.

* 동물 클래스를 정의하기 위해 먹이 클래스를 항상 함께 정의해야 한다.
* 먹는다는 동작이 동물의 핵심 목적이 아니다.
* 동물 클래스들 내의 대부분의 코드는 음식이 뭔지 신경쓰지 않는다.
* 각 동물에 더 많은 타입과 관련된 정보를 제공할 때 boilerplate 코드가 생성되게 된다.

따라서 지금까지 시도한 방법 중에 이 상황에서 좋은 방법은 없다. 근본적인 문제는 **클래스는 데이터 타입이고, 부모 클래스가 구체적인 타입들의 추상화된 개념을 표현하도록 만들고 있다는 것이다. 대신, 디테일한 내용 없이 타입들이 할 수 있는 동작을 나타내기 위한 구조를 언어로 나타내려고 한다.**

<img width="1047" alt="image" src="https://user-images.githubusercontent.com/41438361/173773082-59821068-7e91-43a8-90b5-90c84c56de64.png">

동물을 두 개의 공통 능력(capability)가 있다.

1. 특정 타입의 먹이
2. 먹이를 먹는 연산

# Build an interface

<img width="777" alt="image" src="https://user-images.githubusercontent.com/41438361/173773389-c0e71df2-36f8-4409-ac20-68ed76b50da7.png">

위에서 정의한 능력을 나타내는 인터페이스를 **protocol**로 작성할 수 있다.

<img width="1684" alt="image" src="https://user-images.githubusercontent.com/41438361/173773472-a583f3eb-624e-43a0-b343-5fd9b7e9eb3e.png">

**프로토콜은 프로토콜을 준수하는 타입(conforming type)의 기능을 설명하는 추상화 도구다.** 프로토콜을 사용하면 구현 디테일에서 타입의 능력에 대한 개념을 분리할 수 있다.
즉 타입이 무엇을 할 수 있는지에 대한 개념이 인터페이스를 통해 표현된다.

<img width="1724" alt="image" src="https://user-images.githubusercontent.com/41438361/173773894-7bb7c3c4-1148-4758-9a13-948830c5a929.png">

동물의 능력을 프로토콜 인터페이스로 변환했다.

1. associatedtype `Feed` : 음식의 종류. 연관 타입은 구체적인 타입의 placeholder로 동작한다. 연관타입이 특별한 이유는 프로토콜을 준수하는 특정 타입에 의존한다는 것이다. 따라서 한 구체적인 타입의 인스턴스는 같은 종류의 음식을 가진다는 관계가 보장된다.
2. `eat` 메서드 : 동물의 Feed 타입을 파라미터 타입으로 설정했다. 프로토콜은 이 메서드를 구현하지 않고 구체적인 동물 타입이 이를 구현한다.

<img width="1212" alt="image" src="https://user-images.githubusercontent.com/41438361/173775057-e90fa889-7bc3-4183-a4e0-e44d0a0548ca.png">

`Animal` 프로토콜을 생성하고, 구체적인 동물 타입들이 이를 채택하게 했다. 프로토콜은 클래스에만 한정되지 않고 구조체, 열거형, actor에서도 사용될 수 있다.

* `: Animal` conformance annotation : 컴파일러가 구체적인 타입이 프로토콜 요구사항을 만족했는지 확인한다. 컴파일러는 Feed 타입이 무엇인지 추론할 수 있는데, 파라미터 리스트에서 사용됐기 때문이다. Feed 타입은 typealias를 사용해서 명시적으로도 표현할 수 있다.

# Write generic code

<img width="717" alt="image" src="https://user-images.githubusercontent.com/41438361/173775849-19ccc645-027e-44cc-8712-ac0e5c5600ce.png">

`Farm`의 `feed` 메서드를 구현할 때 `Animal` 프로토콜을 사용할 수 있다. Parametric polymorphism을 사용해서 메서드가 호출될 때 실제 타입으로 대체될 타입 파라미터를 사용할 것이다.

<img width="804" alt="image" src="https://user-images.githubusercontent.com/41438361/173775920-3a26dac6-8c74-48d3-af46-3cfb58e55e77.png">
<img width="949" alt="image" src="https://user-images.githubusercontent.com/41438361/173776009-54346f35-15d3-48c6-bd06-986cb4ab0d73.png">

타입 파라미터는 함수 이름 뒤에 `<>` 기호 사이에 올 수 있다. 타입 파라미터의 이름을 사용해서 참조할 수 있다. 

1. 타입 파라미터는 `Animal` 프로토콜을 준수해야 하므로 타입 파라미터를 protocol conformance를 써서 작성한다.
2. `where` 문뒤에 붙여서 type conformance를 나타낼 수도 있다.

<img width="1269" alt="image" src="https://user-images.githubusercontent.com/41438361/173776926-d2d1e9ff-c03a-493e-8286-31256e223f3d.png">
<img width="991" alt="image" src="https://user-images.githubusercontent.com/41438361/173777007-99e1b4cc-ebec-41ff-a675-909952a4ecec.png">


