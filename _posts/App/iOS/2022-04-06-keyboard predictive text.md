---  
layout: post  
title: "[iOS] - 키보드 prediction bar 없애기"  
subtitle: ""  
categories: app
tags: app-ios keyboard 
comments: true  
header-img: 

---  
  
> 키보드 상단에 나타나는 autocorrect / spellchecking 부분 바 없애기

---


<img width="316" alt="image" src="https://user-images.githubusercontent.com/41438361/161877758-a64b64a9-e5ae-4007-ac78-4cdd27fdce4e.png">

키보드 위쪽에 나타나는 저 부분을 없애려고 한다. 저 부분을 정확하게는 "predictive text", "predictive bar", 혹은
"suggestion bar" 라고 부르는 것 같다. 


코드 상에서는 아래와 같이 해주면 된다. 키보드를 뜨게하는 텍스트 객체가 있을 거고, 예를 들어 그 텍스트 객체가 textView라고 해보자.

```swift
textView.autocorrectionType = .no
textView.spellCheckingType = .no
```

어떤 기기에서는 `autocorrectionType`만 no로 설정해주면 predictive bar가 없어지는데, 어떤 기기에서는 `spellCheckingType`까지
no로 해줘야 predictive bar가 없어지는 걸 확인할 수 있었다.
