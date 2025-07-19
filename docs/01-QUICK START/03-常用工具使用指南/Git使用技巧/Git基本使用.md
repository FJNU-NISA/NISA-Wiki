# Git基本使用

## 安装 Git

### Windows 上安装

[CSDN上的详细教程](https://blog.csdn.net/mukes/article/details/115693833)

## 配置 Git 

### 设置邮箱和用户名

``` bash
git config --global user.name 用户名
git config --global user.email 邮箱
```

 其中 `--global` 代表全局配置, 如果不设置, 只对当前所处仓库生效

> 如果想把仓库同步到 Github 上后, 可以正确显示自己账号和头像, 需要正确设置与 Github 相同的用户名和邮箱
> 
> 如果直接使用自己的邮箱, 在同步仓库时, Github 会发出警告, 这个时候需要把邮箱改为 ******@users.noreply.github.com 的邮箱, 可以前往 [https://github.com/settings/emails](https://github.com/settings/emails), 查看自己对应的地址, 以起到保护隐私的作用
> 
> ![github_email.png](./assets/github_email.png)

## Git 常见使用命令

### 初始化仓库

在项目目录下初始化一个新的 Git 仓库：

```bash
git init
```

### 添加文件进暂存区

将文件添加到暂存区（staging area）：

```bash
# 添加指定文件
git add 文件名

# 添加所有修改的文件
git add .

# 添加所有文件（包括新文件和已删除的文件）
git add -A
```

### 提交暂存区文件

将暂存区的文件提交到本地仓库：

```bash
# 提交并添加提交信息
git commit -m "提交信息"

# 添加并提交（跳过暂存区）
git commit -am "提交信息"
```

### 从远程仓库拉取

从远程仓库获取最新代码：

```bash
# 拉取并合并远程分支
git pull

# 指定远程仓库和分支
git pull origin main

# 获取远程更新但不合并
git fetch
```

### 推送至远程仓库

将本地提交推送到远程仓库：

```bash
# 推送到默认远程分支
git push

# 指定远程仓库和分支
git push origin main

# 首次推送并设置上游分支
git push -u origin main
```

### 添加远程仓库

关联本地仓库与远程仓库：

```bash
# 添加远程仓库
git remote add origin 远程仓库地址

# 查看远程仓库
git remote -v

# 修改远程仓库地址
git remote set-url origin 新的远程仓库地址
```

## 其他常用命令

### 查看状态和历史

```bash
# 查看仓库状态
git status

# 查看提交历史
git log

# 查看简洁的提交历史
git log --oneline

# 查看文件差异
git diff
```

### 分支操作

```bash
# 查看所有分支
git branch -a

# 创建新分支
git branch 分支名

# 切换分支
git checkout 分支名

# 创建并切换到新分支
git checkout -b 分支名

# 合并分支
git merge 分支名

# 删除分支
git branch -d 分支名
```

### 撤销操作

```bash
# 撤销工作区的修改
git checkout -- 文件名

# 撤销暂存区的修改
git reset HEAD 文件名

# 撤销最近一次提交（保留修改）
git reset --soft HEAD^

# 撤销最近一次提交（不保留修改）
git reset --hard HEAD^
```
