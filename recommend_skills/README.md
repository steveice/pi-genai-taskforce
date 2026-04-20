# Recommended Skills for Claude Code

A curated collection of useful Agent Skills for Claude Code.

## Skills

| Skill | Description | Source |
|-------|-------------|--------|
| `karpathy-coding-guidelines` | Four principles from Andrej Karpathy: Think Before Coding, Simplicity First, Surgical Changes, Goal-Driven Execution | [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills) |
| `frontend-design` | Create distinctive, production-grade frontend interfaces that avoid generic AI aesthetics | [anthropics/skills](https://github.com/anthropics/skills) |
| `mcp-builder` | MCP Server development guide for Python (FastMCP) and TypeScript (MCP SDK) | [anthropics/skills](https://github.com/anthropics/skills) |
| `doc-coauthoring` | Structured doc co-authoring workflow: Context Gathering → Refinement → Reader Testing | [anthropics/skills](https://github.com/anthropics/skills) |
| `skill-creator` | Meta-skill for creating, testing, and optimizing Agent Skills | [anthropics/skills](https://github.com/anthropics/skills) |
| `webapp-testing` | Web application testing with Playwright | [anthropics/skills](https://github.com/anthropics/skills) |
| `ml-pipeline` | End-to-end MLOps: MLflow experiment tracking, Kubeflow/Airflow DAGs, Feature Store, model deployment | [Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills) |
| `pandas-pro` | pandas DataFrame operations, data cleaning, vectorized transforms, performance optimization | [Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills) |
| `sql-pro` | SQL query optimization, schema design, window functions, EXPLAIN plan analysis | [Jeffallan/claude-skills](https://github.com/Jeffallan/claude-skills) |

## Usage in Claude Code

Install as a Claude Code plugin marketplace:

```bash
# In Claude Code, register the Anthropic skills marketplace
/plugin marketplace add anthropics/skills

# Or install individual skills
/plugin install example-skills@anthropic-agent-skills
```

Or manually copy any skill folder into your project and reference it in your `CLAUDE.md`.
