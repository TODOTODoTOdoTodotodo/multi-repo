---
name: multi-repo-worktree-orchestrator
description: Orchestrate parallel work across multiple Git repositories by applying repo-specific instructions, requiring user-provided branch names, creating or reusing branches, and using git worktree for safe task isolation and result integration.
---

# Multi Repo Worktree Orchestrator

Use this skill when a task spans multiple Git repositories and should be coordinated with repository-specific instructions, explicit user-provided branch names, and `git worktree` based task isolation.

This skill supports two operating modes:
- Single-agent orchestration: one agent moves across repositories and worktrees while preserving repo-specific rules.
- Multi-agent expansion: a coordinator can later delegate disjoint repo or worktree scopes to sub-agents with the same orchestration model.

## Use This Skill When

- The user asks to modify multiple repositories together.
- The user wants API and frontend work coordinated in parallel.
- The user wants `git worktree` based isolation.
- The user wants repo-specific instruction documents applied during orchestration.
- The user wants separate branches and later integration across repositories.

## Core Rules

- Apply each repository's local instruction documents first, especially `AGENTS.md` and `GEMINI.md`.
- Treat user-provided instruction documents as repository-specific operating rules when local docs are missing or need supplementation.
- Do not revert or clean unrelated existing changes.
- Do not auto-generate branch names.
- Require branch names from the user before starting work.
- Use the exact branch name the user provided for each repository.
- If a branch already exists, check out the existing branch instead of creating a new one.
- If a branch exists only on the remote, check it out as a tracking branch.
- If the branch does not exist locally or remotely, create it with the exact user-provided name.
- Do not use the same branch in multiple worktrees at the same time.
- Do not split parallel work when files or responsibility boundaries overlap.
- Freeze shared contracts before parallelizing dependent implementation work.

## Execution Model

This skill does not open a new terminal application or create OS-level independent sessions.

Operate inside the current Codex execution context by:
- switching `workdir` per repository,
- using `git -C <repo>` when convenient,
- creating repository-specific worktrees,
- applying repository-specific instructions before edits,
- splitting work only when isolation is structurally safe.

The first implementation target is single-agent orchestration with multi-repo and multi-worktree coordination. Multi-agent execution is an extension of the same structure, not a separate workflow.

## Required Inputs

Collect or confirm the following before doing implementation work:
- target repositories,
- absolute path for each repository,
- repository-specific instruction document path or content,
- base branch for each repository,
- working branch name for each repository,
- requested task breakdown,
- shared contract or dependency notes,
- validation commands for each repository,
- merge or integration preference if the user provided one.

Branch-name policy:
- Branch names are mandatory input per repository.
- If the user does not provide a branch name, stop and ask for it.
- Do not infer, synthesize, or standardize a branch name without explicit user input.
- Preserve the user's branch name exactly as provided.

## Repository Intake

For each repository:
1. Confirm it is a Git repository.
2. Read local instruction files if present.
3. Check current branch, status, and existing worktrees.
4. Confirm the requested base branch exists.
5. Confirm the user-provided working branch name.
6. Determine whether the branch exists locally, remotely, or not at all.

Apply instruction precedence in this order:
1. local `AGENTS.md`
2. local `GEMINI.md`
3. user-provided repository instruction document
4. this orchestration skill

If rules conflict, prefer the repository-local instruction document.

## Branch Handling

For each repository:
1. Verify that the user provided a branch name.
2. If missing, ask the user for the branch name and do not proceed further.
3. If the branch exists locally, check it out.
4. If it exists only on the remote, check it out as a tracking branch.
5. If it exists nowhere, create it using the exact user-provided branch name.
6. Use that branch for any later worktree creation.

Additional rules:
- If the user provides the same branch name for multiple repositories, use that exact name in each repository.
- If the user provides different names per repository, preserve each repository's value exactly.
- User-provided naming takes priority over any branch naming convention.

## Task Decomposition

Split work into these categories:
- repository-specific work,
- shared cross-repository work,
- prerequisite work,
- dependent follow-up work,
- conflict-risk work,
- safe parallel work.

Decomposition rules:
- Prefer responsibility boundaries over raw file-count splitting.
- Do not parallelize tasks likely to touch the same file set inside one repository.
- Treat shared API contracts, DTOs, routes, schema changes, and env keys as prerequisite coordination work.
- If a cross-repository contract is unclear, clarify or freeze the interface before implementation.

## Parallelization Rules

Parallel work is allowed when:
- the repositories are different and the contract is already fixed, or
- the same repository can be split into non-overlapping responsibility scopes, or
- existing worktree usage does not conflict with the target branch.

Parallel work is not allowed when:
- the same branch would need to be active in multiple worktrees,
- the same files or ownership areas would overlap,
- the shared contract is still changing,
- current worktree occupancy would create branch conflicts.

If parallelization is unsafe, switch to sequential execution and say why.

## Worktree Strategy

Use `git worktree` only when it improves isolation and does not violate branch exclusivity.

Rules:
- Preserve the original working tree when it has unrelated or uncommitted changes.
- Create separate worktrees only for disjoint tasks.
- Reuse an existing checked-out branch when appropriate instead of creating redundant branches.
- If a required branch is already occupied by another worktree, do not force a second checkout of the same branch. Reorder work, reuse the existing worktree, or ask the user to choose.

Example path patterns:
- `../next-planner-api-<topic>`
- `../next-planner-front-<topic>`

## Editing and Validation

Work only inside the assigned repository or worktree scope.

After changes, run the best available validation in this order:
1. commands required by the repository instruction document,
2. commands explicitly provided by the user,
3. standard commands inferred from local project files such as `package.json`, `build.gradle`, or `Makefile`.

If validation cannot run, state the reason explicitly.

## Result Integration

Before integration, verify:
- shared contracts are consistent across repositories,
- no unresolved branch or worktree conflicts remain,
- validation results are known,
- target branches are clear,
- integration order respects prerequisites and dependencies.

If merge conflicts are likely, report the conflict boundary before attempting automatic integration.

## Final Output

Summarize:
- target repositories,
- applied instruction documents,
- base branch per repository,
- user-provided working branch per repository,
- whether each branch was checked out, tracking-checked-out, or newly created,
- worktrees created or reused,
- task decomposition result,
- parallel versus sequential execution decision,
- per-repository implementation result,
- per-repository validation result,
- integration result,
- remaining risks and manual follow-ups.

## Multi-Agent Extension

If the environment later supports safe sub-agent delegation for this workflow:
- keep one coordinator responsible for decomposition and ownership,
- assign each sub-agent one repository or one non-overlapping worktree scope,
- pass that repository's instruction documents and branch name with the task,
- prohibit two agents from editing the same branch or overlapping file scope,
- collect results centrally before integration.

## Response Pattern

At the start:
- confirm the target repositories and instruction sources,
- check whether branch names were provided for every repository,
- if any are missing, request them before doing setup work.

During execution:
- briefly report which repository, branch, and worktree are active,
- announce when shared contracts are being frozen before parallel work.

At the end:
- report branch handling, worktree handling, validation, and integration status,
- call out any remaining manual decisions or risks.
