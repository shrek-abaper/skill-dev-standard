---
name: skill-dev-standard
description: |
  Use when creating, modifying, or reviewing any SKILL. Covers: frontmatter spec, description style (pushy, 100-300 chars), metadata nesting, allowed-tools, versioning.
  Also trigger: SKILL structure questions, frontmatter validation errors, description writing. Do NOT use for: skill-creator workflow, packaging, or external APIs.
license: MIT
allowed-tools: [Read]
metadata:
  version: "2.2.0"
  type: docs
  valid_until: evergreen
  source_urls:
    - "https://github.com/anthropics/skills"
    - "https://click.palletsprojects.com/"
    - "https://typer.tiangolo.com/"
    - "https://docs.astral.sh/uv/"
  tags:
    - skill-development
    - standards
    - engineering
    - frontmatter
    - cli
    - docs
---

# SKILL Engineering Standards

This SKILL is your project's quality standards library for building AI Agent Skills. When creating or modifying any SKILL, **load this alongside skill-creator** — this SKILL provides engineering standards, skill-creator provides the development workflow.

## What This SKILL Does

This SKILL does **not replace** the official Anthropic skill-creator. It is its **engineering complement**. The two are complementary and **should be loaded together**:

| Concern | Owner |
|---------|-------|
| Development workflow (requirements capture, interviews, testing, evaluation, iteration, packaging) | **skill-creator** (official) |
| Design philosophy (Progressive Disclosure, writing style) | **skill-creator** (official) |
| Evaluation toolchain (eval-viewer, run_loop, benchmark) | **skill-creator** (official) |
| Claude.ai / Cowork environment adaptation | **skill-creator** (official) |
| CLI SKILL engineering standards (exit codes, output protocol, config, permissions) | **this SKILL** |
| Output Schema (inter-SKILL contracts, reducing compose hallucinations) | **this SKILL** |
| Docs SKILL freshness and quality (valid_until, golden-set) | **this SKILL** |
| Frontmatter compliance details (metadata nesting, allowed-tools) | **this SKILL** |
| Semver versioning and Breaking Change workflow | **this SKILL** |
| Quality checklist (CLI / docs / general — three columns) | **this SKILL** |

**Do NOT use this SKILL alone** — it assumes you have already followed skill-creator's workflow through "requirements capture → draft → testing → iteration". This SKILL supplements engineering standards at each stage.
Full hand-off index: see §0.5 below.

## §0.5 Hand-off Index with skill-creator

### Forward hand-off (skill-creator → this SKILL)

| skill-creator stage | Hand off to this SKILL |
|---------------------|------------------------|
| After Capture Intent | Read §1 to choose SKILL type (CLI / docs / hybrid) |
| Write the SKILL.md | Read `references/skillmd-authoring.md` §2.0–2.2 (progressive disclosure + description writing) |
| Configure frontmatter | Read `references/permissions.md` §9 (permissions + allowed-tools) |
| Choose CLI implementation language | Read `references/cli-templates.md` §4.0 (language-agnostic conventions), §4.6 (multi-language comparison) |
| Need output templates / config boilerplates / payload samples | Read `references/project-structure.md` §3.5 (assets/ directory spec) |
| Test Cases stage | CLI SKILL: read `references/testing.md` §10.1 (CliRunner); Docs SKILL: read §10.2 (golden-set) |
| Iterate stage (bug fixing) | Read `references/cli-runtime.md` §7 (error handling), §8 (Output Schema) |
| Before packaging | Read `references/quality-and-release.md` §12 quality checklist |
| Version upgrade | Read `references/quality-and-release.md` §11 (Semver + breaking change workflow) |
| MCP integration (optional) | Read `references/quality-and-release.md` §11.4 (MCP annotation alignment) |

### Reverse hand-off (this SKILL → skill-creator)

| Trigger | Hand off to skill-creator |
|---------|--------------------------|
| Need requirements interview / scope clarification | "Capture Intent + Interview" section |
| Need to run with-skill / baseline comparison | "Running and evaluating test cases" |
| Need to measure description trigger rate | "Description optimization" (run_loop.py) |
| Need to package into .skill file | "Package and Present" (package_skill.py) |
| Need Claude.ai / Cowork environment adaptation | "Claude.ai-specific instructions" / "Cowork-Specific Instructions" |

---

## §1 SKILL Type Decision

### Classification

| Type | Purpose | Use When | Tech Stack |
|------|---------|----------|------------|
| **CLI SKILL** | Wraps CLI tools or APIs | System operations, API integrations, data processing | Click / Typer |
| **Docs SKILL** | Wraps knowledge bases and reference docs | Domain knowledge, API docs, best practices | Markdown + code |
| **Hybrid SKILL** | CLI + Docs combined | Scenarios requiring both operations and knowledge | CLI + Docs |

### Decision Matrix

**Choose CLI SKILL when**:
- Executing system commands or operations
- Calling REST APIs for data interaction
- Wrapping existing command-line tools
- Needing CRUD operations
- Data format conversion or processing

**Choose Docs SKILL when**:
- Encapsulating domain knowledge and best practices
- API reference documentation lookups
- Coding conventions and pattern libraries
- Architecture design guides
- Troubleshooting guides

**Choose Hybrid SKILL when**:
- Need both knowledge queries and operational execution
- Docs contain executable code snippets
- Dynamic queries and real-time operations need to be combined

See official examples: [Anthropic skill-examples repository](https://github.com/anthropics/skills)

---

## Usage Steps

When creating a new SKILL, follow this order:

1. Read §1 (this file) — decide type (CLI / docs / hybrid)
2. Read `references/skillmd-authoring.md` §2.0 — understand Progressive Disclosure principles
3. Read `references/skillmd-authoring.md` §2.2 — write description (trigger-first + pushy style + negative examples)
4. Read `references/project-structure.md` §3 — set up directory structure
5. Read `references/cli-runtime.md` §8, `references/permissions.md` §9 — declare output_schema and permissions
6. Read `references/permissions.md` §9.4 — declare allowed-tools (top-level frontmatter field)
7. Read `references/testing.md` §10 — write minimal test suite
8. Use `references/quality-and-release.md` §12 checklist to validate, then hand to skill-creator to package

When modifying an existing SKILL:
- Directly audit using `references/quality-and-release.md` §12 checklist
- Fill in missing `metadata.output_schema` or `metadata.permissions`
- Confirm `allowed-tools` is declared at the top level
- Optimize description trigger keywords against `references/skillmd-authoring.md` §2.2

---

## Quick Reference

### description writing (highest priority)

The description is the core basis for Agent routing — how it is written directly determines recall rate. It must:
- **Start with verb + scenario enumeration**, not a noun
- **Include 1–2 negative exclusions** to help Agent avoid false triggers
- **100–300 words recommended** (hard limit: 1024 chars); keep key triggers in the first 30 words
- **No `<` or `>`**: will be rejected by quick_validate.py (use `{` `}` for placeholders)
- **Pushy style**: Claude defaults to undertriggering (not using SKILLs when it should). Actively list multiple user phrasings, related concepts, and implicit scenarios — not just a description of functionality.

Good example (CLI type):
```
Use this skill when generating Conventional Commits-compliant commit messages from git diff, staged changes, or change descriptions.
Covers: completing commit type prefixes (feat/fix/refactor/docs), inferring scope from multi-file diffs, generating multi-line commit bodies, appending BREAKING CHANGE annotations.
Also triggers when users say "how should I write this commit", "help me write a proper commit message", "format this as conventional commits".
Do NOT use for: actually running git commit, modifying git config, merging PRs.
```

Bad example:
```
Git commit tool that wraps commit message generation.
```

Full examples and rules → `references/skillmd-authoring.md` §2.2

### Frontmatter compliant fields

Packaging validation only allows top-level fields: `name, description, license, allowed-tools, metadata, compatibility`

All custom fields (output_schema, permissions, valid_until, etc.) go inside `metadata`;
`allowed-tools` is a top-level compliant field — **do NOT put it inside** metadata:

```yaml
---
name: my-skill
description: "Use this skill when... Do NOT use for..."
license: MIT
allowed-tools: [Read, Bash]
metadata:
  version: "1.0.0"
  type: cli          # cli | docs | hybrid
  valid_until: "2026-12-31"   # or "evergreen"
  source_urls:
    - "https://docs.example.com"
  output_schema:
    format: json     # json | text | mixed
  permissions:
    read_paths: ["~/.my-skill/"]
    write_paths: ["~/.my-skill/"]
    network_endpoints: ["https://api.example.com"]
    requires_elevation: false
    accesses_env_vars: ["MY_SKILL_TOKEN"]
---
```

Full template → `references/skillmd-authoring.md` §2.3

### Directory naming conventions

| SKILL type | External reference dir | Knowledge base dir |
|------------|----------------------|-------------------|
| CLI SKILL | `docs/` | — |
| Docs SKILL | — | `references/` |
| Hybrid SKILL | `docs/` | `references/` |

> CLI SKILLs do NOT use `references/` — avoids confusion with the docs SKILL knowledge base directory.

Full structure diagrams → `references/project-structure.md` §3.1–§3.4

### Error output convention

All CLI SKILLs use JSON format for error output. Normal results → stdout, errors → stderr:

```python
# Normal output
click.echo(json.dumps({"status": "ok", "data": result}))

# Error output
click.echo(json.dumps({"status": "error", "code": 1, "message": "..."}), err=True)
sys.exit(1)
```

Exit code convention: 0=success, 1=config error, 2=network error, 3=permission error, 4=param error, 99=unexpected error.

Full spec → `references/cli-runtime.md` §7

---

## Reference Files Index

Read the relevant file on demand — **no need to load everything**:

| File | Coverage | When to read |
|------|---------|-------------|
| `references/skillmd-authoring.md` | §2.0 Progressive Disclosure; §2.1 required fields; §2.2 description writing; §2.3 full SKILL.md template; §2.4 docs SKILL specifics | When writing or reviewing any SKILL.md |
| `references/project-structure.md` | §3.1–3.4 four structure diagrams (flat/modular/docs/hybrid); §3.5 assets/ spec (directory boundaries, placeholders, security) | When designing directory structure or needing output templates |
| `references/cli-templates.md` | §4.0 language-agnostic conventions; §4.1 uv dependency management; §4.3 Click full template; §4.4 Typer template; §4.5 Click quick ref; §4.6 multi-language comparison | When implementing a CLI SKILL |
| `references/docs-development.md` | §5.1 content organization; §5.2 quality standards; §5.3 freshness mechanism (valid_until); §5.4 doc templates | When implementing a Docs SKILL |
| `references/cli-runtime.md` | §6 config management (priority + class template); §7 error handling (exit codes + JSON format); §8 Output Schema (TypedDict + JSON Schema) | When implementing config loading, error handling, or declaring output structure |
| `references/permissions.md` | §9.1–9.3 permissions fields; §9.4 allowed-tools (top-level field) | When declaring permission boundaries |
| `references/testing.md` | §10.1 CliRunner unit test template; §10.2 golden-set Q&A pairs; §10.3 test directory; §10.4 minimum test requirements | When writing tests |
| `references/quality-and-release.md` | §11 Semver + Breaking Change + MCP annotation; §12 quality checklist (CLI/docs/general); §13 minimal templates; Appendix A references | Before packaging or on version upgrades |
