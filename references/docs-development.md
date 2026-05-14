# Docs SKILL Development

> `skill-dev-standard` v2.2.0 — §5 series: content organization, quality standards, freshness mechanism, documentation templates

---

## 5.1 Content Organization Principles

```
references/
├── concepts.md          # Concepts and terminology definitions
├── patterns.md          # Design patterns and best practices
├── examples.md          # Runnable code examples
├── troubleshooting.md   # Common issues and solutions
├── api-reference.md     # API or command index
├── output-schema.json   # Optional: output structure JSON Schema
└── changelog.md         # Update history (including breaking changes)
```

---

## 5.2 Documentation Quality Standards

| Dimension | Requirement |
|-----------|-------------|
| **Accuracy** | Correct information; runnable code examples |
| **Completeness** | Covers main scenarios and edge cases |
| **Clarity** | Concise language, clear logic |
| **Discoverability** | Good table of contents, indexes, search keywords |
| **Freshness** | Declare `valid_until` and `source_urls` in frontmatter |

---

## 5.3 Freshness Mechanism

```markdown
---
valid_until: "2026-12-31"
source_urls:
  - "https://docs.example.com/api/v2"
  - "https://github.com/example/sdk/releases"
---
```

Agent usage rules:
- `valid_until` is a specific date and has expired → warn the user before answering: "Documentation may be outdated; please verify."
- `valid_until: "evergreen"` → content does not depend on versioned APIs; no warning needed
- `source_urls` is non-empty → Agent may proactively suggest the user visit source URLs for the latest information

---

## 5.4 Documentation Templates

### concepts.md

```markdown
# Core Concepts

> Version: v2.3 | Updated: 2026-01-15

## Glossary

| Term | Definition |
|------|------------|
| {term} | {definition} |

## Core Principles

### Principle 1: XXX
...
```

### patterns.md

```markdown
# Design Patterns & Best Practices

## Pattern 1: XXX

**Problem**: Describe what problem you're solving.

**Solution**: Describe the recommended approach.

**Example**:
```python
# Complete runnable code
```

**Note**: List practices to avoid.
```

### examples.md

```markdown
# Code Examples

All examples are based on SDK v2.3, verified on Python 3.11+.

## Basic Examples

### Example 1: XXX
```python
# Complete runnable code including imports
```

**Expected output**:
```json
{"status": "ok", "result": "..."}
```
```
