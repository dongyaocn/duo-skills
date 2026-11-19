# duo-skills

双代理（duo）工作流套件：**Claude Code 做验收编排 + Codex 做实施**，工单驱动、四步验收、双引擎互校。适用于任何 git 仓库的 feature 工单化开发。

## 套件构成

| Skill | 职责 | 调用 |
|---|---|---|
| `duo-setup` | 一次性装载：codex 预检、AGENTS.md 生成、最小工单规范落盘 | `/duo-setup` |
| `duo-intake` | 需求整理与工单拆分：澄清提问（拍板点）→生成 spec（Testing Decisions 先行）→拆可验收粒度工单→确认后置 ready-for-agent | `/duo-intake <需求>` |
| `duo-dispatch` | 派工：读票→自包含简报→登记 claimed→隔离 worktree→后台启动 Codex→三源证据判定完成 | `/duo-dispatch <NN>` |
| `duo-review` | 验收：双轴审查（Standards+Spec）→勾验工单标准→实跑测试→视觉验证→三源核对→ACCEPT 合并归档 / REJECT 分级反馈并携反馈自动重派（Rejects 满 2 升级人工）；终态产出 HTML 验收报告 + 控制台摘要卡 | `/duo-review <NN>` |
| `duo-code-review` | 双轴审查底座（duo-review 步骤①的组合件，也可独立用） | `/duo-code-review` |
| `duo-status` | 工单看板：按日期范围查看已完成/进行中/待办/待分诊四态，含轮次、Rejects、验收数据 | `/duo-status [日期范围] [feature]` |
| `duo-report` | 工作日报：HTML 单文件，大厂日报结构 × 五套风格（极简白/科技深色/商务蓝/数据驾驶舱/杂志风） | `/duo-report [日期范围] [风格1-5] [feature]` |

## 安装

```bash
# 全局安装（用户级，所有项目可用）
npx skills add <你的GitHub用户名>/duo-skills -g

# 或项目级安装（随项目走）
npx skills add <你的GitHub用户名>/duo-skills

# 未推 GitHub 前的本地试用：直接把四个目录拷入 ~/.claude/skills/（Claude Code）
```

## 依赖前提

- **Claude Code**（编排方）与 **Codex CLI**（实施方，`codex exec`）同机可用
- **git worktree**：实施在 `~/duo-worktrees/<仓库短名>/<feature>-<NN>` 隔离现场
- **WSL 环境**（Windows + WSL 双侧工作的机器；纯 Linux/Mac 把路径约定中的 `/mnt/<盘>/` 映射为本地路径即可）

## 快速上手

```text
.scratch/<feature>/               ← 工单区（随主仓库 git 管理）
├── spec.md                       ← 需求 spec
├── issues/NN-*.md                ← 工单（Status: ready-for-agent 即可派）
├── briefs/NN-brief.md            ← 派工简报（duo-dispatch 生成）
├── reports/NN-report.md          ← 实施报告（Codex 交付）
├── reports/NN-review.html        ← 验收报告（duo-review 生成）
└── reports/images/NN-*.png       ← 验收截图
```

```bash
/duo-setup          # 首次：装载套件与工单规范
/duo-dispatch 09    # 派 09 号票给 Codex（后台实施，完成时通知）
/duo-review 09      # 验收：四步取证 → ACCEPT 合并 / REJECT 自动重派
```

## 工作流要点

- **三源证据判完成**：last-message 文件 + 分支 diff 非空 + 报告四要素；退出码与自述不算数
- **四步验收**：双轴审查 / 勾验票面标准 / 实跑测试 / 视觉验证，每步留可复查证据
- **Reject 分级反馈**：硬违规 / 未申报扩展 / 顺带修 / 不修留档；Rejects<2 自动携反馈重派同一 worktree
- **ACCEPT 只允许 ff 合并**；归档后清理 worktree（仅 .duo-* 运行产物）

## 各 skill 详情

见各目录 `SKILL.md`（自包含，含路径约定、失败边界、幂等规则）。
