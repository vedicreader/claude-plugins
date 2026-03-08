---
name: exhash
description: Edit files using hash-addressed line references for verified, auditable changes. Use when you need to make precise edits to specific lines identified by their content hash, or want to view a file with hash prefixes for addressing.
user-invocable: true
allowed-tools: Bash, Read
---

# exhash — Hash-Addressed File Editing

exhash is a file editor that combines line-number and hash-based addressing with classic `ex` editor syntax. It prevents edit conflicts by verifying line content before applying changes.

## Two Core Commands

### `lnhashview <file>` — View with hash prefixes
Shows the file with each line prefixed by a short hash. Use this first to get hashes for editing.

```
$ lnhashview myfile.py
a3f2|1: def hello():
b7c1|2:     print("hello")
9e4a|3:
```

Format: `HASH|LINENUM: content`

### `exhash <file>` — Edit via stdin commands
Apply edits using ex-like syntax with hash addressing:

```bash
exhash myfile.py <<'EOF'
s/b7c1/    print("world")/
EOF
```

## Command Syntax

| Command | Syntax | Description |
|---------|--------|-------------|
| Substitute | `s/HASH/new content/` | Replace line with matching hash |
| Append | `a/HASH/new line/` | Append line after hash-matched line |
| Delete | `d/HASH/` | Delete line with matching hash |
| Insert | `i/HASH/new line/` | Insert line before hash-matched line |

## Dry Run

Add `--dry-run` flag to preview changes without applying:
```bash
exhash --dry-run myfile.py <<'EOF'
s/a3f2/def greet(name: str):/
EOF
```

## Via MCP Tool (in Claude Code)

When the exhash MCP server is running, you can use:
- `lnhashview(file_path)` — returns hash-addressed view
- `exhash_edit(file_path, commands, dry_run=False)` — applies edits

## Workflow

1. Call `lnhashview` on the file to see hashes
2. Identify the lines to change by their hash
3. Write `s/HASH/new content/` commands
4. Run `exhash` (or use dry_run first)
5. The exhash PostToolUse hook automatically logs every edit to `.claude/edit_audit.jsonl`

## Installation

```bash
pip install exhash
# or
cargo install exhash  # CLI only, faster
```

## When to Use exhash vs Edit Tool

- Use **exhash** when you need hash-verified edits, especially for generated or frequently-changing files where line numbers shift
- Use **Edit tool** for interactive single-file edits with string matching
- exhash is better for scripted/repeatable edits and audit trails
