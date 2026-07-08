Apple Developer 계정 하나에서 인증서를 여러 개 만들 수 있음

# Apple Developer 계정

회사 / 개인이 가진 '회원권' 같은 것 

* Apple Developer Program
* Apple Developer Enterprise Program


# Certificate

계정 안에서 발급받는 '신분증'

**앱을 어떤 용도로 배포할 것인지에 따라 서로 다른 인증서를 발급받음**

```
Apple Developer 계정
│
├── Apple Development
├── Apple Distribution
├── Developer ID Application
├── Developer ID Installer
├── Mac Development
├── Mac App Distribution
└── ...
```

Enterprise 계정일 경우 Enterprise 용 인증서들도 추가됨

![[Pasted image 20260708155812.png]]

e.g.

|플랫폼|배포 방식|사용하는 인증서|
|-|-|-|
|iOS|App Store|Apple Distribution|
|iOS|사내 배포(Enterprise)|iOS Distribution (In-House)|
|macOS|Mac App Store|Mac App Distribution|
|macOS|App Store 밖 배포(인터넷, 사내 등)|Developer ID Application|

# 배포

## iOS 사내 배포

Enterprise Program 사용시

```
iOS 앱
    ↓
iOS Distribution (In-House) 인증서
    ↓
IPA 생성
    ↓
MDM 또는 사내 웹으로 배포
```

iOS 는 공증 (Notarization) 이라는 개념이 없음

## macOS 사내 배포

```
Mac 앱
    ↓
Developer ID Application
    ↓
Notarization
    ↓
DMG 또는 PKG
    ↓
사내 배포
```

## Ad Hoc vs In-House

* Ad Hoc : UDID 를 등록한 특정 기기에서만 설치할 수 있는 배포
* In-House : Enterprise Program 회원만 사용할 수 있는 내부 배포 방식
	* UDID 등록 필요 없음
	* 회사 직원 누구나 설치 가능
	* MDM, 사내 포탈 등을 통해 배포

In-House 는 Enterprise 계정에서만 활성화됨

|배포 방식|일반 Developer Program|Enterprise Program|
|-|-|-|
|Development|✅|✅|
|Ad Hoc|✅|✅|
|App Store|✅|❌|
|In-House|❌|✅|



# Provisioning Profile

이 앱을 어떤 인증서로 서명해서 어디까지 설치를 허용할 것인가

```
Provisioning Profile
├── App ID (Bundle Identifier)
├── 어떤 인증서로 서명할지
├── 어떤 기기에서 설치 가능한지(Development/AdHoc)
├── 어떤 Capability를 사용할 수 있는지
└── 어떤 배포 방식인지
```

![[Pasted image 20260708161601.png]]

## Bundle ID 가 다른 경우

e.g. 내 앱에서 configuration 별로 다른 bundle id 를 사용하는 경우

```
* Debug - com.company.myapp.dev
* Release - com.company.myapp
* QA - com.company.myapp.qa
```

각 Bundle ID 마다 App ID 가 다르기 때문에 Provisioning Profile 도 각각 만들어야 함

*같은 Bundle ID, 같은 Apple Distribution 인증서를 사용하는 경우에는 같은 Provisioning Profile 사용 가능*

## 배포 방식이 다른 경우

배포 방식이 결정됨 -> 그 배포방식에 맞는 인증서, 프로비저닝 프로파일을 사용하기 때문에

배포 방식이 다른 경우에도 다른 프로비저닝 프로파일을 사용하게 됨

e.g. 

* App Store 에 배포하고 싶음 (Apple 이 요구하는 것이 정해져 있음)

```
배포 방식
App Store
      │
      ├── Apple Distribution 인증서
      └── App Store Provisioning Profile
```

* Enterprise

```
배포 방식
Enterprise
      │
      ├── In-House Distribution 인증서
      └── In-House Provisioning Profile
```

즉

* Debug -> Development Profile
* QA -> In-House Profile
* Release -> App Store Profile