# headhunter-skills

> 跨平台通用的 **猎头 / 人才研究 AI Skill 合集** —— 一套方法论，多个 AI 助手都能跑。

本仓库收集我在猎头与人才研究工作中沉淀的 AI skill（指令集）。每个 skill 都以**平台无关**的方式编写，可在 WorkBuddy、Claude Code、Cursor、ChatGPT、OpenAI Codex 等多种 AI 助手上使用，不绑定任何单一平台。

> 定位：AI Talent Partner 的工具箱。每个 skill 对应招聘流水线中的一个可复用环节（找人、验证、触达、提效）。

## 这是什么

- **不是某个平台的私有格式**，而是基于通用 Markdown 的方法论 + 薄平台适配层。
- 每个 skill 的核心是一份可复用、可审计的人工经验（怎么找人、怎么验证、怎么避免踩坑）。
- 平台差异只在外层胶水（如"把结果写入飞书表"这类执行动作），换平台时替换对应部分即可，知识本身通用。

## 适用平台

| 平台 | 使用的文件 | 安装 / 启用方式 |
|------|-----------|----------------|
| **WorkBuddy** | `SKILL.md` | 复制目录到 `~/.workbuddy/skills/` |
| **Claude Code** | `SKILL.md` | 复制目录到 `~/.claude/skills/<name>/`（同样吃 `SKILL.md`，frontmatter 一致，几乎零改动） |
| **Cursor** | `adapters/cursor-rules.md` | 放入 `.cursor/rules/` 或项目根 `.cursorrules` |
| **ChatGPT / 自定义 GPT** | `adapters/chatgpt-instructions.md` | 整段粘贴到自定义 GPT 的 **Instructions** |
| **OpenAI Codex CLI** | `adapters/codex.md` | 存为仓库根 `codex.md` / `AGENTS.md` |
| **任意助手 / 通用 Markdown** | `PROMPT.md` | 纯 Markdown 指令版，复制粘贴即用 |

## 仓库结构

```
headhunter-skills/
├── README.md
└── author-backfill/
    ├── SKILL.md                      # WorkBuddy / Claude Code 原生格式（frontmatter: name/summary）
    ├── PROMPT.md                     # 平台无关的纯指令版（任何助手可用）
    └── adapters/
        ├── cursor-rules.md           # Cursor 项目规则
        ├── chatgpt-instructions.md   # ChatGPT / GPTs Instructions
        └── codex.md                  # OpenAI Codex CLI（codex.md / AGENTS.md）
```

> 每个 skill 由「原生格式 + 通用指令版 + 平台适配层」组成，结构统一，便于跨平台复用。

## Skill 索引

| Skill | 用途 | 状态 |
|-------|------|------|
| [author-backfill](./author-backfill/) | 给定公司 / 团队的技术报告或论文（PDF / arXiv / 团队名），提取全部作者并批量检索中文名、学历、Google Scholar、OpenReview、主页、LinkedIn、GitHub、邮箱、研究方向，闭环交叉验证后回填飞书多维表格 | ✅ 可用 |

## 如何新增一个 skill

每个新 skill 请按统一布局添加，保证天然跨平台：

```
<skill-name>/
├── SKILL.md                    # 两平台通用原生格式（WorkBuddy / Claude Code）
├── PROMPT.md                   # 平台无关纯指令版
└── adapters/
    ├── cursor-rules.md         # Cursor
    ├── chatgpt-instructions.md # ChatGPT / GPTs
    └── codex.md                # OpenAI Codex
```

## 设计原则

1. **方法论优先**：核心知识写成纯 Markdown，与执行工具解耦。
2. **薄适配层**：平台相关命令集中在 `adapters/` 与各平台头部，互不污染。
3. **可审计**：每个结论带证据来源与置信度，结论可复核。

## 后续计划

按招聘流程逐步补充分类：talent-research（人才检索 / 回填）、candidate-screening（背调 / 筛选）、outreach（触达）、internal-tools（内部工具）。
