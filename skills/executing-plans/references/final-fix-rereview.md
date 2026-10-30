# Final fix pass: one scoped re-review

When the final fix pass changed any non-test code, dispatch one scoped re-review of the fix diff alone (from the pass's start to its end), on the most capable model, with the findings list it answers. Without a subagent tool, re-read the fix diff yourself against code-reviewer.md and say so in your final message.

Why: a fix's own test answers "addressed" but not "broke nothing the suite misses". On the /lineup close-out build (2026-10-28), the fix copied the reviewer's suggested clause, which hid sessions in a case no test covered; only a re-review caught it.

Fix what the re-review finds in the same pass, by the same TDD rule, and ledger each as `Final: re-review fixed <finding> — <test name> RED→GREEN`. No further re-review; what you decline is a ruling, as above.
