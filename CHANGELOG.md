# 更新日志

skills-cli 更新识别说明：`npx skills update dongyaocn/duo-skills -g` 按 **GitHub main 最新提交**拉取覆盖（无独立版本服务）；每个 SKILL.md 的 `metadata.version` 供人核对本地版本，CHANGELOG 记录演进。

## 1.2.1（2026-11-20）
- **duo-review 升 1.0.2**：活文档回灌 8 条——前端验收 5 条（视觉验证全量不走样 / 「URL 不动·正文空白」即缺陷信号 / 静态证据不可替代渲染 / defineOptions 组件名对齐 KeepAlive cache / setup 抛错毒化全站 router + 漏检自认）+ 框架陷阱 2 条（EP 2.13.2 多级表头 v-for 子列 #header slot 被 patchFlag 白名单静默丢弃 → 一律 :label；structuredClone 遇 Pinia reactive Proxy 必抛 DataCloneError → JSON 序列化克隆）+ 流程纪律 1 条（活文档当轮回灌不攒账——安装副本会被覆盖，本地教训不 push 即丢；本条即该事故的产物）。来源：exam_web 综合研判工单03 round 2 打回与 round 3 人工修复。

## 1.2.0（2026-11-20）
- **新增 duo-summary**：阶段总结——验收/打回/派工等关键节点后主动输出一屏式复盘五段（成果/流水线/风险/数据/下一步）；与 duo-status（被动看板）、duo-report（正式归档）分工明确；空窗期一句话带过不硬凑
- **duo-review / duo-dispatch 升 1.0.1**：尾部新增"收口联动"——流程走完主动跑 duo-summary，不待用户敲命令（"主动总结"的落地机制）

## 1.1.0（2026-11-20）
- **新增 duo-sync**：duo-skills 套件自身的多机迭代升级——status 三方版本巡检 / pull 拉远端刷新本机安装副本 / push 提交回灌（先 fetch 防分叉、rebase 线性、lessons 冲突双保留）；semver 纪律（回灌 patch / 新 skill minor / 破坏性 major）

## 1.0.0（2026-11-20）
- 首个正式版：13 件套（intake/reschedule/setup/dispatch/review/code-review/status/report/portfolio/adr/debt/design/ui）
- 经 merchant-subaccount feature 全程实战（12 工单、631 断言管线、4 轮打回闭环）
- 内置：HTML 验收报告 + 经验回灌自优化 + 并行拆分三层模型 + 受控 rebase + codex exec 探活 onboarding + 基准线审查 + 五风格日报 + 陷阱库 20+ 条
