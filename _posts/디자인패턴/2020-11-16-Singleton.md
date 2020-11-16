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

다시 맨 위의 코드로 돌아가 보겠다.

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

위에서 봤던 늦은 초기화 == 게으른 초기화임을 알 수 있다.

위 코드에서 `instance_` 정적 멤버 변수는 클래스 인스턴스를 저장한다. ㅅㅇ성자가 private이어서 밖에서는 생성할 수 없다. `instance()` 정적 메서드는 **코드 어디에서나 싱글턴 인스턴스에 접근할 수 있게 하고, 싱글턴을 실제 필요할때까지 인스턴스 초기화를 미루는 역할(게으른 초기화)**도 한다.

아래 같이도 코드를 짤 수 있다.

```cpp
class FileSystem {
  public:
    static FileSystem& instance() {
      static FileSystem* instance = new FileSystem();
      return *instance;
    }
    
  private:
    FileSystem() {}
}
```

### 사용하는 이유? 

1. 한 번도 사용하지 않는다면 아예 인스턴스를 생성하지 않는다.
2. 런타임에 초기화 된다. 
  정적 멤버 변수는 자동 초기화 되는 문제가 있다. 
3. 싱글턴을 상속할 수 있다.
  파일 시스템 래퍼가 크로스 플랫폼을 지원해야 한다면 추상 인터페이스를 만든 후 플랫폼마다 구체 클래스를 만들면 된다.
  
상위 클래스를 아래와 같이 만든다.

```cpp
class FileSystem {
  public:
    virtual ~FileSystem(){}
    virtual char* readFile(char* path) = 0;
    virtual void writeFile(char* path, char* contents) = 0;
};
```

그리고 플랫폼 별로 아래와 같이 하위 클래스를 정의한다.

```cpp
class PS3FileSystem : public FileSystem {
  public:
    virtual char* readFile(char* path) { // 소니의 파일 IO API를 사용한다... } 
    virtual void writeFile(char* path, char* contents) { // 소니의 파일 IO API를 사용한다... }
}

class WIIFileSystem : public FileSystem {
  public:
    virtual char* readFile(char* path) { // 닌텐도의 파일 IO API를 사용한다... } 
    virtual void writeFile(char* path, char* contents) { // 닌텐도의 파일 IO API를 사용한다... }
}
```

그리고 FileSystem 클래스를 싱글턴으로 만든다.

```cpp
class FileSystem {
  public:
    static FileSystem& instance();
    
    virtual ~FileSystem(){}
    virtual char* readFile(char* path) = 0;
    virtual void writeFile(char* path, char* contents) = 0;
    
  protected:
    FileSystem(){}
};
```

이제 중요한 인스턴스를 생성하는 부분이다.

```cpp
FileSystem& FileSystem::Instance() {
  #if PLATFROM == PLAYSTATION3 
    static FileSystem* instance = new PS3FileSystem(); 
  #elif PLATFORM == WII 
    static FileSystem* instance = new WiiFileSystem(); 
  #endif 
    return *instance;
}
```

위와 같이 간단하게 시스템에 맞는 파일 시스템 객체를 만들게 할 수 있다. FileSystem::Instance()를 통해서 파일 시스템에 접근하기 때문에 플랫폼 전용 코드는 FileSystem 클래스 내부에 숨겨놓ㅇ르 수 있다.

### 싱글턴이 문제인 이유

1. 전역 변수나 다름없다. 전역 변수는 코드를 이해하기 힘들게 한다. 
2. 전역 변수는 커플링을 조장한다. 인스턴스에 대한 접근을 통제함으로써 커플링을 통제할 수 있다. (커플링 : 어떤 모듈을 변경할 때 다른 모듈의 변경이 요구된다면 커플링이 존재하는 것이다. 중복은 커플링을 항상 수반한다. 또한 어떤 모듈이 다른 모듈의 코드를 사용할 때 발생한다.)
3. 전역 변수는 멀티 스레딩같은 동시성 프로그래밍에 알맞지 않다. 
4. 게으른 초기화는 제어할 수 없다. : 게으른 초기화는 좋은 방법이지만 게임은 다르다. 시스템을 초기화할 때 메모리 할당, 리소스 로딩 등 할 일이 많아 시간이 많이 소요될 수 있다. 이에 많은 시간이 소요된다면 초기화 시점을 제어해야 한다. 처음 로딩을 할 때 게으른 초기화를 하게 만들면 전투 도중 초기화가 시작되는 바람에 버벅되는 현상이 발생한다.

### 대안 - 클래스가 꼭 필요한가?

게임 코드의 싱글턴 클래스 중에는 애매하게 다른 객체 관리용으로만 존재하는 '관리자'가 많다. 

### 오직 한 개의 클래스 인스턴스만 갖도록 보장하기

누구나 전역에서 클래스 인스턴스에 접근할 수 있게 되면 구조가 취야개진다

전역 접근 없이 클래스 인스턴스만 한 개로 보장할 수 있는 방법이 몇가지가 있다.

```cpp
class FileSystem {
  public:
    FileSystem() {
      assert(!instantiated_); //인수 값이 참이면 아무것도 안하지만 거짓이면 그 자리에서 코드를 중지한다.
      instantiated_ = true;
    }
    
    ~~FileSystem() {
      instantiated_ = false;
    }
    
  private:
    static bool instantiated_;
};

bool FileSystem::instantiated_ = false;
```

### 인스턴스에 쉽게 접근하기

쉬운 접근성은 싱글턴을선택하는 큰 이유다. 싱글턴을 사용하면 여러 곳에서 사용해야 하는 ㄱㄱ체에 쉽게 접근할 수 있다.

1. 상위 클래슬부터 얻기
  몬스터나 다른 게임 내 객체가 상속받는 GameObject라는 상위 클래스가 있다. 많은 클래스들이 이를 상속해서 GameObject 상위 클래스에 접근할 수 있다.
  
  아래의 코드는 GameObject를 상속받은 곳에서만 getLog()를 통해 로그 객체에 접근할 수 있다.
  
  ```cpp
  class GameObject {
    protected:
      Log& getLog() {return log_;}
      
    private:
      static Log& log;
  };
  
  class Enemy: public GameObject {
    void doSth() {
      getLog().write("log this.");
    }
  }
  ```
2. 이미 전역인 객체로부터 얻기

log, filesystem, audioplayer를 각각 싱글턴으로 만드는 대신 아래 코드와 같이 짠다.

```cpp
class Game {
  public:
    static Game& instance() {return instance_;}
    
    Log& getLog {return *log_}
    FileSystem& getFileSystem() {return *fileSystem_;}
    AudioPlayer& getAudioPlayer() {return *audioPlayer_;}
    
  private:
    static Game instance_;
    Log* log_;
    FileSystem* fileSystem_;
    AudioPlayer* audioPlayer_;
}
```

이제 Game 클래스 하나만 전역에서 접근할 수 있다. 다른 시스템에 접근하려면 알와 같이 함수를 호출하면 된다.

```cpp
Game::Instance().getAudioPlayer().play(Music_3);
```


[출처](https://boycoding.tistory.com/109)




