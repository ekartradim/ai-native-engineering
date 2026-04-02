# Superpowers vs Compound Engineering — Cursor Portability Analysis

> Is Superpowers easier to move from Claude Code to Cursor than Compound Engineering?  
> Short answer: **yes, significantly** — but with one important caveat on its most powerful skill.  
> Based on live repos — Superpowers v5.0.7, CE 2.52.0, Cursor 2.4+, April 2026.

---

## What each plugin actually is

| | Superpowers | Compound Engineering (CE) |
|--|-------------|---------------------------|
| **Core idea** | A development *workflow* — brainstorm → plan → implement → review → ship | A *toolbox* of specialist agents and skills for any task |
| **Skills count** | ~15 workflow skills | ~20 skills |
| **Agents count** | 3 inline subagent roles (implementer, spec-reviewer, quality-reviewer) dispatched from main agent | 35 named specialist agents |
| **Philosophy** | Opinionated end-to-end process (TDD, YAGNI, DRY) | Composable — use what you need |
| **Install** | `/add-plugin superpowers` in Cursor chat | `bunx @every-env/compound-plugin install compound-engineering --to cursor` + symlink bridge |
| **Target user** | Anyone who wants a structured coding workflow | Teams wanting deep specialist review and research |

---

## Why Superpowers is easier to port to Cursor

### 1. Native Cursor support — no bridge required

Superpowers has a **Cursor plugin marketplace entry**. You install it with one command in Cursor chat:

```
/add-plugin superpowers
```

CE has no native Cursor plugin. It installs into `~/.claude/` (Claude Code's home), and you need a manual symlink bridge to make its 35 agents reachable by Cursor's flat-file agent resolver. That's a one-time setup, but it's friction — and it breaks silently when CE upgrades versions.

### 2. Skills-first, not agents-first

Most of Superpowers' value lives in **skills** (SKILL.md files), not subagents. Skills load directly into the main agent's context. Cursor handles this natively — no namespace resolution, no bridge, no flat-file naming conflicts.

CE's value is split: skills work fine in Cursor, but the specialist agents (the 35 reviewers, researchers, and workflow agents) require the bridge because CE dispatches them via namespace syntax (`compound-engineering:research:repo-research-analyst`) that Cursor rejects.

### 3. Subagent use is contained and predictable

Superpowers does use subagents — the `subagent-driven-development` skill dispatches an implementer, a spec reviewer, and a code quality reviewer per task. But these are **generic roles described inline in the skill**, not named agents with their own `.md` files. The main agent creates the subagent prompt on the fly.

This means there are no agent files to symlink, no naming conflicts, and no namespace issues. The skill just tells Cursor's main agent to "dispatch a subagent with these instructions" — which Cursor supports.

### 4. Zero-dependency design

Superpowers is explicitly zero-dependency by policy (their CLAUDE.md rejects PRs that add external dependencies). No MCP servers required, no external tools, no API keys. It works with whatever model and tools Cursor already has.

CE ships MCP configuration alongside the agents and skills. More powerful, but more setup surface area.

---

## The one real limitation in Cursor

The `subagent-driven-development` skill dispatches **multiple subagents sequentially** (implementer → spec reviewer → quality reviewer, per task). In Claude Code, this is programmatic and automatic. In Cursor, the main agent defaults to sequential execution unless explicitly told otherwise.

For Superpowers this matters less than for CE's parallel reviewer pattern, because `subagent-driven-development` is intentionally sequential by design — you want spec compliance before code quality review. The skill's process flow is:

```
implement → spec review → (fix if needed) → quality review → (fix if needed) → next task
```

This works fine in Cursor. The only place you might feel friction is if you want to run multiple implementation tasks in parallel — Superpowers explicitly warns against this ("Never dispatch multiple implementation subagents in parallel — conflicts").

---

## Skill-by-skill Cursor compatibility

| Skill | Cursor compatible? | Notes |
|-------|--------------------|-------|
| `brainstorming` | ✅ Full | Pure context injection, no agents |
| `writing-plans` | ✅ Full | Pure context injection |
| `executing-plans` | ✅ Full | Batch execution with human checkpoints |
| `subagent-driven-development` | ✅ With awareness | Inline subagent roles work; sequential by design |
| `test-driven-development` | ✅ Full | Process instructions only |
| `systematic-debugging` | ✅ Full | Process instructions only |
| `verification-before-completion` | ✅ Full | Process instructions only |
| `requesting-code-review` | ✅ Full | Checklist skill |
| `receiving-code-review` | ✅ Full | Process instructions only |
| `using-git-worktrees` | ✅ Full | Shell commands, no agent dispatch |
| `finishing-a-development-branch` | ✅ Full | Decision workflow, no agent dispatch |
| `dispatching-parallel-agents` | ⚠️ Partial | Works, but Cursor needs explicit "in parallel in a single message" phrasing |
| `writing-skills` | ✅ Full | Meta skill for creating new skills |

**Summary: 12 of 13 skills work fully. One needs a phrasing adjustment for parallel dispatch.**

---

## Head-to-head: Superpowers vs CE for Cursor users

| Factor | Superpowers | Compound Engineering |
|--------|-------------|----------------------|
| Install friction | Low — one command | Medium — install + symlink bridge |
| Skills work in Cursor | ✅ All | ✅ All |
| Agents work in Cursor | ✅ (inline roles, no files) | ✅ With bridge (35 named agents) |
| Breaks on upgrade | No | Yes — version string in bridge script needs updating |
| Specialist depth | Low — 3 generic roles | High — 35 domain-specific reviewers |
| Workflow structure | High — opinionated end-to-end | Low — pick what you need |
| Good for occasional Cursor users | ✅ Yes | ⚠️ Requires more setup investment |
| Good for power users | ⚠️ May feel constraining | ✅ Yes |

---

## Who should use which

**Use Superpowers if:**
- You use Cursor occasionally and want a structured workflow without setup overhead
- You want the agent to slow you down before coding (brainstorm → plan → implement is the whole point)
- You care about TDD, YAGNI, and systematic debugging as defaults
- You don't want to maintain a symlink bridge or track CE version upgrades

**Use Compound Engineering if:**
- You want deep specialist review (security, performance, TypeScript quality, data migrations, etc.)
- You're already using Claude Code and want the same agents in Cursor
- You're willing to do the one-time bridge setup and re-run it on CE upgrades
- You want composable tools rather than an opinionated workflow

**Use both if:**
- You want Superpowers' workflow structure *and* CE's specialist reviewers
- They don't conflict — Superpowers handles the dev loop, CE handles deep review passes

---

## Installation comparison

**Superpowers in Cursor:**
```
/add-plugin superpowers
```
Done. Verify by asking the agent to help plan a feature — it should trigger `brainstorming` automatically.

**CE in Cursor:**
```bash
# 1. Install
bunx @every-env/compound-plugin install compound-engineering --to cursor

# 2. Symlink bridge (run once, re-run on CE version upgrade)
CE_AGENTS=~/.claude/plugins/cache/local-curated/compound-engineering/2.52.0/agents
TARGET=~/.cursor/agents
mkdir -p "$TARGET"
for f in "$CE_AGENTS"/research/*.md "$CE_AGENTS"/review/*.md \
          "$CE_AGENTS"/workflow/*.md "$CE_AGENTS"/document-review/*.md \
          "$CE_AGENTS"/design/*.md; do
  name=$(basename "$f")
  [ ! -e "$TARGET/$name" ] && ln -s "$f" "$TARGET/$name"
done

# 3. Verify
# In Cursor chat: /ce-namespace-test
```

---

## Bottom line

Superpowers is the right choice for **Cursor-first or occasional Cursor users**. It installs in one command, all skills work natively, and its subagent use is contained enough that Cursor's limitations don't bite. The workflow it enforces (brainstorm → plan → implement → review → ship) is valuable precisely because it's opinionated — you don't need to know which CE skill to reach for.

CE is the right choice when you need **specialist depth** — a TypeScript reviewer that thinks like Kieran, a security audit that checks OWASP, a data migration expert that knows your schema. That depth requires the bridge, but it's a one-time cost.

If you're not an active Cursor power user and want something that just works, start with Superpowers. If you outgrow it, add CE on top.

---

**References:**  
[Superpowers GitHub](https://github.com/obra/superpowers) · [CE Cursor setup](./README.md) · [Cursor vs Claude Code components](./Cursor-vs-Claude-Code.md)
