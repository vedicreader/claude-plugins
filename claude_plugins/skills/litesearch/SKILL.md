---
name: litesearch
description: Search the codebase index semantically and by keyword using litesearch. The index is built at session start via codesigs + litesearch. Use when you need to find functions, classes, or code patterns across the project.
user-invocable: true
allowed-tools: Bash, Read
---

# litesearch — Hybrid Code Search

litesearch combines SQLite FTS5 (full-text search) with vector similarity search (via usearch) using Reciprocal Rank Fusion reranking.

## Core Imports

```python
from litesearch import database
from litesearch.utils import FastEncode
from litesearch.data import pyparse, pkg2chunks, pre
```

## Creating a Database

```python
db = database('.claude/code_index.db')   # on-disk
db = database()                           # in-memory (default ':memory:')
db = database('.claude/code_index.db', wal=True, sem_search=True)
```

`database()` returns a `Database` object wrapping a fastlite SQLite DB with FTS5 and vector extensions registered.

## Creating a Store (FTS5 + Vector Table)

```python
store = db.get_store()               # default table name 'store'
store = db.get_store(name='code')    # custom table name
```

The store automatically has columns: `id` (PK), `content` (TEXT), `embedding` (BLOB), `metadata` (TEXT), `uploaded_at` (FLOAT).

## Encoding Text

```python
encoder = FastEncode()   # ONNX-based, no GPU needed, auto-downloads model

# For documents being stored (adds doc-side prompting)
emb_bytes = encoder.encode_document("def validate_email(addr: str) -> bool:").tobytes()

# For queries (adds query-side prompting)
emb_bytes = encoder.encode_query("email validation function").tobytes()
```

## Inserting Documents

```python
store.insert_all([
    {
        'content': 'def validate_email(addr: str) -> bool:\n    """Check email format."""',
        'embedding': encoder.encode_document('def validate_email...').tobytes(),
        'metadata': '{"file": "utils.py", "name": "validate_email", "kind": "function"}',
    },
    ...
])
```

## Searching

```python
query = "email validation function"
q_emb = encoder.encode_query(query).tobytes()

# Hybrid search with RRF reranking (default, recommended)
results = db.search(query, q_emb)
# → list of dicts with 'id', 'content', 'metadata', 'uploaded_at'

# With options
results = db.search(
    query,
    q_emb,
    columns=['id', 'content', 'metadata'],  # which columns to return
    limit=10,                                 # max results (default 50)
    emb_metric='cosine',                      # distance metric
    rrf=True,                                 # Reciprocal Rank Fusion (default True)
)
```

### `search()` Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `q` | str | required | Query text for FTS5 keyword search |
| `emb` | bytes | required | Query embedding (float16 numpy array `.tobytes()`) |
| `columns` | list | all | Columns to return |
| `where` | str | None | SQL WHERE clause |
| `where_args` | list | None | Parameters for WHERE clause |
| `limit` | int | 50 | Max results |
| `emb_metric` | str | `'cosine'` | `'cosine'`, `'euclidean'`, `'inner_product'`, `'divergence'` |
| `rrf` | bool | True | True = merged RRF list; False = `{'fts': [], 'vec': []}` |
| `dtype` | dtype | `np.float16` | Embedding dtype |

### Separate FTS / Vector Results

```python
results = db.search(query, q_emb, rrf=False)
fts_hits = results['fts']   # keyword matches
vec_hits = results['vec']   # semantic matches
```

## Query Preprocessing

```python
from litesearch.data import pre

# Expands query with wildcards, OR operators, keyword extraction
processed = pre("find email validation", wc=True, wide=True, extract_kw=True)
results = db.search(processed, q_emb)
```

## Indexing Python Code with codesigs

```python
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
    for sig in sigs:
        text = f'{sig.get("signature", "")}\n{sig.get("docstring", "")}'.strip()
        if not text:
            continue
        rows.append({
            'content': text,
            'embedding': encoder.encode_document(text).tobytes(),
            'metadata': json.dumps({
                'file': str(path),
                'name': sig.get('name', ''),
                'kind': sig.get('kind', 'function'),
            }),
        })
    if rows:
        store.insert_all(rows)
```

## Indexing Package Source Code

```python
from litesearch.data import pkg2chunks

# Extract code chunks from an installed package
chunks = pkg2chunks('fasthtml')   # chunks from the fasthtml package
store.insert_all([{
    'content': c['content'],
    'embedding': encoder.encode_document(c['content']).tobytes(),
    'metadata': json.dumps(c),
} for c in chunks])
```

## Parsing a Python File

```python
from litesearch.data import pyparse

chunks = pyparse('myfile.py')           # parse a file path
chunks = pyparse(code='def foo(): ...')  # parse code string
# Returns list of dicts with 'content' and metadata
```

## Session Index (Built Automatically)

The SessionStart hook builds `.claude/code_index.db` at the start of each Claude Code session by running `codesigs` on all `.py/.js/.ts/.rs/.go/.java` files (excluding `.venv/`, `.git/`, etc.).

To query it:
```python
from litesearch import database
from litesearch.utils import FastEncode

db = database('.claude/code_index.db')
encoder = FastEncode()

query = "function that handles authentication"
results = db.search(query, encoder.encode_query(query).tobytes(), limit=5)

for r in results:
    import json
    meta = json.loads(r.get('metadata', '{}'))
    print(meta.get('file'), meta.get('name'))
    print(r['content'][:100])
    print()
```

## Raw SQL Access

```python
# Execute arbitrary SQL on the database
results = list(db.q("SELECT id, content FROM store WHERE content LIKE ?", ["%email%"]))

# Returns AttrDict results
results = list(db.query("SELECT * FROM store LIMIT 5"))
```

## Installation

```bash
pip install litesearch
```

Repo: https://github.com/karthik777/litesearch
