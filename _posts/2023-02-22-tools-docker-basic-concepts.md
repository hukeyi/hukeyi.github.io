---
title: 【工具】Docker：image container volume 概念及其使用
author: hukeyi
date: 2023-02-22 21:07:00 +0800
categories: [工具]
tags: [docker]
math: true
mermaid: true
toc: true
---

> Docker 101 tutorial 的阅读笔记。

Docker 最重要的三个概念：

- container：动态概念，运行中的 app。类似「进程」的概念。
- image：静态概念。
- volume：磁盘。

通过 Docker 创建 app 的简易流程：

1. 创建文件 `Dockerfile`（无扩展名）。包含可读性强的代码，写明 app 的环境构建、启动流程
2. 通过 `Dockerfile` 生成（build） image
3. 通过 `docker run ...` 启动（run）一个容器（i.e. 运行 app）

![Docker 的三个基础概念关系图](/assets/img/2023/docker-file-image-container-relationships.png)

## Image

想象为 app 的安装包。

### 写 Dockerfile

```
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "./src/index.js"]
```

以上是 docker 101 tutorial 给的 Dockerfile。

Dockerfile 就是一个指示工作流程的文件，翻译成人话 belike：

```
以 node:18-alpine 这个 image 为基础
在容器中工作目录为 /app
把当前文件夹中的东西复制到 .（i.e. 工作目录 /app）
命令行执行 `yarn install --production`（i.e. 下载 app 所需的全部依赖，类似 npm install）
当有容器 run 当前 image 时，自动执行 `node ./src/index.js`（i.e. 运行 app 的入口文件）
```

Dockerfile 和 image 的关系，好比程序源代码 `*.cpp` 和可执行程序 `*.exe` 的关系。

### 生成 image

```shell
docker build -t <image-name> <Dockerfile-path>
```

执行上述代码，根据指定 Dockerfile 生成 image（belike 通过源代码 build app 的执行程序）。

## Container

想象为 `*.exe` 执行程序 + 操作系统。

> container 无需“创建”。
> 
> container 是动起来的 image，只要有 image，就可以通过 image 直接 run 一个 container。

### 开启容器

```shell
docker run -d -p port:port <image-name>
```

`<image-name>` 为容器依托的 image 的名字。

> Docker 似乎会给 container 随机取名。

### 删除容器

```shell
docker rm -f <container-id>
```

## Container v.s. Image

container 是运行中的 app；image 是静态的 app 的文件系统。

container 和 image 好比**进程**和**进程控制块 PCB**。

> Simply put, a container is another **process** on your machine that has been isolated from all other processes on the host machine. 
> 
> Since the image contains the container's **filesystem**, it must include everything needed to run the application - all dependencies, configuration, scripts, binaries, etc. The image also contains other configuration for the container, such as environment variables, a default command to run, and other metadata.（from: 官方入门指南 docker 101 tutorial）

只要对应的 image 还存在，container 可以随意删除。container 的所谓「删除」，相当于「关闭应用程序」。执行 `docker rm -f <container-id>` 时可以随意一点，无需担心删掉什么重要的东西。

「执行 container」相当于「开启应用程序」。

## Volume

volume 用于构建虚拟机与真实主机机器某些部分的沟通桥梁。

指南中提到两种类型的 volume：

- Named volume
- Bind Mounts

以上两者都是根据不同的目的，将虚拟机与真实主机上的实际存在的文件目录/数据库连接起来。前者的目的是**使用主机的数据库**；后者的目的是**实时同步主机的工作目录**。

### Named Volume

这种 volume 的目的，用 fancy 术语来说，是为了解决 app 的持久性存储问题。实际上就是给 app 连接一个数据库。

可以类比 windows 系统中的磁盘管理。从 C 盘中新划分一个 named volume，这个磁盘专用于 app 的数据存储。

在主机中创建一个 named volume（在磁盘中划分一个新卷）：

```shell
docker volume create todo-db
```

与虚拟机某个目录连接（把上一步创建的新卷分配给 app getting-started）：

```shell
docker run -dp 3000:3000 -v todo-db:/etc/todos getting-started
```

> `-v todo-db:/etc/todos`：冒号前为保存在主机上的 named volume，后为虚拟机的目录。

上面这条指令表明连接主机上的 `todo-db` Sqlite 数据库文件与虚拟机中的 `/etc/todos` 工作目录。

把本地真实主机的数据库与虚拟机中运行的程序连接起来，让虚拟机 container 能够使用主机上的数据库，这就是 named volume。

### Bind Mounts

这种 volume 的目的，用 fancy 术语来说，是为了实现热重载，人话就是虚拟机中的文件可以实时同步主机中的文件更新。通常用在处于开发阶段的 app 上，方便测试。

```shell
-v "$(pwd):/app"
```

上面这条指令表明连接主机上当前文件路径（必须是绝对路径）与虚拟机中的 `/app` 工作目录。

如果不使用 Bind Mounts，一旦 image 的代码文件更新，只能通过删除当前容器，再重新 build image，再重新 run 容器，才能看到代码更新后的 app 运行效果。使用 Bind Mounts 之后就不需要了，容器/image 即时同步主机的代码更新，实时看到最新运行效果，