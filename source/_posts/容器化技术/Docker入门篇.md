---
title: Docker入门篇
categories:
  - 容器化技术
tags:
  - Docker
  - 容器
toc: true
date: 2020-02-03 01:30:49
---

## Docker入门篇

-----------

Docker是世界领先的软件容器平台,利用Docker可以消除协作编码时"在我的机器上可以正常工作"的问题

Docker将一整套环境打包封装成镜像,无需重复配置环境,解决了环境带来的种种问题. Docker容器间是进程隔离的,互不影响

### Docker术语

1. 镜像: 系统的环境整个打包
2. 容器: 镜像启动后的实例
3. 仓库: 专门存放镜像的地方

### 第一个Docker

[官方链接](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

安装环境: Ubuntu 16.04

1. 更新包索引

    ```shell
    sudo apt-get update
    ```

2. 安装软件包,允许在HTTPS上使用存储库
  
    ```shell
    sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
    ```

3. 添加Docker的官方GPC秘钥

    ```shell
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

    通过查看指纹最后8位,以验证是否拥有带指纹的秘钥

    ```shell
    sudo apt-key fingerprint 0EBFCD88
    ```

    ![检验秘钥](/检验秘钥.png)

4. 设置稳定存储库

    ```shell
    sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
    ```

5. 安装Docker引擎

    ```shell
    <!-- 先更新一遍索引包 -->
    sudo apt-get update

    <!-- 安装最新版本Docker引擎,Docker社区,Docker容器 -->
    sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```

6. 查看Docker版本

    ```shell
    sudo docker version
    ```

    ![Docker版本](Docker版本.png)
  
7. 运行HelloWorld镜像

    ```shell
    sudo docker run hello-world
    ```

    ![DockerHelloWorld](DockerHelloWorld.png)

    运行`docker run hello-world`的过程

    1. 本地找hello-world
    2. 没有,去DockerHub拉取一份hello-world镜像,并运行

### 配置镜像地址

Docker默认的镜像地址DockerHub在国外,访问慢,这里添加阿里云的加速

![配置加速](配置加速.png)
