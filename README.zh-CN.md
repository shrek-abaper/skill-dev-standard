<div align="center">

# 📐 skill-dev-standard

[English](README.md) &nbsp;·&nbsp; **中文**

### 智能体 SKILL 开发的工程级规范

#### *Anthropic 官方 skill-creator 的工程化补充——退出码、输出合约、权限、版本化与发布检查清单。*

> 两个 SKILL 必须同时加载：skill-creator 负责工作流（访谈、评估、迭代、打包），skill-dev-standard 负责工程化（CLI 约定、Output Schema、文档时效、frontmatter 合规、Semver、质量门禁）。只用其中任意一个都会留下盲区。

**退出码 &nbsp;·&nbsp; Output Schema &nbsp;·&nbsp; 配置与凭证 &nbsp;·&nbsp; 权限与沙箱**

**渐进披露 &nbsp;·&nbsp; 对抗 undertrigger 的 description &nbsp;·&nbsp; Golden-Set 评估 &nbsp;·&nbsp; Semver 门禁**

**CLI · 文档 · 混合 &nbsp;·&nbsp; Click / Typer 模板 &nbsp;·&nbsp; 7 个聚焦参考文件 &nbsp;·&nbsp; v2.2.0**

#### 为真正交付 SKILL 的作者而建

[![GitHub Stars](https://img.shields.io/github/stars/shrek-abaper/skill-dev-standard?style=flat-square&color=FFD700&logo=github&logoColor=white&label=Stars)](https://github.com/shrek-abaper/skill-dev-standard/stargazers) [![GitHub Forks](https://img.shields.io/github/forks/shrek-abaper/skill-dev-standard?style=flat-square&color=6E40C9&logo=github&logoColor=white&label=Forks)](https://github.com/shrek-abaper/skill-dev-standard/network/members) [![Contributors](https://img.shields.io/github/contributors/shrek-abaper/skill-dev-standard?style=flat-square&color=2EA043&logo=github&logoColor=white)](https://github.com/shrek-abaper/skill-dev-standard/graphs/contributors) [![Last Commit](https://img.shields.io/github/last-commit/shrek-abaper/skill-dev-standard?style=flat-square&logo=github&logoColor=white)](https://github.com/shrek-abaper/skill-dev-standard/commits/main) [![Version](https://img.shields.io/badge/version-v2.2.0-0066CC?style=flat-square)](./changelog.md) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](./README.zh-CN.md#license) [![Pairs With](https://img.shields.io/badge/pairs%20with-skill--creator-DA291C?style=flat-square&logo=anthropic&logoColor=white)](https://github.com/anthropics/skills)

[![SKILL.md](https://img.shields.io/badge/SKILL.md-000000?style=for-the-badge&logo=markdown&logoColor=white)](./SKILL.md)
[![Changelog](https://img.shields.io/badge/Changelog-v2.2.0-2EA043?style=for-the-badge&logo=keepachangelog&logoColor=white)](./changelog.md)
[![skill-creator](https://img.shields.io/badge/Anthropic-skill--creator-D97757?style=for-the-badge&logo=anthropic&logoColor=white)](https://github.com/anthropics/skills)

```bash
# 打包后放入 Agent 的 skills 目录
python /path/to/skill-creator/scripts/package_skill.py .
cp skill-dev-standard.skill ~/.claude/skills/
```

</div>

大多数 SKILL 在生产中失败，不是因为提示词写得不好，而是因为工程化缺失：调用方期待 JSON，脚本却输出大段自然语言；写操作没有 dry-run 门禁；凭证被硬编码进代码；到了打包时，谁也说不清"完成"的标准是什么。

skill-dev-standard 是官方 skill-creator 旁边的工程化层：语言无关的 CLI 合约（退出码规范、JSON Output Schema、配置优先级与凭证规则）、对抗 undertrigger 的 SKILL.md 写法、文档时效机制（`valid_until`、`source_urls`、golden-set 问答对）、带 breaking change 流程的 Semver，以及每次发布必过的 §12 质量检查清单。skill-creator 依然是工作流引擎，本 SKILL 补上它刻意不覆盖的工程缺口。

> [!NOTE]
> **是补充，不是替代。** skill-creator 负责访谈、评估工具链与打包合规；skill-dev-standard 负责 CLI/文档工程规范与质量检查清单。每次阶段切换都查 SKILL.md 的 §0.5 hand-off 索引——它明确规定了每个章节归谁负责。

**适合谁：**

- **编写 CLI 工具的 SKILL 作者** —— 退出码规范、Output Schema、Click/Typer 模板、配置与权限约定，以及 CliRunner 测试
- **需要统一 SKILL 规范的团队** —— 一份 frontmatter 合约、一套 Semver 策略、一份发布检查清单，适用于所有交付的 SKILL
- **文档类 SKILL 维护者** —— `valid_until` / `source_urls` 时效规则与 golden-set 覆盖验证，让参考知识不会悄悄过期
- **SKILL 审查者与维护者** —— §12 三栏检查清单（CLI / 文档 / 通用）作为打包前门禁
- **封装内部 API 的企业集成开发者** —— 凭证安全、写操作 dry-run 先行、下游智能体可依赖的机器可读输出

**[定位](#定位)** &nbsp;·&nbsp; **[安装](#安装)** &nbsp;·&nbsp; **[Quick Start](#quick-start)** &nbsp;·&nbsp; **[内容结构](#内容结构)** &nbsp;·&nbsp; **[关键规范](#关键规范一览)** &nbsp;·&nbsp; **[Changelog](#changelog)**

---

## 定位

本 SKILL **不替代** skill-creator，而是其**工程化补充**。两者职责互补：

| 关注层面 | 由谁负责 |
|---------|---------|
| 开发流程（访谈、测试、评估、迭代、打包） | **skill-creator**（官方） |
| 设计哲学（Progressive Disclosure、写作风格） | **skill-creator**（官方） |
| 评估工具链（eval-viewer、run_loop、benchmark） | **skill-creator**（官方） |
| Claude.ai / Cowork 环境适配 | **skill-creator**（官方） |
| CLI SKILL 工程规范（错误码、输出协议、配置、权限） | **本 SKILL** |
| Output Schema（SKILL 间合约） | **本 SKILL** |
| 文档 SKILL 时效与质量（`valid_until`、golden-set） | **本 SKILL** |
| frontmatter 合规细节（`metadata` 嵌套、`allowed-tools`） | **本 SKILL** |
| Semver 版本化与 Breaking change 流程 | **本 SKILL** |
| 质量检查清单（CLI / docs / 通用三栏） | **本 SKILL** |

**应当同时加载**——单独使用任意一个都会有盲区。

## 安装

**方式 1：打包安装（推荐用于分发）**

```bash
# 在本仓库根目录用 skill-creator 提供的脚本打包
python /path/to/skill-creator/scripts/package_skill.py .

# 安装到本地 Agent skills 目录
cp skill-dev-standard.skill ~/.claude/skills/
# 或对应你 Agent 工具的 skills 路径（OpenCode / Cursor 等）
```

**方式 2：直接克隆（推荐用于本地开发）**

```bash
git clone <repo-url> ~/.claude/skills/skill-dev-standard
```

## 推荐配置（强烈建议）

为避免 Claude 因 undertrigger 倾向只加载其中一个 SKILL，在你的 `~/.claude/CLAUDE.md`（用户级系统提示）中追加以下内容。

**直接复制下面这段到 `~/.claude/CLAUDE.md`：**

```markdown
## SKILL 开发工作流（强制）

创建、修改、审查、打包任何 SKILL 时：

1. 必须同时加载两个 SKILL：
   - skill-creator（流程引擎 + 评估工具链）
   - skill-dev-standard（工程规范 + 检查清单）

2. 阶段切换时主动查表：每个阶段读 skill-dev-standard 的 §0.5 hand-off 索引，
   确认两个 SKILL 各自负责的章节，不要只读其中一个

3. 打包前强制双重验证：
   - 跑 skill-creator 的 package_skill.py（合规性）
   - 对照 skill-dev-standard 的 §12 检查清单（工程质量）
```

这是系统级指令，比 SKILL description 暗示更可靠。Claude Code / OpenCode 每次会话开始都会读取 `CLAUDE.md`，可保证两个 SKILL 都被加载。

## Quick Start

新建一个 CLI SKILL 的最短路径：

1. 让 Claude 同时加载 skill-creator + skill-dev-standard
2. skill-creator 走 Capture Intent → Interview → 起草 SKILL.md
3. **hand-off 到本 SKILL**：
   - 读 SKILL.md §1 选 SKILL 类型 → CLI
   - 读 `references/skillmd-authoring.md` §2.2 写 description（动词列举 + 多场景 + 偏 pushy）
   - 读 `references/cli-templates.md` §4 套 Click 模板（或 §4.6 选其他语言方案）
   - 读 `references/permissions.md` §9 填 permissions、§9.4 填 allowed-tools
4. skill-creator 跑 evals 迭代
5. 用 `references/quality-and-release.md` §12 检查清单验收 → skill-creator 调 `package_skill.py` 打包

完整 hand-off 索引见 [SKILL.md §0.5](SKILL.md)。

## 内容结构

```
skill-dev-standard/
├── SKILL.md                          # L2 路由层：定位声明 + §0.5 hand-off 索引 + §1 类型决策 + 关键规范速查 + Reference Files Index
├── README.md                         # 英文版文档（主语言）
├── README.zh-CN.md                   # 本文件：中文版文档
├── changelog.md                      # 完整版本历史与迁移说明
├── evals/
│   └── golden-set.yaml               # 知识覆盖验证问答对
└── references/                       # L3 按需加载：7 个聚焦参考文件
    ├── skillmd-authoring.md          # §2：渐进披露、必备要素、description 写法、完整模板
    ├── project-structure.md          # §3：四类目录结构图 + §3.5 assets/ 规范
    ├── cli-templates.md              # §4：核心约定、Click/Typer 完整模板、多语言对照
    ├── docs-development.md           # §5：内容组织、质量标准、时效机制、文档模板
    ├── cli-runtime.md                # §6/§7/§8：配置管理、错误处理、Output Schema
    ├── permissions.md                # §9：permissions 字段 + §9.4 allowed-tools
    ├── testing.md                    # §10：CliRunner 单测、golden-set 问答对
    └── quality-and-release.md       # §11/§12/§13/附录：版本化、质量清单、最小模板
```

## 关键规范一览

| 章节 | 内容 | 参考文件 |
|------|------|---------|
| §0 / §0.5 | Agent 检查顺序 + hand-off 索引 | `SKILL.md` |
| §1 | SKILL 类型决策（CLI / docs / hybrid） | `SKILL.md` §1 |
| §2 | SKILL.md 规范（渐进披露、description 写法、完整模板） | `references/skillmd-authoring.md` |
| §3 | 项目结构（四类结构图 + §3.5 assets/ 目录规范） | `references/project-structure.md` |
| §4 | CLI SKILL 开发（§4.0 语言无关核心约定 + Click/Typer 模板 + §4.6 多语言对照） | `references/cli-templates.md` |
| §5 | 文档 SKILL 开发（含时效机制 `valid_until` / `source_urls`） | `references/docs-development.md` |
| §6 | 配置管理（优先级 + dataclass 模板） | `references/cli-runtime.md` |
| §7 | 错误处理（exit code 规范 + JSON 格式） | `references/cli-runtime.md` |
| §8 | Output Schema（TypedDict + JSON Schema） | `references/cli-runtime.md` |
| §9 | 权限与沙箱（§9.1–§9.3 permissions、§9.4 allowed-tools） | `references/permissions.md` |
| §10 | 测试规范（CliRunner + golden-set） | `references/testing.md` |
| §11 | 版本化（Semver + Breaking change 流程 + MCP annotation 对齐） | `references/quality-and-release.md` |
| §12 | 质量检查清单（CLI / docs / 通用三栏） | `references/quality-and-release.md` |
| §13 | 最小模板（CLI / docs 两套） | `references/quality-and-release.md` |

## Changelog

### v2.2.0 — 2026-05-14

**新增 assets/ 工程化规范**：

- **新增**：§3.5 `assets/` 目录规范——明确四类目录职责边界（`scripts/` 执行、`references/` 理解、`docs/` 查阅、`assets/` 嵌入/交付）
- **新增**：占位符约定（`__VARIABLE__` 双下划线，对齐 skill-creator 官方）、SKILL.md 引用表格模式、大小约束（< 100 KB/文件）、安全要求（禁止真实凭据/PII）
- **新增**：`§0.5` hand-off 新增「需要输出模板/配置样板」触发行
- **更新**：§3.1–§3.4 结构图均补充 `assets/` 可选项；§12 检查清单新增 assets/ 验收项；§13 模板补充 `assets/` 行

### v2.1.0 — 2026-05-10

**通用化重构 + frontmatter 合规修复**：

- **修复**：frontmatter 模板对齐官方 `quick_validate.py`（自定义字段全部嵌入 `metadata:` 嵌套）
- **修复**：description 占位符改用 `{}`（避免 `<>` 触发打包验证错误）
- **新增**：§0.5 与 skill-creator 的 hand-off 索引、§2.0 渐进披露原则、§4.0 语言无关核心约定、§4.6 多语言对照、§9.4 allowed-tools 字段
- **优化**：description 写法指引（100–300 字、pushy 写法、对抗 undertrigger）
- **去私有化**：移除全部私有业务示例和本地路径引用，替换为通用领域示例（Git commit 生成器、AWS S3 知识库）
- **去除**：`archives/` 旧版本归档目录（依赖 git 历史保留）

### v2.0.0 — 2026-05-05

初始公开版本，整合 CLI 和文档 SKILL 规范。

## 反馈与贡献

如发现规范与官方 skill-creator 存在冲突，或有未覆盖的工程场景，欢迎提 issue。

## License

MIT
