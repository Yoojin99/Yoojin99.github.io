---  
layout: post  
title: "[iOS] - Meet passkeys"  
subtitle: ""  
categories: app
tags: app-ios wwdc
comments: true  
header-img: 

---  

[Meet passkeys](https://developer.apple.com/videos/play/wwdc2022/10092/)

Passkey를 보기 전에, 현재 사용되는 인증 기술 password를 먼저 보자.

<img width="784" alt="image" src="https://user-images.githubusercontent.com/41438361/173512611-4828fdd6-4842-4d9a-8ec8-b2f98cfa0df2.png">

비밀번호를 안전하게 사용하는 건 어렵다. 앱과 웹사이트를 디자인할 때 안전과 좋은 사용자 경험간의 교환이 일어나는 상황은 빈번하게 발생한다. 모든 것이 완벽하게 동작해도 피싱이나 비밀번호 재사용은 account compromise를 발생시킬 가능성이 있다.

# Passkeys

<img width="845" alt="image" src="https://user-images.githubusercontent.com/41438361/173513247-7e989dd6-5cc9-469c-ba7d-d0bef4249f26.png">

이런 문제를 개발자 관점에서 해결할 수 있는 passkey가 나왔다. macOS Ventura와 iOS 16에서 사용할 수 있다.

* 비밀번호보다 더 나은 사용자 경험을 제시한다.
* 안전 문제가 발생하지 않는다. 심지어 영상에서는 불가능하다고 말한다.
  * 재사용되는 credential
  * credential 노출
  * 피싱

## Passkey 생성

<img width="509" alt="image" src="https://user-images.githubusercontent.com/41438361/173514055-fe550902-f42d-498f-89b2-a771737b25bc.png">
<img width="506" alt="image" src="https://user-images.githubusercontent.com/41438361/173514120-28b1a6c8-0051-417a-b935-968a9e5a0899.png">

매우 간단하게 passkey를 추가할 수 있다. 기기가 유일하면서도 안전한 key 쌍을 생성해서 iCloud Keychain에 저장했고, 모든 기기에서 동기화가 된다.

## Passkey 사용

<img width="491" alt="image" src="https://user-images.githubusercontent.com/41438361/173514562-617f9c3b-13e2-4a80-8131-200cecfd5564.png">
<img width="494" alt="image" src="https://user-images.githubusercontent.com/41438361/173514696-a574b850-ba46-49f3-832a-50cc23584f46.png">

QuickType 바에 passkey가 뜨고, 이를 탭하면 바로 로그인할 수 있다. 이를 통해 새로 비밀번호를 생성하거나 요구되는 비밃번호 조건들을 맞추기 위해 노력할 필요가 없다.

* 각 passkey는 시스템에 의해 생성되고 한 계정에서만 사용되도록 보장된다. 
* 내장된 phishing resistance로 올바른 앱이나 웹사이트에서만 사용될 수 있게 한다.
* 표준을 준수하기 때문에 다른 플랫폼이나 크로스 플랫폼에서도 곧 사용할 수 있을 것이라고 한다.

<img width="1656" alt="image" src="https://user-images.githubusercontent.com/41438361/173515608-49fd0d14-34ad-4d1f-8684-5088afaacb4d.png">

다른 사용자의 기기에서도 passkey를 사용할 수 있다. 사용자 이름등을 사용해서 로그인 시도를 할 때 내 passkey를 사용하기 위한 qr 코드가 생성되고, 이를 내 폰으로 스캔하면 passkey로 로그인 할 수 있다.

과정은 간단하지만 아래의 과정들이 같이 일어난다.

* local key agreement
* proximity 인증
* end-to-end encrypted communication 채널 생성

## Share

두 사람 이상이 계정을 공유할 수도 있다.

<img width="1081" alt="image" src="https://user-images.githubusercontent.com/41438361/173516528-909384b0-c063-41b7-8336-febc77f532fb.png"><img width="1079" alt="image" src="https://user-images.githubusercontent.com/41438361/173516628-b8f73c04-6b37-44de-b4ef-7c7e54d897e9.png">

1. Account details에서 shared account를 선택한다.
2. Airdrop으로 passkey를 공유할 수 있다.

# Designing for passkeys

<img width="1401" alt="image" src="https://user-images.githubusercontent.com/41438361/173517572-077e1c67-09e1-412a-bc3a-ccb4c0e734ad.png">

Passkey : 비밀번호를 대체하는 것. 로그인하고, 사용하기 쉽고, 안전하다.

* "Passkey" : 일반적, 사용자 관점의 단어. "password"라는 단어 대신에 사용할 수 있다. 
* 애플 플랫폼에서는 이를 나타낼 SF Symbol `person.key.badge`와 `.fill` variant가 있다.

<img width="924" alt="image" src="https://user-images.githubusercontent.com/41438361/173521893-0a53cf72-098c-40bf-87ef-ebdd13a38395.png">
<img width="1666" alt="image" src="https://user-images.githubusercontent.com/41438361/173522183-c0e7b438-8170-4ef2-809b-51c3e4087d70.png">

앱이나 웹사이트에서 전체 인터페이스를 개발할 필요도 없다. 

* AutoFill을 이용해서 passkey를 보여줄 수 있다.
* 애플 플랫폼에서는 passkey를 이용해서 로그인할 때 사용할 수 있는 추가적인 UI 옵션들을 제공한다.

# Passkeys and AutoFill

<img width="719" alt="image" src="https://user-images.githubusercontent.com/41438361/173524182-3448ea83-1c4d-4432-a1bd-6d574e052960.png">

* WebAuthentication/WebAuthn 표준에 기반한다.
* public-key cryptography를 사용한다. 유일한 cryptographic key 쌍이 모든 계정마다 생성된다.
* passkey sign-in을 지원하려면 백엔드 서버에 WebAuthn을 적용해야 한다. 표준 WebAuthn을 구현한 서버는 passkey를 사용할 수 있다.

<img width="1069" alt="image" src="https://user-images.githubusercontent.com/41438361/173524821-079c79e9-80c5-4894-8262-6e3139a26a48.png">

* 애플 플랫폼의 앱에서 passkey는 AuthenticationServices 프레임워크의 ASAuthorization API에 속한다.
* 

# Streamlining sign-in

# How passkeys work

# Multi-factor authentication


ㄴDfas;lkfdj;laskdf
