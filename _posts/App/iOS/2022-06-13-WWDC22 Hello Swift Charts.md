---  
layout: post  
title: "[iOS] - WWDC22 Hello Swift Charts"  
subtitle: ""  
categories: app
tags: app-ios wwdc
comments: true  
header-img: 

---  

[Hello Swift Charts](https://developer.apple.com/videos/play/wwdc2022/10136/)
[Documentation - Creating a chart using Swift Charts](https://developer.apple.com/documentation/charts/creating-a-chart-using-swift-charts)

# Swift Charts

* 애플이 디자인한 차트를 사용할 수 있는 프레임워크
* SwiftUI와 같은 문법 사용

Swift Chart에서는 조합을 통해 차트를 생성한다.

<img width="876" alt="image" src="https://user-images.githubusercontent.com/41438361/173473220-dfe7a28f-85e9-4978-b496-47509b20c59e.png">

Bar 차트에서 바와 같은 시각적인 요소들을 **mark**라고 한다.

## Implement

<img width="1577" alt="image" src="https://user-images.githubusercontent.com/41438361/173473431-963800bf-042c-4b33-b886-c1ac2ab7f1bd.png">

* `value(설명, 실제 값)`

Swift Chart는 자동으로 바를 생성해서 우측 preview에 보여준다.

<img width="1562" alt="image" src="https://user-images.githubusercontent.com/41438361/173473658-bc6af058-e7fd-400e-9a70-38f73f68d86f.png">

바로 나타내고 싶은 요소들을 배열로 만들고, `forEach` 문 내에서 반복해서 요소들을 바 그래프에 표시할 수 있다.

<img width="592" alt="image" src="https://user-images.githubusercontent.com/41438361/173473772-1446098e-dca3-4335-8fde-a00dc9e1eec6.png">

만약 `ForEach`가 차트의 유일한 컨텐츠라면 데이터를 차트 initializer에 바로 넣을 수 있다.

<img width="1571" alt="image" src="https://user-images.githubusercontent.com/41438361/173473913-2bf102a3-e448-4229-aa63-8360acf7f13a.png">

x, y에 있는 값을 바꾸면 Swift Chart가 자동으로 축을 바꿔주고, 차트를 보여줄 더 나은 스타일을 선택해서 보여준다.

차트를 보려면 볼 수 있어야 한다.

* Swift Chart는 VoiceOver를 지원해서 차트의 내용을 음성으로 들을 수 있다.
* Audio Graphs 기능도 지원한다.

<img width="1531" alt="image" src="https://user-images.githubusercontent.com/41438361/173474779-74ff07d9-52ab-44b1-ab59-e56c966a79a1.png">

Swift Chart는 SwiftUI 애니메이션으로 동작하기 때문에 애니메이션을 적용해서 차트가 변하는 것을 보여줄 수 있다.

<img width="1544" alt="image" src="https://user-images.githubusercontent.com/41438361/173475072-582be8e3-da2d-4df4-905a-ca1f22eddff0.png">

`Chart`를 forEach문과 같이 사용해서 데이터를 표시. `forgroundStyle`로 색을 특정 값에 따라 다르게 표시되도록 지정할 수 있다.

<img width="351" alt="image" src="https://user-images.githubusercontent.com/41438361/173475268-c95301cb-8ee0-4d90-96d6-81a5d18f5e35.png"><img width="348" alt="image" src="https://user-images.githubusercontent.com/41438361/173475307-def91c5c-44f5-49b6-a429-fcc90a2b325d.png">

차트 디자인을 쉽게 바꿀 수 있다.

* `PointMark` : 점으로 표시
* `LineMark` : 선으로 표시

<img width="690" alt="image" src="https://user-images.githubusercontent.com/41438361/173475491-5f24e977-c7fc-4eb5-bec1-d541f483c7f0.png"><img width="347" alt="image" src="https://user-images.githubusercontent.com/41438361/173475520-1cbf051a-f070-4d11-8a93-5a8b2fe709bb.png">

여러 스타일을 혼합할 수 있고, 심볼을 적용할 수도 있다.

# Mark + Mark Property

<img width="514" alt="image" src="https://user-images.githubusercontent.com/41438361/173476039-ee7920df-b8f8-4e99-95f0-bc71ad5bab33.png"><img width="1654" alt="image" src="https://user-images.githubusercontent.com/41438361/173476136-da2f83ac-2984-482c-b3c3-481f1ae89bc1.png">

Swift Charts에서는 mark 와 mark property들을 조합해서 차트를 구성한다.

<img width="1512" alt="image" src="https://user-images.githubusercontent.com/41438361/173476730-8f7138e1-809e-40fd-b7ff-c6443b1df282.png">

다크 모드, 다양한 화면 크기, 다이나믹 타입, voiceOver, Audio Graph, High-Constrast, multiplatform이 지원된다.

=> 더 자세한 내용은 Swift Chars: Raise the bar 세션에서 확인가능
