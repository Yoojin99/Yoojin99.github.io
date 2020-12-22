---
title:  "Build.gradle 에서 수정해야 했던 소소한 에러들"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - Android
  
tags:
  - build.gradle
  - sdk
  - packagingOptions
  - exclude
  - multiDexEnabled
  
last_modified_at: 2020-12-22
---

## minSdkVersion

google api를 사용하기 위해 라이브러리를 dependency에 추가하고 함수를 사용했는데, sdk가 최소 26버전에서 작동하는 게 보장되어 있어야 한다는 메세지가 떴다. 이 경우 
build.gradle의 `minSdkVersion`을 26으로 설정했더니 해결되었다.

## multiDexEnabled

간혹 코딩하다 multidex error가 뜨면서 run이 되지 않을 경우가 있을 것이다. 이 경우 build.gradle 파일의 config 안에 `multiDexEnabled` 를 true로 설정해주면 된다. 

나는 [이 페이지](https://namget.tistory.com/entry/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-%EB%A9%80%ED%8B%B0%EB%8D%B1%EC%8A%A4multidex-%EC%97%90%EB%9F%AC-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0)를 참고했다.

## packagingOptions

Run을 시키는 데 ~~ 파일이 os에서 중복되었다는 에러 메세지와 함께 run이 되지 않는 경우가 있었다. 이 경우 아래와 같이 이 에러메세지에 뜬 파일들을 exclude 해주니 해결되었다.

```
packagingOptions {
        exclude 'META-INF/INDEX.LIST'
        exclude 'META-INF/DEPENDENCIES'
}
```

