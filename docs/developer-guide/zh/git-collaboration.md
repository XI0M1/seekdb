# 两人协作开发 Git 常用命令

本文面向两位开发者共同完成一个 seekdb Issue 的场景，介绍分支、提交、拉取、推送、合并及冲突处理。

## 1. 约定远程仓库名称

为避免双方使用不同名称，本文统一约定：

- `fork`：团队成员有权限推送的共享 Fork，例如 `https://github.com/XI0M1/seekdb.git`。
- `origin`：seekdb 官方仓库 `https://github.com/oceanbase/seekdb.git`，通常只用于获取最新代码和提交 Pull Request。

检查本机配置：

```bash
git remote -v
```

仓库所有者当前已经采用上述配置。同学第一次获取代码时可以执行：

```bash
git clone https://github.com/XI0M1/seekdb.git
cd seekdb
git remote rename origin fork
git remote add origin https://github.com/oceanbase/seekdb.git
git fetch --all --prune
```

同学还需要被添加为共享 Fork 的 Collaborator，才能向 `fork` 推送。

## 2. 每个问题创建一个功能分支

功能分支是一条独立开发路线。先从官方最新的 `master` 创建功能分支，再修改代码：

```bash
git fetch origin
git switch -c feature/issue-123 origin/master
git push -u fork feature/issue-123
```

常见命名方式：

- 新功能：`feature/short-description`
- 问题修复：`fix/issue-123`
- 文档修改：`docs/short-description`

一个问题只需创建一次功能分支，不需要每次修改代码都创建或合并分支。

如果已经修改代码但还没有提交，通常也可以直接创建分支，未提交的修改会保留：

```bash
git switch -c fix/issue-123
```

## 3. 同学加入已有功能分支

第一位开发者创建并推送分支后，另一位开发者执行：

```bash
git fetch fork
git switch --track fork/feature/issue-123
```

如果本地已经存在该分支：

```bash
git switch feature/issue-123
git pull --rebase fork feature/issue-123
```

查看当前所在分支及工作区状态：

```bash
git branch --show-current
git status
```

## 4. 日常开发流程

每次开始编码前，先进入功能分支并拉取同学的最新提交：

```bash
git switch feature/issue-123
git pull --rebase fork feature/issue-123
```

修改完成后检查内容：

```bash
git status
git diff
```

只暂存本次需要提交的文件，不建议直接使用 `git add .`：

```bash
git add path/to/source_file path/to/test_file
git diff --cached --check
git diff --cached
```

提交并推送：

```bash
git commit -m "fix: describe the behavior changed"
git push fork feature/issue-123
```

第一次推送新分支时使用：

```bash
git push -u fork feature/issue-123
```

设置上游分支后，后续通常可简写为：

```bash
git pull --rebase
git push
```

两人同时开发时，建议：

1. 开始编码前先拉取。
2. 每个提交只完成一件明确的事情。
3. 提交后尽快推送，减少双方修改同一代码的概率。
4. 推送被拒绝时先拉取并处理冲突，不要直接强制推送。

## 5. 推送被拒绝怎么办

如果同学已经推送了新提交，本地直接推送可能显示 `non-fast-forward`。先把远程提交接到自己的本地提交之前：

```bash
git pull --rebase fork feature/issue-123
git push fork feature/issue-123
```

这里的 rebase 只整理尚未推送的本地提交。不要对双方已经共享的历史随意执行 rebase，也不要使用 `git push --force`。

## 6. 同步官方 master

先更新本地 `master`：

```bash
git fetch origin
git switch master
git merge --ff-only origin/master
```

然后把官方主线的新提交合并进共享功能分支：

```bash
git switch feature/issue-123
git merge origin/master
git push fork feature/issue-123
```

共享分支推荐使用 `merge` 同步官方主线，因为它不会改写同学已经拉取的提交历史。

如果只是个人独占、尚未与别人共享的分支，也可以使用：

```bash
git rebase origin/master
```

## 7. 处理合并冲突

Git 无法自动判断两处修改应如何组合时，会报告冲突。首先查看冲突文件：

```bash
git status
git diff --name-only --diff-filter=U
```

冲突文件中通常包含三个标记：以 `<<<<<<< HEAD` 开始，使用 `=======` 分隔双方内容，并以 `>>>>>>> branch-name` 结束。手动编辑文件，保留正确的最终内容，并删除这三个标记。

### 7.1 继续或取消 merge

解决所有文件后：

```bash
git add path/to/conflicted_file
git status
git commit
git push
```

如果不想继续本次合并：

```bash
git merge --abort
```

### 7.2 继续或取消 rebase

解决当前冲突后：

```bash
git add path/to/conflicted_file
git rebase --continue
```

如果后续还有冲突，重复编辑、`git add` 和 `git rebase --continue`。如果不想继续：

```bash
git rebase --abort
```

rebase 完成后再推送：

```bash
git push
```

## 8. 合并功能分支

推荐在 GitHub 创建 Pull Request：

- 来源：`XI0M1/seekdb` 的 `feature/issue-123`
- 目标：`oceanbase/seekdb` 的 `master`

Pull Request 通过测试和评审后，由有权限的维护者合并。一般不需要开发者先把功能分支手动合并到自己的 `master`。

如果只是需要在共享 Fork 内进行本地合并，可以执行：

```bash
git switch master
git pull --ff-only fork master
git merge --no-ff feature/issue-123
git push fork master
```

对 seekdb 官方贡献时，应优先走 Pull Request，不要把这组本地合并命令当作官方合并流程。

## 9. 合并完成后清理分支

确认 Pull Request 已合并且分支不再需要后，删除本地分支：

```bash
git switch master
git branch -d feature/issue-123
```

删除共享 Fork 上的远程分支：

```bash
git push fork --delete feature/issue-123
```

清理本地已经失效的远程分支记录：

```bash
git fetch --all --prune
```

## 10. 常用检查和恢复命令

查看简洁状态：

```bash
git status --short --branch
```

查看提交图：

```bash
git log --oneline --graph --decorate --all -20
```

查看尚未提交的修改：

```bash
git diff
```

查看已经暂存的修改：

```bash
git diff --cached
```

取消暂存但保留文件修改：

```bash
git restore --staged path/to/file
```

临时保存未提交修改：

```bash
git stash push -m "work in progress"
git stash list
git stash pop
```

丢弃工作区修改会造成代码丢失，执行前必须确认文件内容不再需要：

```bash
git restore path/to/file
```

## 11. 两人协作速查表

每天开始开发：

```bash
git switch feature/issue-123
git pull --rebase fork feature/issue-123
```

完成一小段工作：

```bash
git status
git diff
git add path/to/file
git diff --cached
git commit -m "清晰描述本次修改"
git push
```

推送被拒绝：

```bash
git pull --rebase
git push
```

同步官方主线到共享功能分支：

```bash
git fetch origin
git switch feature/issue-123
git merge origin/master
git push
```

遇到冲突时先运行 `git status`，看清当前处于 merge 还是 rebase，再选择对应的 `--continue` 或 `--abort` 命令。
