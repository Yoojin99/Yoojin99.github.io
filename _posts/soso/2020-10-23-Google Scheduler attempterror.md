---
title:  "Google Scheduler "
excerpt: ""

categories:
  - soso
tags:
  - Google schduler
  - Google function
  - gcf
  - google cloud function
last_modified_at: 
---

GCF(goolge cloud function)에 함수를 하나 만들어놓고 google cloud scheduler를 이용해서 함수를 매일 같은 시간에 잘 돌리고 있었는데, 가끔씩 에러가 나서 멈추는 경우가 발생했다.

Google cloud scheduler에서 로그 보기를 눌러 로그를 확인하니 아래와 같이 "AttemptFinished"라는 문구를 확인할 수 있었다.

![image](https://user-images.githubusercontent.com/41438361/96954568-9bb5e480-152e-11eb-8d2d-9e99e1dd6906.png)

구글링을 열심히 해보니 [이 링크](https://stackoverflow.com/questions/59024925/gcp-cloud-scheduler-throws-error-for-a-http-targettype)를 확인할 수 있었는데, gcf를 돌리는 시간, 데드라인이 
180초로 정해져 있나보다. 즉 180초 안에 함수가 다 실행이 되지 않으면 에러가 나게 된다는 것이다. 골치아픈거는 이렇게 google cloud scheduler에서 한 번 에러가 나면 내가 고칠때까지
다시 돌리지 않는다는 것이다.

하여간 이 에러를 고치려면 gcloud console을 이용하면 된다.

Google cloud platform 웹 페이지에서 오른쪽 상단의 cloud shell icon을 눌러서 터미널을 활성화 시켜주자.

`gcloud scheduler job list` 를 치면 google cloud scheduler로 설정한 함수의 목록(이름)을 볼 수 있고,

`gcloud scheduler jobs update http my-super-job --attempt-deadline 540s` 와 같이 쳐서 해당 함수의 attempt deadline을 조정할 수 있다.
