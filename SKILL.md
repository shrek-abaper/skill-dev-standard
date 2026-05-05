---
name: skill-dev-standard
description: Use this skill whenever creating a new SKILL, modifying an existing SKILL, reviewing SKILL quality, or writing SKILL.md content. Provides project-level SKILL development standards covering description trigger-word writing, output_schema declaration, permissions fields, test specs, and versioning strategy. Use alongside skill-creator — skill-creator handles the workflow, this skill provides the standards. Also trigger when the user asks about SKILL structure, SKILL directory layout, how to write a good description, or what fields a SKILL.md should have.
metadata:
  version: "2.0.0"
  type: docs
  valid_until: evergreen
---

# SKILL 开发规范

这个 SKILL 是你项目的 SKILL 质量标准库。在创建或修改任何 SKILL 时，先读本文档，再调用 skill-creator 执行流程。

## 与 skill-creator 的分工

| 角色 | 负责什么 |
|------|---------|
| **skill-creator** | 流程：访谈需求、跑测试、评估迭代、打包 |
| **skill-dev-standard（本 SKILL）** | 标准：SKILL.md 写法、目录约定、Schema、权限、测试规范 |

## 使用步骤

创建新 SKILL 时按此顺序执行：

1. 读 `references/standard.md` §1 — 确定类型（CLI / 文档 / 复合）
2. 读 §2.2 — 写 description（触发词优先 + 负例排除）
3. 读 §3 — 确认目录结构（CLI 用 `docs/`，文档 SKILL 用 `references/`）
4. 读 §8、§9 — 填写 output_schema 和 permissions（放入 metadata 字段）
5. 读 §10 — 写最小测试集（CliRunner 单测 / golden-set）
6. 用 §12 检查清单验收后，交给 skill-creator 打包

修改已有 SKILL 时：
- 直接用 §12 检查清单审查
- 补全缺失的 metadata.output_schema 或 metadata.permissions
- 对照 §2.2 优化 description 触发词写法

## 关键规范速查

### description 写法（最高优先级）

description 是 Agent 路由的核心依据，必须：
- **以动词 + 场景列举开头**，不以名词开头
- **包含 1-2 个负例排除**，帮助 Agent 避免误触发
- **长度 50-150 字**，关键词前置（前 30 字内出现主要触发词）

好的写法：
```
当用户需要查询、检索、过滤 SAP API 调用日志时使用此 SKILL。
支持按时间范围、接口名称、状态码查询，并能导出为 CSV 或 JSON。
不适用于：实时监控告警、修改日志配置、查询非 SAP 系统的日志。
```

差的写法：
```
SAP API 日志查询工具，封装了日志检索功能。
```

### frontmatter 合规字段

打包验证只允许：`name, description, license, allowed-tools, metadata, compatibility`

自定义字段（output_schema、permissions、valid_until 等）一律放入 `metadata`：

```yaml
---
name: my-skill
description: "当用户需要... 不适用于..."
metadata:
  version: "1.0.0"
  type: cli          # cli | docs | hybrid
  valid_until: "2026-12-31"   # 或 "evergreen"
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

### 目录命名约定

| SKILL 类型 | 外部参考资料目录 | 知识库目录 |
|------------|----------------|----------|
| CLI SKILL | `docs/` | — |
| 文档 SKILL | — | `references/` |
| 复合 SKILL | `docs/` | `references/` |

> CLI SKILL 不使用 `references/`，避免与文档 SKILL 的知识库目录混淆。

### 错误输出规范

所有 CLI SKILL 统一使用 JSON 格式输出错误，正常结果 → stdout，错误 → stderr：

```python
# 正常
click.echo(json.dumps({"status": "ok", "data": result}))

# 错误
click.echo(json.dumps({"status": "error", "code": 1, "message": "..."}), err=True)
sys.exit(1)
```

Exit code 约定：0=成功，1=配置错误，2=网络错误，3=权限错误，4=参数错误，99=未预期错误。

## 完整规范

详细内容见 `references/standard.md`，包含：

- §1 SKILL 类型与决策矩阵
- §2 SKILL.md 完整规范（含 description 模板）
- §3 项目目录结构（扁平 / 模块化 / 文档 / 复合）
- §4 CLI SKILL 开发（Click / Typer 完整模板）
- §5 文档 SKILL 开发（时效机制、内容模板）
- §6 配置管理（优先级、类模板）
- §7 错误处理（exit code、JSON 格式）
- §8 输出 Schema（TypedDict + JSON Schema）
- §9 权限与沙箱声明
- §10 测试规范（CliRunner 模板、golden-set）
- §11 版本化与兼容策略（semver、MCP annotation）
- §12 质量检查清单（CLI / 文档 / 通用三栏）
- §13 SKILL 最小模板

读取时机：遇到具体问题（如"CliRunner 怎么写"）时再读对应章节，不必全文加载。
