# Claude Code — Skills Reference

> Source: [code.claude.com/docs/en/skills](https://code.claude.com/docs/en/skills)

Skills are reusable playbooks stored as `SKILL.md` files. They load **on demand** — content only enters context when invoked, so large reference material is free until needed. Invoke with `/skill-name` or let Claude trigger them automatically when your request matches the description.

---

## File Locations

| Scope      | Path                                              | Shared via         |
|:-----------|:--------------------------------------------------|:-------------------|
| Personal   | `~/.claude/skills/<name>/SKILL.md`                | Your machine only  |
| Project    | `.claude/skills/<name>/SKILL.md`                  | Git / version control |
| Enterprise | Managed settings directory                        | IT deployment      |
| Plugin     | `<plugin>/skills/<name>/SKILL.md`                 | Marketplace        |

Higher-priority scopes override lower ones: Enterprise > Personal > Project > Plugin.

Each skill is a directory. `SKILL.md` is the only required file; add supporting files alongside it as needed.

---

## SKILL.md Structure

```yaml
---
name: my-skill           # becomes /my-skill (defaults to directory name)
description: >           # REQUIRED — tells Claude when to use it; front-load the key use case
  One clear sentence describing what this skill does and when.
disable-model-invocation: true   # optional — only you can invoke it (good for deploys, commits)
user-invocable: false            # optional — Claude-only background knowledge
allowed-tools: Bash(git *) Bash(npm *)  # optional — pre-approve tools; space-separated
context: fork            # optional — run in isolated subagent context
agent: Explore           # optional — which subagent type to use with context: fork
paths:                   # optional — only activate for matching files
  - "src/api/**/*.ts"
effort: high             # optional — low / medium / high / xhigh / max
---

Your instructions here in plain Markdown.
```

**Invocation control:**

| Setting                          | You invoke | Claude auto-invokes |
|:---------------------------------|:-----------|:--------------------|
| (default)                        | ✅         | ✅                  |
| `disable-model-invocation: true` | ✅         | ❌                  |
| `user-invocable: false`          | ❌         | ✅                  |

---

## Arguments

```yaml
---
name: fix-issue
disable-model-invocation: true
---
Fix GitHub issue $ARGUMENTS following our coding standards.
```

Usage: `/fix-issue 123`

Multi-argument positional access:

```yaml
Migrate the $0 component from $1 to $2.
# /migrate-component SearchBar React Vue
```

Other substitutions: `${CLAUDE_SESSION_ID}`, `${CLAUDE_SKILL_DIR}`

---

## Shell Preprocessing (Dynamic Context)

The `` !`command` `` syntax executes **before** Claude sees the content — output replaces the placeholder:

```yaml
---
name: pr-summary
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---
PR diff: !`gh pr diff`
Changed files: !`gh pr diff --name-only`

Summarize this PR in 3–5 bullets. Flag breaking changes.
```

Multi-line variant:

````markdown
```!
node --version
git status --short
```
````

---

## Practical Examples

**Project skill — API review:**
```yaml
# .claude/skills/api-review/SKILL.md
---
name: api-review
description: Reviews REST endpoints against team conventions. Use when writing or reviewing API handlers.
---
Check: RESTful naming, consistent error format `{"error":{"code":"","message":""}}`,
input validation, OpenAPI comments, correct HTTP status codes.
Output as: Critical / Warning / Suggestion.
```

**Personal skill — commit workflow:**
```yaml
# ~/.claude/skills/commit/SKILL.md
---
name: commit
description: Stage and commit current changes with a descriptive message
disable-model-invocation: true
allowed-tools: Bash(git add *) Bash(git commit *) Bash(git status *)
---
1. Review `git status` and `git diff`
2. Stage relevant files
3. Write a Conventional Commits message
4. Commit
```

**Forked skill — deep research:**
```yaml
---
name: deep-research
description: Research a topic thoroughly across the codebase
context: fork
agent: Explore
---
Research $ARGUMENTS. Find relevant files, read the code, return findings with file:line references.
```

---

## Key Behaviors

- Skill content enters context **once** per invocation and stays for the session.
- After `/compact`, the most recent invocation of each skill is re-attached (up to 5,000 tokens each; 25,000 total budget).
- Claude Code watches skill directories — edits take effect in the current session without restart.
- Keep `SKILL.md` under 500 lines; move large reference material to sibling files and reference them with `[see reference.md](reference.md)`.

---

## Distribute Skills

| Method         | How                                             |
|:---------------|:------------------------------------------------|
| Project skills | Commit `.claude/skills/` to version control     |
| Plugin         | Add `skills/` directory inside a plugin         |
| Enterprise     | Deploy via managed settings                     |
| Marketplace    | Publish as a plugin to a team/public marketplace |

→ See **05-claude-code-marketplace-guide.md** for plugin and marketplace distribution.

---

## Anthropic Official Skills Library

> Repo: [github.com/anthropics/skills](https://github.com/anthropics/skills/tree/main/skills)

Anthropic maintains a public library of production-quality skills you can install directly or use as reference implementations. Install any skill by copying its directory into `.claude/skills/` (project) or `~/.claude/skills/` (personal).

### Available Skills

| Skill | Purpose |
|:------|:--------|
| `algorithmic-art` | Generate algorithmic/generative artwork |
| `brand-guidelines` | Apply brand voice and style rules |
| `canvas-design` | Design work on canvas |
| `claude-api` | Work with the Claude API |
| `doc-coauthoring` | Collaborative document writing |
| `docx` | Create and manipulate Word documents |
| `frontend-design` | UI/UX design patterns |
| `internal-comms` | Internal communication templates |
| `mcp-builder` | Build MCP servers |
| `pdf` | PDF creation and extraction |
| `pptx` | Create and edit PowerPoint files |
| **`skill-creator`** | **Create, test, and optimize new skills** |
| `slack-gif-creator` | Generate Slack GIFs |
| `theme-factory` | Design system theming |
| `web-artifacts-builder` | Build web artifact components |
| `webapp-testing` | End-to-end web app testing |
| `xlsx` | Create and manipulate Excel files |

---

### `skill-creator` — Build Skills Interactively

> **This is the most important skill in the library.** Use it to create new skills from scratch, improve existing ones, and optimize triggering accuracy.

**Install:**
```bash
# Copy to your personal skills
cp -r <repo>/skills/skill-creator ~/.claude/skills/skill-creator
```

**Invoke:** `/skill-creator` or describe what you want to build and Claude triggers it automatically.

**What it does — the full loop:**

1. **Capture intent** — interviews you about the skill's purpose, trigger contexts, and expected output format
2. **Draft the skill** — writes a `SKILL.md` with correct frontmatter and body structure
3. **Write test cases** — generates 2–3 realistic test prompts and saves them to `evals/evals.json`
4. **Run evaluations in parallel** — spawns subagents: one with the skill, one without (baseline), simultaneously
5. **Grade quantitatively** — drafts assertions and grades outputs against them while runs are in progress
6. **Launch a review viewer** — opens a browser UI (`eval-viewer/generate_review.py`) showing side-by-side outputs, benchmark stats, and a feedback form
7. **Iterate** — reads your feedback from `feedback.json`, improves the skill, re-runs — repeat until satisfied
8. **Optimize description** — runs `scripts/run_loop.py` to hill-climb the `description` field for better triggering accuracy across 20 eval queries (10 should-trigger, 10 should-not)
9. **Package** — produces a `.skill` file via `scripts/package_skill.py` ready for sharing

**Key design principles baked into the skill:**
- **Generalize, don't overfit** — improvements target the underlying intent, not narrow test-case quirks
- **Explain the why** — instructions tell the model *why* each step matters, not just what to do
- **Bundle repeated work** — if test runs independently write the same helper script, it belongs in `skills/<name>/scripts/`
- **Progressive disclosure** — SKILL.md body stays under 500 lines; large reference material goes in sibling files

**Skill description writing guidance (from `skill-creator`):**

The `description` field is the primary triggering mechanism. To avoid undertriggering, write descriptions that are slightly "pushy" — explicitly name the contexts where the skill should fire:

```yaml
# Too passive:
description: How to build a dashboard for internal data.

# Better:
description: >
  How to build a simple fast dashboard to display internal data.
  Use this skill whenever the user mentions dashboards, data visualization,
  internal metrics, or wants to display any company data — even if they
  don't explicitly ask for a 'dashboard'.
```

**Skill anatomy (from `skill-creator`):**

```
my-skill/
├── SKILL.md                 ← required; frontmatter + instructions
├── scripts/                 ← deterministic helpers bundled with the skill
├── references/              ← large docs; reference from SKILL.md, read on demand
├── assets/                  ← templates, icons, static files
└── evals/
    └── evals.json           ← test cases for iteration and benchmarking
```

**Three-level loading:**
1. `name` + `description` — always in context (~100 words, zero cost until triggered)
2. `SKILL.md` body — loaded on invocation (keep under 500 lines)
3. Bundled resources — read on demand, scripts execute without loading into context
