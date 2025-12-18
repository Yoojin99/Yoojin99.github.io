https://agentskills.io/home

Agent 에 새로운 능력과 전문성을 부여하기 위핸 간단한, 공개된 format. Claude code, cursor, codex 등 많은 AI 개발 tool 에서 채택하고 있음

Agent Skills 는 instruction, scripts, resource 의 folder 로 agent 가 작업을 더 정확하고 효울적으로 할 수 있게 함

# Overview

## Why Agent Skills?

Agent 는 유능하지만 실제 작업을하기 위해 필요한 context 를 가지고 있지 않은 경우가 많음. Skill 은 회사/팀/특정 유저 간의 지식에 접근할 수 있게 해서 on demand 로 load 할 수 있게 함. Skill 로 agent 는 능력을 확장할 수 있음

* Skill authors : 한 번 작성하고 여러 agent 에 배포할 수 있음
* Compatible agents : Skill 을 지원하면 사용자가 agent 에 새로운 능력 부여 가능
* Team, enterprise : 회사관련 지식을 전송 가능/버전 컨트롤 가능한 패키지로 capture 할 수 있음

## What can Agent Skills enable?

* Domain expertise : 특화된 지식을 재사용 가능한 instruction 으로 패키징할 수 있음 e.g. review process ~ data analysis pipeline
* 능력 확장 : agent 에 새로운 능력을 부여할 수 있음 e.g. 프레젠테이션 생성, MCP 서버 빌드, dataset 분석
* 반복 가능한 workflow : 여러 단계의 작업을 일정하고 검사 가능한 workflow 로 만들 수 있음
* Interoperability : 같은 skill 을 여러 다른 skill-compatible agent 에서 사용

# What are skills?

Agent Skills 는 특화된 지식과 workflow 로 AI agent 능력을 확장시키기 위한 간결한 공개된 포맷

Core 는 skill 은 `SKILL.md` 파일을 포함하고 있음. `SKILL.md` 파일은 메타데이터 (최소 `name`, `description` 를 포함하고 있음), 특정 작업을 어떻게 수행해야 하는지 agent 에게 알려주는 instruction 을 포함하고 있음. Skill 은 scripts, templates, reference materials 를 묶을 수 있음

```
my-skill/
├── SKILL.md          # Required: instructions + metadata
├── scripts/          # Optional: executable code
├── references/       # Optional: documentation
└── assets/           # Optional: templates, resources
```

## How skills work

Skill 은 context 를 효율적으로 관리하기 위해 progressive disclosure 를 사용함

1. Discovery : Startup 에서 agent 는 사용 가능한 각 skill 의 이름, description 만을 load 해서, 필요한 경우에 어떤 것을 사용할 수 있는지를 확인함
2. Activation : 작업이 skill 의 description 과 일치하는 경우 agent 는 전체 `SKILL.md` instruction 을 읽음
3. Execution : Agent 는 instruction 을 따르고, 참조된 파일을 로딩하거나 묶여진 코드를 실행함

이 접근법으로 agent 가 on demand 로 더 많은 context 에 접근할 수 있게 하면서 빠르게 유지할 수 있음

## The SKILL.md file

모든 skill 은 `SKILL.md` 로 시작함. 파일은 YAML frontmatter 와 Markdown instruction 으로 이루어짐

```markdown
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents.
---

# PDF Processing

## When to use this skill
Use this skill when the user needs to work with PDF files...

## How to extract text
1. Use pdfplumber for text extraction...

## How to fill forms
...
```

### Frontmatter

`SKILL.md` 파일의 가장 상단에 위치

* `name` : 짧은 identifier
* `description` : 언제 이 skill 을 사용할지

### Markdown body

실제 instruction을 포함함

위의 간단한 구조는 장점이 있음

* 간단한 구조의 장점
	* Self-documenting : 어떤 사용자가 읽어도 어떤 작업을 하는지 이해할 수 있고 개선할 수 있음
	* Extensible : Skill 은 텍스트 instruction 에서 실행가능한 code, asset, template 등으로 복잡도가 다양함
	* Portable : Skill 은 단순한 파일이기 때문에 수정, 버전 제어, 공유가 가능함

# Specification

## Directory structure

skill 은 최소 `SKILL.md` 를 포함한 directory

```
skill-name/
└── SKILL.md          # Required
```

skill 을 지원하기 위해 추가로 `scripts/`, `references/`, `assets/` 같은 디렉토리를 추가할 수 있음

## SKILL.md 형식

`SKILL.md` 파일은 YAML frontmatter 가 먼저 나오고 Markdown content 가 나옴

### Frontmatter (required)

```yaml
---
name: skill-name
description: A description of what this skill does and when to use it.
---
```

|Field|Required|Constraints|
|---|---|---|
|`name`|Yes|Max 64 characters. Lowercase letters, numbers, and hyphens only. Must not start or end with a hyphen.|
|`description`|Yes|Max 1024 characters. Non-empty. Describes what the skill does and when to use it.|
|`license`|No|License name or reference to a bundled license file.|
|`compatibility`|No|Max 500 characters. Indicates environment requirements (intended product, system packages, network access, etc.).|
|`metadata`|No|Arbitrary key-value mapping for additional metadata.|
|`allowed-tools`|No|Space-delimited list of pre-approved tools the skill may use. (Experimental)|

### Body content

Frontmatter 다음에 나오는 markdown 은 skill instruction 을 포함하고 있음. 형식에 대한 제한은 없고, agent 가 작업을 효율적으로 하는데 도움을 주는 방식으로 작성

* 추천하는 section
	* step-by-step instructions
	* input, output 의 예시
	* common edge cases

agent 는 skill 을 활성화할 때 전체 파일을 load 할 것이기 때문에 긴 `SKILL.md` 의 경우 referenced file 로 분리하는 것을 고려하기

## Optional directories

### scripts/

agent 가 실행 가능한 코드를 포함함

* self-contained / clearly document dependencies
* 도움이 되는 에러 메세지 포함
* edge case 를 잘 handling 함

지원되는 언어는 agent 구현에 따라 다르지만 흔한 옵션은 python, bash, javascript

### references/

필요한 경우 agent 가 읽을 수 있는 추가 document 