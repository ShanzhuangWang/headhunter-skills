# Cursor 项目规则 —— author-backfill

> 放在仓库 `.cursor/rules/author-backfill.mdc` 或项目根 `.cursorrules`，类型选 "Always" 或按需 "Agent Requested"。

## 触发场景
用户要求：从技术报告 / 论文（PDF、arXiv、团队名）提取**全部作者**，并批量检索每位作者的中文名、学历、Google Scholar、OpenReview、主页、LinkedIn、GitHub、邮箱、研究方向，整理 / 回填到表格。

## 表格列结构（不动列绝不覆盖）
A=英文名(不动)、B=中文名、C=学历、D=Scholar、E=OpenReview、F=主页、G=LinkedIn、H=GitHub、I=X(不动)、J=邮箱、K=微信、L=电话、M=具体方向。

## 铁律（必须遵守）
1. 大规模并行检索时，**各检索单元只产出本地结果，由主流程合并后一次性入库**，避免并发写同一表导致覆盖。
2. **逐行写精确单元格，绝不写连续块**（连续块会覆盖相邻行 → 数据丢失）。
3. 每批独立子目录 / 独立输出文件，CSV 不带表头。
4. 完成后**必扫全空行**并修复。
5. 邮箱必须走 **EMAIL DEEP-DIVE 全清单**（PDF 脚注 → 主页 mailto → GitHub bio/profile README/CV → Scholar 域名 → OpenReview → 机构规律推断），只查 Scholar 就放弃 = 不合格。
6. **负向证据留痕**：某列穷尽仍无结果，在备注写 `【已查:Scholar/SS/DBLP/OR/GitHub/校网/LinkedIn 均无X】`，避免重复劳动。
7. 频率限制（429）重试降速。

## 核心方法论（要点）
- **四列闭环交叉验证**：以团队论文为消歧 ground truth → 优先学术主页 → GitHub 反挖 → Scholar/OpenReview 反填 → 任一入口顺藤摸瓜确认各列指同一人。
- 穷尽顺序：Scholar → Semantic Scholar → DBLP → OpenReview → llm-people → aminer → arXiv → GitHub → 校网 → LinkedIn。无 Scholar 用 DBLP/ORCID 替代并注明。
- 置信分级每格 `内容｜标签｜依据`（确认 / 高可信 / 推测需人工确认）。LinkedIn 仅头像职位明确匹配才填。

## 完整方法论
见同目录 `PROMPT.md`（通用版）或 `SKILL.md`（WorkBuddy / Claude Code 版）。执行前先读 `PROMPT.md` 获取四列闭环与 EMAIL DEEP-DIVE 全文。
