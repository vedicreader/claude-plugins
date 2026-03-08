---
name: litesearch
description: Search the codebase index semantically and by keyword using litesearch. The index is built at session start via codesigs + litesearch. Use when you need to find functions, classes, or code patterns across the project.
user-invocable: true
allowed-tools: Bash, Read
---

# litesearch — Hybrid Code Search

litesearch combines SQLite FTS5 (keyword search) with vector embeddings (semantic search) using Reciprocal Rank Fusion for best-of-both results.

## Core API

```python
from litesearch import SearchDB

# Open (or create) the index
db = SearchDB('.claude/code_index.db')

# Hybrid search: semantic + keyword combined
results = db.search("function that validates email addresses", n=10)
for r in results:
    print(r['file'], r['name'], r['score'])

# Pure keyword search (FTS5)
results = db.search("validate_email", mode='fts', n=5)

# Pure semantic/vector search
results = db.search("input validation", mode='vector', n=5)
```

## Indexing Code

```python
# Add documents manually
db.add_documents([
    {
        'id': 'myfile.py::validate_email',
        'text': 'def validate_email(addr: str) -> bool:\n    """Check email format."""',
        'file': 'myfile.py',
        'name': 'validate_email',
        'kind': 'function',
    }
])

# Or index a whole directory with codesigs
from codesigs import file_sigs

for path in Path('src').rglob('*.py'):
    sigs = file_sigs(str(path))
    db.add_documents([{
        'id': f'{path}::{s["name"]}',
        'text': f'{s["signature"]}\n{s.get("docstring", "")}',
        'file': str(path),
        'name': s['name'],
        'kind': s.get('kind', 'function'),
    } for s in sigs])
```

## Search Result Fields

```python
result = {
    'id': 'path/to/file.py::function_name',
    'file': 'path/to/file.py',
    'name': 'function_name',
    'kind': 'function',      # or 'class', 'method'
    'text': 'signature + docstring',
    'score': 0.87,           # RRF fusion score
}
```

## Session Index

The SessionStart hook builds `.claude/code_index.db` automatically. It indexes:
- All `.py`, `.js`, `.ts`, `.jsx`, `.tsx`, `.rs`, `.go`, `.java` files
- Excludes `.venv/`, `.git/`, `__pycache__/`, `node_modules/`

Query it:
```python
db = SearchDB('.claude/code_index.db')
results = db.search("your query here")
```

## Distance Metrics

```python
# Cosine (default) — good for semantic similarity
db = SearchDB('.claude/code_index.db', metric='cosine')

# L2 distance
db = SearchDB('.claude/code_index.db', metric='l2')

# Dot product
db = SearchDB('.claude/code_index.db', metric='dot')
```

## PDF and Code Parsing

```python
from litesearch import parse_pdf, parse_code

# Parse a PDF into chunks for indexing
chunks = parse_pdf('document.pdf')

# Parse source code into semantic chunks
chunks = parse_code('src/main.py')
```

## Indexing for Claude

litesearch is how Claude understands your codebase. The index at `.claude/code_index.db` lets Claude:
1. Find relevant functions before making edits
2. Understand the project structure semantically
3. Answer questions about what exists in the codebase

## Installation

```bash
pip install litesearch
```

Repo: https://github.com/karthik777/litesearch
