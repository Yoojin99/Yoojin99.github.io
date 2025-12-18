https://developers.figma.com/docs/figma-mcp-server/

MCP 서버는 중요한 디자인 정보와 context 를 AI agent 에게 전달해서 agent 가 피그마 디자인 파일에서 코드를 생성할 수 있게 해줌.

MCP 서버는 Model Context Protocol 을 사용해서 AI agent 가 data 소스와 소통할 수 있게 해주는 표준화된 인터페이스.

MCP 서버로 아래 작업들을 할 수 있음

* 선택된 프레임에서 코드를 생성 : Figma frame 을 선택하고 코드로 변환.
* 디자인 context 추출 : 변수, component, layout data 를 바로 IDDE 에 넣을 수 있음. 
* Make resource 받기 : Make file 에서 code 리소스를 받을 수 있고 context 로 LLM 에 제공 가능. 이는 프로토타입을 production app 으로 전환하는데 도움을 줌
* Code Connect 로 디자인 시스템 component 를 consistent 하게 유지함 : 실제 component 를 재사용해서 output quality 향상

# Core server features

아래의 tool 제공

* [`generate_figma_design`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#generate_figma_design) (Claude Code, remote only)  : 인터페이스에서 디자인 레이어 생성
* [`get_design_context`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#get_design_context) : Layer / 선택한 것의 design context 불러오기
* [`get_variable_defs`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#get_variable_defs) : Figma 선택한 것에서 variable, style return
* [`get_code_connect_map`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#get_code_connect_map) : Figma node ID 와 대응되는 codebase 에서 대응되는 code component mapping 불러옴
* [`add_code_connect_map`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#add_code_connect_map) : Figma node ID 와 대응되는 codebase 내 code component 간의 mapping 을 추가
* [`get_screenshot`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#get_screenshot) : agent 에게 선택한 것의 스크린샷을 찍도록 함
* [`create_design_system_rules`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#create_design_system_rules) : 디자인을 FE code 로 변환하기 위해 올바른 context 를 제공하기 위해 agent 에 제공되는 rule file 생성
* [`get_metadata`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#get_metadata) : 선택한 것의 기본 정보 (layer ID, name, types, poisition, size) 를 포함한 XML 표현을 리턴
* [`get_figjam`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#get_figjam) : FigJam 다이어그램을 XML 로 변환
* [`generate_diagram`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#generate_diagram) : Mermaid syntax 에서 FigJam 다이어그램 생성
* [`whoami`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#whoami-remote-only) (remote only) : Figma 에 로그인된 user 정보 반환
* [`get_code_connect_suggestions`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#get_code_connect_suggestions) : Code Connect 를 사용해서 Figma node ID 와 대응되는 code component 를 mapping 하는 것에 대한 추천


* 디자인 정보 불러오는 tool
	* [`get_design_context`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#get_design_context)
		* https://developers.figma.com/docs/figma-mcp-server/server-returning-web-code/#about-the-get_design_context-tool : MCP server 가 Figma 디자인을 LLM 이 해석할 수 있는 형식의 포맷으로 인코딩함 
	* [`get_variable_defs`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#get_variable_defs)
	* [`get_metadata`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#get_metadata)
* Project-scope 관련 설정 integration
	* [`create_design_system_rules`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#create_design_system_rules)
* Code connect 관련
	* [`get_code_connect_map`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#get_code_connect_map)
	* [`add_code_connect_map`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#add_code_connect_map)
	* [`get_code_connect_suggestions`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#get_code_connect_suggestions)
	* [`send_code_connect_mappings`](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/#send_code_connect_mappings)


## 테스트 결과

테스트 한 URL : https://www.figma.com/design/08DzfEl17oTQxv6ZRFegCc/%EC%8B%A0%EB%B6%84%EC%A6%9D?node-id=2714-26162&m=dev

(신분증 등록 url)

### get_design_context

https://developers.figma.com/docs/figma-mcp-server/server-returning-web-code/#about-the-get_design_context-tool

이 tool 을 사용하면 MCP server 는 Figma 디자인을 시각적인 형태(?)는 유지하면서 LLM 이 잘 해석할 수 있는 구조화된 format 으로 인코딩함

이 format 은 react 같은 코드 형태를 띄는데 일반적으로 AI agent 가 대용량의 웹 기반 데이터를 기반으로 훈련되었기 때문. 이런 AI 한테 친숙한 구조를 사용하면 AI agent 는 프로젝트에 맞는 언어, 프레임워크, 패턴에 맞는 코드를 return 할 수 있음

Code Connect 가 설정되어 있다면 이 tool 은 사용자가 제공한 codebase 에 대한 정보를 포함해서 tool 을 실행할 것임. 

Raw output :

```
const img202111 = "https://www.figma.com/api/mcp/asset/7283b283-a634-4bc5-870b-888ce1d2b704";
    const img202203 = "https://www.figma.com/api/mcp/asset/523d5412-52a0-436e-a5ba-009fd917a8b5";
    const imgBg = "https://www.figma.com/api/mcp/asset/91742db7-7f43-42d2-b5d5-4e7d523f6dcb";
    const imgSubtract = "https://www.figma.com/api/mcp/asset/ca787328-a400-4a44-9384-9a7045d97e72";
    const imgLogo = "https://www.figma.com/api/mcp/asset/f24dfcad-ae8a-428f-a2f1-8842b3e15c2d";
    const imgImg180Driverlicense = "https://www.figma.com/api/mcp/asset/693851b7-09b4-49e1-9083-f962e20c11a2";
    const imgVector = "https://www.figma.com/api/mcp/asset/bdd1de67-d7fe-4c50-9e0b-0502d8c1c9ed";
    const imgFrame28 = "https://www.figma.com/api/mcp/asset/0f8a961b-6b02-453c-aed9-07c325a93a53";
    const imgShape = "https://www.figma.com/api/mcp/asset/0c6e1284-875f-49a6-b399-661526ebc43a";
    const imgShape1 = "https://www.figma.com/api/mcp/asset/89210346-9a9e-43d4-95e9-dd04e5c24f6b";
    const imgPath = "https://www.figma.com/api/mcp/asset/117dfe1e-c12b-4225-8927-d3798b7ed9c7";
    const img202112 = "https://www.figma.com/api/mcp/asset/1b0c20a1-7f7d-4bba-b099-455f5ab26f78";
    const imgMenu = "https://www.figma.com/api/mcp/asset/8f10cb1d-6c3d-4104-a4e1-17181c54d4dc";
    type NoticeProps = {
      className?: string;
      property?: "default" | "info";
    };

    function Notice({ className, property = "default" }: NoticeProps) {
      const isDe...
    SUPER CRITICAL: The generated React+Tailwind code MUST be converted to match the target project's technology stack and styling system.
    1. Analyze the target codebase to identify: technology stack, styling approach, component patterns, and design tokens
    2. Convert React syntax to the target framework/library
    3. Transform all Tailwind classes to the target styling system while preserving exact visual design
    4. Follow the project's existing patterns and conventions
    DO NOT install any Tailwind as a dependency unless the user instructs you to do so.

    Node ids have been added to the code as data attributes, e.g. `data-node-id="1:2"`.
    These styles are contained in the design: Neutral1 (on Background, on Surface): #262D39, Title1_24B: Font(family: "Roboto", style: Bold, size: 24, weight: 700, lineHeight: 30, letterSpacing: 0), Primary1_2 (Primary Variants): #DFEBFF, BG (Background2, Surface1): #F5F6F7,#080808,
        Primary2 (Background1, Surface2): #FFFFFF, Primary1 (Primary): #066AEB, body1_16: Font(family: "Roboto", style: Regular, size: 16, weight: 400, lineHeight: 22, letterSpacing: 0), Shadow_Card : 택시_배차실패 리스트: Effect(type: DROP_SHADOW, color: #0047E808, offset: (0, 4),
        radius: 16, spread: 0), body2_14B: Font(family: "Roboto", style: Bold, size: 14, weight: 700, lineHeight: 20, letterSpacing: 0), Primary6_1 (Primary Variants): #3581FF, body2_14: Font(family: "Roboto", style: Regular, size: 14, weight: 400, lineHeight: 20, letterSpacing: 0),
        Neutral2 (on Background, on Surface): #262D39, Gray N6: #EEEEEF, Neutral4 (on Background, on Surface): #262D39, Primary2: #FFFFFF.
    Component descriptions: The following components have usage descriptions or documentation links defined in Figma. These descriptions provide important context about the intended usage, best practices, and any constraints for each component. Follow these guidelines when
        implementing or using these components.

    ## TitleBar / Web_title
    **Node ID:** 1267:12194

    우측메뉴\_폰트사이즈14로 변경 / 개발적용 전

    **Documentation:**
    - [https://www.figma.com/file/d0Zi2XllqTIHffhMCvNKW4/TitleBar_%EC%83%81%EC%84%B8?node-id=0%3A1](https://www.figma.com/file/d0Zi2XllqTIHffhMCvNKW4/TitleBar_%EC%83%81%EC%84%B8?node-id=0%3A1)
    Images and SVGs will be stored as constants, e.g. const image = 'https://www.figma.com/api/mcp/asset/550e8400-e29b-41d4-a716-446655440000'. These constants will be used in the code as the source for the image, ex: <img src={image} />. Image assets are stored on a remote server
        for 7 days and can be fetched using the provided URLs until they expire.
    <image content>
```

  Inputs
  fileKey=08DzfEl17oTQxv6ZRFegCc, nodeId=2714:26162

  Primary Data Returned
  No design payload yet. Tool returned a gate message that Code Connect mappings are missing and asked for a user yes/no decision first.

  What This Tool Is Best For
  Implementation-ready UI/code context from a node.

  Blind Spots / Missing Data
  Blocked until you answer the mapping prompt.

  Next Best Tool To Pair
  get_metadata (for structure) + get_screenshot (for visual).

* get_variable_defs

### get_metadata

Raw output 

```
{"fileKey":"08DzfEl17oTQxv6ZRFegCc","nodeId":"2714:26162","clientFrameworks":"unknown","clientLanguages":"unknown"})
  └ <frame id="2714:26162" name="운전면허증 인증/Empty" x="2409" y="256" width="360" height="1050">
      <rounded-rectangle id="2714:26163" name="bg gradient" x="0" y="72" width="360" height="930" />
      <text id="2714:26164" name="운전면허증을 어떻게 등록할까요?" x="20" y="100" width="195" height="60" />
      <frame id="2714:26165" name="img_180_driverlicense" x="20" y="192" width="320" height="188">
        <rounded-rectangle id="2714:26166" name="img_180_driverlicense" x="0" y="1" width="320" height="187.5" />
      </frame>
      <frame id="2825:24479" name="list" x="20" y="420" width="320" height="164">
        <instance id="2825:24480" name="license_card" x="0" y="0" width="320" height="76" />
        <instance id="2825:24481" name="license_card" x="0" y="88" width="320" height="76" />
      </frame>
      <instance id="2825:24482" name="notice" x="20" y="656" width="320" height="298" />
      <instance id="2714:26195" name="system / Bottom_Key_and" x="0" y="1002" width="360" height="48" />
      <instance id="2714:26196" name="system / Status Bar_and_onW" x="0" y="0" width="360" height="24" />
      <instance id="2714:26197" name="TitleBar / Web_title" x="0" y="24" width="360" height="56" />
    </frame>
    IMPORTANT: After you call this tool, you MUST call get_design_context if trying to implement the design, since this tool only returns metadata. If you do not call get_design_context, the agent will not be able to implement the design.
```

 Inputs
  fileKey=08DzfEl17oTQxv6ZRFegCc, nodeId=2714:26162

  Primary Data Returned
  XML-like node tree for frame 운전면허증 인증/Empty with child frames/instances (e.g. license_card, notice, status bar, title bar) and positions/sizes.

  What This Tool Is Best For
  Fast structure overview, node map, and hierarchy inspection.

  Blind Spots / Missing Data
  No rich style/code output; not implementation context.

  Next Best Tool To Pair
  get_design_context

### get_variable_defs

Tool 분석 결과

1. get_design_context for main implementation source
2. get_variable_defs for color/type tokens
3. get_screenshot for visual parity checks
4. get_metadata for precise node navigation/debugging


-> 사용했을 때 `get_design_context` 로 하는게 더 정확도가 높았음. 

```
 const img202111 = "https://www.figma.com/api/mcp/asset/67dc86fc-b6d0-4deb-bfd4-a9551f762aab";
    const img202203 = "https://www.figma.com/api/mcp/asset/eda496e1-c304-4b1b-b015-332d595fa6fe";
    const imgBg = "https://www.figma.com/api/mcp/asset/e1e6d17a-55b1-48e8-848d-f8d5828bc4de";
    const imgSubtract = "https://www.figma.com/api/mcp/asset/afa64c5b-ad43-4a89-8513-43a4dbf86f95";
    const imgLogo = "https://www.figma.com/api/mcp/asset/0f970770-7bf6-431f-a0ae-cb223262b886";
    const imgImg180Driverlicense = "https://www.figma.com/api/mcp/asset/806ece62-814e-407a-ad46-fff4b45e6a00";
    const imgVector = "https://www.figma.com/api/mcp/asset/129faa98-06f4-4dd7-a42c-bdb81aeb9438";
    const imgFrame28 = "https://www.figma.com/api/mcp/asset/110c7382-953d-4661-9fdc-eb0c1bd4493c";
    const imgShape = "https://www.figma.com/api/mcp/asset/8e22086a-f035-4542-be77-dcaf3d59ca26";
    const imgShape1 = "https://www.figma.com/api/mcp/asset/4dfb70c8-68d6-4192-a6c9-573d8de38819";
    const imgPath = "https://www.figma.com/api/mcp/asset/228ffc20-fac8-4721-8455-3a4973ecea85";
    const img202112 = "https://www.figma.com/api/mcp/asset/2f249888-fc6d-49c2-ab90-5229b5635aae";
    const imgMenu = "https://www.figma.com/api/mcp/asset/26a2f479-5a7a-401b-bb9e-f26d2d17beb5";
    type NoticeProps = {
      className?: string;
      property?: "default" | "info";
    };

    function Notice({ className, property = "default" }: NoticeProps) {
      const isDe...
    SUPER CRITICAL: The generated React+Tailwind code MUST be converted to match the target project's technology stack and styling system.
    1. Analyze the target codebase to identify: technology stack, styling approach, component patterns, and design tokens
    2. Convert React syntax to the target framework/library
    3. Transform all Tailwind classes to the target styling system while preserving exact visual design
    4. Follow the project's existing patterns and conventions
    DO NOT install any Tailwind as a dependency unless the user instructs you to do so.

    Node ids have been added to the code as data attributes, e.g. `data-node-id="1:2"`.
    These styles are contained in the design: Neutral1 (on Background, on Surface): #262D39, Title1_24B: Font(family: "Roboto", style: Bold, size: 24, weight: 700, lineHeight: 30, letterSpacing: 0), Primary1_2 (Primary Variants): #DFEBFF, BG (Background2, Surface1): #F5F6F7,#080808,
        Primary2 (Background1, Surface2): #FFFFFF, Primary1 (Primary): #066AEB, body1_16: Font(family: "Roboto", style: Regular, size: 16, weight: 400, lineHeight: 22, letterSpacing: 0), Shadow_Card : 택시_배차실패 리스트: Effect(type: DROP_SHADOW, color: #0047E808, offset: (0, 4),
        radius: 16, spread: 0), body2_14B: Font(family: "Roboto", style: Bold, size: 14, weight: 700, lineHeight: 20, letterSpacing: 0), Primary6_1 (Primary Variants): #3581FF, body2_14: Font(family: "Roboto", style: Regular, size: 14, weight: 400, lineHeight: 20, letterSpacing: 0),
        Neutral2 (on Background, on Surface): #262D39, Gray N6: #EEEEEF, Neutral4 (on Background, on Surface): #262D39, Primary2: #FFFFFF.
    Component descriptions: The following components have usage descriptions or documentation links defined in Figma. These descriptions provide important context about the intended usage, best practices, and any constraints for each component. Follow these guidelines when
        implementing or using these components.

    ## TitleBar / Web_title
    **Node ID:** 1267:12194

    우측메뉴\_폰트사이즈14로 변경 / 개발적용 전

    **Documentation:**
    - [https://www.figma.com/file/d0Zi2XllqTIHffhMCvNKW4/TitleBar_%EC%83%81%EC%84%B8?node-id=0%3A1](https://www.figma.com/file/d0Zi2XllqTIHffhMCvNKW4/TitleBar_%EC%83%81%EC%84%B8?node-id=0%3A1)
    Images and SVGs will be stored as constants, e.g. const image = 'https://www.figma.com/api/mcp/asset/550e8400-e29b-41d4-a716-446655440000'. These constants will be used in the code as the source for the image, ex: <img src={image} />. Image assets are stored on a remote server
        for 7 days and can be fetched using the provided URLs until they expire.
    <image content>
```

```
<frame id="2714:26162" name="운전면허증 인증/Empty" x="2409" y="256" width="360" height="1050">
    <rounded-rectangle id="2714:26163" name="bg gradient" x="0" y="72" width="360" height="930" />
    <text id="2714:26164" name="운전면허증을 어떻게 등록할까요?" x="20" y="100" width="195" height="60" />
    <frame id="2714:26165" name="img_180_driverlicense" x="20" y="192" width="320" height="188">
      <rounded-rectangle id="2714:26166" name="img_180_driverlicense" x="0" y="1" width="320" height="187.5" />
    </frame>
    <frame id="2825:24479" name="list" x="20" y="420" width="320" height="164">
      <instance id="2825:24480" name="license_card" x="0" y="0" width="320" height="76" />
      <instance id="2825:24481" name="license_card" x="0" y="88" width="320" height="76" />
    </frame>
    <instance id="2825:24482" name="notice" x="20" y="656" width="320" height="298" />
    <instance id="2714:26195" name="system / Bottom_Key_and" x="0" y="1002" width="360" height="48" />
    <instance id="2714:26196" name="system / Status Bar_and_onW" x="0" y="0" width="360" height="24" />
    <instance id="2714:26197" name="TitleBar / Web_title" x="0" y="24" width="360" height="56" />
  </frame>

  This is structure/geometry metadata only (no full visual/style internals).
```
