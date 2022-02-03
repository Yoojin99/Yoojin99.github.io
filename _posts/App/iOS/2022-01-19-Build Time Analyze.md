---  
layout: post  
title: "[iOS] - Build Time Analyzing, 빌드 시간 분석하기"  
subtitle: ""  
categories: app
tags: app-ios Xcode preview 
comments: true  
header-img: 

---  
  
> `빌드 시간을 분석하여 빌드가 오래 걸리는 부분을 찾아내 개선하자.`  

---

빌드 시간을 줄여야 하는 이유가 무엇인지, 어떤 부분에서 빌드 시간이 증가하는지, 빌드 시간을 분석할 수 있는 방법들에 대해 여러 글들을 정리하면서 알아봤다.

# 빌드 시간을 단축해야 하는 이유? (Keep the Build Fast)

빌드 시간을 단축해야 하는 이유는 무엇일까? 단순히 빌드에 소요되는 시간을 단축해서 작업 속도를 빠르게 하기 위함이기도 하지만, 더 자세하고 구체적인 이유들이 있다.

먼저 Continous Integration(지속적 통합)이라는 개념이 있다. 지속적 통합은 팀 멤버들이 그들의 작업을 자주 통합하는 소프트웨어 개발 방법으로, 주로 각 사람마다 하루에 최소 한 번 통합해서 하루에 여러번의 통합이 일어나는 것이다.
각 통합은 통합할 때의 에러를 최대한 빨리 탐지하기 위해 테스트를 포함한 자동화된 빌드에 의해 검증된다. 많은 팀들은 이런 접근 방식이 통합할 때의 문제를 줄여주고 팀이 결합력 있는 소프트웨어를 더욱 빠르게 개발하게 해준다는 것을 발견했다.

이 지속적인 통합에서 중요한 요소 중 하나는 빠른 피드백을 제공하는 것이다. CI(Continous Integration) 활동을 가장 저해하는 것은 빌드가 오래 걸리는 것이다. 여기에서 Martinfowler는 보통 빌드에 한 시간이 걸리면 납득할 수 없는 시간이라고 생각한다고 얘기한다.
하지만 이 빌드 속도를 가속화하는 것이 어려운 작업인 경우가 있다.

하지만 대부분의 프로젝트에서 10분 빌드에 대한 XP 가이드라인이 적용되는 것은 합리적이다. 대부분의 현대 프로젝트들이 이를 만족하고 있다. 빌드 시간을 일 분 줄이는 것은 각 개발자들이 매번 commit할 때마다 일 분씩 아끼게 해주는 것이기 때문에 빌드 시간을 줄이는 것은 의미가 있는 작업이다.
CI가 빈번한 commit을 요구하기 때문에, 빌드를 하는데 걸리는 시간도 늘어날 수밖에 없다.

아마 가장 중요한 단계는 deployment pipeline을 설정하는 단계일 것이다. Deployment pipeline(build pipeline/staged build)는 여러 번의 빌드가 순서대로 수행된다는 생각이 뒷받침하고 있다. Mainline에 commit을 하면 첫 번째 빌드를 실행시키는데, 이를 commit build라 한다.
Commit build는 누군가가 mainline에 필요한 빌드이다. Commit build는 빨리 완료되어야 하는 빌드이고, 결과적으로 버그를 찾을 확률이 낮아지게 될 것이다. 이는 버그를 찾는 수준을 조절해서 다른 사람이
계속 작업할 수 있을 만한 적당한 속도를 찾는 것이 중요하다.

Commit build가 성공적으로 완료되면 다른 사람들은 신뢰할 수 있는 기반에서 작업할 수 있다. 이것의 간단한 예시는 바로 두 단계 deployment pipeline이다. 첫 번째 단계는 컴파일을 하고 데이터 베이스가 완전히 제거된 상태에서 더 지역화 된 유닛 
이런 테스트는 굉장히 10분 가이드라인을 지키면서 굉장히 빠르게 실행될 수 있다. 하지만 더 넓은 스케일의 상호작용, 특히 실제 데이터베이스를 포함한 상황에서 발생할 수 있는 버그는 찾지 못할 것이다. 두 번째 단계 빌드는 실제 데이터베이스를 포함하고 더 많은 end-to-end 행동을 포함한 테스트들을 수행할 것이다. 이는 몇 시간이 걸릴 수 있다.

이 시나리오에서 사람들은 첫 번째 단계를 commit build로 사용하고 주 CI 사이클로 사용한다. 두 번째 단계의 빌드는 더 많은 테스팅을 위해 최근에 성공적으로 완료된 commit build를 골라 실행할 수 있을 때 실행된다. 만약 두 번째 빌드가 실패한다면 이는 '모든 것을 멈추는'정도는 아니지만, 팀은 commit build가 계속 실행되게 유지하면서 버그를 최대한 빨리 고쳐야 한다.

만약 두 번째 빌드가 버그를 발견한다면, Commit build가 다른 테스트를 수행해야 할 수도 있다는 사인이다. 이후 단계의 빌드가 실패한 것으로 해당 버그를 잡는 새로운 테스트를 commit build에 포함할수록 버그는 commit build에서 수정되어 있을 것이다. 이 방법으로 commit test들을 할수록 보강될 것이다. 버그를 밝혀낼 수 있는 빠른 속도로 실행되는 테스트를 빌드할 방법이 없을 수 있다. 하지만 대부분의 경우 적절한 테스트를 commit build에 포함시킬 수 있다.

위의 예제는 두 단계 pipeline이었는데, 이 기본 이론을 더 많은 단계로 확장시킬 수 있다. commit build 이후의 빌드들은 병행될 수 있기 때문에 두 번째 단계에서 테스트를 수행하는데 2시간이 소요된다면 테스트들을 반반씩 나눠서 수행하는 두 대의 머신으로 나눠 빌드를 수행할 수 있을 것이다. 

더 자세한 내용은 [martinFowler의 continous Integration 글](https://www.martinfowler.com/articles/continuousIntegration.html)을 읽어보면 좋을 것 같다.

# Speeding Up Slow Swift Build Times

[Speeding Up Slow Swift Build Times 글 원문](https://medium.com/swift-programming/speeding-up-slow-swift-build-times-922feeba5780#.k0pngnkns)

사람들은 타입 추론때문에 Swift 2.2 컴파일러가 단순한 코드를 컴파일하는데 12시간 이상이 걸릴 수 있는 현상에 흥미로워 하는 것 같다.(2016년 글이라 현재는 이런 현상이 일어나지 않는다.) [Matt Nedrich의 이 글](https://spin.atomicobject.com/2016/04/26/swift-long-compile-time/)을 보면 우리는 한 단순한 코드가 어떤 타입이 사용되었는지를 알아내는데 오랜 시간이 걸리는 것을 알 수 있다. 지금 당장 빌드하는데 시간이 적게 걸린다 해도 Swift 코드를 모두 컴파일 할 때의 시간이 점점 늘어나서 빌드하는데 시간이 오래 걸리게 되는 걸 경계해야 한다.

만약 Swift 프로젝트에서 컴파일하는데 너무 오래 걸린다면 컴파일러의 debug-time-function-bodies 옵션을 킬 수 있다. Swift compiler는 빌드 할 때 진단할 수 있는 여러 옵션들이 있다. 

* -driver-time-compilation : High-level timing of the jobs that the driver executes.
* -Xfrontend -debug-time-compilation : Timers for each phase of frontend job execution.
* -Xfrontend -debug-time-function-bodies : Time spent type checking every function in the program.
* -Xfrontend -debug-time-expression-type-checking : Time spent type checking every expression in the program.

Xcode의 Build Settings에 가서 Other Swift Flags를 -Xfrontend -debug-time-function-bodies 로 바꿔주자. 이러면 이제 프로그램의 모든 함수에서 type checking 하는데 걸리는 시간을 알 수 있다. 즉 각 함수마다 컴파일 하는데 걸리는 시간을 기록할 것이다. 

![image](https://user-images.githubusercontent.com/41438361/150289013-d11eef05-d6b6-4944-ab06-763ebbde5dde.png)

이제 Build를 한 다음 Build Report 네비게이터로 이동해서 최근에 빌드된 내역을 본다.

그리고 내가 빌드한 타겟이 나온 빌드 로그에 우 클릭을 하고 Expand All Trnascripts를 눌러 더 자세한 빌드 로그를 보자.

![image](https://user-images.githubusercontent.com/41438361/150289279-ed75a332-a35b-40c5-8ff6-c449272c801c.png)
![image](https://user-images.githubusercontent.com/41438361/150289305-62a0fc7a-9710-46df-b7c5-6c9c32bd9ceb.png)

이제 박스들이 여러 개 보일텐데, 각각은 컴파일 과정 내에서 파일 혹은 한 단계를 나타낸다. 이 박스들에 있는 내용은 로딩되는데 시간이 좀 걸릴 수 있다. 만약 컴파일 시간을 보기 위해 빌드 플래그를 잘 설정했다면, 왼쪽에 빌드에 걸린 시간들을 확인할 수 있을 것이다. 이 라인들을 보고 의심스러운 부분을 찾아낼 수 있다. 백 밀리세컨드보다 더 오래 걸린 부분을 확인할 수 있을 것이다.

![image](https://user-images.githubusercontent.com/41438361/150289676-cfab4bfb-815f-4bff-9248-e4abca3b5e73.png)

예를 들어 아래의 로그를 보면 `viewDidLoad()`를 실행시키는데 111ms 정도가 걸렸다는 뜻이다. 이런 식으로 어떤 부분을 컴파일 하는데 시간이 얼마나 걸리는지를 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/150289882-117d379c-3b88-48c3-8ab9-7c64edc4aa71.png)

# Regarding Swift build time optimizations

내가 clean code라고 생각했던 한 줄의 코드에 새로운 의문을 가질 수 있다. 컴파일러를 만족시키기 위해 9줄의 코드로 리팩토링 되어야 할 필요가 있을까? 어떤 것이 더 중요할까? 간결함이냐 아니면 컴파일러에게 친숙한 코드가 되어야 할지는 프로젝트 크기와 개발자에게 달렸을 것이다.

## Xcode plugin

먼저 로그 파일들을 일일이 살펴보는 것은 시간이 많이 소요된다. 이를 위한 [Xcode plugin](https://github.com/RobertGummesson/BuildTimeAnalyzer-for-Xcode)이 있다.

초기의 목표는 시간이 오래 걸리는 부분을 찾아 고치는 것이었지만 지금은 이 작업이 더 반복적으로 수행되어야 한다고 생각한다고 한다. 이 방법으로 더 효율적으로 코드를 빌드할 수 있고, 프로젝트에 진입할 때 시간을 오래 잡아먹는 함수들을 실행하는 걸 막을 수 있다.

빌드 시간을 확인해보면서 예상치 못한 간단해보이는 한 줄의 코드가 오랜 시간 컴파일이 되는 것을 확인할 수 있다.

## Nil Coalescing Operator

아래의 예제에서 두 뷰를 unwrapping하니 빌드 시간이 99.4% 감소했다.

```swift
// Build time: 5238.3ms
return CGSize(width: size.width + (rightView?.bounds.width ?? 0) + (leftView?.bounds.width ?? 0) + 22, height: bounds.height)

// Build time: 32.4ms
var padding: CGFloat = 22
if let rightView = rightView {
    padding += rightView.bounds.width
}

if let leftView = leftView {
    padding += leftView.bounds.width
}
return CGSizeMake(size.width + padding, bounds.height)
```

Swift3에서는 아래와 같다.

```swift
// Build time Swift 2.2: 5238.3ms
// Build time Swift 3.0: 2922.5ms
return CGSize(width: size.width + (rightView?.bounds.width ?? 0) + (leftView?.bounds.width ?? 0) + 22, height: bounds.height)
```

Swift5에서는 아래와 같다.

```swift
// 9131.9ms
func getCGSize1() -> CGSize {
    return CGSize(width: 10 + (rightView?.bounds.width ?? 0) + (leftView?.bounds.width ?? 0) + 22, height: view.bounds.height)
}

// 2.3ms
func getCGSize2() -> CGSize {
    var padding: CGFloat = 22


    if let rightView = rightView {
        padding += rightView.bounds.width
    }

    if let leftView = leftView {
        padding += leftView.bounds.width
    }

    return CGSize(width: 10 + padding, height: view.bounds.height)
}
```

<img width="615" alt="image" src="https://user-images.githubusercontent.com/41438361/152283152-981f6b6c-ce31-4f27-bbb2-f88dd2f8ad40.png">
<img width="616" alt="image" src="https://user-images.githubusercontent.com/41438361/152283170-1418b845-4247-43b3-9896-96d1f59d2577.png">


## ArrayOfStuf + [Stuff]

```swift
return ArrayOfStuff + [Stuff] // no

ArrayOfStuff.append(stuff) // yes
return ArrayOfStuff
```

아래의 예제를 보면 빌드 시간이 97.9% 감소하는 것을 볼 수 있다.

```swift
// Build time: 1250.3ms
let systemOptions = [ 7, 14, 30, -1 ]
let systemNames = (0...2).map{ String(format: localizedFormat, systemOptions[$0]) } + [NSLocalizedString("everything", comment: "")]
// Some code in-between 
labelNames = Array(systemNames[0..<count]) + [systemNames.last!]

// Build time: 25.5ms
let systemOptions = [ 7, 14, 30, -1 ]
var systemNames = systemOptions.dropLast().map{ String(format: localizedFormat, $0) }
systemNames.append(NSLocalizedString("everything", comment: ""))
// Some code in-between
labelNames = Array(systemNames[0..<count])
labelNames.append(systemNames.last!)
```

Swift3에서는 아래와 같다.

```swift
// See my previous post for the detailed example
// Build time Swift 2.2: 1250.3ms
// Build time Swift 3.0: 92.7ms 🎉
ArrayOfStuff + [Stuff]
```

Swift5에서는 아래와 같다.

```swift
func appendArray1() -> [String] {
    var firstArray = ["A", "B", "C"]
    let secondArray = ["D", "E", "F"]

    firstArray += secondArray

    return firstArray
}

func appendArray2() -> [String] {
    var firstArray = ["A", "B", "C"]
    let secondArray = ["D", "E", "F"]

    firstArray.append(contentsOf: secondArray)

    return firstArray
}
```

<img width="632" alt="image" src="https://user-images.githubusercontent.com/41438361/152283358-4366bac6-4c27-4523-aff3-31bd4fc56e4f.png">


## Ternary operator

3중 연산자를 if/else 문으로 변경하니 92.9% 빌드 시간이 감소했다. 

```swift

// Build time: 239.0ms
let labelNames = type == 0 ? (1...5).map{type0ToString($0)} : (0...2).map{type1ToString($0)}

// Build time: 16.9ms
var labelNames: [String]
if type == 0 {
    labelNames = (1...5).map{type0ToString($0)}
} else {
    labelNames = (0...2).map{type1ToString($0)}
}
```

Swift 3에서는 아래와 같다.

```swift
// Build time Swift 2.2: 365.6ms
// Build time Swift 3.0: 128.4ms
let labelNames = type == 0 ? (1...5).map(type0ToString) : (0...2).map(type1ToString)
```

Swift5에서는 아래와 같다.

```swift
// 65.3ms
func ternaryOperator1(type: Int) {
    let labelName = type == 0 ? "Zero" : "NonZero"
}

// 1.5ms
func ternaryOperator2(type: Int) {
    var labelName: String

    if type == 0 {
        labelName = "Zero"
    } else {
        labelName = "NonZero"
    }
}
```

<img width="679" alt="image" src="https://user-images.githubusercontent.com/41438361/152292358-ac935c98-6448-4a2b-bbc7-bb970e7fb6cd.png">
<img width="682" alt="image" src="https://user-images.githubusercontent.com/41438361/152292391-6f6872ff-53e2-422b-9e56-d5eea24e163a.png">


## CGFloat를 CGFloat으로 캐스팅하기

이미 CGFloat인 값들이었고, 몇 괄호들은 불필요했다. 이를 해결하니 빌드 시간이 99.9% 감소했다.

```swift
// Build time: 3431.7ms
return CGFloat(M_PI) * (CGFloat((hour + hourDelta + CGFloat(minute + minuteDelta) / 60) * 5) - 15) * unit / 180

// Build time: 3.0ms
return CGFloat(M_PI) * ((hour + hourDelta + (minute + minuteDelta) / 60) * 5 - 15) * unit / 180
```

Swift5에서는 아래와 같다.

```swift
// 131.2ms
func casting1() -> CGFloat {
    let float: CGFloat = 32

    return 32 * CGFloat(float)

}

// 0.3ms
func casting2() -> CGFloat {
    let float: CGFloat = 32

    return 32 * float
}
```

<img width="599" alt="image" src="https://user-images.githubusercontent.com/41438361/152292572-f70b0230-a64a-489f-9fde-3844cc9b706d.png">
<img width="596" alt="image" src="https://user-images.githubusercontent.com/41438361/152292589-300d18e1-a583-4f5f-8090-302f2a939a55.png">



## Round()

이 예시는 좀 이상한데, 아래의 예제들은 local, instance 변수들을 섞은 것이다. 문제는 round 자체가 아니라 메서드 내의 코드의 조합일 가능성이 크다. rounding을 제거하니 97.6% 빨라졌다.

```swift
// Build time: 1433.7ms
let expansion = a — b — c + round(d * 0.66) + e
// Build time: 34.7ms
let expansion = a — b — c + d * 0.66 + e
```

Swift5에서는 아래와 같았다.

```swift
// 317.4
func calculateRound1() {
    let num: Double = 3 + 5 + round(8/3)
}

// 7.4
func calculateRound2() {
    let num: Double = 3 + (8 / 3)
}
```

<img width="642" alt="image" src="https://user-images.githubusercontent.com/41438361/152292738-2f69f5b0-d96b-418c-8d06-1831968cb2c0.png">
<img width="647" alt="image" src="https://user-images.githubusercontent.com/41438361/152292761-d794a77d-16cd-4d78-a819-c7e7e5a56231.png">


## Try it out

느린 빌드를 겪고 있든 그렇지 않든 어떤 것이 컴파일러를 헷갈리게 하는지 이해하는 것은 도움이 된다. 아래의 코드는 컴파일 하는데 5초 이상이 걸리는 코드다.

```swift

import UIKit

class CMExpandingTextField: UITextField {

    func textFieldEditingChanged() {
        invalidateIntrinsicContentSize()
    }
    
    override func intrinsicContentSize() -> CGSize {
        if isFirstResponder(), let text = text {
            let size = text.sizeWithAttributes(typingAttributes)
            return CGSize(width: size.width + (rightView?.bounds.width ?? 0) + (leftView?.bounds.width ?? 0) + 22, height: bounds.height)
        }
        return super.intrinsicContentSize()
    }
}
```

위에서 소개한 플러그인으로 정의도리 수 있는 느린 빌드의 요인에는 두 가지 타입이 있다. 첫 번째는 개인 루틴이 컴파일 하는데 너무 오래 걸리는 것이다. 다른 하나는 클로져와 lazy 프로퍼티들이 타입 체킹이 너무 많이 될 때이다. 

## Closures and lazy properties

위의 플러그인에 Occurrences라는 열이 추가되었는데, 이게 추가된 이유는 단순한 코드가 빌드를 늦출 때가 있다는 것을 쉽게 파악할 수 있기 때문이다.

![image](https://user-images.githubusercontent.com/41438361/150450008-65b0b309-7dbd-44fe-b3d0-89dc4a69d71c.png)

위에서 볼 수 있듯이, lazy getter들이 Xcode 빌드 로그에 159번 실행되고 있다. 아래의 코드는 3개의 별개 파일에서 가져온 것으로, 무엇도 코드에서 CMGridView를 참조하고 있지 않다.

```
5.7ms  /CMGridView.swift:63:27 @objc get {}
16.5ms /CMGridView.swift:63:27 @objc get {}
15.5ms /CMGridView.swift:63:27 @objc get {}
```

이걸 보면 컴파일러는 타겟에 있는 모든 .swift 파일마다 lazy 변수에 대한 타입 체킹을 하고 있는 것이다! 이는 Swift 2.2에서는 1905ms 정도 빌드 시간을 늘리는데 Swift 3.0에서 이슈는 여전하지만 빌드 시간은 거의 반 정도 준다.

또 다른 예제를 보자.

```swift
private(set) lazy var chartViewColors: [UIColor] = [
    self.chartColor,
    UIColor(red: 86/255, green: 84/255, blue: 124/255, alpha: 1),
    UIColor(red: 80/255, green: 88/255, blue: 92/255, alpha: 1),
    UIColor(red: 126/255, green: 191/255, blue: 189/255, alpha: 1),
    UIColor(red: 161/255, green: 77/255, blue: 63/255, alpha: 1),
    UIColor(red: 235/255, green: 185/255, blue: 120/255, alpha: 1),
    UIColor(red: 100/255, green: 126/255, blue: 159/255, alpha: 1),
    UIColor(red: 160/255, green: 209/255, blue: 109/255, alpha: 1),
    self.backgroundGradientView.upperColor
]
```

이를 컴파일하는데 2초가 걸리는데 이게 오래 걸린다고 생각할 수 있지만, 클로저를 사용하면 더 심해진다.

![image](https://user-images.githubusercontent.com/41438361/150451688-a8e91413-3a73-4fc7-b3ed-7dc908fbf590.png)

클로저를 사용하면 아래와 같은 코드가 된다.

```swift
private let createChartViewColors = { () -> [UIColor] in
    let colors = [
        UIColor(red: 86/255, green: 84/255, blue: 124/255, alpha: 1),
        UIColor(red: 80/255, green: 88/255, blue: 92/255, alpha: 1),
        UIColor(red: 126/255, green: 191/255, blue: 189/255, alpha: 1),
        UIColor(red: 161/255, green: 77/255, blue: 63/255, alpha: 1),
        UIColor(red: 235/255, green: 185/255, blue: 120/255, alpha: 1),
        UIColor(red: 100/255, green: 126/255, blue: 159/255, alpha: 1),
        UIColor(red: 160/255, green: 209/255, blue: 109/255, alpha: 1),
    ]
    return colors
}
```

## Workaround

이를 개선하려면 코드를 가능하면 private 메서드로 옮기면 된다.

```swift
// Cumulative build time: 56.3ms
private(set) lazy var chartViewColors: [UIColor] = self.createChartViewColors()

// Build time: 6.2ms
private func createChartViewColors() -> [UIColor] {
    return [
        chartColor,
        UIColor(red: 86/255, green: 84/255, blue: 124/255, alpha: 1),
        UIColor(red: 80/255, green: 88/255, blue: 92/255, alpha: 1),
        UIColor(red: 126/255, green: 191/255, blue: 189/255, alpha: 1),
        UIColor(red: 161/255, green: 77/255, blue: 63/255, alpha: 1),
        UIColor(red: 235/255, green: 185/255, blue: 120/255, alpha: 1),
        UIColor(red: 100/255, green: 126/255, blue: 159/255, alpha: 1),
        UIColor(red: 160/255, green: 209/255, blue: 109/255, alpha: 1),
        backgroundGradientView.upperColor
    ]
}
```

위와 같이 코드를 분리하면 빌드 시간은 96.7% 감소된다.

Swift5에서도 확인해보니 아래와 같았다.

* 클로저 사용
<img width="481" alt="image" src="https://user-images.githubusercontent.com/41438361/152293346-bef22b35-9291-4ed1-a2ae-70aedf0b6346.png">
* private 함수로 만들었을 때
<img width="692" alt="image" src="https://user-images.githubusercontent.com/41438361/152293376-a1ef3708-d982-4315-8efa-a934ea2070a3.png">
* private lazy var getter 사용
<img width="580" alt="image" src="https://user-images.githubusercontent.com/41438361/152293417-6e953a0f-d7b7-4b45-b841-1e72786134a1.png">


## Swift 3.0에서의 빌드 시간

Xcode 8.0에서, Xcode의 플러그인 시대가 끝나고 extension의 시대가 시작됐다. Extension에 한계가 있기 때문에 위의 plug in이 별도의 앱으로 만들어졌으니 깃헙에서 확인하면 될 것 같다.

# Analyzing and Improving Build times in iOS

이 부분에서는 

1. XCLogParser로 XCode Project의 빌드 시간을 분석하는 방법
2. Sitrep으로 Swift 코드를 분석하는 법
3. 어떻게 iOS 프로젝트 디버그 빌드 시간을 40%로 줄일 수 있었는지
4. 어떻게 Swift 컴파일러 버그가 릴리즈 빌드를 50분 정도 잡아먹는지, 어떻게 해결했는지

## Debug-Mode

### Sitrep for some stats

앱의 크기와 관련된 걸 알고 싶다면 [Sitrep](https://github.com/twostraws/Sitrep)을 사용해서 분석치를 볼 수 있다. Sitrep은 Swift Project 코드 상태에 대한 빠른 통찰을 준다. 이를 사용해서 각 모듈, 앱, Pods의 코드 줄 수를 세는데 사용할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/150471717-b49f6dee-b56d-4448-a29d-a38a00d44072.png)

Sitrep을 통해 아래의 내용을 확인할 수 있다.

* Class, struct, enum, protocol, extension의 개수
* 전체 코드 줄 수, 소스 라인 줄 수(주석과 공백을 뺀 것)
* 어떤 파일과 타입이 가장 긴지, 또 그들의 코드 줄 수
* 사용하는 import와 얼마나 자주 import 하는지
* UIView, UIViewController, SwiftUI 가 얼마나 있는지 등등

### Using Cocoapods

Cocoapod는 소스 코드를 확인하고 앱 프로젝트에 일부를 이를 알려줌으로써 의존성을 관리한다. 우리가 앱을 컴파일 할때마다 dependencies code도 컴파일된다. 그래서 프로젝트는 프레임워크 하나를 빌드할때마다 빌드 시간이 오래 걸린다. 

아래 이미지는 빌드 리포트의 분석의 일부를 보여주는 걸로, XCLogParser로 생성한 것이다.

![image](https://user-images.githubusercontent.com/41438361/150639663-8d51e8a8-0c45-4308-b8d6-ec9b59dbd645.png)

이를 어떻게 개선할 수 있을까? 프로젝트를 컴파일 할 때마다 변하지 않은 프레임워크들에 대해서도 컴파일하는 걸 피할 수 있는 방법이 있을까?

방법 하나는 외부 dependencies를 한 번만 컴파일하고 미리 컴파일된 프레임워크(dynamic libraries)를 사용하는 것이다.

### Using Carthage(카르타고) and Pre-Compiled Frameworks

Carthage로 우리는 dependencies를 특정하고 앱 타겟 빌드 프로세스와 별개로 한 번만 컴파일할 수 있다. Dependencies가 컴파일 되면, 앱 타겟은 이걸 사용해서 빌드하고, 링크하고, 실행한다.

이 컴파일은 `carthage bootstrap`나 `carthage update`를 실행하면 발생한다. 이 도구는 `xcodebuild`로 프레임워크 별로 타겟을 컴파일하고 결과로 `.framework` object들을 얻게 된다. 

그래서 앱 타겟은 소스 코드 대신에 미리 컴파일된 프레임워크를 참조하게 된다. 우리가 clean build를 할 때, Xcode는 외부 dependencies들을 컴파일 하는 걸 피하고 앱 소스 코드만을 컴파일 한다. 이는 많은 시간을 아끼게 해준다.

Carthage에 의해 관리된 미리 컴파일된 프레임워크를 사용해서 같은 앱을 빌드한 시간을 확인해보자. 40% 감소된 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/150640053-bd286195-1974-423a-a8ef-841aa3fab108.png)

이렇게 향상된 이유는 바로 컴파일 할 코드의 양이 줄었기 때문이아! Pods는 앱 코드의 44.4%를 차지하고 있었다. 대부분의 dependencies들을 Carthage로 옮기니 Pods 는 11.5%로 감소했다.

![image](https://user-images.githubusercontent.com/41438361/150640120-d5a6945d-0ce9-45aa-ab02-860eb3c39636.png)

이는 컴파일 할 코드의 양이 37% 감소했음을 의미한다. 전체 줄 수가 1879990줄에서 118037줄로 감소했다.

### 이후의 개선점들

**Pre-Compiled Modules**

앱의 아키텍처는 modular 아키텍처로, 메인 타겟의 일부로써 컴파일되는 모든 모듈에 의존하고 있다. 더 큰 규모의 프로젝트를 위한 개선점은 이런 모듈들을 Carthage dependecies로 다루는 것이 될 것이다. 이는 미리 각 모듈마다 미리 컴파일하는 것을 요구할 것이지만 메인 타겟을 컴파일링 할 때는 시간을 절약해줄 것이다. 이 접근법은 다른 모듈들의 소스 코드 내의 변화를 다루는 git submodule 들을 사용해서 구현될 수 있다.

**Static Frameworks**

다른 개선점은 static framework들을 사용하는 것이 될 것이다. 이것이 컴파일 시간을 향상시키지는 않더라도 이론적으로는 앱의 launch time을 향상시킨다.

**SPM maybe**

아직 SPM을 시도해보지 않았지만, 같은 걸 Swift Package Manager로 할 수 있고 아마도 프로젝트를 구성하는 데 더 적은 단계를 거치게 되지 않을까? 

**Migrating from Cocoapods to Carthage**

Cocoapods에서 Carthage로 옮기는 작업은 단순했지만 많은 시도와 에러와 테스팅을 거쳐서 오래 걸렸다. Carthage가 Xcode, 빌드 단계, 프로젝트 설정 등에 대한 지식을 요구하는 것은 맞다. 하지만 그닥 어렵지 않아 시도해 볼만 하다.

예를 들어 각 모듈의 Framework search path와 메인 타겟이 `Carthage/IOS/Build` 폴더를 포함한다는 것, 또 Link Binary with Libraries 빌드 단계에 프레임워크를 포함시켜야 하고 문서에 나온대로 Carthage 스크립트를 적절히 만들어야 하는 것도 중요한다.

Firebase stack과 같은 어떤 의존성들은 Carthage를 사용해서 통합하기에 까다롭다. 이런 경우는 Carthage와 Cocoapods를 함께 사용하는 방법을 사용할 수 있다. 


## Release-Mode

몇달 동안, 프로젝트는 컴파일하고 Bitrise에서 TestFlight로 업로드하는데 1.2 시간 이상이 걸렸다. 처음에는 이게 왜인지 모르고 Bitrise 이슈라고 생각했다. 

Bitrise의 한 멋진 기능은 workflow를 실행하고 원격으로 가상 거신을 볼 수 있는 것이다. 이 방법으로 어떤 상황인지 파악할 수 있었다. Bitrise workflows, Build Phase Scripts, Cocoapods에서 Carthage로 옮기는 등 컴파일 시간을 줄이는 여러 시도 후에 문제가 훨씬 단순하다는 것을 알게 되었다. Release-Build 모드 자체가 문제였다. Carthage, post-build scripts(Sourcery, SwiftLint 등), Bitrise도 아무 문제가 없었다. 

그래서 Release-Mode 빌드가 너무 오래 시간이 걸린다는 것을 알게 됐는데, 이유는 왜일까? Xcode는 관련된 정보를 제공하지 않는다. 빌드가 45분 동안 실행되고 컴파일러가 한 10개의 파일에 매달려 있다는 것을 알 수 있었다.

컴파일 시간을 리포트 하기 위해 diagnostic flag와 함께 컴파일 할 떄도 Xcode 리포트는 쓸모없었다. Xcode 리포트는 단지 컴파일 된 파일들의 리스트를 보여주고, 컴파일러는 리포트가 아무런 관련이 없는 정보를 준 후 약 30분 동안 멈춰있었다.

그래서 릴리즈 모드에서의 코드 최적화가 문제가 된다고 생각했고, 이것이 맞았다. 디버그 모드에서 코드 최적화 컴파일러 flag를 설정하고 컴파일하니 6-8 분에서 45분으로 늘어났다.

결국 문제는 코드 최적화였다. 최적화 없는 컴파일 시간은 6-8분이었으나 최적화와 함꼐 빌드하니 45분이 소요됐다.

### XCLogParser로 코드 분석하기

그래서 어떤 클래스, 메서드가 컴파일 하는데 시간이 오래 걸리게 만들었을까? 이런 정보는 Xcode 빌드 리포트에서 확인할 수 없다. 이떄 사용할 수 있는게 XCLogParser다. 

이 도구는 컴파일하는데 오랜 시간이 걸리는 파일들을 확인할 수 있게 해준다. 파일들을 분석하고 나서 굉장히 커서 수상하게 보이는 struct에 대한 switch 문을 확인할 수 있었다. 이를 주석 처리하고 나니 갑자기 빌드 시간이 줄어들었다.

이 이슈를 해결하기 위해 다른 엔티티에 `==`를 호출하는 switch 문 내의 코드를 바꿨다. Identifier를 사용해서 코드를 단순화했고, 이것이 코드 최적화의 복잡도를 낮추는 것으로 보였다. 

### 코드 최적화

이 프로젝트 도중, 코드 최적화와 switch 문을 다루는 것과 관련이 있는 Swift 컴파일러와 관련된 두 이슈를 찾을 수 있었다.

여기서 언급한 케이스에서는 Swift 컴파일러는 40분 동안 매달렸고 switch 문을 포함하는 다른 enum과 비교하는 문장을 포함한 enum 내의 switch 문구를 최적화하는 작업을 했다. 이 현상을 단순한 예시 코드로 표현할 수 없는데, 하여간 내부 구조체의 equals 메서드를 참조하는 switch 문을 바꾸니 문제가 해결되었다.

## 결론

* Sitrep은 프로젝트의 전반적인 수치를 빠르게 분석할 수 있게 해주는 도구다. 클래스, 구조체, 코드 줄 수 등을 셀 수 있다. 시간이 가면서 프로젝트가 어떻게 변하는지 보는 것도 좋다.
* XCLogParser는 우리의 빌드를 분석하기에 좋은 도구다. 우리의 빌드를 이해하는 것은 더 빠른 컴파일 시간을 달성하는데 좋다.
* Bitrise의 [Build with Remote Access](https://blog.bitrise.io/post/remote-access-now-available-on-bitrise)는 라이브에서 CI build에 어떤 문제가 있는지 확인하기 쉽게 해준다. 
* 미리 빌드된 프레임워크는 컴파일 시간을 더 빠르게 하고 이를 위해 최적화하는 데 Carthage는 Cocoapod의 좋은 대체제다.
* 마지막으로, 최적화의 여부는 컴파일하는데 별 차이가 없다. 위의 예시들에서 컴파일러는 코드를 최적화하는 도중 복잡한 Swift 문장에서 멈췄었다. 이는 컴파일러 내의 버그로 인해 발생했을 수 있다. 

# How To Boost Xcode's Compile Time and Runtime

## Project Settings

### 1. New Build System을 사용해라

File > Menu > Workspace Settings(Workspace를 사용하고 있지 않다면 Project Settings) 에서 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/150730651-b7ef7bef-c099-4032-9e0e-facdde9c4a1f.png)

### 2. 활성화된 아키텍처만 빌드하기

프로젝트 빌드 설정에서 "Build Active Architecture Only" 로 이동한다. "Debug"를 "Yes"로, "Release"를 "No"로 설정한다.

![image](https://user-images.githubusercontent.com/41438361/150730923-3734c5d2-d7c2-4d80-8a3c-da9d70e5f959.png)

### 3. dSYM 파일 생성 최적화하기

"Debug Information Format"을 릴리즈 빌드에서 dSYM 파일을 항상 생성하는 것으로 설정한다. 디버그 빌드에서는 필요하지 않을 수 있다.

![image](https://user-images.githubusercontent.com/41438361/150731135-33899db6-57d7-45ba-9b09-ad66eeff4de1.png)


### 최적화 레벨

"Debug"를 "-Onone"으로 설정하고, "Release"를 "-O"나 "-Osize"로 설정한다.

![image](https://user-images.githubusercontent.com/41438361/150734553-9c0c86fd-4877-43fd-a3f5-6240eee85fb2.png)

CocoaPods에서는 아래를 Podfile의 끝에 추가해서 모든 dependencies를 최적화한다.

```rb
post_install do |installer|
 installer.pods_project.targets.each do |target|
   target.build_configurations.each do |config|
     if config.name == 'Debug'
       config.build_settings['OTHER_SWIFT_FLAGS'] = ['$(inherited)', '-Onone']
       config.build_settings['SWIFT_OPTIMIZATION_LEVEL'] = '-O'
     end
   end
 end
end
```

### 컴파일 모드

"Debug"를 "Incremental"로, "Release"를 "Whole Module"로 설정한다.

![image](https://user-images.githubusercontent.com/41438361/150734831-e95cd106-e435-4bfd-b891-97d5a9e80628.png)

CocoaPods에서는 아래를 Podfile의 끝에 추가해서 모든 dependencies를 최적화한다.

```rb
post_install do |installer|
 installer.pods_project.targets.each do |target|
   target.build_configurations.each do |config|
     if config.name == 'Debug'
       config.build_settings['SWIFT_COMPILATION_MODE'] = 'singlefile'
     else
       config.build_settings['SWIFT_COMPILATION_MODE'] = 'wholemodule'
     end
   end
 end
end
```

### 필요할 떄만 런 스크립트를 실행한다.

아래의 스크립트는 빌드 설정과 무관하게 항상 실행된다.

![image](https://user-images.githubusercontent.com/41438361/150735191-99f37f98-207a-4314-9c69-6fde32b33f4f.png)

이는 Debug 설정일때만 실행되면 되니 이를 디버그 빌드에서만 실행되게 바꿀 수 있다.

```rb
if [ "${CONFIGURATION}" = "Debug" ]; then
 "${PODS_ROOT}/SwiftLint/swiftlint"
else
 echo "Not running SwiftLint/swiftlint because we are building for Release"
fi
```

반대로 릴리즈 빌드에서만 실행되게 하려는 실행 스크립트가 있다면 아래와 같이 하면 된다.

```rb
if [ "${CONFIGURATION}" = "Release" ]; then
  "${PODS_ROOT}/FirebaseCrashlytics/run"
else
  echo "Not running FirebaseCrashlytics/run because we are building for debug"
fi
```

## Xcode Setting

### 1. 컴파일 시간을 재기

아래의 커맨드를 터미널에 입력한다.

```
defaults write com.apple.dt.Xcode ShowBuildOperationDuration YES
```

Xcode를 닫고 다시 열고 프로젝트를 빌드하면 컴파일 성공 메세지 끝에 컴파일 시간을 볼 수 있다.

![image](https://user-images.githubusercontent.com/41438361/150735751-5e265b46-f6c6-4e59-ae27-185c6e6d6b9a.png)

### 2. 컴파일 하는데 오래 걸리는 코드를 보기

프로젝트 설정의 "Other Swift Flags" 에 아래의 줄을 추가한다.

```
-Xfrontend -warn-long-function-bodies=100
-Xfrontend -warn-long-expression-type-checking=30
```

`100`, `30`은 밀리세컨이다.

![image](https://user-images.githubusercontent.com/41438361/150736138-bfda6eb8-1ad2-4dbc-9f91-30f9a4cb05a3.png)

이제 프로젝트에서 컴파일 하는데 오래 걸리는 코드가 무엇인지 확인할 수 있다. 이제 이런 영역을 확인해서 다시 작성할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/150736263-1a9929de-2fea-4542-bb4a-f3096697f29e.png)

가장 정확한 컴파일 시간을 알고 싶다면 Clean build를 하고 derived data를 삭제하면 알 수 있다.

### 3. 함수들의 컴파일 시간 확인하기

"Other Swift Flag"에 아래의 줄을 추가한다.

```
-Xfrontend -debug-time-function-bodies
```

이러면 프로젝트에서 각 함수마다 컴파일 시간을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/150736532-8c958e8b-de1d-4cf1-a92d-1d7cc9c3d760.png)

### 4. Xcode에서 병행 빌드 활성화하기(고사양 RAM 필요)

아래의 줄을 커맨드에 친다.

```
defaults write com.apple.dt.Xcode BuildSystemScheduleInherentlyParallelCommandsExclusively -bool NO
```

이를 비활성화하려면 아래의 줄을 커맨드에 친다.

```
defaults delete com.apple.dt.Xcode BuildSystemScheduleInherentlyParallelCommandsExclusively
```

만약 충분한 메모리가 없다면 속도를 더 늦추게 될 수 있다.

## 코드 최적화

### 1. 재사용

만약 아래의 콛와 같이 여러 곳에서 `isValidEmail`이라는 변수가 있는 `String` extension이 있다고 해보자. 만약 코드의 한 복사본이 컴파일 하는데 50ms 가 걸린다면, 다른 복사본도 컴파일하는데 같은 시간이 걸릴 것이다. 따라서 두 개의 복사본이 있다면 100ms 정도 컴파일하는데 걸릴 것이다.


파일 1

```swift
// Bad: Writing isValidEmail variable twice
class LoginViewController: UIViewController {
    //...
}

fileprivate extension String {
    var isValidEmail : Bool {
        let emailRegEx = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
        let predicate = NSPredicate(format: "SELF MATCHES[c] %@", regex)
        return predicate.evaluate(with: self)
    }
}
```

파일 2

```swift
// Bad: Writing isValidEmail variable twice
class SignupViewController: UIViewController {
    //...
}

fileprivate extension String {
    var isValidEmail : Bool {
        let emailRegEx = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
        let predicate = NSPredicate(format: "SELF MATCHES[c] %@", regex)
        return predicate.evaluate(with: self)
    }
}
```

우리는 위의 코드를 새로운 extension 파일로 작성하고 두 파일에서 사용이 가능한 코드의 단일한 버전을 만들 수 있다. 버전이 단일하다면 컴파일 하는데 50ms만 걸리 것이다.

파일 1

```swift
class LoginViewController: UIViewController {
  //...
}
```

파일 2

```swift
class SignupViewController: UIViewController {
  //...
}
```

파일 3

```swift
// Good: Seperate extension which can be used multiple times
extension String {
    var isValidEmail : Bool {
        let emailRegEx = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
        let predicate = NSPredicate(format: "SELF MATCHES[c] %@", regex)
        return predicate.evaluate(with: self)
    }
}
```

또 비슷한 코드를 여러 번 복붙하는 경우가 있는데, 모든 코드는 다르다는 관점에서 컴파일러는 추가적인 불필요한 작업을 하게 된다.

즉 아래와 같이 여러 비슷한 코드가 반복되는 것을 지양해야 한다.

```swift
// Bad: Doing same kind of things again and again in different functions.
class SubscriptionViewController {

  @IBAction func getOneMonth(_ sender: UIButton) {
    tokenInt = 1
    imgViewOneBg.image = UIImage(named: "Path 6")
    imgViewThreeMonthBg.image = nil
    imgViewSixMonthsBg.image = nil
    imgViewOneYearBg.image = nil
        
    imgViewOneBg.layer.borderColor = UIColor.clear.cgColor
    imgViewThreeMonthBg.layer.borderColor = UIColor.lightGray.cgColor
    imgViewSixMonthsBg.layer.borderColor = UIColor.lightGray.cgColor
    imgViewOneYearBg.layer.borderColor = UIColor.lightGray.cgColor
  }

  @IBAction func getThreeMonths(_ sender: UIButton) {
    tokenInt = 3
    imgViewOneBg.image = nil
    imgViewThreeMonthBg.image = UIImage(named: "Path 6")
    imgViewSixMonthsBg.image = nil
    imgViewOneYearBg.image = nil
    
    imgViewOneBg.layer.borderColor = UIColor.lightGray.cgColor
    imgViewThreeMonthBg.layer.borderColor = UIColor.clear.cgColor
    imgViewSixMonthsBg.layer.borderColor = UIColor.lightGray.cgColor
    imgViewOneYearBg.layer.borderColor = UIColor.lightGray.cgColor
  }

  @IBAction func getSixMonths(_ sender: UIButton) {
    tokenInt = 6
    imgViewOneBg.image = nil
    imgViewThreeMonthBg.image = nil
    imgViewSixMonthsBg.image = UIImage(named: "Path 6")
    imgViewOneYearBg.image = nil
    
    imgViewOneBg.layer.borderColor = UIColor.lightGray.cgColor
    imgViewThreeMonthBg.layer.borderColor = UIColor.lightGray.cgColor
    imgViewSixMonthsBg.layer.borderColor = UIColor.clear.cgColor
    imgViewOneYearBg.layer.borderColor = UIColor.lightGray.cgColor
  }

  @IBAction func getOneYear(_ sender: UIButton) {
    tokenInt = 12
    imgViewOneBg.image = nil
    imgViewThreeMonthBg.image = nil
    imgViewSixMonthsBg.image = nil
    imgViewOneYearBg.image = UIImage(named: "Path 6")
       
    imgViewOneBg.layer.borderColor = UIColor.lightGray.cgColor
    imgViewThreeMonthBg.layer.borderColor = UIColor.lightGray.cgColor
    imgViewSixMonthsBg.layer.borderColor = UIColor.lightGray.cgColor
    imgViewOneYearBg.layer.borderColor = UIColor.clear.cgColor
  }  
}
```

위와 같은 코드는 논리적인 접근 방법을 사용해서 아래와 같이 최적화할 수 있다. 코드는 로직에 의해 줄어들고, 비슷한 여러 코드를 컴파일 할 때보다 컴파일 하는데 시간이 적게 걸릴 것이다.

```swift
// Good: Same code rewritten in a better way and less lines to compile faster.
class SubscriptionViewController {

  @IBAction func optionTapped(_ sender: UIButton) {
    tokenInt = sender.tag

    let imageViews: [UIImageView] = [imgViewOneBg, imgViewThreeMonthBg, imgViewSixMonthsBg, imgViewOneYearBg]

    for imageView in imageViews {
        if sender.tag == imageView.tag {
            imageView.image = UIImage(named: "Path 6")
            imageView.layer.borderColor = UIColor.clear.cgColor
        } else {
            imageView.image = nil
            imageView.layer.borderColor = UIColor.lightGray.cgColor
        }
    }
}
```

실제로 여러 파일에 중복되는 코드가 있을 때 아래와 같이 여러 번 컴파일 일어나는 것을 확인할 수 있다.

<img width="549" alt="image" src="https://user-images.githubusercontent.com/41438361/152294032-7c6bd010-ffa8-402a-b61d-f3ce58e3f2c5.png">


### 2. 불필요한 공백 지우기

아무것도 하지 않는 코드 또한 컴파일러에 의해 컴파일되기 때문에 이런 코드를 지우는 것은 컴파일 시간을 줄이는데 도움이 된다.

```swift
// Bad: Having Unnecessary useless code
final class FilterCell: UITableViewCell {

    @IBOutlet weak var labelTitle: UILabel!
    
    override func awakeFromNib() {
        super.awakeFromNib()
    }

    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)
    }
}
```

```swift
// Good: Cleaned version
final class FilterCell: UITableViewCell {

    @IBOutlet weak var labelTitle: UILabel!
}
```

<img width="619" alt="image" src="https://user-images.githubusercontent.com/41438361/152294156-4fd600c2-2971-4cf8-9396-8aef472c3ecd.png">


### 3. 가능한 let을 쓴다.

```swift
// Bad: Using var even we are not modifying it from other place
class ViewController: UIViewController {
  public var constantWidth = 100
  public var constantHeight = 50
}
```

```swift
// Good: Changed var to let because the variable is not going to change.
final class ViewController: UIViewController {
  private let constantWidth: CGFloat = 100
  private let constantHeight: CGFloat = 50
}
```

### 4. 가능한 클래스를 final로 만든다.

```swift
// Bad: No final keyword
class ViewController: UIViewController {
}
```

```swift
// Good: With final keyword
final class ViewController: UIViewController {
}
```

### 5. 프로젝트에서 public과 open을 사용하는 것을 지양하고 가능한 private을 사용한다.(libraries/frameworks 제외)

```swift
// Bad: Open class (frameworka/libraries are exceptions)
open class ViewController: UIViewController {
  open let constantWidth = 100
  public let constantHeight = 50
  var isUpdating: Bool = false
}
```

```swift
// Bad: Public class (frameworka/libraries are exceptions)
public class ViewController: UIViewController {
  let constantWidth = 100
  let constantHeight = 50
  var isUpdating: Bool = false
  @IBOutlet var textLabel: UILabel!

  @objc func gestureRecognized() {
  }
}
```

```swift
// Good: Without open/public (frameworka/libraries are exceptions)
final class ViewController: UIViewController {
  private let constantWidth: CGFloat = 100
  private let constantHeight: CGFloat = 50
  fileprivate var isUpdating: Bool = false
  private(set) var isDataAvailable: Bool = false
  @IBOutlet private var textLabel: UILabel!

  @objc private func gestureRecognized() {
  }
}
```

### 6. 만약 같은 파일에 있다면 가능한 extension들을 private이나 fileprivate로 설정한다.

```swift
// Bad: Avoid non private extension when they don't use outside of the file.
final class VideoRecorder {
}

extension VideoRecorder {

  func capturePhoto() {
  }
  
  func startRecording() {
  }
  
  func stopRecording() {
  }
}
```

```swift
// Good: A good example to make the extension private.
final class VideoRecorder {
}

private extension VideoRecorder {

  func capturePhoto() {
  }
  
  func startRecording() {
  }
  
  func stopRecording() {
  }
}
```

### 7. 타입을 명확히 표시한다.

컴파일러가 추론하게 하지 말자. 만약 타입을 명시하지 않으면 컴파일러는 시간을 들여 타입을 찾아야 한다. 타입을 명시해서 컴파일러가 작업 하는 시간을 줄이는 것이 좋다.

```swift
// Bad: Avoid the automatic type infer. It's an extra overhead for the compiler to determine the variable type
public class ViewController: UIViewController {
  let constantWidth = 100
  var isUpdating = false
  var isDataAvailable = false
  var userNames = [“Amit”, “Yogesh”, “Rohit”]
  var numbers = [1, 2, 3]
  var savedPaymentMethods = [SavedPaymentMethod]()
}
```

```swift
// Good: Always specify type of the variable to reduce compiler type infer work.
final class ViewController: UIViewController {
  private let constantWidth: CGFloat = 100
  fileprivate var isUpdating: Bool = false
  private(set) var isDataAvailable: Bool = false
  private var userNames: [String] = [“Amit”, “Yogesh”, “Rohit”]
  var numbers: [Int] = [1, 2, 3]
  private var savedPaymentMethods: [SavedPaymentMethod] = []
}
```

swift5에서 타입 추론과 관련해 빌드 속도는 많이 개선된 것으로 보인다.

<img width="587" alt="image" src="https://user-images.githubusercontent.com/41438361/152295746-8bd791ee-785f-420b-8b87-467fe0b390ff.png">


타입을 명시하지 않고 `.init`을 쓰는 것도 지양하자. 

```swift
// Bad: Avoid .init because it's an extra overhead for the compiler to determine the type when compiling
button.transform 	= .init(scaleX: 1.5, y: 1.5)
button.contentEdgeInsets = .init(top: 11, left: 32, bottom: 11, right: 32)
tableView.contentSize 	 = .init(width: 100, height: 500)
```

```swift
// Good: Using the Type init version
button.transform 	= CGAffineTransform(scaleX: 1.5, y: 1.5)
button.contentEdgeInsets = UIEdgeInsets(top: 11, left: 32, bottom: 11, right: 32)
tableView.contentSize 	 = CGSize(width: 100, height: 500)
```

또한 shorthand enum을 쓰는 것도 지양한다.

```swift
// Avoid shorthand if that particular line takes significant time to compile.
let action = UIAlertAction(title: "title", style: .default, handler: nil)
```

```swift
// Using full version of the UIAlertAction
let action = UIAlertAction(title: "title", style: UIAlertAction.Style.default, handler: nil)
```

Swift5에서는 아래와 같다.

```swift
func setCatType1() {
    catAddress = .init(location: "location", type: .longHair)
}

func setCatType2() {
    catAddress = CatAddress(location: "location", type: CatType.longHair)
}
```

<img width="618" alt="image" src="https://user-images.githubusercontent.com/41438361/152295991-e3a8b7d4-0705-4298-a6c3-2a68f8de9202.png">
<img width="620" alt="image" src="https://user-images.githubusercontent.com/41438361/152296013-f28aeae4-52cc-4994-8999-9ae21c8d43cc.png">


### 8. Objective-C 타입을 지양

```swift
// Bad: Using Objective-C types in Swift
public class ViewController: UIViewController {
    var dictionary = Dictionary<String, Any>()
    var nsDictionary = NSMutableDictionary()
    var array = Array<String: Any>()
    var nsArray = NSMutableArray()
    var anyObject: AnyObject?
}
```

```swift
// Good: Using native Swift types
final class ViewController: UIViewController {
    private var dictionary: [String: Any] = [:]
    private var array: [String] = []
    var any: Any?
}
```

Swift5에서 확인해보니 아래와 같았다.

```swift
private func check1() {
    var dictionary = Dictionary<String, Any>()
    var array = Array<String>()
    var anyObject: AnyObject?
}

private func check2() {
    var dictionary: [String: Any] = [:]
    var array: [String] = []
    var any: Any?
}
```

<img width="589" alt="image" src="https://user-images.githubusercontent.com/41438361/152296572-6914b92b-a77e-4e21-a752-4232c75b6e6c.png">
<img width="621" alt="image" src="https://user-images.githubusercontent.com/41438361/152296584-ffb36e89-b06c-48f2-a3bd-1f82262f639f.png">


### 9. 한 줄에 긴 연산을 쓰는 것 지양

만약 한 줄에 여러 연산이 있다면, 컴팡일러가 어떤 일이 발생하는 지 파악하기가 힘들다. 만약 코드를 여러 개의 간단한 문장으로 분리한다면 컴파일러가 이런 문장들을 정의하는데 도움이 될 것이고 컴파일 하는데도 시간이 더 적게 걸린다.

```swift
// Bad: Long calculation in single line
let widthHeight = max(min(min(self.bounds.width - 60, self.bounds.height - 100), 100), 45)
```

```swift
// Good: Rewritten in multiple lines
let proposedWidthHeight = min(self.bounds.width - 60, self.bounds.height - 100)
let allowedMaxWidth = min(proposedWidthHeight, 100)
let widthHeight = max(allowedMaxWidth, 45)
```

```swift
// Bad: Multiple calculation in single line
func totalSeconds() -> Int {
    return (hours*60*60) + (minutes * 60) + seconds
}
```

```swift
// Good: Rewritten in multiple lines
func totalSeconds() -> Int {
    let totalHours: Int = hours*3600
    let totalMinutes: Int = minutes * 60
    return totalHours + totalMinutes + seconds
}
```

Swift5에서도 확연하게 컴파일 시간에서 차이가 났다.

```swift
private func check1() {
    let widthHeight = max(min(min(view.bounds.width - 60, view.bounds.height - 100), 100), 45)
}

private func check2() {
    let proposedWidthHeight = min(view.bounds.width - 60, view.bounds.height - 100)
    let allowedMaxWidth = min(proposedWidthHeight, 100)
    let widthHeight = max(allowedMaxWidth, 45)
}
```

<img width="599" alt="image" src="https://user-images.githubusercontent.com/41438361/152297149-f6ba28ca-73b2-40df-bbe3-7c1bf63661ef.png">
<img width="593" alt="image" src="https://user-images.githubusercontent.com/41438361/152297170-bf3c690d-7bea-4054-895d-b17d1d31ee81.png">


### 10. ?? 연산자를 사용해 nil 판단하는 것을 지양

`if-else`을 사용하는 것 대신 `??`를 사용해서 `nil`을 판단하는 것은 컴파일하는 데 더 오랜 시간이 걸리게 한다.

```swift
// Bad: Doing all the calculations in a single line, which increases the compilation complexity.
return CGSize(width: size.width + (rightView?.bounds.width ?? 0) + (leftView?.bounds.width ?? 0) + 22, height: bounds.height)
```

```swift
// Best: Avoid the ?? operator and rewrite same thing with if let
var padding: CGFloat = 22
if let rightView = rightView {
    padding += rightView.bounds.width
}
 
if let leftView = leftView {
    padding += leftView.bounds.width
}
 
return CGSizeMake(size.width + padding, bounds.height)
```

```swift
// Good: Another solution is at least split the complex expression to multiple lines
let rightPadding: CGFloat = rightView?.bounds.width ?? 0
let leftPadding: CGFloat = leftView?.bounds.width ?? 0
return CGSize(width: size.width + rightPadding + leftPadding + 22, height: bounds.height)
```

### 11. lazy 변수들을 가능한 지양

`lazy` 변수들을 선언하는 것은 `lazy`하지 않는 것에 비해 더 오랜 컴파일 시간이 걸린다.

```swift
// Bad: Avoid lazy properties. Use it only on necessary situations.
private lazy var label: UILabel = {
    let label = UILabel()
    label.font = UIFont.systemFontOfSize(19)
    return label
}()
```

```swift
// Good: without lazy, and a seperate configuration function
private let label = UILabel()

//...
private func setup() {
  label.font = UIFont.systemFontOfSize(19)
}
```

### 12. 타입 캐스팅과 C 메서드를 지양

컴파일러는 타입 캐스팅하는데 시간이 오래 걸린다. 아래의 문법은 굉장히 간단해보이지만, 컴파일 하는데 오랜 시간이 걸린다.

```swift
// Using below typecasing functions increased the compilation time for that particular line.
let expiration 		= TimeInterval(expirationDuration)
let floatIndexValue 	= CGFloat(index)
let intWidth 		= Int(frame.size.width)
let doubleValue 	= Double(theString)
let participiantSquare  = sqrt(participantCount)
let ratioFloor  	= floor(ratio)
let ratioCeil	  	= ceil(ratio)
let roundValue 		= round(d * 0.66)
let minValue 		= min(size.width, size.height)
let maxValue 		= max(size.width, size.height)
```

## 사용하지 않는 것들 제거하기

사용되지 않는 코드들도 컴파일 되기 때문에 이를 없애는 것이 좋다.

### 1. 사용되지 않는 코드 제거하기

[Periphery app](https://github.com/peripheryapp/periphery)앱은 사용되지 않는 코드를 찾는 도구다. 거의 99% 정확하지만 그렇지 않은 경우도 있는 듯하다. 이를 사용해서 사용되지 않는 코드를 찾고 없애서 불필요한 컴파일러 작업을 없애는 것이 좋다.

![image](https://user-images.githubusercontent.com/41438361/150741851-b5305d6f-b8a2-491e-aece-31bdd618d7c1.png)

![image](https://user-images.githubusercontent.com/41438361/150741866-b71ff47c-abb4-4af4-9647-9be1769cb868.png)

[이 Swift Script](https://github.com/PaulTaykalo/swift-scripts/blob/master/unused.rb) 도 사용되지 않는 코드를 찾을 수 있다.

![image](https://user-images.githubusercontent.com/41438361/150741995-d32a65f8-5ccf-477c-8430-2b80d71f6a8e.png)


### 2. 사용되지 않는 asset/icon 삭제하기

에셋을 컴파일하는데도 시간이 걸린다. 불필요한 에셋을 지우는 것도 컴파일 시간을 향상시킬 수 있다. [FengNiao](https://github.com/onevcat/FengNiao)는 에셋에서 사용되지 않은 아이콘들을 찾을 수 있다.

![image](https://user-images.githubusercontent.com/41438361/150742154-ec13e5c9-350d-4c7f-a646-3b22f3959e7b.png)


### 3. 사용하지 않는 스토리보드 컨트롤러 삭제하기

스토리보드는 컴파일러 속도를 저하시키는 주요 범인이다. 단일 스토리보드에서 대부분 40-60%의 컴파일 시간을 차지한다.

이 시간을 줄일 수 있는 방법 중 하나는 사용되지 않는 컨트롤러들을 제거하거나 이를 별도의 스토리보드로 옮기고 타겟 멤법십을 해제해서 컴파일 되지 않게 설정하는 것이다.

![image](https://user-images.githubusercontent.com/41438361/150742400-78aedd09-8712-4f01-b9de-cbd9b42d1896.png)

## CocoaPods vs Carthage

요약하면 클린 빌드를 자주 수행하고 빌드 시간을 신경쓴다면 Carthage가 더 좋은 선택이다.

### CocoaPods drawback

CocoaPods는 third-party의 소스코드가 대부분 클린 빌드를 수행할 때마다 컴파일되기 때문에 더 긴 컴파일 시간을 초래한다. 일반적으로 이렇게 할 필요는 없지만 실제로 그렇게 되는 것이다.

### Carthage advantage

클린 빌드에 다시 빌드하지 않기 때문에 매번 dependency를 빌드하지 않아도 된다. 오직 dependency 리스트에 변화가 있을 때에만(새로운 프레임워크를 추가하거나, 프레임워크를 새 버전으로 업데이트 하거나) 외부 dependency들을 빌드하게 된다. 

## Build Time Analyzer

### 1. Build with Timing Summary

내장된 빌드 시간 요약은 컴파일 시간 요약에 대해 대략적으로 알 수 있게 해준다.

![image](https://user-images.githubusercontent.com/41438361/150743300-48093b14-a6ed-456a-b209-a943c79c422c.png)

![image](https://user-images.githubusercontent.com/41438361/150743317-75530a73-b115-4d6d-b7b2-c3c57c122c34.png)

# Bazel로 LINE의 iOS 앱 빌드 속도를 2배 빠르게!

## 의존성 관리 개선

iOS에서 인기있는 의존성 관리 도구에는 CocoaPods, Carthage 등이 있다. CocoaPods는 프로젝트를 클린 빌드할 때마다 모든 pod 라이브러리를 다시 빌드해야 해서 많은 시간이 소요되는 문제가 있었는데, Xcode와 잘 연동되며 외부 라이브러리 소스 코드를 내 소스처럼 보고 디버깅 할 수 있는 도구였다. Carthage는 Swift 의존성 관리 도구로, CocoaPods와는 달리 설정이 거의 없어서 의존성을 사전에 빌드만 해줄 뿐, 프로젝트에 통합 시키는 것은 개발자의 몫이다.

의존성 빌드는 크게 두 가지 방법으로 진행할 수 있다.

1. Carthage 를 통해 사전에 빌드된 artifact를 Github에서 다운로드 : 보안상 지양
2. 로컬에서 빌드 : 앱 성능을 고려해 모든 의존성을 정적 프레임워크로 빌드.

Carthage는 보이지 않는 곳에서 Xcode를 실행시켜 모든 의존성을 [fat 바이너리](https://ko.wikipedia.org/wiki/유니버설_바이너리)로 빌드하는데, 지원하는 모든 아키텍처에 대해 이 과정을 반복한다. Xcode의 아카이브 액션은 자체 설계에 따라 클린 빌드를 하도록 되어 있는데, 이 때문에 빌드에 오랜 시간이 걸렸다. 모든 사람이 동일한 코드를 반복적으로 빌드해야 하면 자원이 많이 낭비되는데, Carthage의 빌드 artifact를 빌드/기기 간 캐싱할 수 있는 방법이 있다.

## 빌드 캐싱 적용

빌드 캐싱에는 오픈 소스 툴인 [Rome](https://github.com/tmspzz/Rome)을 사용할 수 있다. Rome은 사용자의 로컬 디렉토리에 캐싱하거나 원격으로 AWS S3, Minio, Ceph에 캐싱하는 기능을 지원했다. 다만 캐시의 정합성을 검증할 방법이 없었는데, QA 테스트 및 릴리스 빌드가 캐시 포이즈닝(cache poisoning)과 같은 공격에 노출되는 것을 막기 위해 모든 것을 처음부터 재빌드하는 방법도 있다.

## 잔여 이슈

불필요하게 의존성 빌드를 반복하는 것은 피할 수 있었지만, 코드 자체는 빌드해야 한다. 클린 빌드를 하게 되면 거의 바뀌지 않는 빌드 아티팩트가 타깃에서 제거되어 버린다. 목표는 코드의 어느 부분이든 불필요한 재발드는 하지 않는 것으로, 이를 위해 코드 베이스를 모듈로 분리해서 빌드하고 캐싱하는 방법을 적용할 수 있다. Carthage는 아래의 이유로 적용하기 어려웠다.

* 로컬 타깃을 별도의 저장소로 분리하고, 변경이 될 때마다 버전을 관리하고, 원래 저장소에서 참조하는 버전을 업데이트 해야 한다.
* 핵심 프로젝트와 로컬 타깃이 사전에 빌드된 바이너리 형태로 존재해서 디버깅할 수 없다.

위와 같이 진행하면 작업 효율성이 매우 떨어진다. Carthage와 Rome으로 외부 의존성 문제는 어느 정도 해결했지만, 내부 개발 코드의 문제가 남아있는 것이다. 캐싱 방식은 확장성이 제한될 수 밖에 없다.

## Bazel 도입

[Bazel](https://bazel.build/)은 고급 캐싱 기능으로 잘 알려진 오픈 소스 도구 중 하나다. Bazel은 대규모 monorepo 빌드를 목적으로 개발되었고, 애플 플랫폼에서 중요하게 생각하는 헤더 맵과 Clang 모듈, 다중 언어(mixed-language) 다깃 등의 기능을 지원하지 않았다. 하지만 Bazel은 뛰어난 확장성을 가지고 있다. Bazel 자체틑 Java, C 계열 언어로 만들어졌지만 빌드 규칙을 작성하면 어떤 언어라도 빌드 가능하도록 확장시킬 수 있다.


# 빌드 관련 툴

앞서서 빌드 시간을 줄여야 하는 이유, 또 줄일 수 있는 여러 방법들을 대략적으로 훑어봤다. 이제 위에서 봤던 툴들을 실제로 설치해서 사용해보며 정리하려 한다.

## BuildTimeAnalyzer-for-Xcode

Github에 [BuildTimeAnalyzer=for-Xcode](https://github.com/RobertGummesson/BuildTimeAnalyzer-for-Xcode)라는 프로젝트가 있다.

가장 최신 release가 2019년에 나온 릴리즈라 지금 사용하기에는 구버전일 수 있다. 

### 1. 프로젝트 다운로드 받아 프로젝트 열기

![image](https://user-images.githubusercontent.com/41438361/150465864-cc5e0795-1aad-4f7d-87e5-380a2af6fbc9.png)

프로젝트를 다운받아 실행시켰다.

### 2. Product > Archive

Xcode 메뉴에서 Product > Archive를 클릭한다.

![image](https://user-images.githubusercontent.com/41438361/150465967-c4bd1788-2308-4f10-8716-1960d38c92c6.png)

그러면 아래와 같이 Archives 창이 뜨면서 BuildTimeAnalzyer를 확인할 수 있다. 나는 두 번 Archive해서 두 개가 있는데, 처음 archive를 했다면 BuildTimeAnalzyer가 하나만 있을 것이다.

참고로 archive는 단어에서부터 알 수 있듯이 지금까지의 패키지(.app과 관련된 다른 파일)를 모아둔 것이다. 앱 배포에 필요한 안드로이드의 .apk와 비슷한 .IPA 파일을 생성하는 것은 아니지만 .app(AppStore에 올리는데 필요한 symbol과 다른 정보들)을 포함한 디렉토리(.xcarchive)를 생성한다. 이 .xcarchive는 .ipa를 생성하는 시작점으로 사용된다.

### 3. Distribute App

오른쪽의 Distribute App 버튼을 누르고 원하는 저장위치에 저장한다. 그러면 아래와 같이 폴더와 함께 안에 BuildTimeAnalyzer앱이 있는 걸 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/150466212-79d2f31e-e40d-4c31-95d4-d06cad3d76e3.png)

![image](https://user-images.githubusercontent.com/41438361/150466244-a2510962-7255-4c06-9f6b-ec3e2a8c8595.png)

### 4. 앱 실행시키기

![image](https://user-images.githubusercontent.com/41438361/150466334-acdfe472-b8be-46eb-9531-eb0f081616b6.png)

앱을 실행시키면 위와 같이 instruction이 뜬다. 빌드 시간을 분석하고 싶은 프로젝트에서 instruction에 나온대로 하고, 프로젝트를 선택하는 부분에서 해당 프로젝트를 선택하면 아래와 같이 빌드 시간을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/150466507-af4bbe8b-88c8-4c4f-ba54-039277ce1325.png)

<kbd>Command</kbd>+<kbd>Shift</kbd>+<kbd>K</kbd> 를 눌러 Clean build를 하고 툴로 다시 빌드 시간에 대한 분석을 확인하려고 하면 당연하게도 빌드 로그를 분석할 수 없다. 

* 특징 : 전체 빌드 시간, 함수별로 걸린 컴파일 시간, 컴파일러에서 함수가 얼마나 반복되는지, 또 각 항목을 클릭했을 때 해당 부분으로 이동할 수 있다.

## Sitrep

### 설치

[Github](https://github.com/twostraws/Sitrep)에 여러 경로를 통한 설치 방법이 나와있는데, 나는 brew로 설치했다.

터미널을 열어 아래의 커맨드를 입력했다.

```
brew install twostraws/brew/sitrep
```

설치가 잘 됐다.

![image](https://user-images.githubusercontent.com/41438361/150745844-2af3d8f4-9390-4073-a589-ab37846d8b51.png)


### Command line flags

플래그 없이 커맨드 라인으로 실행시키면, Sitrep은 자동으로 현재의 디렉토리를 스캔해서 찾는 내용들을 텍스트로 출력한다. 이에 옵션을 주려면 커맨드 플래그를 이용하면 된다.

* `-c`는 .sitrep.yml 설정 파일이 있다면 해당 파일의 경로를 내가 명시할 수 있게 해준다.
* `-f`는 출력 포맷을 설정한다. 예를 들어 `-f json`은 JSON 형식으로 출력하는 걸 가능하게 한다. 디폴트는 텍스트이고, `-f text`와 같다.
* `-i`는 만약 실제 스캐닝이 요청되었다면 Sitrep이 사용했을 설정들을 보여줘서 디버깅 정보를 출력하게 한다.
* `-p`는 Sitrep이 스캔할 경로를 설정한다. 디폴트는 현재 위치다.
* `-h`는 커맨드 라인에 대한 정보를 출력한다.

### Configuration

스캔하고 싶은 디렉토리 안에 .sitrep.yml 파일을 생성해서 Sitrep의 행동을 커스터마이징 할 수 있다. 

예를 들어 .build 디렉토리와 테스트를 제외하고 싶다면 .sitrep.yml 파일을 아래와 같이 생성해야 한다.

```yml
excluded:
  - .build
  - Tests
```

### 사용해보기

나는 위에서 brew로 설치를 받았기 때문에 바로 `sitrep` 커맨드를 쓸 수 있다. 분석 결과를 보고 싶은 프로젝트 홈 디렉토리에서 `sitrep`을 치니 아래와 같이 나왔다.

![image](https://user-images.githubusercontent.com/41438361/150818877-cc43e1ac-60d4-4cc3-a527-a009bb34f4c0.png)

* Overview : 스캔된 파일 수, 구조체, 클래스, 열거형, 프로토콜, extension의 수
* Sizes : 전체 줄 수, 코드로만 이루어진 줄 수, 가장 긴 파일, 가장 긴 타입. 참고로 Pods 쪽에 있는 것까지 다 스캔하고 있기 떄문에 Pods를 제외하고 싶으면 .sitrep.yml 파일을 만들어 프로젝트 파일에 넣으면 된다.
* Sturcture : import를 얼마나 했는지, 프로젝트에 있는 ViewController, View의 개수

* 특징 : 시간에 따라 변화하는 프로젝트의 개요를 확인하기에는 좋은 것 같으나 빌드 시간을 줄이기 위한 도구로는 적합하지 않은 것 같다.

## XCLogParser

[XCLogParser](https://github.com/MobileNativeFoundation/XCLogParser)는 로그 컨텐츠를 분석하기 위해 다양한 종류의 리포트를 생성한다. XCLogParser는 프로젝트에 있는 모듈마다 빌드 시간, 경고, 에러, 그리고 유닛 테스트 결과를 제공한다.

XCLogParser는 3가지 작업을 할 수 있다.

1. `xcactivitylog` 내용을 `JSON` 문서로 변환한다.
2. `xcactivitylog`의 내용을 다양한 형식의 리포트로 변환한ㄷ.(json, flatjson, summaryJson, chromeTracer, issues, html)
3. `LogStoreManifest.plist` 파일의 내용을 `JSON` 문서로 변환한다.

XCLogParser를 통해 할 수 있는 일들은 아래와 같다.

* 빌드 시간을 이해하고 자세하게 추적한다.
* 유닛 테스트 결과, 경고, 에러를 확인한다.
* Xcode 외부 사용을 위한 다른 개발자 도구를 빌드한다.

### 설치

Homebrew를 이용해서 설치했다.

`$ brew install xclogparser`

### Xcode 통합

post-scheme build action으로 `xcactivitylog` 파일들을 자동으로 파싱할 수 있다. 이 방법으로 빌드가 끝나자마자 가장 마지막의 빌드 로그가 파싱될 것이다. 이를 하려면 프로젝트의 scheme editor를 연다음 왼쪽 패널의 "Build" 부분을 클릭한다. 그러면 "Post-action" run script를 새로 추가할 수 있고 요구되는 파라미터와 함꼐 `xclogparser` 실행파일을 실행시킬 수 있다.

```
xclogparser parse --project MyApp --reporter html
```


근데 문제는 Xcode postbuild에 이걸 넣어서 실행시키려면 sclogparser executable이 있어야 한다고 했는데, 여러 문제에 부딪혀서 일단은 터미널에서 수동으로 입력해줬다.

![image](https://user-images.githubusercontent.com/41438361/150923475-6b00843b-28e5-4dea-82b9-99f87209983e.png)

나는 아래와 같이 이용했다.

```
/usr/local/bin/xclogparser parse --project UICollectionViewVIsibleCells --reporter html --rootOutput ~/Desktop/build/reports
open ~/Desktop/build/reports
```

그러면 프로젝트 내에 build > reports 라는 폴더가 생기는데 여기에 리포트가 생성된다. index.html을 눌러 확인해주면 된다.

* 특징 : 빌드 시간, 에러, 경고, 타임라인(타겟별로 걸리는 빌드 시간), 가장 느린 타겟, 가장 컴파일 오래 걸리는 파일 등등을 굉장히 자세히 확인할 수 있다.

## Carthage

[Carthage](https://github.com/Carthage/Carthage) 는 Cocoa application에 프레임워크를 가장 간단하게 추가하도록 하는 것이 목표다. Carthage는 dependency들을 빌드하고 바이너리 프레임워크를 제공해주지만, 개발자는 프로젝트 구조와 셋업을 모두 통제할 수 있다. Carthage는 자동으로 프로젝트 파일을 수정하거나 빌드 세팅을 건들지 않는다.



## Periphery

![image](https://user-images.githubusercontent.com/41438361/151111796-813bd557-e65e-484d-b8f5-fa3410c54e3f.png)

[Periphery](https://github.com/peripheryapp/periphery)는 Swift 프로젝트에서 사용되지 않는 코드를 찾아내는 도구다.
Periphery의 목적은 사용되지 않은 declaration들을 찾는 것이다. Declaration에는 class, struct, protocol, function, property, constructor, enum, typealias, associatedtype 등이 있다.

### Installation

Homebrew로 설치했다.

```
brew install peripheryapp/periphery/periphery
```

Swift Package Manager로 설치할 수 있는데, 일단 그냥 homebrew로 설치했다.

### How To Use

해당 프로젝트 폴더로 가서 아래의 커맨드를 터미널에 입력한다.

```
periphery scan --setup
```

그러면 아래와 같이 몇 개의 질문에 답을 하고 Periphery가 프로젝트를 스캔해서 결과를 보여줄 것이다.

![image](https://user-images.githubusercontent.com/41438361/151112922-9db9cc23-48e9-41ba-886a-4a180cdc2661.png)


## FengNiao

[FengNiao](https://github.com/onevcat/FengNiao)는 Xcode 프로젝트에서 사용되지 않는 이미지 리소스 파일을 삭제하기 위한 커맨드 라이 툴이다.


* 참고
* https://www.martinfowler.com/articles/continuousIntegration.html
* http://nangpuni.net/?p=957
* https://medium.com/@leandromperez/analyzing-and-improving-build-times-in-ios-5e2b77ef408e
* https://betterprogramming.pub/improve-xcode-compile-and-run-time-8b8f812c17f8
* https://engineering.linecorp.com/ko/blog/improving-build-performance-line-ios-bazel/
