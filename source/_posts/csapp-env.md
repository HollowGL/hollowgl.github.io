---
title: docker配置csapp环境
date: 2023-09-19 21:33:47
updated: 2023-09-19 21:33:47
cover:
thumbnail:
categories: csapp
tags: C
excerpt: 描述了如何使用docker在mac上配置csapp实验环境
---

## 前言

仰慕CSAPP之名久矣，于上学期向图书馆借阅数周，阅之心情舒畅，甚为过瘾，不负盛誉也。然及暑假，嫌其书重，未能携归，忽感空虚万分，孤寂难堪。后新学期至，即以重金购得。

纸上得来终觉浅薄，觉知精髓在实践。数日前始配置实验环境，然其过程艰难，折磨数日，迨至今日，终得成功。念及同经配环境之苦者，遂以此文以记之。

## 概述

CSAPP的实验的部分步骤需要在Linux系统上运行，网上有使用docker配好的镜像，但我尝试运行却遇到很多问题。恰好之前稍微学了点docker，于是干脆自行配置环境。重点参考了网上[这一教程](https://zhuanlan.zhihu.com/p/339047608)。

本地计算机的环境是 `MacBook Air (M1, 2020)`配置、 `macOS Ventura`系统，使用 `iTerm2`终端，以下配置过程均在此运行。我还有一台 Win10 系统电脑，后续可能会按此过程重新配置一遍已验证其跨平台性。


## 环境配置

**下载 Docker Desktop 并换源**，在设置-docker引擎中将 `registry-mirrors` 换成以下内容并重启docker：

```json
"registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn/"
  ]
```
> 我一开始看到这是json格式，以为可以添加多个源，但是后来发现这样做反而没法正常运行，会在 `docker build` 时遇到EOF的报错。
> 不过我现在仍然没有找到出现问题的具体原因，按理说配置多个源应该是被允许的。



**拉取镜像**

```bash
docker pull ubuntu:22.04
```



**编写Dockerfile**

```dockerfile
FROM --platform=linux/amd64 ubuntu:22.04
```
> 如果不使用 `--platform=linux/amd64` 参数，那么后续在运行二进制文件时会报错 `qemu-x86_64: Could not open '/lib64/ld-linux-x86-64.so.2': No such file or directory` 。我是在[stackoverflow](https://stackoverflow.com/questions/71040681/qemu-x86-64-could-not-open-lib64-ld-linux-x86-64-so-2-no-such-file-or-direc)找到了这一解决方案，这也是我为什么要自行编写Dockerfile而不是直接使用官方ubuntu镜像的原因。



**生成镜像**

```bash
docker build -t myubuntu .
```





**建立目录挂载**，`~/Desktop/CSAPP-Labs` 是保存Lab文件的本地位置，挂载后两边的文件将完全同步。

```bash
docker run -it -v ~/Desktop/CSAPP-Labs:/csapp --name=csapp_env myubuntu /bin/bash
```
执行该命令后终端就进入ubuntu系统，可直接运行相关命令


**备份并换apt源**

```bash
cp /etc/apt/sources.list /etc/apt/sources.list.bak
```

接着apt换[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

 - 我使用的是m1芯片的mac，但使用[ubuntu-ports镜像](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu-ports/)却无效，使用普通镜像即可

 - 所有选项都选**否**，Ubuntu版本选**22.04 LTS**

以上选项经过我多次尝试，具体内容如下：

```bash 内容较长，点击展开 >folded
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse

deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
# deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse

# deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse
# # deb-src http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse

# 预发布软件源，不建议启用
# deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
# # deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-proposed main restricted universe multiverse
```

该ubuntu系统中没有文本编辑器（apt 没换源升级之前又没法 `apt install vim` ），我找到了两种方法来更改 `/etc/apt/sources.list`

- 使用本地计算机新建一个文件写入上述文本，将其放入挂载目录 `~/Desktop/CSAPP-Labs` ，那么它将出现在ubuntu系统的 `csapp` 文件中，接下来 `cp /csapp/sources.list /etc/apt/sources.list`
- 可以通过Docker Desktop直接进行编辑，找到容器进入Files直接去对应目录编辑即可



换源后需更新，并重新升级apt

```bash
source /etc/apt/source.list
apt update
```


**安装编译工具链**

```bash
apt install build-essential
```
> 验证安装：`gcc --version`
> 要不是没有编辑器，不然必须给它运行个 `Hello World!`




此时还缺少一些文件，虽然能正常运行 `./dlc bits.c` ，但是会在 `make btest` 时报错 `fatal error: bits/libc-header-start.h: No such file or directory` 
> 运行dlc可能会被报错缺少权限，需 `chmod u+x ./dlc`


还需下载以下工具

```bash
apt-get install gcc-multilib
```
> 我依然是在[stackoverflow](https://stackoverflow.com/questions/54082459/fatal-error-bits-libc-header-start-h-no-such-file-or-directory-while-compili)找到的解决方案
> 原因好像是缺少32位系统的头文件和库文件



最终，make命令成功运行，编译出btest二进制文件，可以直接运行





**退出容器**

```bash
exit
```



**下次启动容器**
需要先打开docker

```bash
docker start csapp_env
docker exec -it csapp_env /bin/bash
```


### 注意事项

1. Docker运行的ubuntu好像直接拥有管理员权限，所有命令直接运行即可，不加 `sudo`
2. 在iterm2终端执行ubuntu命令时，直接 `clear` 就被警告，需使用快捷键 `control` + `L` 
3. 查看系统版本：`cat /etc/os-release`
4. 有时候退出容器，容器并不会自动停止，需使用命令 `docker stop `


## 后记

***[StackOverflow](https://stackoverflow.com/questions)为什么是神！***