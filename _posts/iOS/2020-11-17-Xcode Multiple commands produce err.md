---
title:  "Xcode 빌드시 'Multiple commands produce' error"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - iOS
  
tags:
  - 공부
  - iOS
  - Xcode
last_modified_at: 2020-11-17
---

React Native 프로젝트를 Xcode로 빌드하려니 'Multiple commands produce' 에러가 떴다.

참고로 내 Xcode 버전은 12.2 이상이었던 것 같다.

에러를 간단하게 설명하자면, 한 파일이 중복되어서 프로젝트에 존재하는 현상이다. 

Project>Targets 에 있는 target을 클릭해주고 Build Phases에서 `Copy Bundle Resources`에서 중복된 파일을 삭제해준다. 이 중복이라는 것은
단순히 이 copy bundle resources에서만 중복되는 것이 아니라 프로젝트 전체에 중복된 파일이 있는지 여부를 말하는 것이다.

나같은 경우는 `Googleplay-info.~~` 파일이 로컬 파일 디렉토리에도 존재하고 저 copy bundle resources에서도 존재해서 copy bundle resources에 있는 것을 
삭제해줬더니 에러가 없어졌다.

참고한 링크 주소이다. https://github.com/oblador/react-native-vector-icons/issues/1074
