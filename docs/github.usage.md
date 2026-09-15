# GitHub 完整工作流教程：从新建远程分支到 Pull Request（VS Code 实操）

> 本教程演示完整的团队协作流程：**在 GitHub 网页上新建远程分支 → 用 VS Code 把仓库 Clone 到本地 → 切换分支修改文件 → Commit → Push → 发起 Pull Request → Review 并合并**。
>
> 所有截图均来自真实操作，演示仓库为 [yystju/git_doc](https://github.com/yystju/git_doc)。

## 目录

- [0. 前置准备](#0-前置准备)
- [1. 在 GitHub 网页上初始化仓库](#1-在-github-网页上初始化仓库)
- [2. 在 GitHub 网页上新建远程分支](#2-在-github-网页上新建远程分支)
- [3. 在 VS Code 中 Clone 仓库到本地](#3-在-vs-code-中-clone-仓库到本地)
- [4. 在 VS Code 中切换到开发分支](#4-在-vs-code-中切换到开发分支)
- [5. 修改文件](#5-修改文件)
- [6. 提交更改（Commit）](#6-提交更改commit)
- [7. 推送到远程（Push）](#7-推送到远程push)
- [8. 在 GitHub 上发起 Pull Request](#8-在-github-上发起-pull-request)
- [9. Review 与合并（Merge）](#9-review-与合并merge)
- [10. 同步 main 分支（收尾）](#10-同步-main-分支收尾)
- [附录 A：常见问题](#附录-a常见问题)
- [附录 B：命令速查表](#附录-b命令速查表)

---

## 0. 前置准备

### 0.1 网络代理

本教程环境使用本地 SOCKS5 代理（`127.0.0.1:1080`）访问 GitHub。为让 `git clone` / `git push` 正常工作，建议把代理写入 git 全局配置：

```bash
git config --global http.proxy  socks5://127.0.0.1:1080
git config --global https.proxy socks5://127.0.0.1:1080
```

> 撤销代理：`git config --global --unset http.proxy && git config --global --unset https.proxy`

SSH 方式（`git@github.com:...`）不走 HTTP 代理，需保证 22 端口可达 GitHub；若被墙可改用 SSH over HTTPS（见[附录 A](#附录-a常见问题)）。

### 0.2 Git 身份与 SSH 密钥

```bash
git config --global user.name  "yystju"
git config --global user.email "yystju@hotmail.com"

# 生成 SSH 密钥（如已有可跳过）
ssh-keygen -t ed25519 -C "yystju@hotmail.com"
```

把公钥 `~/.ssh/id_ed25519.pub` 的内容添加到 **GitHub → Settings → SSH and GPG keys → New SSH key**。验证：

```bash
ssh -T git@github.com
# Hi yystju! You've successfully authenticated...
```

### 0.3 登录 GitHub

浏览器中登录 <https://github.com>，后续网页操作均在此登录态下进行。

---

## 1. 在 GitHub 网页上初始化仓库

新建的空仓库（如 `yystju/git_doc`）没有任何提交，也就不存在分支，无法直接"新建分支"。所以先创建第一个提交来产生 `main` 分支：

打开仓库页面，点击 Quick setup 区域的 **creating a new file** 链接（或 **Add file → Create new file**）：

![空仓库 Quick setup 页面](images/01-repo-empty.png)

输入文件名 `README.md`，填写仓库介绍内容：

![创建 README.md](images/02-create-readme.png)

点击右上角 **Commit changes...**，填写提交信息（这是仓库的第一个提交）：

![填写 commit message](images/03-commit-readme.png)

提交成功后，仓库主页出现渲染好的 README，同时 `main` 分支被创建（1 Branch）：

![仓库主页](images/04-repo-main.png)

---

## 2. 在 GitHub 网页上新建远程分支

> **为什么要用分支？** 直接在 `main` 上改动会让主线代码处于"半成品"状态。正确做法：每个功能/每篇文档在独立分支上开发，完成后通过 PR 合并回 `main`。

在仓库主页点击左上角的**分支切换器**（显示当前分支名 `main` 的下拉按钮）：

![分支下拉框](images/05-branch-dropdown.png)

在下拉框中直接输入新分支名（例如 `feature/github-tutorial`）。由于该分支不存在，GitHub 会显示 **"Create branch feature/github-tutorial from main"** 选项：

![输入新分支名，出现 Create branch 选项](images/06-create-branch.png)

点击该项，GitHub 基于当前 `main` 创建远程分支并自动切换过去。分支切换器显示新分支名、仓库变为 **2 Branches**，并提示 "This branch is up to date with main"：

![分支创建完成](images/07-branch-created.png)

> 💡 也可以走 **Branches 页面**：点击 "View all branches" → 右上角 **New branch** 按钮，效果相同。

---

## 3. 在 VS Code 中 Clone 仓库到本地

打开 VS Code，欢迎页（Welcome）的 **Start** 区域提供了 **Clone Git Repository...** 入口：

![VS Code 欢迎页](images/08-vscode-welcome.png)

点击后弹出输入框，粘贴仓库的 SSH 地址（HTTPS 地址也可，见第 0.2 节的认证说明）：

```
git@github.com:yystju/git_doc.git
```

![输入仓库 URL](images/09-clone-url.png)

回车确认后，VS Code 让你选择**克隆到哪个目录**。选择一个父目录（本例为 `~/work/git_doc/build/demo`），VS Code 会在其中创建与仓库同名的文件夹 `git_doc`：

![选择克隆目标目录](images/10-clone-folder.png)

> 同样的事用命令行做就是：
> ```bash
> git clone git@github.com:yystju/git_doc.git
> ```

克隆完成后 VS Code 会弹出通知询问 **"Would you like to open the cloned repository?"**，点击 **Open** 即可在新窗口中打开刚克隆的仓库：

![克隆完成，打开仓库](images/12-repo-opened.png)

> ⚠️ Clone 只会把默认分支 `main` 检出到本地；你在第 2 步创建的远程分支 `feature/github-tutorial` 此时只存在于 `origin`（远程），见下一步。

---

## 4. 在 VS Code 中切换到开发分支

点击 VS Code **左下角状态栏的分支名**（当前显示 `main`），会弹出分支列表。列表中既有本地分支，也有远程分支（`origin/...` 开头）：

![分支列表](images/13-checkout-list.png)

选择 **`origin/feature/github-tutorial`**，VS Code 会自动创建同名的**本地跟踪分支**并切换过去。状态栏随即显示当前分支 `feature/github-tutorial`：

![已切换到 feature 分支](images/14-branch-switched.png)

> 命令行等价操作：
> ```bash
> git checkout feature/github-tutorial
> # 或 git switch feature/github-tutorial
> ```

---

## 5. 修改文件

在 VS Code 的资源管理器中打开 `README.md`，进行修改（本例在文末追加一段说明）。被修改的文件标签页会出现 **圆点（●）** 标记，左侧边栏的**源代码管理（Source Control）**图标上也会出现待提交数量角标：

![在 VS Code 中编辑 README.md](images/15-edit-readme.png)

---

## 6. 提交更改（Commit）

点击左侧**源代码管理（Source Control）**图标，打开提交面板：

- **Changes** 区列出所有被修改的文件（本例为 `README.md`）；
- 在顶部输入框填写 **commit message**（一句话说明"做了什么、为什么"）。

![源代码管理面板，填写 commit message](images/16-commit.png)

点击蓝色 **✓ Commit** 按钮完成提交（若有多个文件且未逐一暂存，VS Code 会提示是否提交全部更改）。

> 命令行等价操作：
> ```bash
> git add README.md
> git commit -m "docs: 在 feature 分支上演示 VS Code 提交流程"
> ```
>
> **Commit message 建议格式**：`<类型>: <描述>`，常用类型有 `docs`（文档）、`feat`（功能）、`fix`（修复）、`refactor`（重构）等。

提交成功后，**Graph** 区域会出现新提交记录，Changes 区清空：

![提交完成，Graph 显示新提交](images/16b-committed.png)

---

## 7. 推送到远程（Push）

Commit 只保存在**本地**。要让 GitHub 上的远程分支拿到这次提交，需要 **Push**：

点击源代码管理面板或状态栏的 **同步 / Synchronize Changes** 按钮（也有 ↓1 ↑1 上下箭头标识），或使用命令面板执行 **Git: Push**。

推送完成后，本地与远程分支保持一致，GitHub 上的 `feature/github-tutorial` 分支已经包含新提交：

![Push 完成后的状态](images/17-push.png)

> 命令行等价操作：
> ```bash
> git push
> # 首次推送新分支并建立跟踪关系：
> git push -u origin feature/github-tutorial
> ```

---

## 8. 在 GitHub 上发起 Pull Request

Push 之后打开仓库主页，GitHub 会自动检测到 `feature/github-tutorial` 有新提交，并在页面顶部显示黄色提示条，点击其中的 **Compare & pull request** 按钮：

![Compare & pull request 横幅](images/18-compare-banner.png)

进入 PR 创建页面，需要确认三处信息：

1. **base: main ← compare: feature/github-tutorial** —— 表示"把 feature 分支的改动合并进 main"，并显示 `Able to merge`（无冲突）；
2. **标题** —— 默认取第一条 commit message，可修改；
3. **描述（description）** —— 说明改动背景与内容，方便 reviewer 理解。

![填写 PR 标题与描述](images/19-pr-form.png)

点击绿色 **Create pull request** 按钮提交。PR #1 创建成功，页面变为 PR 详情页，Reviewer 可以在这里查看 Commits、Files changed（逐行 diff）、并发表评论：

![PR 创建成功](images/20-pr-created.png)

---

## 9. Review 与合并（Merge）

PR 的意义在于**评审**：`Files changed` 标签页展示每一行改动，可以逐行评论、请求修改（Request changes）或批准（Approve）。

确认无误后，点击 **Merge pull request** → **Confirm merge**：

![PR 合并成功](images/21-pr-merged.png)

合并成功后：

- PR 显示紫色 **Merged** 徽章；
- `feature/github-tutorial` 的提交被合入 `main`；
- GitHub 提示 feature 分支可以安全删除（**Delete branch** 按钮），删除远程分支是良好的收尾习惯。

> 命令行等价操作：
> ```bash
> git checkout main
> git merge feature/github-tutorial   # 本地合并（与 PR 二选一）
> git branch -d feature/github-tutorial
> ```

---

## 10. 同步 main 分支（收尾）

合并后，远程 `main` 已经领先本地。回到 VS Code / 终端同步一下：

```bash
git checkout main
git pull origin main
```

整个 **"建分支 → clone → 修改 → commit → push → PR → merge"** 的循环就完整走通了。下一次开发从 **"GitHub 新建分支"** 重新开始即可。

---

## 附录 A：常见问题

| 问题 | 解决方案 |
|------|----------|
| `git clone` / `push` 卡住或超时 | 给 git 配置代理：`git config --global http.proxy socks5://127.0.0.1:1080` |
| `Permission denied (publickey)` | 公钥未添加到 GitHub，或本地私钥不匹配；执行 `ssh -T git@github.com` 验证 |
| SSH 22 端口不通 | 改用 SSH over HTTPS：在 `~/.ssh/config` 添加 `Host github.com\n  Hostname ssh.github.com\n  Port 443\n  User git` |
| push 提示需要登录 | HTTPS 方式需要 PAT（Personal Access Token）认证；推荐改用 SSH 方式 |
| commit 出现 `Author identity unknown` | 配置 `git config --global user.name / user.email`（见 0.2 节） |
| PR 提示冲突无法合并 | 本地先 `git checkout main && git pull`，再 `git checkout feature/xxx && git rebase main`，解决冲突后重新 push |
| 误提交到 main | 若尚未 push：`git reset --soft HEAD~1` 后切到正确分支重新提交；团队仓库建议开启分支保护 |

## 附录 B：命令速查表

```bash
# 分支操作
git branch -a                         # 查看所有分支（含远程）
git switch -c feature/xxx             # 新建并切换本地分支
git push -u origin feature/xxx        # 推送新分支到远程
git branch -d feature/xxx             # 删除本地分支
git push origin --delete feature/xxx  # 删除远程分支

# 日常修改
git status                            # 查看工作区状态
git diff                              # 查看未暂存的改动
git add <file>                        # 暂存文件（. 表示全部）
git commit -m "type: 描述"            # 提交
git log --oneline --graph             # 图形化查看提交历史

# 远程同步
git fetch origin                      # 拉取远程更新（不合并）
git pull origin main                  # 拉取并合并到当前分支
git push                              # 推送当前分支

# 撤销
git restore <file>                    # 丢弃工作区修改
git reset --soft HEAD~1               # 撤销上次 commit（保留改动）
git revert <commit>                   # 用一次反向提交撤销历史提交
```

## 本教程相关产物

- 演示仓库：<https://github.com/yystju/git_doc>（PR #1 已真实合并）
- 本地演示副本：`build/demo/git_doc`
- 全部截图：`docs/images/`
