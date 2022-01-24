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

## CGFloat를 CGFloat으로 캐스팅하기

이미 CGFloat인 값들이었고, 몇 괄호들은 불필요했다. 이를 해결하니 빌드 시간이 99.9% 감소했다.

```swift
// Build time: 3431.7ms
return CGFloat(M_PI) * (CGFloat((hour + hourDelta + CGFloat(minute + minuteDelta) / 60) * 5) - 15) * unit / 180

// Build time: 3.0ms
return CGFloat(M_PI) * ((hour + hourDelta + (minute + minuteDelta) / 60) * 5 - 15) * unit / 180
```

## Round()

이 예시는 좀 이상한데, 아래의 예제들은 local, instance 변수들을 섞은 것이다. 문제는 round 자체가 아니라 메서드 내의 코드의 조합일 가능성이 크다. rounding을 제거하니 97.6% 빨라졌다.

```swift
// Build time: 1433.7ms
let expansion = a — b — c + round(d * 0.66) + e
// Build time: 34.7ms
let expansion = a — b — c + d * 0.66 + e
```

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

## Swift 3.0에서의 빌드 시간

Xcode 8.0에서, Xcode의 플러그인 시대가 끝나고 extension의 시대가 시작됐다. Extension에 한계가 있기 때문에 위의 plug in을 별개의 다른 앱으로 작업중이라고 한다. 

# BuildTimeAnalyzer-for-Xcode

Github에 [BuildTimeAnalyzer=for-Xcode](https://github.com/RobertGummesson/BuildTimeAnalyzer-for-Xcode)라는 프로젝트가 있다.

가장 최신 release가 2019년에 나온 릴리즈라 지금 사용하기에는 구버전일 수 있다. 

## 1. 프로젝트 다운로드 받아 프로젝트 열기

![image](https://user-images.githubusercontent.com/41438361/150465864-cc5e0795-1aad-4f7d-87e5-380a2af6fbc9.png)

프로젝트를 다운받아 실행시켰다.

## 2. Product > Archive

Xcode 메뉴에서 Product > Archive를 클릭한다.

![image](https://user-images.githubusercontent.com/41438361/150465967-c4bd1788-2308-4f10-8716-1960d38c92c6.png)

그러면 아래와 같이 Archives 창이 뜨면서 BuildTimeAnalzyer를 확인할 수 있다. 나는 두 번 Archive해서 두 개가 있는데, 처음 archive를 했다면 BuildTimeAnalzyer가 하나만 있을 것이다.

참고로 archive는 단어에서부터 알 수 있듯이 지금까지의 패키지(.app과 관련된 다른 파일)를 모아둔 것이다. 앱 배포에 필요한 안드로이드의 .apk와 비슷한 .IPA 파일을 생성하는 것은 아니지만 .app(AppStore에 올리는데 필요한 symbol과 다른 정보들)을 포함한 디렉토리(.xcarchive)를 생성한다. 이 .xcarchive는 .ipa를 생성하는 시작점으로 사용된다.

## 3. Distribute App

오른쪽의 Distribute App 버튼을 누르고 원하는 저장위치에 저장한다. 그러면 아래와 같이 폴더와 함께 안에 BuildTimeAnalyzer앱이 있는 걸 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/150466212-79d2f31e-e40d-4c31-95d4-d06cad3d76e3.png)

![image](https://user-images.githubusercontent.com/41438361/150466244-a2510962-7255-4c06-9f6b-ec3e2a8c8595.png)

## 4. 앱 실행시키기

![image](https://user-images.githubusercontent.com/41438361/150466334-acdfe472-b8be-46eb-9531-eb0f081616b6.png)

앱을 실행시키면 위와 같이 instruction이 뜬다. 빌드 시간을 분석하고 싶은 프로젝트에서 instruction에 나온대로 하고, 프로젝트를 선택하는 부분에서 해당 프로젝트를 선택하면 아래와 같이 빌드 시간을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/150466507-af4bbe8b-88c8-4c4f-ba54-039277ce1325.png)

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



* 참고
* https://www.martinfowler.com/articles/continuousIntegration.html
* http://nangpuni.net/?p=957
* https://medium.com/@leandromperez/analyzing-and-improving-build-times-in-ios-5e2b77ef408e
