---
name: safecmd
description: Validate shell commands against an allowlist before running them. Use when you want to check if a command is safe to execute, or need to run a command through safecmd's AST-based safety validator.
user-invocable: true
allowed-tools: Bash, Read
---

# safecmd — Safe Shell Command Validation

safecmd validates shell commands using AST-based bash parsing before execution. It prevents dangerous commands from running by checking against a curated allowlist.

## Key Features

- **AST-based parsing** — understands bash syntax, not just string matching
- **Operator filtering** — catches dangerous operators (`>` to `/dev/`, `$(...)` injection, etc.)
- **Nested command detection** — spots subshell injections and pipe chains
- **Allowlist-based** — you control what's allowed

## Python API

```python
from safecmd import check

# Check if a command is safe
result = check("git status", allowed=["git", "ls", "cat"])
print(result.allowed)  # True or False
print(result.reason)   # Why it was blocked

# With custom allowlist
result = check("rm -rf /", allowed=["rm"])
# result.allowed = False
# result.reason = "blocked: contains dangerous pattern 'rm -rf /'"
```

## Allowlist Configuration

The project allowlist lives at `.claude/safecmd_allowlist.json`:

```json
{
  "allowed_commands": [
    "uv", "git", "python", "pytest",
    "nbdev-export", "nbdev-test",
    "ls", "cat", "grep", "find"
  ]
}
```

Edit this file to customize what Claude Code can run via Bash.

## Hook Behavior

The PreToolUse hook (installed by `claude-plugins setup`) intercepts every Bash tool call:

1. Reads `command` from the tool input
2. Validates against `.claude/safecmd_allowlist.json`
3. If **allowed** → exits silently (command runs normally)
4. If **blocked** → returns a deny decision with explanation

## Via MCP Tool

When the safecmd MCP server is running:

```
validate_command(command="git status")
→ {"allowed": true, "reason": "allowed: git is in allowlist"}

run_safe(command="rm -rf /")
→ {"success": false, "error": "Command blocked: contains dangerous pattern"}
```

## What Gets Blocked by Default

- `rm -rf /` or `rm -rf ~` — destructive deletion
- Fork bombs `:(){ :|:& };:`
- Writing to `/dev/` devices
- `dd if=` — raw disk operations
- `mkfs` — filesystem formatting
- Any binary not in the allowlist

## Adding to the Allowlist

```bash
# Edit the allowlist file
cat .claude/safecmd_allowlist.json
# Add your command to "allowed_commands" array
```

Or use Python:
```python
import json
from pathlib import Path

p = Path('.claude/safecmd_allowlist.json')
data = json.loads(p.read_text())
data['allowed_commands'].append('mycommand')
p.write_text(json.dumps(data, indent=2))
```

## Installation

```bash
pip install safecmd
```
