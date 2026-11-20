# 更新日志

skills-cli 更新识别说明：`npx skills update dongyaocn/duo-skills -g` 按 **GitHub main 最新提交**拉取覆盖（无独立版本服务）；每个 SKILL.md 的 `metadata.version` 供人核对本地版本，CHANGELOG 记录演进。

## 1.1.0（2026-11-20）
- **新增 duo-sync**：duo-skills 套件自身的多机迭代升级——status 三方版本巡检 / pull 拉远端刷新本机安装副本 / push 提交回灌（先 fetch 防分叉、rebase 线性、lessons 冲突双保留）；semver 纪律（回灌 patch / 新 skill minor / 破坏性 major）

## 1.0.0（2026-11-20）
- 首个正式版：13 件套（intake/reschedule/setup/dispatch/review/code-review/status/report/portfolio/adr/debt/design/ui）
- 经 merchant-subaccount feature 全程实战（12 工单、631 断言管线、4 轮打回闭环）
- 内置：HTML 验收报告 + 经验回灌自优化 + 并行拆分三层模型 + 受控 rebase + codex exec 探活 onboarding + 基准线审查 + 五风格日报 + 陷阱库 20+ 条
