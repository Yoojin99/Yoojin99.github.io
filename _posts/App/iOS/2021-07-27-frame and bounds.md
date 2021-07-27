---  
layout: post  
title: "[iOS] - view의 frame, bounds"  
subtitle: ""  
categories: app
tags: app-ios
comments: true  
header-img: 

---  
  
> `frame 과 bounds가 무엇인지, 둘의 차이가 무엇인지 알아보자.`  

---


## Frame

[공식문서](https://developer.apple.com/documentation/uikit/uiview/1622621-frame)에는 아래와 같이 나와 있다.

![image](https://user-images.githubusercontent.com/41438361/127115276-c9cb9d41-bfdd-4e5c-a3e3-f8d4ce525878.png)

즉 **부모 뷰의 좌표계에서**의 자식 뷰의 위치와 크기를 나타내는 프레임 사각형이다. 여기서 중요한 것은 부모 뷰의 좌표계에서의 자식 뷰의 위치와 크기라는 것이다.

이 프레임 값을 바꾸면 `center` 프로퍼티에 있는 좌표와 `bounds`라는 사가형에 있는 사이즈도 바뀌게 된다. 프레임 사각형을 바꾸면 `draw(_:)` 메서드를 호출하지 않고
자동으로 뷰를 다시 출력한다.

## Bounds

![image](https://user-images.githubusercontent.com/41438361/127115714-567cfb16-0e4b-44eb-b16c-7ed13ab00a39.png)

스스로의 좌표계에서의 뷰의 위치와 사이즈를 나타내는 bounds 사각형이다. 위에서 frame은 부모 뷰를 기준으로 위치와 크기를 설정했는데, bounds는
자신의 좌표계에서 위치와 크기를 나타낸다. 

그림으로 나타내면 아래와 같다.

![image](https://user-images.githubusercontent.com/41438361/127116996-abb8d544-040a-4cce-9c27-13f3883915b2.png)

자신의 좌표계에서 위치와 크기를 나타낸다. 파란색 부모뷰와, 부모뷰의 (0,0), 즉 왼쪽 상단 꼭짓점을 기준으로 x축으로 30만큼, y출으로는 50점
떨어진 지점에 한 변의 길이가 60인 정사각형인 자식뷰가 있다. 
 
이 자식뷰의 frame은 부모 뷰를 기준으로 하므로 (30, 50, 60, 60)이 될 것이고, bounds는 자식 뷰 자신의 좌표계를 기준으로 하므로 (0, 0, 60, 60)이 되는 것이다.

frame과 bounds를 각각 바꾸면 어떻게 될까?

먼저 위 그림의 상태에서 자식 뷰의 frame을 (0, 0, 60, 60)으로 바꾸면 아래와 같이 될 것이다. 이는 부모 뷰의 (0,0)에서 시작한다는 말이므로 직관적으로 이해할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/127117554-fdfb7e79-1e24-47ac-959c-cd41dc0b65fa.png)

그렇다면 이번에는 부모 뷰의 origin을 (30, 30)으로 바꿔보자. 그러면 아래와 같은 일이 일어난다.

![image](https://user-images.githubusercontent.com/41438361/127118783-a507a069-dd36-45fa-abb1-1dde38961f8a.png)

부모 뷰는 자신 만의 좌표계를 가지는데, (30, 30)으로 이동했으니 아래 그림처럼 자신의 좌표계의 (30,30)으로 이동하게 된다. 이는 이 위치에서 해당 뷰를 다시 
그리라는 의미가 된다. 그래서 실제로는 부모 뷰가 아래로 내려간 것인데 화면 상에서는 자식 뷰가 올라간 것처럼 보이게 되는 것이다.
