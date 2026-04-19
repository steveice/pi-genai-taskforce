# Claude Code — MCP Setup Reference

> Source: [code.claude.com/docs/en/mcp](https://code.claude.com/docs/en/mcp)

MCP (Model Context Protocol) connects Claude Code to external tools, databases, and APIs. Once connected, Claude can read, query, and act on those systems directly instead of relying on copy-pasted data.

---

## Install a Server

### Remote HTTP (recommended for cloud services)

```bash
claude mcp add --transport http <name> <url>

# Examples
claude mcp add --transport http notion https://mcp.notion.com/mcp
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# With auth header
claude mcp add --transport http my-api https://api.example.com/mcp \
  --header "Authorization: Bearer $TOKEN"
```

### Local stdio (for tools needing system access)

```bash
claude mcp add --transport stdio <name> -- <command> [args...]

# Examples
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:pass@prod.db.com:5432/analytics"

claude mcp add --transport stdio --env AIRTABLE_API_KEY=$KEY airtable \
  -- npx -y airtable-mcp-server
```

> All flags (`--transport`, `--env`, `--scope`, `--header`) must come **before** the server name. `--` separates Claude's flags from the server command.

> **Windows:** Wrap npx with `cmd /c`: `-- cmd /c npx -y @some/package`

---

## Scopes

| Scope     | Loaded in            | Shared with team | Stored in                   |
|:----------|:---------------------|:-----------------|:----------------------------|
| `local`   | Current project only | No               | `~/.claude.json`            |
| `project` | Current project only | Yes (via git)    | `.mcp.json` in project root |
| `user`    | All your projects    | No               | `~/.claude.json`            |

```bash
claude mcp add --transport http stripe --scope project https://mcp.stripe.com
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/anthropic
```

**Precedence:** Local > Project > User > Plugin > Claude.ai connectors

---

## Project `.mcp.json` (team sharing)

Commit this file to version control so the whole team gets the same servers:

```json
{
  "mcpServers": {
    "github": { "type": "http", "url": "https://api.githubcopilot.com/mcp/" },
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": { "Authorization": "Bearer ${API_KEY}" }
    }
  }
}
```

Environment variable expansion: `${VAR}` and `${VAR:-default}` work in `command`, `args`, `env`, `url`, and `headers`.

---

## Manage Servers

```bash
claude mcp list           # list all configured servers
claude mcp get github     # inspect a specific server
claude mcp remove github  # remove a server
/mcp                      # check status / authenticate (inside Claude Code)
```

---

## Authentication

### OAuth (most remote servers)

```bash
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
# then inside Claude Code:
/mcp   # select server → Authenticate → follow browser flow
```

Tokens are stored securely and refreshed automatically.

### Pre-configured credentials (no dynamic registration)

```bash
claude mcp add --transport http \
  --client-id your-client-id --client-secret --callback-port 8080 \
  my-server https://mcp.example.com/mcp

# Non-interactive (CI):
MCP_CLIENT_SECRET=your-secret claude mcp add --transport http \
  --client-id your-id --client-secret --callback-port 8080 \
  my-server https://mcp.example.com/mcp
```

### Dynamic headers (Kerberos, internal SSO, short-lived tokens)

```json
{
  "mcpServers": {
    "internal-api": {
      "type": "http",
      "url": "https://mcp.internal.example.com",
      "headersHelper": "/opt/bin/get-mcp-auth-headers.sh"
    }
  }
}
```

The script must write `{"Header-Name": "value"}` JSON to stdout. Runs on each connection; no caching.

---

## Restrict OAuth Scopes

```json
{
  "mcpServers": {
    "slack": {
      "type": "http",
      "url": "https://mcp.slack.com/mcp",
      "oauth": { "scopes": "channels:read chat:write" }
    }
  }
}
```

---

## MCP Tool Search

By default, tool definitions are deferred — only names load at session start, saving context. Tools are fetched on demand.

```bash
ENABLE_TOOL_SEARCH=false claude   # load all tools upfront
ENABLE_TOOL_SEARCH=auto:5 claude  # load upfront if within 5% of context window
export MAX_MCP_OUTPUT_TOKENS=50000  # raise output limit (default: 25,000)
```

---

## Useful Patterns

**Reference resources with @ mentions:**
```text
Can you analyze @github:issue://123 and suggest a fix?
```

**Run MCP prompts as commands:**
```text
/mcp__github__pr_review 456
/mcp__jira__create_issue "Bug in login flow" high
```

**Import from Claude Desktop (macOS/WSL only):**
```bash
claude mcp add-from-claude-desktop
```

---

## Enterprise: Managed MCP

**Exclusive control** — deploy a fixed set; users cannot add their own:

Place `managed-mcp.json` at the system path:
- macOS: `/Library/Application Support/ClaudeCode/managed-mcp.json`
- Linux/WSL: `/etc/claude-code/managed-mcp.json`

```json
{
  "mcpServers": {
    "github": { "type": "http", "url": "https://api.githubcopilot.com/mcp/" },
    "internal": { "type": "stdio", "command": "/usr/local/bin/internal-mcp" }
  }
}
```

**Policy-based** — allow users to add their own, within limits (in managed settings):

```json
{
  "allowedMcpServers": [
    { "serverName": "github" },
    { "serverUrl": "https://mcp.company.com/*" }
  ],
  "deniedMcpServers": [
    { "serverUrl": "https://*.untrusted.com/*" }
  ]
}
```

Denylist takes absolute precedence. `allowedMcpServers: []` blocks all MCP.

---

## Distribute via Plugin / Marketplace

Bundle MCP servers inside a plugin `.mcp.json` so the team gets them automatically on install:

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

→ See **05-claude-code-marketplace-guide.md** for plugin distribution.

---

## Recommended Team MCP Servers

Install these globally with `--scope user` so they're available in every project. Use `--` to separate Claude's flags from `uvx` commands:

```bash
# Context7 (up-to-date docs for LLMs and AI code editors)
claude mcp add --transport http --scope user context7 https://mcp.context7.com/mcp

# Atlassian Rovo (Jira & Confluence)
claude mcp add --transport http --scope user atlassian https://mcp.atlassian.com/v1/mcp

# Core AWS services
claude mcp add --scope user aws-api -- uvx awslabs.aws-api-mcp-server@latest
claude mcp add --scope user aws-iac -- uvx awslabs.aws-iac-mcp-server@latest
claude mcp add --scope user aws-docs -- uvx awslabs.aws-documentation-mcp-server@latest
claude mcp add --scope user aws-support -- uvx awslabs.aws-support-mcp-server@latest
claude mcp add --scope user aws-pricing -- uvx awslabs.aws-pricing-mcp-server@latest

# AI/ML and Bedrock
claude mcp add --scope user bedrock-kb -- uvx awslabs.bedrock-kb-retrieval-mcp-server@latest

# Data and analytics
claude mcp add --scope user aws-dataprocessing -- uvx awslabs.aws-dataprocessing-mcp-server@latest
claude mcp add --scope user aurora-dsql -- uvx awslabs.aurora-dsql-mcp-server@latest
claude mcp add --scope user valkey -- uvx awslabs.valkey-mcp-server@latest

# Verify
claude mcp list
```

> `--scope user` stores in `~/.claude.json` — available across all projects, not committed to version control.
