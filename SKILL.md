---
name: git-conflicts
description: Resolve Git conflicts during rebase, merge, cherry-pick, and Graphite restacks. Use when a repo is paused on unmerged paths, conflict markers, deleted-vs-modified files, renamed code, or moved functions, especially when parent-branch structural changes must be merged into the current branch.
---

# Git Conflicts

Use this skill when Git or Graphite has paused on conflicts and the next step is to inspect, resolve, stage, verify, and continue.

This skill is for paused conflict states, not general Git usage. Keep the workflow agent agnostic and grounded in the repository state.

## When To Use It

Trigger this skill when one or more are true:

- `git status` shows unmerged paths
- files contain conflict markers
- `git rebase`, `git merge`, or `git cherry-pick` is paused
- `gt restack` or `gt move` paused on conflicts
- the conflict involves deleted files, renamed files, moved functions, or extracted shared code

## Detect The Paused Operation

Detect which tool owns the paused state before resolving anything.

| Paused state | Continue command |
| :--- | :--- |
| Graphite-managed restack or move | `gt continue` |
| Graphite is available and owns the paused operation | `gt continue` |
| Plain Git rebase without Graphite ownership | `git rebase --continue` |
| Git merge | `git merge --continue` |
| Git cherry-pick | `git cherry-pick --continue` |

Rules:

- When Graphite is available for the paused operation, use `gt continue`.
- Do not use `git rebase --continue` for Graphite-managed conflicts.
- Treat `--ours` and `--theirs` carefully. Their meaning changes across merge, rebase, and cherry-pick flows.

## Core Resolution Policy

Default policy:

- For rebase, restack, and cherry-pick flows, accept parent structural changes first, then reapply any child-branch intent that still belongs in the new structure.
- For merge conflicts, inspect both sides symmetrically, but still prefer the surviving canonical location when code was renamed, moved, or extracted.
- Do not restore deleted code only because the child branch touched it.
- Preserve child behavior only when that behavior is still required and can be relocated cleanly into the parent's newer structure.
- If parent changes force removal or override of child behavior, resolve the conflict correctly and explicitly tell the user what behavior changed and why.
- Ask the user only when repository evidence is insufficient to determine product intent.

## Workflow

1. Detect the paused operation and the correct continue command.
2. List unmerged paths with `git status --short` and `git diff --name-only --diff-filter=U`.
3. Inspect each conflicted path before editing.
4. Classify the conflict:
   - plain line edit
   - delete/modify
   - rename or move plus modify
   - moved function or class
   - generated artifact
5. Resolve according to the structural policy.
6. Record any dropped or overridden child behavior for the user-facing summary.
7. Stage only resolved paths.
8. Run the smallest useful verification.
9. Continue the paused operation, preferring `gt continue` whenever Graphite is available.

## Whole-File Shortcuts

`git checkout --ours <path>` and `git checkout --theirs <path>` are allowed only after inspecting the conflict and confirming the whole file should come from one side.

Do not use those shortcuts blindly.

## Escalation And Notification

Ask the user when:

- both sides imply different product behavior
- a deletion may represent intentional feature removal rather than refactoring
- there is no clear successor location for moved or deleted logic
- binary files, submodules, or migrations are conflicted
- both sides changed an API contract incompatibly and repository evidence does not settle the intent

Always tell the user when:

- child-specific behavior was dropped
- child logic was partially preserved but had to move or change shape
- parent structural changes won and changed runtime or user-visible behavior from the child branch

## Conflict Archetypes

For detailed recipes, read [references/conflict-archetypes.md](references/conflict-archetypes.md) when the conflict is structural rather than just textual.
