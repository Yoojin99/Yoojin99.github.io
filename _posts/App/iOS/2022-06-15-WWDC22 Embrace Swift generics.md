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

# Embrace Swift generics

<img width="1151" alt="image" src="https://user-images.githubusercontent.com/41438361/189590673-efee7039-9a72-4425-a47e-d540eff7efbf.png">

Generic은 Swift에서 **추상화된 코드를 작성하기 위한 기본적인 도구**다. Swift 5.7에서는 제네릭 코드를 더 쉽게 이해하고 작성할 수 있게 해주는 새로운 기능이 추가되었다. 

---

Generics 는 **Swift에서 추상화된 코드를 작성하기 위한 기본적인 도구로, 코드의 복잡성을 관리하는데 매우 중요한 역할을 한다.**

<img width="738" alt="image" src="https://user-images.githubusercontent.com/41438361/189822329-395b02b0-3451-4b14-ad8e-7676dd165645.png">

**추상화는 개념을 구체적인 디테일에서 분리한다.** 

## 추상화?

추상화는 여러 형태로 코드에 적용해서 사용하고 있다.

1. Function / Local variable

만약 반복적으로 사용해야 하는 기능이나 값이 있다면 함수나 지역 변수를 사용해서 추상화를 할 수 있다.

```swift
let circleOneCircumference: Double = 2 * .pi * Double(circleOneRadius)
let circleTwoCircumference: Double = 2 * .pi * Double(circleTwoRadius)
```

가령 위와 같이 원의 둘레를 계산하는 코드가 있다고 해보자. `2 * .pi` 부분이 중복되므로 이 기능을 함수로 만들 수 있다.

```swift
func circumference(radius: Double) -> Double {
    return 2 * .pi * radius
}

let circleOneCircumference = circumference(radius: circleOneRadius)
let circleTwoCircumference: circumference(radius: circleTwoRadius)
```

이렇게 기능을 함수로 추출하면 세부 정보(`2 * .pi * r`)가 추상화되고 추상화를 사용한 코드는 세부 정보를 반복하지 않고 현재 일어나고 있는 것에 대한 아이디어를 표현할 수 있다.

## Type 추상화

Swift에서는 **구체적인 타입(Concrete type) 또한 추상화 할 수 있다.**

<img width="650" alt="image" src="https://user-images.githubusercontent.com/41438361/189822529-91b868bc-95c9-4949-a469-20f35266e094.png">

**같은 개념을 공유하지만 디테일이 다른 타입들의 집합이 있을 경우, 이 모든 구체적인 타입들을 다룰 수 있는 추상화된 코드를 작성할 수 있다. **

지금부터 구체적인 타입들을 추상화하는 workflow를 볼 것이다.

1. Concrete type 모델링하기
2. Concrete type들의 집합에서 공통되는 연산 추출/정의하기
3. 이 연산/능력들을 표현할 수 있는 인터페이스 작성하기
4. 인터페이스로 제네릭 코드 작성하기

순으로 진행된다.

농장 시뮬레이션을 위한 코드를 구현하는 것으로 예시를 들 것이다.

<img width="448" alt="image" src="https://user-images.githubusercontent.com/41438361/189822619-543b10a5-151f-4d71-90fd-3092c2fbe1f3.png">

## 1. Concrete type 만들기

```swift
// MARK: - Animal
struct Cow {
    func eat(_ food: Hay) {}
}

// MARK: - Food
struct Hay {
    static func grow() -> Alfalfa {
        return Alfalfa()
    }
}

// MARK: - Crop
struct Alfalfa {
    func harvest() -> Hay {
        return Hay()
    }
}

// MARK: - Farm
struct Farm {
    func feed(_ animal: Cow) {
        let alfalfa = Hay.grow()
        let hay = alfalfa.harvest()
        animal.eat(hay)
    }
}
```

* 동물 :  Cow
	* 먹이를 먹는다.
* 먹이 : Hay
	* 작물을 생성할 수 있다.
* 작물 : Alfalfa
	* 먹이를 재배할 수 있다.
* 농장
	* 동물에게 먹이를 먹인다.

이제 농장에서 소는 먹일 수 있다. **하지만 다른 종류의 동물들도 먹이를 주고 싶다면 어떻게 해야 할까?**

<img width="462" alt="image" src="https://user-images.githubusercontent.com/41438361/189822769-de5192e6-de6b-49f2-a6a0-9db3c5e63f36.png">

### Many Concrete types

가령 소 말고도 말과 닭을 추가로 농장에서 기르고 싶다면 아래와 같이 농장의 `feed(_:)` 메서드를 overload해서 소, 말, 닭 타입의 파라미터를 인자로 받을 수 있게 할 수 있다.

```swift
// overloading
struct Farm {
    // Cow
    func feed(_ animal: Cow) {
        let alfalfa = Hay.grow()
        let hay = alfalfa.harvest()
        animal.eat(hay)
    }
    
    // Horse
    func feed(_ animal: Horse) {
        let root = Carrot.grow()
        let carrot = root.harvest()
        animal.eat(carrot)
    }
    
    // Chicken
    func feed(_ animal: Chicken) {
        let wheat = Grain.grow()
        let grain = wheat.harvest()
        animal.eat(grain)
    }
}
```

하지만 **overload된 메서드들은 모두 굉장히 비슷하게 구현되어 있다.** 따라서 더 많은 종류의 동물들을 기르게 될 경우 boilerplate가 생기게 되고, 코드를 계속해서 반복적으로 작성해야 한다.

**굉장히 비슷하게 구현된 overload들을 작성한다면, 일반화해야 한다는 신호라고 볼 수 있다.**

이렇게 **반복적으로 비슷하게 구현된 overload들을 작성해야 하는 근본적인 이유는 다른 종류의 동물들이 기능상(먹는다는 동작이) 비슷하기 때문이다.**

## 2. Common capabilities 정의하기

동물들은 모두 어떤 종류의 음식을 먹는다는 동작을 한다. 

```swift
struct Cow {
    func eat(_ food: Hay) {
        // hay를 먹는다.
    }
}

struct Horse {
    func eat(_ food: Carrot) {
        // carrot을 우적 우적 씹어 먹는다.
    }
}

struct Chicken {
    func eat(_ food: Grain) {
        // grain을 쪼아 먹는다.
    }
}
```

동물들은 제각각 다른 방법으로 먹을 수 있기 때문에, 위와 같이 `eat(_:)` 메서드의 내부 동작은 동물 마다 다를 것이다.

하고 싶은 것은 **추상화 코드를 통해 `eat(_:)` 메서드를 호출하면서 이를 실행하는 동물 타입에 따라 다르게 동작하게 하는 것이다.**

### Polymorphism

다른 구체적인 타입들에서 다르게 동작할 수 있는 추상화 코드의 능력을 **polymorphism**이라고 한다. 다형성은 한 코드가 코드가 어떻게 사용되는지에 따라 다양한 동작을 할 수 있게 한다.

### Different forms of polymorphism

다형성은 다양한 형태로 나타난다.

<img width="466" alt="image" src="https://user-images.githubusercontent.com/41438361/189822978-f5a5e8e3-d2d5-4b63-a2fb-bd8c3f821d4b.png">

1. Ad-hoc polymorphism : 함수를 overloading하는 것을 의미한다. 함수를 overload하면 함수의 인자 타입에 따라 같은 함수 호출이 다른 의미를 가질 수 있다. "Ad-hoc" polymorphism 이라고 불리는 이유는 일반적으로 사용되는 해결 방법이 아니기 때문이다. 위에서도 봤듯이, overloading은 반복적으로 코드를 작성하게 한다.
2. Subtype polymorphism : Supertype에서 실행되는 코드가 런타임에서 실제로 사용되는 특정 subtype에 따라 다르게 동작한다. 
3. Parametric polymorphism : 제네릭을 통해 나타낼 수 있다. 제네릭 코드는 다양한 타입들을 다룰 수 있는 하나의 코드를 작성할 수 있게 해주고, 구체적인 타입들은 그 자체로 인자로 사용된다.

위에서 이미 overloading을 사용한 예시를 봤으니, subtype polymorphism 부터 사용해보자.

### Subtype polymorphism

Subtype 관계를 표현할 수 있는 방법은 클래스 계층을 사용하는 것이다.

```swift
class Animal {
    func eat(_ food: ???) {
        fatalError("Subclass는 eat 메서드를 꼭 구현해야 한다.")
    }
}

class Cow: Animal {
    override func eat(_ food: Hay) {
        // hay를 먹는다.
    }
}

class Horse: Animal {
    override func eat(_ food: Carrot) {
        // carrot을 우적 우적 씹어 먹는다.
    }
}

class Chicken: Animal {
    override func eat(_ food: Grain) {
        // grain을 쪼아 먹는다.
    }
}
```

`Animal` superclass를 하나 만들고, 모든 animal을 구조체에서 클래스로 바꾼다음 `Animal` superclass를 상속받고 `eat(_:)` 메서드를 오버라이딩하게 했다.

`Animal`은 모든 구체적인 동물 타입들을 표현할 수 있는 추상화된 base-class가 된다. `Animal` 클래스의 `eat(_:)`를 호출하면 subtype polymorphism을 통해 subclass에 구현된 `eat(_:)` 메서드를 호출할 것이다.

하지만 위 코드에는 여전히 문제가 많다.

<img width="1079" alt="image" src="https://user-images.githubusercontent.com/41438361/189603033-4332c04e-9642-47f3-8df8-4e6e0341d799.png">

1. `Animal`의 `eat(_:)` 메서드의 파라미터 타입을 정하지 못했기 때문에 코드를 작성하면 위와 같이 컴파일 에러가 난다.
2. 모든 동물 타입을 구조체에서 **클래스로 바꿔 사용하게 되면 다른 종류의 동물 인스턴스간에 공유하고 싶은 상태가 없음에도 불구하고 reference semantics(객체를 참조를 통해 조작하는 방법)를 사용하게 된다.**
3. Subclass들이 base class의 메서드를 오버라이딩하게끔 강제했는데, 개발자 실수로 오버라이딩을 하지 않은 경우 런타임에서 에러가 날때까지 오버라이딩하지 않았다는 것을 깜빡할 수 있다.
4. **각 동물 subtype이 다른 타입의 음식을 먹는데, 이런 의존성을 클래스 계층에서 표현하기 어렵다.**


#### Any

위 코드가 컴파일 에러가 나지 않게 하는 한 가지 방법은 `Animal` 클래스의 `eat(_:)` 메서드 인자의 타입으로 `Any`같은 특정되지 않은 타입을 사용하는 것이다. 

```swift
class Animal {
    func eat(_ food: Any) {
        fatalError("Subclass는 eat 메서드를 꼭 구현해야 한다.")
    }
}

class Cow: Animal {
    override func eat(_ food: Any) {
        guard let food = food as? Hay else { fatalError("Cow는 Hay만 먹을 수 있다.") }
    }
}

class Horse: Animal {
    override func eat(_ food: Any) {
        guard let food = food as? Carrot else { fatalError("Cow는 Hay만 먹을 수 있다.") }
    }
}

class Chicken: Animal {
    override func eat(_ food: Any) {
        guard let food = food as? Grain else { fatalError("Cow는 Hay만 먹을 수 있다.") }
    }
}
```

하지만 이런 전략을 사용하면 subclass 의 구현부에서 런타임에 전달된 타입이 맞는 타입인지를 구현해줘야 한다.

따라서 추상화된 base class에서 인자의 타입을 `Any`로 설정하고 subclass에서 이 메서드를 오버라이딩하게 되면 

1. **오버라이딩된 메서드들에서 추가적인 boilerplate가 발생하며,**
2. **잘못된 음식 타입을 실수로 전달할 경우 런타임에서만 확인할 수 있는 버그가 발생한다.**

#### Type parameter

`Any` 대신에 `Animal` superclass에서 type parameter를 사용해서 type-safe한 방법으로 동물의 먹이 타입을 표현할 수 있다. 

```swift
class Animal<Food> {
    func eat(_ food: Food) {
        fatalError("Subclass는 eat 메서드를 꼭 구현해야 한다.")
    }
}

class Cow: Animal<Hay> {
    override func eat(_ food: Hay) {
        // hay를 먹는다.
    }
}

class Horse: Animal<Carrot> {
    override func eat(_ food: Carrot) {
        // carrot을 우적 우적 씹어 먹는다.
    }
}

class Chicken: Animal<Grain> {
    override func eat(_ food: Grain) {
        // grain을 쪼아 먹는다.
    }
}
```

타입 파라미터는 각 subclass를 위한 특정 먹이 타입의 placeholder가 된다. 즉 이 placeholder에 특정 먹이 타입을 채우면 된다. 

이 방법 또한 부자연스러운데, 

1. 동물들이 "`eat(_:)`"을 위해 먹이가 필요한 것은 맞지만 먹이를 먹는 것이 동물의 주요 목적인 것이 아니고,
2. 각 동물 클래스 내의 많은 코드들이 먹이를 사용하지 않을 확률이 더 높기 때문이다.

즉 먹이가 중요하지 않음에도 불구하고 동물 클래스를 참조하는 모든 곳에서 먹이 타입을 명시해야 한다는 문제가 있다. 위 코드를 보면 동물 subclass 들에서 구현 부분의 대괄호 안에 명시적으로 먹이 타입을 쓰고 있다.

이런 boilerplate는 동물을 추가할 때마다 작성해야 하므로 번거롭다. 

따라서 지금까지 본 ad-hoc polymorphism을 사용한 overloading, subtype polymorphism을 사용한 방법 모두 좋은 방법이 아니다.

근본적인 문제는 **클래스는 데이터 타입인데, 구체적인 타입들에 대한 개념을 추상화하기 위해 superclass를 복잡하게 만들고 있는 것이다. 대신, 각 타입이 구체적으로 어떻게 동작하는지에 대한 디테일 없이 각 타입의 능력을 표현할 수 있는 구조를 사용하는 것이 좋다.**

#### Common capabilties of an animal

<img width="528" alt="image" src="https://user-images.githubusercontent.com/41438361/189823381-5b3cce22-a7b5-4e1f-a512-94b764711f62.png">

동물들은 두 가지 공통적인 능력(capabilty)이 있다.

* 특정 타입의 먹이
* 특정 타입의 먹이를 먹을 수 있는 연산

## 3. Interface 만들기

앞에서 정의한 능력들을 표현할 수 있는 인터페이스를 만들 수 있다. Swift에서는 이를 **protocol**을 사용해서 만들 수 있다.

<img width="778" alt="image" src="https://user-images.githubusercontent.com/41438361/189823564-fe76f3be-fa55-4be9-8792-d1bc0e5f1340.png">

**Protocol은 conforming type(프로토콜을 준수하는 타입)의 기능을 명시하는 추상화 도구다.** 프로토콜을 사용해서 구체적인 구현 부분에서 타입이 무엇을 하는지에 대한 개념을 분리할 수 있다. 타입이 무엇을 하는지에 대한 개념은 인터페이스를 통해 표현된다. 

위에서 정의한 동물의 능력을 protocol interface로 변환시키면 아래와 같다.

### Interface of an animal

```swift
protocol Animal {
    associatedtype Food
    func eat(_ food: Food)
}
```

* 특정 타입의 먹이 : 연관 타입 `Food`. 타입 파라미터와 마찬가지로, 연관 타입은 구체적인 타입들에서 사용할 수 있는 placeholder가 된다. 연관 타입이 특별한 이유는 프로토콜을 준수하는 구체적인 타입에 ㅢ존한다는 것이다. 이 관계는 보장되기 때문에 프로토콜을 준수하는 한 구체적인 타입의 인스턴스들은 모두 같은 종류의 음식을 갖게 된다.
* 특정 타입의 먹이를 먹을 수 있는 연산 : `eat(_:)` 메서드. 프로토콜은 메서드를 구현하지 않고, 구체적인 동물 타입에서 이를 구현하도록 되어 있다.

구현한 인터페이스를 각 구체적인 동물 타입이 이를 준수하게 할 수 있다.

```swift
protocol Animal {
    associatedtype Food
    func eat(_ food: Food)
}


struct Cow: Animal {
    func eat(_ food: Hay) {
        // hay를 먹는다.
    }
}

struct Horse: Animal {
    func eat(_ food: Carrot) {
        // carrot을 우적 우적 씹어 먹는다.
    }
}

struct Chicken: Animal {
    func eat(_ food: Grain) {
        // grain을 쪼아 먹는다.
    }
}
```

프로토콜을 사용하면 장점이 있다.

1. **프로토콜은 클래스에 한정되지 않기 때문에** 구조체, 열거형, actor에도 사용할 수 있다. 따라서 각 동물이 구조체로 존재할 수 있다. 
2. `: Animal`을 통해 conformance annotation을 작성하면 **컴파일러가 자동으로 각 구체적인 타입들이 프로토콜 요구사항을 구현하는지를 확인한다.** 각 구체적인 동물 타입들은 `eat(_:)` 메서드를 구현해야 하고, 컴파일러는 인자에서 먹이 타입을 보고 `Food`가 무슨 타입인지를 추론할 수 있다. 위 코드에서는 작성하지 않았지만, `Food` 타입이 무엇인지 `typealias`를 통해 명시적으로 작성할 수도 있다.

## 4. Write generic code

```swift
protocol Animal {
    associatedtype Food
    func eat(_ food: Food)
}

struct Farm {
    func feed(_ animal: ???) {
    }
}
```

`Animal` 프로토콜을 사용해서 농장의 `feed(_:)` 메서드를 구현하는데 사용할 수 있다. 우리가 제네릭 코드를 사용하게 된 이유, 문제는 다양한 동물 타입이 새로 만들어질 때마다 boilerplate 코드가 많이 생성된다는 것이었다. 따라서 우리의 목표는 `Animal` 프로토콜을 사용해서 모든 타입의 구체적인 동물 타입에서 동작할 수 있는 하나의 구현부를 작성하는 것이다.

### Parametric polymorphism

이제 메서드가 호출 될 때 구체적인 타입으로 치환될 수 있는 타입 파라미터를 사용하는 것에서 시작해서 parametric polymorphism을 사용할 것이다.

```swift
struct Farm {
    // 방법 1
    func feed<A: Animal>(_ animal: A) {
    }
    
    // 방법 2
    func feed<A>(_ animal: A) where A: Animal {
    }
}
```

타입 파라미터는 함수 이름 뒤에 괄호 안에 작성한다. 

1. 일반적인 변수와 함수 파라미터와 마찬가지로, 원하는 이름을 붙일 수 있다. 
2. 다른 일반적인 타입들과 마찬가지로 함수에서 타입 파라미터의 이름을 사용해서 참조할 수 있다.

여기에서는 타입 파라미터는 `A`라 명명했고, 함수의 `animal` 파리머터의 타입으로 사용했다. 구체적인 동물 타입이 `Animal` 프로토콜을 준수하기를 원하기 때문에 타입 파라미터 뒤에 `: Animal`라는 protocol conformance를 함께 작성했다. 

Protocol conformance는 위 코드에서 볼 수 있듯이 대괄호 안에 작성될 수도 있고, 끝 부분의 `where` 문에서 작성될 수도 있다. 

`feed(_:)` 메서드만 보면 아래와 같다.

```swift
func feed<A>(_ animal: A) where A: Animal
```

타입 파라미터는 parameter list에서 한 번 등장하고, "where" 문에서 파라미터의 conformance 요구사항을 정의하는 부분에서 한 번 등장한다. 여기에서 타입 파라미터를 명명하고 "where" 문을 사용하는 것은 함수를 괜히 더 복잡하게 보이게 만든다.

이런 제네릭 패턴은 굉장히 일반적으로 많이 사용되기 때문에, 이를 나타낼 더 간단한 방법이 있다.

```swift
func feed(_ animal: some Animal)
```

타입 파라미터를 명시적으로 쓰는 대신에, `some Animal`로 protocol conformance를 작성할 수도 있다. 이 표현 방법은 이전 표현과 완전히 동일하지만, 불필요한 타입 파라미터 목록과 "where" 문을 없애서 간결하게 나타낼 수 있게 했다.

**`some Animal`로 작성하는 것이 좋은 이유는 문법적으로 불필요한 부분을 줄이면서 `animal` 파라미터에 대한 정보를 `animal` 파라미터가 사용된 곳에 작성해서 포함할 수 있다는 것이다.**

### some

<img width="273" alt="image" src="https://user-images.githubusercontent.com/41438361/189616777-7ab792c3-035d-42a3-bb8b-392db2b2c639.png">

`some`은 특정한 구체적인 타입이 있음을 명시한다. `some` 뒤에는 항상 conformance 요구사항이 따라온다. 여깅서는 특정 타입은 꼭 `Animal` 프로토콜을 준수해야 함을 나타내고, 파라미터를 통해 `Animal` 프로토콜에서 명시한 요구사항들을 사용할 수 있게 한다.

**`some` 키워드는 파리미터와 결과 타입에서도 사용될 수 있다.  ** 만약 SwiftUI를 사용해 본 사람이라면, 리턴 타입 부분에 `some View`과 같이 써서 `some`을 사용해본 적이 있을 것이다. SwiftUI 뷰에서, body 프로퍼티는 특정 타입의 뷰를 리턴하지만, 코드는 특정 타입이 무엇인지 알 필요 없이 body 프로퍼티를 사용한다.


#### Specific abstract type

잠시 특정 추상화 타입에 대해 이해하고 넘어가도록 하자. "Specific"과 "abstract"가 같이 쓰여 헷갈릴 것이다. 

* Abstract type == Opaque type : 특정 구체적인 타입의 placeholder를 나타낸다.
* Underlying type : opaque type이 사용된 placeholder를 대체하는 특정 구체적인 타입을 의미한다.

Opaque type을 타입으로 갖는 값들은 값이 사용된 곳에서 구체적인 underlying type이 고정(확정)된다.

구체적인 underlying 타입이 고정되기 때문에 이 값을 사용하는 제네릭 코드는 값에 접근할 때마다 같은 underlying 타입을 갖도록 보장받을 수 있다.

#### Opaque type declarations

Opaque type을 정의하는 데는 두 가지 방법이 있다.

<img width="769" alt="image" src="https://user-images.githubusercontent.com/41438361/189824035-1b199bc8-1d49-45d2-b405-145fa0c0290c.png">

1. `some` 키워드 사용
2. 대괄호 안에 타입 파라미터 사용

Opaque 타입은 input, output 모두에 사용될 수 있기 때문에 파라미터나 리턴 타입 부분에서 사용될 수 있다.

<img width="1682" alt="image" src="https://user-images.githubusercontent.com/41438361/189621062-221da9a6-ac87-4aaf-b7b0-5e4228c3aa9a.png">

함수 화살표는 이런 위치를 구분해준다. input 위치는 쉽게 생각해서 함수의 인자, 입력하는 부분이고 output 위치는 리턴 타입 부분으로 생각할 수 있다.

**Opaque type의 위치는 프로그램의 어떤 부분이 추상화된 타입을 볼지, 그리고 어떤 타입이 구체적인 타입을 결정하는지를 결정한다.** 

<img width="658" alt="image" src="https://user-images.githubusercontent.com/41438361/189824205-f269304e-4bcd-40b5-8aa3-8b713496cf16.png">

타입 파라미터는 항상 input 쪽에서 정의되기 때문에, 호출하는 곳에서 underlying 타입을 정하고, 구현하는 곳에서 abstract 타입을 사용한다.

<img width="617" alt="image" src="https://user-images.githubusercontent.com/41438361/189824337-9c72652a-7049-48d8-85a0-5c5533f223df.png">

일반적으로, **opaque 파라미터나 리턴 타입의 값을 제공하는 부분에서는 underlying 타입을 정하고, 그 값을 사용하는 부분에서는 abstract type을 본다. **

이 말이 무슨 말인지 보자.

```swift
protocol Liquid {
    func flow()
}

struct Water: Liquid {
    func flow() {}
}

struct Milk: Liquid {
    func flow() {}
}

struct Human {
    func drink(_ liquid: Liquid) {
        // 사용하는 곳
        print("Human drinks \(liquid)")
    }
}

let human = Human()
let water = Water()
// 값을 제공하는 곳. Water 타입의 인스턴스를 값으로 제공했다.
human.drink(water)
```

위 코드에서는 `Liquid`라는 프로토콜이 있고, 이를 준수하는 `Water`와 `Milk` 두 구체적인 구조체가 있다. 그리고 `Human` 구조체는 `Liquid` 프로토콜을 준수하는 특정 타입의 인스턴스를 `drink(_:)` 할 수 있다. 

`drink(_ liquid: Liquid)` 부분을 보면 `liquid` 파라미터는 `Liquid`라는 abstract 타입, opaque 타입이다. 이 **파라미터를 사용하는 함수의 구현부에서는 `liquid` 파라미터를 사용해서 무언가를 할 때 `Liquid` 프로토콜을 보고, 이 프로토콜에서 명시하는 요구사항들을 사용할 수 있다.**

이제 `human.drink(water)` 부분을 보면 `Water` 타입이라는 **구체적인 underlying 타입의 인스턴스를 생성해서 `drink(_:)` 메서드의 인자로 제공했다. **

따라서 

* 값을 제공하는 부분 (`human.drink(water)`)에서는 underlying type을 정하고
* 값을 사용하는 부분 (`drink(_:)` 메서드의 구현부)에서는 abstract type을 본다.

이제 이것이 어떻게 `some ~`에 적용되는 지 볼 것 이다.

### `some`의 underlying type 추론하기

Underlying 타입은 값에서 추론될 수 있다. `some`은 항상 구체적인 타입이 있다는 것을 나타낸다는 것을 기억하자.

#### Local variables

<img width="293" alt="image" src="https://user-images.githubusercontent.com/41438361/189824459-4e845e93-f4ae-4640-923a-deabdae88e95.png">

지역 변수에서, underlying type은 할당하는 부분에서 **할당된 값에서 추론된다. 이는 opaque type인 지역 변수는 항상 초깃값을 가지고 있어야 함을 의미한다.** 

<img width="419" alt="image" src="https://user-images.githubusercontent.com/41438361/189824544-1bd79e08-0fac-427e-86e1-92b1df3a27ec.png">

만약 초깃값이 없다면 위와 같이 컴파일러는 에러를 띄운다.

<img width="415" alt="image" src="https://user-images.githubusercontent.com/41438361/189824640-f7a3e164-055d-4fc5-9012-a76ae8da6b80.png">

**Underlying 타입은 변수의 범위에서는 항상 고정되어 있기 때문에, underlying 타입을 바꾸려고 하면 마찬가지로 에러가 뜰 것이다.**

#### Parameters

<img width="280" alt="image" src="https://user-images.githubusercontent.com/41438361/189824736-3a883e8a-5659-44b2-944e-87554cb2ca88.png">

**Opaque 타입인 파라미터들의 경우 underlying 타입은 함수를 호출하는 곳에서 사용된 인자 값에서 추론된다.** Swift 5.7에서부터 `some`을 파라미터 위치에서 사용할 수 있다.

<img width="269" alt="image" src="https://user-images.githubusercontent.com/41438361/189824842-a016ac92-f5bf-48fb-a62c-c220c04d7115.png">

**Underlying type은 파라미터의 범위 내에서만 고정되기 때문에, 함수를 호출할 때마다 다른 타입의 인자 값을 제공해도 상관 없다. **

#### Results
<img width="374" alt="image" src="https://user-images.githubusercontent.com/41438361/189824939-7a916c9f-c909-4be9-bbaf-ab173d6c1d86.png">

**Opaque result 타입의 경우, underlying 타입은 구현부에서 리턴되는 값에서 추론된다.**

<img width="521" alt="image" src="https://user-images.githubusercontent.com/41438361/189825011-eb66073a-36c0-4e5d-9394-356fae7dc020.png">

**Opaque result 타입을 갖는 메서드나 연산 프로퍼티는 프로그램 내의 어디에서도 호출될 수 있기 때문에, 이 결과 값의 범위는 전체라고 할 수 있다. 따라서 underlying return 타입이 모든 return 문에서 같아야 함을 의미한다.** 만약 그렇지 않으면 컴파일러는 return 값의 underlying 타입이 상이하다는 에러를 출력할 것이다.

SwiftUI view의 opaque 타입에서는, ViewBuilder DSL이 모든 return type이 같은 underlying return type을 갖도록 control-flow 문을 변환할 수 있다.

<img width="376" alt="image" src="https://user-images.githubusercontent.com/41438361/189825144-c6ab5622-a406-469c-99ca-efcec61acf3d.png">

따라서 위의 경우 ViewBuilder DSL을 사용해서 이슈를 해결할 수 있다. 메서드에 `@ViewBuilder` 어노테이션을 쓰고 return 문을 지우면 결과 값들이 ViewBuilder 타입이 되도록 만들 수 있다.

### Back to farm...

그렇다면 농장의 `feed(_:)` 메서드는 `some`을 이용해서 어떻게 구현해야 할까?

```swift
struct Farm {
    func feed(_ animal: some Animal) {}
}
```

`some`을 파라미터 list에 쓸 수 있는 이유는 opaque type(`Animal`)을 다른 곳에서 참조할 필요가 없기 때문이다. 만약 함수 signature에서 opaque 타입을 여러 번 참조해야 한다면 이때는 타입 파라미터를 쓰는게 유용하다.

### Refer Opaque type multiple times

예를 들어 `Animal` 프로토콜에 또 다른 연관 타입 `Habitat`을 추가했다면, `Farm`에 동물을 인자로 받아 해당 동물의 서식지를 리턴하는 함수 `buildHome`을 아래와 같이 작성할 수 있을 것이다.

```swift
protocol Animal {
    associatedtype Food
    associatedtype Habitat
    func eat(_ food: Food)
}

struct Farm {
    func feed(_ animal: some Animal) {}
    
    func buildHome<A>(for animal: A) -> A.Habitat where A: Animal {}
}
```

이 경우에 result type은 특정 동물 타입에 따라 다르기 때문에 타입 파라미터 `A`를 파라미터 타입으로도 쓰고, result type에도 쓴다. 

제네릭 타입에서도 이렇게 opaque 타입을 여러 곳에서 참조해야 경우가 많이 발생한다.

```swift
struct CandyFactory<Flavor> {
    private var flavors: [Flavor]
    
    init(flavors: [Flavor]) {
        self.flavors = flavors
    }
}

var candyFactor: CandyFactory<Chocolate>
```

위 `CandyFactory` 구조체에서는 타입 파라미터를 사용해서 제네릭 파라미터를 정의했고, 저장 프로퍼티의 타입으로도 사용했으며 멤버와이즈 이니셜라이저에서도 썼다. 또한 다른 문맥에서 제네릭 타입을 참조하려면 명시적으로 대괄호 안에 타입 파라미터를 써야 한다. 

### Farm Simulator again...

```swift
struct Farm {
    func feed(_ animal: some Animal) {
        let crop = type(of: animal).Food.grow()
        let food = crop.harvest()
        animal.eat(food)
    }
}
```

`animal` 파라미터의 타입에 접근해서 `Food` 타입을 통해 `crop`을 생성한다. 그리고 `crop`의 `harvest()` 메서드를 호출해서 먹이를 생산한다. 그리고 최종적으로 이를 동물에게 먹일 수 있다. 이때 이 `feed(_:)` 메서드를 호출하는 곳에서 underlying type이 확정되기 때문에, 컴파일러는 작물 타입과 먹이 타입간의 관계, 그리고 동물 타입을 알 수 있다. 이런 관계가 정해지면 동물에게 잘못도니 먹이를 주는 것을 예방할 수있다.

<img width="468" alt="image" src="https://user-images.githubusercontent.com/41438361/189825335-5689431d-59bd-4c44-8c9c-02df863f5a21.png">

만약 동물에 잘못된 타입의 먹이를 주게 된다면, 컴파일러가 알려주게 된다.

동물의 먹이 타입과 그 작물 간의 관계를 표현하기 위해 다른 프로토콜들을 어떻게 정의했는지 보려면 WWDC22의 다른 세션 "Design protocol interfaces in Swift"를 보면 된다.

이제 모든 동물들에게 먹이를 주는 메서드를 추가해보자.

```swift
struct Farm {
    func feed(_ animal: some Animal) {
        let crop = type(of: animal).Food.grow()
        let food = crop.harvest()
        animal.eat(food)
    }
    
    func feedAll(_ animals: [???]) {}
}
```

`feedAll(_:)`가 인자로 받는 배열의 원소들은 `Animal` 프로토콜을 준수해야는데, 이 배열들은 다른 종류의 동물들을 담을 수 있어야 한다. 여기에서 `some`을 쓸 수 있을까?

<img width="573" alt="image" src="https://user-images.githubusercontent.com/41438361/189825505-aee034ea-73c1-4141-bbde-5aba889a5e83.png">

`some`은 특정한 구체적인 underlying 타입을 뜻하기 때문에 다양한 타입을 담지 못한다. `some`을 사용하면 메서드를 사용하는 시점에서 underlying 타입이 고정되기 때문에 배열에 있는 모든 원소는 같은 타입이어야 한다.

여기에서 '어떤 종류의 동물'을 나타낼 수 있는 supertype이 필요하다.

<img width="340" alt="image" src="https://user-images.githubusercontent.com/41438361/189825572-8186f36b-2f46-4008-b648-73171cad108d.png">

여기에서 **어떤 불특정한 동물 타입을 `any Animal`로 표현할 수 있다. `any` 키워드는 이 타입이 어떤 불특정한 동물 타입을 저장할 수 있다는 것을 의미하고, underlying 타입은 런타임에서 바뀔 수 있다.**

`some` 키워드와 마찬가지로, `any` 키워드는 뒤에 conformance 요구사항이 따라온다.

**`any Animal`은 어떤 종류의 동물 타입이라도 동적으로 저장할 수 있게 해주는 하나의 static type으로, 값 타입으로도 subtype polymorphism을 사용할 수 있게 해준다. 여러 종류의 동물 타입을 유연하게 저장할 수 있기 위해 `any Animal`타입은 메모리 상에서 특별한 표현 방식을 갖게 된다.**

<img width="501" alt="image" src="https://user-images.githubusercontent.com/41438361/189825707-d2f7ce82-ca8b-4cc6-bc23-dd9f40614808.png">

이 표현을 박스로 생각해볼 수 있다. 어떤 경우에 값은 박스보다 작아서 박스 안에 들어갈 수 있다. 하지만 박스에 들어가기에는 너무 큰 경우에는 값은 다른 곳에 할당되어야 하고, 박스는 그 값을 가리키는 포인터를 담는 방식으로 값을 저장해야 할 것이다. 

정적 타입인 `any Animal`이 동적으로 저장할 수 있는 구체적인 동물 타입을 공식적으로 existential type이라고 한다.

### Existential types provide type erasure

이렇게 **다양한 종류의 구체적인 타입에 같은 표현 방식을 쓸 수 있는(다양한 크기를 갖고 있을 다양한 종류의 구체적인 타입들을 같은 박스들을 사용해서 유연하게 저장하는) 전략을 type erasure라고 한다.** 구체적인 타입은 컴파일 타임에 "지워졌다"라고 하고, 구체적인 타입은 런타임에서만 알 수 있다.

위 그림에서 existential 타입 `any Animal`의 두 인스턴스는 같은 정적 타입(`any Animal`)을 갖지만, 다른 동적 타입을 갖는다.

Type erasure는 다른 동물 값들 간의 type-level 차이를 없애서 다른 동적 타입 값들을 같은 정적 타입으로 바꿀 수 있게 한다.

<img width="551" alt="image" src="https://user-images.githubusercontent.com/41438361/189825823-4c8dd79f-da1b-4587-a331-05192759fe9f.png">

Type erasure를 사용해서 다양한 값 타입의 배열을 만들 수 있고, 이를 `feedAll(_:)` 에 적용할 수 있다.

Swift 5.7에서는 `any` 키워드를 연관타입을 가진 프로토콜에 사용할 수 있다.

```swift
struct Farm {
    func feed(_ animal: some Animal) {
        let crop = type(of: animal).Food.grow()
        let food = crop.harvest()
        animal.eat(food)
    }
    
    func feedAll(_ animals: [any Animal]) {
        for animal in animals {
            animal.eat(Animal.Food)
        }
    }
}
```

위와 같이 `animal.eat(Animal.Food)`로 작성하면 컴파일러 에러가 발생한다.

<img width="405" alt="image" src="https://user-images.githubusercontent.com/41438361/189826019-462f53ea-929d-4201-913f-5d5cb6efb34b.png">
Type erasure를 통해 `animals` 배열에 저장하는 구체적인 동물 타입간의 type-level 차이를 없앴기 때문에, 연관 타입과 같이 특정 동물 타입에 의존하는 모든 type 관계도 지워졌기 때문이다. 그래서 이 `animal`이 어떤 타입의 먹이를 기대하는지 알 수 없다.

타입 관계에 의존하기 위해(타입 간의 관계를 이용하기 위해), 특정 타입의 동물이 정해진 문맥으로 다시 돌아가야 한다.

`any Animal`을 직접적으로 호출하는 대신, `some Animal`을 인자로 받는 `feed` 메서드를 호출해야 한다.

```swift
struct Farm {
    func feed(_ animal: some Animal) {
        let crop = type(of: animal).Food.grow()
        let food = crop.harvest()
        animal.eat(food)
    }
    
    func feedAll(_ animals: [any Animal]) {
        for animal in animals {
            feed(animal)
        }
    }
}
```

`any Animal`은 타입 간의 관계를 다 지워버렸기 때문에, `some Animal`을 사용해서 특정 구체적인 타입을 받아와서 해당 동물 타입의 먹이 타입을 사용할 수 있어야 한다. 

<img width="765" alt="image" src="https://user-images.githubusercontent.com/41438361/189826182-69cbd22b-56a6-4380-834b-409d910b8070.png">
`any Animal`은 `some Animal`과 다른 타입이지만, 컴파일러는 `any Animal`의 인스턴스(박스에 담긴 인스턴스/or 포인터)를 unboxing해서 `some Animal`로 바꿀 수 있고 이를 바로 `some Animal` 파리미터로 전달할 수 있다.

Swift 5.7부터 인자를 unboxing할 수 있다.  Unboxing은 컴파일러가 박스를 열고 안에 담긴 값을 꺼내는 거라고 생각하면 된다.

`some Animal` 파라미터의 값은 고정된 Underlying 타입을 가지고 있기 때문에, 연관 타입을 포함해서 underlying 타입의 모든 연산에 접근할 수 있다.

이 기능이 굉장이 멋진이유는 필요할 때 유연하게 저장할 수 있게 하면서 동시에 함수의 범위 내에서 underlying type을 고정해서 특정 타입의 연산들을 사용할 수 있다는 것이다. 그리고 이 unboxing은 예상하는 것과 똑같이 동작하는데, `any Animal` 프로토콜 메서드를 호출하면 underlying 타입의 메서드를 호출하는 방식과 비슷한 방식으로 동작한다.

## Summary

<img width="736" alt="image" src="https://user-images.githubusercontent.com/41438361/189826307-2081b74e-0af0-4999-9d06-80147603d1a5.png">

### `some`

* `some`을 사용하면 underlying 타입이 고정된다.
* 제네릭 코드에서 underlying 타입의 타입 관계를 사용할 수 있게 된다.(underlying 타입의 연관 타입, 여기에서는 특정 구체적인 동물 타입의 먹이 타입을 사용할 수 있다) 따라서 사용하는 프로토콜의 API 와 연관 타입을 모두 사용할 수 있다.

### `any`

* `any`는 임의의 구체적인 타입을 저장할 때 사용한다.
* Type erasure를 사용해서 다양한 collection을 표현할 수 있다.

<img width="601" alt="image" src="https://user-images.githubusercontent.com/41438361/189826406-24c529cf-9093-48bd-8ec8-66f5c3182428.png">

기본적으로는 `some`을 기본으로 쓰고, 임의의 값을 저장할 필요가 있을 때만 `some`을 `any`로 바꾼다. 이러면 type erasure를 사용하는 것에 대한 부담을 지지만 유연한 저장을 사용할 수 있다.

* 참조
* https://dept-info.labri.fr/~strandh/Teaching/MTP/Common/Strandh-Tutorial/uniform-reference-semantics.html
