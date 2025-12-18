
# KMP vs CMP Proof of Concept (PoC) Plan for Native iOS (Tuist + SPM)

KMP, CMP 를 kakaot (Tuist + SPM) iOS 아키텍처에 통합하기 위한 PoC.

* 목표: KMP / KMP + CMP 를 도입했을 때의 tax

KMP / KMP+CMP 를 iOS 프로젝트에 import 하기 위해서는 궁극적으로 XCFramework 를 생성해야 함 

---

## 1. Binary & Build size 증가

* KMP only
* KMP + CMP : Skia graphics engine 을 xcframework 에 포함해서 배포하게 되므로 iOS 최종 app 파일 `.ipa` 의 크기가 증가함

### 1.1 두 종류의 빌드 준비

두 종류의 XCFramework 을 빌드

* KMP only : logic only
* KMP + CMP : logic + view + skia engine

### 1.2 Measurement Metrics

앱 최종 바이너리 크기 측정 => App Thinning report 생성 (*앱 배포를 appstore에 해야지만 얻을 수 있는지 확인*) 

=> **개선/팩트:** **App Store에 올리지 않아도 로컬에서 측정이 가능합니다.** Xcode에서 Archive를 생성한 후, Distribute App -> Development (또는 Ad Hoc) -> 'App Thinning' 옵션을 선택하여 내보내면 각 디바이스별 정확한 `.ipa` 사이즈와 `App Thinning Size Report.txt`를 얻을 수 있습니다.

> 최종 `.ipa` 크기는 App Store download 크기와 같지 않음 (App Thinning)

1. 기본 버전 : KMP 없이 일반 T 프로젝트 빌드
2. KMP only :  Add Target A. Measure the `__TEXT` segment of the Mach-O binary.
3. KMP + CMP :  Skia 엔진 포함 얼마나 늘었는지 확인

> T 앱은 여러 iOS target (App Extension, Widget) 등을 갖고 있기 때문에 XCFramework 를 dynamic framework 로 만들지 않으면 KMP+CMP 를 사용하는 곳마다 복제가 돼서 용량이 곱하기가 되기 때문에 dynamic framework 로 만들어줘야 함

---

## 2. 성능 & 렌더링

CMP 는 자체 skia 엔진을 사용해서 iOS UI component 를 pixel 로 그리기 때문에 사용자 입장에서 이게 iOS native 를 사용하는 것만큼 자연스러운지 확인해야 함

### 2.1 UI 반응성(The "Jank" Test)

Xcode instruments 의 Core Animation 을 사용해서 측정

* Frame Rate : CMP 내 list 를 스크롤링하는게 ProMotion display (iPhone 13 Pro+) 에서 120 FPS 를 유지하는가?
* GPU Utilization : Swift `List` vs CMP `LazyColumn` 을 비교해서 GPU load 차이를 측정
* 가장 이질감이 느껴지는 부분 : TextField (Text input) and Scroll Inertia (스크롤 관성)
	* textfield 에 빠르게 타이핑을 할 때 iOS Native 키보드 이벤트와 렌더링 딜레이가 없는지
	* 스크롤 후 손을 떼었을 때 감속되는 느낌이 iOS Native UIScrollView 와 동일한지

### 2.2 Memory (RAM)

Leaks, VM Tracker

* Resident Memory : CMP canvas (`UIViewController`) 가 활성화되어 있을 때 RAM usage (skia engine 이 memory 에 올라와 있을 것이기 때문)
* Memory Leak Detection : iOS `UINavigationController` 에서 CMP view 를 pop 할 때 Kotlin/Native memory 매니저가 리소스를 해제하는지 확인

---

## 3. Cold start vs Warm Start

CMP 가 skia engine 을 wake up 해줘야 함

1. Cold start : 처음 CMP view 가 뜰 때까지의 시간 측정
2. Warm start : CMP view 를 띄운 이후 다시 native view 로 돌아갔다가 다시 CMP view 를 띄움
3. Pre-warm(optional) : cold start 가 너무 느리다면 `AppDelegate` 등에서 Skia engine 을 initialize 할 수도 있기는 함

---

## 4. 의존성 관리

Multiplatform 모듈에서 kakaot 와 동일한 의존성 등을 가져갈 경우 어떻게 해야 하는가 -> KMP 쪽에서 의존성 역전을 통해 모듈을 주입하는 방식을 사용해야 함

KMP 에 같은 의존성을 추가하고 bundle 에 추가했다면 duplicate symbol 에러가 떨어질 것.

---

## 5. 개발 플로우

KMP 모듈을 어떻게 T 앱에 도입하면서 개발할 지 

### 5.1 디버깅

일반적으로 XCFamework 를 만들어서 코드를 바로 볼 수 없지만 Touchlab xcode-kotlin 을 사용하면 xcode 에서도 kotlin 코드를 보고 debug point 를 찍을 수 있다고 함

**비동기 처리:** Kotlin의 Coroutine/Flow를 Swift의 `async/await`나 RxSwift/Combine으로 변환해야 한다면, KMP 단에 [KMP-NativeCoroutines](https://github.com/rickclephas/KMP-NativeCoroutines) 라이브러리 도입을 PoC에 포함하는 것을 강력히 권장합니다.

### 5.2 개발 및 배포 가이드

자동화

* SPM wrapper 사용 -> 이 방법으로 가지 않을까? code base 는 어떻게 관리될 것인가...
* XCFramework 직접 생성

--

* 주요 확인 포인트
	* 앱 사이즈 
	* 의존성 관리 측면
		* iOS native 앱의 공통 모듈 (공통 팝업 뷰 등 UI 모듈, 카메라 모듈 등) 을 Kotlin 에서도 띄울 수 있는가?
		* iOS native 앱의 모듈을 Kotlin 모듈 쪽에 주입해주는 방식으로 작업이 되어야 할 것 같은데 어떻게 작업을 해야 하는가
		* Kotlin 모듈을 dynamic framework 형태로 iOS Native 앱에 심게 될 것 같은데, Map 같은 큰 모듈을 iOS native 프로젝트와 Kotlin 프로젝트 모두에서 의존성을 갖고 있으면 에러가 발생할 것으로 예상된다. 이는 어떻게 해결하는 것이 좋은가?

**UI의 뼈대와 비즈니스 로직은 Kotlin이 가져가되, 무거운 Native SDK와 디바이스 종속적인 기능은 Swift가 구현하고 Kotlin에게 주입해 준다"** 는 원칙


=> 그쪽에서 어떤 방식으로 개발할지... 네트워크 이런 모듈 같은 것도 다 따로 개발해서 가져다 쓰는 건지? 그러면 대체 얼마나 독립적인 건지
=> KMP 프로젝트 링크 / 권한
=> 트럭커는 현재 풀 KMP 이면 iOS 빈 깡통 프로젝트에 kotlin framework 를 심어서 운용하는 건가?
