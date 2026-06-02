# cc-blackbytes

The Claude Code member of the **blackbytes** family (alongside `pi-blackbytes`, `oc-blackbytes`, and `kiro-blackbytes`).

Blackbytes is an opinionated set of specialist subagents and engineering disciplines. This Claude Code edition is deliberately **thin**: it ships only what Claude Code doesn't already do well natively. Claude Code already has a strong default prompt, native `Explore` and general-purpose agents, worktree isolation, skills, and `WebSearch`/`WebFetch` — so porting the full blackbytes stack would mostly reinvent wheels. Instead, cc-blackbytes adds the genuinely additive pieces:

| Component | Type | What it adds |
|---|---|---|
| `oracle` | agent | Skeptical, **read-only**, Opus + high-effort consult for hard reasoning — architecture calls, tricky debugging, security/perf trade-offs. Checks its own assumptions before answering. No native equivalent. |
| `librarian` | agent | Gated **multi-source** external research with strict citation and no-fabrication discipline. Wraps the web/docs tools in research rigor. |
| `routing-discipline` | skill | Cost-aware guidance on when to delegate vs. do it inline — avoids both over- and under-delegation. |
| `verification-honesty` | skill | A gate order (typecheck → lint → test → build) and a contract against fabricating green results. |
| `commit-and-push`, `review-fresh-eyes`, `suggest-innovation`, `update-docs` | commands | Portable workflow prompts ported from the Pi-blackbytes globals — grouped commits, a fresh-eyes self-review, a single high-value proposal, and a docs refresh. |

It intentionally does **not** ship a system-prompt overlay, custom edit/image/search tools, a `general` executor, or an `explore` agent — Claude Code covers those. Code review is left to the native `/code-review` and `/security-review` skills in v1.

## Install

From a local checkout (development):

```bash
claude --plugin-dir /path/to/cc-blackbytes
```

Via marketplace (once hosted on a git remote):

```
/plugin marketplace add <owner>/cc-blackbytes
/plugin install bytes@blackbytes
```

## Usage

The two agents are invoked automatically by Claude when a task matches their description, or explicitly (plugin components are namespaced by the plugin name, `bytes`):

```
@bytes:oracle     Why does this retry loop deadlock under concurrent writes? <paste context>
@bytes:librarian  What's the current recommended way to configure connection pooling in <library> v3?
```

The two skills auto-trigger when relevant, or you can invoke them with `/bytes:routing-discipline` and `/bytes:verification-honesty`.

The commands are slash commands you invoke explicitly:

```
/bytes:commit-and-push       group changed files into logical commits and push
/bytes:review-fresh-eyes     re-read just-written code as a reviewer and fix issues
/bytes:suggest-innovation    propose the single highest-value next addition
/bytes:update-docs           refresh the README and docs to the current state
```

### When to reach for the oracle

- You've tried a fix twice and it still fails — stop patching, consult.
- A design decision will be expensive to reverse.
- You want a second, skeptical opinion on a security or performance change.

The oracle is read-only: it reasons and advises, it does not edit. Give it enough context inline — it returns a single self-contained recommendation with an effort estimate.

### When to reach for the librarian

Only when **all** hold: you need external info, from multiple sources or a current-year authoritative answer, where a single lookup wouldn't be enough. For a single known-URL fetch or one docs query, just do it directly.

## Layout

```
cc-blackbytes/
├── .claude-plugin/
│   ├── plugin.json        # manifest
│   └── marketplace.json   # marketplace catalog entry
├── agents/
│   ├── oracle.md
│   └── librarian.md
├── skills/
│   ├── routing-discipline/SKILL.md
│   └── verification-honesty/SKILL.md
├── commands/
│   ├── commit-and-push.md
│   ├── review-fresh-eyes.md
│   ├── suggest-innovation.md
│   └── update-docs.md
└── docs/plans/
    └── cc-blackbytes-quick.md
```

## Development

Validate the plugin structure:

```bash
claude plugin validate . --strict
```

## License

MIT
