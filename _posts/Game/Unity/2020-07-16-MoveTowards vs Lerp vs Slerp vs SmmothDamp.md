---
title:  "Unity - Vector3 MoveTowards vs Lerp vs Slerp vs SmoothDamp 차이"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - game
  
tags:
  - game-unity
  - 공부
  - C#
  - MoveTowards
  - Lerp
  - Slerp
  - SmoothDamp
  
last_modified_at: 
---

## Vector3.MoveToward

`public static Vector3 MoveTowards(Vector3 current, Vector3 target, float maxDistanceDelta);`

current point에서 target point로 직선으로 이동한다.리넡 값으로는 current에서 target point로 간 maxDistanceDelta 만큼 간 값을 
리턴한다. 

current에서 target까지의 거리가 maxDistanceDelta보다 작으면 target이 리턴된다. (movement will not overshoot the target)

maxDistanceDelta가 음수 값이면 target에서 멀리 떨어뜨려 놓겠다는 의미도 된다. 

maxDistanceDelta에 deltaTIme에 비례하는 값을 넣으면 시간에 따라 동일한 거리만큼 이동하므로 이동 속도는 일정하다.

가장 정직하게 일정한 거리만큼 이동할 수 있는 방법이다.

## Vector3.Lerp

`public static Vector3 Lerp(Vector3 a, Vector3 b, float t);`

두 벡터 a와 b를 t비율만큼 선형으로 interpolate 한다. 

t는 0~1 사이 값이며 a와 b 사이의 위치 비율이다. 만약 0~1을 넘으면 0~1 사이로 잘린다.

t==0이면 a를 return하고, t==1이면 b를 리턴한다. t==0.5이면 a와 b의 가운데가 리턴된다.

a에 현재 위치, b에 목표 위치, t를 deltaTime에 비례하는 값을 넣게 되면 현재 위치에서 목표지점까지 일정 비율만큼 이동하게 된다.

목표 위치까지의 거리는 이동한 만큼 줄어들게 되므로 결과적으로 이동 속도는 처음이 가장 빠르고 점점 느려지게 된다.

예를 들어 (0,0,0)에서 출발해서 목표지점 b를 (10,0,0), t=0.1로 잡으면 1초마다 lerp를 계산하여 처음에는 0->10의 10%인 1만큼 이동해서 다음 나의 위치가 (1,0,0)이 된다.

다음에 두번째는 1->10의 10%인 0.9만큼 이동하고, 점점 이렇게 이동하는 거리가 줄어들게 된다.

따라서 남은 거리의 동일한 비율만큼 이동하므로 처음 속도가 가장 빠르고 목표지점에 가까워질 수록 느려지는 것이다.

## Vector3.Slerp

`public static Vector3 Slerp(Vector3 a, Vector3 b, float t);`

두 점 사이를 구 형태로 interpolate한다. 

파라미터 t는 lerp와 동일하게 0~1사이의 값이며 a와 b 사이에 interpolate할 비율이다.

Lerp는 a, b를 한 공간의 점이라 보고 둘 사이를 직선으로 이어서 t 비율만큼 이동한 점을 찾는 것이라면,

Slerp는 두 벡터 a, b를 direction으로 보고 두 direction 사이의 중간 direction을 찾는 것이다.

![1](https://user-images.githubusercontent.com/41438361/87639210-dfef5700-c77f-11ea-897b-edf1227c0175.png)

위의 그림을 보면 이해가 쉽다.

빨간점이 a, 초록점이 b라 하면 파란점을 Lerp(a,b,0.5)이고 하얀점은 Slerp(a,b,0.5)이다.

Slerp(현재 위치, 목표 위치, deltaTime * 비율)로 사용하면 기본적으로 lerp랑 비슷하지만 3D인 경우 직선이 아닌 구형으로 이동한다.

따라서 직성 방향으로의 이동속도는 Lerp에 삼각함수 곡선이 곱해진 형태로 Lerp보다 초반 후반이 느리고 중간이 더 빠른 형태가 된다.

![2](https://user-images.githubusercontent.com/41438361/87640052-1a0d2880-c781-11ea-9f48-70cd1f9358b7.png)

## Vector3.SmoothDamp

`public static Vector3 SmoothDamp(Vector3 current, Vector3 target, ref Vector3 currentVelocity, float smoothTime, float maxSpeed = Mathf.Infinity, float deltaTime = Time.deltaTime);`

벡터를 목표 지점까지 시간의 흐름에 비례에 점차적으로 부드럽게 이동시킨다.

카메라를 부드럽게 따라가게 하는데 가장 많이 사용한다.

현재 속도를 ref로 넘긴다.

## 비교

![다운로드](https://user-images.githubusercontent.com/41438361/87640577-df57c000-c781-11ea-9420-b89070cb453f.gif)




