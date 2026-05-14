# Testing Standards

> `skill-dev-standard` v2.2.0 — §10 series: CliRunner unit tests, golden-set Q&A pairs, minimum testing requirements

---

## 10.1 CLI SKILL Testing (CliRunner)

Use `click.testing.CliRunner` for offline unit testing without external service dependencies:

```python
# tests/test_cli.py
import json
import pytest
from click.testing import CliRunner
from scripts.example_skill import cli

@pytest.fixture
def runner():
    return CliRunner()

class TestGetCommand:
    def test_get_returns_json(self, runner):
        result = runner.invoke(cli, ["get", "my-resource"])
        assert result.exit_code == 0
        data = json.loads(result.output)
        assert data["status"] == "ok"
        assert "name" in data

    def test_get_text_format(self, runner):
        result = runner.invoke(cli, ["get", "my-resource", "--format", "text"])
        assert result.exit_code == 0
        assert "name:" in result.output

class TestSetCommand:
    def test_set_requires_confirm(self, runner):
        # Without --yes, simulate user input 'n'
        result = runner.invoke(cli, ["set", "key", "value"], input="n\n")
        assert result.exit_code == 0
        data = json.loads(result.output)
        assert data["status"] == "cancelled"

    def test_set_dry_run(self, runner):
        result = runner.invoke(cli, ["set", "key", "value", "--dry-run"])
        assert result.exit_code == 0
        data = json.loads(result.output)
        assert data["status"] == "dry-run"

    def test_set_with_yes(self, runner):
        result = runner.invoke(cli, ["set", "key", "value", "--yes"])
        assert result.exit_code == 0

class TestErrorHandling:
    def test_missing_config_exits_1(self, runner, tmp_path, monkeypatch):
        monkeypatch.setenv("HOME", str(tmp_path))
        result = runner.invoke(cli, ["get", "anything"])
        assert result.exit_code == 1
```

For non-Python CLIs, write equivalent happy path + error code tests using Bats/Vitest/Go testing.

---

## 10.2 Docs SKILL Testing (Golden-Set Q&A Pairs)

Maintain a minimal set of Q&A pairs for the docs SKILL to verify knowledge coverage quality:

```yaml
# tests/golden-set.yaml
skill: api-reference
version: "1.0.0"
cases:
  - id: basic-auth
    query: "How do I authenticate with the API?"
    expected_keywords: ["Bearer", "Authorization", "token"]
    must_not_contain: ["password", "plaintext"]

  - id: error-404
    query: "What should I do when the API returns 404?"
    expected_keywords: ["not found", "path", "check"]

  - id: rate-limit
    query: "What is the request rate limit?"
    expected_keywords: ["100", "per minute", "429"]
```

---

## 10.3 Test Directory Structure

```
<skill-name>/
├── tests/
│   ├── test_cli.py          # CLI unit tests (CliRunner)
│   ├── test_config.py       # Config loading tests
│   ├── test_schemas.py      # Output schema compliance tests
│   └── golden-set.yaml      # Docs SKILL Q&A pairs (docs SKILL only)
└── pyproject.toml           # includes [tool.pytest.ini_options]
```

---

## 10.4 Minimum Testing Requirements

| SKILL Type | Must Cover |
|------------|-----------|
| CLI SKILL | Happy path for all commands; confirmation/cancel/dry-run for write ops; non-zero exit code scenarios |
| Docs SKILL | ≥5 golden-set Q&A pairs covering main query scenarios |
| Hybrid SKILL | Both required |
