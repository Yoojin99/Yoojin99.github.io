---
title:  "Unity - Instantiate()와 Destroy()"

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - Unity
  
tags:
  - C#
  - 공부
  - Unity
  - Instantiate
  - Destroy
  
last_modified_at: 
---

```C#
GameObject Object.Instantiate<GameObject>(GameObject original, Vector3 position, Quaternion rotation)
```

Instantiate() 함수는 게임 중에 게임 오브젝트를 새로 생성할 수 있다.

게임 실행 전에 미리 다 만들어놓는 방법도 있지만, 그렇게 불가능한 경우도 있다. RPG 게임에서 수많은 아이템, 캐릭터,
혹은 발사되는 총알까지 미리 다 만들어두는 것은 불가능하다. 이럴 때는 게임 중에 게임오브젝트를 새로 생성해야 한다.
생성이라고 하긴 했지만 생성보다는 복제본을 생성한다는 것이 정확하다.

이럴때 Instantiate() 함루를 사용하는데, 매개변수를 보도록 하자.

1. GameObject Original : 생성하고자 하는 게임 오브젝트명. 현재 씬에 있는 게임오브젝트나 Prefab으로 선언된 객체를 의히맣낟.
2. Vector3 position : Vectro3으로 생성될 위치를 결정한다.
3. Quaternion rotation : 생성될 게임 오브젝트의 회전값을 지정한다. 회전을 굳이 줘야할 상황이 아니라면 그냥 기본은 
`Quaternion.identity`로 설정하거나 게임 오브젝트에서 설정된 회전값, `original.transform.rotation`으로 작성해도 된다.

```C#
Instantiate(obj, new Vector3(x,y,z), Quaternion.identity);      // 그냥 회전없음.

Instantiate(obj, new Vector3(x,y,z), obj.transform.rotation);   // obj의 회전값.
```

```C#
GameObject UnityEngine.Object.Instantiate<GameObject>(GameObject original, Transform parent)
```
로 설정해주면 position과 rotation을 설정하지 않고, 특정 하이어라키 위치에서 생성할 수 있다.

넣고 싶은 오브젝트를 두 번째 파라미터인 parent 에 적어주면 복제 생성시 하위 자식으로 생성된다.

반대로 Instantiate로 생성한 오브젝트를 제거, 삭제하려면 Destroy() 함수를 사용해야 한다.

```C#
Destroy(GameObject obj);
```

로 사용하면 된다.

만약 특정 시간을 지연시킨 후 오브젝트를 파괴하고 싶으면

```C#
Destroy(GameObject obj, float time);
```

처럼 넣어주면 된다.

