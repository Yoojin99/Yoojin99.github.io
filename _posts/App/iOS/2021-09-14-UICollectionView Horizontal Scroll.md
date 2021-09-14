---  
layout: post  
title: "[iOS] - UICollectionView에서 horizontal scroll 할 때 주의할 점"  
subtitle: ""  
categories: app
tags: app-ios uicollectionview
comments: true  
header-img: 

---  
  
> `UICollectionView를 횡 스크롤할 때(horizontal scroll) 헷갈리는 점들과 주의할 점들에 대해 짚고 넘어가보자.`  

---

UICollectionView에는 item들을 cell 형태로 추가할 수 있다. 기본적으로 UICollectionView는 세로로 스크롤하는 형태(Vertically)이기 때문에 UICollectionView에
아이템을 여러 개 추가하면 아래와 같이 컬렉션 뷰가 잡히며 스크롤링 되는 것을 알 수 있다. 컬렉션 뷰를 스크롤하는 방향과 같은 레이아웃은 `UICollectionViewFlowLayout`를 통해 설정할 수 있다.

```swift
let collectionViewLayout: UICollectionViewFlowLayout = UICollectionViewFlowLayout()
collectionViewLayout.scrollDirection = .horizontal // 스크롤 할 방향 선택. 

let collectionView: UICollectionView = UICollectionView(frame: .zero, collectionViewLayout: collectionViewLayout)
```

코드에서 위와 같이 작성하면 UICollectionView을 스크롤할 때 가로 방향으로 스크롤 한다는 의미가 된다.

![image](https://user-images.githubusercontent.com/41438361/133198795-45060a56-03ec-43c7-a84d-e1adad837275.png)

또 이렇게 스크롤 할 방향 뿐만 아니라 레이아웃을 통해서 굉장히 많은 것들을 설정할 수 있다. 어떤 값들을 설정할 수 있는지는 애플 [공식문서](https://developer.apple.com/documentation/uikit/uicollectionviewflowlayout?language=objc)에서도 자세한 내용을 확인할 수 있다.

그 중에서도 헷갈릴 수 있는 점이 `minimumLineSpacing`이라고 생각한다. 스크롤을 가로 방향이나 세로 방향으로 설정할 때 라인의 기준이 달라진다. 이게 무슨 말이냐면
스크롤 방향이 세로 방향이면 라인의 기준은 행이 되는데, 스크롤 방향이 가로면 라인의 기준이 열이 되는 것이다.

그래서 처음에 컬렉션 뷰의 스크롤 방향을 가로 방향으로 설정했을 때, 아래 그림에서 아이템 사이의 간격을 넓히기 위해 아래와 같이 설정했었다. 내 예상은 빨간색으로 표시한 부분의 간격이 더 커지는 것을 기대했다.

![image](https://user-images.githubusercontent.com/41438361/133199183-3e57f26f-d259-4fe1-8fe5-1195a254eed3.png)

```swift
collectionViewLayout.minimumInteritemSpacing = 20 // 아이템 사이의 간격을 늘리면 빨간색 부분의 간격도 넓어지지 않을까?
```

하지만 변하는 것은 없었다. 그래서 아래와 같이 코드를 수정하니 빨간색 부분의 간격이 넓어지게 되었다.

```swift
collectionViewLayout.minimumLineSpacing = 20 // minimumInteritemSpacing을 minimumLineSpacing으로 바꿔줬다.
```

문제는 내가 스크롤 방향이 바뀔 때 라인의 기준도 달라진다는 점을 미처 생각하지 못했던 것이다. 흔히 생각하는 line은 행, 가로줄 방향인데, 스크롤 방향이 가로가 되면 라인은 자연스럽게 열 방향이 기준이 된다. 

![image](https://user-images.githubusercontent.com/41438361/133199754-40e6e0c4-377b-49eb-9ddc-8c02eaa1262d.png)

그리고 덧붙이자면 나는 가로 스크롤이 되는 한 줄 컬렉션 뷰를 만드는 상황이었는데, 이 때 웬만해서는 `minimumInteritemSpacing`을 0으로 설정하는 편이 좋다. 이 `minimumInteritemSpacing`의 값을 아예 설정하지 않거나 설정하면 레이아웃이 예상하지 못했던 방향으로 잡히는 현상이 생겼다. 

![Uploading image.png…]()

위와 같이 아이템들이 다 배치되고 나서 오른쪽에 스크롤이 가능한 여백이 생겼는데, 이 현상의 원인을 알아내는대로 이어서 적어보도록 하겠다.
