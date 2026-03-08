---
name: nbdev
description: Work with nbdev notebook-driven development. Use when developing in an nbdev project — exporting code from notebooks, running tests, building docs, or understanding the nbdev workflow.
user-invocable: true
allowed-tools: Bash, Read, Glob
---

# nbdev — Notebook-Driven Development

nbdev turns Jupyter notebooks into production-ready Python packages with integrated docs and tests. The notebooks in `nbs/` are the **source of truth** — never edit the exported Python files directly.

## Core Workflow

```
Edit nbs/*.ipynb → nbdev-export → Python package
                 → nbdev-test   → Run tests
                 → nbdev-docs   → Build docs site
```

## Key CLI Commands

```bash
# Export all notebooks to Python (run after every notebook change)
nbdev-export

# Run all tests in notebooks
nbdev-test

# Build documentation site
nbdev-docs

# Do everything: export + test + clean notebooks
nbdev-prepare

# Clean notebook metadata (reduces git noise)
nbdev-clean

# Install git + Jupyter hooks for this repo
nbdev-install-hooks

# Create a new nbdev project
nbdev-new

# Publish to PyPI
nbdev-pypi
```

## Cell Directives

Control what gets exported from each cell:

```python
#| default_exp mymodule       # Set the module for this notebook's exports
#| export                     # Export this cell to the Python module
#| exporti                    # Export but don't show in docs
#| hide                       # Don't show in docs, don't export
#| hide_line                  # Hide just this line in docs
```

## Project Structure

```
myproject/
├── nbs/
│   ├── index.ipynb          # Package homepage (becomes README.md)
│   ├── 00_core.ipynb        # Exports to myproject/core.py
│   ├── 01_utils.ipynb       # Exports to myproject/utils.py
│   └── ...
├── myproject/               # AUTO-GENERATED — don't edit directly
│   ├── __init__.py
│   ├── core.py
│   └── utils.py
├── settings.ini             # nbdev config
├── pyproject.toml           # or setup.py
└── docs/                    # AUTO-GENERATED
```

## settings.ini

```ini
[DEFAULT]
lib_name = myproject
user = myusername
description = My package description
branch = main
version = 0.1.0
min_python = 3.11
nbs_path = nbs
lib_path = myproject
doc_path = docs
```

## Writing Tests in Notebooks

Tests go in non-exported cells (no `#| export`):

```python
# This cell runs as a test but is not exported
result = my_function(42)
assert result == expected, f"Got {result}"
print("test passed")
```

Use `assert` statements or `fastcore.test` helpers:
```python
from fastcore.test import test_eq, test_fail
test_eq(my_function(2), 4)
test_fail(lambda: my_function('bad'), contains='TypeError')
```

## Documentation

nbdev uses Quarto under the hood. Add markdown between code cells for documentation.

Docstrings become API docs automatically:
```python
#| export
def my_func(x: int, y: int) -> int:
    "Add two numbers together. Returns their sum."
    return x + y
```

## Hooks

`nbdev-install-hooks` sets up:
- **Git pre-commit**: auto-cleans notebooks before commit
- **Jupyter save hook**: auto-cleans on save
- Both reduce merge conflicts in .ipynb files

## IMPORTANT: This Project Uses nbdev

In this project (`claude-plugins`):
- **Edit `nbs/*.ipynb`** — not `claude_plugins/*.py`
- Run `nbdev-export` after any notebook change
- Run `nbdev-prepare` before committing
- The `claude_plugins/` directory is auto-generated

## Installation

```bash
pip install nbdev
nbdev-install-hooks  # In each repo
```

Docs: https://nbdev.fast.ai
