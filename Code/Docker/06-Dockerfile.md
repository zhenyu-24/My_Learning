---
tags:
  - Docker
  - Dockerfile
  - 构建
created: 2026-11-06
---

# 06 Dockerfile

> [!info] 参考
> <https://yeasy.gitbook.io/docker_practice/07_dockerfile>
> 最佳实践：<https://yeasy.gitbook.io/docker_practice/appendix/best_practices>

## 6.1 构建基本流程

```bash
docker build -t myapp:v1 .
docker build -t myapp:v1 -f docker/Dockerfile.prod .
docker build --no-cache -t myapp:v1 .
docker build --build-arg VERSION=1.2 -t myapp:v1 .
```

末尾的 `.` 是**构建上下文**，Docker 会把这个目录打包发给 daemon。上下文越大，构建越慢——这是新手最常见的性能坑。

```dockerignore
.git
node_modules
*.log
dist
```

## 6.2 指令速查

| 指令 | 作用 | 备注 |
| --- | --- | --- |
| `FROM` | 指定基础镜像 | 必须是第一条（`ARG` 除外） |
| `RUN` | 构建时执行命令 | 每次尽量合并，减少层数 |
| `COPY` | 复制本地文件 | 推荐，纯粹 |
| `ADD` | 复制 + 自动解压 tar / 支持 URL | 除解压外优先用 `COPY` |
| `CMD` | 默认启动命令 | 可被 `docker run` 参数覆盖 |
| `ENTRYPOINT` | 入口命令 | 参数会被追加到它后面 |
| `ENV` | 环境变量 | 构建期 + 运行期都有效 |
| `ARG` | 构建参数 | 只在构建期有效 |
| `VOLUME` | 声明匿名卷 | 数据持久化 |
| `EXPOSE` | 声明端口 | **仅文档作用**，不实际映射 |
| `WORKDIR` | 工作目录 | 不存在会自动创建 |
| `USER` | 运行用户 | 生产镜像应避免 root |
| `HEALTHCHECK` | 健康检查 | 配合编排使用 |
| `LABEL` | 元数据 | 作者、版本、仓库地址 |
| `SHELL` | 改默认 shell | 如 `SHELL ["/bin/bash", "-c"]` |

## 6.3 CMD 与 ENTRYPOINT

这是最容易混的一对：

| | `CMD` | `ENTRYPOINT` |
| --- | --- | --- |
| 作用 | 提供默认命令/参数 | 提供固定入口 |
| 被 `docker run` 覆盖 | 整体覆盖 | 不覆盖，只追加参数 |
| 适合 | 可变默认行为 | 明确"这个镜像就是跑这个程序" |

```dockerfile
# 形式一：exec 形式（推荐，PID 1 直接是进程，能收到信号）
CMD ["nginx", "-g", "daemon off;"]

# 形式二：shell 形式（会被包成 /bin/sh -c，信号可能收不到）
CMD nginx -g "daemon off;"
```

```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8000"]
```

此时 `docker run myapp --port 9000` 实际执行 `python app.py --port 9000`。

> [!warning] 为什么容器里 PID 1 很重要
> shell 形式会让 `/bin/sh` 成为 PID 1，它**不转发信号**，`docker stop` 只能等超时后强杀。Java / Python 服务尤其容易出现"停容器要等 10 秒"。用 exec 形式，或加 `--init`。

## 6.4 常见写法对照

```dockerfile
# 不推荐：层数多、缓存易失效
RUN apt update
RUN apt install -y curl git
RUN rm -rf /var/lib/apt/lists/*

# 推荐：一条 RUN 搞定，并在同一层里清缓存
RUN apt update && apt install -y --no-install-recommends curl git \
    && rm -rf /var/lib/apt/lists/*
```

> [!tip] 清缓存必须在同一层
> 镜像分层是"增量"的，下一个 `RUN rm` 只是在上层标记删除，文件仍占着上一层空间。

```dockerfile
# 不推荐：先 COPY 全部代码，改一行就整个重装依赖
COPY . .
RUN npm install

# 推荐：先拷依赖清单，再拷源码，充分利用缓存
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
```

## 6.5 一个可用的 Python 镜像

```dockerfile
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    TZ=Asia/Shanghai

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s \
  CMD python -c "import urllib.request;urllib.request.urlopen('http://127.0.0.1:8000/health')"

CMD ["python", "-m", "app"]
```

几个细节值得记：

- `slim` 比完整版小很多，比 `alpine` 兼容性好（alpine 用 musl libc，编译型依赖容易出问题）
- `PYTHONUNBUFFERED=1` 让日志立刻输出，否则 `docker logs` 看不到实时内容
- 非 root 运行，降低被攻破后的影响面

## 6.6 多阶段构建

把"编译环境"和"运行环境"分开，最终镜像只保留产物：

```dockerfile
FROM golang:1.23-alpine AS builder
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /out/app .

FROM alpine:3.20
RUN apk add --no-cache ca-certificates
COPY --from=builder /out/app /usr/local/bin/app
ENTRYPOINT ["app"]
```

效果：镜像从 ~800MB 降到 ~20MB，同时**源码不会进入最终镜像**。

> [!tip] 命名阶段 + 只构建某阶段
> `docker build --target builder -t app:dev .` 可以只构建到编译阶段，用于调试。

## 6.7 ARG 与 ENV

```dockerfile
ARG NODE_VERSION=20          # 构建期，默认值
FROM node:${NODE_VERSION}

ARG BUILD_DATE               # 无默认值，必须 --build-arg 传入
ENV APP_ENV=production       # 运行期可见
```

```bash
docker build --build-arg NODE_VERSION=18 --build-arg BUILD_DATE=$(date +%F) -t app .
```

> [!warning] 别把密钥写进 ARG / ENV
> 它们会留在镜像历史里，`docker history` 就能看到。密码、token 用 BuildKit 的 secret 挂载：`RUN --mount=type=secret,id=npmrc npm ci`。

## 6.8 最佳实践清单

1. 基础镜像选**具体版本**，别用 `latest`
2. 用 `.dockerignore` 减小上下文
3. 多阶段构建，运行镜像只带产物
4. 一个容器一个进程
5. 合并 `RUN`，同一层清缓存
6. 先拷依赖清单，再拷源码（缓存友好）
7. 非 root 运行（`USER`）
8. 用 `exec` 形式的 `CMD` / `ENTRYPOINT`
9. 加 `HEALTHCHECK`
10. 别装用不到的包（编译器、编辑器、调试工具）
