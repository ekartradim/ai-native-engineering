# Compound Engineering in Cursor

> Using the CE plugin with Cursor IDE. Tested April 2026 — Cursor 2.4+, CE 2.52.0.

---

## The problem in one sentence

CE skills load automatically in Cursor. The problem is **agent dispatch**: CE skills call agents using namespace syntax (`compound-engineering:research:repo-research-analyst`) that Cursor's Task tool rejects. The fix is a one-time symlink bridge that puts all CE agents where Cursor can find them by plain name.

---

## Setup

### Step 1 — Get CE skills visible in Cursor

CE is a **Claude Code plugin** — it always installs into `~/.claude/`, never directly into `~/.cursor/`. Cursor picks up skills from `~/.claude/` automatically as a fallback, so if CE is already installed for Claude Code, your skills are already available in Cursor too.

**Check if CE is already installed:**

```bash
ls ~/.claude/plugins/cache/local-curated/compound-engineering/
# Should show a version folder, e.g. 2.52.0
```

If the folder exists, skip to Step 2.

**If CE is not installed yet**, run the Cursor-targeted installer — this installs CE into `~/.claude/` and also writes a `~/.cursor/` config layer on top:

```bash
bunx @every-env/compound-plugin install compound-engineering --to cursor
```

Either way, Step 2 is required — the installer never handles the agent bridge.

### Step 2 — Symlink bridge

```bash
CE_AGENTS=~/.claude/plugins/cache/local-curated/compound-engineering/2.52.0/agents
TARGET=~/.cursor/agents

mkdir -p "$TARGET"

for f in "$CE_AGENTS"/research/*.md \
          "$CE_AGENTS"/review/*.md \
          "$CE_AGENTS"/workflow/*.md \
          "$CE_AGENTS"/document-review/*.md \
          "$CE_AGENTS"/design/*.md; do
  name=$(basename "$f")
  dest="$TARGET/$name"
  if [ ! -e "$dest" ]; then
    ln -s "$f" "$dest"
    echo "linked: $name"
  fi
done
```

Symlinks (not copies) — no duplication, one source of truth, auto-follows CE source files. When CE upgrades, update the version in `CE_AGENTS` and re-run.

### Step 3 — Verify

In Cursor chat, type `/ce-namespace-test`. This skill runs two live checks:

1. Invokes `simple-analyzer` on a JS snippet — basic dispatch works
2. Invokes `simple-analyzer` + `correctness-reviewer` + `security-reviewer` **in parallel** — bridge active, parallel dispatch works

All three should fire simultaneously. If a CE agent is missing, re-run Step 2.

---

## What works after setup


| Skill             | Status        | Notes                                           |
| ----------------- | ------------- | ----------------------------------------------- |
| `ce-brainstorm`   | ✅ Full        | No agent dispatch needed                        |
| `ce-plan`         | ✅ With bridge | Research agents dispatched via natural language |
| `ce-work`         | ✅ With bridge | Units executed inline, no nested subagents      |
| `ce-review`       | ✅ With bridge | Persona agents run in parallel                  |
| `document-review` | ✅ With bridge | Same as ce-review                               |
| `deepen-plan`     | ✅ With bridge | Research agents dispatched via natural language |
| `frontend-design` | ✅ Full        | No agent dispatch needed                        |
| `feature-video`   | ✅ Full        | No agent dispatch needed                        |
| `agent-browser`   | ✅ Full        | MCP integration, no agents                      |


**What you lose vs Claude Code:** namespace dispatch for plugin agents, and the same multi-hop tooling guarantees (neither product really supports **subagent → subagent** chains as a stable primitive; Claude Code states that explicitly). Everything else works.

---

## Troubleshooting

**CE agent missing after `/ce-namespace-test`** — Re-run the symlink script. If CE was upgraded, update the version string in `CE_AGENTS`.

**Skill not found (`/ce-plan`, etc.)** — Restart Cursor, then check `~/.cursor/skills/` for CE skill folders. Re-run Step 1 if missing.

**Skill appears twice in `/` picker** — Duplicate `name:` across `~/.cursor/skills/` and `~/.claude/skills/`. Remove the `~/.claude/` copy. Rule: one skill = one folder = one `name:`.

**MCP tool errors / degraded accuracy** — Cursor degrades above ~40 active MCP tools. Disable unused servers in `~/.cursor/mcp.json`.

---

## How it works

### Loading order

Cursor scans multiple directories at startup. `.cursor/` always wins over `.claude/` when the same name appears in both — so you never need to move CE files from `~/.claude/` to `~/.cursor/` for skills to work. The gap is only agents.


| Type           | Priority 1 (wins)      | Priority 2 (fallback)  |
| -------------- | ---------------------- | ---------------------- |
| User agents    | `~/.cursor/agents/`    | `~/.claude/agents/`    |
| User skills    | `~/.cursor/skills/`    | `~/.claude/skills/`    |
| Project agents | `.cursor/agents/`      | `.claude/agents/`      |
| Project skills | `.cursor/skills/`      | `.claude/skills/`      |
| Hooks          | `~/.cursor/hooks.json` | `~/.claude/hooks.json` |


### Skills vs subagents

Two separate Cursor primitives that CE conflates:


|                  | Skill                                                                    | Subagent                                            |
| ---------------- | ------------------------------------------------------------------------ | --------------------------------------------------- |
| File             | `~/.cursor/skills/*/SKILL.md`                                            | `~/.cursor/agents/*.md`                             |
| Invoked by       | `/skill-name` or auto-context                                            | `/agent-name` or natural language                   |
| Execution        | Instructions injected into the main agent's context — no separate window | Spawns a separate context window, returns a summary |
| Can spawn others (subagent → subagent) | No                                                                | ⚠️ **Cursor:** not reliable — depends on plan/model/tooling ([e.g. forum](https://forum.cursor.com/t/cant-get-async-subagents-nested-subagents-to-work/152573)); assume **main agent → subagent** only for portable setups |


### Why namespace syntax fails

CE skills dispatch agents like this:

```
Task compound-engineering:research:repo-research-analyst(...)
```

In **Claude Code**, this resolves via a namespace registry (`platform → namespace → agent-name`). In **Cursor**, the Task tool only accepts types Cursor knows: built-in subagents plus **plain-named** custom agents backed by files in `.cursor/agents/` (or fallbacks). Namespace strings such as `compound-engineering:research:...` are rejected at validation. Cursor then looks for a file literally named `compound-engineering:research:repo-research-analyst.md`, which does not exist.

The symlink bridge fixes this by giving all CE agents plain filenames in `~/.cursor/agents/`. The main agent reads skill instructions and delegates via natural language (`/repo-research-analyst`) — Cursor's supported path.


| Capability                          | Claude Code  | Cursor (with bridge)                |
| ----------------------------------- | ------------ | ----------------------------------- |
| Namespace syntax                    | ✅            | ❌ rejected at validation            |
| Plain-name invocation `/agent-name` | ✅            | ✅                                   |
| Skill → agent delegation            | Model emits `Agent`/`Task` calls | NL instruction → model may call Task |
| Nested subagents                    | ❌ main-only per docs | ⚠️ not reliable — assume main→subagent only for portable workflows |
| Parallel dispatch                   | ✅            | ✅ with correct phrasing             |
| Dynamic inline agent creation       | ✅            | ❌ requires a pre-defined `.md` file |


### Parallel dispatch

CE skills in Claude Code emit multiple Task calls in one message — they run in parallel. In Cursor, the main agent defaults to **sequential** unless told otherwise. This matters: `ce-review` with 6 persona reviewers takes ~2 min in parallel vs ~12 min sequential.

**Use this pattern in Cursor skills:**

```
Invoke the following subagents IN PARALLEL in a single message —
do not wait for one to finish before starting the next:
- /correctness-reviewer
- /security-reviewer
- /testing-reviewer
```

Triggers: **"in parallel"**, **"in a single message"**, **"simultaneously"**  
Avoid: "then", "next", "after", "first... then... finally" — these signal serial execution.

---

## Agent file format

```markdown
---
name: my-agent
description: What this agent does. Use when X, Y, Z.
model: fast        # or: inherit, claude-4-sonnet, etc.
readonly: false    # true = no file writes
---

You are a [specialist]. When invoked:
1. Do this
2. Return findings with...
```

`name:` must match the filename (without `.md`). Mismatches cause duplicates in the `/` picker.

---

## What gets symlinked (35 agents, CE 2.52.0)


| Group              | Agents                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `research/`        | `best-practices-researcher`, `framework-docs-researcher`, `git-history-analyzer`, `learnings-researcher`, `repo-research-analyst`                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `review/`          | `agent-native-reviewer`, `api-contract-reviewer`, `architecture-strategist`, `code-simplicity-reviewer`, `correctness-reviewer`, `data-integrity-guardian`, `data-migration-expert`, `data-migrations-reviewer`, `deployment-verification-agent`, `julik-frontend-races-reviewer`, `kieran-python-reviewer`, `kieran-typescript-reviewer`, `maintainability-reviewer`, `pattern-recognition-specialist`, `performance-oracle`, `performance-reviewer`, `reliability-reviewer`, `schema-drift-detector`, `security-reviewer`, `security-sentinel`, `testing-reviewer` |
| `workflow/`        | `bug-reproduction-validator`, `lint`, `pr-comment-resolver`, `spec-flow-analyzer`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `document-review/` | `coherence-reviewer`, `feasibility-reviewer`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `design/`          | `design-implementation-reviewer`, `design-iterator`, `figma-design-sync`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |


---

---

## Also in this folder

| File | What it is |
|------|------------|
| [Cursor-vs-Claude-Code.md](./Cursor-vs-Claude-Code.md) | Side-by-side comparison of skills, memory, agents, hooks, rules, scopes, and MCP between both tools |
| [ce-namespace-test.md](./ce-namespace-test.md) | The `ce-namespace-test` skill file — copy to `~/.cursor/skills/ce-namespace-test/SKILL.md` to install |

---

**Cursor docs:** [Subagents](https://cursor.com/docs/context/subagents) · [Skills](https://cursor.com/docs/context/skills) · [MCP](https://cursor.com/docs/mcp) · [CE Cursor spec](https://github.com/EveryInc/compound-engineering-plugin/blob/main/docs/specs/cursor.md)