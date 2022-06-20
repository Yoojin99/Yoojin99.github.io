---  
layout: post  
title: "[iOS] - Design App Shortcuts"  
subtitle: ""  
categories: app
tags: app-ios wwdc
comments: true  
header-img: 

---  

[Design App Shortcuts](https://developer.apple.com/videos/play/wwdc2022/10169/?time=21)

<img width="464" alt="image" src="https://user-images.githubusercontent.com/41438361/174237355-184245e7-3516-40de-bae7-75d6351b6775.png">

사람들은 Siri와 Spotlight를 통해 기기에서 특정 작업을 수행할 수 있다. Shortcuts는 OS를 통해 사람들이 내 앱에서 작업을 수행할 수 있게 해준다.

<img width="674" alt="image" src="https://user-images.githubusercontent.com/41438361/174237409-7d066d5a-a5d9-4c22-9b7c-5dec82dec174.png">

모든 shortcut은 action이라 불리는 기본적인 요소에서 시작하는데, action은 사람이 앱을 통해 할 수 있는 개별적인 작업이다. 예를 들어 메세지를 보내거나 리마인더를 추가하는 작업이 포함된다.

<img width="725" alt="image" src="https://user-images.githubusercontent.com/41438361/174237571-68e27ff2-d159-4332-b055-06a232f34880.png">

Shortcut은 여러 다른 방법으로 사용될 수 있다.

1. custom shortcut : 앱에서 하나 이상의 action을 통해 생성한 shortcut
2. app shortcut : 앱에서 하나 이상의 액션을 포함해서 개발자가 생성한 shortcut이다.

<img width="500" alt="image" src="https://user-images.githubusercontent.com/41438361/174238025-10cc161b-30d1-4e5a-83c0-f7616afa8cf6.png">

지금까지 사람들은 "Add to Siri" 버튼을 눌러서 app-shortcut을 활성화해야 했다.

<img width="497" alt="image" src="https://user-images.githubusercontent.com/41438361/174238186-a91f03fd-05b8-4e3f-8939-cd7d8a737dad.png"><img width="483" alt="image" src="https://user-images.githubusercontent.com/41438361/174238255-9a4c2eb5-1c98-43b0-84e5-33c321480849.png"><img width="481" alt="image" src="https://user-images.githubusercontent.com/41438361/174238311-b38dbc32-38d4-4550-890d-cbbac0d12d2c.png"><img width="481" alt="image" src="https://user-images.githubusercontent.com/41438361/174238369-d73470d0-550c-4616-9715-19780d88bd68.png">

iOS 16부터 내가 생성한 App Shortcut은 앱이 설치되는 즉시 자동으로 사용할 수 있게 된다. 즉 Siri에서 사용할 수 있고, Spotlight에서 볼 수 있으며 Shortcuts doqdptj qhf tn dlTek.

# Identify a feature

앱이 다양한 기능을 가지고 있는데 어떤 것을 shortcut으로 만들어야 할까?

## Self-contained

<img width="623" alt="image" src="https://user-images.githubusercontent.com/41438361/174238980-ed60f61e-de87-4455-bb5b-f7793f78fe69.png">

앱 밖에서 수행할 수 있는 self-contained 작업에 적용할 수 있다. 

좋은 app shortcut은 앱을 focus할 필요가 없는 shortcut이다. Siri나 검색을 통해 완전히 수행할 수 있는 가벼운 작업을 shortcut으로 만드는 것이 좋다.
(e.g. Siri에게 잠 잘오는 노래 틀어달라고 하기. 이 경우 음악 앱을 열 필요 없이 음악만 재생하는 간단한 작업이다.)

## Straightforward

<img width="594" alt="image" src="https://user-images.githubusercontent.com/41438361/174239422-f84641f0-1fe6-465c-9c88-4ca16775ade8.png">

또한 Shortcut이 분명해야 한다. 하는데 많은 노력을 들이면 안된다. 

<img width="1564" alt="image" src="https://user-images.githubusercontent.com/41438361/174244900-9ef0e4aa-974e-42b9-acff-c9d4feb90fa0.png">

예를 들어 위와 같이 많은 입력과 단계를 요하는 기능은 App Shortcut으로 적절하지 않다. 대신 앱을 직접 사용하지 않고서도 빠르게 처리할 수 있는 작업에 집중한다.

<img width="1423" alt="image" src="https://user-images.githubusercontent.com/41438361/174245205-a754a355-1049-499d-9267-f5823b895bc0.png">

생성할 수 있는 shortcut의 최대 개수는 10개지만, 앱의 핵심기능들을 2개에서 5개의 높은 퀄리티의 shortcut으로 추릴 수 있을 것이다.

# Pick a name

<img width="480" alt="image" src="https://user-images.githubusercontent.com/41438361/174247463-1616f6d3-931e-429d-bcf2-fd26e66a996e.png">
<img width="473" alt="image" src="https://user-images.githubusercontent.com/41438361/174247506-196dcf46-a544-4bcd-a8fe-73177c00574c.png">
<img width="482" alt="image" src="https://user-images.githubusercontent.com/41438361/174247562-8747cde6-34bc-4c75-8f82-3684d7c6ae79.png">

Shortcut에 이름을 붙이는 것도 중요하다. Shortcuts 앱에서 내 앱의 헤더 아래에 표시되고, Spotlight에서 검색할 때 사용되고, Siri를 통해서 shortcut을 실행할 때 이름이 사용되기 때문이다.

## Keep it brief and memorable

가능한 문구를 간결하게 만들어야 한다. 기억하기 쉬우면서 어떤 기능을 하는지 분명하게 나타내야 한다.

## Include your app name

<img width="933" alt="image" src="https://user-images.githubusercontent.com/41438361/174247989-170caf88-d662-4dc7-8161-e6da520ac277.png">

이를 위해 앱 이름을 문구에 포함시켜야 한다. 하지만 앱 이름을 꼭 그대로 사용할 필요는 없다. (e.g. "Order Starbucks Coffee"라는 문구에서 "Starbucks Coffee"가 앱 이름이라고 해보자. 이럴 경우 그냥 "Order Starbucks"라고 해도 된다. 물론 의미상으로는 "Order Starbucks Coffee"가 더 말이 된다.)

## Generate and localize synonyms

<img width="675" alt="image" src="https://user-images.githubusercontent.com/41438361/174248501-e18de89b-9671-4e7f-8c19-fc451da2e947.png">

또한 App Shortcut의 이름으로 Siri를 통해 shortcut을 실행시키기 때문에, 언어마다 사용될 수 있는 다양한 표현을 고려해야 한다. 가령 "Record Voice Memo"를 위와 같이 다양하게 말할 수 있다.

하지만 여기에서 원래의 목적에서 너무 멀어지는 문구는 고려하지 않는 것이 좋다. 마지막의 "Save a Voice Memo"가 그런 경우인데, 목적은 "녹음"하는 것이지 "저장"하는 것이 아니기 때문에 이는 적절한 문구가 아니다.

## Use a dynamic parameter (optional)

<img width="711" alt="image" src="https://user-images.githubusercontent.com/41438361/174249032-cff6f480-0535-43fd-817f-28ece1f6d901.png">

다이나믹 파라미터를 shortcut 이름 안에 넣어서 사람들이 Siri나 Spotlight에 다양한 문구로 말하게 할 수도 있다.

<img width="295" alt="image" src="https://user-images.githubusercontent.com/41438361/174507876-0fa1a249-80ce-43c4-a213-e8f8db7bab13.png"><img width="283" alt="image" src="https://user-images.githubusercontent.com/41438361/174507887-38786196-f15f-4dfc-9a02-4dd489cdb68a.png">

다만 주의사항이 있다.

<img width="1611" alt="image" src="https://user-images.githubusercontent.com/41438361/174507981-91e92c8a-dec0-449e-9b9f-b5d51ffbf45c.png">

1. 문구에 하나의 다이나믹 파라미터만 넣을 수 있다.
2. 다이나믹 파라미터의 개수는 유한해야 한다.

가능한 파라미터 값들의 목록은 앱이 실행됐을 때 업데이트 될 수 있다. 

<img width="1578" alt="image" src="https://user-images.githubusercontent.com/41438361/174508374-7721e701-d798-4e9f-b1c9-b554819c4f94.png">

각 파라미터 값은 App Intent와의 조합으로 app shortcut가 하나씩 생성된다. 이들은 자동으로 생성돼서 Shortcuts 앱에 표시된다. 

<img width="491" alt="image" src="https://user-images.githubusercontent.com/41438361/174508530-dcb5239e-1211-4475-ad16-ec60330caecb.png">

이 파라미터들은 action에 parameter summary를 제공하면 Shortcuts editor에서도 볼 수 있다. 

=> Shortcut, Siri, Suggestions를 위한 action을 디자인하고 Shortcuts editor에 보이게 하는 방법은 "Design great actions for Shortcuts, Siri, and Suggestions" 세션에서 확인할 수 있다. 

<img width="1701" alt="image" src="https://user-images.githubusercontent.com/41438361/174508718-13c34bf4-9fdc-4bcc-9947-11e6da09f459.png">

또한 다이나믹 파라미터를 사용할 때 주의해야 할 점은 문구의 어떤 부분이 파라미터인지를 명확히 해야 한다는 것이다. App shortcut 하나당 하나의 다이나믹 파라미터만 가질 수 있는데, 위에서 "nature sounds"는 변경될 수 있는 두 번째 파라미터처럼 보인다. 

<img width="1681" alt="image" src="https://user-images.githubusercontent.com/41438361/174508840-2a172eb1-7781-4ca4-a4f1-f940e613e984.png">

이를 해결하는 방법은 문구를 간단하게 만드는 것이다.

# Refine your visual

<img width="1612" alt="image" src="https://user-images.githubusercontent.com/41438361/174509173-87cbc52c-d791-4b33-a6c3-d4a48f25147c.png">

Custom Snippets와 Live Activities는 정보를 보여주고, app identity를 드러낼 수 있다. 

## Semi-translucent background

<img width="1425" alt="image" src="https://user-images.githubusercontent.com/41438361/174509323-42fd67c5-5e3f-4313-8cd7-18450ef9e0fe.png">

대부분의 앱이 불투명 배경을 사용하는 것과 달리, snippet은 약간은 투명한 재질을 사용한다. 불투명한 배경을 사용하는 대신 이 반투명한 재질 위에 요소들을 올려서 화면에 보여준다. 

## Vibrant label color

<img width="1127" alt="image" src="https://user-images.githubusercontent.com/41438361/174509479-5a6972ac-5bfb-47b8-9c9e-8db8ca29f1a6.png"><img width="1174" alt="image" src="https://user-images.githubusercontent.com/41438361/174513561-f2c93970-fca3-4c64-90f8-d47cd4d365ad.png">

텍스트를 그릴때는 vibrant label color를 사용해서 반투명한 배경에 잘 대조될 수 있게 한다. 이를 통해 다크 모드에서도 자동으로 텍스트가 잘 보일 수 있게 할 수 있다. 

## Snippets and Live Activities

<img width="1675" alt="image" src="https://user-images.githubusercontent.com/41438361/174513616-34106f0b-d28f-40cc-a072-8976a3b41be0.png">

이제 iOS 16에서는 결과를 보여줄 두 가지 방법이 있다.

1. Live Activities : 사람들이 정보에 지속적으로 접근해야 할 경우(카운트 다운 등)에 사용한다. 이벤트가 끝날 때까지 잠금화면에서도 확인할 수 있다.
2. Custom Snippets : 실시간으로 정보를 확인할 필요가 없고 shortcut 자체가 action이나 정보를 담고 있을 때 사용한다.

<img width="845" alt="image" src="https://user-images.githubusercontent.com/41438361/174526940-ca90b9c1-d205-4423-b82d-9638179329c8.png">

iOS snippet을 더 자세히 보자. 

* supporting dialog : Siri가 말하는 부분
* custom visual : 기본적으로는 supporting dialog와 함께 나타난다.

<img width="1331" alt="image" src="https://user-images.githubusercontent.com/41438361/174535395-c0242283-8565-49ea-ac6e-6b0bc69df67b.png">

supporting dialog가 화면에 나타나는 것과는 관련 없는 경우 소스 코드에서 dialog가 화면에 나타나지 않게 설정해야 한다. 

<img width="1670" alt="image" src="https://user-images.githubusercontent.com/41438361/174535518-7cd7e8fd-5d98-4d57-a83c-64d74b57ed5b.png">

AirPod와 같은 음성 전용 기기에서는 시리는 내가 제공하는 전체 dialog를 읽어준다. 이때는 supporting dialog와 custom visual의 중요한 정보들을 dialog가 포함하고 있어야 한다. 

<img width="1114" alt="image" src="https://user-images.githubusercontent.com/41438361/174535690-de09697a-d600-475b-bdeb-b7d2eb8d70b1.png">

Apple Watch도 이제 Custom Snippet을 지원한다. 따라서 watchOS 9에서 shortcut이 어떻게 동작하는지 확인해야 한다. 

## Spotlight

<img width="486" alt="image" src="https://user-images.githubusercontent.com/41438361/174535973-30514b9a-7c7a-4abe-b709-0af3cc7fd1a0.png"><img width="469" alt="image" src="https://user-images.githubusercontent.com/41438361/174536127-5cb6b929-badd-4f8f-86e3-9a68ef688ff6.png"><img width="490" alt="image" src="https://user-images.githubusercontent.com/41438361/174536279-21fa53ec-071a-4994-b224-d6b9e3c88cea.png">

iOS 16에서 app shortcut은 spotlight에도 뜨게 된다. 

* 만약 누군가가 앱 이름이나 app shortcut을 검색한다면 App shortcut 배열의 첫 번째 요소가 Siri Suggestion에 뜨게 된다. 
* shortcut의 이름으로 바로 검색할 수도 있다.
* 누군가가 처음 Spotlight을 실행할 때 app이 Siri Suggestion인 경우 내 top shortcut이 검색창에 글자를 입력하기도 전에 뜨게 된다.

<img width="1580" alt="image" src="https://user-images.githubusercontent.com/41438361/174536560-1ebfc2a6-9857-4cfe-9fde-80c0aac71db9.png">

각 shortcut이 우측에 SF Symbol을 갖고 있으니 SF Symbol 라이브러리를 확인해서 의도를 정확하게 반영할 수 있는 것을 골라야 한다.

<img width="1432" alt="image" src="https://user-images.githubusercontent.com/41438361/174536928-0572bd06-e097-45ae-9aec-3ddf0f0dfd20.png">

action과 파라미터의 순서가 Spotlight에 뜨는 app shortcut의 순서에도 영향을 미친다. 파라미터의 순서는 완전히 동적이기 앱이 실행될때마다 업데이트 될 수 있다. 

만약 카푸치노를 주문하는 shorcut과 라떼를 주문하는 shortcut이 있는데 고객이 라떼를 주문하는 shortcut을 더 많이 이용했다고 해보자. 이를 Spotlight의 추천 목록에 띄우고 싶다면 entity query나 dynamic options provider를 통해 라떼를 주문하는 shortcut을 첫 원소로 설정할 수 있다. 이러면 Shorcuts 앱과 Spotlight에서도 첫 번째 shortcut이 된다. 파라미터의 수가 증가할 수록 우선순위를 정하는 것이 중요하다.

<img width="1319" alt="image" src="https://user-images.githubusercontent.com/41438361/174537477-1f7f3f35-dce0-4298-b1d6-819d91c0991e.png">

또 Shortcuts 앱에서 shortcut이 노출될 색을 고를 수 있다. 

# Ask for information

Shorcut이 빠르고, 그 자체로도 동작할 수 있지만, 어떤 작업을 하기 위해 정보가 필요할 때가 있다. 

<img width="219" alt="image" src="https://user-images.githubusercontent.com/41438361/174538307-02cfdd30-ae50-4ae7-9086-03864f49107a.png">

"Start meditaion"이라는 파라미터를 포함하지 않은 문구를 생각해보자. 파라미터를 포함해서 어떤 치료를 할 것인지 정하거나 할 치료의 종류를 가정해서 작업을 진행할 수도 있는데, 그렇지 않고 모호한 경우도 있을 것이다.

## Parameter confirmation

<img width="960" alt="image" src="https://user-images.githubusercontent.com/41438361/174538571-d0533314-580e-4a52-a01e-f0cbb00fbd57.png">

요청이 모호한 경우에 정보를 요청할 수 있는데, 의미있는 가정을 해서 사용자에게 확인을 요청할 수도 있다. 이를 Parameter confirmation이라 한다.

가정을 할 때는 이전 선택이나 가장 인기 있는 선택에 기반해서 할 수 있다. Parameter confirmation은 디테일을 정하고 작업을 진행하는 효율적인 방법을 제공한다.

## Disambiguation

<img width="793" alt="image" src="https://user-images.githubusercontent.com/41438361/174538870-1ac13c3c-93ce-48c9-8e44-7f91707d0d1b.png">

만약 사용자가 shortcut을 이전에 한 번도 사용한 적이 없거나 사용자에게 권할 파라미터가 없다고 할 때 짧은 목록을 제공할 수도 있다. 이를 disambiguation이라 한다.

Disambiguation은 shortcut에 익숙하지 않은 사람들에게 사용할 수 있는 값들을 알려준다. 최대 5개까지 목록을 보여줄 수 있는데, 그 이유는 AirPods과 같은 음성 전용 기기에서 Siri가 목록을 읽어주기 때문이다. 

## Open-Ended Request

<img width="912" alt="image" src="https://user-images.githubusercontent.com/41438361/174539197-fda5a3cd-9121-4ee1-8c2d-2494239fb584.png">

숫자, 지역, 문자열과 같이 간단한 목록으로 제공할 수 없는 다양한 응답이 가능한 값이 필요할 수도 있다. 이때 open-ended request를 사용할 수 있다. 사용자가 어떠한 값이라도 말할 수 있기 때문에 앱이 요청하는 정보가 어떤 타입인지를 명확히 해야 한다.

App Intent 프레임워크는 이런 open-ended request를 위한 여러 옵션을 제공한다. 숫자, 날짜, 시간 등을 지원할 수 있다.

<img width="892" alt="image" src="https://user-images.githubusercontent.com/41438361/174539535-25aa8221-4c97-4845-8d9d-16372aa74a00.png">

위에 있는 것 중 파라미터가 요청하는 정보의 타입으로 사용할 수 있는 것이 있다면 이를 사용하는 것이 좋다. Custom entity를 사용할 수도 있다.

App entity를 구현하는 법은 "Dive into App Intents" 세션에서 확인할 수 있다.

## Intent Confirmation

<img width="868" alt="image" src="https://user-images.githubusercontent.com/41438361/174539936-93e151be-b66d-4c58-a930-7012090227a1.png">

앱이 요구하는 모든 정보를 갖고 있어도 shortcut을 실행하기 전에 최종 확인을 해야 할 때가 있는데, 이를 intent confirmation이라고 한다.
이는 결제라던지, 수많은 사람에게 캘린더 일정을 보내는 것과 같은 작업에만 사용하는 것이 좋다. Shortcut은 작업을 빠르게 하는 것이 목적이기 때문에 이를 적절히 사용하는 것도 중요하다.

<img width="539" alt="image" src="https://user-images.githubusercontent.com/41438361/174540272-633caa3a-65b3-4e2b-a16d-6073e4c6bc05.png">

intent confirmation에서 유의해야 할 점은 snippet에 있는 버튼 쌍이다. 버튼은 수행할 작업을 진행할 것인지 여부를 확인할 명확한 동사를 사용해야 하고, confirm과 같은 모호한 단어를 사용하면 안된다.

<img width="736" alt="image" src="https://user-images.githubusercontent.com/41438361/174540520-df3247ba-7409-4c4a-8649-5730800629da.png">

App Intent 프레임워크는 사용할 수 있는 기본 동사와 유의어를 제공한다. 만약 여기에서 벗어날 경우 커스텀 문자열을 제공할 수도 있지만 Siri가 비슷한 문장을 이해할 수 있게 유의어들을 같이 제공해야 한다. 

# Make it discoverable

<img width="1139" alt="image" src="https://user-images.githubusercontent.com/41438361/174541784-31abdbfd-baef-4271-a736-ee8453148146.png">

기존의 add to Siri 버튼 대신에 앱의 기능으로서 app shortcut을 보여줄 수 있게 한다. 앱이 이런 팁을 언제 보여줄 것인지 중요한데, 사람들이 어떤 작업을 하기 전이나 후에 이를 반복할 것으로 예상되는 곳에 쓰는 것이 좋다. 그리고 이런 팁을 닫아서 사용자들이 그만 볼 수 있게도 해야 한다.

<img width="1049" alt="image" src="https://user-images.githubusercontent.com/41438361/174542194-0896a0ad-329d-4ff5-95cc-05a0365f88f9.png">

또한 app shortcut 목록을 보여주는 Shortcuts 앱으로 바로 이동할 수 있는 버튼을 사용할 수 있다. 
