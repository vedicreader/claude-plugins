# claude-plugins

> One-command Claude Code dev environment setup — hooks, MCP servers, and skills for the AnswerDotAI ecosystem.

## What it does

Run one command in any project to get:

| Component | What | How |
|-----------|------|-----|
| **Hooks** | safecmd blocks dangerous Bash; exhash audits every file edit; code indexer runs on session start | `.claude/settings.json` |
| **MCP Servers** | `lnhashview`, `exhash_edit`, `validate_command`, `run_safe`, `run_python` as Claude tools | `.mcp.json` |
| **Skills** | `/exhash` `/safecmd` `/safepyrun` `/litesearch` `/fasthtml` `/monsterui` `/lisette` `/nbdev` `/codesigs` | `.claude/skills/` |
| **Packages** | All AnswerDotAI tools installed in your UV venv | `uv pip install` |

## Install

```bash
pip install claude-plugins
# or
uv add claude-plugins
```

## Setup a project

```bash
cd your-project
claude-plugins setup
```

That's it. Open Claude Code and all hooks, tools, and skills are live.

## Selective setup

```bash
# Skip package installation (hooks, MCP, skills only)
claude-plugins setup --skip-packages

# Install specific components only
claude-plugins setup --components hooks          # hooks only
claude-plugins setup --components mcp            # MCP servers only
claude-plugins setup --components skills         # skills only
claude-plugins setup --components hooks,mcp      # hooks + MCP, no skills

# Target a specific directory
claude-plugins setup --target-dir /path/to/project

# Also index installed packages into the code search index
claude-plugins setup --index-deps
```

## Packages installed

| Package | Purpose |
|---------|---------|
| `python-fasthtml` | Web framework (HTMX-native) |
| `MonsterUI` | Tailwind/FrankenUI components for FastHTML |
| `litesearch` | Hybrid FTS5 + vector code search |
| `lisette` | LLM wrapper (100+ providers) |
| `exhash` | Hash-addressed file editing |
| `safecmd` | Safe shell command validation |
| `safepyrun` | Sandboxed Python execution |
| `codesigs` | Code signature extraction |
| `nbdev` | Notebook-driven development |

## Hooks

| Hook | Event | What |
|------|-------|------|
| `safecmd_hook` | `PreToolUse` on Bash | Validates every shell command against `.claude/safecmd_allowlist.json` |
| `exhash_hook` | `PostToolUse` on Edit/Write | Logs hash-addressed snapshot to `.claude/edit_audit.jsonl` |
| `index_hook` | `SessionStart` | Indexes project (+ optionally venv) with codesigs + litesearch |

Edit `.claude/safecmd_allowlist.json` to control what commands Claude Code can run.

## MCP Tools

When a Claude Code session is active, these tools are available directly:

```
# File editing with hash verification
lnhashview(file_path)                    → hash-addressed line view
exhash_edit(file_path, commands)         → apply verified edits

# Safe command execution
validate_command(command)                → {allowed, reason}
run_safe(command)                        → {success, stdout, stderr}

# Sandboxed Python
run_python(code)                         → {success, stdout, result}
reset_sandbox()                          → clear session state
```

## opencode support

`claude-plugins setup` also writes an `opencode.json` file registering the MCP servers for [opencode](https://opencode.ai) compatibility.

## GitHub Template

This repo is configured as a GitHub template. Click **"Use this template"** to bootstrap a new project with all hooks, MCP servers, and skills pre-configured.

After creating from template:
```bash
uv sync --extra dev   # install all packages
claude                # start Claude Code — hooks are already active
```

## Project structure (nbdev)

This package is developed using [nbdev](https://nbdev.fast.ai). Edit the notebooks in `nbs/`, not the exported Python files.

```bash
uv run nbdev-export    # export notebooks → python
uv run nbdev-test      # run tests
uv run nbdev-prepare   # export + test + clean (before commit)
```

## License

Apache 2.0
