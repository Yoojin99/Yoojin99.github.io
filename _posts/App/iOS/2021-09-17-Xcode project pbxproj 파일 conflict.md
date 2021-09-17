---  
layout: post  
title: "[iOS] - Xcode project.pbxproj conflict와 이후 Xcode의 빌드 에러 해결 과정"  
subtitle: ""  
categories: app
tags: app-ios xcode pbxproj conflict 
comments: true  
header-img: 

---  
  
> `Xcode의 project.pbxproj 파일에서 일어난 conflict와 그 이후 빌드 과정에서 발생했던 에러를 해결하는 과정을 적어보았다`  

---

## `project.pbxproj` 파일 conflict 

여러 사람이 협업하는 과정에서 서로 다른 파일을 작업하고 생성하고 삭제하는 파일도 개발자마다 다르다. Xcode의 설정파일인 project.pbxproj파일에는 이런 파일의
변경사항, 그리고 구성에 대한 내용이 담기기 때문에 개발자들이 작업한 내용을 합칠 때 conflict가 발생할 확률이 높다. 이 `project.pbxproj` 파일은
실제 프로젝트의 설정을 담은 파일로, 프로젝트 내부에서 생성한 파일들을 파일 유형에 따라 reference를 저장하고 있다. 이 파일이 conflict가 일어날 경우
프로젝트 자체가 열리지 않기 때문에 에디터나 외부 툴(vscode)같은 걸로 conflict를 해결해야 한다.

이 부분은 어쩔 수 없다. 나는 vscode로 conflict가 나는 부분을 직접 확인해서 conflict가 나는 부분들을 해결했다. 대부분은 union(accept from both)
을 하면 되지만, 간혹 내 쪽의 수정 사항과 incoming 수정 사항이 겹치는 부분도 있으므로 주의해서 합쳐야 한다. 합치고 나서 빌드까지 잘 되면 좋겠지만
에러로 인해 빌드가 되지 않았다.

## Build input files cannot be found ~ (in target 어쩌구)

처음에 발생했던 에러는 `error: Build input files cannot be found: '/~' (in target '~' from project ~)` 였다. 이 에러의 경우
파일의 경로를 제대로 찾지 못한 경우에 발생하는 에러이다.

이 경우 Xcode에서 해당 파일의 이름이 붉은색으로 표시되어 있을 것이다. Xcode에서 붉게 표시된 파일은 파일을 인식하지 못하는 상태를 의미한다.

![image](https://user-images.githubusercontent.com/41438361/133752197-44ce2b32-a97e-462d-872b-54d0b7957f92.png)

이 파일을 선택하면 Xcode의 오른쪽에 아래와 같이 파일의 위치에 대한 정보가 나올 것이다. 나는 원래 지정되어 있는 위치에 해당 파일이 없어서 빨간색으로
표시한 파일 아이콘을 눌러 파일의 올바른 위치를 설정해주었다. 다른 글들을 보니 이 파일을 임시로 복사해서 다른 곳에다 옮겨놓고 원래 디렉토리에 있던 파일을 삭제해서
복사해둔 애들로 대체하라는데, 내가 했을 때 에러가 해결되지는 않았다. 

![image](https://user-images.githubusercontent.com/41438361/133752615-70bab9ef-a87e-458e-b706-cb8a9ba05ab7.png)

파일의 올바른 경로를 지정해주고 나면 원래 빨간색으로 표시되어 있던 파일이 다시 검은색으로 표시되며 Xcode에서 파일을 인식하고 있음을 나타낼 것이다. 이제
빌드를 하면 이 에러는 해결이 된다.

## Multiple commands produce

위에 뜨는 에러를 해결하고 나니 이번에는 `Multiple commands produce '~': 1) Target '어쩌구' ~` 에러가 떴다. 이 에러의 경우 
오류가 나는 파일이 중복으로 존재하거나 문제가 있음을 나타내는 에러다.

이 경우 위에 나온 target의 build phases 수정하면 된다.

![image](https://user-images.githubusercontent.com/41438361/133753682-dbdb21ca-7702-414c-9ad3-adc6eff63b4a.png)

해당 target의 build phases의 copy bundle resources에서 문제가 되는 파일을 삭제해주면 된다. 삭제하면 되는 파일은 `multiple commands produce` 뒤에
나오는 파일을 없애주면 된다.

여기까지 하고 다시 빌드를 했더니 에러는 더 이상 발생하지 않았다. 

* 참고
  * https://hcn1519.github.io/articles/2018-06/xcodeconfiguration
  * https://zetal.tistory.com/entry/Multiple-commands-produce-Error-%ED%95%B4%EA%B2%B0%EB%B2%95

