---
name: skill-dev-standard
description: |
  Use this skill whenever creating a new SKILL, modifying an existing SKILL, reviewing SKILL quality, or writing SKILL.md content.
  Covers: SKILL.md frontmatter structure, description trigger-word writing (pushy style, 100-300 chars), metadata nesting rules, output_schema declaration, permissions fields, allowed-tools selection, test specs, versioning strategy, and progressive disclosure design.
  Also trigger when the user asks about SKILL structure, SKILL directory layout, how to write a good description, what fields a SKILL.md should have, how to avoid undertriggering, frontmatter validation errors, or how to register allowed-tools.
  Do NOT use for: actually running skill-creator workflow, packaging a skill (use skill-creator), or querying external APIs.
license: MIT
allowed-tools: [Read]
metadata:
  version: "2.1.0"
  type: docs
  valid_until: evergreen
---

# SKILL 开发规范

这个 SKILL 是你项目的 SKILL 质量标准库。在创建或修改任何 SKILL 时，**应当与 skill-creator 同时加载**，本 SKILL 提供工程规范，skill-creator 提供开发流程。

## 本 SKILL 的定位

本 SKILL **不替代** Anthropic 官方 skill-creator，而是其**工程化补充**。
两者是互补关系，**应当同时加载、协同使用**：

| 关注层面 | 由谁负责 |
|---------|---------|
| 开发流程（需求捕获、访谈、测试、评估、迭代、打包） | **skill-creator**（官方） |
| 设计哲学（Progressive Disclosure、写作风格） | **skill-creator**（官方） |
| 评估工具链（eval-viewer、run_loop、benchmark） | **skill-creator**（官方） |
| Claude.ai / Cowork 环境适配 | **skill-creator**（官方） |
| CLI SKILL 工程规范（错误码、输出协议、配置、权限） | **本 SKILL** |
| Output Schema（SKILL 间合约、避免 compose 幻觉） | **本 SKILL** |
| 文档 SKILL 时效与质量（valid_until、golden-set） | **本 SKILL** |
| frontmatter 合规细节（metadata 嵌套、allowed-tools） | **本 SKILL** |
| Semver 版本化与 Breaking change 流程 | **本 SKILL** |
| 质量检查清单（CLI/docs/通用三栏） | **本 SKILL** |

**不要单独使用本 SKILL** —— 它假设你已经按 skill-creator 的工作流走完
"需求捕获 → 草稿 → 测试 → 迭代"，本 SKILL 在每个阶段补充工程规范。
完整 hand-off 索引见 `references/standard.md` §0.5。

## 使用步骤

创建新 SKILL 时按此顺序执行：

1. 读 `references/standard.md` §1 — 确定类型（CLI / 文档 / 复合）
2. 读 §2.0 — 理解渐进披露原则（决定内容放哪一层）
3. 读 §2.2 — 写 description（触发词优先 + pushy 写法 + 负例排除）
4. 读 §3 — 确认目录结构（CLI 用 `docs/`，文档 SKILL 用 `references/`）
5. 读 §8、§9 — 填写 output_schema 和 permissions（放入 metadata 字段）
6. 读 §9.4 — 填写 allowed-tools（frontmatter 顶层字段）
7. 读 §10 — 写最小测试集（CliRunner 单测 / golden-set）
8. 用 §12 检查清单验收后，交给 skill-creator 打包

修改已有 SKILL 时：
- 直接用 §12 检查清单审查
- 补全缺失的 `metadata.output_schema` 或 `metadata.permissions`
- 确认 `allowed-tools` 已在顶层声明
- 对照 §2.2 优化 description 触发词写法

## 与 skill-creator 的衔接时机

**本 SKILL → skill-creator**：
- 完成 §1–§3（类型决策 + SKILL.md 草稿 + 目录）后，调用 skill-creator 的 "Capture Intent + Interview" 阶段确认需求
- 完成 §10 最小测试集后，调用 skill-creator 的 "Run and evaluate test cases" 跑评估
- 完成 §12 检查清单后，调用 skill-creator 的 `package_skill.py` 打包

**skill-creator → 本 SKILL**：
- skill-creator 写 SKILL.md 时遇到 description 写法疑问 → 读本 SKILL §2.2
- skill-creator 跑评估发现 undertrigger → 读本 SKILL §2.2 的"pushy 写法"优化触发词
- skill-creator 准备打包前 → 读本 SKILL §12 检查清单
- skill-creator 配置 `allowed-tools` 时 → 读本 SKILL §9.4

## 关键规范速查

### description 写法（最高优先级）

description 是 Agent 路由的核心依据，必须：
- **以动词 + 场景列举开头**，不以名词开头
- **包含 1-2 个负例排除**，帮助 Agent 避免误触发
- **长度 100–300 字为宜**（硬上限 1024 字符），关键词前置（前 30 字内出现主要触发词）
- **禁含 `<` 或 `>`**：会被 quick_validate.py 拒绝（用 `{` `}` 替代占位符）
- **偏激进（pushy）写法**：主动列举多种用户措辞，对抗 Claude 默认的 undertrigger 倾向

好的写法（CLI 类型）：
```
当用户需要从 git diff、staged changes 生成符合 Conventional Commits 规范的提交消息时使用此 SKILL。
覆盖：补全 commit 类型前缀（feat/fix/refactor/docs）、推断 scope、生成多行 body。
也用于：用户问"这次改动该怎么写 commit"、"帮我写个规范的提交信息"。
不适用于：实际执行 git commit、修改 git 配置。
```

差的写法：
```
Git commit 工具，封装了 commit 消息生成功能。
```

### frontmatter 合规字段

打包验证只允许顶层字段：`name, description, license, allowed-tools, metadata, compatibility`

自定义字段（output_schema、permissions、valid_until 等）一律放入 `metadata`；
`allowed-tools` 是顶层合规字段，**不放在** metadata 下：

```yaml
---
name: my-skill
description: "当用户需要... 不适用于..."
license: MIT
allowed-tools: [Read, Bash]
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

- §0 Agent 使用步骤
- §1 SKILL 类型与决策矩阵
- §2.0 渐进披露原则（L1/L2/L3 分层）
- §2.1–§2.4 SKILL.md 完整规范（含 description 模板）
- §3 项目目录结构（扁平 / 模块化 / 文档 / 复合）
- §4.0 CLI 核心约定（语言无关）
- §4.1–§4.5 CLI SKILL 开发（Click / Typer 完整模板）
- §4.6 多语言对照（Bash / Node.js / Go / Rust）
- §5 文档 SKILL 开发（时效机制、内容模板）
- §6 配置管理（优先级、类模板）
- §7 错误处理（exit code、JSON 格式）
- §8 输出 Schema（TypedDict + JSON Schema）
- §9.1–§9.3 权限与沙箱声明
- §9.4 allowed-tools 字段（顶层，非 metadata 内）
- §10 测试规范（CliRunner 模板、golden-set）
- §11 版本化与兼容策略（semver、MCP annotation）
- §12 质量检查清单（CLI / 文档 / 通用三栏）
- §13 SKILL 最小模板

读取时机：遇到具体问题（如"CliRunner 怎么写"）时再读对应章节，不必全文加载。
