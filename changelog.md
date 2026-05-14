# Changelog

## [2.2.0] — 2026-05-14

### Added
- §3.5 `assets/` directory spec — four-directory responsibility boundaries (`scripts/` execute, `references/` understand, `docs/` reference, `assets/` embed/deliver)
- Placeholder convention (`__VARIABLE__` double-underscore, aligned with skill-creator official), SKILL.md reference table pattern, size constraints (< 100 KB/file), security requirements (no real credentials/PII)
- §0.5 hand-off index: "needs output template/config scaffolding" trigger line
- `assets/` optional entry in §3.1–§3.4 structure diagrams
- `assets/` acceptance items in §12 checklist
- `assets/` row in §13 templates

### Changed
- Architecture refactor: `standard.md` split into 7 focused reference files
- `SKILL.md` promoted to L2 routing layer (absorbs §1 type-decision matrix, adds Reference Files Index)
- All content files translated from Chinese to English; `README.md` rewritten in English (primary language); `README.zh-CN.md` added as Chinese translation

---

## [2.1.0] — 2026-05-10

### Fixed
- Frontmatter template aligned to official `quick_validate.py` — all custom fields nested under `metadata`
- Description placeholder changed to `{}` (avoids `< >` triggering packaging validation errors)

### Added
- §0.5 hand-off index with skill-creator
- §2.0 Progressive Disclosure principle
- §4.0 language-agnostic core conventions
- §4.6 multi-language comparison table
- §9.4 `allowed-tools` field documentation

### Improved
- Description authoring guide: 100–300 chars, pushy style, anti-undertrigger guidance

### Removed
- All private SAP business examples and local path references (replaced with generic examples: Git commit generator, AWS S3 knowledge base)
- `archives/` legacy version archive directory (history preserved in git)

---

## [2.0.0] — 2026-05-05

### Added
- Agent execution steps (§0)
- Description trigger-word conventions (§2.2)
- Output Schema specification (§8)
- Permission declaration standard (§9)
- Testing standards (§10)
- Versioning strategy with Semver (§11)
- MCP annotation alignment hints (§11.4)

### Improved
- Directory naming: CLI uses `docs/`, docs SKILL uses `references/` (resolved ambiguity)
- Unified JSON error format across all CLI SKILLs
- Dependency management migrated to `uv`

---

## [1.0.0] — 2026-05-05

Initial release — consolidates CLI and docs SKILL engineering standards.
