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




# Refine your visual

# Ask for information

# Make it discoverable
