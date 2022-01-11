---  
layout: post  
title: "[iOS] - UICollectionView visibleCells"  
subtitle: ""  
categories: app
tags: app-ios UICollectionView visibleCells
comments: true  
header-img: 

---  
  
> `UICollectionView에서 스크린에 보여지는 cell들만 확인하는 방법이 있을까?`  

---

## visibleCells

UICollectionView를 사용하다 보면 UICollectionView가 출력하고 있는 cell들 중에서 화면에 띄워져 있는 cell들만 알고 싶은 경우도 생길 것이다.

UICollectionView의 property를 찾아보던 중 `visibleCells`를 찾을 수 있었다.

![image](https://user-images.githubusercontent.com/41438361/148900191-d11512e0-bd12-4b35-9b47-aa114ac3395d.png)

UICollectionView의 인스턴스 프로퍼티인 `visibleCells`는 현재 collectionView에 의해 출력된 cell들 중 보여지는 cell들의 배열이다. 만약 보여지는 cell이 없다면 빈 배열이 return 된다고 한다. 

### 예시




dfsdfsd
