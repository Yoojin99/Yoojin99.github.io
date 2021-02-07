---
title:  "Xcode EXUpdates/EXUpdatesAppController.h file not found error"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - app
  
tags:
  - app-ios
  - iOS
  - Xcode
last_modified_at: 2020-11-17
---

React Native project를 다운받고 Xcode로 iOS 폰과 연결시켜서 빌드하려니 문제가 생겼다.

`EXUpdates/EXUpdatesAppController.h file not found` 라는 에러가 뜨며 빌드가 되지 않았다.

1. 먼저 `package.json` 파일에 `react-native-unimodules`라는 모듈이 설치된 것이 반영되었는지 확인한다.
2. 또한 `package.json`에 `expo-updates`가 설치되었는지도 다시 확인한다. 아니라면 `npm install expo-updates --save` 커맨드를 통해 설치한다.
3. `Podfile`(ios 폴더 디렉토리 안에 있을 것이다) 파일에 `use_unimodules`가 있는지 확인한다.

만약 위에 것이 다 확인 되었으면 `cd ios` 커맨드로 `ios` 폴더로 이동하고 `pod install`을 해서 EX 모듈을 설치한다.

[출처](https://stackoverflow.com/questions/63847289/react-native-ios-build-fails-with-error-exupdates-exupdatesappcontroller-h-f)
