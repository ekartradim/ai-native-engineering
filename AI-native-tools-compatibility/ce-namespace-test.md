---
name: ce-namespace-test
description: Verifies the CE→Cursor symlink bridge is working. Use when you want to confirm compound-engineering agents are callable as plain-named subagents in Cursor.
---

# CE → Cursor Namespace Bridge Test

Validates that compound-engineering agents are accessible in Cursor via the symlink bridge in `~/.cursor/agents/`.

## Background

**The problem:**
Compound-engineering skills call agents with namespace syntax:
```
Task compound-engineering:research:repo-research-analyst(...)
```
Cursor's Task tool only accepts a fixed set of subagent types it knows (built-ins plus plain-named custom agents on disk). Namespace strings are rejected at validation. ([Cursor subagents docs](https://cursor.com/docs/context/subagents))

**The bridge:**
CE agents ship with plain `name:` values in their frontmatter (e.g. `name: repo-research-analyst`). By symlinking them into `~/.cursor/agents/`, Cursor discovers them as plain-named agents. The main agent can then delegate to them via natural language — which is Cursor's supported invocation path.

**Key Cursor architecture facts (from [docs](https://cursor.com/docs/context/subagents) and field reports):**
- Skills inject instructions into the **main agent's context** — no separate window
- Subagents run in a **separate context window** — spawned by the **parent** agent via the Task tool
- **Subagent → subagent** nesting is **not a stable guarantee** (plan/model/tooling); design flows as **main orchestrates, subagents execute**
- Subagents can be invoked via `/name`, natural language, automatic delegation, or an explicit Task tool call from the parent model

**Difference from Claude Code:**
In Claude Code, skills and plugin workflows target **`Agent` / `Task` tool calls** to registered (including namespaced) agents — the model emits those calls. [Anthropic docs](https://docs.anthropic.com/en/docs/claude-code/sub-agents) state subagents **cannot** spawn other subagents. In Cursor, a skill only **instructs** the parent; it does not bypass the model, and nested subagent behaviour is **product-dependent** — see [compatibility doc](./Cursor-vs-Claude-Code.md).

## Parallel Subagent Dispatch — Critical Pattern

**The problem:**
CE skills that call multiple agents (e.g. `ce-review` spawning 6 persona reviewers, `ce-plan` spawning research agents) do so in parallel in Claude Code — all Task calls are emitted in a single message. In Cursor via natural language delegation, the main agent defaults to sequential execution unless explicitly instructed otherwise. Sequential = much slower (e.g. 12 min vs 2 min for 6 reviewers on a large diff).

**The fix — always phrase multi-agent instructions like this in Cursor skills:**

Instead of:
```
Use the correctness-reviewer, then the security-reviewer, then the testing-reviewer.
```

Write:
```
Invoke the following subagents IN PARALLEL in a single message — do not wait for one to finish before starting the next:
- /correctness-reviewer
- /security-reviewer
- /testing-reviewer
```

**Rules for parallel phrasing that reliably trigger Cursor's parallel dispatch:**
1. Use the words **"in parallel"** and **"in a single message"** explicitly — aligns with how Cursor describes spawning multiple subagents in one turn ([subagents](https://cursor.com/docs/context/subagents))
2. List agents as bullets, not as a sequence with words like "then", "next", "after"
3. Avoid sequential connectives ("first... then... finally...") — these signal serial execution to the model
4. You can add: "do not wait for one to complete before starting the others"

**When adapting CE skills for Cursor:**
Any CE skill section that reads like "Task agent-a(...) Task agent-b(...) Task agent-c(...)" (multiple Task calls) should be rewritten as a parallel bullet list with explicit parallel language. Single-agent calls need no change.

## How to Re-Run the Bridge Setup

If agents are missing after a CE plugin update, re-run:

```bash
CE_AGENTS=~/.claude/plugins/cache/local-curated/compound-engineering/2.52.0/agents
TARGET=~/.cursor/agents

for f in "$CE_AGENTS"/research/*.md \
          "$CE_AGENTS"/review/*.md \
          "$CE_AGENTS"/workflow/*.md \
          "$CE_AGENTS"/document-review/*.md \
          "$CE_AGENTS"/design/*.md; do
  name=$(basename "$f")
  dest="$TARGET/$name"
  if [ ! -e "$dest" ]; then ln -s "$f" "$dest"; fi
done
```

Update `2.52.0` to the current CE version when upgrading.

## Test Instructions

When invoked, run this test:

1. Use the `simple-analyzer` subagent to analyze the following code:
   ```javascript
   function add(a, b) { return a + b; }
   ```

2. Report:
   - Whether the subagent was found and invoked successfully
   - The analysis result from the subagent
   - The agent name used

3. Then check: is `repo-research-analyst` listed as an available subagent?
   If yes, the symlink bridge is working for CE research agents too.

4. **Parallel dispatch test** — invoke three subagents in a single message to verify parallel execution works:
   ```
   Invoke the following subagents IN PARALLEL in a single message — do not wait for one to finish before starting the next:
   - /simple-analyzer — analyze async function fetchData(url) { const res = await fetch(url); return res.json(); }
   - /correctness-reviewer — review the same function for logic errors
   - /security-reviewer — review the same function for security vulnerabilities
   ```
   All three should fire simultaneously. Confirm all returned results.

## Live Test Results (April 1, 2026)

Tested in Cursor 2.4+ with Compound Engineering 2.52.0 and the symlink bridge active.

### Test 1 — Single subagent (`simple-analyzer`)

**Result: PASS**

`simple-analyzer` was found and invoked successfully. Analyzed `function add(a, b) { return a + b; }` and correctly identified type coercion risks, missing input validation, and lack of TypeScript types.

### Test 2 — Parallel dispatch (3 agents, single message)

**Result: PASS — all 3 fired simultaneously**

Agents `simple-analyzer`, `correctness-reviewer`, and `security-reviewer` all analyzed `async function fetchData(url)` in parallel. Each returned distinct, role-appropriate findings:

| Agent | Top finding |
|---|---|
| `simple-analyzer` | No `res.ok` check, uncaught rejections, unvalidated URL |
| `correctness-reviewer` | HTTP errors silently treated as success (high severity) |
| `security-reviewer` | SSRF on user-controlled URL (high severity) |

### Bridge Status

- `simple-analyzer` — built-in Cursor subagent, always available
- `correctness-reviewer` — CE review agent, available via symlink bridge ✅
- `security-reviewer` — CE review agent, available via symlink bridge ✅
- `repo-research-analyst` — CE research agent, listed in available subagent enum ✅

**Parallel phrasing works.** Emitting all Task calls in a single message without sequential connectives ("then", "next", "after") reliably triggers Cursor's parallel dispatch.

## References

- [Cursor Subagents](https://cursor.com/docs/context/subagents) — format, invocation, limitations
- [Cursor Skills](https://cursor.com/docs/context/skills) — how skills work, discovery paths
- [CE Cursor spec](https://github.com/EveryInc/compound-engineering-plugin/blob/main/docs/specs/cursor.md) — official CE docs for Cursor target
- [CE-Cursor-Symlink-Bridge-Findings.md](../AI-native-tools-compatibility/CE-Cursor-Symlink-Bridge-Findings.md) — full findings from live testing session
