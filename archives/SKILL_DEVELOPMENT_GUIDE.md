# SKILL 开发规范

> 智能体创建 AI Agent Skills 的完整参考文档，涵盖 CLI Skills 和文档 Skills。
>
> **版本**: 2.0.0 | **更新**: 2026-05-05

---

## 目录

- [0. Agent 使用本规范的步骤](#0-agent-使用本规范的步骤)
- [1. SKILL 类型与选择](#1-skill-类型与选择)
- [2. SKILL.md 规范](#2-skillmd-规范)
- [3. 项目结构](#3-项目结构)
- [4. CLI SKILL 开发](#4-cli-skill-开发)
- [5. 文档 SKILL 开发](#5-文档-skill-开发)
- [6. 配置管理](#6-配置管理)
- [7. 错误处理](#7-错误处理)
- [8. 输出 Schema](#8-输出-schema)
- [9. 权限与沙箱声明](#9-权限与沙箱声明)
- [10. 测试规范](#10-测试规范)
- [11. 版本化与兼容策略](#11-版本化与兼容策略)
- [12. 质量检查](#12-质量检查)
- [13. 模板](#13-模板)
- [附录](#附录)

---

## 0. Agent 使用本规范的步骤

**在创建或修改任何 SKILL 之前，按此顺序执行：**

```
1. 读 §1   → 确定 SKILL 类型（CLI / 文档 / 复合）
2. 读 §2   → 起草 SKILL.md（重点：description 触发词 + output_schema）
3. 读 §3   → 创建目录结构
4. 读 §4或§5 → 套用对应模板写实现
5. 读 §9   → 填写权限声明
6. 读 §10  → 写最小测试集
7. 用 §12  → 逐条核对检查清单后提交
```

---

## 1. SKILL 类型与选择

### 1.1 SKILL 分类

| 类型 | 用途 | 适用场景 | 技术栈 |
|------|------|----------|--------|
| **CLI SKILL** | 封装命令行工具或 API | 系统操作、API 集成、数据处理 | Click / Typer |
| **文档 SKILL** | 封装知识库和参考文档 | 领域知识、API 文档、最佳实践 | Markdown + 代码 |
| **复合 SKILL** | CLI + 文档混合 | 需要操作和知识结合的场景 | CLI + Docs |

### 1.2 决策矩阵

**选择 CLI SKILL 当**：
- 需要执行系统命令或操作
- 调用 REST API 进行数据交互
- 封装现有命令行工具
- 需要 CRUD 操作
- 数据格式转换或处理

**选择文档 SKILL 当**：
- 封装领域知识和最佳实践
- API 参考文档查询
- 编码规范和模式库
- 架构设计指南
- 故障排查指南

**选择复合 SKILL 当**：
- 既需要知识查询又需要执行操作
- 文档中包含可执行的代码片段
- 需要动态查询和实时操作结合

### 1.3 参考示例

| SKILL | 路径 | 类型 | 特点 |
|-------|------|------|------|
| sap-apilog-query | `skills-production/sap-apilog-query` | CLI | Click 框架、扁平结构 |
| sap-adt-cli | `/home/shrek/projects/sap-abap-cli/skills/sap-adt-cli` | CLI | Click 框架、REST API |
| lark | `/home/shrek/.agents/skills/lark` | CLI | 模块化、复杂协议 |
| frontend-design | `.agents/skills/document-skills/frontend-design` | 文档 | 设计规范、最佳实践 |

---

## 2. SKILL.md 规范

### 2.1 必备要素

每个 SKILL 必须在 `SKILL.md` 中包含以下要素：

```markdown
---
name: <skill-name>
description: <触发词优先的一句话描述，见 §2.2>
version: "1.0.0"
type: <cli|docs|hybrid>
author: <作者姓名或团队>
tags: [<tag1>, <tag2>]

# 权限声明（见 §9）
permissions:
  read_paths: []
  write_paths: []
  network_endpoints: []
  requires_elevation: false

# 输出 Schema（见 §8）
output_schema:
  format: <json|text|mixed>
  schema_ref: <可选，JSON Schema 文件路径>

# 时效声明（文档 SKILL 必填）
valid_until: <YYYY-MM-DD 或 "evergreen">
source_urls: []
---
```

### 2.2 description 字段写法规范

**description 是 Agent 做工具路由的核心依据，写法直接影响召回率。**

写法规则：
1. **以动词 + 场景列举开头**，不要以名词开头
2. **包含 1-2 个负例排除**，帮助 Agent 避免误触发
3. **长度 50-150 字**，过短丢失上下文，过长影响 embedding 质量
4. **关键词前置**，把最重要的触发词放在前 30 字以内

**好的 description（动词列举 + 负例排除）**：

```
当用户需要查询、检索、过滤 SAP API 调用日志时使用此 SKILL。
支持按时间范围、接口名称、状态码查询，并能导出为 CSV 或 JSON。
不适用于：实时监控告警、修改日志配置、查询非 SAP 系统的日志。
```

**差的 description（名词开头，无排除）**：

```
SAP API 日志查询工具，封装了日志检索功能。
```

### 2.3 完整 SKILL.md 模板

```markdown
---
name: example-skill
description: >
  当用户需要执行 X 操作、查询 Y 数据、处理 Z 文件时使用此 SKILL。
  支持批量处理、格式转换、状态检查。
  不适用于：修改系统配置、跨系统同步、需要管理员权限的操作。
version: "1.0.0"
type: cli
author: AI Agent Team
tags: [example, template]

permissions:
  read_paths: ["~/.example-skill/"]
  write_paths: ["~/.example-skill/"]
  network_endpoints: ["https://api.example.com"]
  requires_elevation: false

output_schema:
  format: json
  schema_ref: "references/output-schema.json"

valid_until: "evergreen"
source_urls: []
---

## 执行规则

### 使用场景
当用户需要执行 X 操作时调用此 SKILL。

**触发示例**：
- "获取 Y 数据"
- "处理 Z 文件"
- "查询 W 状态"

**不触发**：
- "查询 A 系统的数据"（不在支持范围内）
- "修改系统级配置"（需要管理员权限，不在此 SKILL 范围）

### 能力范围

**支持的操作**：
- 功能 1：描述
- 功能 2：描述
- 功能 3：描述

**不支持**：
- 无法直接修改系统配置
- 不支持批量操作（每次一个）

### 使用前提
- 必须已安装必要的依赖（或 SKILL 自动处理依赖）
- 需要有效的配置文件（使用 `init` 命令初始化）
- 需要网络连接

### 安全约定
- 写操作默认需要用户确认（使用 `--yes` 跳过）
- 敏感信息存储在 `~/.example-skill/config.json`，权限 0600
- 不记录任何密码或令牌到日志
- 所有网络请求使用 HTTPS，不跳过证书验证

### 输出格式
- 默认输出：JSON（机器友好）
- 可选文本输出：`--format text`
- 错误信息输出到 stderr，exit code 非 0

### 错误处理

| 错误类型 | Exit Code | 处理方式 |
|---------|-----------|----------|
| 配置缺失 | 1 | 提示用户运行 `init` 命令 |
| 网络错误 | 2 | 显示详细错误信息，建议重试 |
| 权限错误 | 3 | 提示所需权限，建议联系管理员 |
| 参数错误 | 4 | 显示正确用法示例 |
| 资源不存在 | 5 | 返回空结果 JSON，不报错退出 |

## 命令结构

- `init` — 初始化配置
- `get <resource>` — 获取资源
- `set <resource> <value>` — 设置资源（写操作，需确认）
- `list [--filter]` — 列出所有资源
- `delete <resource>` — 删除资源（危险操作，需确认）
```

### 2.4 文档 SKILL 特定要素

```markdown
---
name: api-reference
description: >
  当用户询问 XYZ API 的用法、参数含义、请求示例、错误代码时使用此 SKILL。
  不适用于：调用 API 执行操作（用 CLI SKILL）、查询运行时状态。
version: "1.0.0"
type: docs

valid_until: "2026-12-31"
source_urls:
  - "https://docs.example.com/api/v2"
---

## 执行规则

### 知识覆盖
- API 端点列表及参数说明
- 请求/响应示例（含错误情况）
- 错误代码参考
- 最佳实践与常见坑

### 知识时效
本文档基于 API v2.3，valid_until 为 2026-12-31。
超出日期后，Agent 应提示用户核实最新文档。
```

---

## 3. 项目结构

### 3.1 CLI SKILL 结构（扁平，推荐单文件场景）

```
<skill-name>/
├── SKILL.md                    # SKILL 元数据和执行规则（必选）
├── README.md                   # 用户文档（强烈推荐）
├── .env.example                # 环境变量模板（如需）
├── docs/                       # CLI 工具自身的参考资料（非 SKILL 知识库）
│   ├── api.md                 # 上游 API 文档摘要
│   └── patterns.md            # 使用模式
└── scripts/
    └── <skill_name>.py         # CLI 入口（含内嵌 lib）
```

> **注意**：CLI SKILL 使用 `docs/` 而非 `references/`，以避免与文档 SKILL 的 `references/` 混淆。

### 3.2 CLI SKILL 结构（模块化，推荐多命令场景）

```
<skill-name>/
├── SKILL.md
├── README.md
├── pyproject.toml              # uv/pip 包配置
├── docs/
│   └── <docs>.md
├── scripts/
│   └── <skill_name>_cli.py     # shim 入口脚本
└── <skill_name>_tools/         # 独立包
    ├── __init__.py
    ├── cli.py                  # 主 CLI
    ├── commands/               # 按命令分文件
    │   ├── get.py
    │   └── set.py
    ├── client.py               # HTTP / SDK 客户端
    ├── config.py               # 配置管理
    ├── schemas.py              # 输出 Schema（TypedDict）
    └── errors.py               # 错误码定义
```

### 3.3 文档 SKILL 结构

```
<skill-name>/
├── SKILL.md                    # 必选
├── README.md                   # 必选
└── references/                 # 核心知识库（文档 SKILL 专用目录名）
    ├── concepts.md            # 概念与术语
    ├── patterns.md            # 模式与最佳实践
    ├── examples.md            # 可运行示例
    ├── troubleshooting.md     # 故障排查
    ├── api-reference.md       # API 索引
    └── output-schema.json     # 可选：输出结构声明
```

### 3.4 复合 SKILL 结构

```
<skill-name>/
├── SKILL.md
├── README.md
├── pyproject.toml
├── references/                 # 知识库（复合 SKILL 保留 references/）
│   ├── guide.md
│   └── api.md
├── docs/                       # CLI 工具参考资料
└── scripts/
    └── <skill_name>.py
```

---

## 4. CLI SKILL 开发

### 4.1 依赖管理

**推荐使用 `uv`（2025 年 Python 工具链标准）**：

```toml
# pyproject.toml
[project]
name = "example-skill"
version = "1.0.0"
dependencies = [
    "click>=8.1.0",
    "requests>=2.31.0",
]

[tool.uv]
python = ">=3.11"
```

安装方式：
```bash
uv pip install -e .          # 开发安装
uvx --from . <skill-name>    # 隔离运行（推荐）
```

**如需运行时动态安装依赖**，使用 `uv` 而非 `pip`（速度更快，不污染系统环境）：

```python
import subprocess, sys

def _ensure_deps():
    """运行时按需安装依赖（内网环境请预先确认代理可用）"""
    import importlib.util
    deps = [("click", "click>=8.1.0"), ("requests", "requests>=2.31.0")]
    missing = [spec for mod, spec in deps if not importlib.util.find_spec(mod)]
    if missing:
        subprocess.run(
            ["uv", "pip", "install", "--quiet"] + missing,
            check=True
        )
```

> **注意**：企业内网环境中动态安装可能受代理或镜像限制。若存在此问题，改为在 `SKILL.md` 的"使用前提"中说明预安装步骤，由人工或 CI 完成。

### 4.2 CLI 框架选择

| 场景 | 框架 | 理由 |
|------|------|------|
| 所有场景（默认） | **Click** | 声明式、成熟稳定、行业标准 |
| 新项目 / 类型注解优先 | **Typer** | 基于 type hints，现代 API |
| 需要高度自定义解析 | argparse | 灵活但冗长，非必要不选 |

### 4.3 Click 标准模板

```python
#!/usr/bin/env python3
"""<skill-name> CLI"""

import json
import sys
import click
from pathlib import Path

# ── 版本 ──────────────────────────────────────────────────────────────────────

VERSION = "1.0.0"

def _version_callback(ctx, param, value):
    if not value or ctx.resilient_parsing:
        return
    click.echo(f"<skill-name> {VERSION}")
    ctx.exit()

# ── 主 CLI 组 ─────────────────────────────────────────────────────────────────

@click.group()
@click.option("--debug", is_flag=True, envvar="SKILL_DEBUG", help="启用调试模式")
@click.option("--version", "-v", is_flag=True, callback=_version_callback,
              expose_value=False, is_eager=True, help="显示版本号")
@click.pass_context
def cli(ctx, debug):
    """<Skill 简短描述>

    用法示例：
        <skill-name> get <resource>
        <skill-name> list --filter active
        <skill-name> set <resource> <value> --dry-run
    """
    ctx.ensure_object(dict)
    ctx.obj["debug"] = debug

# ── 初始化 ────────────────────────────────────────────────────────────────────

@cli.command("init")
@click.option("--url", prompt="服务地址", envvar="SKILL_URL", help="服务地址")
@click.option("--token", prompt="认证令牌", hide_input=True, envvar="SKILL_TOKEN")
def init_cmd(url, token):
    """初始化配置文件"""
    config_dir = Path.home() / ".<skill-name>"
    config_dir.mkdir(exist_ok=True)
    config_file = config_dir / "config.json"
    with open(config_file, "w") as f:
        json.dump({"url": url, "token": token}, f, indent=2)
    config_file.chmod(0o600)
    click.echo(json.dumps({"status": "ok", "config": str(config_file)}))

# ── 读操作 ────────────────────────────────────────────────────────────────────

@cli.command("get")
@click.argument("name")
@click.option("--format", "-f", type=click.Choice(["json", "text"]), default="json")
@click.pass_context
def get_cmd(ctx, name, format):
    """获取指定 NAME 的资源"""
    if ctx.obj["debug"]:
        click.echo(f"[DEBUG] get: {name}", err=True)
    result = {"name": name, "value": "example", "status": "ok"}
    _output(result, format)

@cli.command("list")
@click.option("--filter", "-f", "filter_", help="过滤条件")
@click.option("--format", type=click.Choice(["json", "text"]), default="json")
def list_cmd(filter_, format):
    """列出所有资源"""
    results = [{"name": "a"}, {"name": "b"}]
    _output({"items": results, "count": len(results)}, format)

# ── 写操作（含保护）─────────────────────────────────────────────────────────

@cli.command("set")
@click.argument("name")
@click.argument("value")
@click.option("--yes", "-y", is_flag=True, help="跳过确认")
@click.option("--dry-run", is_flag=True, help="模拟执行，不实际写入")
@click.pass_context
def set_cmd(ctx, name, value, yes, dry_run):
    """设置资源（写操作，默认需确认）"""
    if dry_run:
        _output({"status": "dry-run", "would_set": {name: value}}, "json")
        return
    if not yes and not click.confirm(f"确认将 {name} 设置为 {value}?"):
        click.echo(json.dumps({"status": "cancelled"}))
        return
    # 实际写入逻辑
    _output({"status": "ok", "name": name, "value": value}, "json")

# ── 工具函数 ──────────────────────────────────────────────────────────────────

def _output(data: dict, format: str = "json"):
    """统一输出：正常结果 → stdout，错误 → stderr + exit 1"""
    if format == "json":
        click.echo(json.dumps(data, ensure_ascii=False))
    else:
        for k, v in data.items():
            click.echo(f"{k}: {v}")

def _fail(msg: str, code: int = 1):
    click.echo(json.dumps({"status": "error", "message": msg}), err=True)
    sys.exit(code)

if __name__ == "__main__":
    cli()
```

### 4.4 Typer 标准模板

```python
#!/usr/bin/env python3
"""<skill-name> CLI — Typer 版"""

from typing import Annotated, Optional
import typer, json
from pathlib import Path

app = typer.Typer(help="<Skill 简短描述>", no_args_is_help=True)

@app.command()
def init(
    url: Annotated[str, typer.Option(prompt="服务地址", envvar="SKILL_URL")],
    token: Annotated[str, typer.Option(prompt="认证令牌", hide_input=True, envvar="SKILL_TOKEN")],
):
    """初始化配置文件"""
    config_dir = Path.home() / ".<skill-name>"
    config_dir.mkdir(exist_ok=True)
    config_file = config_dir / "config.json"
    with open(config_file, "w") as f:
        json.dump({"url": url, "token": token}, f, indent=2)
    config_file.chmod(0o600)
    typer.echo(json.dumps({"status": "ok", "config": str(config_file)}))

@app.command()
def get(
    name: Annotated[str, typer.Argument(help="资源名称")],
    format: str = typer.Option("json", "--format", "-f"),
    debug: bool = typer.Option(False, "--debug"),
):
    """获取指定资源"""
    if debug:
        typer.echo(f"[DEBUG] get: {name}", err=True)
    result = {"name": name, "value": "example", "status": "ok"}
    typer.echo(json.dumps(result, ensure_ascii=False))

if __name__ == "__main__":
    app()
```

### 4.5 Click 最佳实践速查

| 实践 | 写法 |
|------|------|
| 类型约束 | `type=click.Choice([...])`, `type=click.Path(exists=True)` |
| 环境变量绑定 | `envvar="VAR_NAME"` 或全局 `auto_envvar_prefix="APP"` |
| Context 共享状态 | `@click.pass_context` + `ctx.obj` |
| 版本回调 | `is_eager=True` 确保 `--version` 优先处理 |
| 确认对话框 | `click.confirm("确认?", abort=True)` |
| 错误输出 | `click.echo(msg, err=True)` |
| 进度条 | `click.progressbar(items)` |
| 分页输出 | `click.echo_via_pager(long_text)` |

---

## 5. 文档 SKILL 开发

### 5.1 内容组织原则

```
references/
├── concepts.md          # 概念和术语定义
├── patterns.md          # 设计模式和最佳实践
├── examples.md          # 可运行的代码示例
├── troubleshooting.md   # 常见问题和解决方案
├── api-reference.md     # API 或命令索引
├── output-schema.json   # 可选：输出结构 JSON Schema
└── changelog.md         # 更新历史（含 breaking changes）
```

### 5.2 文档质量标准

| 维度 | 要求 |
|------|------|
| **准确性** | 信息正确，代码示例可运行 |
| **完整性** | 覆盖主要场景和边界情况 |
| **清晰度** | 语言简洁，逻辑清晰 |
| **可查找性** | 良好的目录、索引、搜索关键词 |
| **时效性** | frontmatter 中声明 `valid_until` 和 `source_urls` |

### 5.3 时效机制

```markdown
---
valid_until: "2026-12-31"
source_urls:
  - "https://docs.example.com/api/v2"
  - "https://github.com/example/sdk/releases"
---
```

Agent 使用规则：
- `valid_until` 为具体日期且已过期 → 在回答前提示用户"文档可能过期，建议核实"
- `valid_until: "evergreen"` → 内容不依赖版本，无需提示
- `source_urls` 非空 → Agent 可主动建议用户访问源地址获取最新信息

### 5.4 文档模板

**concepts.md**：

```markdown
# 核心概念

> 版本：v2.3 | 更新：2026-01-15

## 术语表

| 术语 | 定义 |
|------|------|
| <term> | <definition> |

## 核心原理

### 原理 1：XXX
...
```

**patterns.md**：

```markdown
# 设计模式与最佳实践

## 模式 1：XXX

**问题**：描述你要解决什么问题。

**方案**：描述推荐做法。

**示例**：
```python
# 完整可运行代码
```

**注意**：列出需要避免的做法。
```

**examples.md**：

```markdown
# 代码示例

所有示例均基于 SDK v2.3，已在 Python 3.11+ 验证。

## 基础示例

### 示例 1：XXX
```python
# 完整可运行代码，含 import
```

**预期输出**：
```json
{"status": "ok", "result": "..."}
```
```

---

## 6. 配置管理

### 6.1 配置优先级

```
命令行参数 > 环境变量 > 配置文件 > 默认值
```

### 6.2 配置存储选择

| 方式 | 安全性 | 适用场景 |
|------|--------|----------|
| JSON 文件（0600 权限） | 中 | 内网工具、开发环境 |
| 系统密钥链（keyring） | 高 | 生产环境、敏感凭据 |
| 环境变量 | 中高 | CI/CD、容器化部署 |

### 6.3 配置类模板

```python
import json
import os
from dataclasses import dataclass, asdict
from pathlib import Path
from typing import Optional

@dataclass
class SkillConfig:
    url: str
    token: Optional[str] = None
    verify_ssl: bool = True
    timeout: int = 30

    @classmethod
    def from_file(cls, path: Path = Path.home() / ".<skill-name>/config.json"):
        if not path.exists():
            raise FileNotFoundError(
                f"配置文件不存在: {path}\n请运行: <skill-name> init"
            )
        with open(path, "r", encoding="utf-8") as f:
            return cls(**json.load(f))

    @classmethod
    def from_env(cls) -> Optional["SkillConfig"]:
        url = os.getenv("SKILL_URL")
        if not url:
            return None
        return cls(
            url=url,
            token=os.getenv("SKILL_TOKEN"),
            verify_ssl=os.getenv("SKILL_VERIFY_SSL", "true").lower() == "true",
        )

    @classmethod
    def load(cls) -> "SkillConfig":
        """环境变量优先于配置文件"""
        return cls.from_env() or cls.from_file()

    def save(self, path: Path = Path.home() / ".<skill-name>/config.json"):
        path.parent.mkdir(parents=True, exist_ok=True)
        with open(path, "w", encoding="utf-8") as f:
            json.dump(asdict(self), f, indent=2, ensure_ascii=False)
        path.chmod(0o600)
```

---

## 7. 错误处理

### 7.1 输出流约定

| 内容 | 输出流 | 说明 |
|------|--------|------|
| 正常结果 | stdout | 主要数据，供 Agent / 管道消费 |
| 错误 / 警告 | stderr | 异常信息 |
| 调试日志 | stderr | `--debug` 模式下输出 |

### 7.2 Exit Code 约定

所有 SKILL 使用统一的 exit code 规范，在 SKILL.md 的"错误处理"表格中声明：

| Code | 含义 |
|------|------|
| 0 | 成功 |
| 1 | 配置错误（缺失、格式错误） |
| 2 | 网络错误（超时、连接失败） |
| 3 | 权限错误 |
| 4 | 参数错误 |
| 5 | 资源不存在（有时返回 0 + 空结果更合适，视场景选择） |
| 99 | 未预期错误 |

### 7.3 错误输出格式

**统一使用 JSON 格式输出错误**（机器友好，Agent 可解析）：

```python
# errors.py
import json, sys

def fail(message: str, code: int = 99, **extra):
    """输出标准错误 JSON 到 stderr 并退出"""
    payload = {"status": "error", "code": code, "message": message}
    payload.update(extra)
    print(json.dumps(payload, ensure_ascii=False), file=sys.stderr)
    sys.exit(code)

def warn(message: str, **extra):
    """输出警告到 stderr，不退出"""
    payload = {"status": "warning", "message": message}
    payload.update(extra)
    print(json.dumps(payload, ensure_ascii=False), file=sys.stderr)
```

### 7.4 高风险操作保护

```python
@cli.command("delete")
@click.argument("name")
@click.option("--yes", "-y", is_flag=True, help="跳过确认")
@click.option("--dry-run", is_flag=True, help="模拟执行")
def delete_cmd(name, yes, dry_run):
    """删除资源（不可逆操作）"""
    if dry_run:
        click.echo(json.dumps({"status": "dry-run", "would_delete": name}))
        return
    if not yes:
        click.confirm(f"删除 {name}？此操作不可逆", abort=True)
    # 执行删除
    click.echo(json.dumps({"status": "ok", "deleted": name}))
```

---

## 8. 输出 Schema

**为什么需要 Output Schema**：当多个 SKILL 串联时，下游 SKILL 需要知道上游输出结构。Schema 是 SKILL 间合约的 source of truth，减少 Agent compose 时的幻觉错误。

### 8.1 在代码中声明（TypedDict，推荐）

```python
# schemas.py
from typing import TypedDict, List, Optional

class ResourceItem(TypedDict):
    name: str
    value: str
    status: str

class ListResult(TypedDict):
    items: List[ResourceItem]
    count: int

class ErrorResult(TypedDict):
    status: str       # 固定值 "error"
    code: int
    message: str
```

### 8.2 在 SKILL.md frontmatter 中声明

```yaml
output_schema:
  format: json          # json | text | mixed
  success_fields:
    - name: status       # 固定 "ok"
    - name: items        # List[ResourceItem]
    - name: count        # int
  error_fields:
    - name: status       # 固定 "error"
    - name: code         # int，见 §7.2
    - name: message      # str
```

### 8.3 复杂场景使用 JSON Schema 文件

```json
// references/output-schema.json
{
  "$schema": "http://json-schema.org/draft-07/schema",
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "status": { "const": "ok" },
        "items": { "type": "array", "items": { "$ref": "#/$defs/ResourceItem" } },
        "count": { "type": "integer" }
      },
      "required": ["status", "items", "count"]
    },
    {
      "type": "object",
      "properties": {
        "status": { "const": "error" },
        "code": { "type": "integer" },
        "message": { "type": "string" }
      },
      "required": ["status", "code", "message"]
    }
  ],
  "$defs": {
    "ResourceItem": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "value": { "type": "string" },
        "status": { "type": "string" }
      }
    }
  }
}
```

---

## 9. 权限与沙箱声明

**每个 SKILL 必须在 SKILL.md frontmatter 中声明其权限边界**，供 Agent 在执行前进行安全判断。

### 9.1 权限字段

```yaml
permissions:
  read_paths:
    - "~/.example-skill/"      # 配置目录
    - "/tmp/example-skill/"    # 临时目录
  write_paths:
    - "~/.example-skill/"      # 配置目录（写）
  network_endpoints:
    - "https://api.example.com"   # 具体域名，不用通配符
    - "http://proxy.internal:8080"  # 内网代理
  requires_elevation: false     # 是否需要 sudo / admin 权限
  can_spawn_processes: false    # 是否会启动子进程
  accesses_env_vars:
    - "SKILL_URL"
    - "SKILL_TOKEN"
```

### 9.2 权限声明原则

- **最小权限**：只声明实际需要的路径和端点
- **不使用通配符**：`https://*.example.com` 不如 `https://api.example.com` 明确
- **写操作必须显式列出**：读写路径分开声明
- **环境变量列举**：让 Agent 知道 SKILL 会读取哪些环境变量

### 9.3 高权限 SKILL 的额外要求

当 `requires_elevation: true` 时，必须额外说明：

```markdown
## 权限说明

此 SKILL 需要以下提升权限，原因如下：

- **写入 /etc/hosts**：用于本地开发域名映射，仅在 init 阶段执行一次
- **操作 Docker socket**：用于容器生命周期管理

Agent 在调用此 SKILL 前应向用户确认是否接受提权操作。
```

---

## 10. 测试规范

### 10.1 CLI SKILL 测试（CliRunner）

使用 `click.testing.CliRunner` 做离线单测，不依赖外部服务：

```python
# tests/test_cli.py
import json
import pytest
from click.testing import CliRunner
from scripts.example_skill import cli

@pytest.fixture
def runner():
    return CliRunner()

class TestGetCommand:
    def test_get_returns_json(self, runner):
        result = runner.invoke(cli, ["get", "my-resource"])
        assert result.exit_code == 0
        data = json.loads(result.output)
        assert data["status"] == "ok"
        assert "name" in data

    def test_get_text_format(self, runner):
        result = runner.invoke(cli, ["get", "my-resource", "--format", "text"])
        assert result.exit_code == 0
        assert "name:" in result.output

class TestSetCommand:
    def test_set_requires_confirm(self, runner):
        # 不带 --yes，模拟用户输入 'n'
        result = runner.invoke(cli, ["set", "key", "value"], input="n\n")
        assert result.exit_code == 0
        data = json.loads(result.output)
        assert data["status"] == "cancelled"

    def test_set_dry_run(self, runner):
        result = runner.invoke(cli, ["set", "key", "value", "--dry-run"])
        assert result.exit_code == 0
        data = json.loads(result.output)
        assert data["status"] == "dry-run"

    def test_set_with_yes(self, runner):
        result = runner.invoke(cli, ["set", "key", "value", "--yes"])
        assert result.exit_code == 0

class TestErrorHandling:
    def test_missing_config_exits_1(self, runner, tmp_path, monkeypatch):
        monkeypatch.setenv("HOME", str(tmp_path))
        result = runner.invoke(cli, ["get", "anything"])
        assert result.exit_code == 1
```

### 10.2 文档 SKILL 测试（Golden-set 问答对）

为文档 SKILL 维护一个最小问答对集合，用于验证知识覆盖质量：

```yaml
# tests/golden-set.yaml
skill: api-reference
version: "1.0.0"
cases:
  - id: basic-auth
    query: "如何进行 API 认证？"
    expected_keywords: ["Bearer", "Authorization", "token"]
    must_not_contain: ["密码", "明文"]

  - id: error-404
    query: "API 返回 404 怎么处理？"
    expected_keywords: ["资源不存在", "路径", "检查"]

  - id: rate-limit
    query: "请求频率限制是多少？"
    expected_keywords: ["100", "每分钟", "429"]
```

### 10.3 测试目录结构

```
<skill-name>/
├── tests/
│   ├── test_cli.py          # CLI 单测（CliRunner）
│   ├── test_config.py       # 配置加载测试
│   ├── test_schemas.py      # 输出 Schema 合规测试
│   └── golden-set.yaml      # 文档 SKILL 问答对（文档 SKILL 专用）
└── pyproject.toml           # 含 [tool.pytest.ini_options]
```

### 10.4 最小测试要求

| SKILL 类型 | 必须覆盖 |
|------------|---------|
| CLI SKILL | 所有命令的 happy path；写操作的确认/取消/dry-run；exit code 非 0 场景 |
| 文档 SKILL | ≥5 条 golden-set 问答对，覆盖主要查询场景 |
| 复合 SKILL | 两者都要 |

---

## 11. 版本化与兼容策略

### 11.1 版本号规范（Semver）

```
MAJOR.MINOR.PATCH

MAJOR：Breaking change（命令重命名、输出 Schema 字段删除/改类型）
MINOR：向后兼容的新功能（新命令、新可选输出字段）
PATCH：Bug 修复、文档更新、性能优化
```

### 11.2 Breaking Change 声明

在 SKILL.md 中维护 `changelog.md`，每次 MAJOR 版本 bump 必须记录：

```markdown
# Changelog

## [2.0.0] — 2026-05-01

### Breaking Changes
- `get` 命令输出字段 `data` 重命名为 `value`
  - **迁移方式**：将消费方代码中的 `.data` 改为 `.value`
- 移除已废弃的 `--output` 参数，统一使用 `--format`

### 新增
- `list` 命令支持 `--filter` 参数

## [1.1.0] — 2026-03-01
...
```

### 11.3 废弃流程

```
1. 在 PATCH 版本中标记废弃（代码中加 DeprecationWarning，文档中注明）
2. 在下一个 MINOR 版本中继续支持但给出警告
3. 在下一个 MAJOR 版本中移除
4. 最短废弃周期：30 天（或 2 个版本，取较长者）
```

### 11.4 MCP Annotation 对齐（对接 MCP 生态时参考）

若 SKILL 未来需要暴露为 MCP Tool，在 SKILL.md frontmatter 中预留对齐字段：

```yaml
mcp_hints:
  readOnlyHint: false       # true = 只读操作，不修改外部状态
  destructiveHint: true     # true = 可能不可逆（如 delete）
  idempotentHint: false     # true = 相同参数多次调用结果一致
  openWorldHint: true       # true = 会访问外部网络
```

---

## 12. 质量检查

### 12.1 检查清单

**基本要素**：
- [ ] SKILL.md 包含：name, description（触发词优先写法）, version, type, permissions, output_schema
- [ ] description 中有动词列举 + 负例排除
- [ ] 存在 README.md 用户文档
- [ ] 目录结构符合所选类型（CLI 用 `docs/`，文档 SKILL 用 `references/`）
- [ ] 敏感信息（.env, credentials）已加入 .gitignore

**CLI SKILL 特有**：
- [ ] 使用 Click 或 Typer，无自定义 argparse 解析
- [ ] 所有写操作有 `--yes` 跳过确认 + `--dry-run` 模拟
- [ ] 支持 `--help`，帮助文本描述清晰
- [ ] 正常输出 → stdout（JSON），错误 → stderr（JSON）
- [ ] JSON 配置文件设置 0600 权限
- [ ] 配置读取遵循：命令行 > 环境变量 > 文件 > 默认值
- [ ] 在 SKILL.md 中声明 output_schema（至少 format 字段）
- [ ] 在 SKILL.md 中声明 permissions（read_paths, write_paths, network_endpoints）
- [ ] 有 `tests/test_cli.py`，覆盖 happy path + 写操作保护 + 错误场景

**文档 SKILL 特有**：
- [ ] references/ 内容组织合理（concepts / patterns / examples / troubleshooting）
- [ ] 代码示例可运行，标注 Python 版本和 SDK 版本
- [ ] frontmatter 中有 `valid_until` 和 `source_urls`
- [ ] 有 `tests/golden-set.yaml`，至少 5 条问答对
- [ ] 有 changelog.md

**通用**：
- [ ] 版本号遵循 Semver
- [ ] Breaking change 在 changelog.md 中有迁移说明
- [ ] 有 .env.example（如使用环境变量）

---

## 13. 模板

### 13.1 CLI SKILL 最小模板

```
<skill-name>/
├── SKILL.md
├── README.md
├── .env.example
├── docs/
│   └── api.md
├── tests/
│   └── test_cli.py
└── scripts/
    └── <skill_name>.py
```

**SKILL.md 起始内容**：

```markdown
---
name: <skill-name>
description: >
  当用户需要 <动词1>、<动词2> <对象> 时使用此 SKILL。
  支持 <特性1>、<特性2>。
  不适用于：<负例1>、<负例2>。
version: "1.0.0"
type: cli
author: <作者>
tags: []

permissions:
  read_paths: []
  write_paths: []
  network_endpoints: []
  requires_elevation: false
  accesses_env_vars: []

output_schema:
  format: json

valid_until: "evergreen"
---

## 执行规则
...
```

### 13.2 文档 SKILL 最小模板

```
<skill-name>/
├── SKILL.md
├── README.md
├── tests/
│   └── golden-set.yaml
└── references/
    ├── concepts.md
    ├── patterns.md
    ├── examples.md
    └── changelog.md
```

**SKILL.md 起始内容**：

```markdown
---
name: <skill-name>
description: >
  当用户询问 <领域名称> 的 <概念/用法/示例/错误> 时使用此 SKILL。
  不适用于：执行实际操作（用对应 CLI SKILL）。
version: "1.0.0"
type: docs
author: <作者>
tags: []

valid_until: "<YYYY-MM-DD>"
source_urls:
  - "<官方文档 URL>"
---

## 执行规则

### 知识覆盖
- 核心概念与术语
- 设计模式与最佳实践
- 代码示例（已验证）
- 常见问题与排查

### 知识时效
基于 <产品/API> <版本>，valid_until 为 <日期>。
```

---

## 附录

### A. 参考资源

| 资源 | 描述 |
|------|------|
| [Click 官方文档](https://click.palletsprojects.com/) | Python CLI 框架 |
| [Typer 官方文档](https://typer.tiangolo.com/) | 现代 type hints CLI |
| [uv 官方文档](https://docs.astral.sh/uv/) | Python 包管理工具（推荐） |
| [MCP 规范](https://modelcontextprotocol.io/docs/) | Model Context Protocol |
| [JSON Schema](https://json-schema.org/) | 输出 Schema 规范参考 |

### B. 示例项目

| 项目 | 路径 | 类型 |
|------|------|------|
| sap-apilog-query | `skills-production/sap-apilog-query` | CLI（扁平） |
| sap-adt-cli | `/home/shrek/projects/sap-abap-cli/skills/sap-adt-cli` | CLI（扁平） |
| lark | `/home/shrek/.agents/skills/lark` | CLI（模块化） |
| frontend-design | `.agents/skills/document-skills/frontend-design` | 文档 |

### C. 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 2.0.0 | 2026-05-05 | 新增：Agent 执行步骤（§0）、description 触发词规范（§2.2）、输出 Schema（§8）、权限声明（§9）、测试规范（§10）、版本化策略（§11）、MCP Annotation 对齐；优化：目录名歧义（CLI 用 docs/，文档用 references/）、错误处理统一 JSON 格式、依赖管理改用 uv |
| 1.0.0 | 2026-05-05 | 初始版本，整合 CLI 和文档 SKILL 规范 |