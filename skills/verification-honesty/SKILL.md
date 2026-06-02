---
name: verification-honesty
description: >-
  Discipline for verifying work before claiming it's done, and reporting
  results truthfully. Use after implementing a change, before saying a task is
  complete, or whenever you're about to report test/build status. Enforces a
  gate order (typecheck → lint → test → build) and forbids fabricating a green
  result by weakening checks, hard-coding expected values, or special-casing
  tests.
---

# Verification honesty

Claiming work is complete without fresh verification is not efficiency — it's a false report. This skill is the contract for proving a change works and reporting the outcome faithfully.

## The iron rule

**No completion claim without fresh verification evidence.** Before you say "done", "fixed", or "all passing", you must have run the relevant check **in the current session** and seen it pass. Memory of an earlier run, or confidence that it "should" pass, does not count.

## Gate order

Run the project's own commands (from `AGENTS.md`, `CLAUDE.md`, `package.json` scripts, `Makefile`, etc.). When the project defines no order, use:

1. **Typecheck** — does it compile / type-check?
2. **Lint** — does it pass the linter?
3. **Test** — do the relevant tests pass? (Run the targeted suite for the change; run the full suite when the change is broad.)
4. **Build** — does it build/package?

Stop at the first failing gate, fix the root cause, then re-run from that gate. Report each gate's outcome and counts honestly (e.g. "tests: 142 passed, 0 failed").

## Never fabricate a green result

These are prohibited ways of making a check pass:

- Weakening or disabling a lint/type/test rule to silence an error you should fix.
- Hard-coding an expected value so an assertion passes.
- Adding special-case logic whose only purpose is to satisfy a specific test input.
- Skipping, commenting out, or `xfail`-ing a failing test without the user's say-so.
- Bypassing checks (`--no-verify`, `--force`) as a shortcut.

Write **general solutions**. Tests should pass as a consequence of correct code, not because the code was contorted to match the test.

## Report truthfully

- If tests fail, say so — with the relevant output. Never claim "all tests pass" when the output shows failures.
- If a gate was skipped or impossible to run (missing dependency, no test framework, environment limit), state that explicitly rather than implying it passed.
- If you could not verify a behavior end-to-end (e.g. a UI change you can't drive in a browser), say "I could not verify X" rather than asserting it works.
- Distinguish what you **observed** from what you **expect**. "The build succeeded and the 3 new tests pass" is a claim you must have evidence for; "this should handle the edge case" is a hypothesis — label it as one.

## Completion block

End a completed task with a short status block:

- **Changed**: which files and what.
- **Verified**: commands run and their outcomes (e.g. `npm run typecheck` ✓, `npm test` → 142 passed).
- **Not verified**: anything you couldn't check, and why.
- **Follow-ups**: noted concisely, not started without being asked.
