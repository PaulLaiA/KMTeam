---
界: DataBase
门: NoSQL
纲: Redis
tags: ["#Docker","#Note"]
aliases:
  - 
date: 2021-09-07
---

# 基础概念


## 官方

[Empowering App Development for Developers | Docker](https://www.docker.com)

[Docker Documentation](https://docs.docker.com/)

[GitHub - docker/docker-ce: Docker CE](https://github.com/docker/docker-ce)

---

## DevOps

1.  快速部署上线
2.  打包镜像文件,一键测试+发布
3.  便捷的升级+扩缩容
4.  容器化可以使得开发环境与测试环境高度一致
5.  Docker身为内核级别的虚拟化,可以在一台物理机上运行多台容器实例,具备更高效的计算资源利用

---

## Docker 简介

### 什么是Docker

通常描述的 `Docker` 是指 `Docker Engine`

Docker Engine 是一个客户端 - 服务器的应用程式

Docker是一种轻量级的虚拟化方式

由Docker守护进程 , 一个REST API指定与守护进程交互的接口和一个命令行接口 ( CLI ) 与守护进程通信(通过封装REST API)

Docker Engine 从 CLI 中接受 Docker 命令

### Docker 基本组成

-   Docker 主机 [ Host ]
    -   安装Docker程序的机器
-   Docker 仓库 [ Registry ]
    -   保存打包好的软件镜像
    -   仓库分为公有仓库和私有仓库
-   Docker 镜像 [ Images ]
    -   软件打包好后成为镜像
-   Docker 容器 [ Container ]
    -   镜像启动后的实例将成为一个容器
    -   容器是独立运行的一个 或 一组 应用

### Docker 与 操作系统 的区别

传统的虚拟机方式提供的是相对封闭的隔离。Docker利用Linux系统上的多种防护技术实现了严格的隔离可靠性，并且可以整合众多安全工具。从 1.3.0版本开始，docker重点改善了容器的安全控制和镜像的安全机制， 极大提高了使用docker的安全性。

[区别](https://www.notion.so/57f29bf6b5f541d8b9091f70d95bdd0b)