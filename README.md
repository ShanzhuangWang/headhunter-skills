# headhunter-skills

猎头 / 招聘场景自建 **WorkBuddy Skill** 合集。按类别存放各类人员检索、背调、触达、内部提效等自动化 skill，全部由本人（ShanzhuangWang）在实践中沉淀、自用。

> 定位：AI Talent Partner 的工具箱。每个 skill 对应招聘流水线中的一个可复用环节。

## 仓库结构

目前按"先平铺"组织：每个 skill 放在自己的目录里（这是 WorkBuddy skill 的规范，目录名即 skill 名），后续会按招聘流程加顶层分类目录。

```
headhunter-skills/
├── README.md
└── author-backfill/              # 技术报告 / 论文作者信息检索回填
    ├── SKILL.md                  # WorkBuddy / Claude Code（两平台通用 SKILL.md）
    ├── PROMPT.md                 # 通用 Markdown / 任意助手（可粘贴的纯指令版）
    └── adapters/
        ├── cursor-rules.md       # Cursor 项目规则
        ├── chatgpt-instructions.md  # ChatGPT / 自定义 GPT Instructions
        └── codex.md              # OpenAI Codex CLI（codex.md / AGENTS.md）
```

## Skill 索引

| Skill | 用途 | 状态 |
|-------|------|------|
| [author-backfill](./author-backfill/) | 给定公司/团队的技术报告或论文（PDF / arXiv / 团队名），提取全部作者并批量检索中文名、学历、Google Scholar、OpenReview、主页、LinkedIn、GitHub、邮箱、研究方向，闭环交叉验证后回填飞书多维表格 | ✅ 可用 |

## 如何使用

将某个 skill 目录整体复制到 WorkBuddy 的 skills 目录即可启用：

- **用户级（所有项目可用）**：`~/.workbuddy/skills/<skill-name>/`
- **项目级（团队共享）**：`<项目>/.workbuddy/skills/<skill-name>/`

例如：

```bash
cp -r author-backfill ~/.workbuddy/skills/
```

复制后在与 WorkBuddy 对话时直接描述需求（如"帮我把这份技术报告的作者信息检索回填到飞书表"）即可触发对应 skill。

## 跨平台使用（每个 skill 的文件布局）

本仓库的 skill 设计为**平台无关的知识核心 + 薄适配层**，不只能在 WorkBuddy 用：

| 平台 | 用哪个文件 | 怎么用 |
|------|-----------|--------|
| **WorkBuddy** | `SKILL.md` | 整个目录复制到 `~/.workbuddy/skills/`，对话即触发 |
| **Claude Code** | `SKILL.md` | 同样复制到项目的 `~/.claude/skills/<name>/`（Claude Code 也吃 `SKILL.md`，frontmatter 格式一致，几乎零改动） |
| **Cursor** | `adapters/cursor-rules.md` | 内容放进 `.cursor/rules/author-backfill.mdc` 或项目根 `.cursorrules` |
| **ChatGPT / 自定义 GPT** | `adapters/chatgpt-instructions.md` | 整段作为自定义 GPT 的 **Instructions** 粘贴 |
| **OpenAI Codex CLI** | `adapters/codex.md` | 保存为仓库根 `codex.md`（或 `AGENTS.md`）被 Codex 读取 |
| **任意助手 / 通用 Markdown** | `PROMPT.md` | 纯 Markdown 指令版，复制粘贴到任何支持长提示词的助手即可 |

> **核心说明**：`SKILL.md` 与 `PROMPT.md` 的方法论完全一致；差异只在平台胶水层（如 `lark-cli` 回填命令）。换平台时把表格工具命令替换成你手头的即可，知识本身通用。后续每加一个 skill，都按 `SKILL.md` + `PROMPT.md` + `adapters/` 这套布局即可。

## 贡献 / 规范

- 每个 skill 必须有 `SKILL.md`，含 YAML frontmatter（`name` / `summary` / `agent_created`）与正文。
- 正文写明触发场景、端到端流程、关键避坑（铁律）、失败处理。
- 后续计划按招聘流程分类：talent-research（人才检索/回填）、candidate-screening（背调/筛选）、outreach（触达）、internal-tools（内部工具）。
