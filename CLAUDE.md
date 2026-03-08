# Claude Code Instructions — claude-plugins

## Project Type
This is a **full nbdev project**. Jupyter notebooks in `nbs/` are the **source of truth**.

> **CRITICAL**: Edit `nbs/*.ipynb` files — NOT files in `claude_plugins/`. After editing any notebook, run `nbdev-export` to regenerate the Python package.

## Quick Reference

```bash
# After editing notebooks:
uv run nbdev-export

# Run tests:
uv run nbdev-test

# Build docs:
uv run nbdev-docs

# Prepare for commit (export + test + clean):
uv run nbdev-prepare

# Install all dev deps:
uv sync --extra dev
```

## Active Claude Code Hooks

| Hook | Event | Tool | What it does |
|------|-------|------|-------------|
| safecmd | PreToolUse | Bash | Validates shell commands against allowlist before running |
| exhash | PostToolUse | Edit, Write | Logs hash-addressed snapshot of every file edit to `.claude/edit_audit.jsonl` |
| indexer | SessionStart | — | Indexes codebase with codesigs + litesearch into `.claude/code_index.db` |

## Available MCP Tools

These tools are registered in `.mcp.json` and available in every Claude Code session:

- **exhash**: `lnhashview(file)`, `exhash_edit(file, commands)` — hash-addressed file editing
- **safecmd**: `validate_command(cmd)`, `run_safe(cmd)` — safe command validation and execution
- **safepyrun**: `run_python(code)`, `allow_function(fn)` — sandboxed Python execution

## Available Skills

Invoke with `/skill-name` in Claude Code:

- `/exhash` — edit files using hash-addressed line references
- `/safecmd` — validate and run shell commands safely
- `/safepyrun` — run Python code in a sandbox
- `/litesearch` — semantic + keyword search over the codebase index
- `/fasthtml` — build web UIs with FastHTML + HTMX
- `/monsterui` — styled components for FastHTML (cards, forms, navbars, tables, DaisyUI)
- `/lisette` — call LLMs via Lisette wrapper
- `/nbdev` — notebook-driven development workflows
- `/codesigs` — extract function/method signatures from source code

## Notebook Structure

| Notebook | Exports | Purpose |
|----------|---------|---------|
| `nbs/index.ipynb` | — | Package docs homepage |
| `nbs/00_core.ipynb` | `claude_plugins/core.py` | Core helpers and config |
| `nbs/01_hooks.ipynb` | `claude_plugins/hooks/` | All 3 hook scripts |
| `nbs/02_mcp_exhash.ipynb` | `claude_plugins/mcp_exhash.py` | exhash MCP server |
| `nbs/03_mcp_safecmd.ipynb` | `claude_plugins/mcp_safecmd.py` | safecmd MCP server |
| `nbs/04_mcp_safepyrun.ipynb` | `claude_plugins/mcp_safepyrun.py` | safepyrun MCP server |
| `nbs/05_skills.ipynb` | `claude_plugins/skills_gen.py` | Skill generation utilities |
| `nbs/06_setup.ipynb` | `claude_plugins/setup_cmd.py` | `claude-plugins setup` CLI |

## Package Dependencies

All packages are from AnswerDotAI / fastai ecosystem:

| Package | Repo | Purpose |
|---------|------|---------|
| exhash | answerdotai/exhash | Hash-addressed file editing |
| safecmd | answerdotai/safecmd | Safe shell command validation |
| safepyrun | AnswerDotAI/safepyrun | Sandboxed Python execution |
| litesearch | karthik777/litesearch | Hybrid FTS5 + vector code search |
| fastHTML | answerdotai/fasthtml | Web framework (HTMX-native) |
| MonsterUI | answerdotai/MonsterUI | Tailwind/FrankenUI components for FastHTML |
| Lisette | answerdotai/Lisette | LLM wrapper (100+ providers) |
| codesigs | answerdotai/codesigs | Code signature extraction |
| nbdev | fastai/nbdev | Notebook-driven development |

## Code Index

The session-start hook builds a semantic code index at `.claude/code_index.db`.
Use `/litesearch` skill to query it. The index covers all `.py` files outside of `.venv/`, `.git/`, and `nbs/`.
