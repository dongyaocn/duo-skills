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

## 1. Codex 预检

1. 在项目根执行 `codex --version`。命令缺失时报告一句话修复：`npm i -g @openai/codex`；若 nvm-windows 全局包出现拒绝访问/ACL 问题，提示参照用户全局 CLAUDE.md 的 nvm-windows ACL 环境备忘（如有）用 `icacls` 修复对应全局包目录后再试。
2. 执行 `codex login status` 判断认证。未认证时提示用户自行交互执行 `codex login`；skill 不得代输、代点或伪造登录结果。
3. 按项目约定检查 WSL 可达；本项目使用：

```bash
wsl.exe -d Ubuntu-26.04 -- bash -c '. ~/.nvm/nvm.sh && codex --version'
```

4. 任一预检失败都记录命令、退出码或关键错误，但不阻塞第 2、3 步；预检结果最终进入装载汇报。

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

## 5. 装载汇报

完成后输出四行一览，每行给结论与关键证据或失败原因：

```text
预检：<就绪/未就绪；失败项与建议>
AGENTS.md：<新建/校验通过/补节；三节齐全>
工单规范：<已存在/已新建；路径>
安装：<已同步/待 Claude 执行/失败；目标路径>
```

## 故障与边界

- 提炼来源不存在：生成骨架并标 `TODO`，不猜测命令或模块健康度。
- AGENTS 脱敏扫描命中：先删除或抽象化，再复查一次；仍无法安全表达时保留骨架并失败报告。
- Codex 未就绪：不阻塞 AGENTS 与规范落盘，但汇报必须如实列出未就绪项。
- 用户级目录不可写：停在第 4 步，报告路径与权限错误；已完成文件不回滚。
- 任何复制、写盘或校验失败：保留已完成步骤和原始现场，如实汇报，不用部分结果冒充完成。
