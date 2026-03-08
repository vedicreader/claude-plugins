---
name: codesigs
description: Extract function and method signatures from source code for indexing, documentation, or LLM context. Use when you need to understand the API surface of a codebase, build search indices, or generate documentation stubs.
user-invocable: true
allowed-tools: Bash, Read, Glob
---

# codesigs — Code Signature Extraction

codesigs extracts function and method signatures from source code across 12+ languages using syntax-aware parsing. It's the foundation of the codebase indexing pipeline (codesigs → litesearch).

## Supported Languages

Python, JavaScript, TypeScript, Java, Rust, C#, Go, Ruby, PHP, Kotlin, Swift, Lua, CSS

## Core API

### Auto-detect by file extension
```python
from codesigs import file_sigs, ext_sigs

# From a single file (auto-detects language)
sigs = file_sigs('src/utils.py')
for sig in sigs:
    print(sig['name'], sig['signature'])

# From all files with a given extension
sigs = ext_sigs('.py', 'src/')
```

### Language-specific functions
```python
from codesigs import py_sigs, js_sigs, rust_sigs, go_sigs

# Python
sigs = py_sigs('mymodule.py')

# JavaScript / TypeScript
sigs = js_sigs('components/Button.tsx')

# Rust
sigs = rust_sigs('src/main.rs')
```

## Signature Format

Each signature is a dict:
```python
{
    'name': 'validate_email',
    'signature': 'def validate_email(addr: str) -> bool:',
    'docstring': 'Check that addr is a valid email address.',
    'kind': 'function',   # 'function', 'class', 'method', 'async_function'
    'line': 42,           # Line number in source file
    'file': 'utils.py',   # Source file path
}
```

## Indexing a Full Codebase

```python
import json
from pathlib import Path
from codesigs import file_sigs
from litesearch import database
from litesearch.utils import FastEncode

db = database('.claude/code_index.db')
store = db.get_store()
encoder = FastEncode()

for path in Path('src').rglob('*.py'):
    sigs = file_sigs(str(path))
    rows = []
    for s in sigs:
        text = f'{s.get("signature", "")}\n{s.get("docstring", "")}'.strip()
        if not text:
            continue
        rows.append({
            'content': text,
            'embedding': encoder.encode_document(text).tobytes(),
            'metadata': json.dumps({'file': str(path), 'name': s.get('name', ''), 'kind': s.get('kind', 'function')}),
        })
    if rows:
        store.insert_all(rows)

# Now searchable
query = "email validation function"
results = db.search(query, encoder.encode_query(query).tobytes(), limit=5)
```

## Getting Just Signatures for LLM Context

```python
from codesigs import file_sigs

# Get compact signature list for a file
sigs = file_sigs('mymodule.py')
context = '\n'.join(s['signature'] for s in sigs)
print(context)
# def load_config(path: Path) -> dict:
# def save_config(config: dict, path: Path) -> None:
# class ConfigManager:
#     def __init__(self, path: Path):
#     def get(self, key: str, default=None):
```

## CSS Signatures

```python
from codesigs import css_sigs

sigs = css_sigs('styles.css')
# Returns CSS selectors as "signatures"
```

## Project Indexing (SessionStart)

The SessionStart hook automatically runs:
```
codesigs on all .py/.js/.ts/.rs/.go/.java files
    → litesearch index at .claude/code_index.db
```

The index is rebuilt on each Claude Code session start.

## Manual Rebuild

```bash
uv run python -m claude_plugins.hooks.index_hook
```

Or in Python:
```python
from claude_plugins.hooks.index_hook import build_index
from pathlib import Path

n = build_index(Path('.'), Path('.claude/code_index.db'))
print(f'Indexed {n} signatures')
```

## Use Cases

1. **LLM context**: Feed signatures instead of full file content — 10x less tokens
2. **Code search**: Combined with litesearch for semantic search
3. **Documentation**: Generate API docs from signatures + docstrings
4. **Refactoring**: Find all functions matching a pattern before renaming

## Installation

```bash
pip install codesigs
```

Repo: https://github.com/answerdotai/codesigs
