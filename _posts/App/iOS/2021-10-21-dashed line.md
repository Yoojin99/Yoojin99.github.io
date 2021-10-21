---  
layout: post  
title: "[iOS] - 점선, dashed line 그리기"  
subtitle: ""  
categories: app
tags: app-ios, uibezierpath
comments: true  
header-img: 

---  
  
> `UIBezierPath를 이용해서 점선을 그리는 법을 알아보자.`  

---

# UIBezierPath

[애플의 공식문서](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CocoaDrawingGuide/Paths/Paths.html)를 보니 점선을 그리려면 `NSBezierPath`라는 걸 이용해야 하나보다.

![image](https://user-images.githubusercontent.com/41438361/138294433-93fb611b-1aa2-4054-b2bc-57f170f4950f.png)

그렇다면 이 Path라는 게 뭘까?

**Path는 선, 호, 커브와 같은 형태의 모양을 생성하기 위해 사용되는 point들의 컬렉션이다.** 즉 특정 모양을 그릴 때 그리는 선을 구성하는 점들의 집합이라는 뜻 같다. 이를 이용해서 원, 사각형, 폴리곤, 그리고 복잡하고 구부러진 모양들을 그릴 수 있는 것이다. 얘네들이 point들로 구성되어 있기 때문에 path는 가볍고 빠르며, 정확도가 퀄리티를 떨어트리지 않고 여러 곳에서 확장되어 사용될 수 있다고 한다.

방금 위에서 잠깐 봤던 `NSBezierPath` 클래스도 path를 생성하고 관리하기 위한 메인 인터페이스를 제공하는 클래스다. 이 클래스의 객체는 path와 관련해서, path의 외형에 영향을 주는 특징들과 path를 정의하는 point들을 포함한 정보를 캡슐화한다.

그렇다면 Bezier는 무엇을 말하는 걸까? [여기에](https://blog.coderifleman.com/2016/12/30/bezier-curves/) 아주 자세하게 설명이 되어 있어서, 여기의 내용을 참고하면 될 것 같다.
간단히 요약하자면 시작점들과 끝점들이 있고, 이 두 점을 이은 직선을 일정한 속도로 이동하는 또 다른 점들이 있다. 이런 시작점들과 끝점들이 많아질 수록 당연히 이 두 점 사이들을 이동하는 점들이 많아지고, 이 점들을 또 이은 직선을 일정한 속도로 움직이는 점을 만들 수 있게 된다. 이 점이 이동하는 곡선이 2차, 3차, 4차,,,, 베지에 곡선이 되는 것이다. 즉 그냥 점들을 가지고 곡선을 그린 것을 베지에 곡선이라고 생각하면 된다.

그러면 `NSBezierPath`와 `UIBezierPath`는 베이저 곡선을 path를 이용해서 그리는 무엇으로 유추할 수 있다. 공식 문서에서는 아래와 같이 설명한다.

![image](https://user-images.githubusercontent.com/41438361/138294472-6352cb14-8b46-4e55-900e-25ce12770d78.png)

커스텀 뷰에서 렌더링 할 수 있는 곧거나 굽은 선들로 구성된 path 라고 한다. 참고로 iOS 3.2 이상에서 사용 가능하므로 걱정 없이 써도 될 것 같다.

이 클래스를 이용해서 내 path의 형상(geometry)를 정의한다. Path는 위에서도 언급했듯이 사각형, 타원, 호와 같은 간단한 모양에서부터 직선과 곡선을 섞어서 복잡한 폴리곤을 만들 수도 있다. 모양을 정의한 후에, `UIBezierPath` 를 이용해서 현재 드로잉 컨텍스트에서 path를 렌더링하기 위한 추가적인 메서드를 사용할 수 있다.

`UIBezierPath`의 객체는 렌더링 할 때 path를 나타내는 특징들과 형상을 섞는다. 이 geometry와 attribute는 개별적으로 설정할 수 있다. 이를 이용해서 객체를 구성하고 나서 컨텍스트에 그리라고 할 수 있다. 생성, 구성, 렌더링 프로세스가 다 독립적인 단계이기 때문에, 베이저 path 객체는 코드에서 재사용되기 쉽다. 같은 객체를 사용해서 같은 모양을 여러 번 렌더링 할 수 있는데, 렌더링 할 때마다 옵션이 바뀔 수 있다.

Path의 현재 포인트를 조작해서 path의 형상을 설정할 수 있다. 새로운 빈 path 객체를 만들 때, current point는 정의되어 있지 안기 때문에 명시적으로 지정해줘야 한다. Segment를 그리지 않고 current point를 움직이려면 `move(to:)` 메서드를 사용하면 된다. 다른 메서드들은 path에 선이나 곡선을 추가하는 메서드들이다. 새로운 segment들을 추가하는 메서드들은 내가 새로 추가한 선이 current point에서 시작하고, 내가 명시한 점에서 끝난다는 것을 가정한다. segment를 추가한 후에, 새로운 segment의 끝 점은 current point가 된다.

간단한 베지에 path 객체는 open/closed 하위 path들을 가질 수 있다. 그리고 각 하위 path는 연결된 path segment들로 구성된다. `close()` 메서드를 호출하면 current point에서 하위 path의 첫 point로 직선을 그어서 하위 path를 close한다. `move(to:)` 메서드를 호출해서 closing하지 않고 현재의 하위 path를 끝내고 다음 하위 path의 시작점을 설정한다. 베지에 path의 하위 path들은 같은 그리기 attribute를 공유하고 그룹으로 관리된다. 다른 attribute를 가진 subpath를 그리려면, 각 하위 path를 별개의 UIBezierPath 객체에 넣어야 한다.

베지에 path의 형상과 attribute를 구성한 후에, `stroke()`와 `fill()` 메서드를 사용해서 path를 현재 그래픽 context에 그릴 수 있다. `stroke()` 메서드는 현재의 stoke 색깔과 베지에 Path 객체의 attribute를 사용해서 outline을 추적한다. 비슷하게, `fill()` 메서드는 현재 fill color를 사용해서 path에 의해 닫힌 영역을 채운다.

베지에 path 객체를 이용해서 도형을 그리는 것에 추가로, 새로운 clipping region을 정의하는데 사용할 수 있다. `addClip()` 메서드는 path 객체에 의해 나타난 도형과 그래픽 컨텍스트의 현재 clipping region을 교차시킨다. Drawing을 할 때, 새로운 겨비는 부분 아래에 있는 컨텐츠만 실제로 그래픽 컨텍스트에 렌더링 된다.

베지에 path를 사용해서 도형을 그리는 순서는 아래와 같다.

1. Path 객체를 생성한다.
2. `UIBezierPath` 객체에 attribute들을 설정한다. 단순히 그어진 path일 경우 `lineWidth`이나 `lineJoinStyle` 프로퍼티를 설정할 수 있고, 채워진 Path에는 `usesEvenOddFillRule` 프로퍼티를 설정할 수 있다.
3. `moveToPoint:` 메서드를 사용해서 초기 segment의 시작 포인트를 설정한다.
4. 선과 굽은 segment를 추가해서 하위 path를 정의한다.
5. 추가로 마지막 segment의 끝에서 처음으로 직선을 그리는 `closePath`를 호출해서 하위 path를 닫을 수도 있다.
6. 또한 3, 4, 5번째 단계를 반복해서 추가적인 하위 path들을 정의할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/138294514-39aa860e-d566-4bd5-a080-d8bd11d6070e.png)

위의 순서대로 선을 그려보았다. 3번에서 시작점을 설정하고, 4번에서 이 시작점에서 200, 40 위치로 선을 긋는 것 같았다.

애플의 공식문서에 펜타콘 모양을 그리는 예제가 있길래 이를 적용해봤더니 아래와 같이 나왔다.

![image](https://user-images.githubusercontent.com/41438361/138294543-9fb45923-fdc3-41f8-b130-1c8a845f24a3.png)

아직 `closePath`를 하지 않아서 도형이 닫히지 않은 모습이다. closePath를 하게 되면 아래와 같이 도형이 만들어진다.

![image](https://user-images.githubusercontent.com/41438361/138294572-f764daff-b464-4764-ba99-8a3225a5bc7b.png)

이를 UIViewController에서 출력해보도록 코드를 작성했다.

``` swift
import Foundation
import UIKit

class TestView: UIView {
    override func draw(_ rect: CGRect) {
        let path = UIBezierPath()

        path.move(to: CGPoint(x: 0, y: 0))
        path.addLine(to: CGPoint(x: 100, y: 200))

        path.stroke()
    }
}

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        view.backgroundColor = .white

        let test = TestView(frame: .zero)
        test.backgroundColor = .systemBlue
        test.translatesAutoresizingMaskIntoConstraints = false

        view.addSubview(test)

        NSLayoutConstraint.activate([
            test.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor),
            test.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            test.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            test.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }
}
```

TestView에서 선을 긋고, `stroke()` 메서드를 사용해서 path를 그려줬다. 그랬더니 결과는 아래와 같이 나왔다.

![image](https://user-images.githubusercontent.com/41438361/138294602-bf0f0f21-0cfa-4f87-98f4-4ee634c9bd97.png)

이번에는 testView의 autoylayout 설정을 바꿔서 다시 실행했다.

``` swift
test.centerXAnchor.constraint(equalTo: view.centerXAnchor),
test.centerYAnchor.constraint(equalTo: view.centerYAnchor),
test.widthAnchor.constraint(equalToConstant: 300),
test.heightAnchor.constraint(equalToConstant: 300),
```

설정을 바꿨더니 기준 시작점의 위치에 따라 선이 시작하는 지점도 달라진 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/138294621-e76382c0-7b88-4f21-8a05-4180542c6251.png)

이를 이용해서 구현하고자 하는 화면에 있는 dashed line을 만들어보겠다. 우선 코드를 아래와 같이 수정해 선이 수직이고, 뷰의 autolayout 설정에서 높이와 같은 길이를 갖게 했다.

``` swift
class TestView: UIView {

    override init(frame: CGRect) {
        super.init(frame: .zero)
        backgroundColor = .clear
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }

    override func draw(_ rect: CGRect) {
        let path = UIBezierPath()

        path.move(to: CGPoint(x: 0, y: 0))
        path.addLine(to: CGPoint(x: 0, y: frame.maxY))

        path.lineWidth = 4

        UIColor(red: 51/255, green: 140/255, blue: 1, alpha: 1.0).setStroke()

        path.stroke()
    }
}
```

![image](https://user-images.githubusercontent.com/41438361/138294652-6152bfc3-ae1d-4a8a-bcdb-cae8635cc78b.png)

이제 점선을 만들어줄 건데, 이를 위해서는 `setLineDash` 메서드를 사용해야 한다.

![image](https://user-images.githubusercontent.com/41438361/138294670-c967231c-76ae-4f1a-b53f-96336b772048.png)

* pattern : line segment의 길이와 패턴 안의 갭을 포함하고 있는 C 스타일 실수 배열이다. 배열 안의 값은 첫 번째 값부터 첫 번째 line segment의 길이, 그 다음은 첫 번째 갭의 길이, 다음은 seconde line segment의 길이, 그 다음은 두 번째 갭의 길이, 이렇게 반복된다.
* count : `pattern`안의 값들의 개수다.
* phase : 패턴을 그리기 시작할 때의 offset으로, 예를 들어 패턴이 5-2-3-2이고 phase가 6이면 첫 번째 갭의 중앙에서 그리기가 시작될 것이다.

``` swift
override func draw(_ rect: CGRect) {
        let path = UIBezierPath()

        path.move(to: CGPoint(x: 0, y: 0))
        path.addLine(to: CGPoint(x: 0, y: frame.maxY))

        path.lineWidth = 2

        path.setLineDash([6], count: 1, phase: 0)

        UIColor(red: 51/255, green: 140/255, blue: 1, alpha: 1.0).setStroke()

        path.stroke()
    }
```

라인의 두께를 수정했고, `setLineDash` 메서드를 사용해서 dashed line을 만들어줬다. 나 같은 경우는 선이 그어진 부분과 그렇지 않은 부분의 길이가 같은 선을 만들고 싶어 위와 같이 설정했다.

![image](https://user-images.githubusercontent.com/41438361/138294693-4a10cc9b-3943-4082-8cbf-4a8416893c15.png)

이제 점선을 그릴 수 있게 되었다.

* 참조
    * [https://developer.apple.com/documentation/uikit/uibezierpath](https://developer.apple.com/documentation/uikit/uibezierpath)
    * [https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CocoaDrawingGuide/Paths/Paths.html](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CocoaDrawingGuide/Paths/Paths.html)
    * [https://blog.coderifleman.com/2016/12/30/bezier-curves/](https://blog.coderifleman.com/2016/12/30/bezier-curves/)
    * [https://zeddios.tistory.com/814](https://zeddios.tistory.com/814)
    * [https://developer.apple.com/library/archive/documentation/2DDrawing/Conceptual/DrawingPrintingiOS/BezierPaths/BezierPaths.html#//apple\_ref/doc/uid/TP40010156-CH11-SW2](https://developer.apple.com/library/archive/documentation/2DDrawing/Conceptual/DrawingPrintingiOS/BezierPaths/BezierPaths.html#//apple_ref/doc/uid/TP40010156-CH11-SW2)
    * [https://developer.apple.com/documentation/uikit/uibezierpath/1624373-setlinedash](https://developer.apple.com/documentation/uikit/uibezierpath/1624373-setlinedash)
