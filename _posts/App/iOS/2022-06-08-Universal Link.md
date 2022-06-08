---  
layout: post  
title: "[iOS] - Universal Link"  
subtitle: ""  
categories: app
tags: app-ios universal link
comments: true  
header-img: 

---  

# Universal Link

## Universal Link란?

[WWDC 2019](https://developer.apple.com/videos/play/wwdc2020/10098/)에 나온 유니버셜 링크의 정의는

**웹, 앱에 모두 속하는 컨텐츠를 표현하는 HTTPS, HTTP URL이다.** 

Universal link는 **웹 브라우저 대신 앱에서 컨텐츠를 열게 해준다.** 이 말은 **앱 내의 컨텐츠로 연결할 수 있는 링크**라고도 할 수 있다. Universal link를 사용해서 앱 내의 특정 컨텐츠를 열 수 있다.

<img width="511" alt="image" src="https://user-images.githubusercontent.com/41438361/172602089-8b587a0d-b267-46cf-83cb-ac0230bf2f1b.png">

사용자가 universal link를 탭하거나 클릭할 때, 시스템은 Safari나 웹 사이트를 통해 라우팅하지 않고 바로 앱으로 리다이렉트한다. Universal link가 표준 HTTP / HTTPS 링크이기 때문에, 한 URL이 웹사이트와 앱에 동시에 적용될 수 있다.

* 앱이 설치되어 있지 않은 경우 : 시스템이 Safari에서 URL을 연다.
* 앱이 설치되어 있는 경우 : 웹사이트가 자기 대신에 앱이 URL을 열게 허용하는지를 검증하기 위해 웹 서버에 저장된 파일을 확인한다. 이 웹사이트와 앱 사이의 연결을 정의하는 파일은 서버에만 저장할 수 있다.

## Universal Link 지원하기

1. 앱과 웹사이트 사이의 양방향 연결을 생성하고 앱이 처리할 수 있는 URL을 명시한다.
2. 




Universal link를 생성하기 위해서는

1. Associated Domains entitlement라는 특별한 자격을 앱에 추가하고
2. 웹 서버에 JSON 파일을 추가한다.

이 entitlement는 웹 서버의 도메인 이름을 언급하고, 웹 서버는 앱의 application identifier를 언급한다. 이를 통해 앱과 웹사이트 사이에 안전한 양방향 연결을 생성할 수 있고, 이를 통해 웹사이트 대신에 앱이 작업할 수 있게 된다.


* 참조
* https://developer.apple.com/documentation/xcode/allowing-apps-and-websites-to-link-to-your-content?preferredLanguage=occ
* https://developer.apple.com/videos/play/wwdc2020/10098/?time=31
* https://developer.apple.com/documentation/Xcode/supporting-associated-domains
* https://developer.apple.com/documentation/xcode/supporting-universal-links-in-your-app




sdfsdfasdf;l;lk
