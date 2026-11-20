---
name: duo-sync
description: duo-skills 套件自身的多机迭代升级——pull 拉远端刷新本机安装 / push 提交回灌先 fetch 防分叉 / status 三方版本巡检；含 semver 纪律与双机并发约定
metadata:
  version: "1.1.0"
---

# duo-skills 迭代升级（duo-sync）

管理 duo-skills 套件仓库（github.com/dongyaocn/duo-skills）的多机双向同步：本机迭代（经验回灌/新 skill/修订）推 GitHub，另一台机器拉取刷新。**源仓库与安装副本是两处**：`duo-skills` 仓是源头，`~/.claude/skills/duo-*` 是各机安装副本——只改源不刷安装，本机后续会话用的还是旧版。

## 输入

- `<模式>`：`status`（默认，巡检不改动）/ `pull` / `push`
- `<备注>`（push 时可选）：提交说明补充

## 路径与约定

- 源仓库：本机 `E:\uniappProjects\duo-skills`（WSL `/mnt/e/uniappProjects/duo-skills`；他机以 duo-skills 记忆/README 为准）
- 安装副本：`~/.claude/skills/duo-*/`（用户级，全部 duo-* 目录）
- 远端：`origin` → GitHub main；`npx skills update dongyaocn/duo-skills -g` 按 **main 最新提交**拉取覆盖（无独立版本服务，`metadata.version` 仅供人核对）
- 双机并发铁律：**push 前必 fetch**；分叉时 rebase 保线性历史，不 merge；lessons/陷阱类追加文件的冲突**双方内容都保留**（按时间或编号排序合并），不取单边

## 流程

### status（巡检，默认）

1. 源仓库 `git fetch origin` 后输出三行对照：
   - 源仓库 vs origin/main：`领先 N / 落后 N / 一致`（`git rev-list --count` 双向）
   - 安装副本版本：grep 各 `~/.claude/skills/duo-*/SKILL.md` 的 `metadata.version`，与源仓库同名 skill 对比，列出不一致项
   - 工作区状态：未提交改动清单
2. 有落后即提示 `pull`；有领先即提示 `push`；安装副本过期列出刷新指令。

### pull（入向：远端 → 本机）

1. 源仓库 `git pull --ff-only origin main`；分叉（另一机器与本地都推过）时停下列出双方差异提交，请示后 rebase：`git rebase origin/main`，lessons 冲突双保留
2. **刷新本机安装副本**（二选一，优先 a）：
   a. 直接覆盖：`cp -r /mnt/e/uniappProjects/duo-skills/duo-*/ ~/.claude/skills/`（注意尾部斜杠语义，逐目录覆盖）
   b. 标准安装器：`npx skills add dongyaocn/duo-skills -g`（按 main 最新提交全量拉）
3. 刷新后抽查一个 skill 的 `metadata.version` 与 CHANGELOG 顶版本一致，向用户报告"已更新到 vX.Y.Z + 本次变更要点"（读 CHANGELOG 最顶节）

### push（出向：本机迭代 → GitHub）

1. 收集源仓库工作区改动（回灌的 lessons、新 skill、修订），`git add` 明确路径（勿 `add -A` 带入临时产物）
2. **版本纪律（semver，与改动面同步）**：
   - 经验回灌/文案修订 → **patch**：仅改受影响 skill 的 `metadata.version` 尾位
   - 新增 skill / 新增能力节 → **minor**：新 skill 的 version 起小版本，CHANGELOG 记节
   - 流程破坏性变更（改调用方式/删输入形态）→ **major**
3. CHANGELOG.md 顶部加节：`## X.Y.Z（YYYY-MM-DD）` + 逐条变更（哪个 skill、什么改动）
4. **push 前必 fetch**：`git fetch origin` 后若落后先 `git rebase origin/main`（另一台机器可能刚推过）
5. `git push origin main`；失败（远端又有新提交）回到第 4 步，最多重试 2 次后停下报告分叉详情
6. push 成功后同步刷新本机安装副本（同 pull 第 2 步）——**自己的迭代自己机器也要生效**

## 边界

- 只管 duo-skills 仓与安装副本；业务仓库的 duo 工单流程由其余 12 件套负责
- 不做 force push；不删他机提交；lessons 冲突双保留
- 用户只要"看看有没有更新"时跑 status 即止，不擅自 pull/push
