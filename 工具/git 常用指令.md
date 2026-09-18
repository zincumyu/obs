---
tags:
  - 工具
  - git
  - 版本控制
---

# git 小白常用指令速查

> 适用:Windows(Git Bash / PowerShell / VS Code 终端)
> 一句话:git 就是**代码的存档系统**,每次 commit 是一个存档点,可以随时回档、对比、多人协作。

## ⚡ 速查表(最常用,先看这张)

| 想做什么          | 命令                            |
| ------------- | ----------------------------- |
| 下载一个项目        | `git clone 仓库地址`              |
| 看当前状态(哪些改了)   | `git status`                  |
| 把改动加入暂存区      | `git add 文件名` / `git add .`   |
| 存档(提交)        | `git commit -m "说明"`          |
| 上传到远程         | `git push`                    |
| 拉取远程更新        | `git pull`                    |
| 看提交历史         | `git log --oneline`           |
| 看这次改了什么       | `git diff`                    |
| 新建分支          | `git switch -c 名字`            |
| 切换分支          | `git switch 名字`               |
| 合并分支          | `git merge 名字`                |
| 丢弃未提交的改动      | `git restore 文件名`             |
| 把 add 过的撤出暂存区 | `git restore --staged 文件名`    |
| 回退到上一个提交      | `git reset --hard HEAD~1`(慎用) |
| 暂时把改动收起来      | `git stash` / `git stash pop` |

> 详细说明和常见坑见下文各节。

---

## 0. 三个概念(搞懂就入门了)

| 区域 | 是什么 | 类比(写作业) |
| --- | --- | --- |
| 工作区 | 你正在编辑的文件 | 草稿纸 |
| 暂存区 | `add` 之后、`commit` 之前的"购物车" | 把写完的题放进提交袋 |
| 本地仓库 | `commit` 之后的存档 | 装订成册存档 |
| 远程仓库 | GitHub / Gitee / 公司服务器上的仓库 | 交给老师 |

```
工作区 --git add--> 暂存区 --git commit--> 本地仓库 --git push--> 远程仓库
                                                              git pull <--
```

---

## 1. 第一次用:配置身份(只需一次)

```bash
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
git config --global core.autocrlf true   # Windows 换行符,防止 CRLF 警告
```

---

## 2. 拉取与查看

```bash
git clone https://github.com/xxx/repo.git   # 下载整个项目
git status                 # 哪些文件改了/新增了(最常用,随时敲)
git log --oneline          # 一行一条的提交历史
git diff                   # 看具体改了什么(未 add 的)
git remote -v              # 看远程仓库地址
```

---

## 3. 提交三件套(日常 90% 的操作)

```bash
git add .                          # 把所有改动加入暂存区(. = 当前目录全部)
git commit -m "修复了登录bug"      # 存档,-m 后面写改动说明
git push                           # 推到远程
```

> 好习惯:commit 信息写"**做了什么**"(如"修复登录超时""新增导出功能"),不要写"改了一下"。

---

## 4. 拉取更新与冲突

```bash
git pull     # 把远程最新代码拉到本地(先 pull 再 push,避免 push 被拒)
```

**冲突**:两个人改了同一行时,pull/merge 会报 conflict。打开冲突文件,会看到:

```
<<<<<<< HEAD
你改的内容
=======
别人改的内容
>>>>>>> 远程分支名
```

手工保留正确的内容,删掉 `<<<<<<<` `=======` `>>>>>>>` 三行标记,然后 `add` + `commit` 即可。

---

## 5. 分支(做新功能 / 多人协作)

```bash
git branch feature         # 新建分支(不切换)
git switch feature         # 切换到该分支
git switch -c feature      # 新建并切换(一步到位,最常用)
git branch                 # 看所有分支
git merge feature          # 把 feature 合并进当前分支
git branch -d feature      # 删掉已合并的分支
git push -u origin 你的分支名 # 传分支
```

> 为什么要分支:主分支(main/master)始终保持"能跑",新功能在分支上开发,做好再合并回来。

---

## 6. 后悔药

```bash
git restore 文件名               # 丢弃工作区改动(回到上次 add/commit 的状态)
git restore --staged 文件名       # 把 add 过的撤出暂存区(重新变成"未暂存")
git reset --hard HEAD~1          # 回退到上一个提交(改动全丢!慎用)
git revert <commit号>            # 生成一个"反向提交"来撤销,不丢历史(更安全)
git stash                        # 把没提交的改动先收起来(切分支前常用)
git stash pop                    # 取回收起来的改动
```

---

## 7. .gitignore:有些文件不用提交

在仓库根目录建 `.gitignore` 文件,一行一个:

```
__pycache__/
*.log
.env
node_modules/
```

常见忽略对象:编译产物、日志、密钥(如 `.env`)、依赖目录。

---

## 8. 常见坑

1. **push 被拒**(提示先 pull)→ 先 `git pull` 再 `git push`。
2. **提交完发现漏改一个文件** → 改完再 `add` + `commit`(或 `git commit --amend` 合并进上一次)。
3. **切分支前有没提交的改动** → 先 `commit` 或 `stash`,否则改动会被带过去。
4. **merge 冲突** → 按第 4 节手工解决。
5. **在错误目录执行 git** → 先 `cd` 到仓库根目录。
6. **误用 `reset --hard`** → 别慌,`git reflog` 能找到之前所有 commit 号,再 `git reset --hard <commit号>` 救回来。
7. **Windows 换行符警告(CRLF)** → 配置 `core.autocrlf true`(见第 1 节)。

---

## 9. 新手日常节奏(照着做)

```bash
git pull                # 开工前先同步
# ……写代码……
git status              # 看看自己改了什么
git add .
git commit -m "说明"
git push                # 下班前推上去
```

---

## 相关笔记

- [[硬技能（必须会）&&  指令技巧]]
- [[conda 常用指令]]
- [[计算机任务清单]]
## 添加 `.gitignore` 的几种方法

### 方法1：手动创建（最常用）

```bash
# 在项目根目录创建 .gitignore 文件
touch .gitignore
```
### 方法2：直接 echo 写入

```bash
# 创建并写入多条规则
cat > .gitignore << EOF
# 编译文件
*.o
*.exe
*.out

# IDE 文件
.vscode/
.idea/

# 日志
*.log

# 系统文件
.DS_Store
Thumbs.db

# 依赖
node_modules/
vendor/
EOF
```

### . 查看哪些文件被忽略

```bash
git status --ignored
```

# 写入常用规则
```cmd
echo "*.log" >> .gitignore
echo ".DS_Store" >> .gitignore
echo "build/" >> .gitignore
```
# 创建并提交
### ==不能遗忘!!!==

```
echo "*.log" > .gitignore
git add .gitignore
git commit -m "添加 .gitignore"
git push
```
