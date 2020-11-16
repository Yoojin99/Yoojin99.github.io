---
title:  "싱글턴 패턴(Singleton Pattern)"
excerpt: ""

toc: true
toc_sticky: true
toc_label: "페이지 주요 목차"
header:
  teaser: 
  
  
categories:
  - Design Pattern
  
tags:
  - Singleton
  
last_modified_at: 2020-11-16
---

Singleton Pattern은 

`오직 한 개의 클래스 인스턴스만을 갖도록 보장하고, 이에 대한 전역적인 접근점을 제공한다.` 라고 GoF의 디자인 패턴 책에 나와 있다.

즉 인스턴스가 프로그램 내에서 오직 하나만 생성되는 것을 보장하고, 프로그램 어디서든 이 인스턴스에 접근할 수 있도록 하는 패턴이란 뜻이다. 인스턴스가 사용될 때 똑같은 인스턴스를 여러 개 만들지 않고, 기존에 생성한 인스턴스를 가용하게 한다.


## Singleton pattern

### 오직 한 개의 인스턴스만을 갖도록 보장.

인스턴스가 여러개면 제대로 작동하지 않는 경우가 있다.(외부 시스템과 상호작용하며 전역 상태를 과닐하는 클래스 - 파일 시스템 API)

파일 시스템 클래스로 들어온 호출이 이전 작업 전체에 대해 접근할 수 있어야 한다. 아무데서나 파일 시스템 클래스 인스턴스를 만들 수 있으면 다른 인스턴스에서 어떤 작업을 진행중인지 알 수 없다. 
이를 싱글턴으로 만들면 클래스가 인스턴스를 하나만 가지도록 컴파일 단계에서 강제할 수 있다.

최초 이후에 호출된 생성자는 최초의 객체를 리턴하는 방식이다. 인스턴스가 하나만 있기 때문에 공통적으로 쓰이는 자원을 관리, 저장하는 것을 만들때 자주 사용된다.

특히 게임에서는 gamestatemanager라든가 resourceManager 등등 사용된다. c#나 java는 gc가 되는데, c++에서는 메모리 관련 이슈가 있다.

### 전역적인 접근점 제공

로깅, 콘텐츠 로딩, 게임 저장 등 파일 시스템 클래스에 접근하는 시스템에서 파일 시스템 래퍼 클래스를 사용한다. 이들 시스템에서 파일 시스템 클래스 인스턴스르 따로 생성할 수 없으므로
싱클턴 패턴은 하나의 인스턴스만 생성하는 것에 더해, 그 인스턴스를 전역에서 접근할 수 있는 메서드를 제공한다. 

```c++
class FileSystem {
  public:
    static FileSystem& instance() {
      // 게으른 초기화
      if(instance_ NULL) {
        instance_ = new FileSystem();
      }
      
      return *instance_;
    }
  private:
    FileSystem() {}
    static FileSystem* instance_;
}
```

좀 더 간단한게(내가 이해하기 쉬운게) 아래의 코드다.

```c++
class Singleton {
  private:
    Singleton(){};
    static Singleton* instance;
  public:
    static Singleton* GetInstance() {
      return instance;
    }
};

Singleton* Singleton::instance = nullptr;
```

private 생성자와 GetInstance 메소드를 통해 자기 자신을 반환한다. 하지만 메모리 관련 문제가 있다. static 멤버로 instance를 갖고 있기 때문에 data 영역에 할당되고 메모리를 계속 차지하고 있다.

또한 다른 전역 객체의 생성자에서 참조할 경우 문제가 발생할 수도 있다. 이런 현상이 발생하는 이유는 c++에서는 전역인 애들의 생성 순서가 명확히 정해져 있지 않아서 그렇다.
즉 A가 B를 사용하려고 했는데 A가 B 이후에 생성될 수도 있다. 이래서 객체의 생성 시점을 변경해야 한다. 

늦은 초기화 개념을 도입한 것이 아래의 코드이다.

```cpp
class Singleton {
  private:
    Singleton() {};
    ~Singleton() {};
    
    static Singleton* instance;
    
  public:
    static Singleton* GetInstance() {
      if(instance == nullptr) {
        instance = new Singleton();
      }
      
      return instance;
    }
    
    static void PurgeInstance() {
      delete instance;
      instance = nullptr;
    }
};

Singleton* Singleton::instance = nullptr;
```

위의 코드는 `GetInstance()`를 호출할 때 instance가 생성이 된다. 이렇게 코드를 짜면 자원을 메모리와 함께 효율적으로 관리할 수 있다.

new를 통해 instance를 만들었기 때문에 동적메모리 할당이 이루어졌다.(heap 영역에 올라갔을 것)

`PurgeInstance` 메서드는 중간에 싱글톤 객체를 해제하고 싶을 때 호출하면 된다. 예를 들어 게임에서 player를 싱글톤으로 생성하면 한 사이클을 마쳤을 때 중간에 해제하고 다시 생성할 수도 있다.


