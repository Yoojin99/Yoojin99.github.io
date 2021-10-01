---  
layout: post  
title: "[iOS] - Xcode Build System 이해하기"  
subtitle: ""  
categories: app
tags: app-ios xcode build
comments: true  
header-img: 

---  
  
> `Xcode에서 빌드가 어떻게 이루어지는지 알아보자.`  

---

모든 Swift 프로그램은 실제 기기에 돌아가기 전에 여러 과정을 거친다. 이 프로세스는 주로 Xcode 빌드 시스템에 의해 이루어진다.

## Swift code가 어떻게 하드웨어가 실행할 수 있는 형태로 변환되나?

모든 컴퓨터 시스템은 두 부분으로 이루어져 있다. 바로 **소프트웨어**와 **하드웨어**다.

하드웨어는 컴퓨터의 물리적인 부분으로, 컴퓨터, 키보드가 여기 포함된다. 하드웨어는 하드웨어가 어떻게 동작하라는 명령어 집합으로 이루어진 소프트웨어에 의해 통제된다. 소프트웨어가 프로세스와 하드웨어가 조화롭게 동작하도록 지휘하기 때문에, 하드웨어나 소프트웨어는 어느 것도 혼자서 동작할 수 있다.

개발자로서, 우리는 보통 소프트웨어만 다룬다. 하지만 하드웨어는 우리가 swift로 작성하는 코드를 바로 이해할 수는 없다.하드웨어는 0, 1을 나타내는 전기 신호만을 받아들인다.

그렇다면 대체 swift 프로그램은 어떻게 하드웨어가 실행할 수 있는 형태로 바뀌는 것일까? 바로 **Language processing system** 덕분이다. 

## Language Processing System

직역하자면 언어 처리 시스템으로, 소스 언어로 작성된 명령어에서 실행가능한 프로그램을 생성할 수 있게 하는 프로그램의 컬렉션이다. 이 시스템은 개발자가 기계어 대신에 사람이 이해하기 쉬운 high level 언어를 사용할 수 있게 해서 개발의 복잡도를 낮춰줬다.

우리가 iOS나 macOS 개발에 사용하고 있는 언어 처리 시스템은 **Xcode Build System**이다.

## Xcode Build System

Xcode의 주 목표는 실행가능한 프로그램을 생성할 다양한 작업들을 실행하는 것이다.

언어 처리 시스템의 크게 아래의 다섯 부분으로 이루어져 있다.

* Preprocessor
* Compiler
* Assembler
* Linker
* Loader

얘네들은 아래의 그림처럼 동작한다.

![image](https://user-images.githubusercontent.com/41438361/135569970-49952b5c-bd50-479f-93fb-9cec7972215f.png)

이제 각 단계를 자세히 보자.

## Preprocessing

전처리 단계의 목적은 프로그램을 컴파일러가 이해할 수 있는 무언가로 바꿔주는 것이다. Macro를 정의로 대체하고, 의존성을 확인하며 전처리기 지시문(Preprocessor directives)를 해결한다. 

Swift 컴파일러가 전처리기가 없다는 것을 고려하면, Swift 프로젝트에서 매크로를 정의하는 것은 허용되지 않는다. 그래도 Xcode 빌드 시스템은 이를 부분적으로 보상하고, Build Settings에서 설정할 수 있는 Active Compilation Conditions를 통해 전처리를 수행한다.

<img width="419" alt="image" src="https://user-images.githubusercontent.com/41438361/135570433-e4a17c85-110c-4629-af80-6642097fcb77.png">

Xcode는 llbuild라는 낮은 레벨 빌드 시스템을 사용해서 의존성을 해결한다. 옹픈 소스이고, 자세한 내용은 [swift-llbuild 깃헙 페이지](https://github.com/apple/swift-llbuild)에서 확인할 수 있다.

*참고로 Macro는 정해진 순서에 따라 어떻게 특정한 입력 시퀀스(문자열 등등)가 출력 시퀀스로 매핑되어야 하는지를 정의하는 규칙/패턴을 말한다. 하나의 매크로를 특정 출력 시퀀스로 바로 만들어내는 매핑 과정은 매크로 확장이라고 한다. 원래는 C/C++등 매크로를 지원하는 언어에서 약식 문법을 정의하면, 컴파일러가 컴파일을 하기 전에 약식 문법으로 정의된 코드를 원래 코드로 변환한 후 컴파일한다. 이런 기능이 처음 도입된 Lisp에서는 원래 코드로 변환하는 키워드가 원래 코드를 크게 확장한다는 의미에서 macroexpand라 했고, 이 확장 규칙을 macro라 했는데 이 용어가 이후의 언어들에도 적용이 된 것이다.*

## Compiler

컴파일러는 한 언어로 작성된 소스프로그램을 다른 언어의 타겟 프로그램으로 의미가 같게 매핑하는 프로그램이다. 즉 Swift/Objective-C와 C/C++ 코드를 의미가 퇴색되지 않게 기계어 코드로 바꿔주는 것이다.

Xcode는 두 개의 다른 컴파일러를 사용한다. 하나는 Swift, 다른 하나는 Objective-C, Objective-C ++, 그리고 C/C++ 파일에 사용한다.

`clang`은 C 언어들을 위한 애플의 공식 컴파일러다. 얘도 오픈 소스이고, [여기](https://github.com/apple/swift-clang)에서 확인할 수 있다.

`swift`는 Swift 소스 코드를 컴파일하고 실행할 수 있게 Xcode에 의해 사용되는 Swfit 컴파일러 실행 파일이다.


참고로 Xcode의 build setting 메뉴에서 swift와 obj-c 컴파일러를 설정하는 메뉴가 있는 것을 확인할 수 있다.

<img width="289" alt="image" src="https://user-images.githubusercontent.com/41438361/135572145-7619c9b5-1ff1-45d5-9e52-77583c4f0c42.png">

컴팡일러에서는 아래의 작업들이 일어난다.

![image](https://user-images.githubusercontent.com/41438361/135571583-1a956035-c4f9-4539-b144-e0905e396505.png)

컴팡일러는 두 개의 메인 파트로 이루어져 있다. 바로 front end, back end다.

Front end 부분은 소스 프로그램을 의미나 타입 정보 없어 여러 조각으로 분리하고, 여기에 문법적 규칙을 강제한다. 그리고 컴파일러는 이 구조를 이용해서 중간 표현을 생성한다. 또한 소스 프로그램에 대한 정보를 가지고 있는 symbol table을 생성하고 관리한다.

Symbol은 코드나 데이터의 조각의 이름이다.

Symbol table은 내가 지은 변수, 함수, 클래스의 이름을 저장하고 있고, 각 symbol은 데이터의 특정 조각으로 매핑된다.

Swift 컴파일러의 경우에 intermediate representation은 Swift Intermediate Language(SIL)로 불린다. 이는 이후의 분석과 코드의 최적화에 사용된다. 그리고 기계어 코드를 바로 Swift Intermediate Language에서 생성할 수 없어서, SIL은 LLVM Intermediate Representation으로 한 번 더 변환된다.

Back end 단계에서, intermediate representation은 어셈블리 코드로 변환된다.

## Assembler

Assembler는 사람이 읽을 수 있는 어셈블리 코드를 재배치가 가능한 machine code다. 어셈블러는 코드와 데이터의 컬렉션인 Mach-O 파일들을 생성한다.

Machine code는 CPU가 바로 실시킬 수 있는 명령어의 집합을 표현하는 숫자 언어이다. 이게 재배치가 가능하다고 하는 이유는 object file이 메모리 주소 공간의 어디에 있더라도 명령이 이 공간에 대해 상대적으로 실행되기 때문이다. 

Mach-O 파일은 오브젝트 파일, 실행 파일, 라이브러리에 사용되는 iOS, macOS의 특별한 **파일 형식**이다. 이 형식이라는 게 특별한 형식 같은 건 아니고 단지 이 파일은 iOS 기기의 ARM 프로세서나 맥의 Intel 프로세서에서 실행될 의미 단위로 묶인 byte stream이라고 한다. 바이트 순서, CPU 타입, chunk 크기 같은 메타 정보가 들어간다.

Mach-O는 Mach Object file format의 줄임말고, Mach은 커널이다. Mach-O는 마이크로 커널(운영 체제에 추가되어야 하는 매커니즘을 최소한으로 제공하는 초소형 커널)로, macOS, iOS, iPadOS, tvOS, watchOS의 XNU가 Mach을 기반으로 했다.

Mach-O 파일의 구조에 대한 잘 정리된 그림이 있어서 가져왔다.(링크 하단에 첨부)

https://i.imgur.com/Q4w9qLp.png![image](https://user-images.githubusercontent.com/41438361/135578769-92749b8f-eb43-4f6c-a9da-bcd2a1a3b750.png)


## Linker

링커는 iOS나 macOS 시스템에서 실행할 수 있는 단일한 Mach-O 실행파일을 만들기 위해 다양한 오브젝트 파일과 라이브러리를 한데 합치는 컴퓨터 프로그램이다. 링커는 입력으로 두 종류의 파일을 받는다. 바로 assembler 단계에서 나온 오브젝트 파일과 여러 타입(`.dylib`, `.tbd`, `.a`)의 라이브러리이다.

어셈블러와 링커가 모두 결과로 Mach-O 파일을 생성하는 것을 알아차렸을 것이다. 물론 둘 사이에는 차이가 있다.

어셈블리 단계에서 나온 오브젝트 파일은 완성된 형태가 아니다. 이들 중 몇몇은 다른 오브젝트 파일이나 라이브러리에 참조를 두고 있는 조각들을 포함하고 있다. 예를 들어 코드에서 `printf`를 사용한다면, `printf` 함수가 구현된 libc 라이블러리와 이 심볼을 연결하는 애가 링커다. 컴파일러 단계에서 생성된 symbol table을 사용해서 여러 다른 오브젝트 파일과 라이브러리에 걸쳐진 참조 문제들을 해결한다.

Xcode에서 Swift 프로젝트를 빌드할 때 “undefined symbol” 에러를 경험한 적이 있을 것이다.


## Loader

마지막으로, 로더는 os의 일부로, 프로그램을 메모리에 갖고와서 실행한다. 로더는 프로그램을 실행하는 데 필요한 메모리 공간을 할당하고 레지스터를 초기 상태로 만든다.

* 참고
* https://eunjin3786.tistory.com/323
* https://www.vadimbulavin.com/xcode-build-system/
* https://imgur.com/a/PbN8H#0
