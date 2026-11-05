---
tags:
  - Docker
  - 索引
created: 2026-11-06
---

# Docker 笔记（基础版）

> [!info] 参考
> 本系列笔记整理自开源书 **《Docker 从入门到实践》**：
> <https://yeasy.gitbook.io/docker_practice>
> 源码仓库：<https://github.com/yeasy/docker_practice>
> 这里只保留"够用"的部分，细节（每个参数的完整语义、发行版差异）以原书为准。

> [!abstract] 一句话
> Docker 把「应用 + 依赖」打包成**镜像**，用**容器**在隔离环境里跑起来。镜像可分发、容器秒级启停，专治"在我机器上是好的"。

## 这套笔记怎么读

```mermaid
flowchart LR
    A[01 简介] --> B[02 基本概念]
    B --> C[03 安装配置]
    C --> D[04 镜像]
    D --> E[05 容器]
    E --> F[06 Dockerfile]
    F --> G[07 数据与网络]
    G --> H[08 Compose]
    H --> I[09 进阶与生态]
    I --> J[10 镜像速查与排错]
```

## 目录

| 笔记 | 内容 | 关键命令 |
| --- | --- | --- |
| [[01-Docker简介]] | 是什么、为什么、和虚拟机的区别 | — |
| [[02-基本概念]] | 镜像 / 容器 / 仓库、分层、生命周期 | — |
| [[03-安装与配置]] | Ubuntu APT 安装、免 sudo、镜像加速 | `docker info` |
| [[04-镜像管理]] | 拉取、查看、删除、导入导出、commit | `docker pull` / `save` |
| [[05-容器操作]] | 启动、日志、进容器、复制、清理 | `docker run` / `exec` |
| [[06-Dockerfile]] | 指令详解、多阶段构建、最佳实践 | `docker build` |
| [[07-数据与网络]] | 数据卷、绑定挂载、网络类型、端口映射 | `-v` / `--network` |
| [[08-Docker-Compose]] | 多容器编排、compose.yaml、常用命令 | `docker compose up` |
| [[09-进阶与生态]] | Buildx、底层原理、K8s、安全、GPU | `docker buildx` |
| [[10-镜像速查与排错]] | 常用镜像启动命令、错误对照表 | `docker system df` |

## 三个核心对象

| 对象 | 类比 | 说明 |
| --- | --- | --- |
| 镜像 Image | 类 / 安装包 | 只读模板，由多层文件系统叠加而成 |
| 容器 Container | 实例 / 进程 | 镜像的运行态，多一层可写层 |
| 仓库 Registry | 应用商店 | 存放镜像的地方，如 Docker Hub |

## 五分钟上手

```bash
# 跑一个 nginx，把容器 80 端口映射到宿主机 8080
docker run -d --name web -p 8080:80 nginx

# 看容器状态
docker ps

# 浏览器访问 http://localhost:8080 应看到欢迎页

# 看日志、进容器、收尾
docker logs -f web
docker exec -it web bash
docker rm -f web
```

> [!tip] 学习建议
> 命令不用背。记住「**镜像 → 容器 → 网络/数据 → 编排**」这条主线，遇到不会的参数就 `docker <命令> --help`，比翻书快。

> [!warning] 国内网络
> Docker Hub 直连经常超时，先配镜像加速器，见 [[03-安装与配置]]。
