---
title: Docker
date: 2018-08-18 23:30:30
categories:
- DevOps
tags:
- Docker
---
## Concepts

### Images
* 一个只读模板，可以用来创建容器，一个镜像可以创建多个容器
* Docker 提供了一个很简单的机制来创建和更新现有的镜像，甚至可以直接从其他人那里获取做好的镜像直接使用

![Docker File System](docker-file-system.png)

* docker 镜像代表了容器的文件系统里的内容，是容器的基础，镜像一般是通过 Dockerfile 生成的
* docker 的镜像是分层的，所有的镜像（除了基础镜像）都是在之前镜像的基础上加上自己这层的内容生成的
* 每一层镜像的元数据都是存在 json 文件中的，除了静态的文件系统之外，还会包含动态的数据

#### Commands
Search on Docker store
`docker search mysql`

Pull an image from Docker store
`docker pull mysql:latest`

Push an image to Docker store
`docker push`

List all downloaded images
`docker images`

Delete an image
`docker rmi mysql`
`docker rmi mysql tomcat`

Delete all iamges
`docker rmi $(docker images -q)`

Build an image from Dockerfile
`docker build`

Tag an image
`docker tag`

### Container
* 容器是从镜像创建的运行实例，也就是镜像启动后的一个实例称为容器，是独立运行的一个或一组应用。
* docker 利用容器来运行应用，他可以被启动、开始、停止、删除，每个容器都是相互隔离的、保证安全的平台。

#### Commands
Run a container from an image
`docker run --name container-name -d image-name`

List all running containers
`docker ps`

Stop a running container
`docker stop container-name/container-id`

Force stop a running container
`docker kill container-name/container-id`

Start a container
`docker start container-name/container-id`

Restart a container
`docker restart container-name/container-id`

Delete a container
`docker rm container-id`

Delete all containers
`docker rm $(docker ps -a -q )`

Check logs
`docker logs container-id/container-name`

Attach to a running container
`docker attach container-id`

Run a command on a running container
`docker exec -it container-id/container-name bash`

List running processes on a container
`docker top container-id`

Inspect a container
`docker insepct container-id`

Copy file from host to container
`docker cp 文件 container-id:目标文件/文件夹`
`docker cp /tmp/suzhuji.txt 7f237caad43b:/tmp`

Copy file from container to host
`docker cp container-id:目标文件/文件夹 宿主机目标文件/文件夹`
`docker cp 7f237caad43b:/tmp/yum.log /tmp`

### Resoisitory

* 仓库是集中存放镜像文件的场所，类似 git 代码仓库等。
* 仓库（Respository）和仓库注册服务器（Registry）是有区别的。仓库注册服务器一般存放多个仓库，每个仓库又有多个镜像，每个镜像又有不同的标签（tag）。
* 仓库分为公开仓库（public）和私有仓库（private）两种形式。
* 当创建好自己的镜像后，可以通过 push 命令把它上传到公开或私有仓库。