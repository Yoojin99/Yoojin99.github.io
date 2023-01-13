---  
layout: post  
title: "[iOS] - ARM Architecture"  
subtitle: ""  
categories: app
tags: app-ios runloop
comments: true  
header-img: 

---  

# ARM Architecture

## ARM 아키텍처

**컴퓨터 프로세서를 위한 RISC (Reduced Instruction Set Computing) 아키텍처의 일환. Advanced RISC Machines**

ARM 프로세서는 굉장히 적은 instruction, transistor 만을 필요로 하고, 크기가 작기 때문에 작은 기기에도 탑재 가능하다.

* 장점
    * 내장된 보안성
    * 높은 성능과 에너지 효율성
    * 전세계적인 Ecosystem, 영향력
* 기능
    1. Multiprocessing System : ARM 프로세서들은 하나 이상의 프로세서가 수행될 수 있는 multiprocessing 시스템을 지원한다. ARMv6K 는 하드웨어에서 4개의 CPU 를 돌릴 수 있게 지원하고 있다.
    2. Tightly Coupled Memory : ARM 프로세서의 메모리는 강하게 결합되어 있는데, 이는 굉장히 빠른 응답 시간을 보장한다.
    3. Memory Management : ARM 프로세서는 management section 이 있다. 이 섹션 안에는 Memory Management Unit 과 Memory Protection Unit 이 있다. 이 시스템을 사용해서 메모리를 효율적으로 관리한다.
    4. Thumb-2 Technology : Thumb-2 기술은 2003 년에 등장해서 다양한 길이의 명령어 set 을 생성하는데 사용됐다. 초기의 16 비트 길이의 instruction 을 32 비트 길이로 확장시켜준다.
    5. One cycle execution time : ARM 프로세서는 CPU 의 각 명령어에 최적화되어 있다. 고정된 길이의 각 명령어는 현재 instruction 을 수행하기 전에 앞으로 실행할 명령어를 불러올 수 있게 한다. ARM 은 한 cycle 의 CPI (Clock Per Instruction) 을 갖고 있다.
    6. Pipelining : Pipeline 을 사용해서 명령어들을 병행적으로 처리할 수 있다. 명령어들은 쪼개져서 하나의 pipeline 에서 디코딩된다. Pipeline 을 사용하면 throughput (프로세스가 CPU를 얻기까지 기다렸다가 수행되고 CPU를 반납한 시간) 이 개선된다.
    7. Large number of registers : ARM 프로세서의 많은 레지스터들은 메모리 interaction 을 방지한다. 레지스터들은 데이터와 주소를 포함하고, 모든 작업의 로컬 메모리 저장소처럼 동작한다.

## RISC

**Reduced Instruction Set Computer, 작고 최적화된 명령어들의 집합을 사용하는 microprocessor 아키텍처**

* 특징
    * one cycle execution time : RISC 프로세서들은 한 사이클의 CPI (Clock per instruction) 을 갖고 있다. 이는 CPU 의 각 연산들이 최적화되어 있기 때문에 가능하다.
    * pipelining : 작업들 / 명령어들의 일부를 동시에 실행할 수 있게 하는 기술
    * large number of registers : 메모리와의 상호작용을 최대한 적게 하기 위해 많은 register 를 포함하고 있다.

### RISC vs CISC

* RISC : **Reduced Instruction** Set Computer. 한 clock 사이클 내에 실행할 수 있는 간단한 명령어만을 사용한다.
* CISC : **Complex Instruction** Set Computer. 최대한 적은 줄의 명령어를 사용하기 위해 복잡한 명령어를 사용한다.

가령 아래의 그림에서 (2,3) 지점, (5,2) 지점에 있는 두 수를 곱하기 위해서 CISC, RISC 는 각각 다음의 명령어를 수행한다.

![image](https://user-images.githubusercontent.com/41438361/211720685-baae6922-455b-432a-8aca-6b38904b380a.png)

| CISC | RISC |
| --- | --- |
| `MULT 2:3, 5:2` | `LOAD A, 2:3`, `LOAD B, 5:2`, `PROD A, B`, `STORE 2:3, A` |
| 1줄 | 4줄 |
| 복잡한 명령어를 하드웨어에서 바로 실행. 하드웨어가 강조됨. | "단축된 명령어"가 복잡한 명령어에 비해 하드웨어 공간, 레지스터의 공간을 더 적게 사용한다. 소프트웨어가 강조됨. |
| Memory-to-memory : "LOAD", "STORE" 명령어가 하나의 명령어 안에 포함되어 있다. | Register to register : "LOAD", "STORE" 명령어가 독립된 채로 존재한다. |
| 적은 코드 줄 수 | 많은 코드 줄 수 |
| High cycles per second | Low cycles per second |
| 명령어를 실행하는데 multi-clock 소요 | 명령어를 실행하는데 single-clock 소요 |

"LOAD"와 "STORE" 명령어를 분리하는 것이 실제로 컴퓨터가 해야 할 일을 적게 만들어준다. CISC 스타일의 "MULT" 명령어가 실행되면 프로세서는 자동으로 레지스터를 비운다. 만약 이 결과가 다른 연산에서 사용되어야 한다면 프로세서는 메모리에서 데이터를 불러와서 레지스터에 저장해야 한다. 반면 RISC 에서 연산 결과는 다른 값이 로드되기 전까지 항상 레지스터에 남아있다.

### 성능 계산 공식

![image](https://user-images.githubusercontent.com/41438361/211722365-0baf3f15-a4cf-4637-8edb-fb2ba75b405e.png)

위 공식은 컴퓨터의 성능을 측정할 때 사용된다.

* CISC : `instructions / program` 을 최소화하는 대신 `cycles / instruction` 을 희생한다.
* RISC : `cycles / instruction` 을 최소화하는 대신 `instructions / program` 을 희생한다.

## Apple platform

모든 iOS 기기는 ARM 아키텍처를 기반으로 하는 프로세서에서 실행되고, iOS 시뮬레이터는 실행하고 있는 macOS CPU 아키텍처를 상속받는다.

ARM 아키텍처는 시간에 따라 발전하면서 여러 개의 버전이 생겼는데, ARMv6 에서 ARMv7, ARM64 순으로 버전들이 나왔다. 이 버전들마다 사용하는 명령어들도 조금씩 달라졌다.

* 일반 기기
    * armv6, armv7, armv7s : 오래된 버전의 iPhone, iPad 기기에서 사용된 32 bit 아키텍처
    * arm64 : 64 bit 아키텍처
    * arm64e : iPhone 8 이후의 버전에서 주로 사용된 64 bit 아키텍처
* 시뮬레이터
    * i336
    * x86\_64 : 64 bit intel mac simulator
    * arm64 : Apple Silicon mac simulator

### 기기별 ARM

[https://www.innerfence.com/howto/apple-ios-devices-dates-versions-instruction-sets](https://www.innerfence.com/howto/apple-ios-devices-dates-versions-instruction-sets)에서 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/211725511-4b88c6ad-3e21-4371-8741-fb62f3d788cc.png)

### 기기별 호환성

[https://developer.apple.com/kr/support/required-device-capabilities/#ipad-devices](https://developer.apple.com/kr/support/required-device-capabilities/#ipad-devices)에서 확인할 수 있다.

![image](https://user-images.githubusercontent.com/41438361/211725959-33a11574-a970-4854-8578-84e660eac37a.png)

## Fat Binaries / Frameworks

다양한 기기 / 아키텍처 에서 호환되는 바이너리를 생성하는데 iOS, iOS 시뮬레이터를 모두 지원하는 미리 빌드된 프레임워크를 shipping 하고 싶을 때는 어떻게 해야 할까?

한 방법은 두 개의 별도의 프레임워크를 shipping 하는 것이다. iOS 프레임워크는 지원하려는 iOS 에서 사용하는 모든 아키텍처 (특히 64 bit ARM) 를 포함해야 하고, iOS Simulator 프레임워크는 지원하려는 iOS Simulator 에서 사용하는 모든 아키텍처 (특히 64 bit Intel, 64 bit ARM) 를 포함해야 한다. 그러면 클라이언트는 빌드 타겟에 기반해서 올바른 프레임워크를 link 할 것이다. 이 프로세스는 관리하기 어렵고, 생각한 대로 잘 되지 않는 경우가 있다.

이 이슈를 해결하기 더 좋은 방법은 생성된 바이너리들을 결합해서 더 큰 바이너리를 만드는 것이다. 이를 흔히 "Universal Binaries", 혹은 "Fat Binaries" 라 부르고, 이것들은 `lipo` tool 을 사용해서 하나 이상의 플랫폼에서 실행할 수 있는 코드를 포함할 수 있다.

1. iOS SDK 프레임워크를 빌드하고, iOS 아키텍처를 타겟팅한다.
2. iOS Simulator SDK 프레임워크를 빌드하고, macOS 아키텍처를 타겟팅한다.
3. `lipo` 를 사용해서 두 개의 프레임워크를 배포를 위한 하나의 프레임워크로 결합한다.

일반적인 Universal Binary Framework 의 구조는 아래와 같다.

```
UniversalFramework/
├── arm64-slice
└── x86_64-slice
```

개발 단계에서는 굉장히. 좋은 방법일 수 있지만 앱을 AppStore 에 제출할 때 문제가 되기도 하는데, simulator slice (thinning 의 app slicing 에서 말하는 번들 조각을 의미하는 것 같다) 을 포함하고 있기 떄문에 불필요하게 앱의 바이너리 크기를 증가시키고 있고, 이것이 바로 x86\_64 조각을 포함한 dependency 를 가진 앱을 AppStore 에 업로드할 때 [유명한 에러](http://www.openradar.me/radar?id=6409498411401216) 와 함께 app rejection 을 받게 되는 것이다.

> CocoaPods, Carthage 와 같은 대부분의 dependency 매니저들은 AppStore 에 제출하기 전에 자동으로 simulator slice 를 제거한다.

Apple 은 기기와 Simulator 를 동시에 지원하기 위한 새로운 방법으로 XCFramework bundle type 을 소개했다.

## XCFrameworks

Universal Binaries 접근법은 하나의 slice 당 하나의 아키텍처만 지원할 수 있다는 다른 한계가 있었는데, 이 말은 하나의 Universal Framework 에서 여러 플랫폼의 동일한 아키텍처들을 지원하지 못한다는 말과 같다.

2020 년 11월에 Apple 은 arm64 아키텍처를 사용하는 Apple Silicon 프로세서들을 탑재한 새로운 mac 을 출시했다. `lipo` 접근법의 한계 때문에, arm64 시뮬레이터를 지원할 수 없었고 아래 형태의 설정은 가능하지 않았다.

```
UniversalFramework/
├── ios-arm64-slice (기존에 있었던 것)
├── ios-simulator-arm64-slice (arm64 macOS 로 추가되어야 하는 것)
├── ios-simulator-x86_64-slice (기존에 있었던 것)
└── macos-arm64-slice
```

이런 이유 때문에 최근의 Xcode release 에 아래와 같은 이상한 에러가 등장했다.

```
Building for iOS Simulator, but the linked and embedded framework 
'MyFramework.framework'  was built for iOS + iOS Simulator.
```

위 에러가 뜨는 이유는 바이너리 프레임워크가 동일한 아키텍처를 위한 다른 코드들을 여러 곳에서 포함하고 있는데, Xcode 는 이를 어떻게 다뤄야 할 지 모르기 때문이다.

여기에서 XCFrameworks 가 등장한다. XCFrameworks 는 특정 구조를 갖고 있는 새로운 컨테이너 형태다. Universal Frameworks 가 플랫폼 SDK 에 대한 정보를 모르는 slice 들을 포함하고 있는 반면, XCFramework 는 플랫폼에 의해 구성된 slice 들을 갖고 있다. 이렇게 플랫폼을 알고 있다는 점 때문에 XCFramework 는 동일한 아키텍처를 사용하는 하나 이상의 플랫폼을 타겟팅할 수 있게 되어 문제를 해결한다.

`Framework.xcframework` 컨텐츠를 살펴보면 아키텍처가 아니라 플래폼에 의해 묶여진 별개의 프레임워크를 갖고 있는 것을 볼 수 있다.

```
Framework.xcframework/
├── ios-arm64_armv7-slice
├── ios-x86_64-maccatlyst-slice
├── ios-arm64_x86_64-simulator-slice
└── macos-arm64_x86_64-slice
```

## Compilation Directives

특정 플랫폼 / 프로세서 타입을 위한 코드를 작성할 때 적절한 컴파일 조건문을 사용해서 코드를 구분할 수 있다.

``` swift
#if arch(arm64)
   //  arm64 architecture code here.
#elseif arch(x86_64)
   //  x86_64 architecture code here.
#endif
```

만약 `arm64` 아키텍처일 경우에만 포함시키고 싶은 라이브러리가 있을 때 Swift Package Manager 를 사용하면 굉장히 쉽게 처리할 수 있는데, 특정 dependency 를 compilation directive block 안에 감싸서 처리할 수 있다.

``` swift
#if arch(arm64)
let architectureSpecificPackageDependencies: [Package.Dependency] = [Apple Silicon Dependencis]
#else
let architectureSpecificPackageDependencies: [Package.Dependency] = [Intel Package dependencies]
#endif
let package = Package(
    ...
    dependencies: [...] + architectureSpecificPackageDependencies,
    ...
```

## Build Settings

Xcode 는 바이너리 아키텍처 설정을 커스터마이징 할 수 있는 섹션을 제공한다.

![image](https://user-images.githubusercontent.com/41438361/211740099-aa2534ba-8742-4fdb-8b80-34fb9195efed.png)

### Architectures `(ARCHS)`

빌드될 아키텍처들의 목록이다. 주로 플랫폼에 의해 이미 설정되어 있을 것이다. 만약 하나 이상의 아키텍처가 명시되어 있다면 universal 바이너리가 생성된다. 주로 `$(ARCHS_STANDARD)` 로 설정되어 있어서 기기로 빌드할 때는 `armv7 arm64`가 되고 시뮬레이터를 빌드할 때는 `x86_64 arm64` 가 된다.

Xcode 가 앱을 빌드할 때/아카이빙 할 때 link 하는 아키텍처들은 아래의 커맨드를 통해 확인할 수 있다.

```
# list the supported architectures for the simulator
xcodebuild -showBuildSettings -scheme MyApp -sdk iphonesimulator | grep ARCHS
```

```
# list the supported architectures for the real device.
xcodebuild -showBuildSettings -scheme MyApp -sdk iphoneos | grep ARCHS
```

### Build Active Architecture Only (`ONLY_ACTIVE_ARCH`)

M1 mac 이 등장하면서 발생했던 이슈 중 하나는 iOS Simulator 를 위해 linking 할 때, Xcode 12+ 빌드 시스템이 Apple silicon 에서 시뮬레이터를 지원하기 위한 아키텍처로 arm64 라고 정하면서 발생했다.. 그래서 시뮬레이터로 실행한다고 선택하면 arm64 기반의 시뮬레이터를 위한 라이브러리 / 앱을 컴파일하거나 link 할 수 있다. 따라서 Intel 기반의 Mac 은 다음의 에러를 받게 된다.

```
could not find module 'Framework' for target 'arm64-apple-ios-simulator'; found: x86_64-apple-ios-simulator, x86_64
```

이 문제는 "Build Active Architecture Only(`ONLY_ACTIVE_ARCH`)" 설정을 debug 모드에서 `YES` 로 설정해서 해결할 수 있다.

> 이 설정은 'Generic Device' run destination 과 같이 특정 아키텍처를 명시하지 않은 run destination 을 설정하고 빌드할 때 무시된다.

### Excluded Architectures `(EXCLUDED_ARCHS)`

이름에서 알 수 있듯이 컴파일 할 때 배제할 아키텍처들을 명시한다. 만약 Intel mac 에서 컴파일러에게 i386, arm64 시뮬레이터를 무시한다고 설정하고 싶으면 아래와 같이 설정해서 해결할 수 있다.

`EXCLUDED_ARCHS[sdk=iphonesimulator*] = arm64 i386`

* 참고
* [https://www.arm.com/architecture/cpu](https://www.arm.com/architecture/cpu)
* [https://www.geeksforgeeks.org/arm-processor-and-its-features/](https://www.geeksforgeeks.org/arm-processor-and-its-features/)
* [https://cs.stanford.edu/people/eroberts/courses/soco/projects/risc/risccisc/](https://cs.stanford.edu/people/eroberts/courses/soco/projects/risc/risccisc/)
* [https://www.alihilal.com/blog/a-deep-dive-into-ios-cpu-architectures-and-build-settings](https://www.alihilal.com/blog/a-deep-dive-into-ios-cpu-architectures-and-build-settings)
* [https://developer.apple.com/documentation/xcode/writing-arm64-code-for-apple-platforms](https://developer.apple.com/documentation/xcode/writing-arm64-code-for-apple-platforms)
* [https://docs.elementscompiler.com/Platforms/Cocoa/CpuArchitectures/](https://docs.elementscompiler.com/Platforms/Cocoa/CpuArchitectures/)
* [https://ktatsiu.wordpress.com/2016/11/15/about-arm-architectures-for-ios-development/](https://ktatsiu.wordpress.com/2016/11/15/about-arm-architectures-for-ios-development/)
* [https://developer.apple.com/kr/support/required-device-capabilities/#ipad-devices](https://developer.apple.com/kr/support/required-device-capabilities/#ipad-devices)
* [https://www.innerfence.com/howto/apple-ios-devices-dates-versions-instruction-sets](https://www.innerfence.com/howto/apple-ios-devices-dates-versions-instruction-sets)
