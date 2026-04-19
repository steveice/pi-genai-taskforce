# Claude Code — Plugin & Marketplace Guide

> Sources: [code.claude.com/docs/en/plugins](https://code.claude.com/docs/en/plugins) · [/en/discover-plugins](https://code.claude.com/docs/en/discover-plugins) · [/en/plugin-marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)

Plugins are the packaging format for distributing skills, hooks, MCP servers, and agents across a team or the community. A marketplace is a catalog that makes plugins discoverable and installable with a single command.

**Standalone `.claude/` config** → personal or single-project use, short skill names (`/deploy`).  
**Plugin + marketplace** → team sharing, versioned releases, namespaced skills (`/my-plugin:deploy`).

---

## Part I — Creating a Plugin

### Plugin Directory Layout

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # manifest (required)
├── skills/
│   └── <skill-name>/
│       └── SKILL.md
├── agents/
│   └── code-reviewer.md
├── hooks/
│   └── hooks.json
├── .mcp.json                # MCP server definitions
└── settings.json            # optional default settings (only "agent" key supported)
```

> Put only `plugin.json` inside `.claude-plugin/`. Everything else goes at the plugin root.

### Manifest (`plugin.json`)

```json
{
  "name": "my-plugin",
  "description": "Team productivity tools",
  "version": "1.0.0",
  "author": { "name": "Your Name" }
}
```

`name` becomes the skill namespace prefix: `skills/deploy/SKILL.md` → `/my-plugin:deploy`.

### Add Skills

```
my-plugin/skills/deploy/SKILL.md
```

```yaml
---
description: Deploy the application to staging
disable-model-invocation: true
allowed-tools: Bash(npm *) Bash(git *)
---
1. Run `npm test` — abort on failure
2. Run `npm run build`
3. Push to staging
```

### Add Hooks (`hooks/hooks.json`)

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          { "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npm run lint:fix" }
        ]
      }
    ]
  }
}
```

Use `${CLAUDE_PLUGIN_ROOT}` to reference scripts bundled with the plugin (path is stable after install).

### Add MCP Servers (`.mcp.json`)

```json
{
  "mcpServers": {
    "database-tools": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": { "DB_URL": "${DB_URL}" }
    }
  }
}
```

MCP servers in a plugin start automatically when the plugin is enabled and stop when it is disabled.

### Add Agents

```
my-plugin/agents/code-reviewer.md
```

```markdown
---
name: code-reviewer
description: Reviews code for quality and security. Use proactively after changes.
tools: Read, Grep, Glob, Bash
model: sonnet
---
Review checklist: correctness, security, test coverage, style.
Output as: Critical / Warning / Suggestion.
```

### Test Locally

```bash
claude --plugin-dir ./my-plugin
```

Local `--plugin-dir` takes precedence over any installed version of the same plugin name. After edits:

```text
/reload-plugins
```

---

## Part II — Creating a Team Marketplace

A marketplace is a git repository (or local directory) containing a `marketplace.json` catalog.

### Marketplace Structure

```
my-marketplace/
├── .claude-plugin/
│   └── marketplace.json     # catalog
└── plugins/
    ├── dev-tools/           # plugin A
    │   ├── .claude-plugin/plugin.json
    │   ├── skills/…
    │   └── hooks/hooks.json
    └── db-tools/            # plugin B
        ├── .claude-plugin/plugin.json
        └── .mcp.json
```

### `marketplace.json`

```json
{
  "name": "acme-tools",
  "owner": { "name": "ACME DevTools Team", "email": "devtools@acme.com" },
  "metadata": { "description": "Internal tools for ACME engineers" },
  "plugins": [
    {
      "name": "dev-tools",
      "source": "./plugins/dev-tools",
      "description": "Linting, formatting, and commit workflow skills",
      "version": "1.2.0"
    },
    {
      "name": "db-tools",
      "source": "./plugins/db-tools",
      "description": "Read-only database query agent + MCP server",
      "version": "0.9.0"
    },
    {
      "name": "code-formatter",
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-formatter-plugin",
        "ref": "v2.0.0"
      },
      "description": "Auto-formatter backed by a separate repo"
    }
  ]
}
```

**Plugin source types:**

| Type            | Example                                                                   |
|:----------------|:--------------------------------------------------------------------------|
| Relative path   | `"./plugins/my-plugin"` (same repo — recommended for monorepo setups)    |
| GitHub          | `{"source":"github","repo":"org/repo","ref":"v1.0.0"}`                   |
| Git URL         | `{"source":"url","url":"https://gitlab.com/team/plugin.git","ref":"main"}` |
| Git subdirectory | `{"source":"git-subdir","url":"…","path":"tools/claude-plugin"}`         |
| npm             | `{"source":"npm","package":"@acme/claude-plugin","version":"^2.0.0"}`   |

Pin with `"sha": "<40-char-commit-sha>"` for reproducible installs.

---

## Part III — Distributing to Your Team

### Step 1: Host the Marketplace

Push the marketplace repository to GitHub (or any git host):

```bash
git init my-marketplace && cd my-marketplace
# add .claude-plugin/marketplace.json and plugins/
git remote add origin git@github.com:acme-corp/claude-plugins.git
git push -u origin main
```

### Step 2: Team Members Add the Marketplace

```bash
# Inside Claude Code
/plugin marketplace add acme-corp/claude-plugins

# Or from the CLI
claude plugin marketplace add acme-corp/claude-plugins
```

Other source formats:
```bash
/plugin marketplace add https://gitlab.com/acme/plugins.git   # GitLab / any git host
/plugin marketplace add ./my-local-marketplace                 # local path (dev/testing)
```

### Step 3: Install Plugins

```bash
/plugin install dev-tools@acme-tools
/plugin install db-tools@acme-tools

# Or from CLI
claude plugin install dev-tools@acme-tools
```

Choose scope on install:
- **User** (default) — your machine, all projects
- **Project** — writes to `.claude/settings.json`, shared with the team
- **Local** — your machine, this project only

```bash
claude plugin install dev-tools@acme-tools --scope project
```

### Step 4: Activate Changes

```text
/reload-plugins
```

---

## Part IV — Auto-Install for the Team (Recommended)

Add marketplace and plugin config to `.claude/settings.json` in the repo. When teammates open the project, Claude Code prompts them to install:

```json
{
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "dev-tools@acme-tools": true,
    "db-tools@acme-tools": true
  }
}
```

This is the lowest-friction path for team adoption — teammates don't need to run any marketplace commands manually.

---

## Part V — Manage Plugins

```bash
/plugin                          # open plugin manager UI (Discover / Installed / Marketplaces / Errors tabs)
/plugin marketplace list         # list all added marketplaces
/plugin marketplace update acme-tools   # pull latest catalog
/plugin marketplace remove acme-tools  # also uninstalls plugins from it
/plugin disable dev-tools@acme-tools   # disable without uninstalling
/plugin enable dev-tools@acme-tools
/plugin uninstall dev-tools@acme-tools
```

**Auto-updates:** Official Anthropic marketplace has auto-update on by default. For your team marketplace, enable via `/plugin` → Marketplaces → select → Enable auto-update.

---

## Part VI — Convert Existing Config to a Plugin

If you already have skills or hooks in `.claude/`, migrate them:

```bash
mkdir -p my-plugin/.claude-plugin
cat > my-plugin/.claude-plugin/plugin.json <<'EOF'
{"name": "my-plugin", "description": "Migrated from .claude/", "version": "1.0.0"}
EOF

cp -r .claude/skills my-plugin/
cp -r .claude/agents my-plugin/   # if any
mkdir my-plugin/hooks
# Copy hooks object from .claude/settings.json into my-plugin/hooks/hooks.json
```

Test:
```bash
claude --plugin-dir ./my-plugin
```

After verifying, remove the originals from `.claude/` to avoid duplicates.

---

## Part VII — Official Anthropic Marketplace

The official marketplace (`claude-plugins-official`) is pre-loaded. Browse at [claude.com/plugins](https://claude.com/plugins) or:

```bash
/plugin   # → Discover tab
```

Install directly:
```bash
/plugin install github@claude-plugins-official
/plugin install typescript-lsp@claude-plugins-official
```

Notable categories:
- **LSP plugins** — code intelligence for TypeScript, Python, Go, Rust, etc.
- **External integrations** — GitHub, Jira, Figma, Slack, Sentry, Linear, Notion
- **Workflow skills** — commit-commands, pr-review-toolkit, agent-sdk-dev

Submit your own plugin: [claude.ai/settings/plugins/submit](https://claude.ai/settings/plugins/submit)

---

## Part VIII — Enterprise Controls

### Lock down which marketplaces are allowed (managed settings)

```json
{
  "strictKnownMarketplaces": [
    { "source": "github", "repo": "acme-corp/approved-plugins" },
    { "source": "hostPattern", "hostPattern": "^github\\.acme\\.com$" }
  ]
}
```

`strictKnownMarketplaces: []` blocks all marketplace additions. Pair with `extraKnownMarketplaces` to pre-register approved ones.

### Pre-populate plugins in CI / containers

```bash
# At image build time:
CLAUDE_CODE_PLUGIN_CACHE_DIR=/opt/claude-seed \
  claude plugin marketplace add acme-corp/claude-plugins
CLAUDE_CODE_PLUGIN_CACHE_DIR=/opt/claude-seed \
  claude plugin install dev-tools@acme-tools

# At runtime:
export CLAUDE_CODE_PLUGIN_SEED_DIR=/opt/claude-seed
```

The seed directory is read-only — users cannot update seed-managed plugins; admin updates the image instead.

---

## Quick Reference

```bash
# Create & test
claude --plugin-dir ./my-plugin       # test local plugin
/reload-plugins                       # hot-reload after edits
claude plugin validate .              # validate marketplace JSON

# Marketplace management
/plugin marketplace add acme-corp/claude-plugins
/plugin marketplace add https://gitlab.com/team/plugins.git
/plugin marketplace list
/plugin marketplace update acme-tools

# Plugin install
/plugin install dev-tools@acme-tools
/plugin install dev-tools@acme-tools --scope project
/plugin disable dev-tools@acme-tools
/plugin uninstall dev-tools@acme-tools

# Official marketplace
/plugin install github@claude-plugins-official
```

**File summary:**

| What you're sharing | Where it goes in the plugin |
|:--------------------|:----------------------------|
| Skills              | `skills/<name>/SKILL.md`    |
| Hooks               | `hooks/hooks.json`          |
| MCP servers         | `.mcp.json`                 |
| Agents / subagents  | `agents/<name>.md`          |
| Default settings    | `settings.json`             |
