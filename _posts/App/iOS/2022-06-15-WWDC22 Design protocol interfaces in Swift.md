---  
layout: post  
title: "[iOS] - Design protocol interfaces in Swift"  
subtitle: ""  
categories: app
tags: app-ios wwdc
comments: true  
header-img: 

---  

[Design protocol interfaces in Swift](https://developer.apple.com/videos/play/wwdc2022/110353/)

# Understand type erasure

연관 타입을 가진 프로토콜이 existential 타입과 어떻게 상호작용하는지 볼 것이다.

<img width="695" alt="image" src="https://user-images.githubusercontent.com/41438361/173996757-8f41c987-a668-40a9-b7a1-633d5354d9dd.png">

`Animal` 프로토콜에 `produce()` 메서드를 추가해서 음식을 생산하는 것을 추상화했다. 다른 동물마다 달라지는 `produce()`의 리턴 타입을 추상화하는 가장 좋은 방법은 연관 타입을 쓰는 것이다.

연관 타입을 써서 어떤 `Animal` concrete 타입에서 `produce()`를 호출하면 concrete `Animal` 타입에 따라 특정 타입의 `Food`가 리턴됨을 표현했다. 

<img width="957" alt="image" src="https://user-images.githubusercontent.com/41438361/173997085-2bbe5c84-566e-49a9-a8fa-7a6637b56448.png">

다이어그램으로 나타내면 위와 같다. `Self` 는 `Animal` 프로토콜을 준수하는 실제 concrete 타입이 된다. `Self`는 연관 타입인 `Commodity`타입이 있고, 이 연관 타입은 `Food`를 준수한다.

<img width="1388" alt="image" src="https://user-images.githubusercontent.com/41438361/173997266-6e667aee-9a1f-4497-a6c6-3d8ab57c3cff.png">

`Animal` 프로토콜을 구현한 concrete 타입 `Cow`는 `Animal` 프로토콜을 준수하고 `CommodityType`으로 `Milk`를 가지고 있다.

<img width="771" alt="image" src="https://user-images.githubusercontent.com/41438361/173997425-23b8e568-c676-4bd0-9ca9-ec61257110ac.png">

`Farm`의 `animals` 배열은 `any Animal`의 heterogenous 배열이다. 

<img width="1194" alt="image" src="https://user-images.githubusercontent.com/41438361/173997507-ec917d1f-999d-4433-acb9-91cab818edf7.png">

`any Animal` 타입은 동적으로 어떠한 concrete 동물 타입이라도 저장할 수 있는 박스를 표현한 것과 같다. 이렇게 **다른 concrete 타입들을 모두 나타내기 위해 하나의 표현법을 사용하는 것을 type erasure라 한다.**

<img width="1384" alt="image" src="https://user-images.githubusercontent.com/41438361/173997840-8b4ca45d-5aec-4267-a304-49cc7c17e935.png">

`produceCommodities()` 메서드는 동물들의 배열을 map을 통해 한 번씩 돌면서 `produce()` 메서드를 호출한다. 

* `animal` : map 클로저에 있는 `animal` 파라미터는 `any Animal` 타입이다.
* `produce()` : `produce()`의 리턴타입은 연관 타입이다.

**Existential type에 연관 타입을 리턴하는 메서드를 호출하면 컴파일러는 type erasure를 사용해서 호출의 결과 타입이 무엇인지 결정한다.** 

<img width="924" alt="image" src="https://user-images.githubusercontent.com/41438361/173998584-c49b2fca-5b55-4028-93c1-c85be0696f1d.png">

Type erasure는 연관 타입을 동일한 제약을 가진 일치하는 existential type으로 대체한다.

*Existential type? : 특정 추상 타입을 준수할 수 있는 concrete 타입을 existential type이라 한다.*

<img width="1402" alt="image" src="https://user-images.githubusercontent.com/41438361/173998699-075e3da9-9d1b-4bde-aa0b-ffc31ccb68c4.png">

Concrete Animal 타입과 연관 타입인 `CommodityType`의 관계를 `any Animal`과 `any Food`로 대체해서 지워버렸다. `any Food` 타입은 연관 타입인 `CommodityType`의 upper bound라고 불린다.

`produce()` 메서드가 `any Animal`에서 호출되므로 리턴 타입은 지워지고, `any Food` 타입의 값을 준다.

## Type erasure semantics

<img width="1735" alt="image" src="https://user-images.githubusercontent.com/41438361/173999087-a5a2eec9-4077-411c-9f1c-cff8b5b0ceef.png">

Swift 5.7에서는 새로운 기능인 associated-type erasure를 지원한다. 프로토콜 메서드의 결과 타입에 나타나는 연관 타입(화살표 오른쪽에 나온 부분)은 "producing position"이라고 불리는데, 왜냐하면 메서드를 호출하는 것이 이 타입의 값을 생성할 것이기 때문이다.

`any Animal`에서 이 메서드를 호출할 때 컴파일 타임에서는 concrete 결과 타입을 모르지만, upper bound(`any Food`)의 subtype이라는 것은 알고 있다.

<img width="1744" alt="image" src="https://user-images.githubusercontent.com/41438361/173999534-5feef91c-da71-4585-8cf2-0ac3d974da05.png">

예를 들면 런타임에서 `Cow` 객체를 가진 `any Animal`에 `produce()` 메서드를 호출하고 있다. `Cow`에 `produce()` 메서드를 호출하면 `Mail`를 리턴할 것이다.
`Milk`는 `any Food` 내에 저장될 수 있다. 그리고 `any Food`는 `Animal` 프로토콜의 연관 타입인 `CommodityType`의 upper bound가 된다.

<img width="1741" alt="image" src="https://user-images.githubusercontent.com/41438361/173999868-158d37ff-f32b-4adc-86a7-94254123608e.png">

연관 타입이 메서드나 이니셜라이저의 파라미터 리스트에서 사용된 경우를 보자. 위 코드에서 `eat` 메서드는 consuming position에서 연관 타입인 `FeedType`을 가지고 있다.
(위에서 메서드의 리턴타입에 쓰여진 연관 타입을 producing position에 있다고 한 걸 기억할 것이다. 리턴 타입은 producing position에 있다 하니 메서드에 전달하는 파라미터는 consuming position에 있다고 한 것 같다.)

<img width="1733" alt="image" src="https://user-images.githubusercontent.com/41438361/174000275-cb576ce6-8c1a-41a5-abd9-8629c9c83c82.png">

다시 예시를 보면 `Cow`를 저장한 `any Animal`이 있다. `Animal` 프로토콜의 연관 타입인 `FeedType`의 upper bound는 `any AnimalFeed`다.

<img width="1731" alt="image" src="https://user-images.githubusercontent.com/41438361/174003050-3cc0b1aa-969e-4d45-bc0e-6902bd4b759a.png">

하지만 임의의 `any AnimalFeed`가 주어졌을 때 그 타입이 `Hay` concrete 타입을 저장하고 있다는 보장을 할 수 없다. Type erasure는 consuming position에서 연관타입을 사용할 수 없게 한다. 대신, existenial `any` 타입의 박스를 열고 이를 함수에 넘겨줘야 한다.

---

여기까지 봤을 때 이해하기 힘든 부분이 있어서 정리하고 넘어간다.

<img width="1731" alt="image" src="https://user-images.githubusercontent.com/41438361/174003659-07f383e3-c837-43f3-8e98-7ea97110c1f6.png">

여기에서 `animal.produce()`를 했을 때 `Milk`를 리턴한다는 것을 알 수 있었던 이유? : 연관 타입이 producing position에 쓰였다. `any Animal`에 `produce()` 메서드를 호출하면 결과로 어떤 구체적인 타입이 들어올 지 컴파일 타임에서는 알 수 없는데 upper bound인 `anyFood`의 subtype이라는 것은 알 수 있다.

<img width="1730" alt="image" src="https://user-images.githubusercontent.com/41438361/174004056-b525db6e-0593-4b84-ab05-b4eaed198de9.png">

그렇다면 메서드의 파라미터에 연관 타입이 있는 경우는 왜 안되는가? : 연관 타입이 consuming position에 쓰였다. 전환이 다른 방향으로 일어나서 type erasure를 쓸 수 없다. 연관 타입의 upper bound dexistential 타입이 실제 concrete type으로 안전하게 변환될 수 없는데, 그 이유는 concrete type이 무엇인지 알 수 없기 때문이다.

---

## Type erasure with 'Self' result type

<img width="1724" alt="image" src="https://user-images.githubusercontent.com/41438361/174006100-f2955e3f-1181-4c05-8866-7442327dcaab.png">

연관 타입과 관련된 type erasure 행위는 Swift 5.6에서 볼 수 있는 기능과 비슷하다. 참조 타입을 복제하는 프로토콜이 있다고 해보자.

프로토콜은 `Self`를 리턴하는 메서드 `clone()`을 정의한다. `any Cloneable` 타입에 `clone()`을 호출하면 결과 타입은 `Self`가 되고, upper bound로 type erasing된다. `Self` 타입의 upper bound는 프로토콜 자체이므로, 새로운 `any Cloneable`을 받게 되는 것이다.

## Summary

* `any`를 사용해서 특정 프로토콜을 준수하는 concrete 타입을 저장한 existential 타입을 정의할 수 있다. 이제 연관 타입을 가진 프로토콜에서도 동작한다.
* Producing position에서 연관 타입을 사용한 프로토콜 메서드를 호출할 경우 연관 타입은 upper bound(연관 타입 제약을 가진 다른 existential type)로 type-erasing된다. 

# Hide implementation details

<img width="653" alt="image" src="https://user-images.githubusercontent.com/41438361/174015066-4018d2e0-a70d-408f-b9c0-ad34daea37d9.png">

`feedAnimals()` 메서드는 배고픈 동물을에게 먹이를 주는 메서드다. `hungryAnimals`를 연산 프로퍼티로 만들 었다. `filter()`를 `any Animal`에 적용하면 `any Animal`의 새로운 배열이 리턴된다.
`feedAnimals()`는 `hungryAnimals`를 한 번 반복하고 즉시 이 임시 배열을 버린다. 이런 작업은 배고픈 동물들이 많을 때 비효율적이다.

<img width="1021" alt="image" src="https://user-images.githubusercontent.com/41438361/174015710-f78be306-5e92-499f-9194-b37cfb5b6c01.png">

이런 temporary allocation을 피하는 방법 중 하나는 표준 라이브러리의 lazy collection 기능을 쓰는 것이다. `filter`를 `lazy.filter`로 바꾸면 lazy collection을 받을 수 있다.
lazy collection은 `filter`의 결과와 같은 결과를 리턴하는데 temporary allocation을 피한다. 하지만 이제 `hungryAnimals` 프로퍼티의 타입은 복잡한 concrete 타입으로 명시되어야 한다.
이는 불필요한 구현 디테일을 노출한다.

클라이언트, `feedAnimals()`는 우리가 `hungryAnimals`를 구현할 때 `lazy.filter`를 썼는지 알 필요가 없고, 내부를 반복할 수 있는 컬렉션을 받을 수 있다는 것만 알면 된다.

<img width="700" alt="image" src="https://user-images.githubusercontent.com/41438361/174016165-3b16af05-79e3-4af9-8b28-caad1c79da64.png">

Opaque result 타입은 복잡한 concrete 타입을 Collection의 추상화된 인터페이스 뒤에 감출 수 있다. 이제 `hungryAnimals`를 호출하는 클라이언트는 그들이 Collection 프로토콜을 준수하는 어떤 concrete 타입을 받는다는 사실만 알게 된다. 하지만 collection의 구체적인 타입은 모르게 된다.

<img width="699" alt="image" src="https://user-images.githubusercontent.com/41438361/174016516-547e2b50-18e2-4e37-bd56-e5df638e7e96.png">

하지만 이 방법도 정보를 클라이언트에게 너무 많이 감춘다. 우리는 collection의 element 타입을 알 수 없기 때문에 할 수 있는 일은 요소를 전달하는 것이지 `Animal` 프로토콜의 어떤 메서드도 호출할 수 없다.

<img width="1742" alt="image" src="https://user-images.githubusercontent.com/41438361/174016812-6974ce5e-533f-4f06-9ab3-473be48943b6.png">

"some Collection"을 쓸 때 constrained opaque result type을 사용해서 구현 디테일을 감추고 인터페이스에 대한 정보를 노출시키는 정도를 적당히 조절할 수 있다.
Swift 5.7에서부터 Constrained opaque result type을 쓸 수 있다. 

Constrained opaque result type은 프로토콜 이름 뒤에 괄호와 타입 인자를 쓰는 것이다.

<img width="904" alt="image" src="https://user-images.githubusercontent.com/41438361/174017283-6335337b-7d8f-40db-be06-3ad3edcf293f.png">

`hungryAnimals`를 constrained opaque result type으로 정의하면 클라이언트는 이게 "any Animal의 배열의 LazyFilterSequence"라는 것은 모르지만 Collection을 준수하는 concrete type이고, Element 연관 타입은 `any Animal`과 같다는 것을 알게 된다.
이제 `feedAnimals()`내의 반복문에서 `animal` 변수는 `any Animal` 타입을 갖기 때문에 `Animal` 프로토콜 내의 메서드를 호출할 수 있다.

<img width="727" alt="image" src="https://user-images.githubusercontent.com/41438361/174017854-4bff296f-590d-4c0f-919c-6d77d54ebd20.png">

이게 동작할 수 있는 이유는 Collection 프로토콜이 Element 연관 타입을 "primary associated type"으로 정의하고 있기 때문이다.

프로토콜 뒤에 괄호를 붙이고 하나 이상의 연관 타입을 써서 primary 연관 타입을 설정할 수 있다. 

<img width="1729" alt="image" src="https://user-images.githubusercontent.com/41438361/174018231-9293fdfc-7315-4498-a6a2-5c984ec56247.png">

`Collection<Element>`는 `some` 키워드를 사용해서 opaque result type으로도 사용될 수 있고, `any`를 써서 constrained existential 타입으로도 사용할 수 있다.

<img width="1045" alt="image" src="https://user-images.githubusercontent.com/41438361/174018595-51dcc29f-3561-4ad0-b3a9-13747053f41e.png">
<img width="893" alt="image" src="https://user-images.githubusercontent.com/41438361/174018653-5652f6c8-3275-42b3-b4be-5f486576d210.png">

any Animal의 opaque Collection을 사용하면 함수에서 다른 underlying 타입을 사용했을 때 에러가 발생하게 된다. 이를 `any Collection<any Animal>`로 바꿔서 API가 다른 타입을 리턴할 수 있음을 알려야 한다.

# Identify type relationships

Opaque 타입을 써서 제네릭 코드를 작성하는 것은 abstract type relationship에 기반해야 한다.

<img width="690" alt="image" src="https://user-images.githubusercontent.com/41438361/174021225-30e753af-31c4-478c-8c82-a6a0bb03b3b5.png">

`Animal` 프로토콜에 




