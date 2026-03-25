# Alap Expression Parser — Python

Server-side Python port of the Alap expression parser (`src/core/ExpressionParser.ts`). Use this to resolve Alap expressions in Python servers without a Node.js sidecar.

## What's included

- **`expression_parser.py`** — Recursive descent parser for the Alap expression grammar, macro expansion, regex search, config merging
- **`validate_regex.py`** — ReDoS guard for user-supplied regex patterns

## What's NOT included

This is the server-side subset of `alap/core`. It covers expression parsing, config merging, and regex validation — everything a server needs to resolve cherry-pick and query requests.

Browser-side concerns (DOM rendering, menu positioning, event handling, URL sanitization) are handled by the JavaScript client and are not ported here.

## Supported expression syntax

```
item1, item2              # item IDs (comma-separated)
.coffee                   # tag query
.nyc + .bridge            # AND (intersection)
.nyc | .sf                # OR (union)
.nyc - .tourist           # WITHOUT (subtraction)
(.nyc | .sf) + .open      # parenthesized grouping
@favorites                # macro expansion
/mypattern/               # regex search (by pattern key)
/mypattern/lu             # regex with field options
```

## Usage

```python
from expression_parser import ExpressionParser, resolve_expression, cherry_pick_links, merge_configs

config = {
    "allLinks": {
        "item1": {"label": "Example", "url": "https://example.com", "tags": ["demo"]},
        "item2": {"label": "Other",   "url": "https://other.com",   "tags": ["demo", "test"]},
    },
    "macros": {
        "all": {"linkItems": ".demo"}
    }
}

# Low-level: get matching IDs
parser = ExpressionParser(config)
ids = parser.query(".demo")              # ["item1", "item2"]
ids = parser.query(".demo - .test")      # ["item1"]

# Convenience: expression -> full link objects
results = resolve_expression(config, ".demo")
# [{"id": "item1", "label": "Example", ...}, {"id": "item2", ...}]

# Cherry-pick: expression -> { id: link } dict
subset = cherry_pick_links(config, ".test")
# {"item2": {"label": "Other", ...}}

# Merge multiple configs
merged = merge_configs(config1, config2)
```

## Installation

Copy `expression_parser.py` and `validate_regex.py` into your project, or install from PyPI:

```bash
pip install alap-core
# or, with uv (recommended):
uv add alap-core
```

## Used by

- [flask-sqlite](../../../examples/servers/flask-sqlite/) server
- [fastapi-postgres](../../../examples/servers/fastapi-postgres/) server
- [django-sqlite](../../../examples/servers/django-sqlite/) server
