# Git 操作指南

> 这不是一份 Git 命令速查表，而是一个**场景驱动的操作手册**。以下内容模拟了从零开始建立代码仓库、关联远程仓库、日常开发迭代的完整路径。

---

## 场景一：从零开始 — 创建本地项目并关联远程仓库

### 初始化本地仓库

假设你已经开始写代码，项目文件夹名为 `my-project`，里面已经有了若干源代码文件。

```bash
$ cd my-project
$ git init
Initialized empty Git repository in /path/to/my-project/.git/
```

此时 Git 已经开始追踪这个文件夹，但还没有记录任何文件。

### 添加并提交第一版代码

```bash
$ git add .
$ git commit -m "first commit: initial project structure"
[main (root-commit) a1b2c3d] first commit: initial project structure
 12 files changed, 384 insertions(+)
```

`git add .` 表示将当前目录下所有文件加入暂存区，`git commit` 则将它们永久记录到本地仓库历史中。

### 在 GitHub 上创建远程仓库

1. 登录 GitHub，点击右上角 **+** 号，选择 **New repository**。
2. Repository name 填写 `my-project`（与本地项目同名，便于识别）。
3. **不要勾选** “Add a README file” 或 “Add .gitignore”（因为本地已有内容）。
4. 点击 **Create repository**。

GitHub 会进入一个空仓库页面，并显示一行远程地址，类似：
```
git@github.com:你的用户名/my-project.git
```

### 关联本地仓库与远程仓库

```bash
$ git remote add origin git@github.com:你的用户名/my-project.git
$ git branch -M main
$ git push -u origin main
Enumerating objects: 12, done.
To github.com:你的用户名/my-project.git
 * [new branch]      main -> main
Branch 'main' set up to track remote branch 'main' from 'origin'.
```

`-u` 参数让本地 `main` 分支与远程 `origin/main` 建立追踪关系，之后只需 `git push` 即可推送。

---

## 场景二：日常开发 — 提交代码并同步到远程

### 修改代码后的标准提交流程

你在 `src/` 目录下修改了 `app.py`，并新增了一个 `utils.py` 文件。

```bash
$ git status
On branch main
Changes not staged for commit:
  modified:   src/app.py
Untracked files:
  src/utils.py
```

```bash
$ git add src/app.py src/utils.py
$ git commit -m "feat: add utility functions and integrate into app"
[main 2b3c4d5] feat: add utility functions and integrate into app
 2 files changed, 67 insertions(+), 12 deletions(-)
```

```bash
$ git push
Enumerating objects: 8, done.
To github.com:你的用户名/my-project.git
   a1b2c3d..2b3c4d5  main -> main
```

### 如果只想提交部分文件（交互式暂存）

当你修改了多个文件，但希望分批次提交时，可以使用交互模式：

```bash
$ git add -p
```

Git 会逐个文件地展示改动片段，询问你是否将该部分加入暂存区。按 `y` 加入，按 `n` 跳过。

---

## 场景三：拉取远程更新 — 保持本地代码最新

### 别人推送了更新，你需要同步到本地

假设团队成员往 `main` 分支推送了新代码，你在本地执行：

```bash
$ git pull
remote: Enumerating objects: 5, done.
From github.com:你的用户名/my-project
   a1b2c3d..3d4e5f6  main     -> origin/main
Updating a1b2c3d..3d4e5f6
Fast-forward
 README.md | 2 ++
 1 file changed, 2 insertions(+)
```

`git pull` = `git fetch` + `git merge`，它会先下载远程更新，然后自动合并到当前分支。

### 如果拉取时出现冲突怎么办？

当你的本地修改与远程更新修改了同一个文件的同一位置时，Git 会提示冲突：

```
CONFLICT (content): Merge conflict in src/app.py
Automatic merge failed; fix conflicts and then commit the result.
```

此时需要手动编辑冲突文件，Git 会在文件中标记出冲突区域：

```
<<<<<<< HEAD
本地修改的内容
=======
远程拉取的内容
>>>>>>> 3d4e5f6
```

删除 `<<<<<<<`、`=======`、`>>>>>>>` 这些标记，保留你希望使用的最终版本，然后：

```bash
$ git add src/app.py
$ git commit -m "merge: resolve conflict in app.py"
```

---

## 场景四：分支开发 — 在独立线路上工作

### 查看当前有哪些分支

在进行任何分支操作之前，先了解当前仓库的分支状况是一个好习惯：

```bash
$ git branch
* main
```

`*` 号表示当前所在的活跃分支。如果你想查看远程仓库的所有分支，可以加上 `-a` 参数：

```bash
$ git branch -a
* main
  remotes/origin/main
```

### 新建功能分支

当你要开发一个新功能（比如增加用户登录模块），不要在 `main` 上直接改，而是新建一个分支：

```bash
$ git checkout -b feature/user-login
Switched to a new branch 'feature/user-login'
```

这个命令等于 `git branch feature/user-login` + `git checkout feature/user-login`。

### 在分支上提交并推送

在分支上完成若干提交后，将其推送到远程：

```bash
$ git push -u origin feature/user-login
```

### 合并分支到主分支

当功能开发完成并经过测试后，切回 `main`，将分支合并进来：

```bash
$ git checkout main
$ git merge feature/user-login
Updating 3d4e5f6..7e8f9g0
Fast-forward
 src/login.py | 42 ++++++++++++++++++++++++++++++++++
 1 file changed, 42 insertions(+)
```

合并完成后，可以删除不再需要的分支：

```bash
$ git branch -d feature/user-login          # 删除本地分支
$ git push origin --delete feature/user-login   # 删除远程分支
```

---

## 场景五：查看历史与回溯

### 查看提交日志

```bash
$ git log --oneline --graph --all
* 7e8f9g0 (HEAD -> main) feat: add user login module
* 3d4e5f6 (origin/main) docs: update README
* 2b3c4d5 feat: add utility functions
* a1b2c3d first commit: initial project structure
```

### 查看某次提交的详细内容

当你看到某次提交的哈希值（比如 `7e8f9g0`），想查看这次提交具体改了哪些文件的哪些内容时：

```bash
$ git show 7e8f9g0
commit 7e8f9g0 (HEAD -> main)
Author: 你的名字 <your.email@example.com>
Date:   Mon Jul 10 14:30:00 2026 +0800

    feat: add user login module

diff --git a/src/login.py b/src/login.py
new file mode 100644
index 0000000..1234abc
--- /dev/null
+++ b/src/login.py
@@ -0,0 +1,42 @@
+def login(username, password):
+    # 登录逻辑实现
+    return True
```

`git show` 会完整展示该次提交的**元信息**（作者、时间、提交说明）和**具体代码改动**（diff）。如果只想看改动了哪些文件，可以加 `--stat` 参数：

```bash
$ git show 7e8f9g0 --stat
 src/login.py | 42 ++++++++++++++++++++++++++++++++++
 1 file changed, 42 insertions(+)
```

### 切换到某次历史提交

当你想回到过去的某个状态查看代码（而不是当前最新版本）时：

```bash
$ git checkout 3d4e5f6
Note: switching to '3d4e5f6'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.
```

此时你处于 **detached HEAD**（分离头指针）状态，这意味着你不在任何分支上，只是在“查看”那个历史时刻的快照。

- 如果想**只是看看**，看完后切回主分支即可：
  ```bash
  $ git checkout main
  ```

- 如果想**基于这个历史状态继续开发**，可以在当前位置新建分支：
  ```bash
  $ git checkout -b new-branch-from-history
  ```

⚠️ 在 detached HEAD 状态下做的**新提交不会被任何分支引用**，切换回分支后会很难找回来。如果不是临时查看，建议先新建分支再修改。

### 查看仓库目录结构

当你切换到了某次历史提交（或某个分支、标签），想看看当时的仓库里有哪些文件和目录，而不是切换到文件系统里用 `ls` 或 `tree` 命令（因为那些会混入当前工作区未提交的文件），可以使用：

```bash
$ git ls-tree -r 7e8f9g0
100644 blob a1b2c3d    .gitignore
100644 blob 4d5e6f7    README.md
100644 blob 8g9h0i1    mkdocs.yml
100644 blob 5e6f7g8    src/app.py
100644 blob 9i0j1k2    src/utils.py
```

各列含义：
- `100644`：文件权限（普通文件）
- `blob`：表示这是一个文件（`tree` 表示目录）
- `a1b2c3d`：该文件内容的哈希值
- 最后是文件名

加 `-r` 参数会递归展开所有层级

### 撤销最后一次提交（保留修改）

```bash
$ git reset --soft HEAD~1
```

这会撤销上一次提交，但所有改动仍保留在暂存区，你可以重新修改后再提交。

### 彻底回退到某个历史版本（丢弃所有后续修改）

```bash
$ git reset --hard a1b2c3d
HEAD is now at a1b2c3d first commit: initial project structure
```

⚠️ `--hard` 会永久丢弃之后的修改，请谨慎使用。

---

## 场景六：临时保存工作 — 切换分支时不丢失进度

当你在分支上修改到一半，突然需要切换到其他分支修复紧急问题时，可以用 `git stash` 暂存当前未提交的改动：

```bash
$ git stash
Saved working directory and index state WIP on feature/user-login: 7e8f9g0 feat: add user login module
```

切换分支完成工作后，回到原分支恢复暂存的改动：

```bash
$ git stash pop
On branch feature/user-login
Changes to be committed:
  modified:   src/login.py
```

> 💡 **核心原则**：Git 的每一个操作都应该对应一个**具体意图**。在输入命令之前，先想清楚“我现在要解决什么问题”，而不是机械地记忆命令。以上流程覆盖了日常开发 90% 以上的场景，熟练之后自然能触类旁通。