# multi-repo

Codex skill repository.

## Skills

- `multi-repo-worktree-orchestrator`
  - Orchestrates work across multiple Git repositories with repository-specific instructions, explicit user-provided branch names, and `git worktree` based isolation.

## Quick Start

Use `multi-repo-worktree-orchestrator` when one task spans multiple Git repositories and you want Codex to coordinate them safely.

Typical use cases:
- backend and frontend changes that must move together
- shared API, DTO, route, schema, or env-key updates across repositories
- work that should respect repository-specific instruction files such as `AGENTS.md` or `GEMINI.md`
- tasks that benefit from `git worktree` based isolation

Before asking Codex to use this skill, prepare:
- target repository paths
- base branch for each repository
- working branch name for each repository
- requested task breakdown
- validation command for each repository when you know it

Important rules:
- branch names are mandatory and must be provided by the user
- Codex must not invent branch names
- Codex should apply each repository's local instruction files first
- the same branch must not be used in multiple worktrees at the same time
- shared contracts should be frozen before parallel work starts

## How To Ask

Use a request like this:

```text
multi-repo-worktree-orchestrator로 next-planner-api 와 next-planner-front 를 같이 작업해줘.
repo 경로는 /Users/HH191_1/Documents/next-planner-api, /Users/HH191_1/Documents/next-planner-front 이야.
base branch 는 둘 다 develop 이고,
working branch 는 각각 feature/MGTT-123-api, feature/MGTT-123-front 이야.
API 응답 필드 추가랑 프론트 화면 연결까지 같이 해줘.
검증은 api는 ./gradlew test, front는 npm test 로 해줘.
```

You can also ask for a simpler orchestration summary first:

```text
multi-repo-worktree-orchestrator로 accounts-api 와 front-work 를 같이 볼건데,
각 repo 규칙 확인하고 브랜치 준비부터 해줘.
base branch 는 main, develop 이고
working branch 는 feature/account-sync-api, feature/account-sync-front 이야.
```

## What The Skill Does

The skill coordinates work in this order:
1. confirms the target repositories and instruction sources
2. checks whether branch names were provided for every repository
3. reads repository-local instruction documents such as `AGENTS.md` and `GEMINI.md`
4. checks branch existence locally and remotely
5. checks out, tracking-checks-out, or creates the exact user-provided branch
6. decides whether work should be sequential or parallel
7. creates or reuses `git worktree` only when isolation is safe
8. runs repository-appropriate validation and reports integration status

## Minimum Inputs

At minimum, provide:
- repository paths
- base branch per repository
- working branch per repository
- a clear task description

If branch names are missing, the skill is designed to stop and ask for them before doing setup work.
