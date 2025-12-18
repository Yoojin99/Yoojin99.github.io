
# Git Worktree

https://wikidocs.net/307247

Git worktree : 하나의 git 저장소에서 여러 브랜치를 동시에 작업할 수 있게 해주는 기능. 

* As-Is : 일반적으로는 브랜치 전환할 때 `git checkout`/ `git switch` 를 사용하는데 이 경우에는 한 번에 하나의 브랜치에만 작업할 수 있었음
* Git worktree : 같은 저장소의 여러 브랜치를 서로 다른 디렉토리에서 동시에 작업할 수 있어, 브랜치 전환 없이 빠르게 여러 작업을 수행할 수 있음

* Use case
	* 메인 작업 도중 긴급한 버그 수정이 필요한 경우
	* 여러 기능을 병렬로 개발하면서 각각을 빌드/테스트해야 할 때
	* 현재 브랜치의 변경 사항을 커밋하지 않고 다른 작업을 해야 할 때햣 

# Run parallel Claude Code sessions with Git worktrees

여러 작업을 한 번에 수행하는 경우에 각 Claude 세션이 codebase 의 각 copy 를 가져서 충돌하지 않도록 해야 한다.

**Git worktree**는 별도의 작업 디렉토리를 생성해서 각자의 파일과 브랜치를 가지도록 하면서, 같은 repository history 와 remote connection 을 가지도록 해서 이런 문제를 해결함.

이를 통해 Claude 는 하나의 worktree 에서는 피쳐 작업을 하고, 다른 worktree 에서는 버그를 수정하는 작업을 하면서 서로 다른 세션에서 독립적으로 일할 수 있음.

`--worktree` (`-w`) flag 를 사용해서 독립된 worktree 를 생성하고, Claude 를 해당 worktree 에서 실행할 수 있음. 전달하는 값은 worktree directory 이름, branch 이름이 됨.

```bash
# Start Claude in a worktree named "feature-auth"
# Creates .claude/worktrees/feature-auth/ with a new branch
claude --worktree feature-auth

# Start another session in a separate worktree
claude --worktree bugfix-123
```

이름을 뺄 경우 Claude 가 자동으로 하나를 만들어줌

```bash
# Auto-generates a name like "bright-running-fox"
claude --worktree
```

Worktree 는 `<repo>/.claude/worktrees/<name>` 에 생성되고, branch 는 default remote branch 에서 생성됨. Worktree branch 이름은 `worktree-<name>`.

