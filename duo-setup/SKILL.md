---
name: duo-setup
description: 把 duo 套件装载进当前项目：codex 预检、AGENTS.md 生成/校验、最小工单规范落盘、套件安装到用户级。
---

# 装载 duo 套件（duo-setup）

在项目根目录执行四步装载：先检查 Codex，再把工程约定固化为 `AGENTS.md`，补齐最小工单规范，最后把四件套安装到用户级。已完成的前置步骤不因后续失败回滚。

## 铁律

1. 所有产出用中文；安装/复制用户级目录的步骤由 Claude 执行，Codex 不越权写用户主目录。
2. `AGENTS.md`、简报、报告和安装说明不得包含生产远程入口、数据库账号、访问令牌、私有证书路径或任何生产查询步骤；只写“做什么/怎么验”。
3. setup 只创建或校验装载产物，不改业务源码，不重写既有 `Claude.md`/`CLAUDE.md`、工单历史或姊妹 skill 正文。

## 1. Codex 预检（以 `codex exec` 实跑为准）

1. 在项目根执行 `codex --version`。命令缺失时报告一句话修复：`npm i -g @openai/codex`；若 nvm-windows 全局包出现拒绝访问/ACL 问题，提示参照用户全局 CLAUDE.md 的 nvm-windows ACL 环境备忘（如有）用 `icacls` 修复对应全局包目录后再试。
2. **就绪判据 = `codex exec` 实跑探活**（不依赖 `codex login`——国内用户常走第三方大模型厂商：`~/.codex/config.toml` 自定义 model_provider/base_url + 环境变量 API key，此路线无官方 OAuth 登录态但 exec 完全可用）：

```bash
codex exec --skip-git-repo-check "只回复两个字符：OK" 2>&1 | tail -3
```

   输出含模型应答（OK）即**就绪**，并在汇报注明检测到的接入路线线索（官方 OAuth 或自定义 provider，读 `~/.codex/config.toml` 的 model_provider 段落名即可，**不得**回显 base_url 完整地址与 key 名值）。超时（建议 120s）或报错时**未就绪**，记录原始错误并按线索给出修复方向：OAuth 路线提示用户自行 `codex login`（skill 不得代输代点）；provider 路线提示核对 config.toml 与环境变量（同样不回显敏感值）。
3. `codex login status` 降级为辅助信息（供参考，不作就绪判据）；探活失败而 login status 正常时重试一次 exec 并复述两次输出。
4. 按项目约定检查 WSL 可达（exec 探活在**实际派工用的那侧**跑——duo-dispatch 在 WSL 启动 codex，则 WSL 侧探活）；本项目使用：

```bash
wsl.exe -d Ubuntu-26.04 -- bash -c '. ~/.nvm/nvm.sh && codex --version && codex exec --skip-git-repo-check "只回复两个字符：OK"'
```

5. 任一预检失败都记录命令、退出码或关键错误，但不阻塞第 2、3 步；预检结果（含接入路线）最终进入装载汇报与 `.duo-context.md`。

## 2. 生成或校验 AGENTS.md

1. 检查项目根 `AGENTS.md`。存在时校验且缺节补写三节，标题必须为 `## 工程约定摘要`、`## 构建与测试命令`、`## 禁区红线`。
2. 不存在时，读取 `Claude.md`/`CLAUDE.md`、开发指南、技术盘点和领域术语文件，提炼成 40-60 行、面向 AI 工程师的一屏文档。
3. 三节最低内容：
   - 工程约定：交流语言、技术栈边界、目录/模块约定、命名和最小改动原则。
   - 构建与测试：启动/构建命令、本地验证入口、视觉验证和快照落盘约定、API 演练断言风格。
   - 禁区红线：危险操作授权、遗留红线区、数据结构变更对照要求，以及“不连接、不读取、不写入生产”的抽象禁令。
4. 写入前做脱敏自检：对全文做大小写不敏感扫描，覆盖远程登录形式、口令、token、证书/私钥路径、生产主机和数据库连接串；命中即删除或改写为抽象规则。来源文档里的敏感内容不得搬运。
5. 提炼来源全部缺失时，只生成三节骨架，在缺内容处写 `TODO: 待补充项目实测命令或约定`，不得编造。

## 3. 落盘最小工单规范

1. 若 `docs/agents/issue-tracker.md` 存在，只校验其说明了一 feature 一目录、spec 与 issue 路径、`Status:` 行和 `## Comments` 追加约定；不满足时补写缺口，不重写无关内容。
2. 不存在时创建以下最小版本：

```markdown
# Issue tracker: Local Markdown

Issues and specs live in `.scratch/`.

## Conventions

- One feature per directory: `.scratch/<feature-slug>/`.
- The spec is `.scratch/<feature-slug>/spec.md`.
- Tickets are `.scratch/<feature-slug>/issues/NN-<slug>.md`, numbered from `01`; never combine tickets into one file.
- Record triage state in a `Status:` line near the top of each ticket.
- Append discussion and review history under `## Comments` at the bottom; do not rewrite history.

## Publish / fetch

- "Publish to the issue tracker" means create the relevant file under `.scratch/<feature-slug>/`.
- "Fetch the relevant ticket" means read the referenced local markdown file.
```

3. 规范步骤完成后记录目标路径和是否新建。

## 4. 安装套件到用户级

1. 判断当前仓库是否为源头：`.claude/skills/` 下同时存在 `duo-dispatch`、`duo-review`、`duo-code-review`、`duo-setup` 四个目录即为源头。
2. 源头模式下，由 Claude 复制并同步四个完整目录到用户级 skills 目录；目标已存在时按升级语义覆盖。Git Bash 使用 `~/.claude/skills/`；Windows 原生目标为 `C:\Users\<用户>\.claude\skills\`。
3. Git Bash 示例：

```bash
mkdir -p ~/.claude/skills
for skill in duo-dispatch duo-review duo-code-review duo-setup; do
  rm -rf ~/.claude/skills/"$skill"
  cp -R ".claude/skills/$skill" ~/.claude/skills/
done
```

4. 在其他项目执行时，不复制仓库内文件；先检查用户级四目录是否齐。缺失时指引二选一：到源头仓库重跑本 skill，或套件发布后用 `npx skills add` 安装。
5. 目标目录不可写、复制失败或四件套缺失时停在该步；报告目标路径、缺失项和失败原因，保留已完成步骤，不回滚。

## 5. 首次运行 onboarding（自动学习仓库规范与用户习惯）

项目根无 `.duo-context.md` 时执行（存在则跳过；用户显式要求"重建上下文"时覆盖重建）。产出一份首次运行推荐，供 duo-dispatch 生成简报时自动引用。

**数据源**（按序扫描，存在才读；全部缺失则写骨架并标 TODO，不编造）：

1. **Codex 近期会话**：`~/.codex/sessions/`（或 `$CODEX_HOME/sessions/`）按修改时间取最近 2-3 个会话文件（.jsonl 逐行 JSON，大文件只读尾部），提炼：接入路线（官方/第三方 provider）、常见工作目录、反复出现的命令模式；
2. **Claude Code 近期会话与交接文档**：`~/.claude/projects/<当前项目编码路径>/` 最近 1-2 个会话文件（同样只取尾部摘要级信息）；项目内交接类文档（`HANDOFF*.md`、`docs/**/handoff*`、`.scratch/*/spec.md` 的进度态）；
3. **仓库规范**：`CLAUDE.md`/`Claude.md`/`AGENTS.md`、`docs/agents/`、开发指南、踩坑指南（`docs/pitfalls.md` 类活文档是用户习惯金矿——只读不搬）；
4. **用户全局习惯**：`~/.claude/CLAUDE.md`（如有）——语言偏好、工具链约定。

**产出**：项目根 `.duo-context.md`（四节，全部中文，40-80 行）：

```markdown
# duo 首次运行上下文（.duo-context.md）

## 环境事实
- Codex 接入路线：<官方 OAuth / 第三方 provider（provider 名）>；exec 探活结论与日期
- 派工侧：<WSL 发行版 / 原生>；worktree 根目录约定

## 仓库规范要点
- <从 CLAUDE.md/docs 提炼的 5-10 条最常踩的约定，注明来源文件>

## 用户习惯
- <从会话/pitfalls 提炼：交流语言、验证通路偏好、提交信息风格、敏感操作授权习惯等>

## 首次运行推荐
- <当前 .scratch 工单状态扫描：待派票据清单与依赖顺序建议>
- <验证通路：本地演练脚本/视觉验证入口（抽象描述，不含敏感串）>
```

**脱敏铁律**（与第 2 步同款）：写入前全文扫描远程登录形式、口令、token、证书路径、生产主机与连接串——命中即删除或抽象化；会话文件里的凭据**永不搬运**。`.duo-context.md` 含运行环境事实，是否入 git 遵循项目对仓库根新文件的口径（默认建议加入 .gitignore，除非项目约定全量纳管）。

## 6. 装载汇报

完成后输出五行一览，每行给结论与关键证据或失败原因：

```text
预检：<就绪/未就绪；接入路线（官方/provider 名）；失败项与建议>
AGENTS.md：<新建/校验通过/补节；三节齐全>
工单规范：<已存在/已新建；路径>
安装：<已同步/待 Claude 执行/失败；目标路径>
onboarding：<新建 .duo-context.md / 已存在跳过 / 骨架+TODO；数据源命中数>
```

## 故障与边界

- 提炼来源不存在：生成骨架并标 `TODO`，不猜测命令或模块健康度。
- AGENTS 脱敏扫描命中：先删除或抽象化，再复查一次；仍无法安全表达时保留骨架并失败报告。
- Codex 未就绪：不阻塞 AGENTS 与规范落盘，但汇报必须如实列出未就绪项。
- 用户级目录不可写：停在第 4 步，报告路径与权限错误；已完成文件不回滚。
- 任何复制、写盘或校验失败：保留已完成步骤和原始现场，如实汇报，不用部分结果冒充完成。
