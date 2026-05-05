# skill-dev-standard

项目级 SKILL 开发规范，供 Agent 在创建或修改 SKILL 时参考。

## 安装

将 `skill-dev-standard.skill` 文件安装到你的 Agent 环境。

## 使用方式

与 `skill-creator` 配合使用：

- `skill-creator` — 负责 SKILL 开发流程（访谈、测试、迭代、打包）
- `skill-dev-standard` — 负责项目标准（description 写法、目录结构、Schema、测试规范）

创建新 SKILL 时两者会同时触发，互补不冲突。

## 内容结构

```
skill-dev-standard/
├── SKILL.md              # 规范速查 + 与 skill-creator 的分工说明
└── references/
    └── standard.md       # 完整开发规范（2.0.0）
```

## 版本

2.0.0 — 2026-05-05
