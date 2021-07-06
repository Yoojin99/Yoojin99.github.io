---  
layout: post  
title: "[iOS] - UIVIew isHidden, 감출 때 여백 없애기"  
subtitle: ""  
categories: app
tags: app-ios uiTableView style
comments: true  
header-img: 

---  
  
> `UIView에서 isHidden 프로퍼티를 설정해서 뷰를 감추고 다시 표시할 수 있다. 하지만 원래 뷰가 차지하던 자리게 여백이 생기게 된다. 이를 해결하기 위한 방법을 알아보자.`  

---

UIView에는 `isHidden` 의 값을 설정해서 뷰를 감추고 나타낼 수 있다.

![image](https://user-images.githubusercontent.com/41438361/124620868-c5dce600-deb4-11eb-915d-130235e5afe0.png)

내가 원했던 기능은 위와 같이 아래의 뷰를 감추면 그에 따라 상위 컨테이너의 높이가 자동으로 조정되는 것이었다. 하지만 실제로 hide를 하니 아래와 같이 감춘 뷰 부분에 빈 여백이 생기게 되었다.

![image](https://user-images.githubusercontent.com/41438361/124621100-f58bee00-deb4-11eb-8ae2-ceca1785a292.png)

결론부터 말하자면, 위의 두 뷰를 UIStackView 안에 넣어야 한다.

Constraint와 height의 값을 여러번 조정해가며 설정해보았지만, 아직까지 원하는 대로 기능하게 하는 방법은 찾지 못했다. 그래서 아래와 같이 뷰 계층을 구성해야 했다.

![image](https://user-images.githubusercontent.com/41438361/124621322-24a25f80-deb5-11eb-9bd2-76c2cb97ec6a.png)

[Stack Overflow](https://stackoverflow.com/questions/30638142/swift-how-to-hide-element-and-its-space)의 한 글을 보면, 뷰를 hide하는 것은 단순히 visible 하지 않게 만드는 것이지, 뷰 자체를
없애는 것이 아니라고 한다.

