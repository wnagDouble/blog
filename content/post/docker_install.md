---
title: "Docker安装"
date: 2023-12-10T13:42:49+08:00
categories: ['Docker']
tags: ['Docker','笔记']
---
# Docker入门安装&使用
> 仅记录自己学习Docker的过程。安装的是Docker Engine不是Desktop。  
## 运行环境  
- 操作系统：CentOS Linux 8  
## 参考网站  
- [官方CentOS下载Docker Engine指引](https://docs.docker.com/engine/install/centos/)  
## 执行步骤  
### 配置repository  
用于安装和更新Docker。  
```shell
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```  
### 安装Docker Engine  
```shell
sudo yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```  
### 启动Docker  
```shell  
sudo systemctl start docker
```  
### 测试Docker  
```shell  
sudo docker run hello-world
```  
当你执行 `sudo docker run hello-world` 命令时，Docker 会做以下几件事：

1. Docker 会在本地查找名为 "hello-world" 的镜像。
2. 如果在本地没有找到，Docker 会去 Docker Hub（默认的 Docker 镜像仓库）下载 "hello-world" 镜像。
3. Docker 会创建一个新的容器，并在这个容器中运行 "hello-world" 镜像。

"hello-world" 镜像在下载后存储在 Docker 的镜像库中。你可以通过 `docker images` 命令查看所有的 Docker 镜像。

"hello-world" 容器在运行后会存在于 Docker 的容器列表中，即使它已经停止运行。你可以通过 `docker ps -a` 命令查看所有的 Docker 容器，包括已经停止运行的。

所以，"hello-world" 镜像存储在 Docker 的镜像库中，而 "hello-world" 容器存储在 Docker 的容器列表中。
## 删除指定镜像和容器  
如果你想删除本地的 "hello-world" 镜像和容器，你可以按照以下步骤操作：

1. 首先，你需要找到 "hello-world" 容器的 ID。你可以通过运行 `docker ps -a` 命令来查看所有的 Docker 容器，包括已经停止运行的。找到 "hello-world" 容器的 ID 后，你可以使用 `docker rm` 命令来删除它。

2. 然后，你需要找到 "hello-world" 镜像的 ID。你可以通过运行 `docker images` 命令来查看所有的 Docker 镜像。找到 "hello-world" 镜像的 ID 后，你可以使用 `docker rmi` 命令来删除它。  
### 删除指定镜像  
```shell
sudo docker rmi [image id]
```
### 删除指定容器  
```shell
sudo docker rm [container id]
```
## 卸载  
### 卸载Docker  
```shell
sudo yum remove docker-ce docker-ce-cli containerd.io
```  
### 删除所有镜像、容器、卷和自定义配置  
```shell
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```


