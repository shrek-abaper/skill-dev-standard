# CLI SKILL Development Templates

> `skill-dev-standard` v2.2.0 — §4 series: core conventions, dependency management, Click/Typer full templates, multi-language comparison

---

## 4.0 Core Conventions (Language-Agnostic)

The following conventions apply to all CLI SKILLs regardless of implementation language. Python, Bash, Node.js, and Go implementations must all comply:

- Normal results → JSON on stdout
- Errors → JSON on stderr
- Exit codes follow the §7.2 spec (see `references/cli-runtime.md`)
- Write operations must support `--dry-run` and `--yes`
- Config load priority: CLI args > env vars > file > defaults
- Help text available via `--help`

The templates in this section use Python + Click as the primary example (mature ecosystem, largest community). See §4.6 for equivalents in other languages.

---

## 4.1 Dependency Management

**Recommended: `uv` (2025 Python toolchain standard)**:

```toml
# pyproject.toml
[project]
name = "example-skill"
version = "1.0.0"
dependencies = [
    "click>=8.1.0",
    "requests>=2.31.0",
]

[tool.uv]
python = ">=3.11"
```

Installation:
```bash
uv pip install -e .          # dev install
uvx --from . <skill-name>    # isolated run (recommended)
```

**For runtime dynamic dependency installation**, use `uv` instead of `pip` (faster, no system environment pollution):

```python
import subprocess, sys

def _ensure_deps():
    """Install dependencies on demand at runtime (confirm proxy availability in private network environments)."""
    import importlib.util
    deps = [("click", "click>=8.1.0"), ("requests", "requests>=2.31.0")]
    missing = [spec for mod, spec in deps if not importlib.util.find_spec(mod)]
    if missing:
        subprocess.run(
            ["uv", "pip", "install", "--quiet"] + missing,
            check=True
        )
```

> **Note**: Dynamic installation may be restricted by proxy or mirror settings in enterprise private network environments. If this is an issue, document the pre-installation steps under "Prerequisites" in `SKILL.md` and handle them manually or via CI.

---

## 4.2 CLI Framework Selection

| Scenario | Framework | Rationale |
|----------|-----------|-----------|
| All scenarios (default) | **Click** | Declarative, mature, industry standard |
| New projects / type-annotation-first | **Typer** | Based on type hints, modern API |
| Highly custom parsing required | argparse | Flexible but verbose — avoid unless necessary |

---

## 4.3 Click Standard Template

```python
#!/usr/bin/env python3
"""<skill-name> CLI"""

import json
import sys
import click
from pathlib import Path

# ── version ───────────────────────────────────────────────────────────────────

VERSION = "1.0.0"

def _version_callback(ctx, param, value):
    if not value or ctx.resilient_parsing:
        return
    click.echo(f"<skill-name> {VERSION}")
    ctx.exit()

# ── main CLI group ─────────────────────────────────────────────────────────────

@click.group()
@click.option("--debug", is_flag=True, envvar="SKILL_DEBUG", help="Enable debug mode")
@click.option("--version", "-v", is_flag=True, callback=_version_callback,
              expose_value=False, is_eager=True, help="Show version")
@click.pass_context
def cli(ctx, debug):
    """<Brief skill description>

    Usage examples:
        <skill-name> get <resource>
        <skill-name> list --filter active
        <skill-name> set <resource> <value> --dry-run
    """
    ctx.ensure_object(dict)
    ctx.obj["debug"] = debug

# ── init ──────────────────────────────────────────────────────────────────────

@cli.command("init")
@click.option("--url", prompt="Service URL", envvar="SKILL_URL", help="Service URL")
@click.option("--token", prompt="Auth token", hide_input=True, envvar="SKILL_TOKEN")
def init_cmd(url, token):
    """Initialize config file."""
    config_dir = Path.home() / ".<skill-name>"
    config_dir.mkdir(exist_ok=True)
    config_file = config_dir / "config.json"
    with open(config_file, "w") as f:
        json.dump({"url": url, "token": token}, f, indent=2)
    config_file.chmod(0o600)
    click.echo(json.dumps({"status": "ok", "config": str(config_file)}))

# ── read operations ────────────────────────────────────────────────────────────

@cli.command("get")
@click.argument("name")
@click.option("--format", "-f", type=click.Choice(["json", "text"]), default="json")
@click.pass_context
def get_cmd(ctx, name, format):
    """Fetch the resource with the given NAME."""
    if ctx.obj["debug"]:
        click.echo(f"[DEBUG] get: {name}", err=True)
    result = {"name": name, "value": "example", "status": "ok"}
    _output(result, format)

@cli.command("list")
@click.option("--filter", "-f", "filter_", help="Filter condition")
@click.option("--format", type=click.Choice(["json", "text"]), default="json")
def list_cmd(filter_, format):
    """List all resources."""
    results = [{"name": "a"}, {"name": "b"}]
    _output({"items": results, "count": len(results)}, format)

# ── write operations (with guards) ────────────────────────────────────────────

@cli.command("set")
@click.argument("name")
@click.argument("value")
@click.option("--yes", "-y", is_flag=True, help="Skip confirmation")
@click.option("--dry-run", is_flag=True, help="Simulate execution without actual writes")
@click.pass_context
def set_cmd(ctx, name, value, yes, dry_run):
    """Set a resource (write op, requires confirmation by default)."""
    if dry_run:
        _output({"status": "dry-run", "would_set": {name: value}}, "json")
        return
    if not yes and not click.confirm(f"Confirm setting {name} to {value}?"):
        click.echo(json.dumps({"status": "cancelled"}))
        return
    # actual write logic
    _output({"status": "ok", "name": name, "value": value}, "json")

# ── utility functions ─────────────────────────────────────────────────────────

def _output(data: dict, format: str = "json"):
    """Unified output: normal → stdout, error → stderr + exit 1."""
    if format == "json":
        click.echo(json.dumps(data, ensure_ascii=False))
    else:
        for k, v in data.items():
            click.echo(f"{k}: {v}")

def _fail(msg: str, code: int = 1):
    click.echo(json.dumps({"status": "error", "message": msg}), err=True)
    sys.exit(code)

if __name__ == "__main__":
    cli()
```

---

## 4.4 Typer Standard Template

```python
#!/usr/bin/env python3
"""<skill-name> CLI — Typer version"""

from typing import Annotated, Optional
import typer, json
from pathlib import Path

app = typer.Typer(help="<Brief skill description>", no_args_is_help=True)

@app.command()
def init(
    url: Annotated[str, typer.Option(prompt="Service URL", envvar="SKILL_URL")],
    token: Annotated[str, typer.Option(prompt="Auth token", hide_input=True, envvar="SKILL_TOKEN")],
):
    """Initialize config file."""
    config_dir = Path.home() / ".<skill-name>"
    config_dir.mkdir(exist_ok=True)
    config_file = config_dir / "config.json"
    with open(config_file, "w") as f:
        json.dump({"url": url, "token": token}, f, indent=2)
    config_file.chmod(0o600)
    typer.echo(json.dumps({"status": "ok", "config": str(config_file)}))

@app.command()
def get(
    name: Annotated[str, typer.Argument(help="Resource name")],
    format: str = typer.Option("json", "--format", "-f"),
    debug: bool = typer.Option(False, "--debug"),
):
    """Fetch a resource by name."""
    if debug:
        typer.echo(f"[DEBUG] get: {name}", err=True)
    result = {"name": name, "value": "example", "status": "ok"}
    typer.echo(json.dumps(result, ensure_ascii=False))

if __name__ == "__main__":
    app()
```

---

## 4.5 Click Best Practices Quick Reference

| Practice | Pattern |
|----------|---------|
| Type constraints | `type=click.Choice([...])`, `type=click.Path(exists=True)` |
| Env var binding | `envvar="VAR_NAME"` or global `auto_envvar_prefix="APP"` |
| Shared state via Context | `@click.pass_context` + `ctx.obj` |
| Version callback | `is_eager=True` ensures `--version` is handled first |
| Confirmation dialog | `click.confirm("Confirm?", abort=True)` |
| Error output | `click.echo(msg, err=True)` |
| Progress bar | `click.progressbar(items)` |
| Paged output | `click.echo_via_pager(long_text)` |

---

## 4.6 Equivalents in Other Languages

| Language | Recommended CLI Framework | Test Framework |
|----------|--------------------------|----------------|
| Python | Click (default) / Typer | pytest + click.testing.CliRunner |
| Bash | native + getopt | Bats (bats-core) |
| Node.js | Commander.js / yargs | Vitest / Jest |
| Go | cobra | testing (standard library) |
| Rust | clap | cargo test |

Regardless of language, §4.0 core conventions must be satisfied.
