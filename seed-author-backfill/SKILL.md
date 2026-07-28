---
name: seed-author-backfill
summary: 端到端流程：用户给出某个/某些公司或团队的技术报告论文（PDF/arXiv链接/团队名），迅速提取作者名单，批量检索每位作者的中文名、学历、Google Scholar、OpenReview、主页、LinkedIn、GitHub、邮箱（深挖）、研究方向等信息，并回填到飞书多维表格。四列闭环交叉验证 + 邮箱专项深挖。适用于任意公司/团队（ByteDance Seed、OpenAI、DeepSeek、高校实验室等）。触发词：技术报告作者、论文作者检索、作者信息回填、人员表补全。
agent_created: true
---

# 技术报告论文作者信息检索回填（端到端，飞书多维表格）

## 何时使用
- 用户发来某公司/团队的**技术报告或论文**（PDF、arXiv 链接、或只给团队名+报告名），要求找出全部作者并检索每人的学术信息、联系方式，整理成表。
- 用户已有一张作者名单表（飞书多维表格/电子表格），要求"重跑/回填/补全/核对"人员信息。
- 泛化：把"团队名/公司名/报告名"替换为任意目标即可复用。

## 端到端流程总览
```
Phase 0 提取作者名单 → Phase 0.5 自动建表+写入人名（用户零手工）
→ Phase 1 拉表诊断 → Phase 2 生成批次提示词
→ Phase 3 派发子代理批量检索 → Phase 4 验证+数据丢失修复 → Phase 5 覆盖率报告
```

## Phase 0：从技术报告提取作者名单
1. 输入是 arXiv 链接：用 arXiv API（`http://export.arxiv.org/api/query?id_list=XXXX`）或 abs 页面拿完整作者列表；技术报告的"Contributors/Core Contributors"章节往往比 arXiv 元数据更全，**必须下载 PDF 读贡献者章节**（常在文首或附录）。
2. 输入是 PDF：直接读 PDF，提取 Authors + Contributors + Acknowledgements 中的人名。
3. 输入只有团队名：WebSearch 找该团队全部技术报告（官网 publications 页、arXiv listing），逐篇提取并去重合并。
4. **同时记录每篇论文首页脚注/通讯作者邮箱**（`{name}@company.com` 格式规律是后续邮箱推断的 ground truth）。
5. 去重后输出名单（英文名 + 出处论文）。

## Phase 0.5：自动建表 + 自动写入人名（用户无需手动复制人名）
用户只需给论文，**不需要自己往表里贴名字**：
1. 若用户没有现成表：用 lark-cli 新建电子表格（或让用户给一个空表 URL），第 1 行写表头：
   `英文名, 中文名, 学历, Google Scholar, OpenReview, 主页, LinkedIn, GitHub, X, 邮箱, 微信, 电话, 具体方向, 方向摘要, 简历`。
2. 将 Phase 0 提取的去重名单**批量写入 A 列**（英文名，从第 2 行起）。A 列此后视为 ground truth 不再改动。
   ```
   lark-cli sheets +csv-put --url "<URL>" --sheet-id "<SID>" --start-cell "A2" --file names.csv
   ```
   （初次写入连续空表时允许连续块；一旦表内已有数据则回到逐行铁律。）
3. **中文名**不在此阶段猜测——由 Phase 3 子代理检索确认后填入 B 列（主页/Scholar 中文名、GitHub bio 等来源，带置信标签）。若论文 PDF 本身给了中文名对照（少数中文报告有），可直接写入 B 列并标"确认"。
4. 写完后回读 A 列校验行数 = 名单人数，报告用户"共 N 人已写入，范围 A2:A{N+1}"。

## Phase 1：拉表 + 现状诊断
飞书表用 `lark-cli sheets`，env 前缀必须带 `LARKSUITE_CLI_NO_UPDATE_NOTIFIER=1 LARKSUITE_CLI_NO_SKILLS_NOTIFIER=1`：
```
lark-cli sheets +csv-get --url "<URL>" --sheet-id "<SID>" --range "B{start}:M{end}" --format json > scan.json
```
用 Python 解析 `data.annotated_csv`（每行格式 `[row=N] csv...`），按"确认/推测/空"三态统计每列覆盖率，找出**仍无确认链接**的行作为优先目标。
参考列结构（可按用户表调整）：A=英文名(不动), B=中文名, C=学历, D=gscholar, E=OpenReview, F=homepage, G=linkedin, H=github, I=X(不动), J=邮箱, K=微信, L=电话, M=具体方向；不动列绝不覆盖。

## Phase 2：生成批次提示词
- 每批 10 行，生成 `rprompt_NN.txt`，自包含（URL、sheet_id、行号→英文名、列映射、方法论全文）。
- 每批独立子目录 `brNN`，CSV 无表头。

## Phase 3：派发子代理（关键避坑）
**必须前台 + 同步 bash**，否则间歇性报 `Tool TaskStop not found`：
> CRITICAL: Run ALL bash commands SYNCHRONOUSLY (foreground). NEVER use run_in_background.

**推荐（大规模更稳定，已验证）：子代理只写本地 CSV，不直接写飞书。** 每批输出
`results/batch_NN.csv`（13 列：行号,英文名,中文名,学历,Scholar,OpenReview,主页,LinkedIn,
GitHub,邮箱,微信,电话,方向，csv.writer 无表头）；主代理合并所有批次为两块——
`master_BH.csv`（B-H，7 列）与 `master_JM.csv`（J-M，4 列，跳过 I）——各一次 `+csv-put`
（`--start-cell B2` / `J2`）上传。这样彻底规避多代理并发写同一表导致的覆盖/修订冲突，
且只需两次写操作。逐行写精确单元格（B{R}/J{R}）仅保留用于单/少行修补。

若坚持逐行上传（小批量或修补），每批提示词里写死：
> Target rows are NOT contiguous. Upload each row to its exact cell B{R}/J{R}. Do NOT write a single contiguous block.

## Phase 4：验证 + 数据丢失修复（必须做）
重跑后重拉全表，扫描 **B-H 且 J-M 全空** 的行 = 被连续块误清/上传失败的行。用 A 列英文名确认身份，建修复批次逐行回填，反复扫描直到 0 个全空行。

## Phase 5：最终覆盖率报告
确认/推测/空 三态 × 关键列（Scholar/OpenReview/Homepage/GitHub/邮箱），以及"有≥1确认链接的行数占比"。

## 核心方法论：四列闭环交叉验证
1. 以该团队技术报告/论文为消歧 ground truth（提示词中列出已知论文名）。
2. 优先找学术主页（照片+中文名+单位+所有出站链接）。
3. 找 GitHub，读 bio/website/README 反挖 Scholar/主页；查是否属团队 org 或贡献已知 repo。
4. Scholar 主页字段 + 论文列表反填主页并交叉确认。
5. OpenReview 主页字段反填。
6. 任一入口顺藤摸瓜填其余三列，确认四列指向同一人。
7. 穷尽再留空：Scholar→Semantic Scholar→DBLP→OpenReview→llm-people→aminer→arXiv→GitHub→校网→LinkedIn。无 Scholar 用 DBLP/ORCID 替代并注明。
8. LinkedIn 仅头像/职位明确匹配才填，否则留空（不标推测）。
9. 置信分级：每格 `内容｜标签｜依据`，标签 = 确认/高可信/（推测）需人工确认。
10. **负向证据留痕（必做）**：某列穷尽后仍无结果而留空时，不要留纯空白——在该行的方向列末尾或备注里追加一条极简痕迹 `【已查:Scholar/SS/DBLP/OR/GitHub/校网/LinkedIn 均无X】`，注明"已查过哪些信源、确实无该字段"。目的：区分"查过确实没有" vs "还没查"，下一轮重跑直接跳过，避免对同一硬骨头重复劳动。

## 邮箱专项深挖（EMAIL DEEP-DIVE，必须逐条执行，不许只查一两处）
邮箱是最容易漏的字段。**很多人会把邮箱明写在 GitHub 简介或个人主页上**，必须按以下优先级逐一排查，在页面里显式搜索 `Email` / `E-mail` / `mailto:` / `@` / `[at]` / `(at)` / ` AT ` / `[dot]` 字样：
1. **论文 PDF 本身**：首页脚注、通讯作者标注、Contributors 章节常直接给邮箱（最高置信，标"确认"）。
2. **个人主页/学术主页**：
   - 首页顶部联系栏、About/Contact 区块、侧边栏；
   - 查看页面源码中的 `mailto:` 链接（很多主页文字不显示但 mailto 里有）；
   - 反混淆写法：`name [at] gmail [dot] com`、`name AT university DOT edu`、用图片显示的邮箱（读 alt 文本描述）；
   - 主页挂的 **CV/Resume PDF** 里几乎必有邮箱，必须点开读。
3. **GitHub**（重点）：
   - Profile 侧边栏公开 email 字段；
   - **bio 一句话简介**里常写邮箱；
   - **profile README**（`github.com/{user}/{user}` 仓库的 README）常有 "📧 Email: xxx"；
   - 个人主页仓库（`{user}.github.io`）源码里搜 `mailto:`；
   - commit 邮箱：`https://github.com/{user}/{repo}/commit/{sha}.patch` 的 `From:` 行（若是 `users.noreply.github.com` 则弃用）；
   - GitHub API：`https://api.github.com/users/{user}` 的 email 字段与 `https://api.github.com/users/{user}/events/public` 中 PushEvent 的 commit author email。
4. **Google Scholar**：`Verified email at xxx.edu` 只给域名——结合从论文脚注学到的该机构邮箱格式规律（如 `firstname.lastname@company.com`）可推断完整地址，标"（推测）需人工确认"并注明推断依据。
5. **OpenReview profile**：Emails 区块（部分公开，形如 `****@company.com` 也能确认域名）。
6. 公司邮箱推断：同团队论文脚注已出现 N 个 `{pattern}@company.com` 样本时，可按规律推断，必须标"高可信（按机构邮箱规律推断）"。

## 子代理提示词模板骨架
```
# 子代理任务：闭环方法论 v3（含邮箱深挖）
SPREADSHEET: URL + sheet_id
YOUR BATCH (row -> name): ...
COLUMN MAPPING:
- Block1 start B{R}: 中文名,学历,gscholar,OpenReview,homepage,linkedin,github (B-H)
- Block2 start J{R}: 邮箱,微信,电话,具体方向 (J-M)
- 不动 A/I/N/O
方法论: [四列闭环 1-9 全文]
EMAIL DEEP-DIVE: [邮箱专项 1-6 全文，强调在 homepage/GitHub bio/profile README/CV PDF 中显式搜索 Email/mailto/[at] 字样]
置信分级：每格 `内容｜标签｜依据`。
OUTPUT: 独立子目录；csv.writer 无表头；逐行上传 B{R}/J{R}；回读校验非空；报告 <250 词。
CRITICAL: 所有 bash 同步前台执行，绝不后台；逐行上传精确单元格，绝不连续块。
```

## 铁律（踩坑总结）
1. **子代理前台 + 同步 bash**，禁用后台（否则 TaskStop 故障）。
2. **逐行上传精确单元格，绝不连续块**（连续块覆盖邻批行 → 数据丢失，曾造成 11 行被误清）。大规模回填推荐"子代理写本地 CSV + 主代理合并后分两块 csv-put"替代逐行写，同样安全且避免并发冲突。
3. **每批独立子目录 + CSV 无表头**。
4. 完成后**必扫全空行**并修复。
5. **邮箱必须走 EMAIL DEEP-DIVE 全清单**，只查 Scholar verified 域名就放弃 = 不合格。
6. **负向证据留痕**：穷尽仍无的列，写清"已查哪些信源均无"，避免重复劳动。
7. 429 频率限制每日 UTC+8 上午重置；切换模型可绕过。

## 常见失败
- `Tool TaskStop not found`：子代理内部后台了 bash。提示词加"同步 bash"重派。
- 连续块覆盖：子代理图省事写整块。强制逐行 + 验证。
- CSV 带表头导致数据下移：强制 `writerows` 只写数据。
- 子代理 CSV 列数不足 13：部分子代理写出 11–12 列会静默丢失末列（M 方向）。合并脚本已对 <13 补空、>13 折叠末列兜底，但应在子代理提示词里强制断言每行恰好 13 列（空字段也须以逗号占位），并重派修复。
- 文件互相覆盖：多批共用文件名。强制独立子目录。
- 邮箱漏检：子代理只看 Scholar。提示词必须完整贴 EMAIL DEEP-DIVE 清单。
