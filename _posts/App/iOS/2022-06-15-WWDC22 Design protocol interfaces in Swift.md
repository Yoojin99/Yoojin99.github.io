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

`Animal` 프로토콜에 구체적인 동물 먹이 타입을 위해 새로운 연관 타입을 추가했다. 더불어 `eat()` 메서드에서 이 먹이 타입을 사용한다.

<img width="724" alt="image" src="https://user-images.githubusercontent.com/41438361/174022881-31f4a805-e416-4123-a2ac-fcd0ffa41f76.png"><img width="701" alt="image" src="https://user-images.githubusercontent.com/41438361/174022963-12c2d1c5-1d09-4306-83be-053850290c76.png">

그리고 동물 먹이를 만들기 위해서는 적절한 종류의 작물을 기르고, 수확해서 먹이를 생성해야 한다.

두 종류의 concrete 타입을 만들었다.

* `Cow`는 `hay`를 먹는다. `hay`를 기르면 `alfalfa` 작물을 얻을 수 있다. `alfalfa`를 수확하면 `hay`를 얻는다.
* `Chicken`은 `scratch`를 먹는다. `scratch`를 기르면 `millet` 작물을 얻는다. `millet`을 수확하면 `scratch`를 얻는다.

<img width="896" alt="image" src="https://user-images.githubusercontent.com/41438361/174023521-e3f436df-6a14-4867-af66-8d240dc60cb1.png">

위에서 봤던 두 종류의 concrete 메서드를 추상화해서 `feddAnimal()` 메서드를 한 번만 구현해서 소와 닭, 그리고 다른 동물들에게 모두 먹이를 주고 싶다고 해보자. `feedAnimal()`이 `Animal` 프로토콜의 `eat()` 메서드를 호출할 것이다. 그리고 이 `eat()` 메서드는 인자로 먹이를 받고 있기 때문에 consuming position에서 연관 타입을 가진다고 말할 수 있다. `feedAnimal()` 메서드가 `some Animal`을 파라미터 타입으로 갖게 정의해서 existential의 박스를 열고 내용물을 꺼낼 것이다.

<img width="690" alt="image" src="https://user-images.githubusercontent.com/41438361/174024314-579f0715-4ac9-4b95-af95-861d758fdef2.png">

`AnimalFeed`와 `Crop` 프로토콜을 정의하고 내부에 연관 타입을 추가했다. 

* `AnimalFeed` : `Crop`을 준수하는 `CropType` 연관 타입을 가진다.
* `Crop` : `AnimalFeed`를 준수하는 `FeedType` 연관 타입을 가진다.

<img width="1644" alt="image" src="https://user-images.githubusercontent.com/41438361/174024606-9b0eca63-dbc5-4a35-9b83-ed8ed42342a7.png"><img width="1736" alt="image" src="https://user-images.githubusercontent.com/41438361/174024698-872d4b4b-742a-4986-95ea-9779591f3ab8.png">

각 프로토콜의 타입 파라미터의 다이어그램을 위와 같이 표현할 수 있다. 

1. 모든 프로토콜은 `Self` 타입(프로토콜을 준수하는 concrete 타입)을 가진다.
2. 프로토콜은 `Crop`을 준수하는 `CropType` 연관 타입을 가지고 있다.
3. 연관 타입 `CropType`은 중첩된 연관 타입 `FeedType`을 가지고 있다.
4. 이 중첩된 `FeedType`은 또 중첩된 `CropType`을 가지고 있다.
5. ...

이를 통해 `AnimalFeed`와 `Crop`을 준수하는 연관 타입 간에 무한 중첩이 생기는 것이다. 이는 `Crop` 프로토콜에서부터 시작해도 마찬가지다.

<img width="1461" alt="image" src="https://user-images.githubusercontent.com/41438361/174025757-8fd82535-5ea9-4c41-bd97-c89f17fab3b7.png">

`feedAnimal` 메서드 내에서 `Animal` 프로토콜을 준수하는 타입의 값을 가져올 수 있고, `Animal` 은 연관 타입인 `FeedType`을 가지고 있었다.

* `type(of: animal)` : `some Animal: Animal`
* `type(of: animal).FeedType` : `(some Animal).FeedType: AnimalFeed`
* `type(of: animal).FeedType.grow()` : `(some Animal).FeedType.CropType: Crop`
* `crop.harvest()` : `(some Animal).FeedType.CropType.FeedType: AnimalFeed`

그래서 `feed`는 결국 `(some Animal).FeedType.CropType.FeedType` 타입이 되는 것이다.

<img width="1241" alt="image" src="https://user-images.githubusercontent.com/41438361/174026388-550643da-3f50-449f-abba-f10845ac3432.png">

이는 잘못된 타입이다. `eat()` 메서드는 `(some Animal).FeedType`을 요구하는데, 이렇게 작성하면 동물에게 잘못된 먹이를 줄 수도 있다. 
그리고 프로토콜 정의가 너무 일반적이기 때문에 concrete 타입 간의 요구되는 관계를 적절하게 모델링하지도 못했다.

<img width="1778" alt="image" src="https://user-images.githubusercontent.com/41438361/174026893-fef7a3dc-5b9a-46a4-91bb-d92e3d8ab663.png">

hay를 기르면 alfalfa를 얻고, 이를 수확하면 hay를 얻고, 이게 계속 반복된다. 만약 `Alfalfa`가 `Scratch`를 수확한다고 잘못 수정해버렸다고 해보자. 이래도 concrete 타입은 여전히 `AnimalFeed`와 `Crop` 프로토콜의 요구사항을 충족시킨다. 

<img width="1676" alt="image" src="https://user-images.githubusercontent.com/41438361/174028825-7a9fc762-3584-4a0c-aa78-44b5365dcd01.png">

문제는 너무 많은 별개의 연관 타입이 있다는 것이다.(`Self`, `Self.CropType.FeedType`) 이 두 연관 타입들이 실제로 같은 concrete type임을 명시해야 한다.

<img width="394" alt="image" src="https://user-images.githubusercontent.com/41438361/174029000-4d8e211d-8e61-45f0-aad9-a61e51281286.png">

이 연관타입간의 관계를 `where` 문에서 same-type requirement를 설정할 수 있다. 

<img width="687" alt="image" src="https://user-images.githubusercontent.com/41438361/174029268-2a2f061b-8c29-47ba-8851-a80f66155617.png">

위 코드에서는 same-type 요구사항을 추가해서 `AnimalFeed` 프로토콜을 준수하는 concrete 타입에 제약을 설정했다. 

`Self`는 `Self.CropType.FeedType`과 같다고 명시한 것이다.

<img width="1634" alt="image" src="https://user-images.githubusercontent.com/41438361/174029617-cd2b29a4-86d6-436c-95df-75706a29ee65.png"><img width="846" alt="image" src="https://user-images.githubusercontent.com/41438361/174029695-bfbbd36f-99ec-412a-af39-cca70f7c757f.png">

따라서 다이어그램으로 표현했을 때 위와 같이 변한다.

`Self`는 `CropType`을 가지고 있는데, 우리는 same-type 요구사항을 추가해서 `CropType.FeedType`이 `Self`와 같다고 명시했다. 따라서 두 번째 사각형에서 다시 첫 번째 사각형으로 돌아올 수 있는 것이다.

<img width="1644" alt="image" src="https://user-images.githubusercontent.com/41438361/174030007-cb61a4b6-2f6d-42a6-a743-855bc1d63732.png"><img width="929" alt="image" src="https://user-images.githubusercontent.com/41438361/174030104-0ac03101-bb84-45b0-9055-1017b28b2489.png">

`Crop`에서 시작했을 때도 same-type 요구사항을 추가해서 다이어그램을 위와 같이 바꿀 수 있다.

<img width="1653" alt="image" src="https://user-images.githubusercontent.com/41438361/174030280-e0b1217f-804a-4ad0-8f61-5b080fc275cb.png">

이제 동물들에게 올바른 타입의 먹이를 제공할 수 있다.

<img width="1298" alt="image" src="https://user-images.githubusercontent.com/41438361/174030507-151e13df-f648-4cf7-b96a-dde8917af0bb.png">

데이터 모델을 이해하면 이런 다른 중첩 연관 타입을 가진 것들 사이의 동일성을 정의하는데 same-type 요구사항을 사용할 수 있다. 

<img width="1747" alt="image" src="https://user-images.githubusercontent.com/41438361/174030887-3889b800-2e6a-4700-be8f-9c5e87675179.png">

*영상을 보면서 이해하기 어려운 부분들이 많았어서 보는 도중에는 이분이 미웠는데,,,ㅋㅋㅋㅋ 영상 끝날 시점에 웃으면서 마무리하시니까 미웠던게 싹 없어짐*
