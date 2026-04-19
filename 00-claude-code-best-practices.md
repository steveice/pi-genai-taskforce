# Claude Code — Best Practices

> Source: [code.claude.com/docs/en/best-practices](https://code.claude.com/docs/en/best-practices)

Most best practices stem from one constraint: **Claude's context window fills up fast, and performance degrades as it fills.** The context holds every message, file read, and command output — a single debugging session can consume tens of thousands of tokens. Manage it deliberately.

---

## Give Claude a Way to Verify Its Work

This is the single highest-leverage thing you can do. Claude performs dramatically better when it can run tests, compare screenshots, or validate outputs — without this, you become the only feedback loop.

| Goal                        | Instead of…                   | Do this…                                                                                                    |
|:----------------------------|:------------------------------|:------------------------------------------------------------------------------------------------------------|
| Function implementation     | "implement email validation"  | "write validateEmail. test cases: user@example.com → true, invalid → false. run tests after implementing"  |
| UI changes                  | "make the dashboard cleaner"  | "[paste screenshot] implement this design. screenshot the result, compare, list differences, fix them"      |
| Bug fix                     | "the build is failing"        | "build fails with [error]. fix it, verify build succeeds. address the root cause, don't suppress the error" |

Invest in solid verification: test suites, linters, Bash checks, or the [Chrome extension](/en/chrome) for UI.

---

## Explore First, Plan Second, Then Code

Use **Plan Mode** (`Ctrl+R`) to separate research from execution — jumping straight to code often solves the wrong problem.

1. **Explore** (Plan Mode): read files, understand the system, no changes made
2. **Plan** (Plan Mode): ask Claude to produce an implementation plan; press `Ctrl+G` to edit it in your editor
3. **Implement** (Normal Mode): execute the plan, run tests, fix failures
4. **Commit**: `"commit with a descriptive message and open a PR"`

> Skip planning for small, scoped tasks (fixing a typo, renaming a variable). Planning pays off when the change spans multiple files or you're unfamiliar with the code.

---

## Write Specific Prompts

| Strategy                    | Vague                             | Specific                                                                                            |
|:----------------------------|:----------------------------------|:----------------------------------------------------------------------------------------------------|
| Scope the task              | "add tests for foo.py"            | "write a test for foo.py for the logged-out edge case, no mocks"                                   |
| Point to sources            | "why is ExecutionFactory weird?"  | "look through ExecutionFactory's git history and summarize how its API evolved"                     |
| Reference existing patterns | "add a calendar widget"           | "look at HotDogWidget.php as a pattern example, then implement a calendar widget the same way"     |
| Describe the symptom        | "fix the login bug"               | "login fails after session timeout. check src/auth/ token refresh. write a failing test, then fix" |

**Provide rich content:** use `@filename` to reference files, paste screenshots directly, give URLs for docs, or pipe data: `cat error.log | claude`.

---

## Write an Effective CLAUDE.md

Run `/init` to generate a starter file, then refine it. Claude reads this at the start of every session.

```markdown
# Code style
- Use ES modules (import/export), not CommonJS (require)
- Destructure imports when possible

# Workflow
- Typecheck after a series of changes
- Run single tests, not the full suite, for speed
```

**Include:**
- Bash commands Claude can't guess
- Code style rules that differ from defaults
- Test runner and testing preferences
- Repo conventions (branch naming, PR workflow)
- Known environment quirks or required env vars

**Exclude:**
- Anything Claude can infer by reading code
- Standard language conventions
- Long explanations or tutorials
- Information that changes frequently

Keep it short. If removing a line wouldn't cause Claude to make mistakes, cut it. Bloated CLAUDE.md files cause Claude to ignore rules. Add `IMPORTANT:` or `YOU MUST:` for rules that must stick. Check it into git.

Use `@path/to/file` imports for shared content:
```markdown
See @README.md for project overview.
Git workflow: @docs/git-instructions.md
```

---

## Configure Your Environment

| Tool       | How to set up                                              | When it helps                                          |
|:-----------|:-----------------------------------------------------------|:-------------------------------------------------------|
| Permissions | `/permissions` allowlist or auto mode (`--permission-mode auto`) | Stops repetitive approve/deny clicks            |
| CLI tools  | Install `gh`, `aws`, `gcloud`, etc.                        | Most context-efficient way to hit external services    |
| MCP servers | `claude mcp add`                                          | Query databases, read issue trackers, integrate Figma  |
| Hooks      | Edit `.claude/settings.json`, browse with `/hooks`         | Deterministic actions that must happen every time      |
| Skills     | `.claude/skills/<name>/SKILL.md`                           | Domain knowledge and repeatable workflows              |
| Subagents  | `.claude/agents/<name>.md`                                 | Isolated tasks that would clutter main context         |
| Plugins    | `/plugin` → Discover tab                                   | Community skills, hooks, MCP, and language intelligence|

---

## Manage Your Session

**Stop and redirect:** `Esc` halts Claude mid-action; context is preserved so you can steer.

**Rewind:** press `Esc` twice or run `/rewind` to restore conversation, code, or both to any previous checkpoint. Checkpoints persist across sessions.

**Clear between tasks:** `/clear` resets context entirely. Use it aggressively between unrelated tasks.

**Compact with control:** `/compact <instructions>` — e.g., `/compact Focus on the API changes`. Add compaction preferences to CLAUDE.md: `"When compacting, always preserve the list of modified files and any test commands"`.

**Quick questions without growing context:** `/btw <question>` — answer appears in a dismissible overlay, never enters history.

**Resume sessions:** `claude --continue` (most recent) or `claude --resume` (choose from list). Name sessions with `/rename`.

### Common Failure Patterns to Avoid

| Pattern                   | Symptom                                            | Fix                                                          |
|:--------------------------|:---------------------------------------------------|:-------------------------------------------------------------|
| Kitchen-sink session      | Mixing unrelated tasks in one conversation         | `/clear` between tasks                                       |
| Correction loop           | Same mistake corrected twice with no improvement   | `/clear` and write a better initial prompt                   |
| Bloated CLAUDE.md         | Claude ignores rules buried deep in the file       | Prune ruthlessly; convert over-specified rules to hooks      |
| Trust-then-verify gap     | Plausible output that doesn't handle edge cases    | Always provide tests or a verification command               |
| Infinite exploration      | Claude reads hundreds of files, context fills up   | Scope investigations narrowly or delegate to a subagent      |

---

## Use Subagents for Investigation

Context is your fundamental constraint. Subagents run in separate context windows and report back summaries — exploration doesn't consume your main conversation.

```text
Use subagents to investigate how our auth system handles token refresh
and whether we have any existing OAuth utilities to reuse.
```

After implementation: `"use a subagent to review this code for edge cases"`

---

## Let Claude Interview You

For larger features, start with a minimal prompt and ask Claude to interview you:

```text
I want to build [brief description]. Interview me using the AskUserQuestion tool.
Ask about technical implementation, UI/UX, edge cases, and tradeoffs.
Don't ask obvious questions — dig into the hard parts.
When done, write a complete spec to SPEC.md.
```

Then start a **fresh session** to implement the spec. Clean context, clear requirements.

---

## Automate and Scale

**Non-interactive mode** — use in CI, pre-commit hooks, or scripts:
```bash
claude -p "explain what this project does"
claude -p "list all API endpoints" --output-format json
claude -p "analyze this log file" --output-format stream-json
cat error.log | claude -p "find the root cause"
```

**Fan out across files** — batch migrations or analysis:
```bash
for file in $(cat files.txt); do
  claude -p "Migrate $file from React to Vue. Return OK or FAIL." \
    --allowedTools "Edit,Bash(git commit *)"
done
```

**Auto mode for unattended runs:**
```bash
claude --permission-mode auto -p "fix all lint errors"
```
A classifier reviews commands before they run, blocking risky actions while letting routine work proceed without prompts.

**Writer/Reviewer pattern with parallel sessions:**
- Session A: `"Implement a rate limiter for our API endpoints"`
- Session B: `"Review @src/middleware/rateLimiter.ts — look for edge cases and race conditions"`
- Session A: `"Address this review feedback: [Session B output]"`

---

## Related Documents

| Document | Content |
|:---------|:--------|
| `01-claude-code-skills-guide.md` | Creating and configuring skills |
| `02-claude-code-mcp-guide.md` | Connecting external tools via MCP |
| `03-claude-code-hooks-guide.md` | Deterministic automation with hooks |
| `04-claude-code-subagents-guide.md` | Specialized agents and context management |
| `05-claude-code-marketplace-guide.md` | Team distribution via plugins and marketplaces |
