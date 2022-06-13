---  
layout: post  
title: "[iOS] - WWDC22 What's new in Xcode"  
subtitle: ""  
categories: app
tags: app-ios wwdc
comments: true  
header-img: 

---  

[What's new in Xcode](https://developer.apple.com/videos/play/wwdc2022/110427/?time=1222)

# 속도 향상

Xcode 14가 30% 작아짐. 다운로드 속도 향상. 

# SwiftUI with live previews

<img width="690" alt="image" src="https://user-images.githubusercontent.com/41438361/173286074-f284eb65-2801-47e9-b9e7-273b7391ca0e.png">

* 기본적으로 preview canvas는 이제 상호작용할 수 있게 설정됨. 
* 추가적인 코드 작성 없이 canvas에서 각 preview에 대한 추가적인 설정을 할 수 있다. (color scheme, text size, deivce orientation 등)
* 다양한 사이즈 화면에서 뷰가 어떻게 보이는지 확인 가능

# initializer 자동 완성

<img width="895" alt="image" src="https://user-images.githubusercontent.com/41438361/173286517-8fb5fd2f-8aed-4d29-857d-ad03269d4fa4.png">

Xcode 14에서는 Initializer를 입력하기 시작할 때 자동완성된 이니셜라이저를 제공한다.

# SF Symbol

<img width="680" alt="image" src="https://user-images.githubusercontent.com/41438361/173286777-68b221b8-07f9-495a-a5b5-0df20f4c8f63.png">

라이브러리가 이제 모든 SF Symbol을 포함하고 있다.

# code completion

<img width="538" alt="image" src="https://user-images.githubusercontent.com/41438361/173287059-47c72af0-a161-4933-a976-acc8bb555460.png">
<img width="1012" alt="image" src="https://user-images.githubusercontent.com/41438361/173287109-5f48ffbf-1e44-45aa-bff2-80684f9e56d5.png">

쓰고 싶은 파라미터 등을 적어서 쓰고 싶은 함수 등을 바로 찾을 수 있다.

# callers

<img width="1198" alt="image" src="https://user-images.githubusercontent.com/41438361/173291063-5dd42618-63cb-4d2e-bcb8-edbf45a3c789.png">

특정 함수를 호출하는 파일, 함수들의 목록을 보여준다.

# Error dimming

<img width="1421" alt="image" src="https://user-images.githubusercontent.com/41438361/173291224-b393cb01-4a7f-42ea-9211-3f5c89b6948d.png">
<img width="1414" alt="image" src="https://user-images.githubusercontent.com/41438361/173291260-496318ed-88f9-461c-9c31-eb4f79dbea0b.png">

에러를 고치는 즉시 에러가 회색이된다. 이를 통해 Xcode가 에러를 재진단 중인 것을 알 수 있다. 잠시 기다리면 에러가 아예 화면에서 사라지고 Xcode가 모든 에러가 해결되었음을 확인한다.

# show definitions containing the visible code

<img width="1063" alt="image" src="https://user-images.githubusercontent.com/41438361/173291604-3e5bcb2c-bb2f-4ab0-b51d-6cf4c4ce6368.png">

현재 보여지는 코드의 정의가 화면상 보이지 않는 위치에 있을 때 이제 이를 화면에 보여준다.

# Build performance improvements

## Building Framework and Application

<img width="1705" alt="image" src="https://user-images.githubusercontent.com/41438361/173291874-6e2fc586-2160-41e6-b156-39e3d08fb893.png">

Xcode가 여러 target(framework, application)을 빌드학 때 일어나는 과정은 다음과 같다.

1. 프레임워크 소스 컴파일
2. 모듈 생성
3. 애플리케이션 소스를 link, compile
4. Link

<img width="1684" alt="image" src="https://user-images.githubusercontent.com/41438361/173292149-031e16ab-2a06-433f-aeee-9d385c99fc94.png">

Xcode 14는 이런 과정을 재배치해서 병행성을 향상시킨다. 이런 향상된 병행성을 통해 linker가 2배 빨라졌고, Xcode 14는 25% 빌드 시간이 단축됐다.

<img width="1119" alt="image" src="https://user-images.githubusercontent.com/41438361/173292425-243eb3f2-2540-48e1-aed8-213cd4954237.png">

긴 synchronous 작업을 수행할 때도 있는데, 시각화하지 않고서는 이런 작업이 일어나고 있다고 말하기 힘들다. Xcode 14에서는 빌드 로그나 결과 번들의 빌드 타임라인을 보여준다.

=> Demystify parallelization in Xcode builds, Link Fast: Improve build and launch 세션에서 더 알아볼 수 있다. 

## Faster test execution

<img width="331" alt="image" src="https://user-images.githubusercontent.com/41438361/173292842-dc635ae8-af30-46cf-9970-409608b7b649.png">

Xcode 14는 테스트 도중에 타겟과 테스트 클래스 사이의 scheduling 의존성을 제거해서 테스트 실행시간을 30% 빠르게 만든다.

=> Author fast and reliable tests for Xcode Cloud 세션에서 더 알아볼 수 있다.

## Faster notarizing

<img width="282" alt="image" src="https://user-images.githubusercontent.com/41438361/173293110-7053a2c9-1ba8-4aaf-9e48-eef3f3c66097.png">

문서화하는 것도 Xcode 14에서 4배 빨라졌다.

## Interface Builder performance improvements

* Document loading은 50% 빨라졌다.
* device bar에서 iPhone과 iPad간 바꾸는게 30% 빨라졌다.

# Support Easy platform setting

<img width="1131" alt="image" src="https://user-images.githubusercontent.com/41438361/173293575-f32b022b-1909-4c13-aba5-0680c452cf7d.png">

하나의 target으로 어떤 플랫폼을 지원할 것인지 정의할 수 있다. 

=> Use Xcode to build a multiplatform app 세션에서 더 알아볼 수 있다. 

# Make app smaller

<img width="1118" alt="image" src="https://user-images.githubusercontent.com/41438361/173293869-429a7143-3595-4a15-a7d1-fedd40ed98a4.png">

Memory debugger : 애플리케이션에서 메모리 누수를 확인하는 툴. 객체의 모든 참조 경로를 확인할 수 있다. 

# Swift Package plugins

<img width="1122" alt="image" src="https://user-images.githubusercontent.com/41438361/173294134-f3551cb2-2931-450f-9d2d-5fe171cdea5d.png">

Xcode자체를 swift Package plugin으로 확장할 수 있다. 

=> Meet Swift Package plugins 세션에서 더 확인할 수 있다. 

# Localize package resources

<img width="1240" alt="image" src="https://user-images.githubusercontent.com/41438361/173294449-b0a783c6-347f-46f5-8226-167596da5187.png">

애플리케이션과 마찬가지로 package resouce를 localize할 수 있다. 

=> Building global apps: Localization by example 세션에서 더 확인할 수 있다.

# Updated run destination chooser, scheme chooser

<img width="264" alt="image" src="https://user-images.githubusercontent.com/41438361/173294933-724d4ad6-2b94-4f7d-94b5-72e2bb081534.png">

* 최신에 사용한 것을 보여준다.
* 디바이스 이름을 검색할 수 있다.

# Organizer window

## Feedback

<img width="1792" alt="image" src="https://user-images.githubusercontent.com/41438361/173295329-af0c184a-f07b-4cbe-972f-3766e8860279.png">

* Xcode에서 TestFlight 피드백을 보여준다.
* 테스트한 사람의 정보, 디바이스 설정을 보여준다.
* 바로 테스터에게 이메일을 보낼 수 있다.
* 베타 유저에게서 받는 피드백.

## Hangs

<img width="1790" alt="image" src="https://user-images.githubusercontent.com/41438361/173296098-a41c956c-d3dd-43dc-8783-5b64009751d3.png">

앱이 사용자 입력을 받기 위해 잠시 쉬지 않고 메이 쓰레드를 사용할 때 앱이 hang된다고 한다. 앱이 hang되면 사용자는 앱이 반응성이 떨어진다고 생각한다.

Hangs report를 통해 앱스토어 사용자가 겪는 영향력이 큰 hang들을 보여준다. 이를 통해 코드를 재구성할 수 있다.

* hang들의 목록
* 문제가 되는 코드 보여줌, 코드로 바로 이동할 수 있음

=> Track down hangs with Xcode and on-deivce detection 세션에서 더 확인할 수 있다.

# Single Size icon

<img width="1332" alt="image" src="https://user-images.githubusercontent.com/41438361/173296937-c9de8bca-1e17-48a9-b92c-fba987d51104.png">

Xcode가 한 이미지에서 자동으로 모든 다른 사이즈를 생성할 수 있게 할 수 있다.

## Hangs

ㄴDf
asdfasdf
