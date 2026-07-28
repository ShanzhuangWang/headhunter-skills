# author-backfill

> 给定公司 / 团队的技术报告或论文（PDF / arXiv / 团队名），提取全部作者并批量检索中文名、学历、Google Scholar、OpenReview、主页、LinkedIn、GitHub、邮箱、研究方向，闭环交叉验证后回填飞书多维表格。

## 适用平台

本 skill 以**平台无关**的方式编写，可在多种 AI 助手上使用。各平台使用对应文件：

| 平台 | 使用的文件 | 安装 / 启用方式 |
|------|-----------|----------------|
| **WorkBuddy** | `SKILL.md` | 复制目录到 `~/.workbuddy/skills/` |
| **Claude Code** | `SKILL.md` | 复制目录到 `~/.claude/skills/<name>/`（同样吃 `SKILL.md`，frontmatter 一致，几乎零改动） |
| **Cursor** | `adapters/cursor-rules.md` | 放入 `.cursor/rules/` 或项目根 `.cursorrules` |
| **ChatGPT / 自定义 GPT** | `adapters/chatgpt-instructions.md` | 整段粘贴到自定义 GPT 的 **Instructions** |
| **OpenAI Codex CLI** | `adapters/codex.md` | 存为仓库根 `codex.md` / `AGENTS.md` |
| **任意助手 / 通用 Markdown** | `PROMPT.md` | 纯 Markdown 指令版，复制粘贴即用 |

> 核心说明：`SKILL.md` 与 `PROMPT.md` 的方法论完全一致；差异只在平台胶水层（如 `lark-cli` 回填命令）。换平台时把表格工具命令替换成你手头的即可，知识本身通用。

## 目录结构

```
author-backfill/
├── SKILL.md                      # WorkBuddy / Claude Code 原生格式（frontmatter: name/summary）
├── PROMPT.md                     # 平台无关的纯指令版（任何助手可用）
└── adapters/
    ├── cursor-rules.md           # Cursor 项目规则
    ├── chatgpt-instructions.md   # ChatGPT / GPTs Instructions
    └── codex.md                  # OpenAI Codex CLI（codex.md / AGENTS.md）
```

## 完整流程与避坑

详细触发场景、端到端流程（Phase 0→5）、关键铁律与失败处理，见 [`SKILL.md`](./SKILL.md)。
