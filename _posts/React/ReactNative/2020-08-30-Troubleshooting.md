---
title:  "React Native - Troubleshooting, 문제해결"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - react
  
tags:
  - react-rn
  - React Native
  
last_modified_at: 
---

![image](https://user-images.githubusercontent.com/41438361/90485901-2404bb80-e173-11ea-8aaf-141b0555134b.png)

아래에 나올 것들은 RN을 세팅할 때 겪을 수 있는 흔한 이슈들이다. 만약 여기에 없는 문제를 겪는다면 [Github issue](https://github.com/facebook/react-native/issues/)에서 찾길 바란다.

## 이미 사용중인 Port

Metro bundler는 포트 8081 번에서 작동한다. 만약 다른 프로세스가 이미 그 포트를 사용중이라면 그 프로세스를 중지시키거나 bundler가 사용하는 포트를 바꾸면 된다.

### 포트 8081에서 실행중인 프로세스 중간시키기

```
sudo lsof -i :8081
kill -9 <PID>
```

윈도우에서는 리소스 모니터를 이용해서 찾고 Task Manager를 이용해서 중단시킬 수 있다.

### 8081 말고 다른 포트 사용하기

```
npx react-native start --port=8088
```

## NPM locking error

만약 RN CLI를 이용도중 `npm WARN locking Error: EACCES`와 같은 에러를 겪게되면 아래 커맨드를 쳐보라.

```
sudo chown -R $USER ~/.npm
sudo chown -R $USER /usr/local/lib/node_modules
```

## React에 없는 라이브러리

만약 RN을 프로젝트에 수동으로 추가했다면, `RCTText.xcodeproj`나 `RCTImage.xcodeproj`같은 관련있는 것들을 다 포함했는지 확인해라.
다음, 이런 요소들에 의해 빌드된 바이너리들은 앱 바이너리에 링크되어야 한다. `Linked Frameworks and Binaries`를 이용해라.

(추가 예정)


