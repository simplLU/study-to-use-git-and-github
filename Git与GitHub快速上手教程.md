# Git 与 GitHub 快速、灵活使用教程（Windows）

本文按一条完整路径编排：配置身份 → 选择认证方式 → 创建仓库 → 第一次推送 → 日常协作 → 选择合适的工具 → 排错。命令默认在 PowerShell、Windows Terminal 或 VS Code 终端中执行。

## 0. 你的当前环境

已检测到：

- Git：`2.53.0.windows.3`
- VS Code 命令行：可用（`code --version` 可运行）
- GitHub CLI：未安装（`gh` 命令暂不可用）
- 全局 Git 配置：尚未创建

后续命令不会覆盖项目已有文件；涉及远程仓库的操作需要你登录自己的 GitHub 账号。

## 1. 先理解 Git、GitHub 和工具的关系

- **Git**：电脑上的版本控制程序，负责记录每次修改、分支、合并和回滚。
- **GitHub**：保存 Git 仓库的在线平台，负责远程备份、代码评审、Issue、Actions 和协作权限。
- **VS Code / GitHub Desktop / GitKraken**：Git 的操作界面或辅助工具；它们都在调用同一个 Git 仓库。
- **GitHub CLI（`gh`）**：在终端完成登录、创建仓库、PR、Issue 等 GitHub 操作。

核心概念：

```text
工作区（你正在编辑的文件）
   ↓ git add
暂存区（准备提交的修改）
   ↓ git commit
本地仓库（电脑上的历史记录）
   ↓ git push / git pull
远程仓库（GitHub 等网站上的副本）
```

`commit` 只保存到本机；`push` 才会上传到 GitHub；`pull` 会下载并整合远程更新。

## 2. 一次性配置 Git 身份

在终端执行：

```powershell
git config --global user.name "你的姓名或昵称"
git config --global user.email "与你的 GitHub 账号关联的邮箱"
git config --global init.defaultBranch main
git config --global pull.rebase false
git config --global core.autocrlf true
```

每条命令的目的：

- `user.name`、`user.email`：写入每次 commit 的作者信息；它们不是登录密码。
- `init.defaultBranch main`：新仓库默认使用 `main` 分支。
- `pull.rebase false`：团队新手更容易理解的 pull 策略，发生分叉时创建合并提交。
- `core.autocrlf true`：让 Windows 与 Linux/macOS 团队减少换行符差异。

检查结果：

```powershell
git config --global --list
git config user.name
git config user.email
```

隐私提示：公共提交会显示作者名和邮箱。若不希望公开真实邮箱，可在 GitHub 的 **Settings → Emails** 复制 `数字+用户名@users.noreply.github.com`，再作为 `user.email`。

## 3. 连接 GitHub：推荐 SSH，备用 HTTPS + PAT

### 方案 A：SSH（长期使用更省事）

1. 生成密钥（已有密钥时先看 `Get-ChildItem $env:USERPROFILE\.ssh`）：

```powershell
ssh-keygen -t ed25519 -C "你的 GitHub 邮箱"
```

一路按 Enter 可接受默认路径；建议为密钥设置口令。私钥 `id_ed25519` 只能留在电脑上，**绝不能上传或发给别人**。

2. 启动 Windows SSH Agent 并加载私钥：

```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

3. 复制公钥：

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Clipboard
```

打开 GitHub → **Settings → SSH and GPG keys → New SSH key**，标题填“我的 Windows 电脑”，粘贴并保存。

4. 验证连接：

```powershell
ssh -T git@github.com
```

首次出现主机指纹时输入 `yes`。看到“successfully authenticated”即成功（GitHub 不提供 shell 登录属于正常现象）。

### 方案 B：HTTPS + Personal Access Token（PAT）

GitHub 已不接受账号密码进行 Git 推送。打开 GitHub → **Settings → Developer settings → Personal access tokens**，创建 Fine-grained token，只授予目标仓库所需的 `Contents: Read and write` 权限，并设置过期时间。

远程地址使用 `https://github.com/用户名/仓库名.git`。第一次 push 时：

- Username：GitHub 用户名
- Password：粘贴 PAT（输入时不会显示）

Windows 凭据管理器通常会缓存凭据。PAT 泄露后应立刻在 GitHub 撤销并重新生成。

## 4. 创建并推送第一个项目（GitHub 网页方式）

### 4.1 在 GitHub 创建空仓库

GitHub 右上角 **+ → New repository**：填写仓库名，选择 Public 或 Private。若本地已有项目，建议不要勾选 README、`.gitignore`、License，避免首次合并冲突。复制 SSH 地址，例如：

`git@github.com:用户名/my-project.git`

### 4.2 在本地初始化并提交

```powershell
cd "D:\项目\my-project"
git init
git status
git add .
git commit -m "chore: initial commit"
git branch -M main
git remote add origin git@github.com:用户名/my-project.git
git remote -v
git push -u origin main
```

作用：

- `git init`：在当前目录创建 `.git` 历史数据库。
- `git status`：查看未跟踪、已修改、已暂存文件。
- `git add .`：把当前修改放入暂存区；提交前应检查是否包含密钥、密码、构建产物。
- `git commit -m`：创建带说明的本地版本。
- `git branch -M main`：统一主分支名称。
- `git remote add origin`：给 GitHub 地址起名 `origin`。
- `git push -u origin main`：上传主分支，并建立默认跟踪关系；以后可直接 `git push`。

### 4.3 建议马上添加 `.gitignore`

在项目根目录创建 `.gitignore`，按技术栈加入内容，例如：

```gitignore
.env
.env.*
node_modules/
dist/
build/
__pycache__/
.venv/
.idea/
.vscode/*.log
```

若敏感文件已被提交，之后再写 `.gitignore` 不会自动移除它，需要：

```powershell
git rm --cached .env
git commit -m "chore: stop tracking local secrets"
git push
```

已经公开的密钥必须在对应服务商后台撤销；只删除文件不等于历史中消失。

## 5. 每天最常用的工作流

开始工作：

```powershell
git switch main
git pull --ff-only
git switch -c feature/login
```

`pull --ff-only` 可避免不知情地生成合并提交；`switch -c` 创建并切换到功能分支。

完成一小段可验证的修改后：

```powershell
git status
git diff
git add src/README.md
git diff --cached
git commit -m "feat: add login form"
git push -u origin feature/login
```

提交信息建议使用 `feat:`、`fix:`、`docs:`、`refactor:`、`test:`、`chore:` 前缀，并描述结果而不是过程。

## 6. 用 Pull Request 协作

在 GitHub 打开刚推送的分支，点击 **Compare & pull request**。PR 中写清：改了什么、如何验证、是否有已知限制。评审通过后在网页上合并。

合并后本地清理：

```powershell
git switch main
git pull --ff-only
git branch -d feature/login
git push origin --delete feature/login
```

若团队要求先同步主分支再提交 PR：

```powershell
git fetch origin
git rebase origin/main
git push --force-with-lease
```

`--force-with-lease` 会在远程分支被别人更新时拒绝覆盖，比 `--force` 安全；公共主分支不要强推。

## 7. 冲突处理

当 `pull`、`merge` 或 `rebase` 提示冲突：

```powershell
git status
```

打开冲突文件，处理 `<<<<<<<`、`=======`、`>>>>>>>` 标记，保留正确内容，然后：

```powershell
git add 冲突文件
git commit                 # merge 冲突时
# 或：git rebase --continue # rebase 冲突时
```

想取消当前过程：

```powershell
git merge --abort
# 或：git rebase --abort
```

## 8. 安全、回滚与诊断

常用查看命令：

```powershell
git log --oneline --graph --decorate -20
git remote -v
git branch -vv
git diff HEAD~1
```

未提交的本地修改想暂存：

```powershell
git stash push -m "临时保存登录页面"
git stash pop
```

撤销工作区未提交修改（会丢失这些修改，执行前确认）：

```powershell
git restore 路径\文件名
```

修改最近一次提交说明（尚未 push 时）：

```powershell
git commit --amend -m "正确的提交说明"
```

典型问题：

- `Permission denied (publickey)`：检查 `ssh-add -l`、公钥是否贴到正确 GitHub 账号、远程地址是否为 SSH。
- `Repository not found`：仓库名/用户名拼写错误，或账号没有权限。
- `failed to push some refs`：先 `git pull --rebase`，解决冲突后再 push；不要直接强推主分支。
- 大文件被拒绝：使用 Git LFS（`git lfs install`），或把生成物移出 Git。
- 换电脑：安装 Git，配置身份，重新添加 SSH 私钥或使用新密钥，再 `git clone`。

## 9. 选择与 Git 搭配的工具

| 工具/网站 | 适合场景 | 主要用途 |
|---|---|---|
| **VS Code** | 写代码、查看 diff、解决冲突 | 内置 Source Control，安装 GitHub Pull Requests 扩展可管理 PR |
| **GitHub Desktop** | Git 初学者、个人项目 | 图形化 commit、分支、push、PR；底层仍是 Git |
| **GitHub CLI (`gh`)** | 喜欢终端、自动化 | 登录、创建仓库、Issue、PR、Actions |
| **GitKraken / Sourcetree** | 需要可视化分支图 | 复杂分支、冲突和历史浏览 |
| **GitLab / Gitee / Bitbucket** | 企业、国内访问或不同权限体系 | 同样使用 Git；只需更换 remote 地址 |
| **GitHub Actions** | 自动测试、构建、发布 | `.github/workflows/*.yml` 触发 CI/CD |
| **Git LFS** | 模型、音视频、设计源文件 | 大文件单独存储，避免普通 Git 仓库膨胀 |

### 安装 GitHub CLI（可选）

从 <https://cli.github.com/> 安装后：

```powershell
gh auth login
gh repo create my-project --private --source=. --remote=origin --push
gh pr create --base main --head feature/login --title "Add login form" --body "请检查登录校验和测试。"
```

向导中选择 GitHub.com、SSH 和浏览器登录。`gh repo create` 可把“创建远程仓库 + 绑定 origin + 首次推送”合成一步。

## 10. 把本地仓库换成其他平台

Git 不依赖 GitHub。以 Gitee 为例，创建远程仓库后：

```powershell
git remote set-url origin git@gitee.com:用户名/仓库名.git
git push -u origin main
```

查看或删除远程地址：

```powershell
git remote -v
git remote remove origin
```

## 11. 推荐的学习顺序与最小规则

先掌握：`status`、`add`、`commit`、`log`、`switch`、`pull`、`push`、`fetch`。每次提交前运行 `git diff`，每项功能使用独立分支，主分支保持可运行。不要提交 `.env`、私钥、密码、依赖目录和构建产物；不要把 `git push --force` 用在团队共享主分支。

建议在第一个项目中完成这次练习：新建分支 → 修改 README → 提交 → push → 创建 PR → 合并 → 本地 pull → 删除分支。做完这条闭环后，再学习 rebase、Actions 和 Git LFS。

