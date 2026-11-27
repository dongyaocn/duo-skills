# 更新日志

skills-cli 更新识别说明：`npx skills update dongyaocn/duo-skills -g` 按全局锁 `~/.agents/.skill-lock.json` 条目的 `source + ref` 拉远端最新提交，比对技能目录内容哈希识别更新（不读 git tag，也不读 `metadata.version`）；套件级 tag（`vX.Y.Z`）与顶节版本一致，供人读版本 / `git describe --tags` / `-b vX.Y.Z` 钉装，`metadata.version` 供 `/duo-sync status` 核对安装副本；识别不到更新见 duo-sync 的排查表。


## 1.6.1（2026-11-27）
- **duo-dispatch 升 1.1.1**：活文档回灌 1 条（duo-skills-paging-tag 工单02 验收实例）——简报转写 spec 条款须逐分句覆盖，限定语（适用面 / 豁免与留档 / 例外分句）不得省略；转写后对边界输入反向空走，消灭「按字面无法取证又无 reject 判据」的死区。
- **duo-review 升 1.1.1**：活文档回灌 1 条（同上）——双轴审查查出简报转写缺陷时先定责任归属：实施忠实复现简报 = 编排方缺陷，走验收代修 + 回写 spec 订正，不走 reject；两轴增量复核回执后再裁决。
- 来源：工单 02（列表分页必验落四个技能）验收 round 1——初轮双轴阻塞 1 条经代修 `8d7ea1a` 解除后 accept，随 v1.6.0 发版；本版仅回灌经验，无流程变更。

## 1.6.0（2026-11-27）
- **duo-intake 升 1.1.0**：spec 结构与工单模板新增**列表分页必验**——改动含会增长列表的工单，验收标准必含四条断言与两张截图。
- **duo-dispatch 升 1.1.0**：简报规范新增**列表分页契约**（逐字引用后端分页参数名与默认页大小、造 31 条以上断言、取证标准写入验收节）；活文档追加存量未分页列表的识别与留档对策。
- **duo-code-review 升 1.1.0**：smell 基线之后新增固定基线**列表分页基线（30 条红线）**（硬检查点，非判断题）；Standards / Spec 子代理 brief 各增一句。
- **duo-review 升 1.1.0**：勾验新增**套件必验项**（列表分页必验，未执行即 FAIL）；视觉验证要求「第 1 页 + 分页控件」「第 2 页」两张截图；reject 判据增至 6 条；§10 回灌动作改走 `/duo-sync push`。

## 1.5.1（2026-11-27）
- **duo-review 升 1.0.14**：活文档回灌 1 条（duo-skills-paging-tag 工单01 验收实例）——文档里的可复制命令块也是交付物：bash + 交互式 zsh 实跑，负例 / 正例 / 基准漂移三场景，对照命令坐实命中数。
- **duo-dispatch 升 1.0.6**：活文档回灌 2 条（同上）——简报指定文本与自检命令必须互洽（定稿前对理想产物空跑自检）；流程类需求写进简报前按序空走一遍（输入输出推演表、判据写实际输出形态）。
- 来源：工单 01（duo-sync 版本发布纪律）验收 round 1 的编排方简报缺陷与命令块实测缺陷；本版是新 push 流程首次用于回灌发版。

## 1.5.0（2026-11-27）
- **duo-sync 升 1.2.0**：push 流程新增**版本硬校验**（commit 之前：改了 SKILL.md 未升 `metadata.version` 即停止并列出文件）与**套件级 tag**（push 成功后按 CHANGELOG 顶节版本打附注 tag 并推送；tag 已存在须先升版本）；status 增「最新 tag 与顶节版本对比、远端 tag 同步」巡检；pull 刷新后报出 tag；新增「`npx skills update` 识别不到更新」排查表（五类原因）。
- **README**：安装节补更新识别机制与排查表（与 duo-sync 同口径）；套件构成表 duo-sync 行补「套件 tag」。
- **机制说明修订**：duo-sync / README / 本文件顶部说明统一为 skills CLI 实证口径——按锁条目内容哈希识别更新，不读 git tag 与 `metadata.version`；此前「按 main 最新提交拉取覆盖」的表述作废。
- **发布形态**：本版发布套件级 tag `v1.5.0`，并回补 `v1.4.2`（→ `226ccee`）。

## 1.4.3（2026-11-27）
- **duo-review 升 1.0.13**：活文档回灌 6 条（王者御品 pc-merchant-manage 工单05 验收实例）——额度 / 限额类列先核数据源语义（余量制）/ 截图瞬时悬浮元素先查布局公共组件（`Layer.load(0)`）/ 员工态 Forbidden toast 是布局既有行为 / 按钮禁用态取样式实值不凭观感 / `fullPage` 拍不到内滚区下半部 / 报告分项计数从日志统计。
- **duo-dispatch 升 1.0.5**：回灌 7 条简报侧教训（05 简报实例）——数据语义逐字引用数据源 / 选择器派工前 grep 实证 / 简报自洽扫描（镜头表 ↔ 页面规格）/ 展示格式写死 / 易变字段不写成常量 / 残留阻断条款与边界态镜头 / 报告附镜头逐行自证表。
- **发布形态**：自本版起回灌 push 后同步打附注 tag（`v1.4.3`，tag 号 = 节版本号）并推送——用户要求：版本 tag 便于更新识别与版本核对；此前各版本未打 tag。

## 1.4.2（2026-11-27）
- **duo-review 升 1.0.12**：活文档回灌 10 条（王者御品 pc-merchant-manage 工单03 验收实例）——`:class` 绑接口原值撞 Bootstrap 工具类 / 可见性断言禁用 innerText / 工单截图项逐条补拍 / 验收后全量 run-all 改写共享 fixture 须重取基线 / A/B 基线对照法 / MySQL 保留字列名 / HTML 报告图片禁 lazy / Vue 2 点击后等目标态再读 DOM / 名字核对覆盖嵌套属性 / 镜头间状态收口。
- **duo-dispatch 升 1.0.4**：回灌 5 条简报侧教训（05 派工实例）——简报契约派工前逐项实证 / 造态与清理动作的额度副作用写进镜头与清理段 / 员工视角镜头先核权限点可授予 / fixture 基线每票重取 / 参考片段与文字口径一致。

## 1.4.1（2026-11-26）
- **duo-handoff 升 1.0.1（用户反馈修正）**：交接文档落点从 OS 临时目录改为**本项目 `.scratch/handoffs/`** 随 git 管理可追索——临时目录会被系统定期清理，历史交接无法追索（用户定则）；脱敏从建议升为红线（入库即 git 历史永久留痕）；历史交接按日期堆叠成会话史，写新交接前先核对上一份的待办清况。临时目录仅作项目不可写时的降级并标注时效。

## 1.4.0（2026-11-26）
- **新增 duo-handoff（16 件套）**：会话交接——重启 session/清上下文/换机前固化流水线局面。三段式：①交接前收尾（运行态 TaskStop+重启方法留档、临时造态清理确认、双仓 git 快照、异常产物查明形态标注授权要求）；②六节文档落 OS 临时目录不入工作区（主线状态路径引用/环境运行态时点/待办优先级/分支安全约束/suggested skills/memory 索引）；③硬规则（路径引用不复制防双源漂移、凭据 cookie PII 脱敏、git 状态以当轮命令输出为准）。与 memory 分工：durable 事实走 memory，session 局面走 handoff。首战：王者御品 11-26 全线收官交接（双后台管道收尾+cookie 事故文件 `""` 查明+六节文档）。
- **README 套件构成表补全**：补登 1.1.0 起交付但表漏登的 duo-sync、1.2.0 的 duo-summary，加 duo-handoff——套件全景 16 件与仓库实际目录对齐。

## 1.3.6（2026-11-24）
- **duo-review 升 1.0.11**：活文档回灌 2 条（chat-im/11 + entry-reorg/01 联合视觉实例）——①uni H5 视觉脚本 hash 直改导航被静默弹回（与路由状态脱 sync，无错无 toast）→ 页面导航一律 `uni.navigateTo` API，工作台类页面先冷进"我的"页等 refresh 落 setMerchant；②HBuilderX dev 多端口 200 ≠ 新 bundle → 浏览器探测前先服务端 JS grep 特征串直判，清旧实例按端口杀监听进程（cli 需 IDE 活着，匹配名勿用 'cli' 防误杀 CefClientSubProcess）。

## 1.2.2（2026-11-24）
- **duo-dispatch 升 1.0.3**：回灌 3 条简报侧教训（merchant-app-chat 工单01 实例）——同秒消息下"列表预览"断言不可稳定（GROUP BY 取行任意，预览断言只做结构）；越权用例"第三者"必须真无关方（坐席按工作台设计可读本店会话）；共享测试店铺回绑的连环雷（fixture 归属变更须全仓 grep 受影响演练，漂移 3 天 A/B 才归因）。

## 1.2.1（2026-11-20）
- **duo-review 升 1.0.2**：活文档回灌 12 条——前端验收 5 条（视觉验证全量不走样 / 「URL 不动·正文空白」即缺陷信号 / 静态证据不可替代渲染 / defineOptions 组件名对齐 KeepAlive cache / setup 抛错毒化全站 router + 漏检自认）+ 框架陷阱 3 条（EP 2.13.2 多级表头 v-for 子列 #header slot 被 patchFlag 白名单静默丢弃 → 一律 :label；structuredClone 遇 Pinia reactive Proxy 必抛 DataCloneError → JSON 序列化克隆；Fantastic-admin Transition 布局页面模板根禁止 HTML 注释——params 互切全站空白且零报错）+ 流程纪律 4 条（活文档当轮回灌不攒账——安装副本会被覆盖，本地教训不 push 即丢；控制台错误与可处理警告纳入 FAIL 判据——零 error + 零可处理 warn；「全屏页」禁 position:fixed 遮盖 layout；控件语义与原表填报口径对齐——原表 -1 对应控件调低得分）。来源：exam_web 综合研判工单03 round 2 打回与 round 3 人工修复（用户连环指正 12 项闭环）。

## 1.2.0（2026-11-20）
- **新增 duo-summary**：阶段总结——验收/打回/派工等关键节点后主动输出一屏式复盘五段（成果/流水线/风险/数据/下一步）；与 duo-status（被动看板）、duo-report（正式归档）分工明确；空窗期一句话带过不硬凑
- **duo-review / duo-dispatch 升 1.0.1**：尾部新增"收口联动"——流程走完主动跑 duo-summary，不待用户敲命令（"主动总结"的落地机制）

## 1.1.0（2026-11-20）
- **新增 duo-sync**：duo-skills 套件自身的多机迭代升级——status 三方版本巡检 / pull 拉远端刷新本机安装副本 / push 提交回灌（先 fetch 防分叉、rebase 线性、lessons 冲突双保留）；semver 纪律（回灌 patch / 新 skill minor / 破坏性 major）

## 1.0.0（2026-11-20）
- 首个正式版：13 件套（intake/reschedule/setup/dispatch/review/code-review/status/report/portfolio/adr/debt/design/ui）
- 经 merchant-subaccount feature 全程实战（12 工单、631 断言管线、4 轮打回闭环）
- 内置：HTML 验收报告 + 经验回灌自优化 + 并行拆分三层模型 + 受控 rebase + codex exec 探活 onboarding + 基准线审查 + 五风格日报 + 陷阱库 20+ 条

## 1.3.5（2026-11-24）
- **duo-review 升 1.0.10**：活文档回灌 2 条（工单08 round 1 打回实例）——①全局组件 slot 契约用前查源码（PageHeader 无 #actions，未知具名 slot 被静默丢弃，静态全绿按钮消失；工具区按钮视觉验收逐个断言真实渲染）；②Vue Router 同层级同形态动态段冲突（:batchId 与 :talkId 先注册者通吃 /talks/new，编辑路由不可达零警告；多类子路由动态段前加静态前缀段，路由验收逐条实测导航落点）。

## 1.3.4（2026-11-23）
- **duo-review 升 1.0.9**：活文档回灌 1 条（工单05 验收漏检返工实例）——统一形态类需求拆单须枚举全部呈现面（「PC 统一矩阵」漏了渲染实例页，工作台≠唯一呈现点，拆单前 grep 全部呈现组件逐一入范围）；EP `.el-card` 自带 overflow:hidden 捕获卡头 sticky（1.0.8 布局捕获的嵌套形态）——卡片流吸顶须解锁卡片 overflow:visible；滚动检测窗口要算滚动容器 padding。

## 1.3.3（2026-11-23）
- **duo-review 升 1.0.8**：活文档回灌 1 条——Fantastic-admin 布局 `.main` overflow:hidden 捕获页内 sticky（sticky 相对最近 overflow 非 visible 祖先吸附而该容器自身不滚，真实滚动在 window → 页内吸顶永不触发；02 票起任务头吸顶潜伏失效被 06 票验收抓出）；对策：吸顶页自建滚动上下文（根容器 100vh+overflow-y:auto），滚动类交互验收必须实测 getBoundingClientRect 而非静态查样式。来源：exam_web 综合研判工单06 round 1 打回（H5 卡头吸顶不生效根因诊断）。
## 1.3.2（2026-11-21）
- **duo-summary 升 1.3.2**（用户追加反馈）——拍板项硬规则：问题一句+2-4 个可行方案（每方案一行：做法+代价/风险）+推荐标记；不允许只给"干不干"的单一问句；被否方案也要列出并写明为何不推荐。

## 1.3.1（2026-11-21）
- **duo-summary 升 1.3.1**（用户追加反馈）——全篇硬规则：任何位置提到工单号必须紧跟一句话业务说明（"商家删媒体报错修复（09）"），不允许光秃秃编号；流水线表工单列改为"编号+一句话"。

## 1.3.0（2026-11-21）
- **duo-summary 升 1.3.0**（用户拍板：汇报方式整改）——输出结构改为**硬性"先答三问"**：①刚完成了什么（业务语言，工单号/提交号只作坐标括注）②接下来自动做什么（没有就写无在飞任务）③需要你配合什么（拍板项必须在此区、每项带推荐选项）；实施细节（验收轮次/冲突合入手法/断言号）降入明细区或指向 HTML 报告，禁止在三问区堆砌。边界新增：拍板项不得埋在明细里让用户自己找。

## 1.2.7（2026-11-21）
- **duo-review 升 1.0.7**（陷阱库 +1，工单11/15 双案同根）：TP5 `input()` 无后缀默认 `type='s'`——数组入参必抛 `variable type error: array` 500；读任何可能为数组的参数必须显式 `/a`，"没写后缀"恰恰是最严的字符串强转。

## 1.2.6（2026-11-21）
- **duo-review 升 1.0.6**（陷阱库 +2，工单13/14 r1 实例）：① 测试套件动态化改造必须自造 fixture 前提——隐式依赖预置数据的断言族在"店铺无此数据"时空转（空参数被当空集放行返回成功=假绿灯）；② INSERT...SELECT 从悬空源复制静默 0 行——种子 SQL 必须末尾自验计数或改显式 VALUES。

## 1.2.5（2026-11-21）
- **duo-review 升 1.0.5**（陷阱库 +1，工单11 r2 合入实例）：受控 rebase 冲突块无脑 ours+theirs 拼接会复活"新版已替换删除"的旧行——`bash -n` 查不出、`set -u` 运行时才炸（脚本死在清理段、run-all 计数 0/0）；对策：并集按语义取舍（替换型取新版），解后必核结果摘要行完整打印，run-all 0/0 计数即中途夭折信号。

## 1.2.4（2026-11-21）
- **duo-review 升 1.0.4**（陷阱库 +2，工单11/12 验收实例）：① TP5 `post('x/a')` 无条件 `(array)` 包裹标量，json_decode 兼容分支成死代码——旧客户端 JSON 字符串姿势静默变业务拒绝；`/a` 类修复必须先取原值判类型，验收跑旧姿势正例 + A/B 基线对照；② `git checkout <branch> -- <path>` 会写入 index，后续 `checkout -- <path>` 假还原、staged 残留挡 ff 合并——临时挂载一律 `cp`，中招后 `reset HEAD` + `checkout --` 双步还原。

## 1.2.3（2026-11-20）
- **duo-portfolio 升 1.0.1**：html 模式行为升级（用户拍板）——①产物从 `temp/` 临时目录改为 **`.scratch/portfolio/portfolio-<日期>.html` 随 git 提交管理**，每次查看都归档，作为进度复盘的时序依据；②生成后自动经 Windows 侧 Chrome 打开（HKCU `App Paths\chrome.exe` 定位）；③进度视图 Feature 名必须带**中文描述**列（首票标题/spec 一句话概括业务语义）。附首份归档示例 `duo-portfolio/examples/portfolio-2026-11-20.html`（app.snsxny.com 全量 11 feature · 83 票快照）。

## 1.2.2（2026-11-20）
- **duo-dispatch 升 1.0.2 + duo-review 升 1.0.3**：trace-media 工单03 验收回灌 8 条——实施侧 4 条（演练造态必填列与字段长度前置核对 / 断言数据源的分页偏移 / 演练清理对称性——API 造态级联产物一并清 / Vue 单文件"零运行验证"通病——模板标签配对扫描入 DoD）+ 验收侧 4 条（FastAdmin operate formatter 依赖 this.table 须 .call(this) / HBuilderX cli launch web --project 静默 fallback——标记热更验服务归属 / 主仓干净 checkout 可能不可编译——rsync 补齐未提交运行依赖且排除 pages/ / vue 模板与事件绑定缺陷只有真实编译或点击才暴露——四步之④必做最小实证）。附：a873858 首推漏版本纪律，本节补记（教训：回灌 push 前过 duo-sync 纪律清单）。