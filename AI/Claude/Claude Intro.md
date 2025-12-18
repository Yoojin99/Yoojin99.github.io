
# Essential commands

https://code.claude.com/docs/en/quickstart#essential-commands

|Command|What it does|Example|
|---|---|---|
|`claude`|Start interactive mode|`claude`|
|`claude "task"`|Run a one-time task|`claude "fix the build error"`|
|`claude -p "query"`|Run one-off query, then exit|`claude -p "explain this function"`|
|`claude -c`|Continue most recent conversation in current directory|`claude -c`|
|`claude -r`|Resume a previous conversation|`claude -r`|
|`claude commit`|Create a Git commit|`claude commit`|
|`/clear`|Clear conversation history|`/clear`|
|`/help`|Show available commands|`/help`|
|`exit` or Ctrl+C|Exit Claude Code|`exit`|

# Core Concepts

Claude Code 는 terminal 에서 실행되는 agentic assistant.

## The agentic loop

Claude 에 작업을 주면 세 phase 를 거침. 

* gather context
* take action
* verify results

![](https://mintcdn.com/claude-code/ELkJZG54dIaeldDC/images/agentic-loop.svg?w=2500&fit=max&auto=format&n=ELkJZG54dIaeldDC&q=85&s=20dedb60b95d45a1bd60a0cccaf3e1ff)

Loop 는 어떤 작업을 요청했는지에 따라 다르게 적용함. e.g. codebase 에 대한 질문 - context gathering 만 / bug fix cycle - 모든 phase 를 거침. 

Claude 는 이전 단계에서 배운 것을 바탕으로 각 단계에서 어떤 것이 필요한지를 결정함. 사용자는 이 loop 의 일부이기 때문에 중간에 끼어들어서 다른 방향으로 조정하거나 추가 context 를 제공할 수 있음

Claude Code 는 Claude 에 agentic harness 를 제공함. Tool, context management, execution environment 를 제공해서 언어 모델이 코딩 에이전트로 동작할 수 있게끔 함

Agentic loop 은 두 개의 component 로 구동됨. 

* models : 추론
* tools : 도구

## Models

Claude Code 는 Claude model 을 사용해서 code 와 작업에 대한 추론 (두뇌 역할) 을 할 수 있게 함. 

목적마다 model 을 다르게 선택 가능. `/model` 을 통해 모델을 변경할 수 있음 

* Sonnet - 대부분의 코딩 작업
* Opus - 복잡한 아키텍처 추론 작업

## Tools

Tools 는 Claude Code 를 agentic 하게 만듦. Tools 없이 Claude 는 text 만 응답할 수 있음. Tools 를 사용해서 Claude 는 code 를 읽고 파일을 수정, 웹 서치, 외부 서비스와 interact 할 수 있음. 각 tool 은 loop 에 정보를 feed back 함

이미 내장된 4개의 카테고리가 있고 각 카테고리는 다른 agency 성격을 띔

|Category|What Claude can do|
|---|---|
|**File operations**|Read files, edit code, create new files, rename and reorganize|
|**Search**|Find files by pattern, search content with regex, explore codebases|
|**Execution**|Run shell commands, start servers, run tests, use git|
|**Web**|Search the web, fetch documentation, look up error messages|
|**Code intelligence**|See type errors and warnings after edits, jump to definitions, find references (requires [code intelligence plugins](https://code.claude.com/docs/en/discover-plugins#code-intelligence))|

Claude 는 subagent 를 spawn 하거나 사용자에게 질문을 묻거나 다른 작업을 위한 tool 도 갖고 있음.

[Tool 목록](https://code.claude.com/docs/en/settings#tools-available-to-claude)

Claude 는 prompt 에 따라, 또 배우는 것에 따라 tool 을 다르게 선택함.

e.g. "fix the failing tests" prompt

1. test 를 실행해서 실패한 test 확인
2. error output 읽음
3. 관련 source file 찾음
4. 해당 file 을 읽어서 코드를 이해함
5. file 을 수정해서 이슈 수정
6. test 를 다시 수행해서 검증

**Base-capabilities 확장** : 내장된 tool 은 foundation. 아래의 extension 을 사용
* `skill` 로 Claude 의 지식을 확장 (for workflow)
* `MCP` 를 사용해서 외부 서비스에 연결 (for external services)
* `hook` 로 workflow 자동화 
* `subagent` 에 task offload (for delegated work) 

## What Claude can access

특정 디렉토리에서 `claude` 를 실행하면 아래에 대한 접근을 얻음

* 내 Project : directory, subdirectory, file
* Terminal : 내가 실행할 수 있는 모든 command (build tools / git / package managers). Command 로 할 수 있는 작업은 claude 가 다 할 수 있음
* git state : 현재 branch, uncommitted change, recent commit history
* `CLAUDE.md` : Claude 가 모든 seesion 에서 알아야 하는 project-specific instruction, convention, context 을 저장한 md 파일
* 내가 구성한 extensions : MCP servers, skills, subagent, Claude in Chrome (browser interaction)


## Work with sessions

Claude Code 는 conversation 을 local 하게 저장함. 각 message, tool use, result 는 저장되고, rewinding, resuming, forking 할 수 있음. Claude 가 code 를 수정하기 전에 snapshot 을 떠서 원할 경우 revert 할 수 있게 함

**Session 은 독립적.** 각 새로운 session 은 새로운 context window 로 시작하고, 이전 session 의 conversation history 는 모름. Claude 는 auto memory 를 사용해서 세션을 거득ㅂ해서 배울 수 있고, 이런 persistent instruction 을 CLAUDE.md 에 넣을 수 있음.

### Work across branches

각 Claude Code conversation 은 현재 directory 에 엮인 session 이고, resume 할 경우 해당 directory 의 세션들만 볼 수 있음.

Claude 는 현재 branch 의 파일들을 봄. Branch 를 switch 할경우 Claude 는 새로운 branch 의 파일을 보지만 conversation history 는 그대로 유지됨. Claude 는 switching 된 이후에도 논의된 것을 기억함.

Session 이 디렉토리에 묶여져 있기 때문에 git worktrees 를 사용해서 parallel Claude session 을 실행할 수 있음.

### Resume or fork sessions

`claude --continue` / `claude --resume` 을 사용해서 session 을 resume 할 경우 같은 session ID 를 사용했던 곳에서 다시 시작함. 새로운 메세지는 이미 존재하는 conversation 에 추가됨.  전체 conversation history 는 다시 불러와지지만 session-scoped 권한은 그렇지 않기 때문에 다시 허용해줘야 함

![](https://mintcdn.com/claude-code/ELkJZG54dIaeldDC/images/session-continuity.svg?w=2500&fit=max&auto=format&n=ELkJZG54dIaeldDC&q=85&s=d67e1744e4878813d20c6c3f39d9459d)

원래의 session 에 영향을 주지 않고 branch off 를 해서 다른 접근을 하려면 `--fork-session` 플래스를 사용

```
claude --continue --fork-session
```

위 명령어는 conversation history 를 유지하면서 새로운 sessionID 를 생성함. 원래의 session 에는 변화가 없음.

**여러 terminal 에서 같은 session 을 사용할 경우** : 모든 terminal 이 같은 session 파일에 write 를 하게 됨. 충돌은 없지만 conversation 은 뒤섞여질 것. `--fork-session` 을 사용해서 각 terminal 에서 자신만의 clean session 을 사용할 것을 추천

### The context window

Claude 의 context window 가 hodling 하고 있는 것들

* conversation history
* file content
* command outputs
* CLAUDE.md 
* loaded skills
* system instructions

작업하면서 context 는 채워지고, Claude 는 자동으로 압축을 하는데 conversation 앞부분의 instruction 이 잊혀질 수 있음. **Persistent rule 을 `CLAUDE.md` 에 넣고**, `/context` 를 실행해서 차지하는 공간 확인 가능

#### When context fills up

Cluade Code 는 한계에 도달할 때까지 context 를 자동으로 관리함. 오래된 tool 결과를 먼저 정리하고, 필요한 경우 conversation 을 요약함. 사용자의 요청, key code snippets 는 유지되지만 오래된 detailed instruction 은 잊혀질 수 있음.

압축에서 유지되어야 하는 것들은 CLAUDE.md 에 "Compact Instructions" section 을 추가하거나 `/compact` 를 집중해야 하는 것들과 함께 실행시킴. e.g. `/compact focus on the API chages`

`/context` 를 실행해서 현재 사용되는 공간 확인. MCP server 는 매 요청마다 tool 정의를 추가하기 때문에 오직 적은 수의 서버들만이 충분한 context 를 실행할 수 있음. `/mcp` 를 실행해서 서버마다 cost 가 얼마나 드는지 확인.

#### Manage context with skills and subagents

압축과는 별개로 어떤 것들이 context 에 들어갈지를 다른 기능을 사용해서 제어할 수 있음.

* Skills : on demand 로 load 됨. Claude 는 session 시작에서 skill description 을 보는데, skill 이 사용될 때만 전체 content 를 load 함. 직접 invoke 하는 skill 은 `disable-model-invocation: true` 를 사용해서 필요할 때까지 context 에서 제외시킬 수 있음
* Subagents : 메인 conversation 에서 분리된 자신만의 fresh context 를 가짐. Subagents 의 작업은 context 를 넘치게 하지 않고, subagent 의 작업이 완료된 경우에 요약본을 return.

## Stay safe with checkpoints and permissions

Claude 는 두 가지 safety 매커니즘이 있음

* checkpoint : 파일 변경을 undo 할 수 있게 함
* permissions control : Claude 가 질문 없이 어떤 작업을 수행할 수 있는지를 제어

### Undo changes with checkpoints

모든 파일 수정은 reversible. Claude 가 어떤 파일을 수정하기 전에 현재 내용을 snapshot 을 찍음. 뭔가 잘못되면 Esc 를 두 번 눌러 이전 상태로 돌아가게 하거나 Claude 에게 undo 하라고 할 수 있음

Checkpoint 는 git 과는 별개로 session 에 local 함. 파일 변경에 대해서만 커버함. Remote system (database, API, 배포) 에 대한 변경은 checkpoint 될 수 없기 때문에 Claude 는 외부 side effect 를 일으키는 명령을 실행할 때 사용자에게 먼저 물어봄

### Control what Claude can do

`Shift+Tab` 을 눌러 권한 모드를 switching 할 수 있음

* Default : Claude 는 파일 수정, shell command 수행 전 물어봄
* Auto-accept edits : 질문 없이 파일을 수정하지만 command 는 물어봄
* Plan mode : Claude 는 read-only tool 만 사용하고, 실행전에 사용자가 승인할 수 있는 plan 을 만듦
* Delegate mode : Claude 는 직접 구현 없이 agent teammates 들과만 작업함. agent team 이 활성화되어 있는 경우만 사용 가능

특정 command는 `.claude/settings.json` 에 넣어서 Claude 가 매번 물어보지 않게 할 수 있음.

## Work Effectively with Claude Code

### Ask Claude Code for help

Claude Code 는 어떻게 사용해야 하는지를 알려줄 수 있음 e.g. "hook 를 어떻게 설정해?" / "CLAUDE.md 를 구성하는 가장 좋은 방법이 뭐야"

내장된 command

* `/init` : 프로젝트 내 `CLAUDE.md` 를 생성하는데 도움을 줌
* `/agents` : custom subagent 를 구성하는데 도움을 줌
* `/doctor` : 설치된 것을 기반으로 흔한 issue 를 진단해줌

### It's a conversation

Conversational 하기 때문에 완벽한 prompt 는 필요하지 않고, 원하는 것으로 시작해서 정제하는 방향으로

```
> Fix the login bug

[Claude investigates, tries something]

> That's not quite right. The issue is in the session handling.

[Claude adjusts approach]
```

#### Interrupt and steer

어떤 시점이든 Claude 를 interrupt 할 수 있음. 올바르지 않은 길로 가고 있으면 수정된 내용을 입력하고 엔터 입력. Claude 는 현재하고 있는 작업을 멈추고 input 을 바탕으로 접근 방법을 수정할 것. 완료되기까지 기다리거나 처음부터 실행할 필요가 없다!

### Be specific upfront

초기 prompt 가 정확할수록 수정해야 하는 횟수는 적어짐. 특정 파일을 명시하고, 제약사항을 추가하고, 예시 패턴을 알려줘라

```
> The checkout flow is broken for users with expired cards.
> Check src/payments/ for the issue, especially token refresh.
> Write a failing test first, then fix it.
```

"fix the login bug" 같은 모호한 prompt 대신 특정 포인트를 집는 prompt 가 좋음

### Give Claude something to verify against

Claude 는 자신의 작업을 확인할 수 있을 때 성능이 좋음. Test case, 예상되는 UI 스크린샷, 예상되는 output 을 포함시키는 것이 좋음

```
> Implement validateEmail. Test cases: 'user@example.com' → true,
> 'invalid' → false, 'user@.com' → false. Run the tests after.
```

### Explore before implementing

복잡한 문제의 경우 coding 과 research 를 분리하기. Plan mode 를 사용해서 codebase 를 먼저 분석.

```
> Read src/auth/ and understand how we handle sessions.
> Then create a plan for adding OAuth support.
```

Plan 을 검토하고 conversation 을 통해 정제하고 Claude 가 구현하게 하기. Plan 을 껴서 plan-implement 두 단계로 만들면 바로 code 를 만드는 것보다 좋은 결과를 불러옴

### Delegate, don't dictate

Context, direction 을 주고 Claude 가 detail 을 추론하도록. 즉 구체적인 명령어를 명시하거나 할 필요는 없다는 뜻

```
> The checkout flow is broken for users with expired cards.
> The relevant code is in src/payments/. Can you investigate and fix it?
```

# Extend Claude Code

Claude Code 는 code 에 대한 추론을 하는 model + 내장된 tool (for file operation, search, execution, web access). 내장된 tool 은 대부분의 코딩 작업을 커버함.

Extension layer : Claude 가 아는 것을 customize, 외부 service 에 연결, workflow 를 자동화하는 기능

## Overview

Extensions 는 agentic loop 의 다른 부분에 plug 됨

* CLAUDE.md : Claude 가 매 session 에서 보는 persistent context 를 더함
* Skills : 재사용 가능한 지식과 workflow 추가
* MCP : Claude 를 외부 서비스, tool 에 연결
* Subagents : 자신의 loop 를 고립된 context 에서 실행하고 요약본 return
* Agent teams : 여러 독립된 session 을 협력해서 공유된 task 를 실행함
* Hooks : loop 외부에서 실행되는 독립적인 script
* Plugins, marketplaces : 위의 기능을 package, distribute

Skills : 가장 유연한 extension. Skill 은 지식, workflow, instruction 을 포함한 markdown 파일. Skill 은 `/deploy` 와 같은 command 로 실행할 수 있고, Claude 가 판단했을 때 관련있는 경우 자동으로 load 함. 현재 conversation 이나 subagent 를 통한 고립된 context 에서 skill 이 실행될 수 있음

## Match features to your goal

Features 는 Claude 가 모든 session 에서 보는 always-on context 부터 Claude 나 사용자가 직접 깨우는 on-demand, 특정 event 에서 background 에서 돌아가는 것까지 다양함

|Feature|역할|언제 사용하는지|예시|
|-|-|-|-|
|CLAUDE.md|모든 conversation 에서 load 되는 persistent context|Project conventions, "always do X" rules|"Use pnpm, not npm. Run tests before commiting."|
|Skill|Claude 가 사용할 수 있는 Instructions, knowledge, workflows|Reusable content, reference docs, repeatable tasks|"/review" 는 코드의 review checklist 를 확인함|
|Subagent|요약된 결과를 리턴하는 고립된 execution context|Context isolation, parallel tasks, specialized workers|많은 파일을 읽지만 핵심 결과물만 리턴하는 research 작업|
|Agent teams|여러 독립된 Claude Code session 이 협동함|Parallel research, new feature development, debugging with competing hypotheses|Reviewer 를 spawn 해서 security, performance, test 를 동시에 확인할 때|
|MCP|외부 service 에 연결할 때|외부 data/actions|Database query, Slack 에 post, browser control|
|Hook|특정 이벤트일 때 실행되는 script|Predictable automation, LLM 개입 필요 없을 때|모든 파일 수정마다 ESLint 실행|

Plugin 은 packaging layer. Plugin 은 skill, subagent, MCP server 를 하나의 설치 가능한 unit 으로 묶음. Plugin skill 은 namespaced (e.g. `/my-plugin:review`) 되기 때문에 여러 plugin 이 동시에 존재할 수 있음. Plugin 을 사용해서 같은 setup 을 여러 repo 에서 재사용하거나 marketplace 를 통해 다른 곳에 distribute 할 수 있음.

### Compare similar features

#### Skill vs Subagent

Skill, subagent 는 다른 문제를 해결함

* Skill : 어떤 context 에서도 load 할 수 있는 재사용 가능한 content
* Subagent : main conversation 에서 별개로 실행되는 고립된 worker

|Aspect|Skill|Subagent|
|-|-|-|
|무엇인지|Reusable instructions, knowledge, or workflows|자신만의 context 로 실행될 수 있는 고립된 worker|
|장점|Context 간에 content 를 공유함|Context isolation. 작업은 별개로 일어나고 요약만이 return 됨|
|Best for|Reference material, invocable workflows|여러 파일을 읽거나, 병행으로 수행해야 하는 작업|

Skill : Reference / action. 
	* Reference skills : session 동안 Claude 에 지식 부여. e.g. API style guid
	* Action skills : Claude 가 특정 작업을 하도록 함 e.g. 배포 workflow 를 실행하는 `/deploy`

Subagent : Context 가 full 에 가까워지고 있을 때나 context isolation 이 필요한 경우. Subagent 는 엄청 많은 수의 파일을 읽거나 많은 검색을 수행할 수 있고, main conversation 은 오직 요약만 받을 수 있음. Subagent 는 main context 를 소모하지 않기 때문에 중간 단계의 작업이 visible 할 필요가 없을 때 유용함. Custom subagent 는 자신만의 instruction 을 가질 수 있고 skill 을 미리 load 할 수 있음

Skill + Subagent combine 가능. Subagent 는 특정 skill 을 preload 할 수 있고 skill 은 `context: fork` 를 사용해서 고립된 context 에서 실행될 수 있음

#### CLAUDE.md vs Skill

모두 instruction 을 포함하고 있지만 다르게 load 되고 다른 목적을 갖고 있음

|Aspect|CLAUDE.md|Skill|
|-|-|-|
|Loads|매 session, 자동|On demand|
|파일 포함 가능|O, `@path` import 를 통해|O, `@path` import 를 통해|
|workflow trigger 가능|X|O, `/<naame>` 을 통해|
|Best for|"Always do X" rules|Reference material, invocable workflows|

* CLAUDE.md : Claude 가 항상 알고 있어야 하는 것 (coding conventions, build commands, project structure, "never do X" rule) 포함
* Skill : Claude 가 `/<name>` 으로 실행시킬 수 있는 workflow 같은 reference material 가 필요한 경우

Rule of thumb : CLAUDE.md 는 500 줄 이하로 유지. 만약 증가하는 경우 reference content 를 skill 로 이동

#### Subagent vs Agent team

둘 다 작업을 병행화시키지만 구조적으로 다름

* Subagents : session 안에서 실행되고 결과를 main context 에게 반환함
* Agent teams : 서로 소통하는 독립적인 Claude Code session

|Aspect|Subagent|Agent team|
|-|-|-|
|Context|자신만의 context window 를 갖고 있음. 호출자에게 결과를 반환|자신만의 context window 있으며 완전히 독립적임|
|Communication|결과를 main agent 에 report|Teammates 는 서로에게 직접적으로 메세지 함|
|Coordination|Main agent 가 모든 작업을 관리함|공유되는 task|
|Best for|결과만이 중요한 작업|협력과 토론이 필요한 복잡한 작업|
|Token cost|낮음: 결과가 main context 에 요약되어서 전해짐|높음: 각 teammate 가 독립적인 Claude instance|

* Subagent : 더 빠르고, 한 작업에 집중하는 작업자가 필요한 경우. 조사, 검증을 하거나 파일을 리뷰하는 경우. Subagent 는 작업을 하고 요약본을 return 함. 메인 conversation 은 clean 하게 유지됨
* Agent team : teammate 가 finding 을 공유하거나 독립적으로 협력해야 하는 필요가 있는 경우. 충돌되는 가설을 조사하거나 병행 코드 리뷰, 각 teammate 가 독립적인 작업을 갖는 새로운 기능 개발을 하는 경우 좋음

Transition point : parallel subagent 를 실행하고 있는데 context limit 에 도달했을때 / subagent 가 서로와 소통해야 하는 경우 agent team 을 사용하는 것을 고려

#### MCP vs Skill

MCP 는 Claude 를 external service 에 연결하고 Skill 은 Claude 가 아는 것을 확장시킴 (외부 service를 어떻게 효율적으로 사용해야하는지 포함)

|Aspect|MCP|Skill|
|-|-|-|
|What it is|외부 서비스에 연결하기 위한 Protocol|지식, workflow, reference material|
|Examples|Slack integration, database queries, browser control|Code review checklist, deploy workflow, API style guide|

* MCP : Claude 가 외부 시스템과 소통할 수 있게 함. MCP 없이 database 에 query 를 날리거나 Slack 에 posting 을 할 수 없음
* Skills : Claude 가 tool 을 효율적으로 쓸 수 있는지에 대한 지식을 주고 workflow 를 `/<name>` 으로 실행할 수 있게 해줌. Skill 은 팀의 database schema, query pattterns 를 포함할 수 있음

e.g. MCP 서버는 Claude 를 database 에 연결함. Skill 은 Claude 에게 data model, common query patterns, 어떤 tables 을 사용해야 하는지를 알려줌

### Understand how features layer

Feature 는 여러 level 에서 정의될 수 있음 : user-wide, per-project, via plugin, managed policies. Subdirectories 에서 CLAUDE.md 파일을 nesting 할 수 있고, 모노레포에서 특정 package 에 skill 을 위치시킬 수 있음. 여러 레벨에서 같은 feature 가 등장할 경우에는 아래와 같이 동작함

* CLAUDE.md 파일은 additive : 모든 레벨은 Claude 의 context 에 내용을 추가함. Instruction 에 충돌이 있을 때 Claude 는 judgement 를 사용해서 특정 instruction 은 선행으로 충돌난 부분을 해결함
* Skills & subagent 은 이름으로 override 됨 : 같은 이름이 여러 level 에 존재하는 경우, 특정 priority 에 따라 특정 정의만을 따르게 됨 (managed > user > project for skills > CLI flag > project > user > plugin for subagents). Plugin skill 은 충돌을 피하기 위해 namespaced 됨.
* MCP servers 는 이름으로 override 됨. : local > project > user
* Hooks 는 merge 됨 : 등록된 hook 는 소스와 상관없이 일치하는 이벤트에 의해 실행됨

### Combine features

각 extension 은 다른 문제를 해결함

* CLAUDE.md : always-on context
* Skills : on-demand knowledge, workflows
* MCP : external connections
* subagents : isolation
* hooks  : automation

실제 환경에서는 workflow 에 따라 이들을 섞어서 씀.

e.g. CLAUDE.md 에는 project convention, skill 은 deployment workflow, MCP 를 사용해서 database 에 연결, hook 을 사용해서 모든 수정마다 linting.

|Pattern|How it works|Example|
|-|-|-|
|Skill + MCP|MCP 가 연결을 제공하고, skill 은 Claude 가 어떻게 그걸 잘 사용할지를 제공|MCP 는 database 에 연결하고, skill 은 schema 와 query pattern 은 문서화함|
|Skill + Subagent|Skill이 parallel 작업을 위한 subagent 를 spawn|`/review` skill 은 고립된 context 에서 수행되는 보안, 성능, 스타일 subagent 를 실행함|
|CLAUDE.md + skills|CLAUDE.md 는 always-on rule 를 담고 있고, skill 은 on deamnd 에 의해 로딩되는 reference material 을 갖고 있음|CLAUDE.md 는 "우리의 API convention을 따라라" 하고 skill 은 전체 전체 api style guide 에 대한 참조를 갖고 있음|
|Hook + MCP|Hook 는 MCP 를 통해 외부 action 을 실행함|Post-edit hook 는 Claude 가 중요한 파일을 수정했을 때 slack 노티를 보냄|

## Understand context costs

모든 feature 는 Claude 의 context 를 소모함. 너무 많으면 context 를 채우고 Claude 를 덜 효율적으로 만드는 noise 를 만들 수도 있음. Skill 이 올바르게 실행되지 않을 수도 있고, Claude 가 conversation 을 제대로 추적하지 못할 수도 있음.

### Context cost by feature

각 feature 는 다른 loading 전략과 context cost 를 갖고 있음

|Feature|언제 load 되는지|무엇을 load 하는지|Context cost|
|-|-|-|-|
|CLAUDE.md|Session 시작 시|Full content|모든 request 마다|
|Skills|Session 시작 + 사용될 때|시작할 때 descriptions, 사용될 때 전체 content|낮음|
|MCP servers|Session 시작|모든 tool 정의와 schema|모든 request 마다|
|Subagents|spawn 될 때|fresh context with specified skills|메인 세션과는 분리됨|
|Hooks|On trigger|X|추가 context 를 리턴하지 않는한 0|

기본적으로 skill descriptions 는 세션 시작시 load 돼서 Claude 로 하여금 언제 사용할 지를 결정할 수 있게 해줌. Skill 의 frontmatter 에 `disable-model-invocation: true` 로 설정하면 사용자가 직접 호출하지 않는 이상 Claude 에게 노출시키지 않음. 이는 직접 호출해야 하는 skill 이 있는 경우 context cost 를 줄여줌


### Understand how features load

각 feature 는 session 의 다른 point 에서 load 됨. 

![](https://mintcdn.com/claude-code/ELkJZG54dIaeldDC/images/context-loading.svg?w=1100&fit=max&auto=format&n=ELkJZG54dIaeldDC&q=85&s=9844c55d08d2c386672447f2e8518669)

* CLAUDE.md 
	* 언제 : 세션 시작시
	* What loads: CLAUDE.md 파일의 전체 content (managed, user, project level)
	* 상속 : Claude 는 작업 디렉토리 전체의 CLAUDE.md 파일을 읽고 중첩된 것들을 확인 함.
	* Tip : CLAUDE.md 는 500 줄 이하로 유지하고, reference material 은 skill 로 만들어서 on-demand 에서 load되게 함
* Skills : Skills 는 Claude 의 toolkit 의 추가 capabilities. Reference material / `/<name>` 으로 실행하는 invocable workflow. 어떤 것은 내장되어 있고, 직접 만들 수 있음. Claude 는 자기가 알아서 실행할 수도 있고 사용자가 직접 실행할 수도 있음
	* 언제 : skill configuration 에 따라 다름. 기본적으로 session 시작시 description 이 load 되고, 전체 content 는 사용될 때 load 됨. User-only skills (`dsiable-model-invocation: true`) 인 경우 invoke 하기 전에는 아무것도 load 되지 않음 
	* What loads : Model-invocable skill 인 경우 CLaude 는 모든 request 에서 이름, description 을 봄. `/<name>` 으로 skill 을 호출하거나 Claude 가 자동으로 load 할 때 전체 content 가 conversation 에 load 됨
	* Claude 가 어떻게 skill 을 고르는지 : Claude 는 작업과 skill description 을 matching 시켜서 어떤 skill 이 연관 있는지 확인 함. Descriptions 이 모호한 경우 엉뚱한 skill 을 load 하거나 호출을 안 할수도 있음. 특정 skill 을 사용하게 하고 싶으면 `/<name>` 으로 호출하기. 
	* Context cost : 사용되기 전까지 low. User-only skill 의 경우 호출되기 전까지 0 cost
	* In subagents : skills 은 subagent 에서 다르게 동작함. On-demand loading 과는 다르게, skill 은 시작될 때 subagent 에 완전히 preloading 됨. Subagent 는 메인 세션 에서 skill 을 상속받지 않고, 명시적으로 언급해줘야 함.
* MCP server
	* 언제 : 세션 시작시
	* What loads : 연결된 서버의 모든 tool 정의와 JSON schema
	* Context cost : Tool search (기본적으로 활성화되어 있음) 은 context 의 10%까지 MCP tool 을 불러옴
	* Reliability note : MCP connection 은 세션 도중에 silent 하게 실패할 수 있음. 서버 연결이 끊어지면 tool 은 경고 없이 사라지게 됨. Claude 는 더 이상 존재하지 않는 tool 을 사용하려고 할 수 있음. CLaude 가 MCP tool 을 사용하는 것을 실패한다면 `/mcp` 명령어로 연결을 확인할 수 있음
* 



