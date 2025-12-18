https://docs.unity3d.com/Packages/com.unity.cinemachine@3.1/manual/

Cinemachine 은 Unity Camera 를 control 하기 위한 module 의 모음. Cinemachine 은 복잡한 수학 계산, target tracking, composing, blending, shot 간 cutting 을 처리함. 개발 시 많은 시간을 들여 카메라를 직접 조작하거나 스크립트를 짜는 것을 방지하기 위함.

수정사항을 만들 경우 e.g. 애니메이션 변경, 차량 속도 변경 등 Cinemachine 은 동적으로 여기에 적응하며 최적의 shot 을 만들어 냄. Camera script 를 다시 작성할 필요가 없음

Cinemachine 은 FPS, 3인칭, 2D, top down, RTS 등 많은 장르에서 실시간으로 동작함. Scene 에서 필요한 많은 shot 을 제공함.

# Cinemachine Core Concepts

## Essential elements

3 개의 main type 으로 구성됨

* Single **Unity Camera** : Scene 의 이미지를 capture
* **Cinemachine Brain** : Unity Camera 안에서 Cinemachine 기능을 활성화
* 하나 이상의 **Cinemachine Cameras** : Unity Camear 를 돌아가며 control 함

![](https://docs.unity3d.com/Packages/com.unity.cinemachine@3.1/manual/images/concept-base.png)

### Unity Camera

Camera 를 포함하는 GameObject <-> Cinemachine Camera : Unity Camera 를 control 하기 위한 요소들을 포함함

Cinemachine setup 은 무조건 오직 하나의 unity camera 만 가지고 있어야 함.

### Cinemachine Brain

* Scene 내 모든 Cinemachine Camera 를 모니터링
* 어떤 Cinemachine Camera 가 Unity Camera 를 control 할 지 결정
* Unity Camera 를 다른 Cinemachine Camera 가 전달받는 것을 처리

### Cinemachine Camera

Camera placeholder 의 역할을 하는 GameObject 로, 상태에 따라 Unity Camera 를 조정함 (Virtual Cameras)

Cinemachine Camera 가 Unity Camera 를 control 하면 동적으로 property 와 동작을 overriding 함.

* 영향 범위
	* Unity Camera 의 위치
	* Unity Camera 가 aiming 하는 것
	* Unity Camera 의 lens setting
	* Unity Camera 의 post-processing profile
	* Unity Camera 가 시간에 따라 어떻게 동작하는지

#### GameObjects

Cinemachine Camera GameObjects != Unity Camera GameObject

* 독립적으로 동작, 중첩되면 안됨
* Camera Component 를 포함하면 안됨
* Cinemachine Camera component 를 포함해야 함
* 추가적인 Cinemachine 요소를 포함해서 procedural motion 을 관리하고 기능을 확장할 수 있음

#### Single / multiple Cinemachine Cameras

Cinemachine Cameras 를 많이 생성할 수 있지만 필요에 따라 하나의 Cinemachine Camera 로 충분히 커버할 수 있음

* 하나의 캐릭터를 Unity Camera 가 따라가기를 원하면 하나의 Cinemachine Camera 를 사용함
* 여러 곳에서 여러 shot 이 필요할 경우 각 shot 마다 하나의 Cinemachine Camera 를 생성해야 함.

#### Processing power consumption

Cinemachine 은 여러 개의 Cinemachine Camera 를 생성하도록 장려함. Processing power 를 조금만 사용함. 

## Camera control and transitions

### Cinemachine Camera states

각 Cinemachine Camera 는 세 state 중 하나에 속함. 오직 Live 인 것만이 blend 가 발생하지 않는 경우에 Unity Camera 를 조정함. (blend : 두 카메라 간에 smooth transition 을 의미함)

* Live
	* Cinemachine Camera 가 Cinemachine Brain 을 가진 Unity Camera 를 control 함.
	* Blend 가 발생할 때 (한 카메라에서 다른 카메라로 transition 발생) 두 개의 Cinemachine Camera 가 live임. Blend 가 끝나면 오직 하나의 Cinemachine Camera 만 live
* Standby
	* Cinemachine Camera 가 Unity camera 를 조정하지 않음. Target 을 여전히 따라가고, aiming 함
	* 이 상태의 Cinemachine Camera 는 GameObject 가 활성화되어 있고, live Cinemachine Camera 보다 낮은 우선순위의 priority 를 가짐
* Disabled
	* Unity Camera 를 조정하거나 target 을 따라가거나 target 을 aiming 하지 않음
	* Processing power 를 소모하지 않음
	* Cinemachine Camera 를 비활성화하려면 GameObject 를 비활성화하면 됨
	* GameObject 가 비활성화돼도 Cinemachine Camera 가 blend 를 준비하고 있다면 Cinemachine Camera 는 여전히 Unity camera 를 조정할 수 있음

### Live Cinemachine Camera selection

Cinemachine Camera 를 live 를 만드는 조건은 Cinemachine 을 사용하는 context 에 따라 달라짐. 기본적으로 Cinemachine Brain 은 live Cinemachine Camera 를 선택하는 것을 처리하는 책임이 있음

* Brain 은 가장 높은 우선순위의 Cinemachine Camera 를 선택해서 Live 로 만듦
* 여러 CinemachineCamera 가 활성화되어 있고 같은 우선순위를 가지면 가장 최근에 활성화된 게 선택됨
* 비활성화되거나 낮은 우선순위의 CinemachineCamera 는 blend 의 일부일 경우 Live 가 될 수 있음

#### Realtime dynamic events

Cinemachine Camera 의 priority 를 조정하거나 GameObject 를 활성화/비활성화 시켜서 동적으로 게임 내 발생하는 이벤트에 대응할 수 있음.

#### Timeline

Shot 을 관리하기 위해 'Cinemachine with Timeline' 을 사용해서 Cinemachine Camera 를 조정할 수 있음

Cinemachine Camera 를 Timeline 과 함께 사용하면 timeline 은 Cinemachine Brain 우선순위 시스템을overriding 해서 Cinemachine Camera Clip 이 활성화된 경우 Cinemachine Camera 의 우선순위와 활성화된 상태가 무시됨.

### Cinemachine Camera transitions

Cinemachine Camera 간의 transition 을 관리해서 새로운 카메라가 live 가 되도록 할 수 있음.

* 기본적으로 Cinemachine Brain component 의 transition rule 을 따름
* Shot sequencing 을 사용할 때 Timeline Cinemachine track 에서 직접 transition 을 조정할 수 있음

#### Blends

간단한 shot 들과 그 사이의 blending 을 통해 실시간 게임 이벤트에 대응할 수 있게 해줌.

Cinemachine blend 는 fade/wipe/dissolve 가 아님. Cinemachine 은 한 Cinemachine camera 에서 다른 camera 로 위치, 각도 조정을 스무스하게 조정해서 애니메이션을 수행함.

![](https://docs.unity3d.com/Packages/com.unity.cinemachine@3.1/manual/images/concept-transition-blend.png)

#### Cuts

Shot 이 다른 shot 으로 즉각 transition 되는 것

![](https://docs.unity3d.com/Packages/com.unity.cinemachine@3.1/manual/images/concept-transition-cut.png)

## Procedural motion

Cinemachine Camera 는 camera 의 placeholder 로 동작하는 수동적인 GameObject

* 특정 장소에 aiming 을 고정해서 배치 가능
* 다른 GameObject 에 두고 같이 움직이고 회전할 수 있음
* Custom script 를 사용해서 움직이거나 회전시킬 수 있음

더 세밀한 결과를 위해 procedural 동작과 extension 을 Cinemachine Camera 에 추가해서 동적인 이동, 흔들림, target trackinging, shot 구성, 사용자 input 에 대응 등의 작업을 할 수 있음

### Procedural behaviors and extensions

Procdeural behavior : 미리 만들어진 결과를 쓰는 게 아니라, 코드/알고리즘으로 "실시간 생성하거나 제어하는 동작".

카메라가 플레이어를 자동으로 따라감. 충돌 피하고, 부드럽게 회전하고, framing 맞추는데 전부 수학 + 알고리즘으로 계산

Cinemachine Camera component 는 위치, 각도, 렌즈를 조정하기 위한 다양한 옵션을 가지고 있음

#### Position and Rotation Control

Position Control, Rotation Control 을 사용해서 움직임, aiming 을 조정할 수 있음

대부분의 동작은 target GameObject 를 따라가거나 aiming 하게 되어 있음

#### Noise

Noise behabior 를 사용해서 흔들림을 구현하고 효과를 위해 실세계 물리 camera quality 를 시뮬레이션 할 수 있음

각 frame update 때마다 Cinemachine 은 target 을 따라가는 움직임에 별개로 noise 를 추가함. Noise 는 카메라의 위치에 영향을 주지 않음.

#### Extensions

Extension 을 추가해서 특정 usecase 에서 사용

### Target GameObject tracking

Procedural motion 의 핵심. Offset, screen 구성은 target 이 돌아다니는 것에 따라 상대적으로 적용되기 때문에 카메라는 shot 을 유지하기 위해 스스로 적응함

#### Tracking Target and Look At Target Properties

기본적으로 Cinemachine Camera 는 하나의 Tracking Target property 가 있음

* 같이 움직일 Transform 이 필요한 경우 명시함 (position control behavior 를 설정한 경우)
* LookAt target (rotation control behavior 를 설정한 경우 필요한 Transform) 을 명시함

만약 두 개의 다른 Transform 을 사용해야 하면 Use Separate LookAt Target 옵션 선택

#### Target tracking and blends

Cinemachine 이 두 shot 간에 blend 를 할 때 target 또한 상대적임. Cinemachine 은 target 에 대한 화면 위치를 유지하기 위해 노력하지만 target 이 바뀔경우 두 target 위치 간의 interpolation (부드럽게 보간) 함

Camera blend 시 target 이 명시되지 않았다면 Cinemachine 은 위치와 각도만 독립적으로 interpolate 해서 원치 않는 방식으로 원하는 객체가 화면 상에서 움직이게 될 수 있음. (걍 target 을 지정하는 것이 좋다는 말을 하고 싶은듯)

### Behavior and extension selection

Cinemachine Camera component 에 behavior 를 선택하거나 extension 을 추가할 경우 Unity 는 자동으로 Cinemachine Camera GameObject 에 component 를 추가함

#### Custom behaviors and extensions

`CinemachineComponentBase` / `CinemachineExtension` 을 상속해서 custom script 를 작성할 수 있고, custom 으로 이동하는 동작이나 extension 을 작성할 수 있음

## Cinemachine and Timeline

Timeline : Cinematic content, gameplay sequence, audio sequence 등을 생성하기 위해 사용함

Timeline 을 사용해서 Cinemachine Camera 를 활성화/비활성화하고 blending 해서 이동 동작이 구성된 camera 로 shot sequence 를 생성할 수 있음

간단한 shot sequence 의 경우 Timeline 대신 Cinemachine Sequencer Camera 를 사용할 수도 있음

### Live Cinemachine Camera selection

Timeline 이 Cinemachine 을 조작할 때 Cinemachine Brain 이 만든 우선순위 기반의 동작을 overriding 함. Timeline 이 끝나거나 Timeline 이 Cinemachine 을 조작하고 있지 않은 시점이라면 control 은 Cinemachine Brain 으로 돌아감

### Cinemachine Track and Shot Clips

Timeline 은 Cinemachine Track 내 Cinemachine Shot 에서 Cinemachine Camera 를 조작함. 각 shot clip 은 활성화한 다음 비활성화할 CinemachineCamera 를 가리킴. Shot clip sequence 를 사용해서 각 shot 의 순서와 길이를 조정할 수 있음

### Cinemachine Camera transitions

 두 CinemachineCamera 간에 cut 을 하려면 clip 을 바로 다음에 배치하면 되고, blend 를 하려면 clip 을 겹쳐놓으면 됨

![](https://docs.unity3d.com/Packages/com.unity.cinemachine@3.1/manual/images/CinemachineTimelineShotClips.png)

### Multiple Cinemachine Tracks

같은 timeline 내에 여러 개의 Cinemachine Track 을 가질 수 있음. 더 아래에 있는 track 이 위에 있는 것을 override 함

여러 개의 Timeline 을 가질 수도 있는데 가장 최근데 활성화된 게 다른 것을 override 함

### Blend with other Cinemachine Tracks and the Brain

한 Cinemachine Shot Clip 을 별개의 Cinemachine Track 에 있는 것에 blend 할 수 있음.

이를 위해 Inspector 에서 Ease In Duration, Ease Out Duration 시간 값을 조정하면 됨

# Set up a basic Cinemachine environment

* 순서
	* Cinemachine Camera 생성
	* Cinemachine Brain 이 Unity Camera 에 있는지 검증
	* Cinemachine Camera 를 조정해서 어떻게 Unity Camera 에 영향을 끼치는지 확인

## Add a Cinemachine Camera

* Scene view 에서 Cinemachine Camera 로 보고 싶은 곳으로 화면을 이동함
* GameObject > Cinemachine > Cinemachine Camera 

Unity 는 아래 요소를 가진 GameObject 를 새로 생성.

* Cinemachine Camera component
* Scene view camera 의 최신 위치, 방향의 transform

## Verify the Cinemachine Brain presence

Scene 에 첫 Cinemachine Camera 를 생성하면 Unity 는 자동으로 Unity Camera 에 Cinemachine Brain 을 추가함

Cinemachine Brain 가 존재하면 Unity Camera 의 transform, lens setting 은 lock 되고 Camera inspector 에서 바로 수정할 수 없음. 이는 대응되는 CinemachineCamera 의 property 를 변경해서 바꿀 수 있음

## Adjust the Cinemachine Camera properties

1. Game View 열면 현재 Cinemachine Camera 로 설정된 Unity Camera 로 보여지는 Scene 을 볼 수 있음
2. Hierarchy > Cinemachine Camera GameObject 선택
3. Inspector 에서 property 조정
	1. Transform > Position / Rotation
	2. Cinemachine Camera component > Lens

# Set up multiple Cinemachine Cameras and transitions

* 다른 property 를 가진 여러 개의 Cinemachine Camera 를 생성
* Cinemachine Brain 에서 Cinemachine transition 조작
* Live camera 활성화를 테스트, Play mode 에서 transition 조작

## Add Cinemachine Cameras

1. Scene view 에서 Cinemachine Camera 로 frame 하고 싶은 곳으로 이동
2. 여러 다른 위치, 각도를 가진 Cinemachine Camera 생성

## Manage transitions between Cinemachine Cameras

1. Hierarchy > Unity Camera 선택
2. Cinemachine Brain component : Default Blend 선택 / Custom Blends 를 정의한 asset 을 target

## Test the transitions in Play mode

1. Play mode
2. 각 Cinemachine Camera GameObject 들을 활성화 / 비활성화시키면서 blend 확인

# Add procedural behavior to a Cinemachine Camera

Procedural behavior 설정

* Camera 위치, 각도 조절하는 behavior 추가
* 자동 follow, aim 을 tracking 하기 위한 GameObejct 명시
* 실세계 카메라 흔들림을 시뮬레이션하기 위한 default noise 추가

최소 하나의 passive cinemachine camera 가 필요하며 Cinemachine Camera 로 follow 할 수 있는 target GameObject 필요함

## Add Position Control and Rotation Control behaviors

1. Hierarchy 에서 Cinemachine Camera GameObject 선택
2. Inspector > Cinemachine Camera component > Position Control 을 Follow 로 설정
3. Inspector > Cinemachine Camera component > Rotation Control 을 Rotation Composer 로 설정

Editor 에서 Follow Camera 를 하는 것과 동일한 결과.

Cinemachine Camera GameObject 는 오직 하나의 Position Control behavior, Rotation Control behavior 를 가질 수 있음.

## Specify a GameObject to track

1. Inspector > Cinemachine Camera component > Tracking Target 을 하나의 GameObject 에 설정

Cinemachine Camera 는 자동으로 Cinemachine Camera 를 follow behavior 에 따라 GameObject 에 상대적인 위치에 위치시키고, Rotation Composer behavior 에 따라 GameObject 를 보도록 회전함

## Add noise for camera shaking

1. Cinemachine Camera component > Noise property 를 Basic Mutli Channel Perlin 으로 설정
2. Noise Profile 오른쪽의 configuration 버튼 클릭
3. Preset > Noise Setting asset 선택

## Add a Cinemachine Camera Extension

Inspector > Cinemachine Camera > Add Extension

# Set up Timeline with Cinemachine Cameras

* 여러 Cinemachine Camera 를 사용해서 여러 shot 을 지원
* Timeline 을 준비하고, Cinemachine Track 을 생성하고 Cinemachine Shot Clip 을 추가
* Camera cut, blend 관리

## Prepare the Cinemachine Cameras

1. Hierarchy > static / procedural Cinemachine Camera 를 여러 개 생성

## Prepare the Timeline

1. 빈 GameObject 생성, My Timeline 으로 이름 붙임
2. Timeline window 열고 새로운 Timeline Asset 생성

## Create a Cinemachine Track with Cinemachine Shot Clips

1. Hierarchy 에서 Unity Camera GameObject (Cinemachine Brain 포함) 을 Timeline Editor 에 drag 한 후 Create Cinemachine Track

Unity 는 Unity Camera 를 targeting 하는 Cinemachine Track 을 추가함.

![](https://docs.unity3d.com/Packages/com.unity.cinemachine@3.1/manual/images/setup-timeline-cinemachine-track.png)

2. Hierarchy 에서 첫 번째 Cinemachine Camera GameObject 를 Cinemachine Track 에 추가
3. 여러 Cinemachine Track 을 추가해서 순서/기간을 조정

## Create camera cuts

두 개의 Cinemachine Shot Clip 을 겹치지 않게 설정하면 됨

![](https://docs.unity3d.com/Packages/com.unity.cinemachine@3.1/manual/images/setup-timeline-camera-cut.png)

## Create camera blends

두 개의 Cinemachine Shot clip 을 겹치도록 하면 됨

![](https://docs.unity3d.com/Packages/com.unity.cinemachine@3.1/manual/images/setup-timeline-camera-blend.png)


# Use convenient tools and shortcuts

## Pre-build cameras

Cinemachine package 는 특정 use case 를 위한 여러 빌드된 카메라가 있음

* State-Driven Camera
	* 여러 Cinemachine Camera 의 group 의 부모가 되는 Manager Camera 생성, 하위 camera 들을 animation state 변경에 따라 관리
* Follow Camera
	* 캐릭터를 follow, framing 하는 카메라
* Target Group Camera
	* group 을 follow, framing 하는 카메라. 빈 Target Group 을 만들어서 Tracking Target field 에 지정
* FreeLook Camera
	* User Input 이 있는 character-centric free look 시나리오에 적합
* Third Person Aim Camera
	* 삼인칭 aiming 시나리오에 적합. Target 의 회전, 위치에 따라 바뀜.


https://docs.unity3d.com/Packages/com.unity.cinemachine@3.1/manual/samples-tutorials.html

