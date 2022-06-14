---  
layout: post  
title: "[iOS] - WWDC22 Hello Swift Charts"  
subtitle: ""  
categories: app
tags: app-ios wwdc
comments: true  
header-img: 

---  

[Hello Swift Charts](https://developer.apple.com/videos/play/wwdc2022/10137/?time=1290)

# Swift Charts

* Data Visualization : 앱에 더 많은 정보를 포함시킬 수 있다.
* Communicate Data : 데이터를 완전히 반영할 수 있다.
* Accessible

<img width="1637" alt="image" src="https://user-images.githubusercontent.com/41438361/173496122-6424ab22-2d08-48e9-9c72-a370faf2f4aa.png">

* 선언형 문법으로 차트를 만들 수 있다.
* 커스터마이징 가능

## Marks and composition

Swift Chart는 블럭들을 조합해서 다양한 차트를 만들 수 있다. 

<img width="1023" alt="image" src="https://user-images.githubusercontent.com/41438361/173496397-5443f094-cc9e-426b-a16e-24d5af1c5970.png">

**Mark** : 데이터를 나타내는 시각적인 요소

<img width="1585" alt="image" src="https://user-images.githubusercontent.com/41438361/173496582-5388b28e-387c-494d-8d0e-465d72cd5d57.png">

실제로 사용할 때는 차트에 제공할 구조체나 튜플 배열을 전달하고, `ForEach`문을 써서 각 요소마다 mark를 생성한다.

<img width="707" alt="image" src="https://user-images.githubusercontent.com/41438361/173496752-f4d50c40-fcf4-4d11-8c06-6e343bde5054.png">

`ForEach`가 차트의 유일한 컨텐츠라면 위와 같이 차트에 바로 데이터를 전달할 수 있다.

### modifier

<img width="1613" alt="image" src="https://user-images.githubusercontent.com/41438361/173496851-842f0379-470d-44ca-8c16-d3e2cf559133.png">

Mark에 많은 SwiftUI modifier를 적용할 수 있다.

* `.foregroundStyle` : mark 색깔
* `.accessibilityLabel`, `.accessibilityValue` : VoiceOver를 위해 커스터마이징 할 수 있는 modifier

<img width="1601" alt="image" src="https://user-images.githubusercontent.com/41438361/173497720-9998c5f3-912a-4de3-a17d-3807727ecd2d.png">

* `.symbol` : mark에 기호를 붙여서 표시
* `interpolationMethod` : 선 그래프가 부드럽게 꺾이게 할 수 있다.

<img width="517" alt="image" src="https://user-images.githubusercontent.com/41438361/173497803-54214103-da9d-4123-8de4-3b82ffe4f4aa.png">
<img width="1623" alt="image" src="https://user-images.githubusercontent.com/41438361/173497838-0eb78646-dcd7-4126-9346-7f6d7106b0ad.png">

* `.position` : 바 차트에서 바가 표시되는 형식을 바꾼다.

<img width="1595" alt="image" src="https://user-images.githubusercontent.com/41438361/173502096-18c88a96-a206-4b4d-a82f-4b864ead3a1b.png">

`.charYScale` : 도메인의 값이 범위를 설정. 축을 보면 0~200까지만 나와있다.

<img width="1603" alt="image" src="https://user-images.githubusercontent.com/41438361/173502217-4daf817d-5ac9-4e4f-86e4-0038ded8ac5f.png">

`.chartForgroundStyleScale` : 각 요소의 색을 결정

### switching chart type

<img width="727" alt="image" src="https://user-images.githubusercontent.com/41438361/173497267-8cc82aa3-b745-41a4-8be8-925f1cdf901d.png">
<img width="748" alt="image" src="https://user-images.githubusercontent.com/41438361/173497237-b4a8fc14-aa06-4aa9-8260-b44d427942ad.png">

선언형 문법 덕분에 차트 타입을 바꾸기 쉽다.

### Marks

<img width="1649" alt="image" src="https://user-images.githubusercontent.com/41438361/173497963-1141ead3-db3c-4db9-8a8a-63196a680f87.png">

Swift Chart는 다양한 종류의 mark type을 지원하고, 이들을 조합해서 복잡한 차트를 만들 수 있다.

<img width="1602" alt="image" src="https://user-images.githubusercontent.com/41438361/173498104-e4655e2e-b254-4160-bdc1-13165afe3530.png">

AreaMark를 사용해서 범위 그래프 표시 가능.

<img width="1603" alt="image" src="https://user-images.githubusercontent.com/41438361/173498203-28fa0114-4a17-4297-8913-57290678a48c.png">

BarMark와 RectanlgeMark를 사용해서 바 내부에 직사각형 마크를 표시 가능. 추가로 `width: .ratio(0.6)` 과 같은 코드를 각 Mark에 적용해서 너비를 조정할 수도 있다.

<img width="1593" alt="image" src="https://user-images.githubusercontent.com/41438361/173501188-a1b3e91d-aafc-44f9-a6e2-c6832956d77a.png">
<img width="1593" alt="image" src="https://user-images.githubusercontent.com/41438361/173498490-ba499ab5-5778-4813-8d46-de6bd3db71ee.png">

RuleMark를 추가해서 평균 값을 보여주고, `.annotation` modifier를 통해 rule mark 위에 텍스트 라벨을 추가할 수 있다.

## Plotting data with mark properties

<img width="1639" alt="image" src="https://user-images.githubusercontent.com/41438361/173501331-de882231-db4f-4fad-acc0-a2d2e1c4f374.png">

* Quantitative : 숫자 값
* Nominal/Categorical : 카테고리, 그룹
* Temporal : 시간

<img width="1667" alt="image" src="https://user-images.githubusercontent.com/41438361/173501517-5ca01d49-4357-4002-ae92-cda7dafda9f6.png">
<img width="1704" alt="image" src="https://user-images.githubusercontent.com/41438361/173501558-737ecff5-0522-4a4c-85d6-9fc539f6ab6a.png">

X, Y 축의 데이터 타입에 따라 mark의 동작이 달라진다. 바의 방향은 숫자 데이터가 어떤 축에 있는지에 따라 달라진다.

<img width="1579" alt="image" src="https://user-images.githubusercontent.com/41438361/173501711-66715cf1-fee4-4397-a72e-93d76b5a72ee.png">

Swift chart는 6개의 mark type, 6개의 mark property를 가지고 있다. 여기에 데이터 타입 세 개의 조합으로 다양한 형태의 그래프를 만들 수 있다.

## Customizations

<img width="1655" alt="image" src="https://user-images.githubusercontent.com/41438361/173502381-4e426add-91df-41aa-89bb-df4047b2788a.png">
<img width="1640" alt="image" src="https://user-images.githubusercontent.com/41438361/173502460-7f8428c3-5655-4cd1-b958-18da578c3d14.png">

* axes, legends : 차트를 해석하게 돕는다.
* plot area : 두 축 사이의 영역으로 mark로 데이터를 나타낸다.

Swift Chart에서는 이들을 커스터마이징 할 수 있다.

### Axes & legends

<img width="1600" alt="image" src="https://user-images.githubusercontent.com/41438361/173502761-2955b8cb-15e6-42dd-9db3-61c5dafe363b.png">

AxisMark를 사용해서 축을 커스터마이징할 수 있다. 

`stride(by:)` : 월별 라벨을 추가
`AxisValueLabel` : 월별 라벨이 화면상에서 잘리는 것을 방지하기 위해 한 글자씩 나타나게 함

<img width="1610" alt="image" src="https://user-images.githubusercontent.com/41438361/173503137-c939f07f-28b3-491a-940f-0bddbd6dbbb8.png">

위와 같이 분기 별로 축 표현 가능

* `charXAxis(.hidden)` : 축을 보이지 않게 한다.
* `charLegend(.hiddnet)` : legend 영역을 보이지 않게 한다.

### Plot area

<img width="1572" alt="image" src="https://user-images.githubusercontent.com/41438361/173503668-d5aef0f3-9cba-4d45-a441-b982062710dc.png">

`.frame` modifier 를 사용해서 Plot 영역이 특정 크기나 비율을 갖게 할 수 있다.

<img width="1596" alt="image" src="https://user-images.githubusercontent.com/41438361/173503861-4e2b63af-b3db-48ed-8c2f-f67e8c6f22b5.png">

* `.background` : 배경색
* `.border` : 경계

<img width="1664" alt="image" src="https://user-images.githubusercontent.com/41438361/173504144-295d1c26-d79b-489f-822b-4407fcc36a11.png">

X, Y scale에 접근할 수 있는 ChartProxy. 

* `position(for:)` 메서드를 사용해서 주어진 데이터 값이 차트에 표시된 곳의 위치를 가져온다.
* `value(at:)` : 주어진 위치에 있는 데이터 값을 가져온다.

<img width="538" alt="image" src="https://user-images.githubusercontent.com/41438361/173504527-131ca0b7-201b-454b-9a68-a30b741f9794.png">
<img width="905" alt="image" src="https://user-images.githubusercontent.com/41438361/173504564-c753fae6-e14e-48dc-ab2a-490218801742.png">

드래그로 차트 내의 영역을 선택해서 작업을 할 수 있게 할 수 있다. `.chartOverlay`나 `.chartBackground` modifier에서 chart proxy 객체를 가져올 수 있다.
SwiftUI의 overlay, background modifier랑 비슷하다고 한다.

<img width="1571" alt="image" src="https://user-images.githubusercontent.com/41438361/173504776-d0fe81a7-eea2-4045-b9fc-de8563af39a5.png">

* `.chartOverlay` modifier : chart proxy를 제공한다.
* `GeometryReader` : overlay view의 geometry에 접근할 수 있게 한다.
* `Rectangle` view : SwiftUI의 DragGesture에 응답하는 뷰

1. 드래그 제스처 시작 : `startX`, `currentX`로 시작점의 x, 차트의 plot area 내의 위치의 x좌표를 찾는다.
2. 구한 좌표를 통해 chart proxy를 사용해서 일치하는 Date 값을 찾고, SwiftUI state를 설정한다.

<img width="1607" alt="image" src="https://user-images.githubusercontent.com/41438361/173505253-e87f701e-8629-4e7d-8783-56e63c7aa465.png">

설정된 state에 따라 `RectangleMark`를 정의해 선택된 Date 범위를 시각화할 수 있다.

=> Design App Experiences with Charts, Design an Effective Chart 세션에서 더 확인 가능
