# duo-skills

双代理（duo）工作流套件：**Claude Code 做验收编排 + Codex 做实施**，工单驱动、四步验收、双引擎互校。适用于任何 git 仓库的 feature 工单化开发。

## 套件构成

| Skill | 职责 | 调用 |
|---|---|---|
| `duo-setup` | 一次性装载：codex 预检、AGENTS.md 生成、最小工单规范落盘 | `/duo-setup` |
| `duo-intake` | 需求整理与工单拆分：澄清提问（拍板点）→生成 spec（Testing Decisions 先行）→拆可验收粒度工单→确认后置 ready-for-agent | `/duo-intake <需求>` |
| `duo-reschedule` | 存量工单并行重排：两两冲突分析（文件/语义/迁移表/演练数据）→ 派工波次方案（拍板）→ 改写并行标注与状态 | `/duo-reschedule [feature|工单号列表]` |
| `duo-dispatch` | 派工：读票→自包含简报→登记 claimed→隔离 worktree→后台启动 Codex→三源证据判定完成 | `/duo-dispatch <NN>` |
| `duo-review` | 验收：双轴审查（Standards+Spec）→勾验工单标准→实跑测试→视觉验证→三源核对→ACCEPT 合并归档 / REJECT 分级反馈并携反馈自动重派（Rejects 满 2 升级人工）；终态产出 HTML 验收报告 + 控制台摘要卡 | `/duo-review <NN>` |
| `duo-code-review` | 双轴审查底座（duo-review 步骤①的组合件，也可独立用） | `/duo-code-review` |
| `duo-portfolio` | 技术总监驾驶舱：跨 feature 进度/效能（一次通过率·打回分类·周期时间）/风险雷达三视图，HTML 驾驶舱可选 | `/duo-portfolio [范围] [html]` |
| `duo-adr` | 架构决策记录：起草（上下文/决策/后果+备选对比+冲突检测）与评审（决策 vs 代码现状漂移检测） | `/duo-adr <主题或编号>` |
| `duo-debt` | 架构债务台账：工单留档自动收割+利息分级（频率×成本）+偿还窗口建议（与排期联动） | `/duo-debt [登记|收割|偿还建议]` |
| `duo-ui` | UI 视觉设计：HTML 高保真稿（多屏多态+两轮拍板确认）·实现差异评审（阻断/细节分级）·设计 token 规范；已决稿=工单视觉基准 | `/duo-ui 设计|评审|规范 ...` |
| `duo-design` | 方案设计沙盘：模块边界→数据流→失败模式 FMEA→备选对比→回滚路径→验证策略，产出直通 duo-intake | `/duo-design <主题>` |
| `duo-status` | 工单看板：按日期范围查看已完成/进行中/待办/待分诊四态，含轮次、Rejects、验收数据 | `/duo-status [日期范围] [feature]` |
| `duo-report` | 工作日报：HTML 单文件，大厂日报结构 × 五套风格（极简白/科技深色/商务蓝/数据驾驶舱/杂志风） | `/duo-report [日期范围] [风格1-5] [feature]` |
| `duo-summary` | 阶段总结：验收/打回/派工等关键节点后主动输出一屏式复盘（先答三问：刚完成什么/接下来/需要配合什么） | `/duo-summary [feature]` |
| `duo-handoff` | 会话交接：重启 session/换机前固化流水线局面——先收运行态/造态/git 快照，六节交接文档落本项目 .scratch/handoffs/ 随 git 可追索（含 suggested skills 与入库脱敏红线） | `/duo-handoff [下个 session 聚焦方向]` |
| `duo-sync` | 套件自身迭代升级：status 版本与套件 tag 巡检 / pull 拉远端刷新本机安装副本 / push 提交回灌（先 fetch 防分叉、semver 纪律、版本硬校验、套件级 tag 发布、识别不到更新的排查表） | `/duo-sync [status|pull|push] [备注]` |

## 安装

```bash
# 全局安装（用户级，所有项目可用）
npx skills add <你的GitHub用户名>/duo-skills -g

# 或项目级安装（随项目走）
npx skills add <你的GitHub用户名>/duo-skills

# 未推 GitHub 前的本地试用：直接把四个目录拷入 ~/.claude/skills/（Claude Code）
```

**更新识别机制**：`npx skills update dongyaocn/duo-skills -g` 按全局锁 `~/.agents/.skill-lock.json` 条目的 `source + ref` 拉远端最新提交，比对技能目录内容哈希识别更新——不读 git tag，也不读 `metadata.version`。套件级 tag（`vX.Y.Z`，与 CHANGELOG 顶节版本一致）供人读版本 / `git describe --tags` / `-b vX.Y.Z` 钉装；`metadata.version` 供 `/duo-sync status` 核对安装副本。

**识别不到更新时按表排查**（先跑 `npx skills check` 看输出文案，再对号入座）：

| 现象 | 根因 | 对策 |
|---|---|---|
| `No global skills tracked in lock file` | 本机没有全局锁 `~/.agents/.skill-lock.json`（技能是手动拷入、从未 `npx skills add … -g` 安装），或锁 `version < 3` 被新版 CLI 当空锁（旧版 CLI 写入） | 手动拷入属正常，用 `/duo-sync pull` 刷新即可；要 CLI 纳管则执行 `npx skills add dongyaocn/duo-skills -g` 生成 / 重写 v3 锁（含 `skillFolderHash`），再 `npx skills check` 复核 |
| `cannot be checked automatically: No version tracking` | 锁条目缺 `skillFolderHash` / `skillPath` | 同上重装；重装后 `npx skills check` 复核 |
| 一直「up to date」但文件确实旧 | 安装时带 `-b <tag\|commit>` 把 ref 钉死 | 查锁条目 `ref`；去掉 `-b` 重装或改钉新 tag |
| `✗ Failed to check skills from source` | GitHub API 失败或匿名限流 | 设 `GITHUB_TOKEN` 后重试；或改用 `/duo-sync pull` 直拷 |
| CLI 完全不认识 duo 技能 | 技能是手动 `cp` 到 `~/.claude/skills/` 的，不在锁内 | 属正常；用 `/duo-sync pull` 刷新，或改用 `npx skills add` 纳管 |

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
