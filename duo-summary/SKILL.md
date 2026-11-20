---
name: duo-summary
description: 工作流阶段总结——验收合入/打回/派工等关键节点后主动输出一屏式复盘（成果/流水线/风险/数据/下一步五段）；比 duo-status 多分析建议，比 duo-report 轻量即时，控制台 markdown 不落文件
metadata:
  version: "1.2.0"
---

# 阶段总结（duo-summary）

在 duo 工作流的关键节点输出一屏式复盘总结。与看板/报告的分工：**duo-status** 回答"工单什么状态"（数据看板，用户主动查）；**duo-report** 是正式归档日报（HTML 文件）；**duo-summary** 回答"这轮干成了什么、现在什么局面、接下来干什么"——即时分析，节点后主动跑。

## 主动触发时机（不待用户敲命令）

1. **验收 ACCEPT 合入后**（duo-review 尾部联动）
2. **REJECT 打回后**（duo-review 尾部联动）
3. **派工启动后**（duo-dispatch 尾部联动；多票并行时合并成一次总结）
4. feature 收官（全部工单 done）时
5. 用户敲 `/duo-summary`（手动兜底）

同一会话内连续动作（验收 A → 立即派工 B）不必每步都总结，以"用户视角的一个工作段落"为粒度合并输出。

## 输入

- `<feature>`（可选）：限定 `.scratch/<feature>/`；缺省取最近活跃（issues/ 有 mtime 变动的最新 feature）

## 数据源（只读）

1. `.scratch/<feature>/issues/NN-*.md` 全量 Status 行（四态+轮次+Rejects+提交短号）与 Comments（grep `— REJECT（Rejects:` 拿打回明细；grep 留档/存量/待拍板关键词）
2. 近 24h 主仓 git log（duo/ 分支与主干合并提交）
3. `tests/reports/` 最新 run-*.json（各套件分数）
4. `.scratch/<feature>/logs/` 最近 codex 日志 mtime（实施中任务的现场证据）
5. `.scratch/<feature>/tmp/` 最近探针/验尸脚本名（诊断线索，只列名不读内容）

## 输出结构（控制台 markdown，一屏内）

```markdown
## 🎯 <feature> 阶段总结 · <MM-DD HH:mm>

**本轮成果**
- <工单动作链：验收 X（提交）/打回 Y（Rejects: N，一句话原因）/派工 Z>
- <关键数字：S 套件 a/b、run-all 通过率、断言总数变化>

**流水线**
| 态 | 工单 | 备注（阻塞链/日志时间/合并提交） |

**问题与风险**
- <REJECT 明细一行一条>
- <留档账：存量缺陷/环境故障/演练代修清单>
- <需要用户拍板的事项>

**数据资产**
- <新增/更新的演练脚本、探针、报告>

**下一步**
1. <解锁顺序与建议动作，按优先级>
```

## 边界

- 只读，不改动任何文件（与 duo-status 同约束）
- 不做 duo-report 的正式归档职责（用户要存档时提示 `/duo-report`）
- 无工单动作的空窗期跑了就一句话带过（"流水线无变化：<现状一行>"），不硬凑五段
- 总结里的判断（如"零回归"）必须有当轮证据支撑（套件分数比对/断言族比对），没有证据就写"未验证"
