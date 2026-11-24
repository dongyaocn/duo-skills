---
name: duo-review
description: 接收工单号并对已派工分支做四步验收，accept 则合并置 done；reject 则写回分级反馈并携反馈自动重派，Rejects 满 2 升级人工
metadata:
  version: "1.0.10"
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

- **Spec 轴审查要区分「实施忠实」与「spec 正确」**（wechat-draftbox-sync 工单02 实例）/ 双轴 Spec 审查发现缺陷时先判责任归属：实施忠实复现了 spec 的口径缺陷（决策函数未按 source 区分→本地 txt 行被同步覆盖丢正文），错在编排方 / 反馈小节如实写明「spec 缺陷传导至实施，责任在编排方」，同时把修复口径写死进反馈（如「toUpdate 落库前只执行 source='wechat_draft' 行」），避免实施方自行猜方向；spec 侧的契约语义修订（01 决策模块）在归档备注中留账。
- **reject 终态的 HTML 报告勿漏**（wechat-draftbox-sync 工单02 实例）/ §9 说「accept 与 reject 终态均产出」，但携反馈重派节奏下 reject 报告易被跳过直接进 dispatch / reject 当轮即产出 `NN-review-reject-r<N>.html`（简版亦可），数据从工单 Comments 分级小节取，与 accept 版互补成完整时间线。
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
- **视觉验证是全量步骤不是取样**（Vue/前端向，工单03 exam_web 实例）/ 只走列表页、预览页"点不进去"就绕过 → round 1 漏检 preview 整页空白，round 2 才暴露多打一轮 / 每条新路由必须导航 + 每个入口按钮真实点击 + URL 断言；reject 判据已齐也不得提前收口；未走到的页面清单显式列在反馈里。
- **「点击后 URL 不动 / 正文空白」本身就是缺陷信号** / 不是走查障碍而是 setup 抛错/路由挂起的直接症状（工单03：preview setup TypeError + 导航毒化）/ 当场追根因，禁止绕过。
- **静态证据不可替代渲染** / lint 全绿 + file:line 审查通过 ≠ 页面能渲染——EP 表头白名单静默丢列这类缺陷只有视觉步骤能抓（工单03：四级表头只剩类级 7 列，lint 零错误）/ 凡 UI 改动必跑视觉步骤，无豁免。
- **新页必须 `defineOptions({ name })` 匹配路由 cache include**（Fantastic-admin KeepAlive）/ 组件名失配缓存失效，控制台「该页面组件未设置组件名」警告即 FAIL 项（工单03 三页全漏）/ 双轴审查加检查点：新路由页 grep 组件名 vs cache 数组。
- **一条路由 setup 抛错毒化全站 router** / 未捕获错误经导航扩散，其他页面点击也空白（工单03 用户实测）/ 视觉走查收尾必须回点 2-3 个既有页面验证导航健康；验收方漏检要自认（round 1 漏检致 round 2 打回不怪实施方），责任归属写进报告。
- **EP 2.13.2 多级表头白名单陷阱** / v-for 嵌套 el-table-column 带 `<template #header>` 编译为 patchFlag=1024（DYNAMIC_SLOTS），被 `ElTableColumn.render()` 的 `patchFlag !== 1024` 过滤**静默丢弃**——列不渲染不注册无警告（render 内 try/catch 吞错）/ 嵌套列一律 `:label` prop，不用 #header slot；官方 demo 全静态列所以不触发。
- **structuredClone 禁对 Pinia 响应式数据** / `ref(数组).value` 是深层 reactive Proxy，`structuredClone(Proxy)` 必抛 `DataCloneError: could not be cloned`（工单03：复制新版本按钮从 round 1 起点击即崩，前两轮没点到该入口）/ store 内深拷贝纯 JSON 形态数据用 `JSON.parse(JSON.stringify(...))`，或 toRaw 后确认无深层代理。
- **活文档教训当轮回灌源仓库，不攒账** / 安装副本 `~/.claude/skills/duo-*` 会被 skills add / 手动同步**整体覆盖**，本地追加不 push 到 duo-skills 源 = 必丢（工单03 实证：5 条追加教训被覆盖，靠工单 Comments 记录才重建）/ 活文档追加后当轮执行 `/duo-sync push`。
- **控制台错误与可处理警告纳入 FAIL 判据**（用户指令 2026-11-20） / 验收方只按 error 类型过滤+检查时机偏差，漏掉用户实测到的 5 个 unhandledrejection（fetch 返回 HTML 兜底页按 JSON 解析炸）/ 视觉走查收尾必须全量记录 console：error + warn + unhandledrejection；能定位根因的警告当轮修复（如 EP el-radio `label`→`value` 弃用警告），环境噪声（框架资源预加载、无后端时接口报错）显式列进报告并区分责任归属——「零 error」口径不够，目标是**零 error + 零可处理 warn**。
- **「全屏页」禁用 position:fixed 遮盖 layout**（工单03 三连指正） / fixed inset:0 z-index 壳把页面盖在整站 layout 之上（侧栏菜单高亮却被遮住），PageHeader/PageMain 永远进不了主区——先调 flex/样式都只是表象 / 全屏演示页也按常规路由页渲染（workbench 先例），「退出全屏」改「返回列表」语义。
- **业务语义以原表口径核对，控件语义与填报口径对齐**（工单03 用户指正） / 原表填 -1（扣分）、页面控件正数（扣分值）→ 同一动作两种符号=歧义；控件应直接表达得分（默认满分、往下调即扣分、[0,满分]），必要时加提示条说明与原表的对应关系 / 审查时对「控件值含义 vs 业务口径」逐项过目。
- **Fantastic-admin Transition 布局：页面组件模板根禁止 HTML 注释**（工单03 血泪） / layout 用 `<Transition mode="out-in"><KeepAlive><component :key="route.fullPath">`，页面模板根若为 `注释 + 元素`，params 互切（同路由销毁重建）时离场动画定位失败→新组件永不挂载→`.main` 整区空白，且**零错误零警告**（HMR 风暴会加剧并掩盖复现条件）/ 页面模板根必须是纯单元素，说明写进 script 注释或文件头注释区；同类「hash 变了路由对但视图空」先查根节点形态，再用主仓既有 params 路由（如 organization/detail）互切做站点级对照排除。
- **Fantastic-admin 布局 `.main` overflow:hidden 捕获页内 sticky**（工单06 round 1 实测） / 页面内 sticky 吸顶（表头/卡头/吸底）全部失效——sticky 相对最近 overflow 非 visible 祖先吸附，layout 的 `div.main` 是 overflow:hidden 且随内容撑高自身永不滚动，而真实滚动发生在 window（实测滚动 1500px 时任务头 rect.top=-1400，02 票交付起就潜伏失效） / 需页内吸顶的页面自建滚动上下文：根容器 `height:100vh（或 100dvh）+ overflow-y:auto`，使组件成为最近滚动祖先，既有 sticky 链全部复活、window 滚动条收敛；视觉验收对「滚动吸顶类」交互必须实测滚动断言 getBoundingClientRect，静态查 position:sticky 样式挂上≠生效。
- **统一形态类需求拆单须枚举全部呈现面 + sticky 捕获嵌套形态**（工单05 返工实例） / 「PC 统一矩阵」只拆了填答工作台、漏了模板渲染实例页——同一业务形态散布多页时拆单前 grep 全部呈现点（组件/页面）逐一入范围；EP `.el-card` 自带 `overflow:hidden` 会把卡头 sticky 捕获在本卡内永不触发（1.0.8 布局捕获的嵌套形态）——卡片流吸顶须解锁卡片 overflow:visible 再 sticky header，滚动检测窗口记得算滚动容器 padding / 拆单漏页与检测盲区双教训同源：验收按「用户可达的每个呈现面」清点。
- **全局组件 slot 契约用前查源码**（工单08 r1 实例） / PageHeader 仅支持 #title/#content/default 三个 slot，实施方自造 #actions 放工具按钮——Vue 对未知具名 slot 静默丢弃零警告，三页「新建/统计」入口全部消失而 lint/eslint/tsc 全绿 / 使用全局组件非默认 slot 前先读组件源码 slot 定义；视觉验收对「工具区按钮」逐个确认真实渲染（querySelectorAll button 断言），静态绿≠运行时在。
- **Vue Router 同层级同形态动态段冲突**（工单08 r1 实例） / `talks/:batchId` 与 `talks/:talkId` 同层同形态，按注册序先匹配者通吃——`/talks/new` 落进列表页（batchId='new'），编辑路由彻底不可达且无任何警告 / 同一资源下多类子路由时，动态段前必须加静态前缀段区分（如 `talks/edit/new`、`talks/edit/:talkId`）；路由验收逐条实测导航落点（URL+组件特征断言），不能只看路由表注册成功。


## 收口联动（v1.0.1）

验收流程走完（ACCEPT 合入或 REJECT 打回登记毕）后，**主动执行 `/duo-summary`** 输出阶段总结（工单动作链/关键数字/打回原因/下一步解锁序），不待用户追问。多票连验时以工作段落为粒度合并一次。
- **FastAdmin operate formatter 依赖 this.table**（trace-media 工单05 实例） / 自定义 operate 包装函数普通调用时 this=undefined，读 dragsort_url 炸 → 渲染中断卡 loading（页面级白屏症状）/ 包装处必须 `.call(this)`；审查 admin view/js 的 formatter 包装逐处过目。
- **HBuilderX cli launch web 的 --project 静默 fallback**（工单03 实例） / --project 传未导入项目的绝对路径时不报错，fallback 编译已导入的同名/主项目——端口递增、服务起的却是旧代码，视觉验证全盘失效 / 先 `project close` 其他项目再 launch（或传项目名），并做"标记热更"验证服务归属：往目标文件塞临时标记串，探针读 DOM 确认标记在再取证。
- **验收 worktree 基点必须复核为主线最新**（工单03 实例,初判曾误报） / EnterWorktree/新建 worktree 默认基于"当前 HEAD"——主仓若处 detached 老点（本案 9795349,落后主线十余提交）,检出即缺一堆后来入库的文件,首编白屏,易误判为"主仓有未提交依赖" / 建 worktree 后立即 `git log --oneline -1` 与主线最新比对,落后即以正确基点重建;确认缺文件时先 `git ls-files` 核其在库状态再下"未提交"结论;rsync 补齐是兜底而非根治（须排除 pages/ 保实施代码）。
- **vue 模板与事件绑定缺陷只有真实编译/点击才暴露**（工单03 r1 实例） / 双轴静态审查与 node --check 均验不出"view 未闭合/方法未定义"，交付方自述不可信 / 四步之④视觉步骤必做最小实证：页面可编译（无 Failed to compile overlay）+ 主入口可点击（无 handler undefined）；对 vue 票可先用零依赖标签配对扫描快速筛模板结构。
- **TP5 对 AJAX 的 GET 走 json_encode 分支**（工单08 S13-40 实例） / GET 带 X-Requested-With 头时 TP5 把整页 HTML json_encode 成字符串返回（引号转义+`\n` 字面量），页面内 __token__/表单正则提取全落空 / 脚本取登录页/含表单页一律裸 GET（不带 XHR/Accept 头），XHR 头只给真正期望 JSON 的接口。
- **PHP json_encode 默认转义 `/` 为 `\/`**（工单06 S13B-33 实例） / 原始响应字节里 `/uploads/x` 是 `\/uploads\/x`，contains 按字面 `/uploads` 匹配必 missing / 对原始响应做子串断言时按转义形匹配，或先经 python json.loads 归一再断言。
- **子模块场景 accept 须先合子模块层再合主仓层**（weight-mode 工单01 实例） / duo-review 的 ff-only 只做了主仓层合并（报告+子模块指针），backend 子模块层 duo/ 分支上的业务提交未合入子模块 main——worktree --force 清理时子模块独立 git clone 全部丢失 / accept 路径第 5 步改为：先 `git -C <子模块路径> merge --ff-only duo/<feature>/<NN>` 合入子模块，再在主仓做 ff-only 合入指针+报告；简报 DoD 也须加"子模块层提交后、主仓层提交前，在子模块目录 git push origin duo/<feature>/<NN>（防清理丢失）"。
- **TP5 `post('name/a')` 会杀死字符串入参**（工单11 r1 实例） / `typeCast 'a'` 对任意标量无条件 `(array)` 包裹（Request.php:1141-1143），表单传的 JSON 字符串 `'["x"]'` 变 `[0=>'["x"]']`，控制器里 `is_string($raw)` 恒假——json_decode 兼容分支成死代码，旧客户端姿势静默变业务拒绝（实测净增 4 条断言回归）/ `/a` 修"表单数组 500"类工单时必须先取原值判类型：字符串先 json_decode、已是数组直接过滤；验收必跑旧姿势正例 + A/B 基线对照；"保留 XX 兼容"未经实测不得写进报告。
- **`git checkout <branch> -- <path>` 写 index，后续 `checkout -- <path>` 是假还原**（工单11/12 验收实例） / 挂载实施版文件做实跑若用 `git checkout branch -- path`，该文件同时进 index；跑完 `git checkout -- path` 只从 index 还原＝没还原，staged 残留随后挡 ff 合并（merge Aborting）/ 临时挂载一律 `cp` 工作区文件（不碰 index）；已中招则 `git reset HEAD <path>` + `git checkout -- <path>` 双步还原。
- **受控 rebase 拼接式并集会复活被删除的旧行**（工单11 r2 合入实例） / 冲突块无脑 ours+theirs 拼接,对"新版整段替换旧版"型改动(r2 循环清理替代 r1 两行直删)会把旧行复活——`bash -n` 语法通过仍藏雷(`set -u` 下 unbound 运行时才炸,脚本死在清理段、run-all 计数 0/0);验收方定向 grep 捞 OK 行还会漏看"摘要行没打印" / 并集解冲突按语义取舍(替换型取新版,扩展型才拼接);解后必核**结果摘要行完整打印**(如 `== S13 结果：PASS=n FAIL=m`),grep 消息只统计已见摘要的运行;run-all 0/0 计数=脚本中途夭折的直接信号,当场追因。
- **测试套件动态化改造必须自造 fixture 前提**（工单13 r1 实例） / 把硬编码店铺改成动态后，原隐式依赖"该店铺有预置数据"（商品/证件/额度）的断言族会空转——空参数（`goods_ids[]=`）被控制器当空集放行返回成功，断言 actual=1 假成功（实测 9 断言空转，有机门禁整族失效）/ 动态化清单里每个"取自店铺的数据"都要问"这店铺没有这数据怎么办"——预置不可假设的（商品等）一律运行时 INSERT fixture+对称清理；空值参数的"成功"要当红灯看。
- **INSERT...SELECT 从可能不存在的源复制会静默 0 行**（工单14 r1 实例） / 复制源是"原挂主"这类悬空引用时 SELECT 为空、INSERT 0 行不报错，后续全链空转（实测：auth 复制 0 行→merchant 上下文空→两套件 15/101、43/141） / 种子 SQL 凡 INSERT...SELECT 必须末尾自验计数（SELECT COUNT 断言，非 1 即败），或直接显式 VALUES 种子；验收方对"复制/迁移类种子"先查源行存在性再跑。
- **TP5 `input()` 无后缀默认 `type='s'`，数组入参必炸**（工单11/15 双案同根） / `post('name')` 不带 `/a` 时框架按字符串强转（Request.php:1017 默认 `$type='s'`），数组/表单数组输入直接 `variable type error：array` 500（Request.php:1162）——"没写后缀"不等于"原样返回"，恰恰是最严的转换 / 读任何可能为数组的参数（ids/permissions/files 等）一律显式 `/a`；审查时对无后缀的 `post('…')` 逐处过目问"这参数会不会是数组"。
- **接口演练断言必须复刻前端真实传参姿势**（工单 mbo-score 01 实例） / batchQuickScoreResult 验收断言按直觉传 `params=[{...}]` JSON 字符串返回假成功（`get('params')` 得到字符串，foreach 告警空转循环零次、无写库、state=true）/ 断言前先看前端调用源码的参数序列化方式（Qs.stringify 产生 `params[0][assess_id]=…` 数组格式），照抄姿势再断言；"假成功"要对照写库副作用甄别（data 只回显入参=没执行）。
- **headless 下 el-table fixed 列按钮 innerText 为空**（工单 mbo-score 03 实例） / 断言按钮文案用 innerText 恒空串、误判按钮无文本（fixed="right" 列的独立表格副本视觉隐藏，innerText 遵循可见性）/ DOM 断言一律用 textContent / outerHTML，按钮存在性用 offsetParent+data-testid 双验。
- **合成 el.click() 在 el-table fixed 列副本上产生作用域伪影**（hsd 工单01 实例） / JS 合成点击（querySelector().click() / evaluate）点 el-table fixed="right" 列按钮时，Vue 闭包作用域解析异常——handler 收到的 row 为 undefined，请求打 `undefined` 主键、console 报 Invalid prop + Uncaught（无 .catch 时），极易误判为前端传参缺陷 / 行为类断言一律用真实输入事件（CDP click 工具/Playwright click），合成 click 只用于"按钮存在性"断言；遇"请求参数 undefined"先换真实点击复测再定性，console error 栈含 pptr:evaluate 即伪影铁证。
