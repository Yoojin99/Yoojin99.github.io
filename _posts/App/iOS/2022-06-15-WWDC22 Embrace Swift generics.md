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

## some

<img width="1269" alt="image" src="https://user-images.githubusercontent.com/41438361/173776926-d2d1e9ff-c03a-493e-8286-31256e223f3d.png">
<img width="991" alt="image" src="https://user-images.githubusercontent.com/41438361/173777007-99e1b4cc-ebec-41ff-a675-909952a4ecec.png">

타입 파라미터를 명시적으로 쓰고 "where"문을 쓰는게 실제 그 문장보다 더 복잡하게 보인다. 타입 파라미터를 명시적으로 쓰는 대신에 protocol conformance를 `some Animal`과 같이 써서 표현할 수 있다. 

<img width="731" alt="image" src="https://user-images.githubusercontent.com/41438361/173935468-ab7a9c81-93a5-44b6-849e-7ed4fdb86831.png">

**"some" 키워드는 내가 사용하는 특정 타입(구체적인 타입)이 있다는 것을 의미하고, 뒤에 conformance 요구사항을 붙인다.**

또한 파라미터, 결과 타입에서 사용할 수 있다. 

<img width="1639" alt="image" src="https://user-images.githubusercontent.com/41438361/173937321-c9563491-6091-460b-b53f-39f3b1f346ab.png">

SwiftUI 코드를 작성한 적 있다면 위와 같은 코드를 본 적이 있을 것이다. Body 내에서 view의 특정 타입(구체적인 타입)을 리턴하는데 이 body를 사용하는 코드는 특정 타입이 무엇인지 알 필요가 없다.

## opaque type & underlying type

* **opaque type** : 특정 concrete type의 placeholder로서 동작하는 abstract type
* **underlying type** : placholder 자리에 들어가는 특정 concrete type

<img width="1654" alt="image" src="https://user-images.githubusercontent.com/41438361/173939458-e3e60f07-f779-4751-bf49-34010ce00f65.png"><img width="1287" alt="image" src="https://user-images.githubusercontent.com/41438361/173939878-8fc84278-e407-4e98-802b-20cadef7fec3.png">

`some` 키워드를 사용하는 타입과 괄호 사이의 타입 파라미터는 모두 opaque type을 정의한다. input, output에 모두 opaque 타입을 사용할 수 있기 때문에 파라미터나 결과 위치에서 정의될 수 있다.

Named type paramenter는 항상 input 쪽에 위치하고 있기 때문에 호출하는 곳에서 underlying 타입을 결정하고, 구현쪽에서 추상화된 타입을 사용한다. 

일반적으로 opaque 파라미터나 result 타입의 값을 제공하는 부분에서 underlying type을 결정하고, 해당 부분을 사용하는 곳에서는 추상화 타입을 본다. 무슨 말인지 알아보자.

## local variable

<img width="1718" alt="image" src="https://user-images.githubusercontent.com/41438361/173940767-09b6bacf-ae79-431e-a5c8-19efc56d2415.png">

Underlying 타입은 값에서 추론된다. 로컬 변수의 경우 underlying 타입은 오른쪽 위치의 값에서 추론된다. 

<img width="892" alt="image" src="https://user-images.githubusercontent.com/41438361/173941023-f714be81-e5da-40c6-b8cb-5b1e4f5b04d6.png"><img width="892" alt="image" src="https://user-images.githubusercontent.com/41438361/173941118-b3c71a81-234d-4068-889c-9308ee91e410.png">

이는 opaque 타입을 가진 지역 변수가 항상 초깃값을 가져야 한다는 것을 뜻하고 초깃값을 제공하지 않으면 컴파일러가 에러를 발생시킨다. 또한 underlying 타입은 고정되어야 하기 때문에 underlying 타입을 바꾸려고 한다면 에러가 발생할 것이다.

## parameter

<img width="1716" alt="image" src="https://user-images.githubusercontent.com/41438361/173941360-f5b08c3d-e488-4751-99a7-86e4a03edd30.png">

Opaque 타입을 파라미터에서 쓰면 underlying 타입은 호출되는 곳의 인자 값에서 추론된다. 파라미터 위치에서 "some"을 쓰는 것은 Swift 5.7부터 가능하다. Underlying 타입은 파라미터의 범위만을 제한하기 때문에 호출을 하는 곳들에서 다른 타입의 인자를 전달할 수 있다.

## results

<img width="1732" alt="image" src="https://user-images.githubusercontent.com/41438361/173942303-92a57849-2c6e-49ee-aa1a-bc0743f82893.png">

Opque 결과 타입을 가진 메서드나 연산 프로퍼티는 어디서든지 호출될 수 있기 때문에, 리턴되는 값의 타입이 유일해야 한다.

<img width="840" alt="image" src="https://user-images.githubusercontent.com/41438361/173942588-23c495ca-1d0a-45e1-81ce-939a44b0dadb.png"><img width="1175" alt="image" src="https://user-images.githubusercontent.com/41438361/173942658-8eae5b9a-be86-48f6-8045-e1b241f429ec.png">

만약 리턴되는 타입이 여러개라면 컴파일러가 에러를 발생시킨다.

<img width="818" alt="image" src="https://user-images.githubusercontent.com/41438361/173942832-9c920678-ee8d-414b-af85-f9d80ad39a59.png">

SwiftUI view의 opaque 타입은 ViewBuilder DSL이 같은 underlying return type을 갖게 하도록 변환할 수 있다. 그

`@ViewBuilder` annotation 을 메서드에 작성하고 return 문을 없애면 결과가 ViewBuilder 타입에 의해 만들어진다.

<img width="1304" alt="image" src="https://user-images.githubusercontent.com/41438361/173962808-d8a13b3c-5e9a-4d28-b7ef-b39ccc456dbb.png">

## back to farm

`Farm`의 `feed` 메서드를 보면, opaque 타입을 다른 곳에서 참조할 필요가 없기 때문에`some`을 파라미터 리스트에 쓸 수 있다. opaque 타입을 함수 signature에서 여러 번 사용한다면 name 타입 파라미터가 유용하다. 

가령 `Animal` 프로토콜에 `Habitat`이라는 연관타입을 추가하고, 농장에 있는 동물에게 서식지를 지어주고 싶다고 해보자. 이 경우 result 타입은 특정 동물 타입에 따라 달라질 것이기 때문에 타입 파라미터 `A`가 인자와 리턴 타입에서 필요하다.

<img width="747" alt="image" src="https://user-images.githubusercontent.com/41438361/173963278-1d493070-fb8e-409c-afdb-b5455fa7d075.png">

코드는 타입 파라미터를 제네릭 타입, 저장 프로퍼티의 타입 파라미터, 멤버와이즈 이니셜라이저에 사용할 수 있다. 다른 문맥에서 제네릭 타입을 참조하려면 괄호 안에 명시적으로 특정 타입을 적어야 한다. 정의에 있는 괄호는 제네릭 타입이 어떤 타입을 사용할 것인지를 정해주기 때문에 opaque 타입은 항상 제네릭 타입에서 이름이 붙여져야 한다.

<img width="799" alt="image" src="https://user-images.githubusercontent.com/41438361/173963648-bde84438-3c58-4bc6-ae1f-179c924c8c43.png">

* `crop` : `animal` 파라미터의 타입을 사용해서 `Feed` 연관 타입에 접근해서 `Feed.grow()` 메서드를 호출할 수 있다. `Feed.grow()`를 통해 작물의 인스턴스를 받는다. 

`animal`의 underlying 타입이 고정되었기 때문에 컴파일러는 `crop`, `produce`, `animal` 타입 간의 관계를 알 수 있다. 이런 정적 관계를 통해 동물에게 이상한 종류의 음식을 먹이는 걸 방지할 수 있다.

<img width="793" alt="image" src="https://user-images.githubusercontent.com/41438361/173964086-7f118660-63e4-4347-9c86-72e9d9291e1c.png">

`feedAll` 메서드는 다양한 동물들의 배열을 받아 먹이를 주는 메서드다. 이 배열이 다른 종류의 동물들도 저장하게 하고 싶다. `some Animal`을 사용할 수 있을까?

<img width="1175" alt="image" src="https://user-images.githubusercontent.com/41438361/173964175-e22fd9a9-3386-430e-a2ae-4458e7a2eeff.png">

`some`은 underlying type을 고정시키기 때문에 다양한 타입을 받을 수 없다. Underlying 타입이 고정돼서 배열 내의 모든 요소가 같은 타입을 가져야 한다. 

## any

<img width="670" alt="image" src="https://user-images.githubusercontent.com/41438361/173964296-88f89e56-04fa-47e4-8f27-9a21c3a54714.png">

임의의 `animal` 타입을 표현하기 위해 `any Animal`이라고 쓸 수 있다. **`any` 키워드는 이 타입이 어떤 임의의 `animal` 타입을 저장할 수 있고, underlying 타입은 런타임때 달라질 수 있음을 나타낸다.** `some`과 마찬가지로 뒤에 conformance 요구사항이 따라온다.

**`any Animal`은 동적으로 어떤 종류의 concrete animal 타입이라도 저장할 수 있는 능력을 가진 하나의 정적 타입이고, 값 타입에서 subtype polymorphism을 사용할 수 있게 한다.**

## Existential types provide type erasure

<img width="1603" alt="image" src="https://user-images.githubusercontent.com/41438361/173964655-6c65f217-cbda-4c97-87af-cc728296ac7e.png">

`any Animal`을 박스라고 생각해보자. 박스에 바로 들어갈 수 있을만큼 작은 값이 있을 것이고, 박스에 들어가기는 너무 커서 값은 다른 곳에서 할당되고, 박스는 그 값을 가리키는 포인터를 저장하고 있을 수 있다.

`any Animal`이 동적으로 저장할 수 있는 어떤 concrete animal을 **existential 타입**이라고 한다. 그리고 **다른 concrete 타입들을 모두 표현하기 위해 하나의 표현법을 사용하는 전략을 "type erasure"라 한다.** 컴파일 타임에서 concrete 타입은 지워지고, concrete 타입은 런타임에서만 알 수 있다.

위의 두 종류의 existential 타입은 같은 정적 타입을 갖지만, 다른 동적 타입을 갖는다. 

<img width="1229" alt="image" src="https://user-images.githubusercontent.com/41438361/173966091-c6def0c1-7f36-4006-866f-ac2d8a850405.png">

Type erasure는 다른 animal 값 사이의 type-level 차이를 없애서 다른 동적 타입들을 같은 정적 타입으로서 사용할 수 있게 해준다. `feedAll` 메서드에서도 type erasure를 쓸 수 있다.

<img width="1696" alt="image" src="https://user-images.githubusercontent.com/41438361/173966684-31aef303-a1f5-4ce7-a5d3-381f10e791ae.png">

Swift 5.7부터 `any` 키워드를 연관 타입을 가진 프로토콜에 사용할 수 있다.

<img width="863" alt="image" src="https://user-images.githubusercontent.com/41438361/173967260-1358b84b-50fc-4461-8e55-ceb3b20db672.png">

`feedAll` 메서드 안에서 동물마다 돌며 `Animal` 프로토콜의 `eat` 메서드를 호출했다. 이 메서드 안에서는 특정 underlying 동물 타입의 `Feed` 타입을 알아야 한다. 하지만 컴파일 에러가 발생하는데, 특정 동물 타입 간의 type-level 차이를 없애서 특정 animal 타입에 의존하는 타입 관계들(연관 타입 포함)도 없어졌기 때문이다. 그래서 `Feed` 가 무엇인지 알 수 없다. `any Animal`에서 바로 `eat`를 호출하는 것이 아니라 `some Animal`에서 `feed` 메서드를 호출해야 한다.

<img width="1686" alt="image" src="https://user-images.githubusercontent.com/41438361/173968753-5aac1b7a-33a9-4ce3-8cbf-6047e088d5bf.png">

`any Animal`은 `some Animal`과는 다른 의미를 갖는데, 컴파일러는 `any Animal`의 인스턴스를 박스를 열고 내부의 underlying 값을 꺼내서 `some Animal`로 만들어줄 수 있다. Swift 5.7에서부터 이런 unboxing이 가능하다. 컴파일러가 박스를 열고, 내부에 저장된 값을 꺼낸다고 생각하면 된다.

`some Animal`은 고정된 underlying 타입을 가지고 있기 때문에 underlying 타입의 모든 연산, 연관 타입에 접근할 수 있다. 

<img width="766" alt="image" src="https://user-images.githubusercontent.com/41438361/173970579-8562bf6b-4bcc-4083-96d7-66e995c64222.png">

`feed` 메서드에 각 animal을 전달해서 구현할 수 있다. 

## some vs any

<img width="1642" alt="image" src="https://user-images.githubusercontent.com/41438361/173970663-05f80d96-bb0b-4c79-93d5-e1408f49576a.png">

* some : underlying 타입이 고정된다. 제네릭 코드에서 underlying 타입에 type 관계가 형성되기 때문에 사용하는 프로토콜의 API와 연관 타입에 접근할 수 있다.
* any : 임의의 concrete 타입을 저장할 때 사용한다. Type erasure를 지원해서 다양한 타입의 컬렉션을 나타낼 수 있고, underlying 타입을 명시하지 않아도 되며, 옵셔널을 사용할 수 있고 구현 디테일을 추상화할 수 있다.

<img width="1293" alt="image" src="https://user-images.githubusercontent.com/41438361/173970996-55c31896-fb15-47aa-a545-e3a2221f9f65.png">

영상에서는 "some"을 기본적으로 사용하고 임의의 값을 저장해야 하는 곳에서만 "some"을 "any"로 바꾸라고 하고 있다.
