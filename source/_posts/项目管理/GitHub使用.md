---
title: GitHub使用
date: 2018-04-03 17:33
categories:
  - 项目工具
tags:
  - 版本控制
  - 代码托管
  - Git
toc: true
---
### GitHub

#### 简介

GitHub可以用来托管代码,也可以用来进行版本控制,它相比与其他版本控制工具的优势在于,可以非常方便的进行分支操作

#### 1.注册账户

#### 2.新建一个仓库(repository)

![新建仓库](newRepo.png)

#### 3.配置GitHub公钥

在Git终端输入`ssh-keygen -t rsa -C "email@email.com"`
在系统用户下的`/.ssh`目录,找到`id_rsa.pub`文件,其中存放的就是秘钥.

再进入github个人设置,选择SSH and GPG keys,将`id_rsa.pub`文件中的内容复制进去
如:
![设置秘钥](setSSHKey.png)

#### 4. clone仓库

使用 `git clone <项目链接>`
将github仓库克隆到本地

#### 5. 将本地项目上传到新建仓库

  1. git init
    初始化项目
  2. git add .
    添加文件索引
  3. git commit -m "msg"
    提交到暂存区
  4. git remote add origin git@XXXXXX
    添加为github仓库源点
  5. git push -u origin master
    将源点作为主干推送到github
  6. 第5步上传被拒绝时
     git pull origin master --allow-unrelated-histories
     再重复回到第2步
