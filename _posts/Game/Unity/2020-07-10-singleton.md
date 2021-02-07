---
title:  "Singleton"
excerpt: "Singleton"

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
  
last_modified_at: 
---

Singleton Pattern(싱글톤 패턴)
==========

## 싱글톤 패턴

애플리케이션이 시작될 때 어떤 클래스가 **최초 한번만** 메모리를 할당하고(Static) 그 메모리에 인스턴스를 만들어 사용하는 디자인 패턴

생성자가 여러 차례 호출되더라도 실제로 생성되는 객체는 하나고 최초로 새엉된 이후에 호출된 생성자는 최초에 
생성한 객체를 반환한다.

즉, 싱글톤 패턴은 단 하나의 인스턴스를 생성해 사용하는 디자인 패턴이다. (똑같은 인스턴스를 만들어 내는 것이 아니라,
기존에 있는 인스턴스를 사용하게 한다)

## 쓰는 이유

고엊된 메모리 영역을 얻으면서 한 번의 new로 인스턴스를 사용하기 때문에 메모리 낭비를 방지한다.

또한 싱글톤으로 만들어진 클래스의 인스턴스는 전역 인스턴스이기 때문에(하나를 가지고 계속 써야 하니 당연하다) 다른 클래스의 인스턴스들이
데이터를 공유하기 쉽다.

DBCP(Database Connection Pool)처럼 공통된 객체를 여러개 생성해서 사용해야 하는 상황에서 사용한다.

또한 인스턴스가 절대적으로 한 개만 존재하는 것을 보증하고 싶을 때 사용한다.

한번 인스턴스를 생성하고 두 번째 이용부터는 객체 로딩 시간이 많이 줄어서 성능이 좋아진다는 장점이 있다.

## 싱글톤 패턴의 문제점

싱글톤 인스턴스가 너무 많은 일을 하거나 많은 데이터를 공유시킬 경우 다른 클래스의 인스턴스들 간에 결합도가 높아져 "개방-폐쇄 원칙"을 위반하게 된다.
이거는 객체 지향 설계 원칙에서 벗어나게 되는 것이다.

따라서 수정이 어려워지고 테스트하기 어려워진다.

꼭 필요한 경우가 아니면 지양한다.

## 멀티쓰레드에서 안전한 싱글톤 클래스, 인스턴스 만드는 법

### 1. Thread safe Lazy Initialization (게으른 초기화)

```C#
public class ThreadSafeLazyInitialization{
 
    private static ThreadSafeLazyInitialization instance; 
 
    private ThreadSafeLazyInitialization(){} //private 생성자로 외부에서 생성을 막음
     
    public static synchronized ThreadSafeLazyInitialization getInstance(){ //synchronized 키워드 사용해서 safe하게
        if(instance == null){
            instance = new ThreadSafeLazyInitialization();
        }
        return instance;
    }
 
}
```

이 방법은 synchronized가 성능저하를 발생시키므로 권장하지 않는다.

## 유니티에서의 싱글톤

유니티에서의 싱글톤은 2가지 버전으로 구분할 수 있다.

1. C# 싱글톤 버전
2. Monobehaviour 버전

### 1) C# 싱글톤 버전(스레드 세이프한 버전)

```C#
public sealed class Singleton
{
    private static Singleton instance = null;
    private static readonly object padlock = new object();

    private Singleton()
    {
    }

    public static Singleton Instance
    {
        get
        {
            lock (padlock)
            {
                if (instance == null)
                {
                    instance = new Singleton();
                }
                return instance;
            }
        }
    }
}
```

### 2) 제네릭을 사용한 방법

```C#
using System;  
using System.Reflection;  

public class Singleton<T> where T : class  
{   
   private static object _syncobj = new object();   
   private static volatile T _instance = null;  

   public static T Instance
   {  
       get  
       {  
           if (_instance == null)  
           {  
               CreateInstance();  
           }  
           return _instance;  
       }  
   }  

   private static void CreateInstance()  
   {   
      lock (_syncobj)  
      {  
          if (_instance == null)  
          {  
              Type t = typeof(T);  

              // Ensure there are no public constructors...  
              ConstructorInfo[] ctors = t.GetConstructors();  
              if (ctors.Length > 0)  
              {  
                  throw new InvalidOperationException(String.Format("{0} has at least one accesible ctor making it impossible to enforce singleton behaviour", t.Name));  
              }  

              // Create an instance via the private constructor  
              _instance = (T)Activator.CreateInstance(t, true);  
           }  
       }      
   }   
}  
```

## 2. Monobehaviour 버전

### 1) 간단한 방법

```C#
using UnityEngine;
using System.Collections;

public class Singleton: MonoBehaviour
{
    public static Singleton instance = null;              //Static instance of GameManager which allows it to be accessed by any other script.

    //Awake is always called before any Start functions
    void Awake()
    {
        //Check if instance already exists
        if (instance == null)
        {        
            //if not, set instance to this
            instance = this;
        }
        //If instance already exists and it's not this:
        else if (instance != this)
        {        
            //Then destroy this. This enforces our singleton pattern, meaning there can only ever be one instance of a GameManager.
            Destroy(gameObject);    
        }   

        //Sets this to not be destroyed when reloading scene
        DontDestroyOnLoad(gameObject);
    }
}
```

### 2) Gnereic을 사용한 방법

```C#
using UnityEngine;
using System.Collections;

/// <summary>
/// Be aware this will not prevent a non singleton constructor
///   such as `T myT = new T();`
/// To prevent that, add `protected T () {}` to your singleton class.
/// As a note, this is made as MonoBehaviour because we need Coroutines.
/// </summary>
public abstract class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    private static T _instance = null;
    private static object _syncobj = new object();
    private static bool appIsClosing = false;

    public static T Instance
    {
        get
        {
            if (appIsClosing)
                return null;

            lock (_syncobj)  
            {  
                if (_instance == null)
                {
                    T[] objs = FindObjectsOfType<T>();

                    if (objs.Length > 0)
                        _instance = objs[0];

                    if (objs.Length > 1)
                        Debug.LogError("There is more than one " + typeof(T).Name + " in the scene.");

                    if (_instance == null)
                    {
                        string goName = typeof(T).ToString();
                        GameObject go = GameObject.Find(goName);
                        if (go == null)
                            go = new GameObject(goName);
                        _instance = go.AddComponent<T>();
                    }
                }
                return _instance;
            }
        }
    }

    /// <summary>
    /// When Unity quits, it destroys objects in a random order.
    /// In principle, a Singleton is only destroyed when application quits.
    /// If any script calls Instance after it have been destroyed,
    ///   it will create a buggy ghost object that will stay on the Editor scene
    ///   even after stopping playing the Application. Really bad!
    /// So, this was made to be sure we're not creating that buggy ghost object.
    /// </summary>
    protected virtual void OnApplicationQuit()
    {
        // release reference on exit
        appIsClosing = true;
    }
}
```


### 사용 방법

```C#
public class Manager : Singleton<Manager> {
	protected Manager () {} // guarantee this will be always a singleton only - can't use the constructor!

	public string myGlobalVar = "whatever";
}
```
