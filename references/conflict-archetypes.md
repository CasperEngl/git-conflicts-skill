# Conflict Archetypes

Use these recipes after identifying the paused operation and the correct continue command.

## 1. Parent Deleted File, Child Modified It

Recognize it when:

- the old path is marked deleted on one side and modified on the other
- the parent branch removed the file entirely or replaced it with a different structure

Default decision:

- keep the deletion if the parent removed the file intentionally or replaced it elsewhere
- do not recreate the file at the old path by default

Preserve child intent when:

- the child change still matters and there is a clear successor file, module, or call site where the behavior now belongs

Escalate when:

- the deletion might represent a product decision rather than a refactor
- there is no clear new home for the child logic

Notify the user when:

- any child behavior from the deleted file was dropped
- child behavior was relocated into a new structure

Stage and verify:

- stage the deleted path and any successor-path edits together
- run the smallest targeted check that exercises the moved behavior or import path

## 2. Parent Renamed Or Moved File, Child Modified Old Path

Recognize it when:

- Git reports rename or modify conflicts
- the child branch edited the old location after the parent moved the file

Default decision:

- keep the parent's new path as canonical
- port valid child edits into the renamed or moved file

Preserve child intent when:

- the child change still applies in the new location without reintroducing old structure

Escalate when:

- the move changed the file's responsibility and it is unclear whether the child behavior still belongs there

Notify the user when:

- some child behavior no longer fits after the move
- the move changes behavior, ownership, or import paths

Stage and verify:

- stage the new path and any import-call-site fixes
- verify with the narrowest test, typecheck, or lint command that covers the moved file

## 3. Function Or Class Moved Or Extracted

Recognize it when:

- both sides edited related logic, but the parent extracted it into another file or utility
- the old implementation still exists only in conflict markers or stale call sites

Default decision:

- keep the extracted or shared location as canonical
- re-home valid child logic into the extracted function, shared module, or new call path

Preserve child intent when:

- the child change still affects the same behavior and can be applied cleanly in the new implementation

Escalate when:

- the parent extraction changed semantics and it is unclear whether the child change should survive

Notify the user when:

- child-only behavior had to be dropped because the new shared abstraction no longer supports it
- the child behavior survived but in a different implementation shape

Stage and verify:

- stage the canonical implementation plus any updated call sites
- run the smallest check that exercises the extracted path

## 4. Both Sides Deleted Code

Recognize it when:

- both sides removed the same path or behavior

Default decision:

- keep the deletion and clean up any leftover references

Preserve child intent when:

- not applicable unless one side actually relocated the behavior instead of deleting it

Escalate when:

- the repository contains conflicting evidence about whether the feature should still exist

Notify the user when:

- the child branch had unique behavior that is now gone

Stage and verify:

- stage the deletion and any reference cleanup
- verify imports, routing, or registrations that previously pointed at the removed code

## 5. Child Deleted Code, Parent Changed It

Recognize it when:

- the child branch removed a file or behavior that the parent later edited

Default decision:

- inspect why the child deleted it
- in merge flows, treat this as a real semantic decision
- in rebase-like flows, prefer the parent's surviving structure unless the child deletion is clearly still intended

Preserve child intent when:

- repository evidence shows the deletion was deliberate and still correct

Escalate when:

- the deletion appears product-driven and the current repository state does not settle it

Notify the user when:

- the deletion did not survive and the parent behavior was kept

Stage and verify:

- stage the kept or deleted path consistently with any callers
- verify the affected behavior directly

## 6. Generated Files And Lockfiles

Recognize it when:

- the conflicted file is generated output, a lockfile, or a snapshot

Default decision:

- prefer regenerating from the authoritative source when practical
- otherwise take the side that matches the chosen dependency or source-of-truth change

Preserve child intent when:

- the child branch's source changes still exist and regeneration reproduces them

Escalate when:

- both sides changed inputs in incompatible ways and regeneration is not straightforward

Notify the user when:

- dependency or generated-output choices caused child branch output to be dropped

Stage and verify:

- stage the regenerated artifact and any source files that justify it
- verify by rerunning the generator or the narrowest dependency/install check

## 7. Import Fallout After Delete Or Rename

Recognize it when:

- the main conflict is resolved, but imports, exports, or registrations still point to removed or renamed paths

Default decision:

- repair the callers to use the canonical surviving path

Preserve child intent when:

- the import change is only mechanical and the child behavior still exists in the new location

Escalate when:

- import fallout reveals deeper behavior changes that the repository does not clarify

Notify the user when:

- import repair accompanies a behavior change from the child branch

Stage and verify:

- stage the caller fixes with the structural resolution
- verify via typecheck, test, or targeted search for stale references

## 8. Graphite-Specific Pause And Continue

Recognize it when:

- `gt restack` or `gt move` paused
- Graphite owns the paused operation

Default decision:

- resolve conflicts using the same structural rules as a rebase
- use `gt continue` when resuming

Preserve child intent when:

- it still belongs in the parent's newer structure

Escalate when:

- the paused state is ambiguous or the Graphite operation is no longer the owner of the rebase state

Notify the user when:

- parent changes overrode child behavior during the restack

Stage and verify:

- stage resolved paths with `git add` or `gt add`
- verify the narrowest affected behavior, then resume with `gt continue`
