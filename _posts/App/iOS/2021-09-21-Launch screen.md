---  
layout: post  
title: "[iOS] - Launch Screen 만들기"  
subtitle: ""  
categories: app
tags: app-ios launchScreen 
comments: true  
header-img: 

---  
  
> `앱을 실행했을 때 뜨는 Launch Screen을 만들어보자.`  

---

## Launch Screen?

Launch Screen은 앱을 실행했을 때 잠깐 나타났다가 사라지는 화면이다. 앱을 키자마자 나타났다가 앱의 첫 번째 화면으로 빠르게 전환되는데, 이는
앱이 빠르고, 반응성 있는 인상을 줄 수 있다. Launch Screen은 단순히 미적인 요소 때문이 아니라, 앱이 실행되는 것이 빠르고, 즉시 사용할 수 있을 것처럼 보이게 
의도된 것이다. 

![image](https://user-images.githubusercontent.com/41438361/134183166-6cc6365b-ae8f-43e9-b86c-532236e0df5b.png)

다양한 크기의 화면에서 launch screen을 띄우려면 Xcode의 스토리보드를 사용하면 된다. 하나의 스토리보드로 모든 launch screen을 관리할 수 있다.
중요한 것은, launch screen을 위해 정적 이미지를 사용하면 안된다. iOS 14 이상부터 launch screen은 25MB로 크기가 제한되어 있다. 또한 launch screen
이 기기의 노출 모드




참조

* https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/launch-screen/
* https://developer.apple.com/documentation/uikit/app_and_environment/responding_to_the_launch_of_your_app
