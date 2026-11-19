---
name: duo-review
description: 接收工单号并对已派工分支做四步验收，accept 则合并置 done；reject 则写回分级反馈并携反馈自动重派，Rejects 满 2 升级人工
metadata:
  version: "1.0.0"
---

# 验收 Codex 交付（duo-review）

按工单号对已派工交付做结构化验收：先核状态与现场，再按四步取证、三源交叉印证，全部通过才走 accept 合并归档；判定 reject 时写回分级反馈并自动重派。**报告自述、退出码和 last-message 不是通过依据。**

**机器前提**：WSL 发行版名与 node 加载方式（`. ~/.nvm/nvm.sh`）以宿主机器约定为准；文中 `Ubuntu-26.04` 为本机示例。

## 路径约定

- 编排方从当前项目根推导 `<主仓库WSL路径>`：Windows 项目根 `E:\x\y` 对应 WSL `/mnt/e/x/y`；示例 `E:\projects\my-app` 对应 `/mnt/e/projects/my-app`。
- `<仓库短名>` 是主仓库根目录名；实施与清理使用 `~/duo-worktrees/<仓库短名>/<feature>-<NN>`。历史无仓库维度的 worktree 不迁移、不改名。

## 输入

- `<NN>`：两位工单号；可搭配 `<feature>` 指明工单目录，未给时从当前 feature 目录识别。

## 铁律

1. 只验收 `Status: claimed` 且登记了实施分支的工单；状态缺失、已是 `done`、分支缺失或歧义时终止。
2. 四步验收每步都要留下可复查证据；无证据视同未通过。
3. `git diff`、完成报告、工单验收标准三源必须互相印证；矛盾即必须 reject。
4. accept 只允许 `git merge --ff-only`；不能快进合并时停止报告，不擅自产生 merge commit。
5. 不修改实施产物来“补证据”，不伪造审查、测试、UI 或裁决结论。
6. reject 后按累计 Rejects 计数执行：低于 2 次自动携反馈重派；达到 2 次置 `ready-for-human` 并停止重派，不得发起第三次自动派工。

## 流程（顺序执行，失败即停）

### 1. 定位与前置校验

1. 定位 `.scratch/<feature>/issues/NN-*.md`；命中数不唯一或文件不存在时终止。
2. 校验工单头为 `Status: claimed`，并从同一行提取 `分支 duo/<feature>/<NN>`。若状态不是 claimed，先终止；`Status: done` 说明本票已验收，不重复裁决。
3. 校验本地分支存在、唯一且与登记名一致；记录分支头提交号。
4. 计算并记录基线提交：`git merge-base HEAD duo/<feature>/<NN>`。后续分支审查范围固定为 `<基线提交>..duo/<feature>/<NN>`。
5. 用 `git diff --stat <基线提交>..duo/<feature>/<NN>` 确认 diff 非空；空 diff 视为未完成，停止验收。
6. 确认实施 worktree 内 `.scratch/<feature>/reports/NN-report.md` 存在，且含改动清单、自检证据、偏离说明、已知遗留问题四要素。实施 worktree 已清理或无法访问时，改读合并后主工作区同路径；两处都不存在即停止。

### 2. 四步验收

#### ① 双轴审查

1. 调用本仓库 `/duo-code-review` skill 审查 `<基线提交>..duo/<feature>/<NN>`，明确要求覆盖两轴：Standards（仓库规范、安全红线、明显缺陷）与 Spec（是否符合工单目标、范围、禁区与 DoD）。
2. 要求报告给出结论、阻塞/非阻塞分级、文件与行级证据；保存结论要点，accept 时归档到工单 Comments。
3. `duo-code-review` 不可用时，按同等双轴人工审查，并在裁决摘要明确写“duo-code-review skill 不可用，已人工双轴”。

#### ② 勾验工单验收标准

1. 提取工单 `## 验收` 或等价节中的每条 checkbox，建立逐条验收表。
2. 对每条标注 PASS/FAIL，并附证据：diff 位置、审查报告节、命令输出路径或摘要、快照路径。证据必须指向真实交付物，不能只引用完成报告自述。
3. 任一条无法证实即 FAIL；不整段合并判定。

#### ③ 实跑测试与自检

1. 从工单、简报和完成报告中核对声明的测试/自检命令；三者不一致时，以工单和简报为裁决依据，先澄清差异。
2. 在实施 worktree 实跑；实施 worktree 不可用时，先恢复可验证环境：重建实施 worktree，或在主工作区用临时路径检出实施分支后实跑。仍无法建立可运行环境时停在现场，等待人工裁决。记录每条命令、退出码和关键输出。
3. 工单声明无测试命令时，注明豁免理由，并核对是否有可用的最小静态检查或文本核对。

#### ④ 视觉验证

1. 改动涉及用户可见 UI 时，按项目视觉验证约定执行；本项目使用 MCP 快照并把证据存 `temp/`，记录快照路径与关键界面结论。
2. 非 UI 改动写一句豁免说明，例如“纯 skill 文案改动，无 UI 路径”。

### 3. 三源核对

1. 交叉核对 git diff、完成报告与工单验收标准：改动清单是否覆盖且只覆盖简报范围，报告声明是否有 diff、审查、测试或 UI 证据支撑。
2. 核对禁区与偏离说明：越出简报范围必须有简报允许的偏离说明；否则按越界处理。
3. 完成报告与 diff 矛盾、报告缺失四要素（改动清单/自检证据/偏离说明/已知遗留问题）、或证据指向不存在的产物时，必须 reject。
4. 实施分支跟踪了 `.duo-*` 运行产物时，必须 reject。

### 4. 裁决

出现任一情形即必须 reject：

1. 任一工单验收标准无证据。
2. 双轴审查发现阻塞性问题，包括安全问题、红线违反或明显缺陷。
3. 改动越出简报范围且无偏离说明。
4. 完成报告与 git diff 矛盾。
5. 实施分支跟踪了 `.duo-*` 运行产物。

四步验收与三源核对全部通过才裁决 accept；出现上述任一 reject 判据时，立即执行 Reject 路径，不停在口头打回，也不让用户充当两引擎间的反馈中转。

## Reject 路径（裁决 reject 后顺序执行）

### 4b. 写回反馈、计数与升级

1. 先解析累计 `Rejects: M`：优先读工单 `Status:`，再统计 `## Comments` 中匹配 `— REJECT（Rejects:` 的小节（与 duo-dispatch §1.4 同契约）；两者矛盾时不写回，停止并报告冲突。当前次为 `N = M + 1`，写回后累计数为 `Rejects: N`。
2. 在工单 `## Comments` 末尾追加小节，标题必须是 `### YYYY-MM-DD HH:mm 验收 round N — REJECT（Rejects: N）`。正文按四类分级逐项列出真实判定证据、问题、期望行为和修复要求；某类无内容时写“无”：
   - `【硬违规】`：违反红线、验收标准、安全规范或存在阻塞性缺陷。
   - `【未申报扩展】`：越出简报范围且偏离说明未申报。
   - `【顺带修】`：非阻塞但本次必须修复的缺陷或一致性改进。
   - `【不修留档】`：不要求本次修复的风险、重复或后续跟进事项，写明保留理由。
3. 小节结尾必须写一句：`修复后报告追加【补充：打回与修复（round N+1）】小节；git add -A（.claude 用 -f）追加提交，禁止 push。`
4. 把工单 `Status:` 置为分级状态：`N < 2` 时置 `ready-for-agent（YYYY-MM-DD HH:mm 验收打回，Rejects: N；反馈见 ## Comments）`；`N = 2` 时置 `ready-for-human（Rejects: 2，升级人工裁决；两次打回原因见 Comments）`。除该行和第 2 目新增小节外，不重写工单历史。
5. 反馈与 Status 写回后，先在主仓库只登记工单文件并提交，建议信息：`chore(<feature>): 工单NN 验收打回 round N（Rejects: N）`；不 push。提交失败时立即停止，不重派、不输出升级结论，保留已写回的反馈、Status、Rejects 计数、分支、worktree 与现场，报告失败点和日志路径；不伪造结果。
6. 提交成功且 `Rejects: N < 2` 时保留实施分支与 `~/duo-worktrees/<仓库短名>/<feature>-<NN>`，确认其仍指向登记分支且无未预期改动，然后直接调用 `/duo-dispatch <NN>` 重派同一工单；重派必须复用该 worktree 与分支，不新建、不清理。重派启动失败时保留反馈提交、Status、Rejects 计数、分支、worktree 与日志，报告失败点，不伪造重派。
7. 提交成功且 `Rejects: N` 达到 2 时不再重派：保留实施分支、worktree、简报、报告与全部日志，并向用户输出升级说明——两次打回原因各一句摘要，以及分支名、worktree 路径、日志路径；后续处置等待用户裁决。

## Accept 路径（全部通过后顺序执行）

### 5. 快进合并

1. 确认当前工作分支正确、工作区无会阻碍合并的未提交改动；必要时先让用户处理，不吞并无关文件。
2. 在主仓库当前工作分支执行 `git merge --ff-only duo/<feature>/<NN>`。
3. 合并失败、产生冲突或 Git 拒绝快进时立即停止；保留分支、worktree 与现场，报告两个分支的当前位置，等待人工裁决。
4. 合并成功后记录新的 HEAD 提交号。

### 6. 工单置 done 并归档

1. 把工单 `Status:` 置为一行 done 摘要：`done（YYYY-MM-DD HH:mm 验收 accept，round N，提交 <合并后HEAD短提交号>；审查/标准/测试/UI 四项通过）`。`N` 从 1 开始；存在 reject 历史时取下一轮次。除这一行外不改工单原有内容。
2. 在工单 `## Comments` 末尾追加：

```markdown
### YYYY-MM-DD 验收 round N — ACCEPT

- 双轴审查：<结论要点、阻塞项数量、非阻塞项处置>
- 验收标准：逐条 PASS；关键证据为 <diff/报告/输出位置摘要>
- 测试与自检：<真实命令与结果；无测试命令时写豁免理由>
- 视觉验证：<快照路径与结论；非 UI 改动写豁免说明>
```

3. Comments 只追加验收小节，不重写既有历史。

### 7. 清理 worktree

在 WSL 侧执行清理；Windows 侧不可见 WSL 路径，不得用 Windows shell 删除 worktree：

```bash
wsl.exe -d Ubuntu-26.04 -- bash -c 'git -C ~/duo-worktrees/<仓库短名>/<feature>-<NN> status --short && git -C <主仓库WSL路径> worktree remove --force ~/duo-worktrees/<仓库短名>/<feature>-<NN>'
```

`--force` 用于丢弃 `.duo-brief.md`、`.duo-last-message*` 等不提交的运行产物。清理仅限 `.duo-*` 运行产物；调用 `/duo-review` 即视为对该清理的概括授权；出现未预期改动仍应停止并请示。清理前若出现未预期的源码改动或未提交交付物，停止清理并报告现场。

### 8. 提交工单状态变更

1. 在主仓库只登记工单文件：`git add .scratch/<feature>/issues/NN-*.md`。
2. 提交状态与裁决归档，建议信息：`chore(<feature>): 工单NN 置 done——验收通过合并(<合并后HEAD短提交号>)`。
3. 不 push；运行产物与无关文件不登记。

### 9. HTML 验收报告与控制台摘要（accept 与 reject 终态均产出）

1. **HTML 验收报告**写 `.scratch/<feature>/reports/NN-review.html`（单文件、深浅色自适应、无外部依赖；reject 报告名 `NN-review-reject-r<N>.html` 区分轮次），结构五区：
   - **结论头**：工单号+标题、ACCEPT（绿）/REJECT（红）徽章、round 数、裁决时间、分支与合并提交（或打回反馈在工单的锚点）；
   - **任务卡**：工单目标一句摘要、实施仓库与提交（多仓分列）、派工/打回/重派时间线；
   - **四步验收状态卡**：双轴审查/勾验标准/实跑测试/视觉验证，各带 PASS·FAIL 徽章与一句结论；
   - **测试明细表**：每套演练一行（名称、命令或来源、PASS/FAIL 计数、关键输出摘录）；`php -l`/`bash -n` 等静态检查与豁免项（注明豁免理由）也入表；
   - **截图画廊**（视觉步骤有产物时）：入选报告的截图从 `temp/` 归档到 `.scratch/<feature>/reports/images/`，文件名 `<NN>-<屏名>.png`（工单号前缀防混），HTML 以同目录相对路径 `./images/<NN>-*.png` 引用并标注屏名与断言点；`temp/` 保持过程工作区语义（诊断图/中间产物不入库）。
   报告数据必须来自本轮真实取证（工单 Comments、演练输出、截图文件），不虚构数字；reject 报告的测试明细如实反映 FAIL 项与分级反馈。
2. **控制台摘要卡**：裁决终了在会话输出一张 markdown 摘要（用户在 TUI 一眼看清），固定字段：工单号+标题、结论徽章、round/Rejects 计数、四步验收四格（✓/✗）、测试计数行（如 `S9 213/213 · 增量 99/99 · 基线 139/139 · 视觉 18/18`）、合并提交（或反馈落点）、HTML 报告路径、下一步（accept→下一张票；reject→重派中/升级人工）。
3. **报告与截图纳入 git**：`NN-review*.html` 与 `reports/images/<NN>-*.png` 归档截图随工单状态变更一并 `git add` 提交（验收证据持久化，用户明确要求管理）；控制台摘要卡即席输出不留文件。提交信息沿用第 8 步格式，可在正文注一句"附验收报告 HTML 与 N 张截图"。

## 登记与幂等

1. 重复收到同一 `/duo-review NN` 请求时，先重读工单 Status。
2. `Status: done` 时终止并指出已有验收结论、合并提交或工单归档位置，不重复合并、重写 Comments 或再次清理。
3. `Status:` 不是 claimed 且不是 done 时，报告当前状态与原因后终止。

## 故障与边界

- **merge 冲突或不能 ff**：停在快进合并步，保留实施分支、worktree 与完整四步证据，报告冲突文件和两侧提交位置；不改代码、不产生 merge commit、不置 done。
- **测试跑不起来**：停在实跑步，保留原始报错、命令、退出码与环境状态；不把环境问题解释为测试通过，不进入合并。
- **合并后发现测试失败**：不置 done，先在工单登记事故，再报告用户裁决；回退、reset 等敏感操作必须由用户明确确认，本 skill 不得自行 reset。
- **视觉验证环境不可用**：停在视觉验证步，保留尝试记录与现有现场，请用户裁决或提供可用环境；不用截图缺席替代结论。
- **证据或现场丢失**：以能复核的证据为准；不能复核时按对应步骤失败处理，报告缺失项，不回填猜测。
- **reject 计数或反馈冲突**：停止写回与重派，保留现有状态和现场，报告状态行、Comments 小节与计数的冲突。
- **重派启动失败**：保留 `ready-for-agent`、Rejects 计数、分支、worktree、反馈提交与日志；报告启动命令、失败点和日志路径，不伪造重派，不删除现场。

### 10. 经验回灌与工具自优化（每次验收终态执行）

验收（accept 或 reject）归档后，评估本轮是否有**可回灌经验**，有则完善 duo 套件并自动推送 GitHub——这是 duo 工具的自优化机制，用户明确要求持续执行。

**回灌判定**（满足其一才入库，单次业务 bug 不入，避免噪音）：

1. **环境类陷阱**：会在其他机器/项目复现的坑（数据库特性、框架版本行为、容器缓存、shell 转义等）；
2. **流程缺陷**：duo 套件自身规则的漏洞（如"验收方资产适配未随归档提交"类事故）；
3. **实施方反复犯的错**：写进简报规范可预防的 codex 通病（如断言函数姿势、报告入库）。

**执行动作**（判定为"有"时）：

1. 编辑 duo-skills 仓库（`E:\uniappProjects\duo-skills\`，跨机器时以 `~/duo-skills` 或用户告知的路径为准）对应 skill 的"常见陷阱与对策"节——验收侧问题进 `duo-review/SKILL.md`，派工/简报侧进 `duo-dispatch/SKILL.md`；每条一句话三元组：**现象 / 根因 / 对策**；
2. 同步回本机用户级：`cp` 改动后的 SKILL.md 到 `~/.claude/skills/<同名目录>/`；
3. 自动提交推送（用户已授权 duo-skills 仓库的自动化管理）：

```bash
cd /mnt/e/uniappProjects/duo-skills && git add -A && \
git -c user.name=DongYao -c user.email=dongyao@snxny.com commit -m "docs(lessons): <一句话经验>" && git push origin main
```

push 失败（网络/权限）不阻塞验收结论，报告失败并保留改动待下次一并推送。判定为"无"则跳过本步，不留噪音记录。

## 常见陷阱与对策（活文档——每轮回灌在此追加）

- **TP5.0 模板无 `|raw` 过滤器** / 视图写 `{$var|raw}` 即编译 fatal / admin 视图一律裸 `{$var}` 输出（php -l 查不出模板函数，视觉步骤才暴露）。
- **MySQL enum 裸数字 UPDATE 陷阱** / `SET enum_col=1` 的 1 被解释为枚举索引（enum('0','1')：'' =0,'0'=1,'1'=2）而非字符串值 / 演练脚本造态一律字符串形式 `SET col='1'`。
- **迁移显式 COLLATE 炸跨表 JOIN** / 新表显式 `COLLATE utf8mb4_unicode_ci` 与库默认 `0900_ai_ci` 不一致，JOIN 报 ERROR 1267 / 建表不指定 COLLATE，继承库默认。
- **opcache 与 TP5 模板缓存不失效** / 主工作区切分支后旧行为诡异混跑（新旧行为交替出现是典型信号）/ `docker restart php-app` + 清 `runtime/temp` 编译产物，验收代跑前必做。
- **演练脚本断言四姿势** / 实施方脚本通病 / ① pyget 路径含 `['data']` 层；② contains 四参语义（id/描述/子串/响应）；③ 造态 INSERT 显式全部必填列（无默认值列被 `2>/dev/null` 吞成假失败）；④ admin 模块不走无 session 的 curl 直打。
- **归档铁律：验收方资产适配必须随归档提交回主线** / 打回侧线里的演练载荷适配若只在侧线提交，归档 checkout 实施分支头时被丢弃 → 后续全量假性回退 / 归档提交必须包含验收方对演练脚本的一切修改。
- **wsl.exe 内联命令引号地狱** / 嵌套引号/管道/反引号被外层 shell 拆坏（cut/sed 报 no such file、zsh no matches found）/ 含复杂语法的 WSL 命令一律写脚本文件到 /mnt/e 再 `wsl.exe bash <脚本>`。
- **验收方代修尺度** / 小缺陷（字符级修复）走 reject 流程会让 Rejects 满 2 升级人工，代价不成比例 / 字符级小改代修入实施分支 + 工单 Comments 如实记录"验收代修"；结构性缺陷才 reject。
- **并行组合入协议**：同组并行票按依赖序逐票 ff 合入；后票 merge-base 被前票合入推走、ff-only 失败时，执行**受控 rebase**——在实施 worktree 内 `git rebase <主分支>` 逐冲突解决（只取双方意图的并集，不改业务语义），复跑该票关键断言全绿后重走第 5 步合入；冲突涉及业务语义取舍或 rebase 超过 5 个文件时停下请示用户。rebase 仅限编排方为合入执行，不违反对实施方的禁令。
- **TP5 try 块内 error() 被 catch(\Exception) 吞** / `$this->error()` 抛 HttpResponseException（extends Exception），写在 `try{...}catch(\Exception)` 内的 error 会被吞成兜底文案（getMessage 恒空串）/ gate 校验类的 error 一律放 try **外**，或 catch 链首加 `catch (\think\exception\HttpResponseException $e) { throw $e; }` 放行（pitfalls #47 惯例）——审查时对"try 内含 error"的模式逐处过目（工单10 实例：chainRecord 门禁提示全变"上链失败："）。
- **统计卡与列表数据一致性断言（用户方法论，必设防线）** / 管理端"统计数字（服务端渲染）与列表行数（ajax）"两套数据源在列名/过滤条件错时会静默分叉（实例：join 列名写错→列表接口 500→UI 空态，统计卡正常，页面呈现"待审 2 但列表空"）/ 验收必设断言：**列表接口 code=1 且行数 == 统计口径 SQL 计数**；视觉步骤看到"统计与列表对不上"直接判 FAIL，不得以"页面结构达成"放过。接口数据与 UI 渲染数据对不上=重要测试标志。
- **演练须覆盖 admin 表格真实接口** / SQL 直查/超管模拟验不出接口层 500（列名错误在 join 里）/ admin 表格类功能至少一条走"登录态 curl docs/records 接口断言 code+行数"（AJAX 登录通路稳定，cookie jar 复用）。
- **`git checkout -- .` 全量还原误伤工作区私改** / 目标本只还原个别文件，却把 stash pop 回来的本地环境配置一并抹掉（五件套险丢）/ 还原**逐一指定路径** `git checkout -- <path>`；stash pop 后记录其 hash（输出里有），误删可 `git stash apply <hash>` 从未 gc 的对象救回——工单10 实战救回成功。
- **测试管线跑器优先（CI 对标基石）** / 逐套手跑费时且易漏 / 实跑步骤优先 `bash tests/run-all.sh`（full 五套件约 35s，产出 JSON 报告 `tests/reports/run-*.json`）；变更面明确时加 `--affected <paths>`（smoke）。**全量成本已低于 1 分钟的门禁必须全量跑**，"全量很贵"的直觉先实测再信。JSON 报告是 duo-portfolio 趋势分析的数据源，验收归档时在测试明细注明报告路径。
- **工单置态后必须 grep 验证** / python str.replace 落空时不报错，"打印成功"具有欺骗性（工单10 实例：claimed/done 两次置态均落空，Status 停留在旧态，duo-status 首跑才抓出）/ 每次 Status 行写回后立即 `grep "^Status:" <工单>` 复述实值；批量置态脚本同理，落空即停。
