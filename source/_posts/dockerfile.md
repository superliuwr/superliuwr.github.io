---
title: Dockerfile
date: 2018-08-20 23:13:56
categories:
- DevOps
tags:
- Docker
---

Docker 可以通过 Dockerfile 的内容来自动构建镜像。Dockerfile 是一个包含创建镜像所有命令的文本文件，通过 docker build 命令可以根据 Dockerfile 的内容构建镜像。

# Dockerfile
## FROM
* FROM 必须是 Dockerfile 中非注释行的第一个指令，即一个 Dockerfile 从 FROM 语句开始。
* FROM 可以在一个 Dockerfile 中出现多次，如果有需求在一个 Dockerfile 中创建多个镜像。

## RUN
有两种使用方式
* RUN (the command is run in a shell - /bin/sh -c - shell form)
* RUN [“executable”, “param1”, “param2”] (exec form)

每条 RUN 指令将在当前镜像基础上执行指定命令，并提交为新的镜像，后续的 RUN 都在之前 RUN 提交后的镜像为基础，镜像是分层的，可以通过一个镜像的任何一个历史提交点来创建，类似源码的版本控制。

exec 方式会被解析为一个 JSON 数组，所以必须使用双引号而不是单引号。exec 方式不会调用一个命令 shell，所以也就不会继承相应的变量，如：
`RUN [ "echo", "$HOME" ]`

这种方式是不会达到输出 HOME 变量的，正确的方式应该是这样的
`RUN [ "sh", "-c", "echo", "$HOME" ]`

> RUN 产生的缓存在下一次构建的时候是不会失效的，会被重用，可以使用 --no-cache 选项，即 docker build --no-cache，如此便不会缓存。

## CMD
CMD 有三种使用方式:
* CMD [“executable”,”param1”,”param2”] (exec form, this is the preferred form, 优先选择)
* CMD [“param1”,”param2”] (as default parameters to ENTRYPOINT)
* CMD command param1 param2 (shell form)

CMD 指定在 Dockerfile 中只能使用一次，如果有多个，则只有最后一个会生效。

CMD 的目的是为了在启动容器时提供一个默认的命令执行选项。如果用户启动容器时指定了运行的命令，则会覆盖掉 CMD 指定的命令。

> CMD 会在启动容器的时候执行，build 时不执行，而 RUN 只是在构建镜像的时候执行，后续镜像构建完成之后，启动容器就与 RUN 无关了。

## EXPOSE
`EXPOSE <port> [<port>...]`

告诉 Docker 服务端容器对外映射的本地端口，需要在 docker run 的时候使用 -p 或者 -P 选项生效。

## ENV
`ENV <key> <value>       # 只能设置一个变量`
`ENV <key>=<value> ...   # 允许一次设置多个变量`

指定一个环节变量，会被后续 RUN 指令使用，并在容器运行时保留。

```bash
ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy
```

等同于

```bash
ENV myName John Doe
ENV myDog Rex The Dog
ENV myCat fluffy
```

## ADD
`ADD <src>... <dest>`

ADD 复制本地主机文件、目录或者远程文件 URLS 从 <src> 并且添加到容器指定路径中 <dest>。

<src> 支持通过 GO 的正则模糊匹配，具体规则可参见 Go filepath.Match

```sh
ADD hom* /mydir/        # adds all files starting with "hom"
ADD hom?.txt /mydir/    # ? is replaced with any single character
```

<dest> 路径必须是绝对路径，如果 <dest> 不存在，会自动创建对应目录
<src> 路径必须是 Dockerfile 所在路径的相对路径
<src> 如果是一个目录，只会复制目录下的内容，而目录本身则不会被复制

## COPY
`COPY <src>... <dest>`

COPY 复制新文件或者目录从 <src> 添加到容器指定路径中 <dest>。用法同 ADD，唯一的不同是不能指定远程文件 URLS。

## ENTRYPOINT
```sh
ENTRYPOINT [“executable”, “param1”, “param2”] (the preferred exec form，优先选择)
ENTRYPOINT command param1 param2 (shell form)
```

配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖，而 CMD 是可以被覆盖的。如果需要覆盖，则可以使用 docker run --entrypoint 选项。

每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个生效。

### Exec form ENTRYPOINT 例子

1. 通过 ENTRYPOINT 使用 exec form 方式设置稳定的默认命令和选项，而使用 CMD 添加默认之外经常被改动的选项。
```Dockerfile
FROM ubuntu
ENTRYPOINT ["top", "-b"]
CMD ["-c"]
```

2. 通过 Dockerfile 使用 ENTRYPOINT 展示前台运行 Apache 服务
```Dockerfile
FROM debian:stable
RUN apt-get update && apt-get install -y --force-yes apache2
EXPOSE 80 443
VOLUME ["/var/www", "/var/log/apache2", "/etc/apache2"]
ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

### Shell form ENTRYPOINT 例子
1. 这种方式会在 /bin/sh -c 中执行，会忽略任何 CMD 或者 docker run 命令行选项，为了确保 docker stop 能够停止长时间运行 ENTRYPOINT 的容器，确保执行的时候使用 exec 选项。
```Dockerfile
FROM ubuntu
ENTRYPOINT exec top -b
```

2. 如果在 ENTRYPOINT 忘记使用 exec 选项，则可以使用 CMD 补上:
```Dockerfile
FROM ubuntu
ENTRYPOINT top -b
CMD --ignored-param1 # --ignored-param2 ... --ignored-param3 ... 依此类推
```

## VOLUME
VOLUME ["/data"]
创建一个可以从本地主机或其他容器挂载的挂载点

## USER
USER daemon
指定运行容器时的用户名或 UID，后续的 RUN、CMD、ENTRYPOINT 也会使用指定用户。

## WORKDIR
WORKDIR /path/to/workdir
为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录。可以使用多个 WORKDIR 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。

WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
最终路径是 /a/b/c。

WORKDIR 指令可以在 ENV 设置变量之后调用环境变量:

ENV DIRPATH /path
WORKDIR $DIRPATH/$DIRNAME
最终路径则为 /path/$DIRNAME。

# Docker RUN vs CMD vs ENTRYPOINT
* RUN executes command(s) in a new layer and creates a new image. E.g., it is often used for installing software packages.
* CMD sets default command and/or parameters, which can be overwritten from command line when docker container runs.
* ENTRYPOINT configures a container that will run as an executable.

## The bottom line
* Use RUN instructions to build your image by adding layers on top of initial image.
* Prefer ENTRYPOINT to CMD when building executable Docker image and you need a command always to be executed. Additionally use CMD if you need to provide extra default arguments that could be overwritten from command line when docker container runs.
* Choose CMD if you need to provide a default command and/or arguments that can be overwritten from command line when docker container runs.

# dockerfile 最佳实践
## Do not use 'latest' base image tag

## Remove unneeded files after each RUN step

## Use proper base image

## Use multi-stage builds

## 使用 .dockerignore 文件
为了在 docker build 过程中更快上传和更加高效，应该使用一个 .dockerignore 文件用来排除构建镜像时不需要的文件或目录。例如,除非 .git 在构建过程中需要用到，否则你应该将它添加到 .dockerignore 文件中，这样可以节省很多时间。

## 避免安装不必要的软件包
为了降低复杂性、依赖性、文件大小以及构建时间，应该避免安装额外的或不必要的包。例如，不需要在一个数据库镜像中安装一个文本编辑器。

## 每个容器都跑一个进程
在大多数情况下，一个容器应该只单独跑一个程序。解耦应用到多个容器使其更容易横向扩展和重用。如果一个服务依赖另外一个服务，可以参考 Linking Containers Together。

## 最小化层
Merge multiple RUN commands into one

我们知道每执行一个指令，都会有一次镜像的提交，镜像是分层的结构，对于 Dockerfile，应该找到可读性和最小化层之间的平衡。

## 多行参数排序
如果可能，通过字母顺序来排序，这样可以避免安装包的重复并且更容易更新列表，另外可读性也会更强，添加一个空行使用 \ 换行:

```Dockerfile
RUN apt-get update && apt-get install -y \
  bzr \
  cvs \
  git \
  mercurial \
  subversion
```

## 创建缓存
镜像构建过程中会按照 Dockerfile 的顺序依次执行，每执行一次指令 Docker 会寻找是否有存在的镜像缓存可复用，如果没有则创建新的镜像。如果不想使用缓存，则可以在 docker build 时添加 --no-cache=true 选项。

从基础镜像开始就已经在缓存中了，下一个指令会对比所有的子镜像寻找是否执行相同的指令，如果没有则缓存失效。在大多数情况下只对比 Dockerfile 指令和子镜像就足够了。ADD 和 COPY 指令除外，执行 ADD 和 COPY 时存放到镜像的文件也是需要检查的，完成一个文件的校验之后再利用这个校验在缓存中查找，如果检测的文件改变则缓存失效。RUN apt-get -y update 命令只检查命令是否匹配，如果匹配就不会再执行更新了。

为了有效地利用缓存，你需要保持你的 Dockerfile 一致，并且尽量在末尾修改。

## Use "exec" inside entrypoint script

## Dockerfile 指令
FROM: 只要可能就使用官方镜像库作为基础镜像

RUN: 为保持可读性、方便理解、可维护性，把长或者复杂的 RUN 语句使用 \ 分隔符分成多行

不建议 RUN apt-get update 独立成行，否则如果后续包有更新，那么也不会再执行更新

避免使用 RUN apt-get upgrade 或者 dist-upgrade，很多必要的包在一个非 privileged 权限的容器里是无法升级的。如果知道某个包更新，使用 apt-get install -y xxx

标准写法
RUN apt-get update && apt-get install -y package-bar package-foo

例子:
```Dockerfile
RUN apt-get update && apt-get install -y \
    aufs-tools \
    automake \
    btrfs-tools \
    build-essential \
    curl \
    dpkg-sig \
    git \
    iptables \
    libapparmor-dev \
    libcap-dev \
    libsqlite3-dev \
    lxc=1.0* \
    mercurial \
    parallel \
    reprepro \
    ruby1.9.1 \
    ruby1.9.1-dev \
    s3cmd=1.1.0*
```

CMD: 推荐使用 CMD [“executable”, “param1”, “param2”…] 这种格式，CMD [“param”, “param”] 则配合 ENTRYPOINT 使用

EXPOSE: Dockerfile 指定要公开的端口，使用 docker run 时指定映射到宿主机的端口即可

ENV: 为了使新的软件更容易运行，可以使用 ENV 更新 PATH 变量。如 ENV PATH /usr/local/nginx/bin:$PATH 确保 CMD ["nginx"] 即可运行

ENV 也可以这样定义变量：
```Dockerfile
ENV PG_MAJOR 9.3
ENV PG_VERSION 9.3.4
RUN curl -SL http://example.com/postgres-$PG_VERSION.tar.xz | tar -xJC /usr/src/postgress && …
ENV PATH /usr/local/postgres-$PG_MAJOR/bin:$PATH
```

ADD or COPY: ADD 比 COPY 多一些特性「tar 文件自动解包和支持远程 URL」，不推荐添加远程 URL
Prefer COPY over ADD
如不推荐这种方式:
```Dockerfile
ADD http://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```

推荐使用 curl 或者 wget 替换，使用如下方式:
```Dockerfile
RUN mkdir -p /usr/src/things \
    && curl -SL http://example.com/big.tar.gz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```

如果不需要添加 tar 文件，推荐使用 COPY。

# References
* [Dockerfile Notes](https://blog.opskumu.com/docker.html#%E4%B8%83dockerfile)
* [Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
* [How to write excellent Dockerfiles](https://rock-it.pl/how-to-write-excellent-dockerfiles/)
* [Docker RUN vs CMD vs ENTRYPOINT](http://goinbigdata.com/docker-run-vs-cmd-vs-entrypoint/)