<div align="center">

# 📐 skill-dev-standard

**English** &nbsp;·&nbsp; [中文](README.zh-CN.md)

### Engineering-Grade Standards for Agent SKILL Development

#### *The engineering complement to Anthropic's official skill-creator — exit codes, output contracts, permissions, versioning, and a release checklist.*

> Load both SKILLs together: skill-creator owns the workflow (interviews, evals, iteration, packaging); skill-dev-standard owns the engineering (CLI conventions, Output Schema, docs freshness, frontmatter compliance, Semver, quality gates). One without the other leaves blind spots.

**Exit Codes &nbsp;·&nbsp; Output Schema &nbsp;·&nbsp; Config & Credentials &nbsp;·&nbsp; Permissions & Sandbox**

**Progressive Disclosure &nbsp;·&nbsp; Anti-Undertrigger Descriptions &nbsp;·&nbsp; Golden-Set Evals &nbsp;·&nbsp; Semver Gates**

**CLI · Docs · Hybrid &nbsp;·&nbsp; Click & Typer Templates &nbsp;·&nbsp; 7 Focused References &nbsp;·&nbsp; v2.2.0**

#### Built for SKILL Authors Who Ship

[![GitHub Stars](https://img.shields.io/github/stars/shrek-abaper/skill-dev-standard?style=flat-square&color=FFD700&logo=github&logoColor=white&label=Stars)](https://github.com/shrek-abaper/skill-dev-standard/stargazers) [![GitHub Forks](https://img.shields.io/github/forks/shrek-abaper/skill-dev-standard?style=flat-square&color=6E40C9&logo=github&logoColor=white&label=Forks)](https://github.com/shrek-abaper/skill-dev-standard/network/members) [![Contributors](https://img.shields.io/github/contributors/shrek-abaper/skill-dev-standard?style=flat-square&color=2EA043&logo=github&logoColor=white)](https://github.com/shrek-abaper/skill-dev-standard/graphs/contributors) [![Last Commit](https://img.shields.io/github/last-commit/shrek-abaper/skill-dev-standard?style=flat-square&logo=github&logoColor=white)](https://github.com/shrek-abaper/skill-dev-standard/commits/main) [![Version](https://img.shields.io/badge/version-v2.2.0-0066CC?style=flat-square)](./changelog.md) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](./README.md#license) [![Pairs With](https://img.shields.io/badge/pairs%20with-skill--creator-DA291C?style=flat-square&logo=anthropic&logoColor=white)](https://github.com/anthropics/skills)

[![SKILL.md](https://img.shields.io/badge/SKILL.md-000000?style=for-the-badge&logo=markdown&logoColor=white)](./SKILL.md)
[![Changelog](https://img.shields.io/badge/Changelog-v2.2.0-2EA043?style=for-the-badge&logo=keepachangelog&logoColor=white)](./changelog.md)
[![skill-creator](https://img.shields.io/badge/Anthropic-skill--creator-D97757?style=for-the-badge&logo=anthropic&logoColor=white)](https://github.com/anthropics/skills)

```bash
# Package, then drop into your Agent's skills directory
python /path/to/skill-creator/scripts/package_skill.py .
cp skill-dev-standard.skill ~/.claude/skills/
```

</div>

Most SKILLs fail in production not because of bad prompts, but because of bad engineering: scripts print prose where callers expect JSON, write operations run without dry-run gates, credentials end up hardcoded, and nothing defines what "done" looks like at packaging time.

skill-dev-standard is the engineering layer beside the official skill-creator: a language-agnostic CLI contract (exit codes, JSON Output Schema, config priority and credential rules), SKILL.md authoring rules that resist undertriggering, a docs freshness mechanism (`valid_until`, `source_urls`, golden-set Q&A), Semver with a breaking-change workflow, and the §12 quality checklist that gates every release. skill-creator remains the workflow engine — this SKILL fills the engineering gaps it deliberately doesn't cover.

> [!NOTE]
> **A complement, not a replacement.** skill-creator owns interviews, the eval toolchain, and packaging compliance; skill-dev-standard owns CLI/docs engineering standards and the quality checklist. Consult the §0.5 hand-off index in SKILL.md at every stage transition — it states exactly which sections belong to whom.

**Who it's for:**

- **SKILL authors building CLI tools** — exit-code spec, Output Schema, Click/Typer templates, config and permissions conventions, plus CliRunner tests
- **Teams standardizing agent skills** — one frontmatter contract, one Semver policy, one release checklist across every SKILL they ship
- **Docs-skill maintainers** — `valid_until` / `source_urls` freshness rules and golden-set coverage, so reference knowledge doesn't silently rot
- **Skill reviewers and maintainers** — the §12 three-column checklist (CLI / docs / universal) as the pre-packaging gate
- **Enterprise integrators wrapping internal APIs** — credential safety, dry-run-first write operations, and machine-readable output downstream agents can rely on

**[Positioning](#positioning)** &nbsp;·&nbsp; **[Installation](#installation)** &nbsp;·&nbsp; **[Quick Start](#quick-start)** &nbsp;·&nbsp; **[Content Structure](#content-structure)** &nbsp;·&nbsp; **[Key Specs](#key-specs-at-a-glance)** &nbsp;·&nbsp; **[Changelog](#changelog)**

---

## Positioning

This SKILL does **not replace** skill-creator — it is its **engineering complement**. Their responsibilities are complementary:

| Concern | Owner |
|---------|-------|
| Dev workflow (interviews, testing, eval, iteration, packaging) | **skill-creator** (official) |
| Design philosophy (Progressive Disclosure, description style) | **skill-creator** (official) |
| Eval toolchain (eval-viewer, run_loop, benchmark) | **skill-creator** (official) |
| Claude.ai / Cowork environment adaptation | **skill-creator** (official) |
| CLI SKILL engineering standards (exit codes, output protocol, config, permissions) | **This SKILL** |
| Output Schema (inter-SKILL contracts) | **This SKILL** |
| Docs SKILL freshness & quality (`valid_until`, golden-set) | **This SKILL** |
| Frontmatter compliance details (`metadata` nesting, `allowed-tools`) | **This SKILL** |
| Semver versioning & breaking change workflow | **This SKILL** |
| Quality checklist (CLI / docs / universal three-column) | **This SKILL** |

**Both should be loaded simultaneously** — using either one alone will leave blind spots.

## Installation

**Option 1: Packaged install (recommended for distribution)**

```bash
# Package from repo root using skill-creator's script
python /path/to/skill-creator/scripts/package_skill.py .

# Install to local Agent skills directory
cp skill-dev-standard.skill ~/.claude/skills/
# Or the skills path for your Agent tool (OpenCode / Cursor, etc.)
```

**Option 2: Direct clone (recommended for local development)**

```bash
git clone <repo-url> ~/.claude/skills/skill-dev-standard
```

## Recommended Configuration (Strongly Encouraged)

To prevent Claude from undertriggering and loading only one SKILL, append the following to your `~/.claude/CLAUDE.md` (user-level system prompt).

**Copy this block directly into `~/.claude/CLAUDE.md`:**

```markdown
## SKILL Development Workflow (Mandatory)

When creating, modifying, reviewing, or packaging any SKILL:

1. ALWAYS load both SKILLs together:
   - skill-creator (workflow engine + eval toolchain)
   - skill-dev-standard (engineering standards + quality checklist)

2. Consult the hand-off index at every stage transition: read §0.5 in skill-dev-standard
   to confirm which sections each SKILL owns. Do not read only one of them.

3. Dual validation before packaging:
   - Run skill-creator's package_skill.py (compliance check)
   - Walk through skill-dev-standard's §12 quality checklist (engineering quality)
```

This is a system-level directive, more reliable than SKILL description hints. Claude Code / OpenCode reads `CLAUDE.md` at the start of every session, ensuring both SKILLs are loaded.

## Quick Start

Shortest path to create a new CLI SKILL:

1. Have Claude load both skill-creator + skill-dev-standard
2. skill-creator runs Capture Intent → Interview → drafts SKILL.md
3. **Hand off to this SKILL**:
   - Read SKILL.md §1 to select SKILL type → CLI
   - Read `references/skillmd-authoring.md` §2.2 to write the description (verb enumeration + multi-scenario + pushy style)
   - Read `references/cli-templates.md` §4 for the Click template (or §4.6 for other language options)
   - Read `references/permissions.md` §9 to fill in `permissions`; §9.4 for `allowed-tools`
4. skill-creator runs evals and iterates
5. Use `references/quality-and-release.md` §12 checklist for acceptance → skill-creator calls `package_skill.py` to package

Full hand-off index: see [SKILL.md §0.5](SKILL.md).

## Content Structure

```
skill-dev-standard/
├── SKILL.md                          # L2 routing layer: positioning + §0.5 hand-off index + §1 type decision + key spec quick-ref + Reference Files Index
├── README.md                         # This file: installation, config, quick start
├── README.zh-CN.md                   # Chinese translation
├── changelog.md                      # Full version history with migration notes
├── evals/
│   └── golden-set.yaml               # Q&A pairs for knowledge coverage verification
└── references/                       # L3 on-demand: 7 focused reference files
    ├── skillmd-authoring.md          # §2: progressive disclosure, required elements, description style, full template
    ├── project-structure.md          # §3: four directory structure diagrams + §3.5 assets/ spec
    ├── cli-templates.md              # §4: core conventions, Click/Typer full templates, multi-language comparison
    ├── docs-development.md           # §5: content organization, quality standards, freshness mechanism, doc templates
    ├── cli-runtime.md                # §6/§7/§8: config management, error handling, Output Schema
    ├── permissions.md                # §9: permissions field + §9.4 allowed-tools
    ├── testing.md                    # §10: CliRunner unit tests, golden-set Q&A pairs
    └── quality-and-release.md       # §11/§12/§13/Appendix: versioning, quality checklist, minimal templates
```

## Key Specs at a Glance

| Section | Content | Reference File |
|---------|---------|----------------|
| §0 / §0.5 | Agent check order + hand-off index | `SKILL.md` |
| §1 | SKILL type decision (CLI / docs / hybrid) | `SKILL.md` §1 |
| §2 | SKILL.md spec (progressive disclosure, description style, full template) | `references/skillmd-authoring.md` |
| §3 | Project structure (four structure diagrams + §3.5 assets/ spec) | `references/project-structure.md` |
| §4 | CLI SKILL development (§4.0 language-agnostic core conventions + Click/Typer templates + §4.6 multi-language) | `references/cli-templates.md` |
| §5 | Docs SKILL development (incl. freshness: `valid_until` / `source_urls`) | `references/docs-development.md` |
| §6 | Config management (priority + dataclass template) | `references/cli-runtime.md` |
| §7 | Error handling (exit code spec + JSON format) | `references/cli-runtime.md` |
| §8 | Output Schema (TypedDict + JSON Schema) | `references/cli-runtime.md` |
| §9 | Permissions & sandbox (§9.1–§9.3 permissions, §9.4 allowed-tools) | `references/permissions.md` |
| §10 | Testing standards (CliRunner + golden-set) | `references/testing.md` |
| §11 | Versioning (Semver + breaking change workflow + MCP annotation alignment) | `references/quality-and-release.md` |
| §12 | Quality checklist (CLI / docs / universal three columns) | `references/quality-and-release.md` |
| §13 | Minimal templates (CLI + docs) | `references/quality-and-release.md` |

## Changelog

### v2.2.0 — 2026-05-14

**Added assets/ engineering spec**:

- **Added**: §3.5 `assets/` directory spec — four-directory responsibility boundaries (`scripts/` execute, `references/` understand, `docs/` reference, `assets/` embed/deliver)
- **Added**: Placeholder convention (`__VARIABLE__` double-underscore, aligned with skill-creator official), SKILL.md reference table pattern, size constraints (< 100 KB/file), security requirements (no real credentials/PII)
- **Added**: §0.5 hand-off index gains "needs output template/config scaffolding" trigger
- **Updated**: §3.1–§3.4 structure diagrams all add `assets/` optional entry; §12 checklist adds assets/ acceptance items; §13 templates add `assets/` row

### v2.1.0 — 2026-05-10

**Generalization refactor + frontmatter compliance fix**:

- **Fixed**: frontmatter template aligned to official `quick_validate.py` (all custom fields nested under `metadata`)
- **Fixed**: description placeholder changed to `{}` (avoids `< >` triggering packaging validation errors)
- **Added**: §0.5 hand-off index with skill-creator, §2.0 progressive disclosure principle, §4.0 language-agnostic core conventions, §4.6 multi-language comparison, §9.4 `allowed-tools` field
- **Improved**: description authoring guide (100–300 chars, pushy style, anti-undertrigger)
- **Deprivatized**: removed all private SAP business examples and local path references, replaced with generic domain examples (Git commit generator, AWS S3 knowledge base)
- **Removed**: `archives/` legacy version archive directory (history preserved in git)

### v2.0.0 — 2026-05-05

Initial public version, consolidates CLI and docs SKILL standards.

## Feedback & Contributing

If you find a conflict between this SKILL's standards and the official skill-creator, or encounter engineering scenarios not covered here, please open an issue.

## License

MIT
