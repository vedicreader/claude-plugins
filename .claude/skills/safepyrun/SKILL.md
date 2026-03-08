---
name: safepyrun
description: Run Python code in a sandboxed environment using safepyrun. Use when you need to execute Python safely with controlled access to stdlib and approved functions, preventing accidental harm from LLM-generated code.
user-invocable: true
allowed-tools: Bash
---

# safepyrun — Sandboxed Python Execution

safepyrun is an allowlist-based Python sandbox for LLM-generated code. It runs code in-process using RestrictedPython, giving access to a curated subset of stdlib without containerization overhead.

## Core API

```python
import asyncio
from safepyrun import RunPython

runner = RunPython()

# Run code async
result = await runner("print('hello')")
print(result.stdout)   # 'hello\n'
print(result.stderr)   # ''
print(result.result)   # None (or return value)
```

## Session State with `_` Convention

Variables ending with `_` are exported back to the caller's namespace, enabling state across multiple calls:

```python
result = await runner("data_ = [1, 2, 3]")
# data_ is now in the runner's namespace

result = await runner("total_ = sum(data_)")
# Can reference data_ from previous call
print(result.result)  # 6
```

## Default Allowed Libraries

The sandbox pre-approves these stdlib modules:
- `re`, `json`, `math`, `collections`, `datetime`
- `pathlib` (read-only), `urllib` (URL handling)
- `functools`, `itertools`, `operator`
- `base64`, `hashlib`, `uuid`

## Allowing Additional Functions

```python
import my_safe_function
runner.allow(my_safe_function)

# Now available in sandbox:
result = await runner("result_ = my_safe_function(data)")
```

## Filesystem Write Access

```python
# Allow writing to specific paths only
runner.allow_write('/tmp/output/', policy='prefix')

# Or with a validation function
runner.allow_write(lambda p: p.startswith('/tmp/'), policy='callable')
```

## Async Support

Full async/await support inside the sandbox:

```python
result = await runner("""
import asyncio

async def fetch():
    await asyncio.sleep(0.1)
    return 42

result_ = await fetch()
""")
```

## Via MCP Tool (in Claude Code)

When the safepyrun MCP server is running:

```
run_python(code="result_ = 2 + 2\nprint(result_)")
→ {"success": true, "stdout": "4\n", "result": 4}

run_python(code="import os; os.system('rm -rf /')")
→ {"success": false, "error": "PermissionError: os.system not allowed"}

reset_sandbox()
→ {"success": true, "message": "Sandbox reset"}
```

## Philosophy: "Safe-ish"

safepyrun is designed to prevent **accidental harm** from well-meaning LLMs — not to stop determined adversaries. It's appropriate for:
- Running LLM-generated data analysis code
- Executing Claude-suggested transformations
- Testing code snippets safely
- Notebook-style interactive exploration

## Installation

```bash
pip install safepyrun
```

Repo: https://github.com/AnswerDotAI/safepyrun
