# Git/GitHub 知识点总结（对话汇总）

> 目的：将前面对话中出现的**知识点**整理为一份便于查阅与落地的文档。涵盖：核心概念与安装、基础命令与常见参数、提交与分支规范、合并与变基、性能与维护、高阶技巧、常见情景与急救。

---

## 1. 概览

* **Git**：分布式版本控制系统；核心是不可变的提交快照与引用（分支/标签）。
* **GitHub**：以 Git 为底座的协作平台（PR、Code Review、Actions、Issues/Projects、安全治理）。
* **规范化目标**：统一提交与分支命名、强制评审与自动化检查、清晰的发布/回滚策略，提升可追溯性与交付质量。

---

## 2. 学习路径（提纲摘要）

1. 核心概念与安装
2. Git 基础操作（status/diff/add/commit/branch/merge/rebase/fetch/pull/push 等）
3. 分支策略与协作流程（Trunk-Based vs GitFlow）
4. 提交与分支命名规范（Conventional Commits 等）
5. PR 与评审要求（模板、CODEOWNERS、SLA）
6. 版本与发布（SemVer、tag、Release Notes）
7. 高阶技巧（rebase -i/cherry-pick/revert/stash/bisect/worktree/LFS/filter-repo）
8. 平台能力（Actions、Packages、Projects）
9. 安全与合规（保护分支、签名提交、Dependabot、Scanning）
10. 故障恢复与急救
11. 本地效率与工具链（别名、hooks）
12. 团队落地与度量

---

## 3. 核心概念

* **工作区（Working Directory）**：正在编辑的真实文件。
* **暂存区（Index/Stage）**：下一次提交的“候选清单”（`git add` 放入）。
* **本地仓库（Local Repo）**：`.git/` 目录，保存提交历史与对象。
* **远端仓库（Remote）**：GitHub/GitLab 上的副本，用于协作与备份。
* **Commit**：一次不可变的快照（含作者、时间、父提交、树对象）。
* **Branch**：指向某提交的可移动引用（如 `main`）。
* **HEAD**：当前所处的分支/提交指针。
* **.gitignore**：忽略不纳入版本控制的文件/目录。

---

## 4. 安装与初始化（要点）

* **安装**：

  * Windows：git-scm 安装包（含 Git Bash、Credential Manager）。
  * macOS：`brew install git` 或 Xcode Command Line Tools。
  * Linux：`apt/dnf/yum` 安装。
* **首次配置**：

  ```bash
  git config --global user.name  "你的名字"
  git config --global user.email "你的邮箱@example.com"
  git config --global init.defaultBranch main
  git config --global color.ui auto
  ```
* **凭据**：

  * **SSH（推荐）**：`ssh-keygen -t ed25519 -C "mail"` → 添加到 GitHub → `ssh -T git@github.com` 测试。
  * **HTTPS**：配 Credential Manager，简易但常需登录缓存。
* **忽略**：项目内 `.gitignore`；全局忽略 `core.excludesfile=~/.gitignore_global`。
* **首提与推送**：

  ```bash
  git init
  git remote add origin git@github.com:<user>/demo.git
  git branch -M main
  git add README.md && git commit -m "feat: initial commit"
  git push -u origin main
  ```

---

## 5. 常用命令与常见参数（速查）

### 5.1 查看/暂存/恢复

* `git status -sb`：简洁状态；`-b` 显示分支。
* `git diff`（工作区 vs 暂存区）；`git diff --staged`（暂存区 vs HEAD）；`--name-only`/`--stat`/`--word-diff`。
* `git add <path>`；交互式分块：`git add -p`；全量：`-A`；只更新已跟踪：`-u`；意图添加：`-N`。
* `git restore <path>`（丢弃工作区改动；可 `--source <commit>` 指定来源）。
* `git restore --staged <path>`（取消暂存；旧等价：`git reset HEAD <path>`）。

### 5.2 提交

* `git commit -m "msg"`（`-m` 可重复多次写多行）。
* 常用参数：

  * `-a`（自动暂存已跟踪修改）、`--amend`（修补上一条提交）、
  * `--signoff`（DCO）、`-S/--gpg-sign`（签名提交）、
  * `-F <file>`（从文件读取说明）、`--no-verify`（跳过钩子，慎用）、
  * `--allow-empty`（空提交）、`--author`/`--date`/`--reset-author`、
  * `--fixup=<sha>`/`--squash=<sha>`（配合 `rebase -i --autosquash`）。

### 5.3 历史与分支

* `git log --oneline --graph --decorate --all`；过滤：`--author/--grep/--since/--until`；详情：`-p`/`--stat`。
* `git branch -vv`（显示上游）；`-a`（含远端）；删除：`-d/-D`；重命名：`-m/-M`。
* `git switch <branch>`；创建并切换：`-c/-C`；回到上一个：`-`；游离：`--detach <sha>`。

### 5.4 合并与变基

* **merge**：`--ff-only`（只快进）、`--no-ff`（强制合并提交）、`--no-commit`、`-X ours/theirs`（冲突偏好）、`--abort`。
* **rebase**：`-i`（交互式）、`--autosquash`（配 fixup/squash）、`--onto <new> <old> <branch>`、`--continue/--skip/--abort`、`-r/--rebase-merges`。

### 5.5 同步远端

* **fetch**：`git fetch origin`（只更新远端跟踪分支，不改当前分支/工作区）；`--all`、`--prune`、`--tags`、`--depth`/`--deepen`/`--unshallow`、`--filter=blob:none`。
* **pull**：`fetch + 集成（merge 或 rebase）`；推荐 `git pull --rebase` 或 `--ff-only`。
* **push**：`-u/--set-upstream`（建立上游）、`--force-with-lease`（安全强推）、`--tags`、`--delete <branch>`、`--atomic`。

### 5.6 标签与发布

* `git tag`：轻量 vs 注解（`-a`，可 `-s` 签名）；推送 `--tags` 或单个标签名。

### 5.7 暂存工作现场 / 回退

* `git stash push -u/-a`、`list`、`show -p`、`pop`/`apply`、`drop`、`stash branch`。
* `git revert <sha>`（生成反向提交，安全可推）。
* `git reset --soft|--mixed|--hard`（危险：共享分支禁用 `--hard`）。
* `git reflog`（找回“走丢”的指针）。

---

## 6. 专题：题到即查

* `git init`：创建新仓库；`-b <name>` 指定初始分支；`--bare` 创建裸仓。
* `git remote add origin <url>`：添加远端；`-f` 添加即 fetch；查看：`git remote -v`。
* `git branch -M main`：强制把当前分支改名为 `main`（覆盖同名）。
* `git push -u origin main`：推送并设置上游，后续可直接 `git push/pull`。
* `git restore <path>`：丢弃工作区改动（可 `--source <commit>`）。
* `git restore --staged <path>`：取消暂存，仅影响 Index。

---

## 7. 策略与性能

### 7.1 协作与历史（对性能影响很小）

* **合并策略**：Merge commit / Rebase + FF / Squash。各有取舍（历史可读性 vs 粒度保留）。
* **分支模型**：Trunk-Based/GitHub Flow（短分支、快节奏） vs GitFlow（分支多、发布明确）。

### 7.2 存储/传输/检出（直接影响性能）

* **浅克隆**：`git clone --depth=1`；后续 `--deepen`/`--unshallow`。
* **部分克隆**：`--filter=blob:none` 按需取 blob。
* **稀疏检出**：`git sparse-checkout` 只检出需要目录。
* **大文件**：Git LFS 托管二进制，避免仓库膨胀。
* **维护**：`git maintenance run`（编排）、`git gc`/`git repack`（打包）、commit-graph、FSMonitor。

---

## 8. 维护命令与输出理解

* **`git maintenance run`**：高层编排，包含 `gc --auto`、写 commit-graph、`pack-refs`、（可选）prefetch。适合日常。
* **`git gc --aggressive`**：更激进的压缩与重打包，耗时长，偶尔使用（比如大改历史后）。
* **`git repack`**：底层打包工具（`-ad` 重新打并删旧包）。
* **输出字段理解**：`Enumerating objects` → `Counting` → `Delta compression using ... threads` → `Compressing objects` → `Writing objects` → `Total ... (delta N), reused M, pack-reused K`（激进模式通常**重算更多对象**，`reused` 下降、pack 可能更小）。
* **验证效果**：`git count-objects -Hv`、查看 `.git/objects/pack/*.pack` 体积，对比维护前后。
* **使用建议**：

  * 日常：`git maintenance run` / `git maintenance start`；
  * 重大历史变更后：可一次 `git gc --aggressive`；
  * CI：尽量用浅/部分克隆从源头减负。

---

## 9. 提交信息规范（多选其一并固化到团队）

* **Conventional Commits（推荐）**：`<type>(<scope>)!: <subject>`；`type` 如 `feat`/`fix`/`docs`/`refactor`/`test`/`build`/`ci`/`chore`/`revert`；支持自动化（changelog、语义化发版）。
* **简化前缀**：仅用 `feat:`/`fix:` 等，门槛低但自动化弱。
* **工单驱动**：`PROJ-123: ...` 或 `feat(x): ... (#123)`，追踪强但易忽略动机描述。
* **Emoji/Gitmoji**：可视化强、自动化弱。
* **命令式动词 + 50/72 规则**：经典但结构化不足。

**提交最佳实践**：

* 原子提交：**一次提交只做一件事**（一个逻辑变更）。
* 标题 ≤ 50 字符；标题与正文之间空一行；正文解释动机、影响、测试要点；必要时页脚（`BREAKING CHANGE:`、`Refs #123`）。
* 未推前可 `--amend` 整理；已共享历史**不随意改写**。
* 工具落地：`commitlint` + `husky`（本地钩子），CI 校验。

---

## 10. 合并（merge）与变基（rebase）

* **merge**：把另一分支合入当前分支；

  * 快进（FF）或产生合并提交（Non-FF）。
  * 参数：`--ff-only`、`--no-ff`、`--no-commit`、`-X ours/theirs`、`--abort`。
  * 优点：不改历史，保留分支脉络；缺点：主干可能更“分叉”。
* **rebase**：把一串提交**搬到**新的基底上重放，获得线性历史；

  * 参数：`-i`、`--autosquash`、`--onto`、`--continue/--skip/--abort`、`-r`。
  * 优点：线性整洁；风险：**改提交 ID**，共享分支慎用。
* **团队策略建议**：

  * 主干整洁：PR 采用 **Squash merge**；
  * 保留粒度：PR “Rebase and merge”（需文化约定）；
  * 需要完整脉络：普通 Merge（必要时 `--no-ff`）。

---

## 11. 高阶技巧

* **rebase -i**：交互式整理历史；`pick/reword/edit/squash/fixup/drop`；配 `--autosquash` 极高效。
* **cherry-pick**：把某个提交复制到当前分支（`-x` 保留来源）；合并提交用 `-m <parent>`。
* **revert**：生成反向提交，**安全可推**（回滚共享分支首选）。
* **stash**：临时搁置工作；`-u/-a` 是否包含未跟踪/忽略文件；`stash branch` 从 stash 建分支。
* **bisect**：二分搜索首个坏提交；可 `bisect run` 全自动。
* **worktree**：同一仓多个工作副本并行开发；`add/list/remove/prune`；分支同一时刻只在一个 worktree 检出。
* **Git LFS**：追踪大二进制；`git lfs track`/`install`；注意配额与历史迁移影响。
* **git filter-repo**：历史清洗（删大文件/敏感信息、重写作者、抽子目录等）；**改写历史须全员重拉**。
* **子模块/子树/monorepo**：

  * 子模块：外部仓的固定提交指针；权责清晰、更新流程复杂。
  * 子树：把外部仓内容合入目录，可选择性同步；开发者体验好，回溯略麻烦。
  * Monorepo：统一管理、跨项目协作强；需要工具链（partial/sparse、增量构建）。

---

## 12. fetch vs pull（常见混淆）

* **`git fetch origin`**：只更新远端跟踪分支（如 `origin/main`），**不改当前分支/工作区**。安全用于“先看看外面更新了啥”。
* **`git pull`**：`fetch + 集成（merge 或 rebase）` 到**当前分支**。
* **推荐用法**：

  * 线性同步：`git pull --rebase` 或 `git fetch && git rebase origin/main`；
  * 避免意外 merge：`git pull --ff-only`；
  * 清理远端删除的分支：`git fetch --prune`。

---

## 13. 规范化团队规则（摘录）

* **提交与分支**：原子提交；Conventional Commits；分支命名 `feature/…`、`bugfix/…`、`hotfix/…`、`release/…`、`chore/…`、`docs/…`；禁止直推受保护分支。
* **同步策略**：`pull.rebase=true`（或要求 `--ff-only`）；PR 必须通过 CI；必要时用 `--force-with-lease`，仅限个人分支。
* **质量与安全**：本地格式化/静态检查（pre-commit）；`.editorconfig` 与 `.gitignore` 入仓；发布打注解标签并遵循 SemVer；可启用提交签名；`rerere` 复用冲突解决。

**推荐全局配置**：

```bash
git config --global init.defaultBranch main
git config --global pull.rebase true
git config --global rebase.autoStash true
git config --global push.default simple
git config --global rerere.enabled true
# Windows/跨平台团队按需：
# git config --global core.autocrlf input
# 常用别名
git config --global alias.st "status -sb"
git config --global alias.lg "log --oneline --graph --decorate --all"
```

---

## 14. 急救与诊断

* **误操作找回**：`git reflog` → 找到目标 SHA → 新建分支/重置。
* **回滚错误变更**：共享分支优先 `git revert`；个人分支慎用 `reset --hard`。
* **仓库变慢/变大**：用浅/部分克隆与稀疏检出从源头减负；定期 `git maintenance run`；必要时 `git gc --aggressive`；大文件上 LFS；用 `git verify-pack` + `rev-list` 查找巨型对象。

---

## 15. 建议的练习路线（摘取）

1. 用 `git add -p` 练“原子提交”；
2. `rebase -i --autosquash` 整理 3–5 条提交；
3. `cherry-pick -x` 把修复移植到 release 分支；
4. 制造冲突并用 `rebase` 解决；
5. `stash` 保存/恢复工作；
6. `bisect run` 写脚本定位首个坏提交；
7. `worktree` 并行维护 main 与 release；
8. LFS 追踪一个大文件类型并观察 PR 指针 diff；
9. 在临时仓试一次 `filter-repo` 清理历史（务必在非共享副本）。

---

> 本文仅汇总“知识点”，需要将其中任意模块扩展为**实操剧本/模板脚本/清单**，可在对应章节基础上细化。
