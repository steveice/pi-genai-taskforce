# Making Claude Code Behave: What Your Global CLAUDE.md Should Look Like

## Why a Global CLAUDE.md Matters

Everyone's development environment is different, and the way you use Claude Code varies widely. When working with Claude Code, I try to put project-specific differences in each project's `CLAUDE.md` — even different subdirectories can have their own dedicated `CLAUDE.md`. But the "constitutional principles" that truly govern Claude Code's behavior live in `~/.claude/CLAUDE.md`. This is the fundamental law for using Claude Code on this machine.

## Principles for Writing CLAUDE.md

Here are the principles I follow when writing my CLAUDE.md. Since it's your own personal "constitution," keep it simple:

- **Progressive Disclosure:** Put details in the project-level `CLAUDE.md`.
- **60-Line Rule:** Keep it under 60 lines. Never exceed 80 lines.
- **Clear Execution Boundaries:** Be explicit about what's forbidden. Don't say "try to avoid" — say "absolutely not allowed."
- **Clean Markdown Structure:** After writing your `CLAUDE.md`, run a linter to check formatting.
- **No Technical Details:** Details belong in the project. The global config must not contain any tech stack specifics, because different projects use different implementations.
- **Testability:** Always write tests first when developing. For agent-based projects, find a way to do end-to-end (E2E) testing.
- **Regular Cleanup:** Outdated rules are worse than no rules — they mislead AI behavior.
- **Avoid Duplication:** Don't repeat information already in `MEMORY.md` or Git history.
- **Use Latest Data:** Require the AI to search the web and use Context7 or similar MCP tools to get up-to-date information.

I've revised this `CLAUDE.md` many times and found the current approach works best for me. It may not suit everyone, but hopefully it's a useful reference.

---

## Sample Global CLAUDE.md — Key Guidelines

Below is a sample template written in English. Adapt it to your own working environment.

---

### Your Environment

#### Hardware
- Machine type: DGX Spark with GB10 Blackwell GPU
- Memory: 128GB UMA architecture (not GPU VRAM) — capable of running very large models
- CPU: arm64 (not x86), 20 cores
- Storage: 4TB disk, 64GB SWAP

#### Software
- OS: Ubuntu Linux 24.04 (ported version)
- CUDA 13.0 installed, system driver version 580 — update this file if driver changes
- UMA memory means `nvidia-smi` cannot show memory usage; use standard tools like `free -h` instead

---

### Development Standards

#### Package & Environment Management
- All system services default to docker/podman — avoid native installation whenever possible
- Default package manager is brew; use apt-get only when permissions allow
- **Never use pip directly.** Always use uv to create virtual environments. Running Python in the system environment is not allowed
- For quick execution, prefer uvx first; only install packages when absolutely necessary
- Node.js projects must use nvm for installation, npm for packages, and npx for execution

#### Project Management
- All software projects must be pushed to GitHub. Default is public repo, using the project folder name as the repo name
- All software projects must include a properly formatted README.md

#### Language & Security
- All output must be in Traditional Chinese (Taiwan usage)
- Simplified Chinese or Traditional Chinese with mainland China terminology is strictly forbidden
- Never expose passwords, keys, access tokens, or API keys directly in any conversation
- Secrets are stored in environment variables; read from env vars or the `.env` file in the project root when needed

#### Getting Latest Information
- For anything related to APIs, functions, or code, always search the web before starting work
- Use Context7 or equivalent MCP tools for up-to-date information
- When searching, cross-reference by: who, what, when (today's date), where (your location), and why (project goal)

#### Testing First
- All projects must have tests with adequate coverage
- Use your tools — computer use, browser tools — to perform end-to-end testing
- Understand your User Stories and leverage test case methodologies
- Unit tests and functional tests are mandatory

#### CLI Tool Usage
- Highest priority: use built-in CLI tools
- If a task can be done with a CLI tool, use the CLI tool
- Before any development, always confirm your current directory with `pwd`
- When needing to access parent directories, always ask for permission first
- **Under absolutely no circumstances execute `rm -rf /`.** If this command is ever generated during any process, it is an unforgivable violation

#### CLI Tool Inventory
- **Package management:** brew (Linuxbrew), apt (arm64), uv (Python virtual env management), pip3 (reference only — direct use forbidden), npm (via nvm), snap
- **Development tools:** git, gh (GitHub CLI), node (via nvm), python3, java (OpenJDK), gcc/g++, cmake, make, hf (HuggingFace CLI)
- **GPU/CUDA:** nvidia-smi, nvcc
- **Containers:** docker, podman
- **Multimedia & utilities:** ffmpeg, curl, wget, jq, yq (via brew), rg (ripgrep), fzf, eza (via brew), htop, vim, tmux
