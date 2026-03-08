---
name: monsterui
description: Build styled FastHTML UIs with MonsterUI — pre-styled Tailwind/FrankenUI components including cards, forms, navbars, modals, tables, and DaisyUI elements. Use when building or modifying FastHTML applications that need polished UI without writing CSS.
user-invocable: true
allowed-tools: Bash, Read, Glob
---

# MonsterUI — Styled Components for FastHTML

MonsterUI is a Python UI component library for FastHTML built on FrankenUI (Tailwind CSS + UIkit) and DaisyUI. Write production-quality web UIs entirely in Python — no CSS, no JSX.

## Setup

```python
from fasthtml.common import *
from monsterui.all import *

# Pick a theme and create app
hdrs = Theme.blue.headers()           # CDN-based (requires internet)
hdrs = Theme.blue.local_headers()     # local files (offline-safe)

app, rt = fast_app(hdrs=hdrs)

@rt
def index():
    return Titled("My App", Card(CardBody("Hello World")))

serve()
```

## Theme Colors (12 options)

```python
Theme.slate    Theme.stone   Theme.gray    Theme.neutral
Theme.red      Theme.rose    Theme.orange  Theme.green
Theme.blue     Theme.yellow  Theme.violet  Theme.zinc
```

### Theme Headers Options

```python
Theme.blue.headers(
    mode='auto',          # 'auto', 'light', 'dark'
    icons=True,           # include UIkit icons
    daisy=True,           # include DaisyUI
    highlightjs=False,    # code syntax highlighting
    katex=False,          # LaTeX math rendering
    apex_charts=False,    # ApexCharts JS library
)
```

---

## Layout Components

### Card

```python
Card(
    CardHeader(H3("Title"), P("Subtitle", cls=TextT.muted)),
    CardBody(
        P("Main content here"),
        Button("Action", cls=ButtonT.primary),
    ),
    CardFooter(Small("Footer text")),
)

# Simple card (no sub-components needed)
Card("Quick content", header="Title", footer=Button("OK"))
```

### Grid

```python
Grid(
    Card("Item 1"),
    Card("Item 2"),
    Card("Item 3"),
    cols=3,   # number of columns
)

# Responsive grid (auto-fit)
Grid(*items, cols_sm=1, cols_md=2, cols_lg=3)
```

### Container & Sections

```python
Container(content)          # max-width centered container
Section(content)            # page section with padding

# Div layout helpers
Div(content, cls=DivCentered)   # horizontally centered
Div(content, cls=DivVStacked)   # vertical stack
Div(content, cls=DivHStacked)   # horizontal stack
Div(content, cls=DivFullySpaced) # space-between flex
```

---

## Typography

```python
H1("Page Title")
H2("Section")
H3("Subsection")
H4("Sub-subsection")
H5("Small heading")
H6("Tiny heading")

Subtitle("Supporting text under a heading")
Em("italic")
Strong("bold")
Small("small text")
Mark("highlighted")
Del("strikethrough")
Ins("inserted")
Blockquote("A quote", cite="Author")
Caption("Table or figure caption")
CodeSpan("inline_code()")
CodeBlock("def foo():\n    return 42", lang='python')
```

### Text Styling Enums

```python
# Apply via cls= parameter
P("Muted text", cls=TextT.muted)
P("Bold", cls=TextT.bold)
P("Small", cls=TextT.sm)
P("Large", cls=TextT.lg)
P("Centered", cls=TextT.center)
P("Primary color", cls=TextT.primary)

# Preset combinations
H2("Hero", cls=TextPresets.hero)
P("Body", cls=TextPresets.body_sm)
```

---

## Form Components

### Labeled Inputs (label + input combined)

```python
LabelInput("Email", type="email", placeholder="user@example.com", id="email")
LabelInput("Password", type="password", id="pwd")
LabelInput("Name", value="John", id="name")

LabelSelect("Country",
    options=["USA", "UK", "Canada"],
    id="country"
)
# Or with Option() components:
LabelSelect("Role",
    Option("Admin", value="admin"),
    Option("User", value="user", selected=True),
    id="role"
)

LabelCheckboxX("Accept terms", id="terms")
LabelRange("Volume", min=0, max=100, value=50, id="vol")
```

### Raw Inputs

```python
Input(type="text", placeholder="Enter text", name="q")
TextArea("Default content", name="body", rows=5)
Select(Option("A"), Option("B"), name="choice")
Switch(checked=True, name="enabled")
Upload(name="file", accept=".pdf,.csv")
UploadZone("Drop files here", name="files", multiple=True)
```

### Form Layout

```python
Form(
    LabelInput("Name", id="name"),
    LabelInput("Email", type="email", id="email"),
    LabelSelect("Role", options=["admin", "user"], id="role"),
    Button("Submit", cls=ButtonT.primary),
    hx_post="/submit",
    hx_target="#result",
)
```

---

## Buttons

```python
Button("Click me")
Button("Primary", cls=ButtonT.primary)
Button("Danger", cls=ButtonT.destructive)
Button("Ghost", cls=ButtonT.ghost)
Button("Small", cls=ButtonT.sm)
Button("Loading...", cls=ButtonT.primary, disabled=True)

# LoaderButton: shows spinner while HTMX request is in flight
LoaderButton("Save", hx_post="/save", hx_target="#msg")

# Toggle button (on/off state)
ToggleBtn("Dark Mode", id="theme-toggle")
```

---

## Navigation

### NavBar (top navigation)

```python
NavBar(
    A("Home", href="/"),
    A("About", href="/about"),
    NavParentLi(
        "Dropdown",
        NavContainer(
            LiA("Option 1", href="/opt1"),
            LiA("Option 2", href="/opt2"),
        )
    ),
    brand=H3("My App"),
)
```

### Tabs

```python
TabContainer(
    Li(A("Tab 1", href="#tab1"), cls="uk-active"),
    Li(A("Tab 2", href="#tab2")),
    Li(A("Tab 3", href="#tab3")),
)
```

### ScrollSpy

```python
# Auto-highlights nav item based on scroll position
ScrollSpy(
    NavContainer(
        LiA("Section 1", href="#s1"),
        LiA("Section 2", href="#s2"),
    ),
    cls=ScrollspyT.bold,
)
```

---

## Modal

```python
# Trigger button
Button("Open Modal", uk_toggle="target: #my-modal")

# Modal component
Modal(
    ModalDialog(
        ModalHeader(H3("Modal Title"), Button(UkIcon("close"), cls="uk-modal-close")),
        ModalBody(P("Modal content here")),
        ModalFooter(
            Button("Cancel", cls="uk-modal-close"),
            Button("Confirm", cls=ButtonT.primary),
        ),
    ),
    id="my-modal",
)
```

---

## Tables

```python
# From list of dicts (auto-generates headers from keys)
data = [
    {"name": "Alice", "role": "Admin", "active": True},
    {"name": "Bob", "role": "User", "active": False},
]
TableFromDicts(data)

# With custom column selection
TableFromDicts(data, cols=["name", "role"])

# From separate headers + rows
TableFromLists(
    headers=["Name", "Role"],
    rows=[["Alice", "Admin"], ["Bob", "User"]],
)

# Raw Table
Table(
    Thead(Tr(Th("Name"), Th("Role"))),
    Tbody(
        Tr(Td("Alice"), Td("Admin")),
        Tr(Td("Bob"), Td("User")),
    ),
)
```

---

## DaisyUI Components

### Alert

```python
Alert("Success!", cls=AlertT.success)
Alert("Warning!", cls=AlertT.warning)
Alert("Error!", cls=AlertT.error)
Alert("Info message", cls=AlertT.info)
```

### Loading Spinners

```python
Loading()                           # default spinner
Loading(cls=LoadingT.spinner)
Loading(cls=LoadingT.dots)
Loading(cls=LoadingT.ring)
Loading(cls=LoadingT.ball)
Loading(cls=LoadingT.bars)
Loading(cls=LoadingT.infinity)
Loading(cls=LoadingT.lg)            # large
Loading(cls=LoadingT.sm)            # small
```

### Toast Notifications

```python
Toast(
    Alert("Saved!", cls=AlertT.success),
    cls=(ToastHT.end, ToastVT.top),   # position: top-right
)
# Positions: ToastHT.start/center/end, ToastVT.top/middle/bottom
```

### Steps (Progress Indicator)

```python
Steps(
    LiStep("Account", cls="uk-active"),
    LiStep("Profile"),
    LiStep("Payment"),
    LiStep("Confirm"),
)
```

---

## Icons

```python
UkIcon("heart")                    # UIkit icon by name
UkIcon("check", ratio=2)           # 2x size
UkIcon("close", cls="uk-text-danger")

# Common icon names:
# check, close, plus, minus, trash, pencil, search, settings,
# home, user, mail, phone, lock, unlock, star, heart,
# arrow-left, arrow-right, arrow-up, arrow-down,
# triangle-up, triangle-down, warning, info, question
```

---

## Markdown Rendering

```python
from monsterui.all import render_md

html = render_md("# Hello\n\nThis is **markdown** with `code`.")
# Returns FastHTML FT element (safe to use directly in routes)

@rt('/docs')
def get():
    md = Path('README.md').read_text()
    return Titled("Docs", render_md(md))
```

Enable extras:
```python
# In Theme headers:
hdrs = Theme.blue.headers(highlightjs=True, katex=True)
```

---

## Charts (ApexCharts)

```python
hdrs = Theme.blue.headers(apex_charts=True)

ApexChart(
    options={
        "chart": {"type": "line"},
        "series": [{"name": "Sales", "data": [30, 40, 35, 50]}],
        "xaxis": {"categories": ["Jan", "Feb", "Mar", "Apr"]},
    },
    id="my-chart",
)
```

---

## ThemePicker (Dev Tool)

```python
# Add a floating theme switcher (useful during development)
ThemePicker()
```

---

## Progress Bar

```python
Progress(value=75, max=100)
Progress(value=30, cls="uk-progress-success")
```

---

## Accordion / Details

```python
Accordion(
    Details(Summary("Section 1"), P("Content 1")),
    Details(Summary("Section 2"), P("Content 2")),
)

Details(Summary("Click to expand"), P("Hidden content"), open=True)
```

---

## Complete App Example

```python
from fasthtml.common import *
from monsterui.all import *

app, rt = fast_app(hdrs=Theme.blue.headers())

todos = []

@rt
def index():
    return Titled("Todo App",
        Container(
            Card(
                CardHeader(H2("My Todos")),
                CardBody(
                    Form(
                        LabelInput("New todo", id="task", placeholder="Add a task..."),
                        Button("Add", cls=ButtonT.primary, hx_post="/add", hx_target="#list"),
                    ),
                    Div(id="list", *[
                        Div(
                            CheckboxX(checked=False),
                            Span(t),
                            cls=DivHStacked,
                        ) for t in todos
                    ]),
                ),
            )
        )
    )

@rt
def add(task: str):
    todos.append(task)
    return Div(*[
        Div(CheckboxX(checked=False), Span(t), cls=DivHStacked)
        for t in todos
    ])

serve()
```

## Installation

```bash
pip install MonsterUI
```

Requires: `python-fasthtml`

Repo: https://github.com/answerdotai/MonsterUI
