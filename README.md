# skill-dev-standard

> 工程级 SKILL 开发规范，与 Anthropic 官方 [skill-creator](https://github.com/anthropics/skills) 协同使用。

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
| 文档 SKILL 时效与质量（valid_until、golden-set） | **本 SKILL** |
| frontmatter 合规细节（metadata 嵌套、allowed-tools） | **本 SKILL** |
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

这是系统级指令，比 SKILL description 暗示更可靠。Claude Code / OpenCode 每次会话开始都会读取 CLAUDE.md，可保证两个 SKILL 都被加载。

## Quick Start

新建一个 CLI SKILL 的最短路径：

1. 让 Claude 同时加载 skill-creator + skill-dev-standard
2. skill-creator 走 Capture Intent → Interview → 起草 SKILL.md
3. **hand-off 到本 SKILL**：
   - 读 §1 选 SKILL 类型 → CLI
   - 读 §2.2 写 description（动词列举 + 多场景 + 偏 pushy）
   - 读 §4 套 Click 模板（或 §4.6 选其他语言方案）
   - 读 §9 填 permissions、§9.4 填 allowed-tools
4. skill-creator 跑 evals 迭代
5. 用 §12 检查清单验收 → skill-creator 调 `package_skill.py` 打包

完整 hand-off 索引见 [`references/standard.md` §0.5](references/standard.md)。

## 内容结构

```
skill-dev-standard/
├── SKILL.md              # 速查规范：定位声明 + 与 skill-creator 的衔接 + 关键规范速查
├── README.md             # 本文件：安装、配置、Quick start
└── references/
    └── standard.md       # 完整规范（v2.1.0，13 章 + 附录）
```

## 关键规范一览

| 章节 | 内容 |
|------|------|
| §0 | Agent 检查顺序（工程规范的检查点，不是开发流程） |
| §0.5 | 与 skill-creator 的 hand-off 索引 |
| §1 | SKILL 类型决策（CLI / docs / hybrid） |
| §2 | SKILL.md 规范（含 §2.0 渐进披露、§2.2 description 写法） |
| §3 | 项目结构（含 CLI 用 `docs/` vs 文档用 `references/` 命名约定） |
| §4 | CLI SKILL 开发（§4.0 语言无关核心约定 + Click/Typer 模板 + §4.6 多语言对照） |
| §5 | 文档 SKILL 开发（含时效机制 valid_until / source_urls） |
| §6 | 配置管理（优先级 + dataclass 模板） |
| §7 | 错误处理（exit code 规范 + JSON 格式） |
| §8 | Output Schema（TypedDict + JSON Schema） |
| §9 | 权限与沙箱（§9.1–§9.3 permissions、§9.4 allowed-tools） |
| §10 | 测试规范（CliRunner + golden-set） |
| §11 | 版本化（Semver + Breaking change 流程 + MCP annotation 对齐） |
| §12 | 质量检查清单（CLI / docs / 通用三栏） |
| §13 | 最小模板（CLI / docs 两套） |

## Changelog

### v2.1.0 — 2026-05-10

**通用化重构 + frontmatter 合规修复**：

- **修复**：frontmatter 模板对齐官方 `quick_validate.py`（自定义字段全部嵌入 `metadata:` 嵌套）
- **修复**：description 占位符改用 `{}`（避免 `<>` 触发打包验证错误）
- **新增**：§0.5 与 skill-creator 的 hand-off 索引、§2.0 渐进披露原则、§4.0 语言无关核心约定、§4.6 多语言对照、§9.4 allowed-tools 字段
- **优化**：description 写法指引（100–300 字、pushy 写法、对抗 undertrigger）
- **去私有化**：移除全部 SAP 业务示例和本地路径引用，替换为通用领域示例（Git commit 生成器、AWS S3 知识库）
- **去除**：`archives/` 旧版本归档目录（依赖 git 历史保留）

完整变更见 `references/standard.md` 附录 C。

### v2.0.0 — 2026-05-05

初始公开版本，整合 CLI 和文档 SKILL 规范。

## 反馈与贡献

如发现规范与官方 skill-creator 存在冲突，或有未覆盖的工程场景，欢迎提 issue。

## License

MIT
