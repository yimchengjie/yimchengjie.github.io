---
title: Docker镜像，容器，仓库
categories:
  - 容器化技术
tags:
  - Docker
  - 容器
toc: true
date: 2020-02-03 01:30:49
---
## Docker镜像
### 镜像列表
docker images可以列出本地主机的镜像
```shell
root@ycjubuntu:/home/yanchengjie# docker images
REPOSITORY             TAG       IMAGE ID       CREATED         SIZE
codercom/code-server   latest    dc6f07d1c0f8   15 months ago   1.63GB
```
+ repository: 镜像的仓库源
+ TAG: 镜像的标签，代表这这个镜像的版本号
+ IMAGE ID: 镜像ID
+ CREATED: 镜像创建时间
+ SIZE: 镜像大小
### 查找镜像
可以再镜像仓库docker hub中查找镜像`https://hub.docker.com/search?q=mysql&type=image`
![[Pasted image 20230418152332.png]]
也可以使用命令查找docker search
```shell
root@ycjubuntu:/home/yanchengjie# docker search mysql
NAME                            DESCRIPTION                                      STARS     OFFICIAL   AUTOMATED
mysql                           MySQL is a widely used, open-source relation…   14047     [OK]      s 
mariadb                         MariaDB Server is a high performing open sou…   5358      [OK]       
percona                         Percona Server is a fork of the MySQL relati…   603       [OK]       
phpmyadmin                      phpMyAdmin - A web interface for MySQL and M…   781       [OK]       
circleci/mysql                  MySQL is a widely used, open-source relation…   29                   
bitnami/mysql                   Bitnami MySQL Docker Image                       83                   [OK]
```
### 拉取镜像
```shell
root@ycjubuntu:/home/yanchengjie# docker pull mysql
Using default tag: latest
latest: Pulling from library/mysql
72a69066d2fe: Pull complete 
93619dbc5b36: Pull complete 
99da31dd6142: Pull complete 
626033c43d70: Pull complete 
37d5d7efb64e: Pull complete 
ac563158d721: Pull complete 
d2ba16033dad: Pull complete 
688ba7d5c01a: Pull complete 
00e060b6d11d: Pull complete 
1c04857f594f: Pull complete 
4d7cfa90e6ea: Pull complete 
e0431212d27d: Pull complete 
Digest: sha256:e9027fe4d91c0153429607251656806cc784e914937271037f7738bd5b8e7709
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest
```
### 删除镜像
如果有容器需要先删除容器
```shell
root@ycjubuntu:/home/yanchengjie# docker rmi codercom/code-server
Error response from daemon: conflict: unable to remove repository reference "codercom/code-server" (must force) - container dc7ded6807bf is using its referenced image dc6f07d1c0f8
root@ycjubuntu:/home/yanchengjie# docker rm dc7ded6807bf
dc7ded6807bf
```
再删除镜像
```shell
root@ycjubuntu:/home/yanchengjie# docker rmi codercom/code-server
Untagged: codercom/code-server:latest
Untagged: codercom/code-server@sha256:e73d681aae4fdc76197bac643289378823ee53fc029c511ec55313db20d92598
Deleted: sha256:dc6f07d1c0f8d405ca9cfc560013d177d5e55ca52cb9a95eca86edf7f0f29e26
Deleted: sha256:ca64ba22be886a7b0a47db23fb4ff6a6c13e120120fc13582cf3d64ced57d3c5
Deleted: sha256:040d3b75e1eb18417bdcd7e78cf735bc68edaf4c9df2cebac266644b40541fd3
Deleted: sha256:82decfa255dba1f017221fd77a1e64a994ff71942c5c31b2475b0bf0d695b188
Deleted: sha256:a2682de32758c255837df20753ee5d11ac6ea50618dd55bdc316f440d7f68f9d
Deleted: sha256:ced4742baa852e79f0dfdb84c865fe20d3f44c5def0b33854d6c34bc287bfaf2
Deleted: sha256:5bbc0672efa0a10612b61446a18612f35f7781918165fe6aa822a94416af5e1c
Deleted: sha256:929fea9d60f4adbc6a93684165ef376c40d03a69f090a51514adff2121d47215
Deleted: sha256:11936051f93baf5a4fb090a8fa0999309b8173556f7826598e235e8a82127bce
```
### 更新镜像
先启动一个基础容器ubuntu
```shell
root@ycjubuntu:/home/yanchengjie# docker run -it ubuntu
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
7b1a6ab2e44d: Pull complete 
Digest: sha256:626ffe58f6e7566e00254b638eb7e0f3b11d4da9675088f4781a50ae288f3322
Status: Downloaded newer image for ubuntu:latest
root@59fd729d25be:/# 
```
已经进入容器， 在容器内进行调整
```shell
root@59fd729d25be:/# apt-get update
Get:1 http://archive.ubuntu.com/ubuntu focal InRelease [265 kB]
Get:2 http://security.ubuntu.com/ubuntu focal-security InRelease [114 kB]
Get:3 http://security.ubuntu.com/ubuntu focal-security/multiverse amd64 Packages [28.5 kB]
Get:4 http://archive.ubuntu.com/ubuntu focal-updates InRelease [114 kB]    
Get:5 http://security.ubuntu.com/ubuntu focal-security/universe amd64 Packages [1029 kB]
Get:6 http://archive.ubuntu.com/ubuntu focal-backports InRelease [108 kB]
Get:7 http://archive.ubuntu.com/ubuntu focal/restricted amd64 Packages [33.4 kB]
Get:8 http://archive.ubuntu.com/ubuntu focal/main amd64 Packages [1275 kB]
Get:9 http://security.ubuntu.com/ubuntu focal-security/restricted amd64 Packages [2065 kB]
Get:10 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages [2593 kB]
Get:11 http://archive.ubuntu.com/ubuntu focal/universe amd64 Packages [11.3 MB]
Get:12 http://archive.ubuntu.com/ubuntu focal/multiverse amd64 Packages [177 kB]
Get:13 http://archive.ubuntu.com/ubuntu focal-updates/restricted amd64 Packages [2203 kB]
Get:14 http://archive.ubuntu.com/ubuntu focal-updates/universe amd64 Packages [1325 kB]
Get:15 http://archive.ubuntu.com/ubuntu focal-updates/main amd64 Packages [3075 kB]
Get:16 http://archive.ubuntu.com/ubuntu focal-updates/multiverse amd64 Packages [31.2 kB]
Get:17 http://archive.ubuntu.com/ubuntu focal-backports/universe amd64 Packages [28.6 kB]
Get:18 http://archive.ubuntu.com/ubuntu focal-backports/main amd64 Packages [55.2 kB]
Fetched 25.9 MB in 13s (1986 kB/s)                                         
Reading package lists... Done
root@59fd729d25be:/# exit
exit
root@ycjubuntu:/home/yanchengjie# 
```
