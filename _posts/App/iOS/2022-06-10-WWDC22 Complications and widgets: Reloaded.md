---  
layout: post  
title: "[iOS] - Complications and widgets: Reloaded"  
subtitle: ""  
categories: app
tags: app-ios wwdc
comments: true  
header-img: 

---  

[Embrace Swift generics](https://developer.apple.com/videos/play/wwdc2022/10050/)

# Complication timeline

<img width="1449" alt="image" src="https://user-images.githubusercontent.com/41438361/174199445-4a589926-0b9f-4565-80b3-1b2c90af681f.png">

Complications는 watchOS 플랫폼의 핵심으로, 워치 화면에 빠르게 한눈에 알아볼 수 있는 정보를 출력한다. 

* watchOS 2에서 ClockKit은 나만의 complications를 생성할 수 있게 했다.
* watchOS 5에서 그래픽이 발전된 complication이 소개됨
* watchOS 7에서 SwiftUI complication과 여러 complication이 소개됐다.

<img width="571" alt="image" src="https://user-images.githubusercontent.com/41438361/174199740-04531679-fc80-44e4-81c4-6e3fb7401c26.png">

iOS 16, watchOS 9의 WidgetKit으로 두 플랫폼에서 모두 동작 가능한 위젯을 만들 수 있다. 이를 위해 `WidgetFamily` 타입에 새로운 위젯을 추가했다.
모두 "accessory"로 시작한다.

<img width="1676" alt="image" src="https://user-images.githubusercontent.com/41438361/174199711-23a81b85-6f2a-4bae-a5a3-97b0df1159f4.png">

`accessoryRectangular` : 여러 줄의 텍스트나 작은 그래프, 차트를 보여줄 수 있다. ClockKit의 `graphicRectangular`와 비슷하다.

<img width="1596" alt="image" src="https://user-images.githubusercontent.com/41438361/174199915-522f901c-3c28-4e56-8677-74dc8c5adf52.png">

`accessoryCircular` : 작은 정보, 게이지, 진행상황을 나타내기 좋다. ClockKit의 `graphicCircular`와 비슷하다.

<img width="1582" alt="image" src="https://user-images.githubusercontent.com/41438361/174199979-dd55d038-2ee0-4b6a-9856-c64a21fc9de9.png">

`accessoryInline` : 시간 부분 위에 올라오는 텍스트 전용 슬롯

<img width="793" alt="image" src="https://user-images.githubusercontent.com/41438361/174200094-437565d4-6909-4dbb-a50f-74a9b5a23a4e.png">

`accessoryCorner` : watchOS에서만 사용 가능. 게이지, 텍스트가 있는 둥그런 위젯

=> watchOS와 complication-specific 기능을 보려면 "Go further with WidgetKit complication" 세션에서 확인 가능

# Colors

<img width="1636" alt="image" src="https://user-images.githubusercontent.com/41438361/174200344-573f7b3b-7975-493b-a779-69dd4d5e14d5.png">

세 개의 다른 렌더링 모드.

1. full color
2. accented
3. vibrant

<img width="1060" alt="image" src="https://user-images.githubusercontent.com/41438361/174200440-40ac8502-8906-4b5e-ad04-0c78a3efaa33.png">

* `WidgetRenderingMode` 타입이 이 세 개의 다른 모드를 가지고 있다. 
* `widgetRenderingMode` keypath를 사용해서 `Environment`에서 이 값에 접근할 수 있다.
* 접근 후에 컨텐츠를 모드에 맞게 조정한다.

## Full color

<img width="1683" alt="image" src="https://user-images.githubusercontent.com/41438361/174200645-d68c0c04-8ce1-494c-ae91-7f6c1c7ff050.png">

Full-color 모드에서 컨텐츠를 내가 명시한대로 나타난다. 많은 complication이 full color로 화려하게 나타난다.

## Accented 

<img width="1617" alt="image" src="https://user-images.githubusercontent.com/41438361/174200769-d4b7c75c-2536-4f78-b36c-eadb1c872bd7.png">

Accented 렌더링 모드에서 뷰는 두 종류의 그룹으로 나뉘어 다르게 색이 보여진다. `.widgetAcentable()` 뷰 modifier를 사용해서 그룹을 어떻게 보여줄 지 시스템에 알려줄 수도 있고,  컨텐츠를 Widget Rendering Mode environment 값에 따라 바꿀 수 있다.

<img width="1707" alt="image" src="https://user-images.githubusercontent.com/41438361/174200948-c5999851-95d0-4251-afd2-bbadf9d702bc.png">

시스템은 여러 방법으로 컨텐츠의 색을 설정할 수 있다. 어떤 건 검은 배경화면이지만 watchOS 9에서는 새로운 배경 색으로 설정할 수 있다. 

## Vibrant

<img width="1690" alt="image" src="https://user-images.githubusercontent.com/41438361/174201097-0c592369-7655-408f-92e6-c1ef49a9ea87.png">

iOS vibrant 렌더링 모드에서는 컨텐츠가 완전이 desaturated 된 후 잠금 화면 배경에 맞게 색칠된다. 시스템은 greyscale 컨텐츠를 material appearance로 매핑한다.
이 material은 배경에 있는 컨텐츠에 맞게 적절하게 나타난다. 추가로 잠금 화면은 vibrant 렌더링 모드에 colored tint를 줄 수 있게 구성되었다. 

밝은 색은 더 불투명하고 밝게 나타나고 어두운 색은 배경을 가리지 않을 정도로만 흐리고 덜 밝게 나타난다. 

<img width="983" alt="image" src="https://user-images.githubusercontent.com/41438361/174201462-b2888bf2-a1a2-441c-8e9a-f50ceff8a699.png">

사용자가 글자를 볼 수 있게 이 모드에서 투명한 색을 사용하기 보다는 어두운 색을 사용해서 컨텐츠를 보여주는 것이 좋다. 

<img width="1608" alt="image" src="https://user-images.githubusercontent.com/41438361/174201592-c5b2abbd-0ccd-4d84-89d7-de821694857c.png">

`AccessoryWidgetBackground` 뷰를 사용해서 위젯에 배경을 줄 수 있다. 

<img width="1633" alt="image" src="https://user-images.githubusercontent.com/41438361/174201769-ce60ebaa-3879-4933-ad2e-3908fcef66cb.png">

배경 뷰는 위젯 렌더링 모드에 따라 다르게 나타날 수 있고 워치 화면이나 잠금 화면의 스타일에 맞게 조정될 수 있다. 

# Project setup

<img width="1596" alt="image" src="https://user-images.githubusercontent.com/41438361/174201911-71d856b6-6e3d-43b4-88ca-8d6d5c93c607.png">

프로젝트에 Widget extension target을 추가해서 시작할 수 있다.

## Xcode

먼저 이미 존재하는 iOS target과 코드를 공유하는 새로운 watchOS target을 추가할 것이다.

<img width="313" alt="image" src="https://user-images.githubusercontent.com/41438361/174202460-12343c54-434b-4901-a605-10f10f34179c.png">

1. iOS widget extension target을 복제해서 적절한 이름으로 바꿔준다.

<img width="870" alt="image" src="https://user-images.githubusercontent.com/41438361/174202568-df451a11-84ef-4710-8460-29ebe3ccbd38.png">

2. bundler identifier를 watch app에 맞게 바꿔준다

<img width="862" alt="image" src="https://user-images.githubusercontent.com/41438361/174202650-84a91cd7-4a6e-4749-b32b-a2ba1bb2a7ca.png">

3. watchOS로 target을 설정한다

<img width="1273" alt="image" src="https://user-images.githubusercontent.com/41438361/174202758-6c877107-9dfd-4dd4-ab7c-aec15e40dadd.png">

4. watch app에 새로운 extension을 embed한다.

## Code 설명

*이미 프로젝트에 존재하는 코드를 간략하게 설명하고 넘어가는 부분이므로 자세히 안 봐도 된다*

<img width="824" alt="image" src="https://user-images.githubusercontent.com/41438361/174202867-558483b9-d727-44aa-a957-da05ef9719cc.png">

* `TimelineProvidier` : 시스템이 컨텐츠를 reload할 때 사용됨

<img width="803" alt="image" src="https://user-images.githubusercontent.com/41438361/174202970-8ae63369-bb03-4088-b42d-482af2b35185.png">

* SwiftUI가 다른 familiy들에 맞게 컨텐츠를 생성할 때 사용하는 뷰

<img width="818" alt="image" src="https://user-images.githubusercontent.com/41438361/174203069-8e1b3108-be28-4eea-8715-402e019b772d.png">

* widget configuration

<img width="820" alt="image" src="https://user-images.githubusercontent.com/41438361/174203108-b5c085f0-4301-4e36-b31d-8e9a3b797485.png">

* Xcode preview provider

이 앱은 이미 iOS 홈 스크린 위젯을 지원하는데, 새로운 family를 추가할 것이다.

<img width="812" alt="image" src="https://user-images.githubusercontent.com/41438361/174203252-2c530dc7-2806-4bee-b88e-57066fdaaaa4.png">

System family는 watch에서 쓸 수 없기 때문에 `supportedFamilies`를 플랫폼에 따라 다르게 하기 위해 platform macro를 사용했다.

<img width="821" alt="image" src="https://user-images.githubusercontent.com/41438361/174203357-9828f4b9-5f0d-42d6-a086-d0ee3e3fee7b.png">

그리고 preview provider에 새로운 family를 위한 preview를 추가했다.

<img width="1685" alt="image" src="https://user-images.githubusercontent.com/41438361/174203430-e2902104-b375-4e0a-bb5a-87962ee665e0.png">

이제 watchOS에 빌드하기 전에 새로운 `IntentRecommendation` API를 구현해야 한다. Intent가 configurable하기 때문에 preconfigured list를 제공해야 한다. 이를 `IntentTimelineProvider`의 새로운 `recommendations` 메서드를 오버라이딩 해서 할 수 있다.

<img width="899" alt="image" src="https://user-images.githubusercontent.com/41438361/174203739-e13c1264-fcdb-4dbb-8978-a3f9cf6089b4.png">

이제 빌드가 잘 된다.

<img width="590" alt="image" src="https://user-images.githubusercontent.com/41438361/174203798-6812f2a3-0908-48df-bc77-dcab280ad355.png">

작은 위젯을 위한 컨텐츠도 새로운 form factor에 잘 맞지 않는 것을 확인할 수 있다. 새로운 위젯 family는 홈 화면의 iOS 위젯보다 더 작기 때문에 complication의 컨텐츠를 신경써야 한다.

# Making glanceable views

## Circular

<img width="1783" alt="image" src="https://user-images.githubusercontent.com/41438361/174223706-9939efe7-77c9-412b-8674-5d9b56c3d363.png">

* `progressViewStyle` 를 추가해서 진행상황의 정도를 나타낼 수 있다.

<img width="1791" alt="image" src="https://user-images.githubusercontent.com/41438361/174223930-2a1c5b48-8fe4-4c47-a6ec-b6c79b0bead3.png">

이 prgress view에 애니메이션을 적용하는게 많은 timeline entry를 필요로 할 수 있다는 점이다. 대신, SiwftUI의 새로운 auto-updating ProgressView를 사용할 수 있다. 오직 하나의 timeline entry만 필요하게 된다.

## Rectangular

<img width="1790" alt="image" src="https://user-images.githubusercontent.com/41438361/174224120-20fe1a3a-800c-47d7-b495-d117516e6aeb.png">

Rectangular preview는 더 많은 공간을 차지하기 때문에 여기에서는 3줄이 들어가게 만들었다. 캐릭터의 이름을 강조하기 위해 텍스트 사이즈를 설정하고, headline 폰트 스타일을 사용하고, `widgetAccentable` modifier를 사용해서 색을 설정한다.

<img width="436" alt="image" src="https://user-images.githubusercontent.com/41438361/174224321-aafd60c5-fd53-47f0-bafc-00253475ee88.png">

Accent mode에서는 위와 같이 보인다. 캐릭터의 이름이 accent 색을 띄고 있다. 환경에 맞게 위젯과 complication이 어우러지게 하려면 기본 font 파라미터와 스타일을 사용하는 것이 중요하다.

iOS와 watchOS의 폰트 스타일과 크기는 다르다. iOS는 일반적인 텍스트 디자인을 사용하는데, watchOS는 더 두꺼우면서 둥그런 텍스트 디자인을 사용한다.

<img width="1685" alt="image" src="https://user-images.githubusercontent.com/41438361/174224655-2bda265f-c69a-47d8-85f6-b8833dd274b2.png">

위젯과 complication은 iOS와 watchOS에서 인접하게 나타난다. 폰트 스타일을 `Title`, `Headline`, `Body`, `Caption`을 사용하는 것을 권장한다.

<img width="1711" alt="image" src="https://user-images.githubusercontent.com/41438361/174224882-c757027e-cd2a-4959-808f-90909d5b965b.png"><img width="1749" alt="image" src="https://user-images.githubusercontent.com/41438361/174224964-864bc935-5f8b-42da-8ab9-8f30a49287a0.png">

watch OS에서 아바타를 추가로 띄웠는데, iOS에서 확인해도 똑같이 잘 나온다.

## accessoryInline

<img width="1671" alt="image" src="https://user-images.githubusercontent.com/41438361/174225076-f09e0177-63de-460c-82e6-09f1cf235ba3.png">

accessoryInline 스타일은 텍스트와 이미지를 표시할 수 있다. Inline accesory는 시스템에서 정의된 색과 폰트가 지정된다는 것에 주의하자.

워치 화면 하단에 텍스트를 출력하게 했는데, 공간에 비해 너무 길다. 이 경우 `ViewThatFits`를 사용한다. 

<img width="1503" alt="image" src="https://user-images.githubusercontent.com/41438361/174225422-fd3cfba0-6557-472e-a2a7-391be62bc84c.png">

길이가 다양한 여러 뷰를 제공할 수 있는데, 이러면 `ViewThatFits`는 잘리는 일 없이 사용가능한 공간에 딱 맞는 첫 번째 뷰를 선택할 것이다. 그래서 여기에서는 두 번째 `Text`가 워치 화면에서 보여지고 있다.

<img width="1556" alt="image" src="https://user-images.githubusercontent.com/41438361/174225505-305e156d-661d-4e4b-8989-aa26bb925a2f.png">

만약 저 두 줄의 주석문이 주석 해제가 된다면 두 번째 `Text`가 출력될 것이다.

# Privacy

디바이스가 어떤 상태에 있는지를 고려해야 한다.

* redacting content state : 보안상 컨텐츠를 보이지 않게 하는 모드
* low-luminance state : 화면 밝기가 낮은 상태

<img width="1705" alt="image" src="https://user-images.githubusercontent.com/41438361/174230462-03991fec-c48c-476c-9b6a-82930e8a1311.png">

iOS 잠금 화면의 경우 기본적으로는 디바기으가 잠겨져 있어도 컨텐츠를 보이게 한다. 

<img width="1655" alt="image" src="https://user-images.githubusercontent.com/41438361/174230560-addc7a5c-d6ff-4b7d-8972-b9f689e5a87e.png">

하지만 Settings에서 이런 설정을 바꿔서 사용자가 기기가 잠겼을 때 컨텐츠를 보이지 않게 설정할 수 있다.

<img width="1670" alt="image" src="https://user-images.githubusercontent.com/41438361/174230626-6654283d-7429-4a20-b281-0075c8e93b0b.png">

watch OS에서 기기는 사람이 차고 있는 한 잠금 해제 상태다. 차고 있지만 사용하고 있지 않은 상태에서는 always-on 상태가 되고, 컨텐츠는 밝기를 낮춰 보여준다.

기본적으로 low luminance 상태에서 컨텐츠는 감춰지지 않는다. 

<img width="1667" alt="image" src="https://user-images.githubusercontent.com/41438361/174230850-875d7f6f-0b77-4927-bbbb-7000a6494d0d.png">

잠금화면과 마찬가지로 사용자가 이 always-on 상태에서 complication content를 감추게 설정할 수 있다. 이 상태에서는 컨텐츠가 모두 redaction과 Low luminance 모드 모두에서 잘 작동되도록 보장해야 한다.

<img width="1615" alt="image" src="https://user-images.githubusercontent.com/41438361/174231024-17255196-ef51-401f-8133-17c208eb6a46.png">

워치에서 위젯은 always-on display 를 지원해야 한다. 이를 `\.isLuminanceReduced` environment 값을 통해 always-on 모드에서 컨텐츠가 어떻게 나타나게 할 지 설정할 수 있다.

<img width="1639" alt="image" src="https://user-images.githubusercontent.com/41438361/174231411-6c73ad69-24d1-452f-9437-b9350cd3a7cf.png">

기본적으로 안전 모드는 TimelineProvider가 생성하는 뷰에서 컨텐츠를 감춘 뷰를 보여줄 것이다. `.privacySensitive` modifier를 사용해서 감춰질 뷰만을 표시할 수 있다.
