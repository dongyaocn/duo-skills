---
name: duo-status
description: 按日期范围查看工单看板——已完成/进行中/待办/待分诊四态汇总，含轮次、Rejects、合并提交与验收数据；只读扫描 .scratch 工单区
metadata:
  version: "1.0.0"
---

# 工单看板（duo-status）

按日期范围输出 duo 工单四态看板。只读，不改动任何文件。

## 输入

- `<日期范围>`（可选）：`2026-11-19`、`2026-11-19~2026-11-20`、`today`、`yesterday`、`this-week`；缺省=全部
- `<feature>`（可选）：限定 `.scratch/<feature>/`；缺省=扫描所有 feature 目录

## 数据源与解析规则

1. 工单文件：`.scratch/<feature>/issues/NN-*.md`，每张解析：
   - **Status 行**：`done（YYYY-MM-DD HH:mm ...）` / `claimed（...）` / `ready-for-agent` / `ready-for-human` / `needs-triage` / `needs-info` / `wontfix`
   - **done 时间**：Status 行内首个 `YYYY-MM-DD HH:mm`（即验收时间）——日期过滤以此为准
   - **轮次与 Rejects**：Status 行内的 `round N` 与 Comments 中 `— REJECT（Rejects:` 小节计数
   - **合并提交**：Status 行内的提交短号
   - **标题**：文件首个 `# ` 行
2. 验收数据（已完成票展示用）：`.scratch/<feature>/reports/NN-review*.html` 存在时提取测试计数行（如 `S9 213/213 · 增量 99/99`）；无则显示 `—`
3. 进行中（claimed）额外显示：worktree 路径推断（`~/duo-worktrees/<仓库短名>/<feature>-<NN>` 是否存在）、最近日志文件（`logs/NN-codex*.log` 的 mtime）

## 输出格式（控制台 markdown）

```text
## 📋 工单看板 · <feature 或 全部> · <日期范围>

### ✅ 已完成（N）
| # | 工单 | 验收时间 | 轮次/Rejects | 合并提交 | 验收数据 |
|---|------|----------|--------------|----------|----------|

### 🔄 进行中（N）
| # | 工单 | 派工时间 | 现场状态（worktree/日志） |

### 📥 待办（N）  ← ready-for-agent / ready-for-human 分列
| # | 工单 | 状态 | 依赖备注 |

### ⏳ 待分诊（N）  ← needs-triage / needs-info
| # | 工单 | 状态 |

**汇总**：已完成 N · 进行中 N · 待办 N · 待分诊 N ｜ 期间打回 M 次 · 平均轮次 X.X
```

日期范围外的 done 工单不列；claimed/ready 类不受日期过滤（它们是"当前态"）——但汇总行注明口径。无工单时输出"范围内无工单"一句话，不造空表。

## 边界

- Status 行格式异常（无时间戳/未知态）：归入"待分诊"并在该行标注 `⚠ 状态行异常`。
- 一个 feature 目录无 issues/ 子目录：跳过。
- 本 skill 不做任何 git 操作与文件写入。
