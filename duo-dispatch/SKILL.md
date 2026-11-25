---
name: duo-dispatch
description: 接收工单号并把 ready-for-agent 工单派给 codex 实施；含历史 reject 反馈时自动携反馈重派同一分支与 worktree
metadata:
  version: "1.0.3"
---

# 派工给 Codex 实施（duo-dispatch）

按工单号派一张已就绪工单：读票、写自包含简报、登记 claimed、建立或复用隔离 worktree、后台启动 Codex，并以三源证据判定完成。**退出码不是完成依据。**

**机器前提**：WSL 发行版名与 node 加载方式（`. ~/.nvm/nvm.sh`）以宿主机器约定为准；文中 `Ubuntu-26.04` 为本机示例。

## 路径约定

- 编排方从当前项目根推导 `<主仓库WSL路径>`：Windows 项目根 `E:\x\y` 对应 WSL `/mnt/e/x/y`；示例 `E:\projects\my-app` 对应 `/mnt/e/projects/my-app`。
- `<仓库短名>` 是主仓库根目录名；worktree 使用 `~/duo-worktrees/<仓库短名>/<feature>-<NN>`，避免跨仓库同名碰撞。
- `<worktree目录名>` 是 `~/duo-worktrees/<仓库短名>/` 下一级目录名，即 `<feature>-<NN>`；可用第 6 步 `<worktree>` 路径的基名确认。

## 输入

- `<NN>`：两位工单号；可搭配 `<feature>` 指明工单目录，未给时从当前 feature 目录识别。

## 流程（顺序执行，失败即停）

### 1. 读工单

1. 定位 `.scratch/<feature>/issues/NN-*.md`；命中数不唯一或文件不存在时终止。
2. 校验工单头包含 `Status: ready-for-agent`；不满足时终止，并说明当前状态与原因。
3. 提取目标、范围、逐条验收、依赖与禁止事项，作为简报素材。
4. 检查 `## Comments` 是否含 `— REJECT（Rejects:`（与 duo-review 4b.1 同契约）。若为重派，解析累计 `Rejects: M`；状态与 reject 反馈小节计数矛盾、反馈小节缺失或无法定位时停止，不启动实施。

### 2. 生成简报

1. 创建 `.scratch/<feature>/briefs/NN-brief.md`。
2. 简报必须自包含，至少写清：目标、范围、逐条验收标准、仓库工程约定摘要、禁区、DoD。
3. 若第 1 步识别到 reject 历史，在简报主体后附加 `## 历史反馈` 节：按时间顺序原样收录每一条 `### ... — REJECT（Rejects: ...）` 小节的标题与正文，不缩写、不改写、不省略任何修复要求；并在 DoD 中明确要求“修复全部历史反馈所列问题”。
4. DoD 必须包含：在 `duo/<feature>/<NN>` 提交，实施提交信息用 conventional commits 描述改动本身，禁止使用“置 done”等验收方归档措辞；完成报告写 `.scratch/<feature>/reports/NN-report.md`；报告含改动清单、自检证据（含必跑测试命令与结果）、偏离说明、已知遗留问题（无则写“无”）；不得纳入 `.duo-*` 运行产物（提交前 `git rm --cached --ignore-unmatch .duo-*`）；不修改无关文件；禁止 push、merge、rebase。
5. **脱敏铁律**：简报不含生产 SSH、数据库、密钥相关路径与凭据；需要环境细节时只给抽象约定或让实施方做本地预检。
6. 抽查完整简报（含历史反馈）后继续；发现敏感串立即删除后再继续。

### 3. 登记 claimed

1. 把工单 `Status:` 置为 `claimed`。
2. 同一行附派工时间与分支名；首次派工写 `claimed（YYYY-MM-DD HH:mm 派工，分支 duo/<feature>/<NN>）`，重派写 `claimed（YYYY-MM-DD HH:mm 重派，分支 duo/<feature>/<NN>；Rejects: M）`。
3. 不改工单其他内容；重派时不得删除或重写 `## Comments`。

### 4. 建/复用 worktree

首次派工在 WSL 侧执行增删；Windows 侧不可见 WSL 路径，不得用 Windows shell 操作 worktree 目录。

```bash
wsl.exe -d Ubuntu-26.04 -- bash -c '. ~/.nvm/nvm.sh && git -C <主仓库WSL路径> worktree add ~/duo-worktrees/<仓库短名>/<feature>-<NN> -b duo/<feature>/<NN>'
```

分支已存在或路径已存在时停在该步，先报告冲突，不清理或复用旧现场。

重派时不得执行 `worktree add`：校验 `~/duo-worktrees/<仓库短名>/<feature>-<NN>` 已存在、其检出分支恰为 `duo/<feature>/<NN>`，且工作区除 `.duo-*` 运行产物外无未预期改动。现场缺失、分支不一致或存在未预期改动时停在该步，保留工单 `ready-for-agent` 与 Rejects 计数，报告现场状态等待人工裁决；不新建、不清理、不覆盖交付现场。

### 5. 送简报

把简报拷入 worktree 根 `.duo-brief.md`；该文件不提交，仅供本次会话读取。

```bash
wsl.exe -d Ubuntu-26.04 -- bash -c 'cp <主仓库WSL路径>/.scratch/<feature>/briefs/NN-brief.md ~/duo-worktrees/<仓库短名>/<feature>-<NN>/.duo-brief.md'
```

### 6. 启动实施

先创建日志文件；首次派工用 `.scratch/<feature>/logs/NN-codex.log` 与 `.duo-last-message.txt`，重派用 `NN-codex-r<M+1>.log` 与 `.duo-last-message-r<M+1>.txt` 区分历史现场。再按工单 01 定案契约原样使用以下模板；`<worktree>`、`<last-message文件>`、`<指令>`、`<日志>` 按本次派工替换：

```bash
wsl.exe -d Ubuntu-26.04 -- bash -c '. ~/.nvm/nvm.sh && cd <worktree> && codex exec -s workspace-write --add-dir <主仓库WSL路径>/.git --add-dir <主仓库WSL路径>/.git/worktrees/<worktree目录名> -o <last-message文件> "<指令>" < /dev/null > <日志> 2>&1'
```

`codex exec` 必须带两条 `--add-dir`：

1. `--add-dir <主仓库WSL路径>/.git`：授予共享主仓库 `.git` 可写权限；worktree 工作区在 WSL 本地 fs，而共享 `.git` 在主仓库。
2. `--add-dir <主仓库WSL路径>/.git/worktrees/<worktree目录名>`：授予共享 `.git` 内本 worktree 元数据目录可写权限。实测只有第一条时，普通 `git add/commit` 仍会在 `.git/worktrees/<名>/` 创建锁文件时被拒绝；两条齐备后才能正常提交。

本次提示词必须是：

```text
阅读 .duo-brief.md 并执行其中全部指令，完成后按简报 DoD 提交到当前分支
```

推荐替换后的形态：

```bash
wsl.exe -d Ubuntu-26.04 -- bash -c '. ~/.nvm/nvm.sh && cd ~/duo-worktrees/<仓库短名>/<feature>-<NN> && codex exec -s workspace-write --add-dir <主仓库WSL路径>/.git --add-dir <主仓库WSL路径>/.git/worktrees/<feature>-<NN> -o ~/duo-worktrees/<仓库短名>/<feature>-<NN>/.duo-last-message.txt "阅读 .duo-brief.md 并执行其中全部指令，完成后按简报 DoD 提交到当前分支" < /dev/null > <主仓库WSL路径>/.scratch/<feature>/logs/NN-codex.log 2>&1'
```

示例：`<主仓库WSL路径>`=`/mnt/e/projects/my-app`、`<仓库短名>`=`my-app`。

长任务后台运行的正确姿势（2026-11-23 Windows 宿主实例）：**不要在 `bash -c` 引号内加 ` &`**——wsl.exe 返回即 WSL 会话结束，后台任务被杀（日志文件不创建、无产物，且 `pgrep -fc "codex exec"` 会匹配到残留 bash 造成"在跑"假象）。应使用 Claude Bash 工具的 `run_in_background=true` 启动整条 wsl.exe 命令（引号内不加 `&`），由工具后台化并保活 wsl.exe；启动后 30 秒内核验真实二进制进程（`ps aux | grep "codex exec" | grep -v grep`）与日志文件已创建且有内容。启动后记录基线提交、日志路径与 `-o` 路径。

**执行铁律**：

1. `< /dev/null` 必须存在；Codex 会读 stdin，缺省会让后台任务挂死。
2. stdout/stderr 必须裸重定向到日志；禁止任何管道（`tail`、`grep`、`tee` 等）。
3. 重定向必须位于 `bash -c` 引号内；`< /dev/null`、`> 日志`、`2>&1` 都不得写在 Windows 侧执行段。

### 7. 完成检测

等待任务结束或到达工单约定的超时后，逐项核对三源：

1. `-o` 指定的 last-message 文件存在。
2. worktree 分支 diff 非空（以派工前记录的基线提交为对比起点）。
3. worktree 内 `.scratch/<feature>/reports/NN-report.md` 存在，并含改动清单、自检证据、偏离说明、已知遗留问题四要素。

三项齐备才可进入验收；缺任何一项都视为未完成。退出码、last-message 中的自述成功、“报告说已完成”，都不能单独作为完成依据。

**已知环境故障（2026-11-24 实例，连续两轮复现）**：codex exec 退出阶段写 `-o` last-message 文件报 `No such file or directory`（疑似实施方按 DoD「.duo-* 不入库」清理 untracked 文件所致），任务以退出码 1 结束但实施完备。处置：`-o` 文件缺失而其余两项齐备时，**以日志文件末尾 stdout 中的 codex 最终完成自述作为 last-message 等价证据**（日志裸重定向含全部输出），在验收归档中如实注明该现场备注；不得仅因 `-o` 缺失判未完成，也不得据此忽略其余两项的实质核验。

## 故障与边界

- **启动失败**：保留工单 claimed、日志与已建现场，报告失败点、命令、日志路径和可见错误；停在该步，不伪造完成报告。
- **重派现场缺失或不洁**：保留工单 `ready-for-agent` 与 Rejects 计数，保留分支、worktree、历史简报与日志；报告缺失/冲突项，不重建现场、不删除历史反馈。
- **超时**：按约定停止等待并报告超时、已流逝时长、日志/last-message/diff/报告当前状态；不猜测成功，不删除现场，等人工裁决。
- **空 diff**：报告三源核对结果与 last-message 摘要，判定未完成；不补写报告、不改动实施产物、不进入验收。
- **报告缺失或四要素不齐**：按未完成处理；可把事实状态写回工单 Comments，但不把 Status 改为 done。

## 简报规范陷阱与对策（活文档——派工前扫描，验收回灌在此追加）

- **简报必含"环境教训"节**：目标机器的已知陷阱必须前置告知实施方（从 `duo-review/SKILL.md` 的"常见陷阱与对策"节同步最新清单——TP5 模板过滤器/朴素幂等 SQL/勿显式 COLLATE/enum 字符串 UPDATE 等），否则实施方会在沙箱里重踩一遍才交付。
- **简报必含"完成报告入库"要求**：DoD 明确报告文件随分支 git 提交（worktree 删除后未入库的报告会丢失），并写明精确路径。
- **跨库票验证通路写成可复制命令块**：stash 保护清单（脏文件逐一列名）→ detach 分支头 → 执行迁移 → 跑演练 → 切回 → stash pop，一段 bash 让验收方零思考代跑。
- **沙箱限制预告知**：codex 沙箱禁 docker/目标端口、可能无 php——简报写明"脚本交付+报告附精确复现命令，由验收方代跑"，避免实施方浪费时间尝试不可能的自检。
- **演练脚本断言四姿势写进简报验收节**（pyget data 层/contains 四参/INSERT 全列/admin 不直打）——实施方脚本自伤是 round 1 打回的高频原因，预防成本远低于返工。
- **保护性业务约束必须在工单与简报里显式点名**（wechat-draftbox-sync 工单03 实例）/ 工单写「直发篇数计入 daily_limit」未点明「微信每账号每日仅 1 次群发」的既有硬保护，实施方把 daily_limit 当群发次数授权、删掉 todayCount≥1 保护改循环多篇发送，测试还固化了错误行为 / 涉及配额/频率/次数/限流类语义时，简报必须逐条列出既有保护性约束原文（代码位置+业务含义+「不得移除」），并要求测试断言保护仍然生效——不能假设实施方从代码推断出业务红线。
- **派工时间戳与分支名登记后立即提交工单文件**：避免验收期间 detached 切换把登记态弄丢或挡 checkout。
- **简报引用 .duo-context.md**：项目根存在 duo-setup 产出的 `.duo-context.md` 时，"环境教训"节并入其环境事实（Codex 接入路线等）与用户习惯要点，让简报带上该项目的定制上下文。
- **并行组连派**：工单标注 `并行组: <组名>` 且组内票均 ready 时可连续派工（每票独立 worktree 与日志，逐张走完第 1-6 步）；并行组最多 2-3 张同时在场，验收合入按组内依赖序串行进行（协议见 duo-review）。
- **Codex 限流中断的续跑模式** / `codex exec` 因 5 小时额度中断时半成品现场保留完好（worktree 未提交改动不丢）/ 不重派从头做：排程限额重置后（如 `sleep N &&` 后台）用**携上下文提示词**续跑（"你此前因限额中断，工作区已有半成品改动，阅读简报与现状继续完成并按 DoD 提交"），日志用 r<N+1> 后缀区分轮次。

## 收口联动（v1.0.1）

派工启动（codex 后台跑起、claimed 登记提交）后，**主动执行 `/duo-summary`** 给出当前流水线全景（在实施/待派/阻塞链），多票连派合并一次总结；与验收节点总结同理，不待用户敲命令。

- **演练造态必填列与字段长度前置核对**（trace-media 实例） / 造态 INSERT 漏必填列被 `2>/dev/null` 吞成假失败（fa_trace_product_batch.external_goods_id 必填）、trace_code 超 varchar(17) 静默截断致断言错位 / 简报"环境教训"节列明造态表的必填列与字段长度上限，INSERT 显式全列且造态值带长度自检。
- **断言数据源的分页偏移**（trace-media 实例） / 列表断言默认打第一页，挂载行（新造数据 id 靠前/靠后取决于排序）可能落在第二页，断言空转 FAIL / 断言查询显式带 offset/limit 或 sort 覆盖全量，造态后先核行落位再断言。
- **演练清理对称性**（trace-media 实例） / API 造态的级联产物（如建档批次落 fa_trace_batch_document）不在清理段，残留污染下一轮验收现场（审核台 74 条悬空文档事故）/ 清理段与造态段一一对称，级联表逐一 DELETE 前缀数据并复核计数归零。
- **Vue 单文件"零运行验证"是实施方通病**（trace-media 工单03 r1 实例） / node --check 只覆盖 script 段，模板未闭合标签、@tap 绑定未定义方法只有真实编译才暴露，报告却声称"逐项核对" / 简报 DoD 强制模板自检：零依赖标签配对扫描（栈式配对非自闭合标签）+ 事件绑定方法名 grep 双向核对（模板引用 ∪ methods 定义），报告附真实输出；验收方以真实 H5 编译复验。
- **跨端契约字段名单一来源**（trace-media 工单03 实例） / 工单 A 写"id="、工单 B 契约写"media_id"，前后端各照所写实现，联调期才发现链路不通 / 简报中的接口入参/返回字段名必须逐字引用后端实现（post('xxx') 原文），多工单共用的契约在 spec 冻结字段名后简报只引用不转写。
- **"列表预览=最新一条"断言在同秒消息下不可稳定**（merchant-app-chat 工单01 实例） / 会话列表 GROUP BY 在同秒并列消息间取行任意（d5/d8 同秒，两轮分别取到不同行，d5、d8 两版锚点先后假阴）/ 预览类断言只做结构（行存在/预览非空/未读/字段齐备），要断内容身份须先隔离该会话（清残留或保证目标消息无同秒并列）。
- **越权用例的"第三者"必须是真无关方**（merchant-app-chat 工单01 实例） / 坐席/员工按客服工作台设计本就可读本店客户会话，拿它断"越权空返回"必假阳 / 第三方账号须与目标店铺、会话双方均无关联（如另一店铺的商家账号），并前置清理其与任一方的会话残留。
- **共享测试店铺回绑的连环雷**（merchant-app-chat 工单01 实例） / 跨 feature 改共享 fixture 归属（店铺 142 回绑新商家 13900000242）后，依赖"OWNER=归属方"前提的演练整体断链，且未入 run-all 的演练长期不暴露（漂移 3 天无人知，A/B 基线才归因）/ 环境事实变更（店铺回绑/账号改绑）落地时全仓 grep 受影响演练与账号常量并同步修正；简报含"fixture 前提"说明（账号↔店铺归属矩阵）。
- **本地部署域名的生产构建产物 API 硬编码指向远程**（hsd 工单01 实例） / v4/v3 本地测试域名 serve 的是前端生产构建产物（.env.production 打包时 baseURL 写死 https://www 远程后端），浏览器打开 v4 验证前端按钮显隐实际打到生产后端——改动永远验不到，且远程库密码/数据与本地不同（登录失败假象）/ 验证"前端显隐+本地后端"全链路必须起前端 dev server（`pnpm dev`，.env.development 指向本地 API 域名），简报"环境教训"写明该项目的 dev server 端口与 API 指向矩阵；纯接口层验证可绕前端在部署域页面上下文直接 fetch 本地 API（同域无跨域）。
- **子模块实施票的 dispatch 命令三处特化**（hsd 工单01 实例） / worktree 按"主仓库"约定建在主仓会把提交落错仓库：实施仓库是 git 子模块时 worktree 须建在子模块仓库上，`.git` 是 gitfile、实际 git dir 在主仓 `.git/modules/<子模块>/`——`--add-dir` 第一条指向该实际 git dir（而非 `<主仓库>/.git`）；完成报告住主项目 `.scratch`（worktree 内没有），需追加第三条 `--add-dir <主项目>/.scratch` 最小授权 / 简报 DoD 写报告主项目绝对路径并注明"不随子模块分支提交"。
- **造数前必须 SHOW INDEX 核目标表唯一键**（hsd annual-performance-fixes 工单01 实例） / 断言脚本对考核表 INSERT...SELECT 新月位，撞上 (month,shop) 唯一键整语句原子回滚 0 行且报错点在语句中部，后续语句全不执行 / 简报"环境教训"节要求造数前 SHOW INDEX 列出目标表唯一键；真实月位被占用时改用"工资/明细行挂既有真实考核行"策略（不插考核行，仅插挂载行），清理按挂载工号对称删除
- **回归基线不可用存量结果行，须造数前现算快照**（同上工单01 实例） / 场景D拿存量 avg_scores 行当基线，重算值必然漂移（存量行是陈旧数据/旧口径算的，实测 77.44 vs 87.02 假回归）——且该假失败会浪费一整轮打回 / 简报验收节写死："回归基线=造数前先跑一次目标接口现算快照，造数后复跑对比"；快照样本避开受造数污染的口径（如选无组内转换的员工类）
- **简报环境教训必含"登录通路实测"条目**（同上工单01 实例） / 账号文档密码会过期（文档口径与库内实际不一致，且该系统登录为明文比对），断言脚本登录段一跑就死，且早期失败会连环暴露 cleanup 引用未建备份表等次生问题 / 简报环境教训写明"实跑断言前先单发登录请求验通路"，登录凭据从库内实测取得而非照抄文档；脚本登录失败路径不得触发引用未建资产的清理段（清理段对备份表做存在性判断）
