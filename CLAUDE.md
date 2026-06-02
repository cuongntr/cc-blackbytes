# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`cc-blackbytes` is a **Claude Code plugin** (a marketplace package), not an application. There is no compiled artifact, runtime, or test suite — the "source" is Markdown prompt files plus two JSON manifests. The deliverable is correct prompt content and valid manifest structure.

It is the Claude Code member of the **blackbytes** family (siblings: `pi-blackbytes`, `oc-blackbytes`, `kiro-blackbytes`). The personas here are ports of the Pi-blackbytes originals, re-homed onto Claude Code's native primitives.

## Validate

```bash
claude plugin validate . --strict
```

This is the primary check — it must pass with no warnings. There is no `npm`/`make`/lint/test step; the manifests are validated against the schemas referenced in their `$schema` fields.

## Architecture & the core design constraint

The guiding principle is **thinness: ship only what Claude Code lacks natively.** This is the most important thing to understand before adding anything, because the obvious additions are deliberately excluded.

Claude Code already provides a strong default system prompt, native `Explore` + general-purpose agents, `isolation: worktree`, `/workflows`, skills, `WebSearch`/`WebFetch`, native `Edit`/`Read`, the context7 MCP for docs, and `/code-review` + `/security-review`. So this plugin does **not** ship a system-prompt overlay, custom edit/search/image tools, an `explore` or `general` executor agent, or a `reviewer` agent. Before adding a component, confirm it isn't already covered by a native CC capability — if CC does it, porting it just fights the native behavior.

Four core components, each justified only because it's genuinely additive:

- **`agents/oracle.md`** — read-only deep-reasoning consult (`tools: Read, Grep, Glob`, `model: opus`, `effort: high`). The read-only restriction is load-bearing: the prompt repeatedly tells it to hand work back rather than work around the lack of edit/exec tools. No native equivalent.
- **`agents/librarian.md`** — gated multi-source external research (`tools: Read, Grep, Glob, WebSearch, WebFetch`). Its allowlist grants **no MCP tool**, so the prompt describes a web-only surface (WebSearch/WebFetch) and must never instruct it to call context7 or any tool absent from that allowlist — a prior commit (`eacdba6`) existed specifically to remove context7 references it couldn't call. Keep this invariant.
- **`skills/routing-discipline/SKILL.md`** — when to delegate vs. do inline. Its routing matrix references the *other* components by name (Explore, oracle, librarian, general-purpose); keep it in sync if components change.
- **`skills/verification-honesty/SKILL.md`** — the typecheck → lint → test → build gate order and a no-fabricated-green contract.

`commands/*.md` adds slash commands (`/commit-and-push`, `/review-fresh-eyes`, `/suggest-innovation`, `/update-docs`) — portable workflow prompts ported from the Pi-blackbytes globals. Each is a Markdown file with a `description` frontmatter field and a prompt body; the filename becomes the command name. These are the only stateful workflow Pi globals worth porting — Pi's `setup-models`/`blackbytes-status` were TS runtime commands and are deliberately omitted (CC-owned UI / Pi-specific).

The agents auto-trigger on description match (or `@bytes:oracle` / `@bytes:librarian`); the skills auto-trigger or run via `/bytes:routing-discipline` and `/bytes:verification-honesty`. Agents, skills, and commands are auto-discovered from `agents/`, `skills/<name>/SKILL.md`, and `commands/*.md` — no manifest registration needed.

**Naming**: the plugin's `name` field is `bytes` (short, kebab-case), which namespaces every component — commands/agents/skills surface as `/bytes:*` and `bytes:oracle`, and the install ref is `bytes@blackbytes`. `displayName` is `"Blackbytes"` (shown in the `/plugin` picker only; not used for namespacing). The repo/directory is still named `cc-blackbytes` and the marketplace is `blackbytes` — don't conflate the three. If you change `name`, it's a breaking change for the install ref and every `/<name>:command`, and the plugin-entry `name` in `marketplace.json` must match it.

## Conventions when editing

- **Agent/skill frontmatter**: agents use `name`, `description`, optionally `model`/`effort`/`tools`; skills use `name`, `description`. Do **not** add `hooks`, `mcpServers`, or `permissionMode` to plugin agents — those fields aren't supported in plugin-supplied agents and will fail strict validation.
- **`description` fields are the routing logic.** Claude selects an agent/skill from its description, so these are precision-engineered (note oracle's and librarian's explicit "use ONLY when…" / "DO NOT use for…" gates). Edit them as carefully as code — loosening a gate changes when the component fires.
- **The two manifests must agree.** `.claude-plugin/plugin.json` (the manifest) and `.claude-plugin/marketplace.json` (the catalog entry that lists this plugin via `source: "./"`) both carry a `name` and `description`; keep them consistent with each other and the README table.
- **Keep personas thin and CC-idiomatic.** When porting from a sibling, strip tool names and mechanics that don't exist in Claude Code rather than copy-pasting. A persona that references a tool not in its `tools:` allowlist is a bug (see the context7 lesson above).

## Not part of the plugin

`.beads/` is a gitignored task-tracking scaffold — not part of the plugin. `docs/plans/cc-blackbytes-quick.md` is tracked but isn't loaded by the plugin runtime; it records the original scope decisions and the rationale for what was deliberately omitted. Read it when deciding whether a proposed addition fits the thinness constraint.
