# Permissions & Sandbox Declaration

> `skill-dev-standard` v2.2.0 — §9 series: permissions fields, allowed-tools, principle of least privilege

---

## 9. Permissions & Sandbox Declaration

**Every SKILL must declare its permission boundary in the SKILL.md frontmatter**, enabling the Agent to make security checks before execution.

### 9.1 Permission Fields

Place `permissions` under `metadata`:

```yaml
---
name: example-skill
description: "Use this skill when... Do NOT use for..."
metadata:
  permissions:
    read_paths:
      - "~/.example-skill/"      # config directory
      - "/tmp/example-skill/"    # temp directory
    write_paths:
      - "~/.example-skill/"      # config directory (write)
    network_endpoints:
      - "https://api.example.com"      # specific domain, no wildcards
      - "http://proxy.internal:8080"   # internal proxy
    requires_elevation: false     # whether sudo / admin is required
    can_spawn_processes: false    # whether the SKILL spawns child processes
    accesses_env_vars:
      - "SKILL_URL"
      - "SKILL_TOKEN"
---
```

### 9.2 Permission Declaration Principles

- **Least privilege**: only declare paths and endpoints actually needed
- **No wildcards**: `https://api.example.com` is clearer than `https://*.example.com`
- **Write operations must be listed explicitly**: declare read and write paths separately
- **List env vars**: let the Agent know which environment variables the SKILL reads

### 9.3 Additional Requirements for High-Privilege SKILLs

When `requires_elevation: true`, you must include an additional explanation:

```markdown
## Permission Notes

This SKILL requires the following elevated permissions for the reasons listed:

- **Write to /etc/hosts**: for local development domain mapping, executed only once during init
- **Access Docker socket**: for container lifecycle management

The Agent must confirm with the user before invoking this SKILL whether they accept the elevated operations.
```

---

## 9.4 allowed-tools (Top-Level Field, Not Inside metadata)

`allowed-tools` is a top-level frontmatter compliance field (sibling of `metadata`), declaring the set of tools this SKILL permits Claude to use:

```yaml
---
name: my-skill
description: "Use this skill when... Do NOT use for..."
allowed-tools: [Read, Bash, Edit]   # omit to allow all tools by default
metadata:
  version: "1.0.0"
  type: cli
---
```

**Minimization principle**:
- Pure docs SKILL: `[Read]` is usually sufficient
- CLI SKILL: usually `[Read, Bash]`
- SKILLs that don't modify files: do **not** grant `Write` / `Edit`
- SKILLs that don't need network access: do **not** grant `WebFetch` / `WebSearch`

Note: `allowed-tools` and `metadata.permissions` are complementary — the former is a whitelist at the Claude tool layer, the latter describes the resources the SKILL actually accesses (paths, network endpoints, env vars). Both must be declared.
