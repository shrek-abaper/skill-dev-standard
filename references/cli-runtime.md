# CLI Runtime Standards

> `skill-dev-standard` v2.2.0 — §6/§7/§8: configuration management, error handling, Output Schema

---

## 6. Configuration Management

### 6.1 Config Priority

```
CLI args > env vars > config file > defaults
```

### 6.2 Config Storage Options

| Option | Security | Use Case |
|--------|----------|----------|
| JSON file (0600 permissions) | Medium | Internal tools, dev environments |
| System keychain (keyring) | High | Production, sensitive credentials |
| Environment variables | Medium-high | CI/CD, containerized deployments |

### 6.3 Config Class Template

```python
import json
import os
from dataclasses import dataclass, asdict
from pathlib import Path
from typing import Optional

@dataclass
class SkillConfig:
    url: str
    token: Optional[str] = None
    verify_ssl: bool = True
    timeout: int = 30

    @classmethod
    def from_file(cls, path: Path = Path.home() / ".<skill-name>/config.json"):
        if not path.exists():
            raise FileNotFoundError(
                f"Config file not found: {path}\nRun: <skill-name> init"
            )
        with open(path, "r", encoding="utf-8") as f:
            return cls(**json.load(f))

    @classmethod
    def from_env(cls) -> Optional["SkillConfig"]:
        url = os.getenv("SKILL_URL")
        if not url:
            return None
        return cls(
            url=url,
            token=os.getenv("SKILL_TOKEN"),
            verify_ssl=os.getenv("SKILL_VERIFY_SSL", "true").lower() == "true",
        )

    @classmethod
    def load(cls) -> "SkillConfig":
        """Env vars take priority over config file."""
        return cls.from_env() or cls.from_file()

    def save(self, path: Path = Path.home() / ".<skill-name>/config.json"):
        path.parent.mkdir(parents=True, exist_ok=True)
        with open(path, "w", encoding="utf-8") as f:
            json.dump(asdict(self), f, indent=2, ensure_ascii=False)
        path.chmod(0o600)
```

---

## 7. Error Handling

### 7.1 Output Stream Conventions

| Content | Stream | Notes |
|---------|--------|-------|
| Normal results | stdout | Primary data, consumed by Agent or pipeline |
| Errors / warnings | stderr | Exception messages |
| Debug logs | stderr | Emitted in `--debug` mode |

### 7.2 Exit Code Conventions

All SKILLs use a unified exit code convention, declared in the SKILL.md "Error Handling" table:

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Config error (missing, invalid format) |
| 2 | Network error (timeout, connection failure) |
| 3 | Permission error |
| 4 | Argument error |
| 5 | Resource not found (returning 0 + empty result may be more appropriate in some scenarios) |
| 99 | Unexpected error |

### 7.3 Error Output Format

**Always output errors as JSON** (machine-friendly, Agent-parseable):

```python
# errors.py
import json, sys

def fail(message: str, code: int = 99, **extra):
    """Output standard error JSON to stderr and exit."""
    payload = {"status": "error", "code": code, "message": message}
    payload.update(extra)
    print(json.dumps(payload, ensure_ascii=False), file=sys.stderr)
    sys.exit(code)

def warn(message: str, **extra):
    """Output warning to stderr without exiting."""
    payload = {"status": "warning", "message": message}
    payload.update(extra)
    print(json.dumps(payload, ensure_ascii=False), file=sys.stderr)
```

### 7.4 High-Risk Operation Guards

```python
@cli.command("delete")
@click.argument("name")
@click.option("--yes", "-y", is_flag=True, help="Skip confirmation")
@click.option("--dry-run", is_flag=True, help="Simulate execution")
def delete_cmd(name, yes, dry_run):
    """Delete a resource (irreversible operation)."""
    if dry_run:
        click.echo(json.dumps({"status": "dry-run", "would_delete": name}))
        return
    if not yes:
        click.confirm(f"Delete {name}? This operation is irreversible.", abort=True)
    # perform deletion
    click.echo(json.dumps({"status": "ok", "deleted": name}))
```

---

## 8. Output Schema

**Why Output Schema matters**: when multiple SKILLs are composed, downstream SKILLs need to know upstream output structure. Schema is the source of truth for inter-SKILL contracts, reducing hallucination errors during Agent composition.

### 8.1 Declare in Code (TypedDict, recommended)

```python
# schemas.py
from typing import TypedDict, List, Optional

class ResourceItem(TypedDict):
    name: str
    value: str
    status: str

class ListResult(TypedDict):
    items: List[ResourceItem]
    count: int

class ErrorResult(TypedDict):
    status: str       # fixed value "error"
    code: int
    message: str
```

### 8.2 Declare in SKILL.md Frontmatter

Place `output_schema` under `metadata`:

```yaml
---
name: example-skill
description: "Use this skill when... Do NOT use for..."
metadata:
  version: "1.0.0"
  output_schema:
    format: json          # json | text | mixed
    success_fields:
      - name: status       # fixed "ok"
      - name: items        # List[ResourceItem]
      - name: count        # int
    error_fields:
      - name: status       # fixed "error"
      - name: code         # int, see §7.2
      - name: message      # str
---
```

### 8.3 Use JSON Schema File for Complex Scenarios

```json
// references/output-schema.json
{
  "$schema": "http://json-schema.org/draft-07/schema",
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "status": { "const": "ok" },
        "items": { "type": "array", "items": { "$ref": "#/$defs/ResourceItem" } },
        "count": { "type": "integer" }
      },
      "required": ["status", "items", "count"]
    },
    {
      "type": "object",
      "properties": {
        "status": { "const": "error" },
        "code": { "type": "integer" },
        "message": { "type": "string" }
      },
      "required": ["status", "code", "message"]
    }
  ],
  "$defs": {
    "ResourceItem": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "value": { "type": "string" },
        "status": { "type": "string" }
      }
    }
  }
}
```
