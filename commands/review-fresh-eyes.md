---
description: Review recently changed code with fresh eyes and fix obvious issues
---

Carefully read over all of the new code you just wrote, plus any existing code you just modified, with "fresh eyes" — as if reviewing someone else's pull request. Look closely for obvious bugs, errors, broken edge cases, confused logic, leftover scaffolding, and inconsistencies with the rest of the codebase.

Fix anything you uncover. Think hard about each change before applying it, and after fixing, re-verify per the project's gate order (typecheck → lint → test → build) — don't claim it's clean without fresh evidence.
