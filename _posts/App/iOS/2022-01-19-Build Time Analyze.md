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


# BuildTimeAnalyzer-for-Xcode

Github에 [BuildTimeAnalyzer=for-Xcode](https://github.com/RobertGummesson/BuildTimeAnalyzer-for-Xcode)라는 프로젝트가 있다.

가장 최신 release가 2019년에 나온 릴리즈라 지금 사용하기에는 구버전일 수 있다.






* 참고
* https://www.martinfowler.com/articles/continuousIntegration.html
* http://nangpuni.net/?p=957
* 
