---
title: "docker常用命令整理"
date: 2022-05-08T18:16:48+08:00
updated:
copyright: true  # 暂不支持
category: 想法与笔记
tags:
    - docker
toc: true
---

最近在部署一些应用时候用到了docker，上一次用docker也是几年前了，很多命令印象已经不深刻了，这里稍微复习整理了一下常用的命令，方便以后部署应用使用。

## container 相关

### 运行一个container

启动一个container是docker应用部署的关键一步，有许多配置选项需要注意。

```bash
docker run [flags] XXX/IAMGE_TO_RUN
```

<!--more-->

其中各种常用的flags设置如下：

- `-d` detach mode 在后台打开
- `-i` 交互式使用docker（官方文档给出的解释是 *Keep STDIN open even if not attached*）
- `-t` 启一个模拟终端，常与-i一同使用
- `-e` 设置环境变量
- `-p 1010:80` 端口映射；这里的例子中，外部的1010端口，会映射到container的80端口
- `-v outside:/src` volume 挂载；这里的例子中， 外部的outside卷，会挂在到容器/src目录下
    - 使用绝对路径可以直接挂载外部的目录
- `--restart=unless-stopped` 指定容器exit后的状态，常用`unless-stopped`选项，其含义是重启容器直到手动停止
- `--rm` 停止容器时候，自动删除容器
- `--network your-network-name` 使container连接到某一个network中
- `--network-alias container-host-name` 给该container赋予一个网络别名（用于多个container之间网络通信）

### container管理

我们常需要管理container状态，包括启动/停止/删除某个container，查看正在运行的container等等，我个人主要用到下面三个命令：

```bash
# 显示现在正在运行的container
docker ps

# 停止某个容器
docker stop <the-container-id>

# 删除某个容器
docker rm <the-container-id>
docker rm -f <the-container-id> #强制删除某个容器，不需要stop直接删除
```

### 进入/推出到一个正在运行的 container

有时候我们需要进入容器内部，进入和退出的命令如下：

```bash
# 进入一个正在运行的容器
docker exec -it [container-id] bash

# 从容器中退出
exit
```

### 多个container之间网络通信

docker中container之间是相互独立的，无法相互访问。如果需要多个container相互访问（如一个container是主应用，另一个app是数据库），我们须将其放到相同的network下。

```bash
docker network create your-network-name
```

在docker run 命令中，加入`--network your-network-name` 选项，使该container处于你设定的network下。

我们还可以在docker run 命令中加入 `--network-alias container-host-name` 选项，其作用相当于给该container一个host name，方便进行网络配置时候使用。

此外，启动一个需要用到多个container的应用也可以使用docker compose，使用更加方便一些。

## image相关

### Dockerfile

这里给出官方教程中的Dockerfile样例。更详细的命令介绍可以参见[Dockerfile reference | Docker Documentation]([https://docs.docker.com/engine/reference/builder/](https://docs.docker.com/engine/reference/builder/))。

```docker
# syntax=docker/dockerfile:1
FROM node:12-alpine
RUN apk add --no-cache python2 g++ make
WORKDIR /app
COPY . .
RUN yarn install --production
CMD ["node", "src/index.js"]
EXPOSE 3000
```

### docker build

使用`docker build`命令，可以生成Dockerfile所配置的image。

```java
docker build -t getting-started .
```

- `-t` 制定build出来image的名字
- 最后的`.` 指定从哪个目录下搜寻Dockerfile

### 上传image到docker hub

我们可以将个人配置的image上传到docker hub进行分享。

```bash
#登陆dockerhub
docker login -u YOUR-USER-NAME

# 给image 一个 tag
docker tag getting-started YOUR-USER-NAME/getting-started

# 上传
docker push YOUR-USER-NAME/getting-started
```