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
2. Identify the child branch and its parent, then find the merge base (the commit where the child diverged).
3. Summarize the intent of both sides before touching any conflicts. This frames every subsequent resolution decision.
   - **Child intent**: read `git log --oneline <merge-base>..<child-tip>` and `git log -p <merge-base>..<child-tip>` (or `git diff <merge-base>..<child-tip>` when commits are noisy). Produce a short prose summary of *why* the child branch exists — the feature, fix, or refactor it is trying to land.
   - **Parent intent since divergence**: read `git log --oneline <merge-base>..<parent-tip>` and inspect each commit (`git show <sha>` or `git log -p`) to summarize *why* the parent moved — what features, refactors, renames, or deletions landed on the parent after the child branched off. Call out structural changes (renames, moves, extractions, deletions) explicitly because they drive the resolution policy.
   - Record both summaries before editing files. Use them to detect when a conflict is really an intent collision rather than a textual one.
   - **In plan mode**: surface both summaries to the user as part of the plan so they can confirm the framing before any resolution happens.
   - **In build mode**: keep the summaries internal and use them as guidelines while resolving conflicts. Do not pause to present them; only mention them in the final report when they explain a non-obvious resolution.
4. List unmerged paths with `git status --short` and `git diff --name-only --diff-filter=U`.
5. Inspect each conflicted path before editing, cross-referencing the two intent summaries to see whether the child's purpose still holds under the parent's new structure.
6. Classify the conflict:
   - plain line edit
   - delete/modify
   - rename or move plus modify
   - moved function or class
   - generated artifact
7. Resolve according to the structural policy.
8. Record any dropped or overridden child behavior for the user-facing summary.
9. Stage only resolved paths.
10. Run the smallest useful verification.
11. Continue the paused operation, preferring `gt continue` whenever Graphite is available.

When reporting back to the user in **plan mode**, include both intent summaries (child since divergence, parent since divergence) alongside the planned resolution approach. In **build mode**, the summaries are working notes — only reference them in the final report when they explain why a conflict was resolved a particular way (especially when child behavior was dropped or relocated).

### Picking the right refs

- For a Graphite stack, the parent is `gt parent` (or visible in `gt log short`); the child tip is `HEAD` (or the branch being restacked).
- For a plain rebase, the parent tip is the upstream (`@{upstream}` or the branch passed to `git rebase`); the child tip is the original branch HEAD before the rebase started (`ORIG_HEAD`).
- For a merge, both sides are explicit: `MERGE_HEAD` is the incoming side, `HEAD` is the current side.
- The merge base is `git merge-base <parent> <child>`.

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
