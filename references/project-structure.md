# Project Structure

> `skill-dev-standard` v2.2.0 — §3: directory layouts, naming conventions, assets/ spec

---

## 3.1 CLI SKILL Structure (Flat — recommended for single-file scenarios)

```
<skill-name>/
├── SKILL.md                    # SKILL metadata and execution rules (required)
├── README.md                   # User documentation (strongly recommended)
├── .env.example                # Environment variable template (if needed)
├── docs/                       # CLI tool's own reference materials (not a SKILL knowledge base)
│   ├── api.md                 # Upstream API doc summary
│   └── patterns.md            # Usage patterns
├── assets/                     # Optional: static content for output (templates, config boilerplates, payload samples)
└── scripts/
    └── <skill_name>.py         # CLI entry point (with inlined lib)
```

> **Note**: CLI SKILLs use `docs/` not `references/`, to avoid confusion with the docs SKILL knowledge base directory.

---

## 3.2 CLI SKILL Structure (Modular — recommended for multi-command scenarios)

```
<skill-name>/
├── SKILL.md
├── README.md
├── pyproject.toml              # uv/pip package config
├── docs/
│   └── <docs>.md
├── assets/                     # Optional: static content for output
├── scripts/
│   └── <skill_name>_cli.py     # shim entry script
└── <skill_name>_tools/         # standalone package
    ├── __init__.py
    ├── cli.py                  # main CLI
    ├── commands/               # one file per command
    │   ├── get.py
    │   └── set.py
    ├── client.py               # HTTP / SDK client
    ├── config.py               # configuration management
    ├── schemas.py              # output schemas (TypedDict)
    └── errors.py               # error code definitions
```

---

## 3.3 Docs SKILL Structure

```
<skill-name>/
├── SKILL.md                    # required
├── README.md                   # required
├── assets/                     # optional: images, data samples, and other output resources
└── references/                 # core knowledge base (docs SKILL-exclusive directory name)
    ├── concepts.md            # concepts and terminology
    ├── patterns.md            # patterns and best practices
    ├── examples.md            # runnable examples
    ├── troubleshooting.md     # troubleshooting guides
    ├── api-reference.md       # API index
    └── output-schema.json     # optional: output structure declaration
```

---

## 3.4 Hybrid SKILL Structure

```
<skill-name>/
├── SKILL.md
├── README.md
├── pyproject.toml
├── references/                 # knowledge base (hybrid SKILLs keep references/)
│   ├── guide.md
│   └── api.md
├── assets/                     # optional: static content for output
├── docs/                       # CLI tool reference materials
└── scripts/
    └── <skill_name>.py
```

---

## 3.5 assets/ Directory (Optional)

`assets/` is the fourth Bundled Resource type defined by the official skill-creator: **"Files used in output (templates, icons, fonts)"**. It stores **static files used as raw material during SKILL execution** — the Agent reads them and embeds, renders, or delivers them to the user, rather than using them for knowledge understanding or code execution.

### Four-Directory Boundary

| Directory | Content type | Agent action |
|-----------|-------------|-------------|
| `scripts/` | Executable code | Execute (`bash` / `python` run, get results) |
| `references/` | Knowledge documents | Read and understand (Read, extract knowledge to guide generation) |
| `docs/` | Tool reference materials | Read and understand (Read, look up interface specs) |
| `assets/` | **Static content resources** | **Read raw content, replace placeholders / embed in docs / deliver directly to user** |

### Use Cases

**Files that belong in assets/**:

| Type | Typical example | Notes |
|------|----------------|-------|
| Output UI templates | `report-template.html` | Contains `__PLACEHOLDER__`; Agent replaces and renders to a temp file |
| Config boilerplates | `oauth-template.json`, `jco-props.template` | Contains `<placeholder>`; user copies and fills in real values |
| API payload samples | `po-create.json`, `idoc-orders05.xml` | Request body reference for upstream APIs; Agent shows or references as needed |
| Image assets | `icon.svg`, `logo.png` | Icons or diagrams to embed in output docs (prefer `.svg`) |
| Font files | `NotoSansSC.ttf` | Fonts bundled for doc-generation SKILLs; prefer CDN when possible |

**Files that do NOT belong in assets/**:

| Wrong placement | Correct location |
|----------------|-----------------|
| Executable scripts (`.py`, `.sh`) | `scripts/` |
| API docs, best practice guides | `references/` or `docs/` |
| Test fixtures / test data | `tests/fixtures/` |
| Eval workspace artifacts (`benchmark.json`, `meta.json`) | Workspace directory (outside the SKILL directory); not distributed with the SKILL |
| Documentation that requires reading to use | `references/` |

### Placeholder Convention

Runtime substitution variables in template files use `__VARIABLE_NAME__` double-underscore format (aligned with official skill-creator):

```html
<!-- assets/report-template.html -->
<title>__REPORT_TITLE__</title>
<p>Generated: __GENERATED_DATE__</p>
```

Corresponding replacement logic in `scripts/`:

```python
from pathlib import Path

def render_asset(name: str, **vars: str) -> str:
    """Read an assets/ template, replace __KEY__ placeholders, return as string."""
    path = Path(__file__).parent.parent / "assets" / name
    content = path.read_text(encoding="utf-8")
    for key, value in vars.items():
        content = content.replace(f"__{key.upper()}__", value)
    return content
```

Config boilerplates may use `<placeholder>` notation (more human-readable); do not mix the two styles — document which convention you use in SKILL.md.

### Explicit Reference Required in SKILL.md

Every file in `assets/` must have a corresponding usage note in SKILL.md, covering: **when to read it, how to substitute or deliver it to the user**. Orphaned assets (not referenced in SKILL.md) are redundant and should be deleted or moved.

Recommended pattern in SKILL.md — list all assets in a table:

```markdown
## Scripts and Resources

| Resource | Purpose |
|---------|---------|
| `assets/configs/oauth-template.json` | OAuth2 config boilerplate; display to user during init, guide them to replace `<placeholder>` fields |
| `assets/payloads/po-create.json`     | Purchase order creation payload sample; reference directly during debugging or documentation |
| `scripts/example_skill.py`           | CLI entry point; run `--help` to see subcommands |
```

### Size and Format Constraints

| Constraint | Recommended | Reason |
|-----------|-------------|--------|
| Single file size | < 100 KB | SKILL packaged as zip; large files slow installation |
| `assets/` total size | < 500 KB | Keep SKILL package lightweight |
| Image format | Prefer `.svg` | Text format, diffable, small size |
| Font files | Use sparingly (< 200 KB/file) | Large; use only when necessary, otherwise prefer CDN |
| Binary files (`.docx` etc.) | Acceptable but use carefully | Prefer generating dynamically via script; templates acceptable when necessary |

### Security Requirements

- **No real credentials**: assets files are distributed with the SKILL package — no real API tokens, passwords, or private keys
- **Use placeholders in templates**: `<your-client-secret>` ✓, `AKIAIOSFODNN7EXAMPLE` (real format) ✗
- **Payload samples use synthetic data**: `VIN001234`, `OAUTH_EXT_CLIENT`, and other synthetic identifiers
- **XSS protection**: if assets content is inserted into HTML, the `scripts/` side handles escaping; no unescaped injection points in templates

### Directory Structure Example

```
<skill-name>/
├── SKILL.md
├── scripts/
│   └── <skill_name>.py
├── references/
│   └── api.md
└── assets/                            # optional; create subdirectories by scenario
    ├── report-template.html           # HTML output template (__PLACEHOLDER__ style)
    ├── configs/
    │   ├── oauth-template.json        # OAuth2 config boilerplate (<placeholder> style)
    │   └── jco-props.template         # JCo connection properties template
    └── payloads/
        ├── po-create.json             # Purchase order creation payload sample
        └── idoc-orders05.xml          # IDoc message sample
```

> **Tip**: Subdirectories by scenario (`configs/`, `payloads/`, `templates/`) improve readability but are not required. Single files can go directly in the `assets/` root.
