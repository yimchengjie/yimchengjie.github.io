---
title: Git简单教程
date: 2018-03-31 08:33
categories:
  - 项目工具
tags:
  - 版本控制
  - Git
toc: true
---

### 什么是 Git

Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。

### Git 在 Windows 的安装和配置

[安装包下载地址](https://gitforwindows.org/)

完成安装之后，就可以使用命令行的 git 工具（已经自带了 ssh 客户端）了，另外还有一个图形界面的 Git 项目管理工具。

在开始菜单里找到"Git"->"Git Bash"，会弹出 Git 命令窗口，你可以在该窗口进行 Git 操作。

#### 配置用户信息

配置个人的用户名称和电子邮件地址：

```shell
git config --g user.name "yanchengjie"
git config --g user.email ycj996425271@live.com
```

#### 查看配置信息

要检查已有的配置信息，可以使用 git config --list 命令：

```shell
git config --list
```

### Git 的工作流程

Git 工作的一般流程为:

1. 克隆 Git 资源作为工作目录。
2. 在克隆的资源上添加或修改文件。
3. 如果其他人修改了，你可以更新资源。
4. 在提交前查看修改。
5. 提交修改。
6. 在修改完成后，如果发现错误，可以撤回提交并再次修改并提交。

### Git 工作区、暂存区和版本库

**工作区**：就是你在电脑里能看到的目录。
**暂存区**：英文叫 stage, 或 index。一般存放在 ".git 目录下" 下的 index 文件（.git/index）中，所以我们把暂存区有时也叫作索引（index）。
**版本库**：工作区有一个隐藏目录.git，这个不算工作区，而是 Git 的版本库。

### Git 创建仓库

Git 使用 `git init` 命令来初始化一个 Git 仓库，Git 的很多命令都需要在 Git 的仓库中运行，所以 `git init` 是使用 Git 的第一个命令。

### Git 克隆仓库

我们使用 git clone 从现有 Git 仓库中拷贝项目

```shell
git clone git://github.com/XXXX/XXX
```

### Git 添加文件到缓存

使用命令 `git add <fileName>`

### Git 移除缓存中的文件

使用命令 `git rm <fileName>`

### Git 将缓存区添加到仓库

使用命令`git commit -m "备注信息"`

### Git 分支管理

列出分支命令:`git branch`(master 为主分支)
创建分支命令:`git branch (branchName)`
切换分支命令:`git checkout (branchNaem)`
删除分支命令:`git branch -d (branchName)`
合并分支命令:`git merge (branchName)`

### Git 查看提交历史

使用命令 `git log`
