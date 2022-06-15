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
* 다양한 권한을 다룬다. 비밀번호, security key, Sign in with Apple.
* AutoFill 지원과 같이 새로 사용할 수 있는 메서드가 추가됨.

<img width="1094" alt="image" src="https://user-images.githubusercontent.com/41438361/173715552-af892b5f-770a-46c4-9cfe-b6cecdb8e9bf.png">

Passkey를 앱에서 사용하려면 webcredential 서비스를 이용해서 associated domain을 설정해야 한다.

=> Introducing Password AutoFill for Apps와 What's new in Universal Links 세션에서 확인할 수 있다.

<img width="1587" alt="image" src="https://user-images.githubusercontent.com/41438361/173715662-df594c64-11a2-49f9-9b93-b49dd18aeb4a.png">

사용자 이름을 입력하는 부분이 `username` textContentType을 사용하고 있어야 한다. 이를 통해 어디에서 passkey 추천을 할 것인지 시스템이 알 수 있다.

<img width="1065" alt="image" src="https://user-images.githubusercontent.com/41438361/173715827-92437ec9-602c-45a4-976f-8db7d55d91f4.png">

1. `challenge` : `WebAuthn` request를 통해 서버에서 받은 것
2. `provider` : passkey request를 위해 동작하는 ASAuthorizationProvider
3. `request` : 존재하는 passkey로 로그인하기 위한 assertion request. WebAuthn에서는 로그인하기 위해 assertion이 필요하다.
4. `controller` : request를 실제로 다루는 것. passkey request로 인스턴스를 생성하고, delegate와 presentationContextProvider를 설정한다.
5. `performAutoFillAssistedRequest()` : request를 시작한다.

<img width="479" alt="image" src="https://user-images.githubusercontent.com/41438361/173716266-68247b84-fc03-44ea-a2d8-cfd38cf90aca.png">

이 request가 앱에서 동작하고 있는 한 사용자 이름 필드가 포커싱되면 시스템은 QuickType 바에 사용 가능한 passkey를 제공한다. **이 request가 사용자 이름 필드가 포커싱 되기 전의 뷰 라이프사이클에서 시작되게해서 키보드가 뜰 때 passkey가 준비되어 있게 해야 한다.**

QuickType 바의 아이템이 선택되면 Face ID를 인식하고, 로그인을 마무리하기 위해 ASAuthorizationController delegate 콜백을 받는다. 

<img width="1251" alt="image" src="https://user-images.githubusercontent.com/41438361/173716610-70c9ed9b-90e2-4f9e-b63d-e33639060afc.png">

인증이 성공하면 `didCompleteWithAuthorization` 콜백을 받는다.

1. 받은 credential의 타입을 확인한다. passkey 로그인의 경우 `ASAuthoriztionPlatformPublicKeyCredentialAssertion`이다.
2. Assertion 객체는 로그인하기 위해 백엔드에서 필요한 필드들을 포함한다. 
3. 이 값들을 읽고 서버에서 검증한다음 로그인을 완료할 수 있다.

## AutoFill-assisted requests

<img width="433" alt="image" src="https://user-images.githubusercontent.com/41438361/173716978-2518973d-2c4e-4fec-bf34-727ce0d31576.png"><img width="430" alt="image" src="https://user-images.githubusercontent.com/41438361/173717060-5591308c-ea4b-44bb-ad79-ea19462c1d4f.png">

1. 근처 기기에서 passkey 로그인을 하도록 할 수 있다. 이 경우 위에서 사용한 코드를 그대로 사용하면 된다.
2. 사용자가 passkey가 없는 경우 로그인 폼을 사용하면 된다. QuickType 바에서 비밀번호 추천을 받거나 필드에 직접 비밀번호를 입력할 수 있다. 

<img width="1576" alt="image" src="https://user-images.githubusercontent.com/41438361/173717477-5068e25e-9522-43c5-99b8-cb922aa1dc44.png">

이미 passkey를 사용하는 사용자가 AutoFill에서 추천하는 것 대신에 직접 사용자 이름을 입력했다면 AutoFill request를 취소하고 ASAuthorizationController를 사용해서 modal passkey 로그인 시트를 보여주게 해야 한다. AutoFill request를 modal request로 바꾸러면 `performAutoFillAssistedRequests` 메서드를 호출하는 부분을 `performRequests()`로 바꿔주면 된다.

이를 통해 사용 가능한 모든 passkey를 보여주는 modal sheet를 보여줄 수 있다.

## User Verification

<img width="1333" alt="image" src="https://user-images.githubusercontent.com/41438361/173717970-96a12849-ce6c-44ef-b2aa-4a0e84b6d66c.png">

WebAuthn을 사용할 때 알아둬야 할 것은 Apple 플랫폼이 사용자 식별(UV)를 어떻게 하느냐다. UV는 WebAuthn response의 Boolean 필드로 현재 사용자가 기기의 주인인지 authenticator가 확인한 결과다. 

애플 기기에서는 바이오매트릭스나 비밀번호, passcode를 나타내는 값을 사용한다. 애플 플랫폼은 바이오매트릭스를 사용가능할 때 항상 passkey를 사용할 때 UV를 요구하게 되어 있으므로 이를 걱정하지 않아도 된다. 

WebAuthn request를 만들 때 uv 요구사항을 명시하는 옵션이 있다. 기본 값은 `userVerification: "preferred"`다. 항상 기본값을 상요해서 바이오매트릭스가 없는 기기에서 발생할 수 있는 문제를 피해야 한다.

# Streamlining sign-in

<img width="1066" alt="image" src="https://user-images.githubusercontent.com/41438361/173718667-d12ca174-dd32-4ace-baff-88b24a20a3c5.png">

AutoFill 로그인에 추가로 ASAuthorization API는 다양한 기능을 제공함.

## Using passkey allow lists

<img width="1670" alt="image" src="https://user-images.githubusercontent.com/41438361/173719927-fd3c5053-5286-436e-b5a1-0c0d00743e01.png">

사용자 이름이 입력된 후 modal passkey 시트를 띄울 때, 기기에 저장된 여러 계정에 대한 passkey 목록을 보여주는 것이 기본 동작이다. 이 목록을 보여줄 때 조건에 일치하는 계정에 대한 passkey만 보여주도록 제한할 수 있다.

<img width="1437" alt="image" src="https://user-images.githubusercontent.com/41438361/173720149-6456abaa-8e4d-4504-abd8-76e8d1a1da2b.png">

`credentialIDs` : 사용자 이름을 사용해서 일치하는 credential ID들을 배열로 만든 것. Credentail ID는 passkey의 유일한 식별자다. Webauthn 서버는 주어진 사용자 이름으로 credential ID를 확인할 수 있는 방법을 알고 있어야 한다.

## Silent fallback requests

<img width="1659" alt="image" src="https://user-images.githubusercontent.com/41438361/173720849-77a50c4a-a580-47b6-8937-7f4ccdc5b6a7.png">

passkey가 없는 경우 modal passkey request를 생성할 때 일어나는 동작들. Allow list를 사용하는데 일치하는 passkey가 없을 경우에도 적용된다. 

* Default : 근처 기기의 passkey로 로그인할 수 있는 QR 코드를 보여준다.
* delegate callback을 통해 다른 방법으로 로그인하게 할 수 있다. 전통적인 로그인 폼을 보여주기 전에 존재하는 credential을 보여주게도 할 수 있다.

<img width="1420" alt="image" src="https://user-images.githubusercontent.com/41438361/173720929-f00f4637-d487-40ea-b681-6243ebc7a3f1.png">

만약 `preferImmediatelyAvailableCredentials` 옵션을 사용하면 modal sheet를 띄울 때 QR 코드를 보여주는 것 대신에 에러와 함께 delegate callback을 받게 된다.

<img width="1538" alt="image" src="https://user-images.githubusercontent.com/41438361/173721076-a1a0d7f1-acb2-4d1b-9e23-d80cdad5458e.png">

* 사용자가 시트를 보고 이를 직접 해제한 경우
* preferImmediatelyAvailableCredential 옵션을 전달했는데 사용가능한 credential이 없는 경우

ASAuthorizationError를 받게 된다. 위 코드에서는 문맥에 따라 다른 동작을 처리하게 하면 된다. 가령 일반적인 로그인 폼을 보여주기 전에 로컬 credential을 테스트하는 옵션으로 이를 사용할 수 있고, 위 코드의 `showSignInForm` 메서드를 통해 폼을 보여줄 수 있다. 

<img width="1292" alt="image" src="https://user-images.githubusercontent.com/41438361/173733254-4b2d8331-da12-4037-9302-32ddba215e5e.png">

기기에 조건을 만족하는 credential이 한 개 이상 있는 경우 전체 modal sheet가 옵션에 상관 없이 보여진다. 영상에서는 default fallback와 함께 AutoFill-assisted request나 modal request를 함께 사용해서 현재 기기에 passkey가 없는 경우 근처 기기에서 로그인할 수 있게 하는 옵션도 사용할 수 있게 하라고 권고한다.

## Combined credential requests

<img width="1667" alt="image" src="https://user-images.githubusercontent.com/41438361/173733524-2709c65e-3422-4bff-bcfd-8e98ff619b32.png"><img width="442" alt="image" src="https://user-images.githubusercontent.com/41438361/173733804-04026ba8-208d-41bf-88ab-a17d638793d0.png">

가령 앱이 passkey, password, Sign in with Apple request를 생성했다 하자. 기기는 세 개의 다른 계정에 저장된 세 개의 다른 credential을 가지고 있고 화면에 이것들이 모두 보여진다. 하지만 대부분의 경우에는 한 사람당 한 명의 계정을 갖고 있을 확률이 높다. 이 경우에 combined credential request는 시트에 오로지 한 계정만을 보여준다.

<img width="1590" alt="image" src="https://user-images.githubusercontent.com/41438361/173733781-dd66d4cc-4a61-47cf-b777-a618e4db61c8.png">

존재하는 ASAuthorization request에 credential type을 추가하는 것은 간단하다.

* `passwordRequest`, `signInWithAppleRequest` 부분 : 추가되는 request type을 위한 provider와 request를 생성한 다음
* `controller` 부분 : 위에서 생성한 것들을 controller에 전달한다.

이러면 modal sheet는 해당 credentail type에서 사용 가능한 모든 credential을 보여줄 수 있다.

<img width="1502" alt="image" src="https://user-images.githubusercontent.com/41438361/173734144-b0844bb0-2ba6-4d6d-a103-66b812b41c9d.png">

어떤 타입의 credential type이 사용됐는지와는 상관 없이 같은 delegate callback을 받는다. 내가 받은 credential type을 확인하고 해당 타입에 따라 적절하게 로그인을 처리한다.

# How passkeys work

## Password sign in

<img width="1664" alt="image" src="https://user-images.githubusercontent.com/41438361/173734797-aa32afd3-1a4e-4211-8873-ba7b0579e371.png">

비밀번호로 로그인할 때, 비밀번호를 입력하면 hashed 되고 salted 된다.(salted 된다는 것은 찾아보니 비밀번호 보안과 관련되어 사용되는 용어라고 한다. 음식에 소금을 치는 것처럼 비밀번호에도 특별한 작업을 한다는 느낌인 것 같다.) 이 보안 처리가 된 결과가 서버에 저장된다. 이후에 같은 hashed, salted 값을 생성할 수 있으면 계정을 사용할 수 있는 것이다. 이렇게 서버에 저장되는 값은 공격받기 쉽다. 

만약 해커들이 이 값을 가지게 되면 비밀번호가 무엇인지 유추할 수 있고 계정에 접근할 수 있게 된다.

## Passkey sign in

<img width="1674" alt="image" src="https://user-images.githubusercontent.com/41438361/173735675-9888081a-ccee-405c-83c6-3360c64c8931.png">

Passkey는 연관된 키의 쌍을 가지고 있다. 이 키는 기기에 의해 안전하고 유일하게 생성된다.

<img width="1707" alt="image" src="https://user-images.githubusercontent.com/41438361/173735767-64ca98ba-4bc4-4e97-ab33-98227725d42c.png">

* private key : 기기에 저장되어 있다. 실제로 로그인하기 위해 필요한 키.
* public key : 서버에 저장되어 있다. 말 그대로 public하다.

서버는 private key가 무엇인지 절대로 알지 못하고, 기기는 private key를 안전하게 보관한다. 로그인할 때 다음의 절차가 이루어진다.

1. 서버가 기기에게 일회성의 challenge를 보낸다. 애플 플랫폼의 passkey는 표준 ES256 challenge response 알고리즘을 사용한다.
2. Private key를 통해 challenge에 대한 유효한 해답, signature를 로컬에서 생성하고 이를 서버에 보낸다. Private key는 기기에만 있고 다른 곳으로 전송되지 않는다.
3. 서버는 받은 해답을 검증하고, 유효하다면 로그인 할 수 있다. Publick key는 해답이 valid한지를 확인할 수 있지만 해답 자체를 생성할 수는 없다. 즉 서버는 private key가 실제로 무엇인지 알 필요 없이 내가 맞는 private key를 가지고 있는지 알 수 있다는 뜻이다.

서버가 private key를 모르기 때문에 유출될 사용자 credential이 없다.

<img width="1547" alt="image" src="https://user-images.githubusercontent.com/41438361/173736556-6f0c3a73-9a69-4b41-ae7a-aae6ab4e56d9.png"><img width="1256" alt="image" src="https://user-images.githubusercontent.com/41438361/173736593-b4d77488-bc40-46cb-9dc6-7862aad36b6b.png">

Passkey는 두 기기 사이에서 안전하고 피싱이 일어나지 않게 사용될 수 있다.

* client : 내가 로그인하고자 하는 기기나 웹 브라우저
* authenticator : 내 passkey를 가진 기기

<img width="1424" alt="image" src="https://user-images.githubusercontent.com/41438361/173736721-1d0ddc4a-c2dc-4b54-8cf8-847371d2fb63.png">

1. Client가 QR 코드를 보여준다. QR 코드는 일회성 encryption key의 쌍을 encode하는 URL을 포함한다.
2. authenticator는 네트워크 relay 서버에 대한 routing 정보를 포함하는 Bluetooth advertisement를 생성한다.

이 두 과정을 통해 서버를 선택하고 routing 정보를 공유할 수 있다. 추가로 두 가지의 추가 기능을 제공한다.

* 서버가 볼 수 없는 out-of-band key agreement를 수행한다. 이를 통해 네트워크 상에서 동작하는 것들은 end-to-end encrypted되며 서버는 아무것도 알 수 없다.
* 두 기기가 physical proximity에 있는지를 검증할 수 있다. 이는 이메일로 전달됐거나 이상한 웹사이트가 보여주는 QR 코드를 동작할 수 없게 하는데, 원격 해커가 Bluetooth advertisement를 받을 수 없기 때문이다.

<img width="1536" alt="image" src="https://user-images.githubusercontent.com/41438361/173737495-c86b1e88-2986-464a-81ae-90a2e600d397.png">

위의 local exchange(위에서 언급한 1, 2단계)가 끝나고 key agreement가 수행되면 두 기기는 폰에 의해 선택된 relay 서버에 연결된다. 여기서부터 표준 FIDO CTAP 연산을 수행하는데, 이는 앞서서 키에 의해 암호화됐기 때문에 relay 서버는 무엇이 일어나는지 알 수 없다.

<img width="1393" alt="image" src="https://user-images.githubusercontent.com/41438361/173737537-64a2aec1-1dc0-4e43-9a0a-f72885593948.png">

웹사이트는 cross-device communication에 관여하지 않는다. Cross-device corss-platform 로그인은 passkey가 사용될 수 있는 곳 어디서든지 사용될 수 있는 시스템 기능이다.

# Multi-factor authentication

<img width="1651" alt="image" src="https://user-images.githubusercontent.com/41438361/173751870-c159ecfb-fa41-4a4d-a7ad-855122054d4e.png">

오늘날 인증은 여러 factor에 의해 이루어지고 이 factor들은 공격의 종류에 따라 안전할 수도 있고 그렇지 않을 수 있다. 

<img width="1725" alt="image" src="https://user-images.githubusercontent.com/41438361/173752287-5bbaee3c-e49f-470d-ba8f-4ba6709c06be.png">

* password : 머릿속에 있는 비밀번호는 거의 대부분의 요소에 취약하다.
* Password manager : 유일하고 high-entropy 문자열을 생성하는데는 좋지만 device theft에 의해 로컬 보안 문제가 생길 수 있고 피싱될 수 있다.
* SMS/time-based code : theft, 피싱 문제를 환경에 따라 해결할 수도 있고 그렇지 않을 수 있다.
* Passkey : 앱, 웹사이트 서버가 private 키를 가지고 있지 않기 때문에 노출될 수 없다.

*Next steps*

1. 서버에 WebAuthn을 적용한다.
2. passkey를 사용할 수 있게 앱, 웹사이트를 업데이트한다.
3. 사용자가 password를 사용하지 않게 한다.
