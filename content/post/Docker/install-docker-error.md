---
title: "【docker】Centos8.0安装docker报错：Problem package docker-ce-3..."
date: 2020-09-25
lastmod: 2020-09-25
draft: false
tags: ["Docker","Centos","yum"]
categories: ["Docker"]
author: "Jalen"
---

### 事情经过
> 在[Centos8.0下用yum安装Docker](/post/docker/yum_install_docker_repo/)，执行命令后报错，内容大概是需要containerd.io的版本 >= 1.2.2-3

执行的命令如下:

```
sudo yum install docker-ce docker-ce-cli containerd.io
```
详细报错信息
```
Last metadata expiration check: 0:01:32 ago on Sat 26 Sep 2020 12:10:38 AM CST.
Error: 
 Problem: package docker-ce-3:19.03.13-3.el7.x86_64 requires containerd.io >= 1.2.2-3, but none of the providers can be installed
  - cannot install the best candidate for the job
  - package containerd.io-1.2.10-3.2.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.13-3.1.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.13-3.2.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.2-3.3.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.2-3.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.4-3.1.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.5-3.1.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.2.6-3.3.el7.x86_64 is filtered out by modular filtering
  - package containerd.io-1.3.7-3.1.el7.x86_64 is filtered out by modular filtering
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)
```

#### 解决办法

1. 降低安装的docker版本
2. 手动安装更高版本的`containerd.io`

对于第1种方法，[这里](https://www.runoob.com/docker/centos-docker-install.html)有比较详细的介绍，我们主要介绍第2种办法

**下载`containerd.io`的rpm包，安装新版containerd.io**
```
## 如果你的系统没有安装wget，先安装
$sudo yum install -y wget

## Centos7执行下面的命令 
$wget https://download.docker.com/linux/centos/7/x86_64/edge/Packages/containerd.io-1.3.7-3.1.el7.x86_64.rpm

## Centos8执行下面的命令
$wget https://download.docker.com/linux/centos/8/x86_64/edge/Packages/containerd.io-1.3.7-3.1.el8.x86_64.rpm

## 执行完后，当前目录下会有一个containerd.io-1.3.7-XXX.rpm文件
## 使用这个rpm安装
$sudo yum install -y containerd.io-1.3.7-3.1.el8.x86_64.rpm

## 安装完后再安装docker
$sudo yum install docker-ce docker-ce-cli containerd.io

```

#### 检查docker是否安装成功
```
#启动docker

$systemctl start docker

#检查docker版本
$docker -v
```