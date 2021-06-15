---  
layout: post  
title: "[iOS] - UIScrollView의 ContentOffset, ContentInset, ContentSize"  
subtitle: ""  
categories: app
tags: app-ios 
comments: true  
header-img: img/dev/app/ios/XcodeImg.png

---  
  
> `ContentOffset, ContentInset, ContentSize 차이에 대해 알아보자.`  

---

UIScrollView의 프로퍼티 중 `contentInset`, `contentOffset`, `contentSize`가 가장 많이 사용되는 것들 중 하나일 것이다. 

## [contentOffset](https://developer.apple.com/documentation/uikit/uiscrollview/1619404-contentoffset)

정의 : scroll view 의 origin에서 content view의 origin까지 얼마나 떨어졌는지를 나타내는 포인트

즉 사용자가 영역을 스크롤 한 후에 있는 지점을 의미한다. 그래서 사용자가 스크롤을 할 때마다 이 지점이 바뀌게 된다. 만약 주어진 포지션이 존재할 때, 해당 위치로 스크롤하기 위해서 코드 상으로 설정될 수 있다.

```swift
scrollView.setContentOffset(CGPoint(x: 50, y: 50), animated: true)
```

## [contentInset](https://developer.apple.com/documentation/uikit/uiscrollview/1619406-contentinset)

정의 : content view의 Inset이 safe area 나 scroll view에서 얼마나 떨어졌는지를 의미한다.

즉 `contentInset`은 UIScrollView에서 `innnerView`까지 얼마나 마진이 떨어져있는지를 의미한다. `childView`에 내부 여백을 준다고 생각하면 된다. 이 프로퍼티는 오로지 코드 상으로만 설정될 수 있다.

```swift
scrollView.contentInset = UIEdgeInsets(top: 7, left: 7, bottom: 7, right: 7)
```

## [contentSize](https://developer.apple.com/documentation/uikit/uiscrollview/1619399-contentsize)

정의 : 컨텐츠 뷰의 사이즈

`contentSize`는 `UIScrollView`의 컨텐츠의 사이즈이고, `scrollView`가 얼마나 길어질 수 있는지를 나타낸다. 이것은 pagination처럼 동적이 될 수도 있고 contact list처럼 정적이 될 수도 있다. 런타임에서 값이 바뀔 수도 있다.

```swift
scrollView.contentSize = CGSize(width: self.view.frame.size.width, height: 500)
```

![image](https://user-images.githubusercontent.com/41438361/121978571-73337100-cdc3-11eb-853b-1f78d3bb086c.png)


출처

* https://betterprogramming.pub/contentoffset-contentinset-and-contentsize-of-a-uiscrollview-5ae8beb0f1db
* 
