---
title:  "Github - `.gitignore` github 특정 file push 제외시키기"

categories:
  - soso
tags:
  - Github
---

Github에 파일들을 올릴 때 보안상의 문제로 주의해서 올려야 하는 파일, 혹은 아예 올리지 말아야 하는 파일들이 있다.

이런 파일들을 만약 그대로 github에 업로드하게 되면 크롤링을 통해 암호 키나 인증 파일같은 것이 그대로 유출되어 심각한 문제를 야기할 수 있다.

이때 github의 **secret** 기능을 사용하거나 **.gitignore** 를 사용하면 된다.

이 중에서 이 `.gitignore` 파일에 대해 적어보려고 한다.

`.gitignore` 파일을 생성하여 안에 제외하고 싶은 형식, 혹은 파일들을 적어두면 프로젝트를 push할 때 특정 파일들이 업로드 되는 것을 방지할 수 있다.

만드는 방법은 아주아주 간단하다.

1. 프로젝트 파일에 `.gitignore` 이라는 파일을 만든다. 이 파일은 항상 디렉토리의 최상단 위치에 있어야 한다.
2. 파일에 제외하고 싶은 파일 형식, 혹은 이름을 적는다.

예를 들어서 `.gitignore`파일에 `credentials.json`이라고 적으면, push할 때 `credentials.json` 파일은 업로드 되지 않는 것이다.
