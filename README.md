# Git conflicts

`git-conflicts` helps an agent resolve paused Git conflict states safely and consistently.

It is designed to:

- detect which tool owns the paused operation
- resolve structural conflicts such as deleted files, renames, and moved logic
- prefer parent-branch structural changes in rebase-like flows
- preserve child intent only when it still belongs in the new structure
- tell the user when resolving the conflict required dropping or overriding child behavior
- resume with the correct continue command, using `gt continue` when Graphite is available

## When to use it

Use `git-conflicts` when a repository is paused on:

- `git rebase`
- `git merge`
- `git cherry-pick`
- `gt restack`
- `gt move`

It is especially useful when conflicts involve:

- deleted-vs-modified files
- renamed or moved files
- extracted functions or shared modules
- generated files or lockfiles
- import fallout after structural changes

## What it does

The skill helps the agent:

- inspect the paused conflict state and choose the right continue command
- identify whether the conflict is textual or structural
- merge valid child-branch intent into the parent branch's newer structure
- escalate only when repository evidence does not settle product intent
- verify the smallest useful surface before continuing

## Notes

- If Graphite owns the paused operation, use `gt continue`.
- If parent changes force removal or override of child behavior, the user should be told exactly what changed and why.
