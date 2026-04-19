# Claude Code — Hooks Reference

> Source: [code.claude.com/docs/en/hooks-guide](https://code.claude.com/docs/en/hooks-guide)

Hooks are shell commands (or LLM calls) that execute automatically at lifecycle points. They provide **deterministic** automation — things that must happen regardless of what Claude decides. Use skills for guidance; use hooks for guaranteed execution.

---

## Configuration

Add hooks to a settings file. Scope determines who and where they apply:

| File                          | Scope                 | Shared? |
|:------------------------------|:----------------------|:--------|
| `~/.claude/settings.json`     | All your projects     | No      |
| `.claude/settings.json`       | This project          | Yes (commit to repo) |
| `.claude/settings.local.json` | This project          | No (gitignored) |
| Skill/agent frontmatter       | While that skill/agent is active | Yes |

```json
{
  "hooks": {
    "<EventName>": [
      {
        "matcher": "<tool-name or empty string>",
        "hooks": [
          { "type": "command", "command": "<shell command>" }
        ]
      }
    ]
  }
}
```

---

## Events

| Event                | Matcher filters on   | Notes                                        |
|:---------------------|:---------------------|:---------------------------------------------|
| `SessionStart`       | Session source       | `startup`, `resume`, `clear`, `compact`      |
| `UserPromptSubmit`   | —                    | Fires before Claude processes your prompt    |
| `PreToolUse`         | Tool name            | **Can block the action** (exit 2)            |
| `PermissionRequest`  | Tool name            | Can auto-approve or deny                     |
| `PostToolUse`        | Tool name            | Fires after success                          |
| `PostToolUseFailure` | Tool name            | Fires after failure                          |
| `Stop`               | —                    | When Claude finishes responding              |
| `Notification`       | Notification type    | `permission_prompt`, `idle_prompt`           |
| `SubagentStart`      | Agent type name      |                                              |
| `SubagentStop`       | Agent type name      |                                              |
| `CwdChanged`         | —                    | When Claude `cd`s                            |
| `FileChanged`        | Literal filenames    | Watch specific files                         |
| `PreCompact`         | `manual`, `auto`     |                                              |
| `PostCompact`        | `manual`, `auto`     |                                              |
| `ConfigChange`       | Config source        | `user_settings`, `project_settings`, etc.   |
| `SessionEnd`         | Exit reason          | `clear`, `resume`, `logout`, `other`         |

Use `|` to match multiple tool names: `"matcher": "Edit|Write"`

---

## How Hooks Communicate

**Input:** JSON on stdin — always includes `session_id`, `cwd`, `hook_event_name`, and event-specific fields (e.g., `tool_name`, `tool_input`).

**Output:**

| Exit code | Meaning                                                                              |
|:----------|:-------------------------------------------------------------------------------------|
| `0`       | Allow. Stdout is added to Claude's context (for `SessionStart`/`UserPromptSubmit`). |
| `2`       | Block. Write reason to stderr — Claude receives it as feedback.                     |
| Other     | Error (non-blocking). First stderr line appears in transcript.                       |

For structured control, exit `0` and write JSON to stdout:

```json
{
  "hookSpecificOutput": {
    "hookEventName": "PreToolUse",
    "permissionDecision": "deny",
    "permissionDecisionReason": "Use rg instead of grep"
  }
}
```

`permissionDecision` values: `"allow"` | `"deny"` | `"ask"` | `"defer"` (non-interactive only)

> Do not mix exit 2 with JSON output — Claude Code ignores JSON when you exit 2.

---

## The `if` Field (Fine-Grained Filtering)

Filter by tool name **and** arguments — the hook process only spawns on a match:

```json
{
  "matcher": "Bash",
  "hooks": [
    { "type": "command", "if": "Bash(git *)", "command": ".claude/hooks/git-policy.sh" }
  ]
}
```

Uses the same syntax as permission rules: `"Bash(git *)"`, `"Edit(*.ts)"`.

---

## Common Recipes

### Desktop notification when Claude needs input

```json
{
  "hooks": {
    "Notification": [
      { "matcher": "", "hooks": [
        { "type": "command", "command": "osascript -e 'display notification \"Claude needs attention\" with title \"Claude Code\"'" }
      ]}
    ]
  }
}
```
Linux: `notify-send 'Claude Code' 'Needs attention'`

### Auto-format after every edit

```json
{
  "hooks": {
    "PostToolUse": [
      { "matcher": "Edit|Write", "hooks": [
        { "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npx prettier --write" }
      ]}
    ]
  }
}
```

### Block edits to protected files

Create `.claude/hooks/protect-files.sh`:
```bash
#!/bin/bash
INPUT=$(cat)
FILE=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')
for pattern in ".env" "package-lock.json" ".git/"; do
  [[ "$FILE" == *"$pattern"* ]] && echo "Blocked: $pattern is protected" >&2 && exit 2
done
```
```bash
chmod +x .claude/hooks/protect-files.sh
```
```json
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "Edit|Write", "hooks": [
        { "type": "command", "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/protect-files.sh" }
      ]}
    ]
  }
}
```

### Re-inject context after compaction

```json
{
  "hooks": {
    "SessionStart": [
      { "matcher": "compact", "hooks": [
        { "type": "command", "command": "echo 'Reminder: use Bun not npm. Current sprint: auth refactor.'" }
      ]}
    ]
  }
}
```

### Auto-approve a specific permission prompt

```json
{
  "hooks": {
    "PermissionRequest": [
      { "matcher": "ExitPlanMode", "hooks": [
        { "type": "command", "command": "echo '{\"hookSpecificOutput\":{\"hookEventName\":\"PermissionRequest\",\"decision\":{\"behavior\":\"allow\"}}}'" }
      ]}
    ]
  }
}
```

### Reload direnv on directory change

```json
{
  "hooks": {
    "CwdChanged": [
      { "hooks": [
        { "type": "command", "command": "direnv export bash >> \"$CLAUDE_ENV_FILE\"" }
      ]}
    ]
  }
}
```

### Log all bash commands

```json
{
  "hooks": {
    "PostToolUse": [
      { "matcher": "Bash", "hooks": [
        { "type": "command", "command": "jq -r '.tool_input.command' >> ~/.claude/command-log.txt" }
      ]}
    ]
  }
}
```

---

## Advanced Hook Types

| Type      | When to use                                               | Decision format           |
|:----------|:----------------------------------------------------------|:--------------------------|
| `command` | Deterministic rules — always use this first               | Exit codes + JSON stdout  |
| `prompt`  | Judgment calls that need LLM reasoning (Haiku by default) | `{"ok": true/false, "reason": "..."}` |
| `agent`   | Verification requiring file reads or command runs         | Same `ok`/`reason` format |
| `http`    | Forwarding events to an external audit or policy service  | JSON response body        |

**Prompt hook (check tasks complete before stopping):**
```json
{
  "hooks": {
    "Stop": [
      { "hooks": [
        { "type": "prompt", "prompt": "Are all tasks complete? If not: {\"ok\": false, \"reason\": \"what's left\"}" }
      ]}
    ]
  }
}
```

---

## Key Rules

- Hooks **tighten** restrictions — they cannot loosen past permission rules.
- A `deny` hook fires even in `bypassPermissions` mode.
- `PermissionRequest` hooks do not fire in non-interactive mode (`-p`); use `PreToolUse` instead.
- If `Stop` hooks loop forever, read `stop_hook_active` from input and `exit 0` when it's `true`.
- If your shell profile prints to stdout (unconditional `echo`), wrap it in `if [[ $- == *i* ]]; then ... fi` — hooks run in non-interactive shells.

**Debug:** `claude --debug-file /tmp/claude.log` then `tail -f /tmp/claude.log` in another terminal.  
**Inspect:** `/hooks` inside Claude Code — lists all configured hooks by event.

---

## Distribute via Plugin / Marketplace

Package hooks in `hooks/hooks.json` inside a plugin so the team gets them on install:

```json
{
  "hooks": {
    "PostToolUse": [
      { "matcher": "Write|Edit", "hooks": [
        { "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npm run lint:fix" }
      ]}
    ]
  }
}
```

→ See **05-claude-code-marketplace-guide.md** for plugin distribution.
