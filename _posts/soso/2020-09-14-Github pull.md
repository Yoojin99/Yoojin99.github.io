---
title:  "Github - pull, 원격 로컬 저장소 동기화"
excerpt: ""

categories:
  - soso
tags:
  - image upload
---

하나의 프로젝트로 여러 명이 협업할 때 보통 Github를 많이 사용한다. Turtoise svn, gitlab도 사용해봤지만 github가 가장 사용성이 좋은 것 같다.

프로젝트를 협업할 때 보통 원격에 있는 저장소(repository)를 fork 해서 복사본을 만든다. 그리고 작업을 할 때는 fork를 해서 복사되어 새로 생긴 그 저장소에서 branch를 여러개 만들어서 작업하는 방식을 취한다.

작업이 끝나고 나면, pull request를 하고, 원래 저장소에는 merge가 되었을 것이다. 그런데 이 상태에서 와 머지됐네 이제 마저 작업해야지 하면 안된다.

원격 저장소에는 내가 작업하는 동안 다른 사람들이 또 작업을 하고, commit을 하고, merge를 했을 것이다.

그러므로 원격 저장소(기본 프로젝트)에 A, B, C가 추가될 동안 나는 D만 추가해서 풀 리퀘를 했을 때 내 로컬 저장소에는 A, B, C라는 내용이 없는 것이다.

이때 git pull 명령어를 통해 원격 저장소와 내 저장소를 동기화 한다.

```
git pull upstream (기본 저장소의 branch 이름)
```

을 해주면 동기화가 된다.

1. 만약 github 웹 페이지에서 `This branch is X commits behind ~`라고 뜰 경우

그냥 위에 쓴 커맨드를 입력해주면 문제는 해결된다. 이후 다시 github 페이지에서 확인하면 `This branch is even ~`라고 뜬다.

2. 만약 github 웹 페이지에서 `This branch is X commits ahead, Y commits behind ~`라고 뜰 경우

```
git pull --reabse upstream (기본 저장소의 branch 이름)
```

을 해주면 된다. 만약 conflict가 났다 어쩌구 저쩌구 뜨면 그거는 직접 보면서 conflict 나는 부분을 해결하고, 다시 commit, push, pull 해야 할 것이다.
