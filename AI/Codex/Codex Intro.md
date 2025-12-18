
# Codex CLI features

## Running in interactive mode

Codex 는 repo 읽기, 수정하기, command 를 실행하기 위한 full-screen terminal UI 를 실행함

```
# codex interactive mode 열기
codex

# 초기 prompt 를 포함시켜서 interactive mode 열기
codex "Explain this codebase to me"
```

세션이 활성화되어 있는 경우 아래 작업들을 할 수 있음

* prompt, code snippet, screenshots 를 전송
* codex 가 변경사항을 만들기 전에 plan 을 설명하는 것을 보고 inline 에서 허용/거절 할 수 있음
* TUI 에서 syntax-highlight 된 markdown 코드와 변경사항을 확인하기
* 위/아래 방향키를 사용해서 draft history navigate
* Ctrl + C 를 누르거나 `/exit` 을 통해서 interactive session 닫기

# Config file

## Config basics

Codex 는 하나 이상의 location 에서 configuration detail 을 읽음. 보안을 위해서 Codex 는 project 를 신뢰하는 경우에만 config 파일을 load 함.

### Codex Configuration file

* user-ldevel configuration defaults - `~/.codex/config.toml`
* project overrides - `.codex/config.toml`

CLI 와 IDE extension 은 같은 configuration layer 를 공유함.

* Configuration 으로 할 수 있는 것
	* default model, provider 설정
	* approval policies, sandbox setting 설정
	* MCP server 구성

### Configuration precedence 

1. CLI flag, `--config` override
2. Profile value (`--profile <name>`)
3. Project config file `.codex/config.toml` : project root 로부터 현재 작업하고 있는 directory 순
4. User config `~/.codex/config.toml`
5. Built-in defaults

만약 project 가 신뢰받지 못한다면 codex 는 project-scope 인 `.codex` layer 를 무시하고, user, system, built-in default 로 넘어갈 것임.

### Common configuration options

#### Default model

```toml
model = "gpt-5.2"
```

### Approval prompts

언제 Codex 가 생성된 command 를 실행하기 전에 멈추고 물어보는지를 설정

```toml
approval_policy = "on-request"
```

* `untrusted`
* `on-request`
* `never`

[Common sandbox and approval combinations](https://developers.openai.com/codex/security#common-sandbox-and-approval-combinations)

|Intent|Flags|Effect|
|Auto (preset)|flag 필요 없음 / `--full-auto`|Codex 는 workspace 내에서 읽기, 쓰기, command 실행이 가능함. Codex 는 workspace 외부에서 수정을 하거나 network 에 접근해야 할 때 허락이 필요함|
|Safe read-only browsing|`--sandbox read-only --ask-for-approval on-request`|Codex 는 파일을 읽고 질문에 답을 할 수 있음. 수정, command 실행, network 에 접근해야 할 때 허락이 필요함|
|Read-only non-interactive (CI)|`--sandbox read-only --ask-for-approval never`|Codex 는 읽기만 할 수 있고, 그외에는 허락을 받지 않음|
|Automatically edit but ask for approval to run untrusted commands|`--sandbox workspace-write --ask-for-approval untrusted`|Codex 는 파일을 읽고 쓸 수 있지만, 신뢰되지 않는 command 를 실행하기 전에는 허락을 받아야 함|
|Dangerous full access|`--dangerously-bypass-approvals-and-sandbox` / `--yolo`|Sandbox 없음, 허락 없음|


### Sandbox level

Codex 가 command 를 실행할 때 filesystem 과 네트워크에 얼마나 접근을 하게 할 것인지를 조정

```toml
sandbox_mode = "workspace-write"
```

[Sandbox and approvals](https://developers.openai.com/codex/security#sandbox-and-approvals)

Codex security 는 함께 동작하는 두 개의 layer 로 동작함

* Sandbox mode : Codex 가 model 이 생성한 command 를 실행할 때 기술적으로 무엇을 할 수 있는지. e.g. 어디에 write 를 할 수 있고 network 에 접근을 할 수 있는지
* Approval policy : action을 실행하기 전에 Codex 가 꼭 물어봐야 하는 시점이 언제인지 e.g. sandbox 를 떠날 때, network 를 사용할 때, 신뢰되지 않는 command 를 실행할 때

Codex 는 어디에서 실행하느냐에 따라 다른 sandbox 모드를 사용함

* Codex CLI/IDE extension : OS-level 매커니즘이 sandbox 정책을 강제함. 기본적으로는 네트워크 접근이 없고, write 허용 권한은 활성화된 workspace 에 제한되어 있음.

### Reasoning effor

Model 에 적용된 reasoning effort

```toml
model_reasoning_effort = "high"
```

### Communication style

```toml
personality = "friendly" # or "pragmatic" or "none"
```

`/personality` 로 active session 에서 overriding 가능.

### Command environment

Codex 가 생성된 command 에 대해 어떤 환경 변수를 forwarding 할 것인지

```toml
[shell_environment_policy]
include_only = ["PATH", "HOME"]
```

### Log directory

Codex 가 local log 파일을 작성하는 경로

```toml
log_dir = "/absolute/path/to/codex-logs"
```

### Feature flags

`config.toml` 에서 `[feature]` table 을 사용해서 optional / experimental 능력을 toggle

```toml
[features]
shell_snapshot = true           # Speed up repeated commands
```

# Rules

Rules : Codex 가 sandbox 밖에서 실행할 수 있는 command 를 관리함

## Create a rules file

1. `./codex/rules/` 생성 e.g. (`~/.codex/rules/default.rules`)
2. rule 추가
3. codex 재시작

```
# Prompt before running commands with the prefix `gh pr view` outside the sandbox.
prefix_rule(
    # The prefix to match.
    pattern = ["gh", "pr", "view"],

    # The action to take when Codex requests to run a matching command.
    decision = "prompt",

    # Optional rationale for why this rule exists.
    justification = "Viewing PRs is allowed with approval",

    # `match` and `not_match` are optional "inline unit tests" where you can
    # provide examples of commands that should (or should not) match this rule.
    match = [
        "gh pr view 7888",
        "gh pr view --repo openai/codex",
        "gh pr view 7888 --json title,body,comments",
    ],
    not_match = [
        # Does not match because the `pattern` must be an exact prefix.
        "gh pr --repo openai/codex view 7888",
    ],
)
```

Codex 는 startup 때 모든 team config location 아래의 `rules/` 를 스캔함. TUI 의 allow list 에 command 를 추가하면 Codex 는 그걸 `~/.codex/rules/default.rules` 에 써서 이후에는 prompt 를 스킵하도록 함.

# Model Context Protocol

Codex 가 third-party tools 과 context 에 접근할 수 있게 함

# Agent Skills

Agent skill 을 사용해서 특정 작업에 적합한 능력을 확장시키기. Skill 은 instruction, resource, optional script 를 패키징해서 Codex 가 workflow 를 안정적으로 수행할 수 있게 함. Skill 은 team, community 간에 공유할 수 있음. Skill 은 [open agent skills 표준](https://agentskills.io/home) 를 따름

Skill 은 progressive disclosure 를 사용해서 context 를 효율적으로 관리함. Codex 는 각 skill 의 metadata (`name`, `description`, file path, `agents/openai.yaml` 의 추가 메타 데이터) 로 시작함. Codex 는 skill 을 사용해야 하는 경우에 전체 `SKILL.md` instructions 을 load

Skill 은 `SKILL.md` 파일과 다른 optional script, reference 를 포함한 디렉토리. `SKILL.md` 파일은 `name`, `description` 을 꼭 포함해야 함

<img width="896" height="296" alt="Image" src="https://github.com/user-attachments/assets/4b3b8672-de67-4846-874a-fef3d7a9b855" />

## Codex 가 skill 을 쓰는 방법

1. 명시적 호출 : skill 을 prompt 에 직접 추가. CLI/IDE 에서 `/skills` 를 입력하거나 `$` 로 skill 을 언급
2. 암묵적 호출 : Codex 는 skill `description` 이 작업과 매칭될 경우 skill 을 사용할 수 있음

암묵적 매칭의 경우 