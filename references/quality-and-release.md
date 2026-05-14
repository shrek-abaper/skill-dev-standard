# Quality Checks & Release

> `skill-dev-standard` v2.2.0 — §11/§12/§13/Appendix: versioning strategy, quality checklist, minimal templates, reference resources

---

## 11. Versioning & Compatibility Strategy

### 11.1 Version Number Convention (Semver)

```
MAJOR.MINOR.PATCH

MAJOR: Breaking change (command rename, output schema field removal/type change)
MINOR: Backward-compatible new features (new commands, new optional output fields)
PATCH: Bug fixes, documentation updates, performance improvements
```

### 11.2 Breaking Change Declaration

Maintain `changelog.md` at the root directory (sibling of `README.md`). Every MAJOR version bump must document:

```markdown
# Changelog

## [2.0.0] — 2026-05-01

### Breaking Changes
- `get` command output field `data` renamed to `value`
  - **Migration**: Update consumer code from `.data` to `.value`
- Removed deprecated `--output` flag; use `--format` instead

### Added
- `list` command now supports `--filter` flag

## [1.1.0] — 2026-03-01
...
```

### 11.3 Deprecation Process

```
1. Mark as deprecated in a PATCH release (add DeprecationWarning in code, note in docs)
2. Continue support in the next MINOR release but emit warnings
3. Remove in the next MAJOR release
4. Minimum deprecation period: 30 days (or 2 versions, whichever is longer)
```

### 11.4 MCP Annotation Alignment (Reference When Integrating with MCP Ecosystem)

If a SKILL will be exposed as an MCP Tool in the future, add these alignment fields under `metadata` in the SKILL.md frontmatter:

```yaml
---
name: my-skill
description: "Use this skill when... Do NOT use for..."
metadata:
  version: "1.0.0"
  mcp_hints:
    readOnlyHint: false       # true = read-only, does not modify external state
    destructiveHint: true     # true = potentially irreversible (e.g., delete)
    idempotentHint: false     # true = identical results for same args across calls
    openWorldHint: true       # true = makes external network requests
---
```

---

## 12. Quality Checks

### 12.1 Checklist

**Essentials**:
- [ ] SKILL.md contains: `name`, `description` (trigger-word-first style); all custom fields (`version`, `type`, `permissions`, `output_schema`) are nested under `metadata`
- [ ] `description` includes verb enumeration + negative exclusions; no `<` or `>` characters
- [ ] `README.md` user documentation exists
- [ ] Directory structure matches the selected type (CLI uses `docs/`; docs SKILL uses `references/`)
- [ ] Sensitive files (`.env`, credentials) are in `.gitignore`

**CLI SKILL Specific**:
- [ ] Uses Click or Typer; no custom argparse parsing
- [ ] All write operations have `--yes` (skip confirmation) + `--dry-run` (simulate)
- [ ] `--help` is supported with clear help text
- [ ] Normal output → stdout (JSON); errors → stderr (JSON)
- [ ] JSON config files have 0600 permissions
- [ ] Config loading follows: CLI args > env vars > file > defaults
- [ ] `metadata.output_schema` is declared (at minimum the `format` field)
- [ ] `metadata.permissions` is declared (`read_paths`, `write_paths`, `network_endpoints`)
- [ ] `allowed-tools` is declared at the frontmatter top level (see `references/permissions.md` §9.4)
- [ ] `tests/test_cli.py` exists, covering happy path + write operation guards + error scenarios

**Docs SKILL Specific**:
- [ ] `references/` content is organized logically (concepts / patterns / examples / troubleshooting)
- [ ] Code examples are runnable and annotated with Python and SDK versions
- [ ] `metadata` in frontmatter includes `valid_until` and `source_urls`
- [ ] `tests/golden-set.yaml` exists with at least 5 Q&A pairs
- [ ] `changelog.md` exists at the root

**Universal**:
- [ ] Version number follows Semver
- [ ] Breaking changes include migration instructions in `changelog.md`
- [ ] `.env.example` exists (if env vars are used)
- [ ] If using `assets/`: every asset file is explicitly referenced in SKILL.md; no real credentials or PII; single file < 100 KB

---

## 13. Templates

### 13.1 CLI SKILL Minimum Template

```
{skill-name}/
├── SKILL.md
├── README.md
├── .env.example
├── docs/
│   └── api.md
├── assets/             # optional: output templates, config scaffolds, payload examples
├── tests/
│   └── test_cli.py
└── scripts/
    └── {skill_name}.py
```

**SKILL.md starter content**:

```markdown
---
name: {skill-name}
description: |
  Use this skill when the user needs to {verb1}, {verb2} {object}.
  Supports {feature1}, {feature2}.
  Also triggers when: user asks "{typical-phrasing-1}", "{typical-phrasing-2}".
  Do NOT use for: {negative-example-1}, {negative-example-2}.
license: MIT
allowed-tools: [Read, Bash]
metadata:
  version: "1.0.0"
  type: cli
  author: {author}
  tags: []
  valid_until: "evergreen"
  permissions:
    read_paths: []
    write_paths: []
    network_endpoints: []
    requires_elevation: false
    accesses_env_vars: []
  output_schema:
    format: json
---

## Execution Rules
...
```

### 13.2 Docs SKILL Minimum Template

```
{skill-name}/
├── SKILL.md
├── README.md
├── changelog.md
├── assets/             # optional: images, data samples, and other output resources
├── tests/
│   └── golden-set.yaml
└── references/
    ├── concepts.md
    ├── patterns.md
    └── examples.md
```

**SKILL.md starter content**:

```markdown
---
name: {skill-name}
description: |
  Use this skill when the user asks about {domain-name}: {concepts/usage/examples/errors}.
  Covers: {core-scenario-1}, {core-scenario-2}.
  Also triggers when: user asks "{typical-phrasing-1}", "{typical-phrasing-2}".
  Do NOT use for: executing actual operations (use the corresponding CLI SKILL).
license: MIT
allowed-tools: [Read]
metadata:
  version: "1.0.0"
  type: docs
  author: {author}
  tags: []
  valid_until: "{YYYY-MM-DD}"
  source_urls:
    - "{official docs URL}"
---

## Execution Rules

### Knowledge Coverage
- Core concepts and terminology
- Design patterns and best practices
- Code examples (verified)
- Common issues and troubleshooting

### Knowledge Freshness
Based on {product/API} {version}; valid_until is {date}.
```

---

## Appendix

### A. Reference Resources

| Resource | Description |
|----------|-------------|
| [Click documentation](https://click.palletsprojects.com/) | Python CLI framework |
| [Typer documentation](https://typer.tiangolo.com/) | Modern type-hints CLI |
| [uv documentation](https://docs.astral.sh/uv/) | Python package manager (recommended) |
| [MCP Specification](https://modelcontextprotocol.io/) | Model Context Protocol |
| [JSON Schema](https://json-schema.org/) | Output schema spec reference |
| [Anthropic skill-examples](https://github.com/anthropics/skills) | Official SKILL examples repository |

### B. Version History

| Version | Date | Changes |
|---------|------|---------|
| 2.2.0 | 2026-05-14 | **Added**: §3.5 `assets/` directory spec — four-directory responsibility boundaries, use cases, placeholder convention `__VAR__`, SKILL.md reference table pattern, size/security constraints; §3.1–§3.4 structure diagrams updated with `assets/` optional entry; §0.5 hand-off index gains "needs output template/config scaffolding" trigger; §12 checklist adds `assets/` verification items; §13 templates add `assets/` row; **Architecture refactor**: `standard.md` split into 7 focused reference files; `SKILL.md` promoted to L2 routing layer (absorbs §1 type-decision matrix, adds Reference Files Index) |
| 2.1.0 | 2026-05-10 | **Fixed**: frontmatter template aligned to official `quick_validate.py` (custom fields nested under `metadata`); **Fixed**: description placeholder changed to `{}` (avoids `< >` triggering packaging validation errors); **Improved**: description authoring guide (100–300 chars, pushy style, anti-undertrigger); **Added**: §0.5 hand-off index with skill-creator, §2.0 progressive disclosure principle, §4.0 language-agnostic core conventions, §4.6 multi-language comparison, §9.4 `allowed-tools` field; **Deprivatized**: removed all private examples and local path references, replaced with generic domain examples (Git commit generator, AWS S3 knowledge base); **Removed**: `archives/` legacy version archive directory |
| 2.0.0 | 2026-05-05 | **Added**: Agent execution steps (§0), description trigger-word conventions (§2.2), Output Schema (§8), permission declaration (§9), testing standards (§10), versioning strategy (§11), MCP annotation alignment; **Improved**: directory name ambiguity resolved (CLI uses `docs/`, docs SKILL uses `references/`), unified JSON error format, dependency management migrated to `uv` |
| 1.0.0 | 2026-05-05 | Initial release, consolidates CLI and docs SKILL standards. |
