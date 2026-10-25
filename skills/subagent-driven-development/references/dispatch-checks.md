# Dispatch Checks

Part of superpowers:subagent-driven-development. Read it when BASE is not
the ledger's last recorded head, and before dispatching a docs task.

**BASE drift.** Before every dispatch, BASE must equal the previous task's
recorded final head from the ledger (for Task 1, the head setup left).
Unexpected drift means another writer — a concurrent session, another
tool — is on this checkout: STOP and investigate authorship before
dispatching anything. Attribute mystery commits by their content (e.g. a
plan bug present verbatim in a commit whose dispatch had corrected it),
not by commit trailers — legitimate commits lack trailers too. Never
dispatch into a moved tree.

**Docs tasks carry a quote-verification gate in the dispatch:** every
quoted UI string, enum list, or count the doc asserts gets one grep
against the component or schema that renders or defines it, at write
time. Stale quotes are otherwise caught one stage later each — an
advisor pass, a task review, the final whole-branch review — every stage
more expensive than the grep.
