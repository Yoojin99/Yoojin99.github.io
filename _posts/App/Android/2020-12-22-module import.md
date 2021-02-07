---
title:  "Module import, dependency 문제 해결"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
categories:
  - app
  
tags:
  - app-and
  - module
  - dependency
  
last_modified_at: 2020-12-22
---

원래 python, c#, c++ 만 했었는데 어쩌다 안드로이드 앱을 개발하게 돼서 java를 다루게 되었다. 내가 만들고자 했던 앱은 google stt, tts api, google dialogflow cx를 이용해서 앱과 음성으로 대화할 수 있는
봇을 만드는 것이었다.

그러기 위해서는 google api를 사용하기 위해 필요한 관련 모듈을 설치해야 했는데, 공식 홈페이지에는 다음과 같이 나와있어서 android 와 java에 익숙하지 않아서 어떻게 이 dependency들을 설치해야 하는지 한참을 헤맸다.

![image](https://user-images.githubusercontent.com/41438361/102839732-841ead80-4444-11eb-9e40-1cf23b41f0cd.png)

[출처](https://cloud.google.com/speech-to-text/docs/quickstart-client-libraries)

대체 이 dependenies들은 뭐고 이 코드는 어디에 넣는 것이며 아는 게 하나도 없었다. 우선 위와 같이 `<>`과 같은 tag안에 있는 애들은 maven을 사용할 경우 pom.xml에 넣으라는데 pom.xml을 찾으려니 보이지 않았다.
알고보니 maven을 사용하는 것과 gradle을 사용하는 것과는 차이가 있다고 한다. 그래서 나는 gradle 방식을 사용하고 있었기 때문에 `build.gradle`에 위의 dependencies 들을 문법에 맞춰 넣어줘야 했다.

가장 먼저 프로젝트 파일을 열어보도록 하자. 프로젝트를 열 때는 아래와 같이 `app`으로 열어주는 것이 가장 편하다. project로 열 때나 app으로 열 때와 보여지는 구성이 조금 달라지는데, app일 때 app/gradle 잘 
나눠서 보기 좋다.

![image](https://user-images.githubusercontent.com/41438361/102840104-600f9c00-4445-11eb-8eb8-71202848cf25.png)

build.gradle 파일은 저 디렉토리 목록의 거북이 아이콘이 있고 `Gradle Scripts` 라고 되어 있는 곳에 있다. build.gradle 파일이 두 개 이상이 나올 수 있는데, build.gradle 뒤에 나와있는 괄호 안에
`Module: ~~.app`이라고 되어 있는 build.gradle 파일을 열어주면 된다.

파일안을 보면 아래와 같이 dependencies ~ 라고 나와 있는 부분이 있다. 바로 여기가 import 할 모듈들이나 라이브러리들을 입력하는 곳이다. python과 비교했을 때 import ~ 하는 부분과 비슷하다고 생각하면 될 것 같다.

`implementation ~`라고 적혀있는데, 룰이 있다. `implementation 그룹이름:이름:버전`

예를 들어 `com.google.cloud:google-cloud-texttospeech:1.2.4`를 보면 com.google.cloud 그룹의 google-cloud-texttospeech의 1.2.4 버전을 사용하겠다는 의미가 된다.

이런식으로 직접 build.gradle에 직접 dependency들을 적어줘도 되지만, 버전이 뭔지 불분명할 때도 있고, 검색을 통해서 찾는 것이 훨씬 편하기 때문에 나는 검색을 통해 라이브러리를 추가하는 것을 추천한다.

검색을 통해 dependency를 추가하는 방법은 아래와 같다.

![image](https://user-images.githubusercontent.com/41438361/102840821-05773f80-4447-11eb-9e20-551ab131c9da.png)

우선 Project Sturcture 메뉴를 열어준다. 

![image](https://user-images.githubusercontent.com/41438361/102840932-47a08100-4447-11eb-839f-47767b07e299.png)

그리고 저 + 버튼을 클릭해서 Library dependency 를 눌러준다. 그러면 아래와 같은 검색창이 나오는데 여기서 원하는 라이브러리를 찾아 추가할 수 있다. 좀 짜증나는 점은 이름을 정확하게 쳐야 검색이 된다.
예를 들어 com.google.cloud를 찾고 싶은데 com.google 만 검색하면 com.google.cloud를 찾을 수 없다. 그래서 정확하게 풀네임을 검색하는 것이 좋다.

하여간 원하는 버전까지 선택해주고 ok를 누르면 자동으로 build.gradle 파일에 dependency가 추가된다. 
