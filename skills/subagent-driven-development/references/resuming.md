# Resuming From an Existing Ledger

Part of superpowers:subagent-driven-development. This plan's workspace
already has a ledger at `<workspace>/progress.md`, or the plan has
checked-off tasks.

If the ledger's first line names your plan file, tasks with a `Task <N>:
complete` line are DONE — do not re-dispatch them; resume at the first
task without one. A task whose last line is a fix round is mid-loop:
resume the loop at the next round. A ledger whose first line names a
different plan file — or a stray ledger at the old flat path
`.superpowers/sdd/progress.md` — is another plan's progress: leave it in
place and start your own, fresh. Directories under
`.superpowers/sdd/done/` are archives of finished runs — never a resume
source.

The committed plan is the cross-session progress map: a checked-off task
is durable resume state for later sessions, even when the ledger is gone.
If the ledger and the plan's checkboxes disagree, stop and reconcile them
with `git log` before dispatching more work. After compaction, trust the
ledger, the plan checkboxes, and `git log` over your own recollection; if
`git clean -fdx` destroyed the workspace, recover from the committed plan
checkboxes and `git log`.
