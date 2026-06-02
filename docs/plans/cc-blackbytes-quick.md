# cc-blackbytes — Quick Plan

**Status**: Active
**Variant**: lightweight (greenfield repo, but de-facto spec'd by 3 sibling ports + capability analysis)
**Date**: 2026-06-02

**Scope**: A Claude Code marketplace plugin — the fourth member of the `blackbytes` family (after `pi-`, `oc-`, `kiro-`). Ships only what Claude Code lacks natively: two specialist subagents (`oracle`, `librarian`) and two discipline skills (`routing-discipline`, `verification-honesty`).

**Out of scope** (deliberately skipped — Claude Code owns this substrate):
- Bytes v2 system-prompt overlay (CC has a strong tuned default prompt; an overlay would fight it)
- `hashline_edit`, `look_at`, `ast_search/replace`, web/docs HTTP clients (native `Edit`/`Read`/`WebSearch`/`WebFetch` + context7 MCP cover these)
- `explore` and `general` agents (CC ships native Explore + general-purpose + `isolation: worktree` + `/workflows`)
- nested-session progress UI, model-family prompt variants, `/setup-models` wizard (CC-owned UI; CC is Claude-only)
- No `reviewer` agent in v1 — CC session already has `/code-review` + `/security-review` skills. Revisit if they prove insufficient.

**Estimate**: <1 day (mostly prompt porting + manifest).

## Approach
- Port the **oracle** persona from `pi-blackbytes/src/sub-agents/oracle.ts`: read-only, Opus, high effort, the long-context-handling + high-risk self-check guardrails, effort-tag output contract. Strip Pi-specific tool names; restrict to CC read tools via `tools:` allowlist.
- Port the **librarian** persona from `pi-blackbytes/src/sub-agents/librarian.ts`: the strict 3-condition gate (external + multi-source + direct-tools-insufficient), citation/no-fabrication policy, external-content-safety, request-classification. Map tools to `WebSearch`, `WebFetch`, `Read`, `Grep`, `Glob` — a web-only research surface (no MCP tool in the allowlist, so the prompt must not assume one).
- Distill the Bytes overlay's **delegation routing matrix** (cost/useWhen/avoidWhen) into a `routing-discipline` skill — the cost-aware "don't over-delegate" discipline, adapted to CC's native + plugin agents.
- Distill the Bytes **verification contract** (typecheck→lint→test→build, never fabricate green) into a `verification-honesty` skill.
- Manifest: `.claude-plugin/plugin.json` (`name` required; add version/description/author/keywords) + a `.claude-plugin/marketplace.json` so it's installable. Agents in `agents/`, skills in `skills/<name>/SKILL.md` (auto-discovered).

## Tasks
- [ ] T1: `.claude-plugin/plugin.json` + `marketplace.json`
- [ ] T2: `agents/oracle.md`
- [ ] T3: `agents/librarian.md`
- [ ] T4: `skills/routing-discipline/SKILL.md`
- [ ] T5: `skills/verification-honesty/SKILL.md`
- [ ] T6: `README.md`
- [ ] T7: validate (`claude plugin validate ./cc-blackbytes --strict` + plugin-validator agent)

## Verify
- `claude plugin validate . --strict` passes (no warnings).
- Agent frontmatter uses only CC-supported fields (no `hooks`/`mcpServers`/`permissionMode` in plugin agents).
- Manual smoke: enable via `--plugin-dir`, confirm `oracle`/`librarian` appear in `/agents` and the two skills in `/`; fire one oracle consult.

## Notes / risks
- R1: Overlap with native review skills → mitigated by omitting `reviewer` in v1.
- R2: Duplication with `kiro-blackbytes` (also Claude-targeted) → justified only because CC plugin can leverage worktree isolation / workflows / skills Kiro lacks; keep personas thin and CC-idiomatic rather than copy-paste.
- R3: librarian's `tools:` allowlist grants no MCP tool, so its prompt must describe a web-only surface (WebSearch/WebFetch + local Read/Grep/Glob) and must not instruct it to use context7 or any tool it can't actually call.
