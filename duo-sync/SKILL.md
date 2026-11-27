---
name: duo-sync
description: duo-skills 套件自身的多机迭代升级——pull 拉远端刷新本机安装 / push 提交回灌先 fetch 防分叉 / status 三方版本巡检；含 semver 纪律与双机并发约定
metadata:
  version: "1.2.0"
---

# duo-skills 迭代升级（duo-sync）

管理 duo-skills 套件仓库（github.com/dongyaocn/duo-skills）的多机双向同步：本机迭代（经验回灌/新 skill/修订）推 GitHub，另一台机器拉取刷新。**源仓库与安装副本是两处**：`duo-skills` 仓是源头，`~/.claude/skills/duo-*` 是各机安装副本——只改源不刷安装，本机后续会话用的还是旧版。

## 输入

- `<模式>`：`status`（默认，巡检不改动）/ `pull` / `push`
- `<备注>`（push 时可选）：提交说明补充

## 路径与约定

- 源仓库：本机 `E:\uniappProjects\duo-skills`（WSL `/mnt/e/uniappProjects/duo-skills`；他机以 duo-skills 记忆/README 为准）
- 安装副本：`~/.claude/skills/duo-*/`（用户级，全部 duo-* 目录）
- 远端：`origin` → GitHub main；`npx skills update dongyaocn/duo-skills -g` 按全局锁 `~/.agents/.skill-lock.json` 条目的 `source + ref` 拉远端最新提交，比对技能目录内容哈希识别更新——**不读 git tag，也不读 `metadata.version`**；套件级 tag（`vX.Y.Z`，与 CHANGELOG 顶节版本一致）供人读版本 / `git describe --tags` / `-b vX.Y.Z` 钉装，`metadata.version` 供 status 核对安装副本
- 双机并发铁律：**push 前必 fetch**；分叉时 rebase 保线性历史，不 merge；lessons/陷阱类追加文件的冲突**双方内容都保留**（按时间或编号排序合并），不取单边

## 流程

### status（巡检，默认）

1. 源仓库 `git fetch origin` 后输出四行对照：
   - 源仓库 vs origin/main：`领先 N / 落后 N / 一致`（`git rev-list --count` 双向）
   - 安装副本版本：grep 各 `~/.claude/skills/duo-*/SKILL.md` 的 `metadata.version`，与源仓库同名 skill 对比，列出不一致项
   - 工作区状态：未提交改动清单
   - 最新 tag：`git describe --tags --abbrev=0` 与 CHANGELOG 顶节版本对比；`git ls-remote --tags origin` 核对远端 tag 是否同步
2. 有落后即提示 `pull`；有领先即提示 `push`；安装副本过期列出刷新指令；tag 与顶节版本不一致或远端缺 tag，提示发布未走完（回到 push 补打）。

### pull（入向：远端 → 本机）

1. 源仓库 `git pull --ff-only origin main`；分叉（另一机器与本地都推过）时停下列出双方差异提交，请示后 rebase：`git rebase origin/main`，lessons 冲突双保留
2. **刷新本机安装副本**（二选一，优先 a）：
   a. 直接覆盖：`cp -r /mnt/e/uniappProjects/duo-skills/duo-*/ ~/.claude/skills/`（注意尾部斜杠语义，逐目录覆盖）
   b. 标准安装器：`npx skills add dongyaocn/duo-skills -g`（按 main 最新提交全量拉）
3. 刷新后抽查一个 skill 的 `metadata.version` 与 CHANGELOG 顶版本一致，向用户报告"已更新到 vX.Y.Z + 本次变更要点"（读 CHANGELOG 最顶节）；同时报出最新 tag（`git describe --tags --abbrev=0`），与顶节版本不一致说明源头机器发版未完成

### push（出向：本机迭代 → GitHub）

1. 收集源仓库工作区改动（回灌的 lessons、新 skill、修订），`git add` 明确路径（勿 `add -A` 带入临时产物）
2. **版本纪律（硬校验 + semver，与改动面同步）**：
   - **版本硬校验（commit 之前必做）**：先 `git fetch origin`，对 `git diff --name-only <merge-base>`（`HEAD` 与 `origin/main` 的 merge-base，即待推提交 + 暂存区 + 未暂存改动的改动面；commit 前后任一时刻跑都有效）列出的全部 `SKILL.md` 逐文件核对 frontmatter `metadata.version` 是否有变更行；正文改了而版本未动 → **停止并列出文件**，升完版本再提交。可复制命令（无「未升版本」行 = 通过）：

     ```bash
     git fetch origin && git rev-list --left-right --count HEAD...origin/main
     # 首行输出「N M」：左 = 领先（待推提交数）、右 = 落后；右侧非 0 先 git pull --ff-only origin main（第 4 步还会再 fetch）
     # 比较基准取 merge-base、对象取工作区（已提交 + 暂存 + 未暂存）：落后 / 分叉时上游改动不会混进来冒充版本变更行
     BASE=$(git merge-base HEAD origin/main)
     for f in $(git diff --name-only "$BASE" | grep 'SKILL\.md$'); do
       git diff "$BASE" -- "$f" | grep -qE '^[+-]  version: "' || echo "未升版本: $f"
     done
     # 有「未升版本: <文件>」行 = 该文件改了正文未升版本，停止；一行都没有 = 通过
     ```

   - 经验回灌/文案修订 → **patch**：仅改受影响 skill 的 `metadata.version` 尾位
   - 新增 skill / 新增能力节 → **minor**：新 skill 的 version 起小版本，CHANGELOG 记节
   - 流程破坏性变更（改调用方式/删输入形态）→ **major**
3. CHANGELOG.md 顶部加节：`## X.Y.Z（YYYY-MM-DD）` + 逐条变更（哪个 skill、什么改动）
4. **push 前必 fetch**：`git fetch origin` 后若落后先 `git rebase origin/main`（另一台机器可能刚推过）
   - **套件级 tag 预检（push 之前，第 3 步 CHANGELOG 新节补完后做）**：取 CHANGELOG 顶节版本 `X.Y.Z`，`git tag -l "vX.Y.Z"` 已存在 → 该版本已发布过，同版本不得二次发布；把顶节升到新版本号（按第 2 步 semver 规则）后再继续
5. `git push origin main`；失败（远端又有新提交）回到第 4 步，最多重试 2 次后停下报告分叉详情
   - push 成功后打**套件级 tag**：读 CHANGELOG 顶节版本 `X.Y.Z` → `git tag -a vX.Y.Z -m "duo-skills vX.Y.Z"` → `git push origin vX.Y.Z`；tag 推送失败不算发布完成，重试并在报告里写明
6. push 成功后同步刷新本机安装副本（同 pull 第 2 步）——**自己的迭代自己机器也要生效**

## 边界

- 只管 duo-skills 仓与安装副本；业务仓库的 duo 工单流程由其余 12 件套负责
- 不做 force push；不删他机提交；lessons 冲突双保留
- 用户只要"看看有没有更新"时跑 status 即止，不擅自 pull/push

## 故障排查：`npx skills update` 识别不到更新

先在本机跑 `npx skills check` 看输出文案，对号入座：

| 现象 | 根因 | 对策 |
|---|---|---|
| `No global skills tracked in lock file` | 本机没有全局锁 `~/.agents/.skill-lock.json`（技能是手动拷入、从未 `npx skills add … -g` 安装），或锁 `version < 3` 被新版 CLI 当空锁（旧版 CLI 写入） | 手动拷入属正常，用 `/duo-sync pull` 刷新即可；要 CLI 纳管则执行 `npx skills add dongyaocn/duo-skills -g` 生成 / 重写 v3 锁（含 `skillFolderHash`），再 `npx skills check` 复核 |
| `cannot be checked automatically: No version tracking` | 锁条目缺 `skillFolderHash` / `skillPath` | 同上重装；重装后 `npx skills check` 复核 |
| 一直「up to date」但文件确实旧 | 安装时带 `-b <tag\|commit>` 把 ref 钉死 | 查锁条目 `ref`；去掉 `-b` 重装或改钉新 tag |
| `✗ Failed to check skills from source` | GitHub API 失败或匿名限流 | 设 `GITHUB_TOKEN` 后重试；或改用 `/duo-sync pull` 直拷 |
| CLI 完全不认识 duo 技能 | 技能是手动 `cp` 到 `~/.claude/skills/` 的，不在锁内 | 属正常；用 `/duo-sync pull` 刷新，或改用 `npx skills add` 纳管 |
