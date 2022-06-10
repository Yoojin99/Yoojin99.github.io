---  
layout: post  
title: "[iOS] - Platforms State of the Union"  
subtitle: ""  
categories: app
tags: app-ios wwdc
comments: true  
header-img: 

---  

[영상](https://developer.apple.com/videos/play/wwdc2022/102/?time=3999)

1. Vision for Platform
2. System Experience
3. New APIs

# Vision for Platform

Swift, SwiftUI, Xcode 모두 업데이트 됨

## Swift

Fast, Modern, Safe. 

* Concurrency
* Readability
* Customization
* Performance

### Concurrency

새로운 오픈 소스 패키지 async algorithm : sequence algorithm에 concurrency 적용 가능

<img width="432" alt="image" src="https://user-images.githubusercontent.com/41438361/172970406-93987655-cff0-4b96-a30e-62abe39b5334.png">

타임 유닛을 나타내는 새로운 clock type, async algorithm이 이를 기반으로 동작함.

Distributed actors : actor isolation을 더욱 발전시킴. 여러 프로세스, 기기간에 소통할 수 있다. `distributed` 키워드를 붙이면 actor, method를 원격에서 접근될 수 있게 한다.

<img width="521" alt="image" src="https://user-images.githubusercontent.com/41438361/172970747-18e908fe-6a4b-48fe-ba4a-9617c6a561a6.png">

### Usability

Regular Expression 개선. 새로운 regular expression literal.

<img width="446" alt="image" src="https://user-images.githubusercontent.com/41438361/172970970-6d37ac4c-fbb0-4880-9ca6-76fd94603fa0.png">

Regex Builder : literal 을 builder로 변환시켜줌

<img width="1151" alt="image" src="https://user-images.githubusercontent.com/41438361/172971305-f685a50b-bdf4-44a3-b457-8c0ca2fdf375.png">

### Generic

`some` 키워드를 통해 코드를 간결하게 만듦

<img width="577" alt="image" src="https://user-images.githubusercontent.com/41438361/172971526-5a9d2c34-46a8-4ad6-b7e3-375f32f1f37c.png">
<img width="348" alt="image" src="https://user-images.githubusercontent.com/41438361/172971561-3f389fab-4782-447d-9ff5-68b5d88893d6.png">

`any` 키워드 : 어떤 타입의 ~라도 가질 수 있게 해줌

<img width="320" alt="image" src="https://user-images.githubusercontent.com/41438361/172971603-26986a9e-77da-4de6-9255-c293f898aa50.png">

### Swift Package Manager

Swift package manager는 앱의 dependency를 관리하고 다른 개발자가 출시한 패키지를 사용할 수 있게 해줌.

Package Plugins : Plugin은 다른 의존성과 마찬가지로 프로젝트에 쉽게 추가할 수 있는 Swift package. 앱 내의 코드로 존재하지 않고, 앱을 빌드하는 걸 도와주는 코드임.

적용하는 방법 두 가지.

1. Command plugin : 필요할 때 사용하는 것
2. Build plugin : 프로젝트가 빌드될때마다 사용됨

## SwiftUI

### App navigation

새로운 navigation API : 서택을 저장, 복구할 수 있고, 네비게이션 스택 전체를 바꿀 수 있다.

### Layout

새로운 Grid API

<img width="284" alt="image" src="https://user-images.githubusercontent.com/41438361/172973276-b7d68538-66ff-436b-a12a-eb4a9c5da3c8.png">

Custom Layout API : 레이아웃 로직을 재사용할 수 있게 도와줌

### 새로운 Interface elements

Half sheets

<img width="388" alt="image" src="https://user-images.githubusercontent.com/41438361/172973467-e95f1ad6-5278-481c-95c5-0f34607eb4b8.png">

Share sheets

### Swift Charts

SwiftUI에서 사용 가능한 커스터마이징 가능한 차트 프레임워크

<img width="340" alt="image" src="https://user-images.githubusercontent.com/41438361/172973728-d346d08c-91e2-4ede-ac0c-292735765108.png">

# System Experience

## WidgetKit

개인화, 아름답게 변경. 위젯이 중요함. 

WidgetKit

<img width="520" alt="image" src="https://user-images.githubusercontent.com/41438361/172989696-21998bfa-bdb2-4371-a7d2-ff0e73eac75c.png">
<img width="581" alt="image" src="https://user-images.githubusercontent.com/41438361/172989723-d20551f3-1662-43a6-b7a8-cbfe2557a67a.png">

Circular Widget : 이미지, 게이지, 텍스트 포함
Rectangular : 더 넓은 정보 포함 가능
Inline : 심볼, 텍스트

## Live Activities

실시간으로 변경되는 걸 반영할 수 있음. iOS 16 ~

<img width="291" alt="image" src="https://user-images.githubusercontent.com/41438361/172989803-5ea28e5b-e456-441f-9363-1ef334b7e2d9.png">

## Messages Collaboration API

앱에서 협동할 수 있는 기능들을 메세지와 페이스 타임으로 가져올 수 있음. (메세지와 페이스 타임을 사용하면서 기능을 협업할 수 있다는 뜻인듯)

## App Intents, App, Shortcuts

앱의 기능을 시스템에서 접근가능하게 해서 시리, shortcut을 통해 접근할 수 있게 함. 기존에 shortcut을 사용하기 전에 직접 등록해야 했지만 이를 자동으로 등록하게 해줌.

시리를 통해 shortcut을 실행시킬 수도 있고 spotlight에서 shortcut을 찾을 수 있다.

<img width="517" alt="image" src="https://user-images.githubusercontent.com/41438361/172990585-688d0e48-af6a-4231-9247-aea19095ef81.png">
<img width="575" alt="image" src="https://user-images.githubusercontent.com/41438361/172990898-6b2845e2-2fe0-4261-8f4c-0785ebe13780.png">

## Passkeys

Passkey는 인증 플로우를 빠르고 효율적으로 만들고 비밀번호와 관련된 보안 이슈를 해결한다. 

Passkey를 iPhone에서 생성한다. 비밀번호 입력할 필요 없다. 이를 iCloud Keychain에 저장한다. -> 다른 모든 애플 기기에서 안전하게 사용가능

WWDC 영상 여러곳에서 지속적으로 강조되는 사항이지만 애플이 개인 정보 보호에 신경을 쓰는 만큼 passkey의 안정성에 대해서도 매우 자신있게 얘기하고 있다. 특히 피싱이 아예 불가능하다고 한다.

# New APIs

## iPadOS

데스크탑과 비슷한 경험 제공.

DriverKit

## watchOS

CallKit

## Ads and Privacy

* SKAdNetwork : 사용자 추적없이 광고. 유연성 있게 개선

## ScanKit, RoomPlan

## Focus

Focus filter를 사용해서 필터링.

## Metal3

애플 플랫폼에서 게임 등의 앱을 개발하는데 도움을 주는 그래픽스, 연산 API. 

<img width="466" alt="image" src="https://user-images.githubusercontent.com/41438361/172994009-c4bd8393-b854-45fb-9dd1-ec8580d88030.png">

## MapKit

<img width="1376" alt="image" src="https://user-images.githubusercontent.com/41438361/172994448-fcf3d0bf-e424-4420-8f17-8784f7372471.png">

* 3D 지도, 디테일 향상
* 다크모드에 따라 다르게 보임
* LookAround : 3D 뷰(사진) - 네이버 거리뷰와 비슷
* Apple Map Server APIs :Geocode, Reverse Geocode, Search, 등등
* 사용자 정보 안전하게 관리

## Weather

<img width="1728" alt="image" src="https://user-images.githubusercontent.com/41438361/172994987-e080217b-7f44-46bd-8415-797777f721c7.png">

WeatherKit : Swift API, REST API. 특정 횟수 이상 호출하려면 추가 요금 내야함

## Live Text, VisionKit

<img width="1084" alt="image" src="https://user-images.githubusercontent.com/41438361/172995493-1a1859ae-c394-4bca-bc5c-8964d6a52ac3.png">

* Live Text API : 사진 내의 글과 상호작용 가능
* Data Scanner API

*한국어 가능!!*
