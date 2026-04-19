# Claude Code — Subagents Reference

> Source: [code.claude.com/docs/en/sub-agents](https://code.claude.com/docs/en/sub-agents)

A subagent is a specialized assistant running in its **own isolated context window** with a custom system prompt, specific tool access, and independent permissions. Claude delegates tasks to subagents; results come back as a summary, keeping your main conversation clean.

**Use a subagent when** a task would flood your context with output you won't need again (test runs, log parsing, codebase exploration), or when you need enforced tool restrictions. **Use the main conversation** when you need iterative back-and-forth or phases that share context.

---

## Built-In Subagents

| Agent           | Model     | Tools       | Purpose                               |
|:----------------|:----------|:------------|:--------------------------------------|
| `Explore`       | Haiku     | Read-only   | File search, codebase exploration     |
| `Plan`          | Inherited | Read-only   | Research during plan mode             |
| `general-purpose` | Inherited | All       | Multi-step tasks needing writes       |

---

## File Locations & Scopes

| Location              | Scope             | Priority |
|:----------------------|:------------------|:---------|
| Managed settings      | Organization-wide | 1 (highest) |
| `--agents` CLI flag   | Current session   | 2        |
| `.claude/agents/`     | Current project   | 3        |
| `~/.claude/agents/`   | All your projects | 4        |
| Plugin `agents/` dir  | Plugin users      | 5        |

Commit `.claude/agents/` to version control to share project subagents with the team.

---

## Subagent File Format

```markdown
---
name: code-reviewer               # required; lowercase, hyphens
description: >                    # required; Claude uses this to decide when to delegate
  Reviews code for quality and security. Use proactively after code changes.
tools: Read, Grep, Glob, Bash     # allowlist — omit to inherit all
disallowedTools: Write, Edit      # denylist — applied before tools allowlist
model: sonnet                     # sonnet | opus | haiku | inherit (default)
permissionMode: acceptEdits       # default | acceptEdits | auto | bypassPermissions | plan
maxTurns: 20                      # cap agentic turns
memory: project                   # user | project | local — enables cross-session memory
skills:                           # inject full skill content at startup
  - api-conventions
  - error-handling-patterns
isolation: worktree               # run in isolated git worktree
background: false                 # true = always background
color: blue                       # red|blue|green|yellow|purple|orange|pink|cyan
---

You are a senior code reviewer. When invoked:
1. Run `git diff` to see recent changes
2. Review for: correctness, security, test coverage, style
3. Output as: Critical / Warning / Suggestion with fix examples
```

**Model resolution order:** `CLAUDE_CODE_SUBAGENT_MODEL` env var → per-invocation param → frontmatter → main session model.

---

## Invoking Subagents

**Natural language** (Claude decides whether to delegate):
```text
Use the code-reviewer subagent to check the auth module
```

**@-mention** (guaranteed delegation):
```text
@"code-reviewer (agent)" review the auth changes
```

**Run whole session as a subagent:**
```bash
claude --agent code-reviewer
```

**Set session default in `.claude/settings.json`:**
```json
{ "agent": "code-reviewer" }
```

**CLI session-only (not saved):**
```bash
claude --agents '{"code-reviewer": {"description": "...", "prompt": "...", "tools": ["Read","Grep"], "model": "sonnet"}}'
```

---

## Controlling Capabilities

**Restrict spawnable subagents** (when running as main agent via `--agent`):
```yaml
tools: Agent(worker, researcher), Read, Bash   # only these subagent types can be spawned
```

**Scope MCP servers to a subagent only:**
```yaml
mcpServers:
  - playwright:
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
  - github          # reference to an already-configured server
```

**Disable specific subagents:**
```json
{ "permissions": { "deny": ["Agent(Explore)", "Agent(my-agent)"] } }
```

---

## Persistent Memory

```yaml
memory: project   # or user / local
```

When enabled:
- Memory stored at `.claude/agent-memory/<name>/` (project) or `~/.claude/agent-memory/<name>/` (user)
- `MEMORY.md` index is injected at session start (first 200 lines / 25KB)
- Read, Write, Edit tools are auto-enabled for memory management

Include in the agent body so it maintains itself:
```markdown
Update your agent memory as you discover patterns, conventions, and architectural decisions.
```

---

## Subagent Hooks

**In subagent frontmatter** (fires only while that subagent is active):
```yaml
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
```

**In `settings.json`** (fires in the main session on subagent lifecycle events):
```json
{
  "hooks": {
    "SubagentStart": [{ "matcher": "db-agent", "hooks": [{"type":"command","command":"./setup-db.sh"}] }],
    "SubagentStop":  [{ "hooks": [{"type":"command","command":"./cleanup-db.sh"}] }]
  }
}
```

---

## Context Management

**Foreground vs. background:**
- Foreground: blocks main conversation; permission prompts pass through.
- Background: concurrent; permissions pre-approved at launch. Press **Ctrl+B** to background a running task.

**Resuming a subagent** (requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`):
```text
Continue that code review and now analyze the authorization logic
```
Claude uses `SendMessage` with the agent ID to resume — retains full conversation history.

**Auto-compaction:** triggers at ~95% capacity; override with `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE=50`.

**Transcripts:** stored at `~/.claude/projects/{project}/{sessionId}/subagents/agent-{id}.jsonl`. Retained for `cleanupPeriodDays` (default: 30).

---

## Example Agents

**Read-only code reviewer:**
```markdown
---
name: code-reviewer
description: Reviews code for quality, security, and maintainability. Use proactively after code changes.
tools: Read, Grep, Glob, Bash
model: inherit
---
Run `git diff`, review changed files, output Critical / Warning / Suggestion with fix examples.
```

**Debugger with write access:**
```markdown
---
name: debugger
description: Diagnoses and fixes errors, test failures, and unexpected behavior. Use proactively when issues arise.
tools: Read, Edit, Bash, Grep, Glob
---
Capture error + stack trace → reproduce → isolate root cause → implement minimal fix → verify.
```

**Read-only DB analyst with hook guard:**
```markdown
---
name: db-reader
description: Execute read-only database queries for analysis and reporting.
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---
Write SELECT queries only. Explain that you have no write access if asked.
```

---

## Distribute via Plugin / Marketplace

Place agent `.md` files in an `agents/` directory inside a plugin so the team gets them on install.

→ See **05-claude-code-marketplace-guide.md** for plugin distribution.
