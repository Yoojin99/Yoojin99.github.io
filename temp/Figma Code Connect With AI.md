

# Figma Code Connect

https://help.figma.com/hc/en-us/articles/23920389749655-Code-Connect
https://developers.figma.com/docs/code-connect/

* 전제조건 : Organization / Enterprise Plan + Full / Dev 권한

codebase 와 Figma 사이의 bridge 역할

Design file 내 component - Repository 내 component 연결. 구현되면 AI agent 에게 실제 코드의 reference를 줌으로써 더 Figma MCP server 의 능력을 향상시킬 수 있음

## Code Connect 를 쓰는 두 가지 방법

Code repository 를 Figma 에 바로 연결하는 방법

* Code Connect UI
	* Figma 내부. 설정 쉽고, 디자인 컴포넌트를 코드로 연결할 간단하고 시각적인 방법을 원할 겨우. Github repo 에 바로 연결할 수 있음
* Code Connect CLI
	* repo 내부에서 local 하게 동작하며 개발자들이 더 유연하게 사용할 수 있음. Property mapping, dynamic code example 을 지원해서 통합과 정확도를 고려함

### Code Connect 가 MCP 와 동작하는 방법

Code Connect UI, CLI 는 Figma MCP Server 에 feed 돼서 연결된 component 들이 AI 활성화된 tool 과 code editor 에서 접근할 수 있게 함. 

## Code Connect UI

UI - Dev 모드 활성화 후 Insepct 창에서 Code Connect 확인

* Connection types
	* Github repository 연결 (optional)
		* Figma 가 Github repo 에 연결할 수 있게 권한 허용하고 component 를 직접 선택할 수 있음 (Github 에 연결하려면 file 의 owner 여야 가능)
	* 직접 mapping
		* 관련 파일 / URL + component 이름으로 component 연결
		* mapping 되면 Figma MCP server 에 공유됨. 해당 component 를 사용하면 관련 코드가 design context 에 포함돼서 AI agent 에 전달됨

## Enhanced MCP codegen with previews

MCP codegen : 연결된 source file 을 기반으로 예시 코드 preview 를 만들어주는 것

Code Connect UI 에 MCP codegen 이 포함되어 있어서 component 가 codebase 에 어떻게 나타날 지 AI 생성된 snippet 을 볼 수 있음

## Code Connect CLI

https://developers.figma.com/docs/code-connect/quickstart-guide/

Component 정의된 것을 연결하는 것 외에도 property 를 연결할 수 있음. 이미 존재하는 디자인 시스템이 있고 디자인, 개발을 하면서 디자인 시스템에 지속적이고 정확한 반영을 하고 싶은 경우 유용

* 요구사항
	* component 를 포함한 디자인 시스템 codebase, Figma design library
	* Node.js 18 +
	* Personal access token (Code Connect scope : Write, File content scope : Read)

iOS 의 경우 SwiftUI 가 지원되는 Code Connect CLI Platform Version : v1.3.13

### Dependecy 설정

```swift
let package = Package(
    name: "ExampleProject",
    platforms: [...],
    products: [...],
    dependencies: [
        .package(url: "https://github.com/figma/code-connect", from: "1.0.0"),
    ],
    targets: [
      .target(
         name: "ExampleTarget",
         dependencies: [
               .product(name: "Figma", package: "code-connect")
         ]
      )
    ]
)
```

### Code Connect CLI 설치

```
npm install --global @figma/code-connect@latest
```

* privacy : `figma connect`를 사용하면 Figma 는 아래 데이터를 얻게 됨
	* component 가 추가된 path
	* component 가 구현된 repository URL
	* .figma 파일 내 property 와 code

### Interactive setup

Code Connect 파일들을 빠르게 생성하기 위해 Code Connect CLI 는 interactive setup 을 포함함.


### Connect

1. Code Connect CLI 설치
2. Interactive setup 
3. 첫 번째 compone

---

# Figma MCP


---
  
0. 사전 연동

* Figma MCP 연동
* remote / local 어떤걸로 써야하는지 -> Remote
* Remote : codex 로 remote mcp 연결
* local : Figma Desktop dev mode (enable desktop mcp server?)
* Figma MCP Guide : https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server

* Remote MCP server : `https://mcp.figma.com/mcp` 엔드포인트에 바로 연결됨

* Codex 에 mcp 연결

* `codex mcp add figma --url https://mcp.figma.com/mcp`

1. 인증 / 토큰 처리?

* codex 에서 mcp 사용할 때

* `codex mcp login figma`

* 연결됐는지 확인 : `codex mcp list`

2. Skill 의 입력을 안정화 : Figma -> Design Spec (중간 단계물)

* MCP 로 Figma 에서 정보들 가져와서 중간 스펙 (JSON/MD) 로 정리

* 스펙 바탕으로 SU 코드 생성

3. Skill 작성

4. Codebase context

* 우리 디자인 시스템

* 공용 검포넌트

* 폴더 구조

* 네이밍

* SU 작성 규칙?

5. 플로우 구축

* input : 대상 figma 링크 + ...

* codex 가 mcp 통해 design context 가져옴

* 중간 스펙 생성

* SU 코드 생성

* 빌드 + 테스트

* diff 리뷰 / pr

  
  
  

* Figma MCP 연동

* 연동은 어떻게? remote / local / ...?

* claude code skill -> codex 에서 사용하기로

*