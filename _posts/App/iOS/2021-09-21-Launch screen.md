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

중요한 것은, **launch screen을 위해 정적 이미지를 사용하면 안된다. iOS 14 이상부터 launch screen은 25MB로 크기가 제한되어 있다.**

* 앱의 첫 화면이 될 Launch screen은 꼭 하나만을 디자인 해야 한다. 만약 앱이 켜지는 게 끝났을 때 다 다르게 보이는 요소들을 포함시킨다면 사람들은 launch screen과 첫 화면 사이에 별로 보기 좋지 않은 flash 현상을 보게 된다. 또한 launch screen이 기기의 appearnce mode(dark mode 등)과 일치하게 해야 한다.
* Launch Screen에 텍스트를 포함시키지 않는다. Launch Screen에 있는 컨텐츠는 변하지 않기 때문에, 화면에 있는 텍스트는 지역에 따라 바뀌지 않는다.
* 사람들은 빠르게 컨텐츠에 접근하고 작업을 수행할 수 있는 앱을 가치있다고 생각한다. 앱의 인터페이스를 닮은 launch screen을 디자인하면 앱이 즉시 시작되는 것과 같은 느낌을 준다. 빠르게 켜지는 것과 더해져서, 이런 디자인은 앱이 즉각적으로 반응한다는 느낌을 주게 한다. 게임으로 따지면, launch screen은 게임이 처음 보여주는 화면으로 부드럽게 전환되어야 한다.
* 광고를 넣지 말아야 한다. Launch screen은 브랜딩 기회로 쓰는 곳이 아니다. 처음 보게 되는 화면이 splash screen이나 "about" 창과 같이 보여지게 디자인 하면 안된다. 

애플 문서에서는 위와 같이 얘기한다. 하지만 대개 처음 앱을 실행하게 되면 떠올리는 화면은 앱을 대표하는 로고가 있는 화면이 나오고, 그 화면이 보여진 상태로 로딩이 이루어졌다가 앱의 첫 화면이 보이는 그런 형태를 띈다. 

Xcode에서는 기본적으로 내가 loading screen으로 설정할 수 있는 스토리보드를 생성한다. 이게 거의 항상 잘 작동하긴 하는데, Xcode 12와 iOS 14에 새롭게 추가된 옵션들이 있다.

## The history of launch screens

Launch screen은 Xcode 6 이전에는 asset 카탈로그에 정적 이미지를 제공해서만 만들 수 있었다. Xcode 6와 함께 iOS 8에서는 이 정적 이미지를 스토리보드 파일로 대체할 수 있게 되었다.


참조

* https://developer.apple.com/design/human-interface-guidelines/ios/visual-design/launch-screen/
* https://developer.apple.com/documentation/uikit/app_and_environment/responding_to_the_launch_of_your_app
* https://www.avanderlee.com/xcode/launch-screen/
