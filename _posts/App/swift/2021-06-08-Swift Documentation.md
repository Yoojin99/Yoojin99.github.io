---  
layout: post  
title: "Swift - Documentation, 문서화하기"  
subtitle: ""  
categories: app
tags: app-swift
comments: true  
header-img: 
---  
  
> `Swift의 문법을 이용해서 코드를 문서화 하는 방법을 알아보자.`  

---

## 문서화 주석(Documentation Comments) & Swift에 맞는 Markdown

만약 이전에 Markdown 형식으로 문서를 작성해보지 않았더라고, 몇 분만에 이를 익힐 수 있다.

### 기본 Markup

문서화 주석은 일반 주석과 비슷해 보이지만, 추가적인 부분이 더 있다. 한 줄의 문서화 주석은 슬래시가 세 개로 구성된다. (`///`)

반면 여러 줄의 문서화 주석은 여는 구분 기호에 추가적인 별표 문자가 붙는다. (`/** ... */`)

표준 마크다운은 문서화 주석의 규칙을 아래와 같이 적용한다.

* 문단은 빈 줄로 구분한다.
* 순서가 없는 리스트는 칸 문자(bullet characters)로 표시된다. (`-`, `+`, `*`, `•`)
* 순서가 있는 리스트는 숫자(1, 2, 3, ...) 뒤에 온점을 붙이거나(`1.`) 오른쪽 괄호를 붙여서(`1)`) 표시한다.
* 헤더는 `#` 기호 뒤에 붙여서 표시하거나 아래 줄에 `=`나 `-`를 붙여서 표시한다.
* 링크와 이미지도 붙일 수 있고, 웹을 기반으로 하는 이미지는 Xcode에서 바로 보여진다.

``` swift
/**
    # Lists

    You can apply *italic*, **bold**, or `code` inline styles.

    ## Unordered Lists

    - Lists are great,
    - but perhaps don't nest;
    - Sub-list formatting...

      - ...isn't the best.

    ## Ordered Lists

    1. Ordered lists, too,
    2. for things that are sorted;
    3. Arabic numerals
    4. are the only kind supported.
*/
```

문단 나누는 것을 더 보면, 아래의 코드는 한 문단으로 구성되지만,

``` swift
/**
    This is your User documentation.
    A very long one.
*/
```

아래의 코드는 두 문단으로 구성된다.

``` swift
/**
    This is your User documentation (This is summary).

    A very long one (This will be shown in the discussion section).
*/
```

### 요약 & 설명

문서화 주석 앞에 나오는 문단(paragraph)는 문서의 요약(Summary) 부분이 된다. 추가 컨텐츠는 `Discussion` 섹션으로 묶이게 된다.

*만약 문서화 주석이 문단이 아닌 다른 것에 의해 시작된다면, 모든 컨텐츠는 Discussion에 속하게 된다. 또한 Discussion 부분에 추가를 하고 싶다면 새로운 문단을 작성하면 된다. 첫 번째 문단 아래에 오는 다른 것들은 다 discussion 부분에 속한다.*

``` swift
/**
    This is your User documentation.
    A very long one.
*/
struct User {
    let firstName: String
    let lastName: String
}
```

위의 코드는 아래와 같이 요약 부분이 있는 문서를 생성한다.
![image](https://user-images.githubusercontent.com/41438361/121201535-90da7500-c8af-11eb-9399-42ccafcb1536.png)

### 파라미터 & 리턴 값

Xcode는 몇 개의 특별한 필드를 인식해서 이를 심볼의 설명과는 분리시켜 놓는다. 파라미터, 리턴 값, 그리고 throws 부분은 글머리 기호 뒤에 콜론(`:`)을 붙일 때 Quick Help 팝업과 inspector에서 구분 되어서 나온다.

* 파라미터: `Parameter <param name>:`으로 시작하며 파라미터의 설명이 이어서 나온다.
* 리턴 값: `Returns:` 로 시작하며 리턴 값에 대한 정보가 이어서 나온다.
* 던져진 에러들: `Throws:`로 시작하며 throw될 수 있는 에러에 대한 설명이 이어서 나온다. Swift가 `Error` 규정에서 벗어나는 에러들은 타입을 체크하지 않기 때문에, 에러에 대해 문서화를 올바르게 하는 것은 중요하다.

``` swift
/**
 Creates a personalized greeting for a recipient.

 - Parameter recipient: The person being greeted.

 - Throws: `MyError.invalidRecipient`
           if `recipient` is "Derek"
           (he knows what he did).

 - Returns: A new string saying hello to `recipient`.
 */
func greeting(to recipient: String) throws -> String {
    guard recipient != "Derek" else {
        throw MyError.invalidRecipient
    }

    return "Greetings, \(recipient)!"
}
```

Hacker News의 "tabs vs.spaces?"의 쓰레드보다 더 많은 인자를 포함하는 메서드를 문서화하고 있는가? `Parameters:` 를 적고 아래에 파라미터를 글머리표로 구분한 리스트로 구성해라.

``` swift
/// Returns the magnitude of a vector in three dimensions
/// from the given components.
///
/// - Parameters:
///     - x: The *x* component of the vector.
///     - y: The *y* component of the vector.
///     - z: The *z* component of the vector.
func magnitude3D(x: Double, y: Double, z: Double) -> Double {
    return sqrt(pow(x, 2) + pow(y, 2) + pow(z, 2))
}
```

참고로 파라피터를 개별로 다 작성할 수도 있다.

``` swift
- Parameter x: ...
- Parameter y: ...
```

Throws 부분을 작성한 예시는 아래와 같다.

``` swift
/// A way that this user greets others.
///
/// Use this method to get the greeting for the user. The person you specify don't affect the way user greet, just the first name, and last name will be used.
/// - Warning: The greeting is always in the caller localization.
/// - Throws:
///     - `MyError.invalidPerson`
///     if `person` is not known by the caller.
///     - `MyError.hardToPronounce`
///     if `person` name is longer than 20 characters.
/// - Parameter person: `User` you want to greet
/// - Returns: A greeting of the current `User`.
func greeting(person: User) throws -> String {
  return "Hello \(person.firstName) \(person.lastName)"
}
```

화면 상에서는 아래와 같이 나온다.
![image](https://user-images.githubusercontent.com/41438361/121201623-9fc12780-c8af-11eb-8540-87e99aff2b74.png)
참고로 Throws 같은 경우는 구분자가 하나만 있을 수 있고, 여러개가 존재할 수 없다.

### 추가 필드

`Parameters`, `Throws`, `Returns`에 추가로, Swift에 맞는 마크다운은 아래와 같이 대강 구성될 수 있는 다른 필드들을 정의한다.

**Algorithm/Safety Information**

* `Precondition` : 전제조건
* `Postcondition` : 이후에 성립되어야 하는 조건
* `Requires` : 요구사항
* `Invariant` : 변형성
* `Complexity` : 복잡도
* `Important` : 중요한 점
* `Warning` : 경고

**Metadata**

* `Author` : 저자
* `Authors` : 저자들
* `Copyright` : 저작권
* `Date` : 날짜
* `SeeAlso` : 추가로 봐야할 점들
* `Since` : 생성된 때
* `Version` : 버전

**General Notes & Exhortation**

* `Attention` : 주의할 점
* `Bug` : 버그
* `Experiment` : 실험
* `Note` : 참고
* `Remark` : 비고
* `ToDo` : 해야 할 것

위의 각 필드들은 두꺼운 글씨의 헤더에 텍스트 블럭으로 Quick Help에 아래와 같이 표시된다.

![image](https://user-images.githubusercontent.com/41438361/121201671-a9e32600-c8af-11eb-8b66-f93c5e321f56.png)

### Code blocks

코드 블럭을 써서 함수의 적절한 사용이나 구현할 때 세부적인 내용을 보여준다. 코드 블럭은 최소 4개의 공백으로 만든다.

``` swift
/**
    The area of the `Shape` instance.

    Computation depends on the shape of the instance.
    For a triangle, `area` is equivalent to:

        let height = triangle.calculateHeight()
        let area = triangle.base * height / 2
*/
var area: CGFloat { get }
```

세개의 백틱(\`\`)이나 물결표(`~`)로 코드를 감싸서 코드 불럭을 만들 수 있다.

``` swift
/**
    The perimeter of the `Shape` instance.

    Computation depends on the shape of the instance, and is
    equivalent to:

    ~~~
    // Circles:
    let perimeter = circle.radius * 2 * Float.pi

    // Other shapes:
    let perimeter = shape.sides.map { $0.length }
                               .reduce(0, +)
    ~~~
*/
var perimeter: CGFloat { get }
```

## 실제 구현 예제

전체 클래스에 이를 적용해본 예제는 아래와 같다.

``` swift
/// 🚲 A two-wheeled, human-powered mode of transportation.
class Bicycle {
    /// Frame and construction style.
    enum Style {
        /// A style for streets or trails.
        case road

        /// A style for long journeys.
        case touring

        /// A style for casual trips around town.
        case cruiser

        /// A style for general-purpose transportation.
        case hybrid
    }

    /// Mechanism for converting pedal power into motion.
    enum Gearing {
        /// A single, fixed gear.
        case fixed

        /// A variable-speed, disengageable gear.
        case freewheel(speeds: Int)
    }

    /// Hardware used for steering.
    enum Handlebar {
        /// A casual handlebar.
        case riser

        /// An upright handlebar.
        case café

        /// A classic handlebar.
        case drop

        /// A powerful handlebar.
        case bullhorn
    }

    /// The style of the bicycle.
    let style: Style

    /// The gearing of the bicycle.
    let gearing: Gearing

    /// The handlebar of the bicycle.
    let handlebar: Handlebar

    /// The size of the frame, in centimeters.
    let frameSize: Int

    /// The number of trips traveled by the bicycle.
    private(set) var numberOfTrips: Int

    /// The total distance traveled by the bicycle, in meters.
    private(set) var distanceTraveled: Double

    /**
     Initializes a new bicycle with the provided parts and specifications.

     - Parameters:
        - style: The style of the bicycle
        - gearing: The gearing of the bicycle
        - handlebar: The handlebar of the bicycle
        - frameSize: The frame size of the bicycle, in centimeters

     - Returns: A beautiful, brand-new bicycle,
                custom-built just for you.
     */
    init(style: Style,
         gearing: Gearing,
         handlebar: Handlebar,
         frameSize centimeters: Int)
    {
        self.style = style
        self.gearing = gearing
        self.handlebar = handlebar
        self.frameSize = centimeters

        self.numberOfTrips = 0
        self.distanceTraveled = 0
    }

    /**
     Take a bike out for a spin.

     Calling this method increments the `numberOfTrips`
     and increases `distanceTraveled` by the value of `meters`.

     - Parameter meters: The distance to travel in meters.
     - Precondition: `meters` must be greater than 0.
     */
    func travel(distance meters: Double) {
        precondition(meters > 0)
        distanceTraveled += meters
        numberOfTrips += 1
    }
}
```

<kbd>⌥ – Option</kbd>을 누른 상태에서 이니셜라이저 선언 부분을 클릭하면, 설명이 말머리표로 구분되어 예쁘게 나온다.

![image](https://user-images.githubusercontent.com/41438361/121201751-b8314200-c8af-11eb-8607-ddfe74e69fd3.png)

그리고 마찬가지로 `travel`에 대한 Quick Documentation을 열면 파라미터는 별도의 필드로 구분되어 나오는 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/121201778-bebfb980-c8af-11eb-9007-9152a83f591b.png)

추가로 또 다른 예제는 아래와 같다.

``` swift
/**
 This is your User documentation.
 A very long one.
 
 # Text
 It's very easy to make some words **bold** and other words *italic* with Markdown. You can even [link to Google!](http://google.com)
 
 # Lists
 Sometimes you want numbered lists:

 1. One
 2. Two
 3. Three

 Sometimes you want bullet points:

 * Start a line with a star
 * Profit!

 Alternatively,

 - Dashes work just as well
 - And if you have sub points, put two spaces before the dash or star:
   - Like this
   - And this
 
 # Code
 
     if (isAwesome){
       return true
     }

*/
struct User {
    let firstName: String
    let lastName: String
}
```

위 코드는 아래와 같이 표시된다.

![image](https://user-images.githubusercontent.com/41438361/121201853-cc753f00-c8af-11eb-8952-bc1962bb0712.png)

## MARK / TODO / FIXME

Objective-C에서 전처리기 지시문 `#pragma mark`는 기능을 의미있고 쉽게 찾을 수 있는 섹션으로 나누기 위해 사용되었다. Swift에서는 이를 `// MARK:`를 이용해서 같은 작업을 할 수 있다.

아래의 주석들은 Xcode source navigator에 노출된다.

* `// MARK:`
* `// TODO:`
* `// FIXME:`

기존에 사용되었던 `NOTE`나 `XXX`와 같은 주석들은 Xcode에서 인식되지 않는다.

*`#pragma`와 같이, 하나의 대시(`-`)가 뒤에 붙여진 마크들은 앞에 수평 분할기가 표시된다.*

이게 실제로 작동하는 것을 보기 위해, 위의 `Bicycle` 클래스가 `CustomStringConvertible` 프로토콜을 채택하고, `description` 프로퍼티를 구현해서 확장하는 방법은 아래와 같다.

![image](https://user-images.githubusercontent.com/41438361/121201886-d4cd7a00-c8af-11eb-8073-b40096418345.png)

``` swift
// MARK: - CustomStringConvertible

extension Bicycle: CustomStringConvertible {
    public var description: String {
        var descriptors: [String] = []

        switch self.style {
        case .road:
            descriptors.append("A road bike for streets or trails")
        case .touring:
            descriptors.append("A touring bike for long journeys")
        case .cruiser:
            descriptors.append("A cruiser bike for casual trips around town")
        case .hybrid:
            descriptors.append("A hybrid bike for general-purpose transportation")
        }

        switch self.gearing {
        case .fixed:
            descriptors.append("with a single, fixed gear")
        case .freewheel(let n):
            descriptors.append("with a \(n)-speed freewheel gear")
        }

        switch self.handlebar {
        case .riser:
            descriptors.append("and casual, riser handlebars")
        case .café:
            descriptors.append("and upright, café handlebars")
        case .drop:
            descriptors.append("and classic, drop handlebars")
        case .bullhorn:
            descriptors.append("and powerful bullhorn handlebars")
        }

        descriptors.append("on a \(frameSize)\" frame")

        // FIXME: Use a distance formatter
        descriptors.append("with a total of \(distanceTraveled) meters traveled over \(numberOfTrips) trips.")

        // TODO: Allow bikes to be named?

        return descriptors.joined(separator: ", ")
    }
}
```

이를 코드에서 모두 보여준다.

``` swift
var bike = Bicycle(style: .road,
                   gearing: .freewheel(speeds: 8),
                   handlebar: .drop,
                   frameSize: 53)

bike.travel(distance: 1_500) // Trip around the town
bike.travel(distance: 200) // Trip to the store

print(bike)
// "A road bike for streets or trails, with a 8-speed freewheel gear, and classic, drop handlebars, on a 53" frame, with a total of 1700.0 meters traveled over 2 trips."
```

## Quick help popover

위에서도 잠깐 언급했는데,<kbd>⌥ – Option</kbd>를 눌러서 quick help를 열 수 있다.

![image](https://user-images.githubusercontent.com/41438361/121201921-ddbe4b80-c8af-11eb-999c-391c4f25f259.png)

또한 Quick help는 Quick Help inspector 패널에서도 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/121201977-e747b380-c8af-11eb-8577-c212dbde3a52.png)

### Quick Help documentation 자동 생성

문서를 더 쉽게 생성할 수 있다.<kbd>⌥ – Option</kbd>+<kbd>⌘ - command</kbd>+<kbd>/</kbd>를 누르거나 Xcode 메뉴의 Editor > Structure > Add documentation 를 선택하면 문서 템플릿을 만들 수 있다.

``` swift
/// <#Description#>
/// - Parameter person: <#person description#>
```

*이 기능은 Xcode 버전에 따라 다를 수 있다.*

## Code completion hint

타이핑을 시작할 때, Xcode는 해당 클래스에서 추천하는 함수/프로퍼티/열거형의 리스트와 맨 아래에 해당 작업으로 어떤 일을 할 수 있는지 힌트를 준다.

![image](https://user-images.githubusercontent.com/41438361/121202015-ee6ec180-c8af-11eb-88d4-8d845b0f3b3f.png)


출처

* 원문 : [Swift Documentation](https://nshipster.com/swift-documentation/)
* 추가로 참고해 덧붙인 내용 : [Swift Documentation](https://sarunw.com/posts/swift-documentation/)
