---
title: "K8S笔记8 利用K8S部署贝亲婴童商城"
date: 2021-01-15T09:47:00+08:00
lastmod: 2021-01-15T09:47:00+08:00
draft: false
keywords: ["K8S","贝亲商城"]
description: "desc"
tags: ["K8S"]
categories: ["K8S"]
author: "钱文军"
---

 经过前几篇的学习，已经基本了解了如何部署一个应用、如何创建文件共享、如何通过服务实现负载均衡以及如何通过`Renetd`实现服务对外暴露。接下来学习实战，如何部署一个通过 `Spring Boot` 开发的商城web应用——贝亲商城。

# 部署拓扑图

![image-20210115201331863](http://cdn1.jalen-qian.com/20210115201332odau4FI8RN.png)

通过这个拓扑图，我们可知：

- master节点上的文件共享区有数据库脚本，用来初始化`MySql`数据库，并部署在一个pod上
- 通过`beiqin-db-service`这个服务，在集群上暴露`MySql`服务
- 通过`openjdk:8u222`这个基础镜像，部署web服务（使用SpringBoot开发），容器对外暴露端口是80
- 通过`beiqin-app-service`这个服务，对外暴露贝亲商城应用的端口，也是80
- 采用`Renetd`实现宿主机IP和`beiqin-app-service`服务虚拟IP的映射，最终能够对集群外网提供服务

# 部署贝亲商城

## 部署资源

我们将部署脚本还有服务jar包、mysql脚本文件等，放到`/usr/local/data/www-data/beiqin/`目录下。

其中，`/usr/local/data/www-data/`是前文我们配置好的文件共享区。

以下是部署贝亲商城应用所需的所有资源文件，下面将一一创建并部署

```shell
[root@master www-data]# tree beiqin
beiqin
├── beiqin-app-deploy.yml   # web应用部署脚本文件
├── beiqin-app-service.yml  # web应用服务部署脚本文件
├── beiqin-db-deploy.yml    # mysql部署脚本文件
├── beiqin-db-service.yml   # mysql服务部署脚本文件
├── dist
│   ├── application.yml     # 应用程序配置文件
│   └── beiqin-app.jar      # spring-boot开发的应用程序jar包，可以在openjdk容器中直接运行
└── sql
    └── beiqin.sql          # mysql脚本文件
```

## 部署贝亲商城

### 1. master创建NFS文件共享，并挂载到Node节点

这里我们简单沿用 []()