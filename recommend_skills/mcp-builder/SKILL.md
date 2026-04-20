---
name: mcp-builder
description: Guide for creating high-quality MCP (Model Context Protocol) servers that enable LLMs to interact with external services through well-designed tools. Use when building MCP servers to integrate external APIs or services, whether in Python (FastMCP) or Node/TypeScript (MCP SDK).
---

# MCP Server Development Guide

## Overview

Create MCP (Model Context Protocol) servers that enable LLMs to interact with external services through well-designed tools. The quality of an MCP server is measured by how well it enables LLMs to accomplish real-world tasks.

## High-Level Workflow

### Phase 1: Deep Research and Planning

#### API Coverage vs. Workflow Tools
Balance comprehensive API endpoint coverage with specialized workflow tools. When uncertain, prioritize comprehensive API coverage.

#### Tool Naming and Discoverability
Use consistent prefixes (e.g., `github_create_issue`, `github_list_repos`) and action-oriented naming.

#### Context Management
Design tools that return focused, relevant data. Support pagination where applicable.

#### Actionable Error Messages
Error messages should guide agents toward solutions with specific suggestions and next steps.

#### Study MCP Protocol Documentation
Start with the sitemap: `https://modelcontextprotocol.io/sitemap.xml`
Then fetch specific pages with `.md` suffix for markdown format.

#### Recommended Stack
- **Language**: TypeScript (recommended) or Python
- **Transport**: Streamable HTTP for remote servers (stateless JSON), stdio for local servers

### Phase 2: Implementation

#### Project Structure
- API client with authentication
- Error handling helpers
- Response formatting (JSON/Markdown)
- Pagination support

#### For Each Tool, Define:

**Input Schema:**
- Use Zod (TypeScript) or Pydantic (Python)
- Include constraints and clear descriptions

**Output Schema:**
- Define `outputSchema` where possible for structured data
- Use `structuredContent` in tool responses

**Annotations:**
- `readOnlyHint`: true/false
- `destructiveHint`: true/false
- `idempotentHint`: true/false
- `openWorldHint`: true/false

### Phase 3: Review and Test

- No duplicated code (DRY principle)
- Consistent error handling
- Full type coverage
- Clear tool descriptions

**TypeScript:** `npm run build` then `npx @modelcontextprotocol/inspector`
**Python:** `python -m py_compile your_server.py` then MCP Inspector

### Phase 4: Create Evaluations

Create 10 complex, realistic evaluation questions that:
- Are independent and read-only
- Require multiple tool calls and deep exploration
- Have single, clear, verifiable answers
- Won't change over time

## SDK References

- **Python SDK**: `https://raw.githubusercontent.com/modelcontextprotocol/python-sdk/main/README.md`
- **TypeScript SDK**: `https://raw.githubusercontent.com/modelcontextprotocol/typescript-sdk/main/README.md`
