---  
layout: post  
title: "[iOS] - iOS Memory Deep Dive"  
subtitle: ""  
categories: app
tags: app-ios memory
comments: true  
header-img: 

---  
  
> `iOS 메모리에 대해 깊이 알아보자.`  

---

WWDC 2018에 발표되었던 "iOS Memory Deep Dive"을 보고 정리했다.

이 영상에는 아래의 내용이 포함되어 있다.

1. Memroy Footprint가 무엇인지? (Page Size, Dirty Memory, Swap Memory)
2. Memory Footprint를 profiling하는 도구
3. 메모리 그래프를 분석하기 위해 vmmap과 heap command-line 도구
4. 이미지 렌더링 형식
5. UIGraphicsBeginImageContextWithOptions 대신 사용할 수 있는 UIGraphicsImageRenderer
6. 앱이 백그라운드에 있을 때 최적화하기

우리가 메모리를 줄이는 이유는 무엇인가? 이에 대한 대답은 사용자에게 더 나은 앱 사용 경험을 제공할 수 있기 때문이다. 앱이 빠르게 동작할 수 있고, 시스템이
더 잘 동작한다. 또한 앱이 메모리에 오래 머물러 있을 수 있다. 내 앱 뿐만 아니라 다른 앱도 메모리에 더 오래 있을 수 있다. 여기까지만 봐도 많은 점들이
향상된다. 

# Memory FootPrint

메모리를 감소하는 것에 대해 말하고 있지만, 더 정확하게는 memory footprint을 줄이는 것이다. 모든 메모리는 동일하게 생성되는 것이 아니다. 이게 무슨 말인지 더 보려면 page에 대해 얘기해야 한다.

## Pages

메모리 페이지는 시스템에 의해 주어진 것이고, heap에 여러 개의 객체를 담을 수 있다. 그리고 몇 객체는 실제로 여러 개의 page에 걸칠 수 있다. 

![image](https://user-images.githubusercontent.com/41438361/134643464-faf94dcc-1a44-4d14-85c5-fdc6e45db449.png)

* 보통 16KB
* 페이지 타입: Clean / Dirty
* 앱이 사용한 메모리: Page의 수 X Page size

![image](https://user-images.githubusercontent.com/41438361/134643709-66244841-57a3-453f-adc2-a6ee54f6fe3c.png)

페이지 타입으로 Clean / Dirty가 있는데 옐ㄹ 들어 20,000개의 정수의 배열을 할당하고 싶다고 해보자. 그러면 시스템은 6개의 페이지를 줄 것이다. 이때, 
페이지는 내가 할당할 때는 clean하다. 

![image](https://user-images.githubusercontent.com/41438361/134644028-df73a535-519c-49fe-b45b-a38abf9133aa.png)

하지만 내가 만약 이 배열의 첫 번째 요소에 값을 쓴다면, 이 page는 dirty해진다. 첫 번째 요소 뿐만 아니라 마지막 페이지에 값을 써도 마찬가지로 dirty해진다.

![image](https://user-images.githubusercontent.com/41438361/134644122-9e5c8346-2965-4d78-b56f-20d610f23338.png)

이때 값을 쓴 첫 번째와 마지막을 제외하고 가운데의 4개의 페이지는 앱이 여기에 값을 쓰지 않았기 때문에 아직 clean하다. 

## Memory mapped files

이 파일들은 디스크에 있지만 메모리에 로드된 애들이다. 

예를 들어 내가 읽기 전용 파일을 사용한다면, 얘네들은 항상 clean page가 될 것이다. 커널도 디스크에서 RAM으로 오고 나갈때 실제로 이들을 관리한다. 

무슨 소리인지 모르겠어서 예시를 들고 왔다. JPEG가 있다고 해보고, 이게 memory mapped in일 때 50KB이고, 메모리에 4 페이지를 차지한다고 해보자.
이때, 4번째 페이지는 완전히 차 있지 않은 상태이기 때문에 다른 것들을 추가로 저장할 수 있다. 하지만 이전의 세 페이지는 항상 시스템에 의해 제거될 수 있는 상태여야 한다. 
그리고 우리는 보통 앱의 footprint와 profile이 메모리의 dirty, compressed, clean한 segment를 가지고 있다고 말한다.

![image](https://user-images.githubusercontent.com/41438361/134645518-fa930dd6-3df5-4d9e-86bb-32ed4de33de3.png)

### Clean memory

클린 메모리는 페이지 아웃 될 수 있는 데이터다. 얘네들은 바로 위에서 언급한 memory-mapped 파일이다. 이미지, data Blob, training model, framework가 될 수 있다.

![image](https://user-images.githubusercontent.com/41438361/134645783-1d465e81-d0c2-4094-a757-2d48cbf73cab.png)

위는 클린 메모리인 프레임워크를 보여준 것인데, 모든 프레임워크는 DATA CONST 섹션을 가지고 있다.

얘네들은 보통 clean하지만, 런타임에 메서드 뒤섞기와 같은 이상한 것들을 하면 dirty해질 수 있다. 


### Dirty memory

Dirty memory는 앱에 의해 쓰여진 메모리이다. 얘네들은 할당된 그 어떤 것이라도 될 수 있다. 객체, 문자열, 배열 등등이 여기 포함된다. 

![image](https://user-images.githubusercontent.com/41438361/134646427-41ba4d06-f5e0-406d-8f25-1391a371bc73.png)

얘네들은 디코딩된 이미지 버퍼, 프레임워크가 될 수 있다. 프레임워크는 data 섹션과 data dirty 섹션을 가지고 있다.

여기서, 클린 메모리와 더러운 메모리에 프레임워크가 두 번 언급된 것을 확인할 수 있다.
내가 link한 프레임워크는 memory와 dirty memory를 사용할 수 있다. 












