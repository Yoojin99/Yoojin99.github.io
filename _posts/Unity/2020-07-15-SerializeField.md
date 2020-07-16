---
title:  "Unity - SerializeField"

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - Chatbot
  
tags:
  - C#
  - 공부
  - Unity
  - SerializeField
  
last_modified_at: 
---

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
