---
title:  "Unity - SerializeField, Serializable "

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
  - Unity
  - SerializeField
  - java
  
last_modified_at: 
---

직렬화는 무엇일까?

## 직렬화(serialization)

1. 객체를 '연속적인 데이터'로 변환하는 것. 반대과정은 '역직렬화'라고 한다.
2. 객체의 인스턴스변수들의 값을 일렬로 나열하는 것

![1](https://user-images.githubusercontent.com/41438361/89632762-b4264380-d8dd-11ea-8a42-84390196ad55.png)

1. 객체를 저장하기 위해서는 객체를 직렬화해야 한다.
2. 객체를 저장한다는 것은 객체의 모든 인스턴스 변수의 값을 저장하는 것이다.

![2](https://user-images.githubusercontent.com/41438361/89632831-d61fc600-d8dd-11ea-9057-190e78f240f8.png)

그림에서도 볼 수 있듯이 변수들에 있는 값만 따로 저장하고 있다.

여기 아래는 java 코드이다. [출처](https://shin6666.tistory.com/entry/%EC%A7%81%EB%A0%AC%ED%99%94serialization)

**ObjectInputStream, ObjectOutputStream**

객체를 직렬화하여 입출력할 수 있게 해주는 보조스트림이다.

```java
ObjectInputStream(InputStream in)
ObjectOutputStream(OutputStream out)
```

- 객체를 파일에 저장하는 방법

```java
FileOutputStream fos = new FileOutputStream("objectfile.ser");
ObjectOutputStream out = new ObjectOutputStream(fos);

out.writeObject(new UserInfo());
```

- 파일에 저장된 객체를 다시 읽어오는 방법

```java
FileInputStream fls = new FileInputStream("objectfile.ser");
ObjectInputStream in = new ObjectInputStream(fls);

UserInfo info = (UserInfo)in.readObejct();
```

코드를 보다보니 클래스 선언부 초입에 `[SerializeField] ~~`라고 되어있는 부분이 많았다.

간단히 얘기하면, 원래 unity에서는 private 변수로 설정하면 해당 변수들이 inspector창에서는 나타나지 않는 것으로 알고 있다.

```
public int Speed1 = 10; //Insepctor 창에서 보여진다.
private int Speed2 = 10; //Insepctor 창에서 보여지지 않는다.
```

그런데, 만약

```
[SerializeField]
private int Speed = 10; //Inspector 창에서 보여진다.
```

위에처럼 SerializeField를 설정하고 private 변수를 설정하면 Inspector 창에서 보여줄 수 있는 것이다.

[유니티 공식 문서](https://docs.unity3d.com/kr/530/ScriptReference/SerializeField.html)에서는 다음과 같이 설명한다.

유니티가 private 필드를 직렬화하도록(Serialization)하도록 설정한다.

특별한 경우가 아니면 사용하지 않는다. 직렬화를 하게 되면 Unity의 Inspector창에서 필드들이 노출이 된다.

사용자는 모든 스크립트 컴포넌트를 직렬화 하고, 스크립트 컴포넌트들을 직렬화된 버전으로 다시 로드하고 재생성한다.

직렬화될 수 있는 타입은 다음과 같다.

- All classes inheriting from UnityEngine.Object, for example GameObject, Component, MonoBehaviour, Texture2D, AnimationClip.
- All basic data types like int, string, float, bool.
- Some built-in types like Vector2, Vector3, Vector4, Quaternion, Matrix4x4, Color, Rect, LayerMask.
- Arrays of a serializable type
- List of a serializable type)
- Enums
- Structs


## Serializable

이 Unity에서 제공하는 Serialization API를 이용하면 바이너리 형태의 스트림으로 데이터를 변환하여 파일 등으로 저장할 수 있다. [참고](https://m.blog.naver.com/PostView.nhn?blogId=yoohee2018&logNo=220724696138&proxyReferer=https:%2F%2Fwww.google.com%2F)

코딩 순서는 밑과 같다.

* Save
  1. 게임 데이터 일반 변수 선언 및 값 할당 (A, 일반적으로 static)
  2. Serializable 클래스 선언(B)
  3. FileStream 클래스 이용하여 파일 생성(F)
  4. Serializable 클래스에 게임 데이터 담기(A->B)
  5. BinaryFormatter 클래스 이용하여 B를 Serialize 하여 F에 쓰기 (B->F)
  6. File CLose
  
* Load
  1. FileStream 클래스 이용하여 파일 오픈(F)
  2. BinaryFormatter 클래스 이용하여 Deserialize하여 Serializable 클래스에 담기(F->B)
  3. Serializable 클래스에 담긴 데이터를 실제 게임에서 쓰이는 일반 변수에 재할당(B->A)
  4. 일반 변수 사용 (화면에 render 등)(A 사용)
  
게임에 쓰이는 일반 변수를 A라 하면, 직렬화 가능한 클래스인 B를 거쳐 파일 F에 저장하게 된다.

저장할 때는 A->B->F, 로딩할 때는 F->B->A 가 되는 것이다.

직렬화 가능한 클래스/변수는 **스크립트에서 직접 게임에 이용되는 것이 아니라, Binary 형태로 직렬화 하기 위해 잠깐 거치는 temp table 같은 것이다.**

직렬화 가능한 클래스라는 것은 따로 특정 타입이 정해져있는 것이 아니라, 클래스 선언부에 `[Serializable]` 키워드를 붙이면 된다.

Public String, int, Float, bool 등과 같은 기본 데이터 타입은 별도의 키워드 없이 가능하고, private 변수는 `[SerializeField]` 키워드를 사용하면 직렬화가 가능하다.

위에서 기본 데이터 타입은 별도의 키워드 없이 가능하다 했는데, public float 과 같은 경우 바로 Serialize 가능하기 떄문이다.

하지만 스크립트 직렬화는 아래와 같은 제약이 있다.

![image](https://user-images.githubusercontent.com/41438361/89634185-ee90e000-d8df-11ea-9391-dad6d048017d.png)

이걸 이용하면 사용자별로 게임에 필요한 변수들을 생성하고 받는데 좋다고 생각한다. 서버랑 연동해서 변수를 받는 법도 다음에 매니저님꼐 여쭤봐야겠다.

사용하기 위한 코드는 다음과 같다.

```C#
using UnityEngine;
using System.Collections;
//serialized 데이터 저장을 위해 필요한 namespace
using System;
using System.Runtime.Serialization.Formatters.Binary;
using System.IO;

public class GameController : MonoBehaviour{

  public static float fTime; //A1 일반변수
  public static int userLevel = 0; //A2 일반변수
  
  [Serializable] //B 직렬화 가능한 클래스
  public class PlayerData{
    public int userLevel;
    public float fTime;
  }
  
  void Start(){
    LoadData();
  }
  
  void Update(){
    //F5 키를 누르면 저장함수 호출
    if(Input.GetKeyDown(KeyCode.F5)){
      SaveData();
    }
    
    if(Input.GetKeyDown(KeyCode.F9)){
      userLevel++;
    }
    
    //fTime 계속 증가
    fTime += Time.deltaTime;
  }
  
  public void SaveData(){
    BinaryFormatter bf = new BinaryFormatter();
    FileStream file = File.Create(Application.persistentDataPath + "/playerInfo.dat");
    
    PlayerData data = new PlayerData();
    
    //A --> B에 할당
    data.userLevel = userLevel;
    data.fTime = fTime;
    
    //B 직렬화하여 파일에 담기
    bf.Serialize(file, data);
    file.Close();
  }
  
  public void LoadData(){
    BinaryFormatter bf = new BinaryFormatter();
    FileStream file = File.Open(Application.persistentDataPath + "/playerInfo.dat", FileMode.Open);
    
    if(file != null && file.Length >0){
      //파일 역직렬화하여 B에 담기
      PlayerData data = (PlayerData)bf.Deserialize(file);
      
      //B-->A에 할당
      userLevel = data.userLevel;
      fTime = data.fTime;
      
      Debug.Log(userLevel);
      Debug.Log(fTime);
    }
    
    file.Close();
  }
}

```





