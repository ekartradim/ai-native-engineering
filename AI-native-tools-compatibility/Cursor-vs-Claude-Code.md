# Cursor vs Claude Code — Component Comparison

> Quick reference for the main building blocks of both tools.  
> Useful when porting workflows between them or understanding why something works in one but not the other.  
> Based on live docs — Cursor 2.4+, Claude Code 2.1.x, April 2026.

---

## At a glance

| Component | Cursor | Claude Code |
|-----------|--------|-------------|
| Persistent instructions | Rules (`.cursor/rules/`) + `AGENTS.md` | `CLAUDE.md` + `.claude/rules/` |
| Cross-session memory | No built-in auto-memory | Auto memory (`MEMORY.md`, written by Claude) |
| Skills | `SKILL.md` files, context-injected | `SKILL.md` files, context-injected |
| Subagents | `.cursor/agents/*.md`, flat file-based | `.claude/agents/*.md`, namespace-aware |
| Agent dispatch | Strict enum of built-in types + plain-name custom agents | Any registered agent including namespaced plugin agents |
| Nested subagents | ⚠️ Limited — see Subagents section | ❌ [Blocked in product docs](https://docs.anthropic.com/en/docs/claude-code/sub-agents) |
| Hooks | `hooks.json`, shell commands + prompt hooks | `hooks` in settings, shell commands + HTTP + prompt hooks |
| Plugins | Not applicable (no plugin system) | `.claude/plugins/` — installs skills, agents, MCP config |
| Rules/instructions scope | Project, User, Team (paid plans) | Project, User, Org (managed policy) |
| MCP | `~/.cursor/mcp.json` or `.cursor/mcp.json` | `.mcp.json` or `~/.claude/mcp.json` |

---

## Skills

Skills package reusable workflows and domain knowledge. They load into the main agent's context — no separate window.

| | Cursor | Claude Code |
|--|--------|-------------|
| File format | `SKILL.md` with YAML frontmatter | Same |
| Locations | `.cursor/skills/`, `~/.cursor/skills/`, also loads from `.claude/skills/`, `.codex/skills/` as fallback | `.claude/skills/`, `~/.claude/skills/` |
| Invoked by | `/skill-name` or auto-applied by agent based on context | `/skill-name` or auto-applied by agent |
| Disable auto-apply | `disable-model-invocation: true` in frontmatter | Not a frontmatter option — always context-eligible |
| Can include scripts | Yes (`scripts/`, `references/`, `assets/` subdirs) | Yes (same structure) |
| Can spawn subagents | No — skill is context only. Main agent decides to delegate | Workflow text often uses `Task(...)` / `Agent(...)` — the model still emits the tool call; not a hard-coded bypass of the LLM |
| Install from GitHub | Yes, via Settings → Rules → Remote Rule | Via plugin system or manual clone |

**Key difference:** In Claude Code, plugin skills are written so the model reliably emits **`Agent` tool calls** (from 2.1.63 onward; `Task` is an alias). In Cursor, a skill can only **instruct** the main agent in natural language; the model then decides whether to call the Task tool — there is no separate non-LLM API to spawn subagents from a skill file.

**Docs:** [Cursor Skills](https://cursor.com/docs/context/skills) · [Claude Code Skills](https://docs.anthropic.com/en/skills)

---

## Memory / Persistent instructions

How each tool carries knowledge across sessions.

| | Cursor | Claude Code |
|--|--------|-------------|
| Primary file | `AGENTS.md` (simple) or `.cursor/rules/*.mdc` (structured) | `CLAUDE.md` |
| User-level file | `~/.cursor/rules/` or User Rules in settings | `~/.claude/CLAUDE.md` |
| Org / policy level | Team Rules (dashboard, paid plans) | Managed policy: `/Library/Application Support/ClaudeCode/CLAUDE.md` |
| Scoped rules | Glob patterns on rule files | `.claude/rules/` with `paths:` frontmatter |
| Auto memory | ❌ None | ✅ Claude writes `MEMORY.md` itself; first 200 lines loaded every session |
| Session memory | Context window only — no persistence | Context window + auto memory file |
| `@import` syntax | ❌ | ✅ `@path/to/file` in CLAUDE.md |
| Cross-tool | Cursor reads `AGENTS.md`; Claude Code reads `CLAUDE.md`. To share: import one from the other | Claude Code ignores `AGENTS.md` unless imported via `@AGENTS.md` in CLAUDE.md |

**Scopes compared:**

| Scope | Cursor | Claude Code |
|-------|--------|-------------|
| Current session only | User Rules in Settings | Conversation context |
| Current project | `.cursor/rules/` or `AGENTS.md` | `CLAUDE.md` or `.claude/rules/` |
| All projects (personal) | `~/.cursor/rules/` | `~/.claude/CLAUDE.md` |
| All users in org | Team Rules (dashboard) | Managed policy CLAUDE.md (`/etc/claude-code/`) |

**Docs:** [Cursor Rules](https://cursor.com/docs/context/rules) · [Claude Code Memory](https://docs.anthropic.com/en/docs/claude-code/memory)

---

## Subagents

Subagents run in a **separate context window** — isolated from the main conversation.

| | Cursor | Claude Code |
|--|--------|-------------|
| File locations | `.cursor/agents/`, `~/.cursor/agents/`, also reads `.claude/agents/`, `.codex/agents/` | `.claude/agents/`, `~/.claude/agents/`, plugin `agents/` |
| Scope priority | `.cursor/` > `.claude/` > `.codex/` | CLI flag > `.claude/agents/` > `~/.claude/agents/` > plugin |
| Session-only agents | ❌ | ✅ `--agents` CLI flag (JSON, not saved to disk) |
| Agent dispatch key | Flat filename — `agent-name.md` only | Flat filename **or** namespace — `plugin:namespace:agent-name` |
| Nested subagents | ⚠️ **Not generally guaranteed** — a subagent may lack the Task tool (`subagent_type` + prompt) depending on **plan**, **model** (e.g. some Cursor Composer routes), and product version. Cursor staff have noted **Max Mode** can matter for nested behaviour on some plans ([forum](https://forum.cursor.com/t/cant-get-async-subagents-nested-subagents-to-work/152573)). Treat nested orchestration as **best-effort**, not a spec you can rely on everywhere. | ❌ **Product rule** — [Anthropic docs](https://docs.anthropic.com/en/docs/claude-code/sub-agents): *"Subagents cannot spawn other subagents"* (delegate from **main** only; use skills or chain from main) |
| Built-in agents | `explore`, `bash`, `browser` | `explore`, `plan`, `general-purpose`, `bash` |
| Tool restriction | `readonly: true` in frontmatter (no writes/shell) | `tools:` allowlist or `disallowedTools:` denylist |
| Model per agent | `model: fast \| inherit \| <model-id>` | `model: haiku \| sonnet \| opus \| inherit \| <model-id>` |
| Subagent memory | ❌ No per-subagent memory | ✅ `memory: user \| project \| local` — own `MEMORY.md` |
| Subagent hooks | ❌ | ✅ `hooks:` field in frontmatter |
| Subagent MCP scope | Inherits parent MCP tools | Inherits + can add/restrict with `mcpServers:` field |
| Permission mode | `readonly: true` only | `permissionMode: default \| acceptEdits \| dontAsk \| bypassPermissions \| plan` |
| Git worktree isolation | ❌ | ✅ `isolation: worktree` — runs in temp git worktree |
| Invoke explicitly | `/agent-name` or natural language | `/agent-name` or natural language |
| Parallel dispatch | ✅ — agent sends multiple Task tool calls in one message | ✅ — multiple `Agent` / `Task` calls in one assistant turn |
| Parallel phrasing tip | Must say **"in parallel in a single message"** — defaults to sequential otherwise | Model-emitted tool calls in one turn run in parallel |

### Task tool and how subagents are invoked (Cursor)

The **parent** agent (main chat) reaches subagents by a **Task-style tool**: a concrete tool call with a **subagent type** (built-in or plain-custom) and a task prompt. That is **not** the same as “NL-only” — once the model decides to call Task, the invocation is a normal tool call. It is still **LLM-gated**: nothing in a `SKILL.md` file directly triggers Task without the model choosing to do so. **Most reliable user steer:** pick the subagent from the slash menu (`/agent-name`) so **which** specialist runs is explicit; the model still composes the task text.

**Key difference on nested subagents:** Claude Code documents a **hard** rule: subagents cannot spawn other subagents. Cursor has **no equivalent single sentence** in the public changelog summary; nested spawning is **reported inconsistently** in the field (some users see no Task tool inside a subagent). Do not assume nested subagent trees work everywhere in Cursor.

**Key difference on namespace dispatch:** Claude Code resolves `compound-engineering:research:repo-research-analyst` via a plugin namespace registry. Cursor only knows flat filenames — that string is rejected unless the file is literally named that. This is the root cause of the CE symlink bridge need.

**Docs:** [Cursor Subagents](https://cursor.com/docs/context/subagents) · [Cursor 2.4 changelog (subagents)](https://cursor.com/changelog/2-4) · [Claude Code Subagents](https://docs.anthropic.com/en/docs/claude-code/sub-agents)

---

## Hooks

Hooks run shell commands (or LLM prompts) automatically at lifecycle events.

| | Cursor | Claude Code |
|--|--------|-------------|
| Config file | `hooks.json` | `hooks` section in `.claude/settings.json` |
| Locations | `~/.cursor/hooks.json` (global), `.cursor/hooks.json` (project) | `~/.claude/settings.json` (user), `.claude/settings.json` (project) |
| Hook types | Shell command, prompt (LLM-evaluated) | Shell command, HTTP endpoint, prompt |
| Claude `.claude/hooks.json` compatibility | ✅ Cursor loads it as fallback | N/A |
| Per-subagent hooks | ❌ | ✅ `hooks:` frontmatter field in agent definition |

**Cursor hook events:**

| Event | When |
|-------|------|
| `sessionStart` / `sessionEnd` | Session open/close |
| `preToolUse` / `postToolUse` / `postToolUseFailure` | Any tool call |
| `subagentStart` / `subagentStop` | Subagent spawned/finished |
| `beforeShellExecution` / `afterShellExecution` | Shell commands |
| `beforeMCPExecution` / `afterMCPExecution` | MCP tool calls |
| `beforeReadFile` / `afterFileEdit` | File access/edits |
| `beforeSubmitPrompt` | Before prompt sent to model |
| `preCompact` | Before context compaction |
| `stop` | Agent finishes responding |
| `beforeTabFileRead` / `afterTabFileEdit` | Tab (inline completions) only |

**Claude Code hook events (selected):**

| Event | When |
|-------|------|
| `SessionStart` / `SessionEnd` | Session lifecycle |
| `PreToolUse` / `PostToolUse` | Any tool call (can block) |
| `SubagentStart` / `SubagentStop` | Subagent lifecycle |
| `UserPromptSubmit` | Before Claude processes your prompt |
| `InstructionsLoaded` | When a CLAUDE.md or rules file loads (useful for debugging) |
| `PreCompact` / `PostCompact` | Context compaction |
| `WorktreeCreate` / `WorktreeRemove` | Git worktree lifecycle |
| `FileChanged` | Watched file changed on disk |
| `TeammateIdle` | Agent teams: teammate going idle |

**Docs:** [Cursor Hooks](https://cursor.com/docs/agent/hooks) · [Claude Code Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks)

---

## Plugins (Claude Code only)

Cursor has no plugin system. Claude Code has a first-class plugin layer.

| Concept | Claude Code |
|---------|-------------|
| Install | `bunx @every-env/compound-plugin install <plugin> [--to cursor]` |
| Cache location | `~/.claude/plugins/cache/<source>/<name>/<version>/` |
| What a plugin can ship | Skills, agents, commands, MCP config, hooks |
| Scope | User-global; `--to cursor` writes a Cursor-compatible layer on top |
| CE specifically | Ships 35 agents, ~20 skills, MCP config — all in `~/.claude/plugins/cache/local-curated/compound-engineering/` |

**Docs:** [Claude Code Plugins](https://docs.anthropic.com/en/plugins)

---

## Rules / Instructions scopes summary

| Scope | Cursor | Claude Code |
|-------|--------|-------------|
| **Session-only** | User Rules in Settings UI | Inline conversation context |
| **Project** | `.cursor/rules/*.mdc` or `AGENTS.md` | `CLAUDE.md` or `.claude/rules/*.md` |
| **User (personal, all projects)** | `~/.cursor/rules/` or Settings → User Rules | `~/.claude/CLAUDE.md` or `~/.claude/rules/` |
| **Team / Org** | Team Rules via dashboard (paid plans) | Managed policy CLAUDE.md at system path |
| **Rule precedence** | Team → Project → User | Org policy → Project → User (more specific wins) |
| **Path-scoped rules** | `globs:` frontmatter in `.mdc` files | `paths:` frontmatter in `.claude/rules/*.md` |
| **Always-on rules** | `alwaysApply: true` | Rules without `paths:` load unconditionally |
| **Explicit-only rules** | `@mention` in chat | Task-specific — use skills instead |

---

## MCP (Model Context Protocol)

| | Cursor | Claude Code |
|--|--------|-------------|
| Config file | `.cursor/mcp.json` (project) or `~/.cursor/mcp.json` (user) | `.mcp.json` (project) or `~/.claude/mcp.json` (user) |
| Per-subagent MCP | ❌ Subagents inherit parent tools | ✅ `mcpServers:` field in subagent frontmatter |
| Tool limit | ~40 tools before accuracy degrades | No documented hard limit |
| Compatibility | Also reads `.claude/mcp.json` as fallback | Reads `.mcp.json` only |

---

## File compatibility matrix

Cursor reads from both `.cursor/` and `.claude/` as fallback. This means CE files in `~/.claude/` work in Cursor without moving them.

| File type | `.cursor/` | `.claude/` loaded by Cursor? | `.cursor/` wins on conflict? |
|-----------|-----------|------------------------------|------------------------------|
| Agents | ✅ Primary | ✅ Yes | ✅ Yes |
| Skills | ✅ Primary | ✅ Yes | ✅ Yes |
| Hooks | ✅ Primary | ✅ Yes | ✅ Yes |
| MCP config | ✅ Primary | ✅ Yes | ✅ Yes |
| Rules / CLAUDE.md | `AGENTS.md` or `.cursor/rules/` | `CLAUDE.md` read by Claude Code only | N/A — different files |

---

**Cursor docs:** [Subagents](https://cursor.com/docs/context/subagents) · [Skills](https://cursor.com/docs/context/skills) · [Rules](https://cursor.com/docs/context/rules) · [Hooks](https://cursor.com/docs/agent/hooks) · [MCP](https://cursor.com/docs/mcp)  
**Claude Code docs:** [Subagents](https://docs.anthropic.com/en/docs/claude-code/sub-agents) · [Memory](https://docs.anthropic.com/en/docs/claude-code/memory) · [Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks) · [Plugins](https://docs.anthropic.com/en/plugins)
