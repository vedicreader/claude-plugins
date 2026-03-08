---
name: fasthtml
description: Build web applications with FastHTML — a Python-native HTMX framework. Use when building or modifying FastHTML web apps, routes, components, or HTMX interactions.
user-invocable: true
allowed-tools: Bash, Read, Glob
---

# FastHTML — Python-Native Web Framework

FastHTML builds fast, scalable web applications using HTMX and hypermedia patterns. No JavaScript frameworks needed — Python generates HTML directly.

## Quick Start

```python
from fasthtml.common import *

app, rt = fast_app()

@rt('/')
def get():
    return Titled('Hello', P('World'))

serve()
```

## Core Concepts

### `fast_app()` — Create App
```python
app, rt = fast_app(
    hdrs=(Style(':root { font-family: sans-serif }'),),  # extra headers
    live=True,   # live reload in development
    debug=True,  # show errors in browser
)
```

### Route Decorators
```python
@rt('/users')
def get():          # GET /users
    return ...

@rt('/users')
def post(name: str):  # POST /users (auto-parses form/query params)
    return ...

@rt('/users/{id}')
def get(id: int):   # GET /users/123
    return ...
```

### HTML Elements as Python Functions

Every HTML tag is a Python function:
```python
Div(P("hello"), cls="container")  # <div class="container"><p>hello</p></div>
Button("Click me", hx_post="/api", hx_target="#result")
Form(Input(name="q"), Button("Submit"), action="/search", method="post")
```

### HTMX Attributes
```python
# Any hx_* attr becomes hx-* in HTML
Button("Load", hx_get="/data", hx_target="#content", hx_swap="innerHTML")
Div(id="content")  # target for HTMX swap

# Trigger on input
Input(hx_post="/search", hx_trigger="keyup changed delay:300ms", hx_target="#results")
```

### `Titled()` — Page with Title
```python
@rt('/')
def get():
    return Titled(
        'My App',
        H2('Welcome'),
        P('Content here'),
    )
```

### Forms
```python
@rt('/submit')
def post(name: str, email: str):
    # Auto-parses form fields
    return P(f'Hello {name}!')

@rt('/submit')
def get():
    return Form(
        Label('Name:', Input(name='name')),
        Label('Email:', Input(name='email', type='email')),
        Button('Submit'),
        method='post', action='/submit'
    )
```

## Database with MiniDataAPI

```python
from fasthtml.common import *

app, rt, todos, Todo = fast_app(
    'todos.db',
    todo={'id': int, 'title': str, 'done': bool},
)

@rt('/todos')
def get():
    return Ul(*[Li(t.title) for t in todos()])

@rt('/todos')
def post(todo: Todo):
    return todos.insert(todo)
```

## Serving

```python
serve()          # Default: localhost:5001
serve(port=8080) # Custom port
```

## Key Patterns

### HTMX Partial Updates
```python
@rt('/item/{id}')
def delete(id: int):
    items.delete(id)
    return ''  # Return empty to remove element with hx_swap='outerHTML'

# In UI:
Li(item.name, Button("X", hx_delete=f"/item/{item.id}", hx_swap="outerHTML", hx_target="closest li"))
```

### WebSockets / SSE
```python
from fasthtml.common import *

app, rt = fast_app()

@app.ws('/ws')
async def ws(msg: str, send):
    await send(Div(f'Echo: {msg}', id='messages'))
```

## Installation

```bash
pip install python-fasthtml
```

Repo: https://github.com/answerdotai/fasthtml
