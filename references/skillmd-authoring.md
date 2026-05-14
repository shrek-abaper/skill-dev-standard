# SKILL.md Authoring Guide

> `skill-dev-standard` v2.2.0 — §2: SKILL.md structure, frontmatter, description writing, templates

---

## 2.0 Progressive Disclosure (Most Important Design Principle)

SKILL content is organized into three load layers. Decide where content belongs based on **when it needs to be read**:

| Layer | Content | When loaded | Size constraint |
|-------|---------|------------|----------------|
| L1 metadata | name + description | Always in Agent context | description ≤ 1024 chars |
| L2 SKILL.md body | Quick-reference rules, decision points, reference pointers | Loaded entirely when SKILL triggers | Recommended ≤ 500 lines |
| L3 references/ or docs/ | Full specs, long API docs, example libraries | Agent reads on demand | No limit |

**Deciding which layer content belongs to**:
- "Agent needs to glance at this every time" → L2 (SKILL.md)
- "Agent only needs this when hitting a specific problem" → L3 (references/ or docs/)
- SKILL.md exceeding 500 lines → split into L3, keep SKILL.md as index only

**Typical anti-pattern**: Stuffing full API references, error handling tables, and command lists into SKILL.md, causing the Agent to load thousands of irrelevant lines on every trigger.

---

## 2.1 Required Fields

Every SKILL must include the following in `SKILL.md`:

- `name`: kebab-case, ≤ 64 chars
- `description`: trigger-first description, see §2.2, ≤ 1024 chars, **no `<` or `>`**
- `license`: open source license (e.g. MIT)
- `allowed-tools`: list of tools the SKILL is permitted to use
- `metadata`: contains all custom fields (version, type, permissions, output_schema, etc.)

```yaml
---
name: "your-skill-name"
description: |
  Use this skill when you need to VERB1, VERB2 OBJ.
  Supports FEATURE1, FEATURE2.
  Also triggers when users say "typical phrasing 1", "typical phrasing 2".
  Do NOT use for: NEGATIVE1, NEGATIVE2.
license: MIT
allowed-tools: [Read, Bash]
metadata:
  version: "1.0.0"
  type: cli
  author: "TEAM_NAME"
  tags: ["TAG1", "TAG2"]
  valid_until: "evergreen"
  source_urls: []
  output_schema:
    format: json
    schema_ref: "references/output-schema.json"
  permissions:
    read_paths: ["~/.your-skill-name/"]
    write_paths: ["~/.your-skill-name/"]
    network_endpoints: ["https://api.example.com"]
    requires_elevation: false
    accesses_env_vars: ["SKILL_TOKEN"]
  mcp_hints:
    readOnlyHint: false
    destructiveHint: false
    idempotentHint: true
    openWorldHint: true
---
```

---

## 2.2 description Field Writing Guide

The description is the core basis for Agent routing — how it is written directly determines recall rate.

**Core rules**:
1. **Start with verb + scenario enumeration**, not a noun
2. **Include 1–2 negative exclusions** to help avoid false triggers
3. **100–300 words recommended** (hard limit: 1024 chars) — better to be longer than to miss typical trigger scenarios
4. **Front-load key terms**: most important trigger keywords within the first 30 words
5. **No `<` or `>`**: will be rejected by packaging validation (use `{` `}` or omit placeholders)
6. **Pushy style**: Claude defaults to undertriggering (not using SKILLs when it should). Actively list multiple user phrasings, related concepts, and implicit scenarios — not just a description of functionality.

**Good description (CLI type — verb enumeration + negative exclusion + pushy phrasing)**:

```
Use this skill when generating Conventional Commits-compliant commit messages from git diff, staged changes, or change descriptions.
Covers: completing commit type prefixes (feat/fix/refactor/docs), inferring scope from multi-file diffs, generating multi-line commit bodies, appending BREAKING CHANGE annotations.
Also triggers when users say "how should I write this commit", "help me write a proper commit message", "format this as conventional commits".
Do NOT use for: actually running git commit, modifying git config, merging PRs.
```

**Good description (Docs type — AWS S3 knowledge base)**:

```
Use this skill when asked about AWS S3 API usage, bucket policies, object lifecycle, storage classes, cross-region replication, presigned URLs, CORS configuration, or encryption options.
Covers: boto3 SDK calls, IAM permission design, common error codes (403/404/SlowDown/RequestTimeout), cost optimization patterns, CloudFront integration best practices.
Also triggers for: "how does S3 versioning work", "how to implement multipart upload", "S3 vs EBS — which to choose".
Do NOT use for: actually executing S3 API calls (use the aws-cli SKILL), billing queries.
```

**Bad description (noun-first, too short, no negative example)**:

```
Git commit tool that wraps commit message generation functionality.
```

**Length reference**:
- Official skill-creator self-description: ~95 words (English)
- Official built-in SKILL descriptions: typically 80–200 words
- Hard upper limit: 1024 chars (enforced by quick_validate.py)

---

## 2.3 Full SKILL.md Template

```markdown
---
name: example-skill
description: |
  Use this skill when you need to perform X operations, query Y data, or process Z files.
  Supports batch processing, format conversion, and status checks.
  Also triggers when users say "how do I query X", "help me process this Z file", "where is Y data".
  Do NOT use for: modifying system configuration, cross-system sync, operations requiring admin privileges.
license: MIT
allowed-tools: [Read, Bash]
metadata:
  version: "1.0.0"
  type: cli
  author: AI Agent Team
  tags: [example, template]
  valid_until: "evergreen"
  source_urls: []
  output_schema:
    format: json
    schema_ref: "references/output-schema.json"
  permissions:
    read_paths: ["~/.example-skill/"]
    write_paths: ["~/.example-skill/"]
    network_endpoints: ["https://api.example.com"]
    requires_elevation: false
    accesses_env_vars: ["EXAMPLE_TOKEN"]
---

## Execution Rules

### Use Cases
Invoke this SKILL when the user needs to perform X operations.

**Trigger examples**:
- "Get Y data"
- "Process Z file"
- "Check W status"

**Does NOT trigger for**:
- "Query data from system A" (out of supported scope)
- "Modify system-level config" (requires admin privileges, outside this SKILL's scope)

### Capabilities

**Supported operations**:
- Feature 1: description
- Feature 2: description
- Feature 3: description

**Not supported**:
- Cannot directly modify system config
- Does not support batch operations (one at a time)

### Prerequisites
- Required dependencies must be installed (or the SKILL handles them automatically)
- A valid config file is needed (initialize with the `init` command)
- Network connectivity required

### Security Conventions
- Write operations require user confirmation by default (use `--yes` to skip)
- Sensitive data stored in `~/.example-skill/config.json` with 0600 permissions
- No passwords or tokens logged
- All network requests use HTTPS; certificate verification is never skipped

### Output Format
- Default output: JSON (machine-friendly)
- Optional text output: `--format text`
- Error messages go to stderr with non-zero exit code

### Error Handling

| Error Type | Exit Code | Handling |
|-----------|-----------|---------|
| Config missing | 1 | Prompt user to run `init` command |
| Network error | 2 | Display detailed error, suggest retry |
| Permission error | 3 | Show required permissions, suggest contacting admin |
| Parameter error | 4 | Show correct usage examples |
| Resource not found | 5 | Return empty result JSON, exit 0 |

## Command Structure

- `init` — initialize configuration
- `get <resource>` — retrieve a resource
- `set <resource> <value>` — set a resource (write operation, confirmation required)
- `list [--filter]` — list all resources
- `delete <resource>` — delete a resource (destructive, confirmation required)
```

---

## 2.4 Docs SKILL-Specific Fields

```markdown
---
name: {skill-name}
description: |
  Use this skill when asked about {domain} concepts, usage, examples, or errors.
  Covers: {core scenario 1}, {core scenario 2}, {core scenario 3}.
  Also triggers when users say "{typical phrasing 1}", "{typical phrasing 2}".
  Do NOT use for: executing actual operations (use the corresponding CLI SKILL), querying runtime state.
license: MIT
allowed-tools: [Read]
metadata:
  version: "1.0.0"
  type: docs
  valid_until: "2026-12-31"
  source_urls:
    - "https://docs.example.com/api/v2"
---

## Execution Rules

### Knowledge Coverage
- API endpoint list and parameter descriptions
- Request/response examples (including error cases)
- Error code reference
- Best practices and common pitfalls

### Knowledge Freshness
This document is based on API v2.3 with valid_until set to 2026-12-31.
After that date, the Agent should prompt the user to verify against the latest docs.
```
