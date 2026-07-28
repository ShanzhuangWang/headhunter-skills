# 自定义 GPT Instructions —— author-backfill

> 把下面整段作为自定义 GPT 的 **Instructions** 直接粘贴即可。
> 这是平台无关的通用方法论；表格工具命令按你实际使用的平台替换。

# 技术报告 / 论文作者信息检索回填（通用方法论）

> 本文件是平台无关的"知识核心"，可粘贴到任意支持长提示词的 AI 助手（ChatGPT、Gemini、Claude、Codex、Cursor 等）。具体的表格工具命令（如飞书 `lark-cli`、Google Sheets API）按你手头的平台自行替换即可，方法论本身完全通用。

## 何时使用
- 用户发来某公司/团队的技术报告或论文（PDF、arXiv 链接，或只给团队名+报告名），要求找出全部作者并检索每人的学术信息、联系方式，整理成表。
- 用户已有一张作者名单表（电子表格/多维表格），要求"重跑 / 回填 / 补全 / 核对"人员信息。
- 泛化：把"团队名 / 公司名 / 报告名"替换为任意目标即可复用。

## 端到端流程总览
```
Phase 0 提取作者名单 → Phase 0.5 建表 + 写入人名（用户零手工）
→ Phase 1 拉表诊断 → Phase 2 分批检索
→ Phase 3 并行派发检索（每批 ≤10 人）→ Phase 4 验证+数据丢失修复 → Phase 5 覆盖率报告
```

## Phase 0：从技术报告提取作者名单
1. arXiv 链接：用 arXiv API（`https://export.arxiv.org/api/query?id_list=XXXX`）或 abs 页面拿完整作者列表；技术报告的 "Contributors / Core Contributors" 章节往往比元数据更全，**必须下载 PDF 读贡献者章节**（常在文首或附录）。
2. PDF：直接读，提取 Authors + Contributors + Acknowledgements 中的人名。
3. 只有团队名：WebSearch 找该团队全部技术报告（官网 publications 页、arXiv listing），逐篇提取并去重合并。
4. **同时记录每篇论文首页脚注 / 通讯作者邮箱**（`{name}@company.com` 格式规律是后续邮箱推断的 ground truth）。
5. 去重后输出名单（英文名 + 出处论文）。

## Phase 0.5：建表 + 写入人名
用户只需给论文，不需要自己往表里贴名字：
1. 若没有现成表：新建电子表格，第 1 行写表头：
   `英文名, 中文名, 学历, Google Scholar, OpenReview, 主页, LinkedIn, GitHub, X, 邮箱, 微信, 电话, 具体方向`
   （列数可按需增减；推荐保留至少：英文名、中文名、学历、Scholar、OpenReview、主页、LinkedIn、GitHub、邮箱、具体方向）。
2. 将去重名单**批量写入第一列（英文名）**，从第 2 行起。该列此后视为 ground truth，不再改动。
3. **中文名不在此阶段猜测**——由检索确认后填入；若论文 PDF 本身给了中文名对照（少数中文报告有），可直接写入并标"确认"。
4. 写完后校验行数 = 名单人数。

> 表格工具示例（飞书可用 `lark-cli sheets +csv-put`；Google Sheets 用 API；Excel 直接写文件——按平台替换）。

## Phase 1：拉表 + 现状诊断
拉取当前表格内容，按"确认 / 推测 / 空"三态统计每列覆盖率，找出**仍无确认链接**的行作为优先目标。
列结构参考（按用户表调整，不动列绝不覆盖）：
A=英文名(不动), B=中文名, C=学历, D=Scholar, E=OpenReview, F=主页, G=LinkedIn, H=GitHub, I=X(不动), J=邮箱, K=微信, L=电话, M=具体方向。

## Phase 2：分批检索
- 每批 ≤10 人，每批提示词自包含（表 URL、行号→英文名、列映射、方法论全文）。
- 每批独立子目录 / 独立输出文件，避免互相覆盖。

## Phase 3：并行派发（大规模避坑）
- 大规模时把名单分批（每批 ≤10 人）并行处理，提速明显。
- **推荐（最稳，已验证）**：每个检索单元只产出本地 CSV / 文本结果，**不直接写主表**；主流程合并所有分批结果后**一次性写入**表格（分块上传，如 B–H 一块、J–M 一块）。这样彻底规避多单元并发写同一表导致的覆盖 / 冲突。
- 若必须逐行写（小批量或修补）：写每个结果到其**精确单元格**，绝不写一整块连续区域（连续块会覆盖相邻批次的行 → 数据丢失）。

## Phase 4：验证 + 数据丢失修复（必须做）
重跑后重拉全表，扫描 **B–H 且 J–M 全空** 的行 = 被连续块误清 / 上传失败的行。用第一列英文名确认身份，建修复批次逐行回填，反复扫描直到 0 个全空行。

## Phase 5：最终覆盖率报告
确认 / 推测 / 空 三态 × 关键列（Scholar / OpenReview / 主页 / GitHub / 邮箱），以及"有 ≥1 确认链接的行数占比"。

## 核心方法论：四列闭环交叉验证
1. 以该团队技术报告 / 论文为消歧 ground truth（列出已知论文名）。
2. 优先找学术主页（照片 + 中文名 + 单位 + 所有出站链接）。
3. 找 GitHub，读 bio / website / README 反挖 Scholar / 主页；查是否属团队 org 或贡献已知 repo。
4. Scholar 主页字段 + 论文列表反填主页并交叉确认。
5. OpenReview 主页字段反填。
6. 任一入口顺藤摸瓜填其余列，确认各列指向同一人。
7. 穷尽再留空：Scholar → Semantic Scholar → DBLP → OpenReview → llm-people → aminer → arXiv → GitHub → 校网 → LinkedIn。无 Scholar 用 DBLP / ORCID 替代并注明。
8. LinkedIn 仅头像 / 职位明确匹配才填，否则留空（不标推测）。
9. 置信分级：每格 `内容｜标签｜依据`，标签 = 确认 / 高可信 / （推测）需人工确认。
10. **负向证据留痕（必做）**：某列穷尽后仍无结果而留空时，不要留纯空白——在备注里追加一条极简痕迹 `【已查:Scholar/SS/DBLP/OR/GitHub/校网/LinkedIn 均无X】`，注明"已查过哪些信源、确实无该字段"。目的：区分"查过确实没有" vs "还没查"，下一轮直接跳过，避免对同一硬骨头重复劳动。

## 邮箱专项深挖（EMAIL DEEP-DIVE，必须逐条执行，不许只查一两处）
邮箱是最容易漏的字段。**很多人会把邮箱明写在 GitHub 简介或个人主页上**，必须按以下优先级逐一排查，在页面里显式搜索 `Email` / `E-mail` / `mailto:` / `@` / `[at]` / `(at)` / ` AT ` / `[dot]` 字样：
1. **论文 PDF 本身**：首页脚注、通讯作者标注、Contributors 章节常直接给邮箱（最高置信，标"确认"）。
2. **个人主页 / 学术主页**：联系栏、About/Contact、侧边栏；查看页面源码中的 `mailto:`；反混淆写法 `name [at] gmail [dot] com`、`name AT university DOT edu`、图片邮箱读 alt 文本；主页挂的 CV/Resume PDF 几乎必有邮箱，必须点开读。
3. **GitHub**（重点）：Profile 公开 email 字段；bio 一句话简介；profile README（`github.com/{user}/{user}`）常写 "📧 Email: xxx"；个人主页仓库（`{user}.github.io`）源码搜 `mailto:`；commit 邮箱（`.patch` 的 `From:` 行，若是 `users.noreply.github.com` 则弃用）；GitHub API 的 email 字段与公开事件的 commit author email。
4. **Google Scholar**：`Verified email at xxx.edu` 只给域名——结合论文脚注学到的机构邮箱格式规律（如 `firstname.lastname@company.com`）可推断完整地址，标"（推测）需人工确认"并注明依据。
5. **OpenReview profile**：Emails 区块（部分公开，形如 `****@company.com` 也能确认域名）。
6. 公司邮箱推断：同团队论文脚注已出现 N 个 `{pattern}@company.com` 样本时，可按规律推断，必须标"高可信（按机构邮箱规律推断）"。

## 检索单元提示词模板骨架
```
# 任务：四列闭环方法论（含邮箱深挖）
TARGET ROWS (row -> name): ...
COLUMN MAPPING:
- 中文名,学历,Scholar,OpenReview,主页,LinkedIn,GitHub
- 邮箱,微信,电话,具体方向
- 不动 英文名 / X 列
方法论: [四列闭环 1-9 全文]
EMAIL DEEP-DIVE: [邮箱专项 1-6 全文]
置信分级：每格 `内容｜标签｜依据`。
OUTPUT: 独立文件；每行为该作者各字段；回读校验非空；报告 <250 词。
```

## 铁律（踩坑总结）
1. **大规模并行检索时，各单元只写本地结果，由主流程合并后一次性入库**，避免并发写同一表导致覆盖。
2. **逐行写精确单元格，绝不连续块**（连续块覆盖邻批行 → 数据丢失）。
3. **每批独立子目录 / 独立输出文件**，CSV 不带表头。
4. 完成后**必扫全空行**并修复。
5. **邮箱必须走 EMAIL DEEP-DIVE 全清单**，只查 Scholar verified 域名就放弃 = 不合格。
6. **负向证据留痕**：穷尽仍无的列，写清"已查哪些信源均无"，避免重复劳动。
7. 频率限制（429）注意重试与降速；切换模型可绕过。

## 常见失败
- 连续块覆盖：检索单元图省事写整块。强制逐行 + 验证。
- CSV 带表头导致数据下移：强制只写数据。
- 列数不足：写出少数列会静默丢失末列。合并脚本对不足补空、超量折叠末列兜底，但应在提示词里强制每行列数一致（空字段也须占位），并重派修复。
- 文件互相覆盖：多批共用文件名。强制独立子目录。
- 邮箱漏检：只看 Scholar。提示词必须完整贴 EMAIL DEEP-DIVE 清单。
