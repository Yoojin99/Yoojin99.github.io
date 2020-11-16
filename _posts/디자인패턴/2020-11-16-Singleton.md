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

### 전역적인 접근점 제공

로깅, 콘텐츠 로딩, 게임 저장 등 파일 시스템 클래스에 접근하는 시스템에서 파일 시스템 래퍼 클래스를 사용한다. 이들 시스템에서 파일 시스템 클래스 인스턴스르 따로 생성할 수 없으므로
싱클턴 패턴은 하나의 인스턴스만 생성하는 것에 더해, 그 인스턴스를 전역에서 접근할 수 있는 메서드를 제공한다. 

```c++
class FileSystem {
  public:
    static FileSystem& instance() {
      if(instance_ NULL) {
        instance_ = new FileSystem();
      }
      
      return *instance_;
    }
}
```
