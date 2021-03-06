# Docker

## 简介

### 什么是Docker

- 一般指虚拟化容器技术

<!-- more -->

### 优势

- 高系统资源利用率
- 虚拟容器启动快，响应快
- 可使用虚拟化来提供一致的运行环境
- 使用 Dockerfile 定制运应用运行的容器配置，简化持续集成、持续交付、持续部署工作
- 可迁移（容器屏蔽了平台的差异性）
- 易维护、易扩展（镜像复用、镜像层级构建、镜像库）
- 对比传统虚拟机的优势
  - 启动快
  - 占用存储空间小（传统虚拟机的虚拟机镜像一般为 GB 级别）
  - 性能高（直接共享宿主机内核）
  - 单主机可支持上千容器

## 基本概念

### Docker 镜像

- 一个静态化的文件系统，为容器运行提供资源及相关配置
- 镜像是静态化的，生成后不会改变
- 镜像可分层叠加、复用

### Docker 容器

- 基于镜像创建的一个系统进程（实例），用来运行应用
- 容器基于镜像创建属于自己的存储层，该存储层随容器摧毁而消亡
- 可使用数据卷来保存容器产生的数据

### Docker Registry

- 一个集中存储、分发镜像的服务
- Registry 中可包含多个仓库，仓库中包含多个标签，每个标签对应一个镜像
- Registry 中的镜像格式 <仓库名>:<标签>，例如 ubuntu:16.04
- 有公有的 Registry 服务，如 Docker Hub。也可搭建私有服务

## 使用镜像

### 获取镜像

- docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
- 不指定仓库地址则默认使用 Docker Hub
- 仓库名中的用户名默认使用官方名 library
- 例如 docker pull ubuntu:18.04

### 运行

- docker run -it --rm ubuntu:18.04 bash
  - -it: i 表示交互（interactive），t 表示分配一个终端（tty）
  - --rm：容器退出后自动删除容器
  - bash: 容器启动后执行的命令

### 列出已下载的镜像

- docker image ls

### 删除镜像

- docker image rm [选项] <镜像1> [<镜像2> ...]
- 可以使用 ID 、<仓库名>:<标签>、摘要来进行删除
- 删除分为两步
  - Untagged 删除镜像对应的标识 ID
  - Deleted 删除镜像

### 使用 docker commit 保存容器

- docker commit 可以将一个运行的容器保存为一个镜像，类似快照
- docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
  - 例如 docker commit webserver nginx:v2 
- docker diff 查看变更内容
- docker history 查看镜像历史记录
- docker commit 只适用于临时保存容器内容，不适用于制作镜像

### 使用 Dcokerfile 定制镜像

- FROM
  - 指定服务类基础镜像：nginx/redis/mysql...
  - 指定基础系统镜像：ubuntu/debian/centos...
  - 指定空白镜像：scratch
- RUN
  - 注意每个 RUN 会构建一层镜像
  - 格式
    - shell 格式： RUN <命令>
    - exec 格式：RUN ["可执行文件", "参数1", "参数2"]
- 构建镜像
  - docker build [选项] <上下文路径/URL/->
  - 在 Dockerfile 所在目录执行 docker build -t tagname .
  - 指定上下路径会将该路径下的所有文件提交给 docker 引擎，然后执行 Dockerfile 中的指令
  - Dockerfile 文件名仅仅是约定，可以通过 -f 指定任意文件
  - 默认会将上下文环境中的 Dockerfile 文件作为 Dockerfile
  - 可以使用 .dockerignore 来指定忽略不提交给 docker 引擎的文件
  - 其它构建方式
    - 使用 Git repo: docker build https://github.com/twang2218/gitlab-ce-zh.git#:11.1
    - 使用给定 tar 压缩包构建：docker build http://server/context.tar.gz
    - 从标准输入中读取 Dockerfile 进行构建： docker build - < Dockerfile 或 cat Dockerfile | docker build -
    - 从标准输入中读取上下文压缩包进行构建： docker build - < context.tar.gz

### 其他操作镜像的方式

- 从 rootfs 压缩包导入镜像 docker import [选项] <文件>|<URL>|- [<仓库名>[:<标签>]]
- 使用 docker save 保存镜像为文件，方便传输
  - docker save <镜像名> | bzip2 | pv | ssh <用户名>@<主机名> 'cat | docker load'
- 使用 docker load 加载使用 docker save 保存的镜像文件

## 操作容器

### 查看容器

- docker container ls / docker ps

### 启动/终止容器

- 创建并启动容器
  - docker run [选项] 镜像 [命令] [ARG...]
  - 例如 docker run -it ubuntu:18.04 bash
- 终止容器：docker stop/docker container stop [容器id或名称]
- 启动终止状态的容器
  - 使用 docker stop/docker container stop 终止容器
  - 对于尚未删除的容器，还能使用 docker start/docker container start 来重新启动该容器
- 重启容器：docker restart/docker container restart [容器id或名称]

### 后台运行

- 使用 -d 选项使容器后台运行：docker run -d ubuntu:18.04
- 获取容器输入日志：docker logs/docker container logs [容器id或名称]

### 进入容器

- attach
  - docker attach [容器id或名称]
  - 此时在终端里 exit，容器会停止
- exec
  - docker exec -it [容器id或名称] [命令]
    - 例如 docker exec -it nginx bash
    - -i: 交互模式
    - -t: 分配伪终端
  - 利用 docker exec 进入容器，在终端里 exit，容器不会停止

### 导入/导出容器

- 导出容器 docker export [容器id或名称] -o [文件名]
- 导入容器为镜像 docker import [OPTIONS] file|URL|- [REPOSITORY[:TAG]]
- docker import 与 docker load 的区别
  - docker import 用来导入容器快照文件，docker load 导入镜像存储文件
  - 使用docker import 导入的镜像没有完整的历史信息与元数据，docker load 会保留原镜像完整的历史信息与元数据

### 删除容器

- 删除终止的容器：docker rm/docker container [OPTIONS] CONTAINER [CONTAINER...]。可以使用 -f 选项强制删除，删除时，Docker 会发送 SIGKILL 信号给容器
- 清理所有终止状态的容器：docker container prune [OPTIONS]

## Docker 数据管理

### 数据卷

- 数据卷的特点
  - 可以供多个容器共享、重用
  - 对数据卷修改立即生效
  - 独立，修改数据卷不影响镜像
  - 数据卷不会随容器删除而销毁
- 使用数据卷
  - 创建数据卷：docker volume create [VOL-NAME]
  - 查看数据卷信息： docker volume inspect VOL-NAME
  - 查看容器使用的数据卷信息：docker inspect CONTAINER
  - 启动容器时挂在数据卷： docker run -d -v VOL-NAME:/TARGET-DIR CONTAINER [COMMANDS ]
  - 删除数据卷
    - docker volume rm VOL-NAME
    - 删除未使用的数据卷：docker volume prune
    - 删除容器时同时删除挂载的数据卷：docker rm -v CONTAINER

### 挂载主机目录/文件

- docker run -d --mount type=bind,src=/projects/dist,ro,dst=/usr/share/nginx/html -p 8080:80 nginx

## Docker 网络管理

### 映射端口

- 使用 -P 将会随机映射主机的端口到容器内的开放端口，这个随机范围由 `/proc/sys/net/ipv4/ip_local_port_range` 定义
- 使用 -p 指定映射端口
  - -p, --publish ip:[hostPort]:containerPort | [hostPort:]containerPort
  - 映射范围端口 -p 1234-1234:1222-1224, 端口数须相等
  - 可以重复使用 -p 来指定多个端口映射

### 网络使用基础

- 创建网络
  - docker network create -d bridge mynet, -d 指定网络驱动类型，默认为 bridge
- 创建容器时指定连接的网络
  - docker run -d --name n1 --network my-net nginx
- 同网络中的容器互联
  - 在其他容器中可直接使用容器名连接，如 ping n1

### 配置容器 DNS

- 容器启动时默认使用主机的网络配置： hosts/dns 配置

- 启动时为单个容器指定配置： docker run -h HOSTNAME --dns=IP --dns-search=DOMAIN CONTAINER

- 通过 /etc/docker/daemon.json 进行全局配置

  ```json
  {
    "dns" : [
      "114.114.114.114",
      "8.8.8.8"
    ]
  }
  ```

## Dockerfile

Dockerfile 示例

```dockerfile
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```

### FROM

- 说明
  - 指定 Docker 基础镜像

### COPY

- 格式
  - COPY [ --chown=<user>:<group> ] <源路径>... <目标路径>
  - COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]
- 说明
  - 如果 <源路径> 是一个目录，COPY 复制 <源路径> 下的文件/目录， 不包含 <源路径> 本身
  - COPY 用于将 <源路径> 的文件/目录复制到镜像中的 <目标路径>

### ADD

尽量使用 COPY 指令

- 格式
  - ADD 包含和 COPY 一样的使用格式和功能
- 比 COPY 额外增加的功能
  - <源路径> 可以是一个 URL
  - 如果 <源路径> 是一个本地 tar 压缩包(gzip, bzip2, xz)，压缩包中的内容会自动解压到目标目录中

### RUN

- 说明
  - 构建镜像时执行的指令，可多个，每个一层

### CMD

- 格式

  - CMD ["executable", "param1", "param2"…]

    推荐使用，例如启动 httpd 服务

    CMD ["apache2","-DFOREGROUND"]

  - 搭配 ENTRYPOINT 使用：CMD ["param", "param"]

  - shell 格式：CMD <executable>

- 说明

  - 用于指定容器启动时，主进程执行的指令

    执行命令后，需要保持前台运行，否则进程退出，容器停止

  - 使用 docker run 运行镜像时，docker run 后指定的命令会覆盖 CMD

### ENTRYPOINT

- 格式
  - 与 CMD 格式一样，支持 CMD 的功能
- 说明
  - 用于指定镜像运行时的主命令，指定了该选项后，CMD 的内容将作为参数传递给 ENTRYPOINT 指定的命令

### ENV

- 格式
  - ENV <key> <value>
  - ENV <key1>=<value1> <key2>=<value2>...
- 说明
  - 指定容器的环境变量

### ARG

- 指定镜像构建时的环境变量，不影响容器运行环境：ARG <参数名>[=<默认值>]

### VOLUME

- 格式
  - VOLUME <路径1> <路径2>
  - VOLUME ["<路径1>", "<路径2>"]
- 说明
  - 容器启动时，如果没有指定挂载卷，挂载点会自动挂载为匿名卷，容器停止，匿名卷会删除
  - 用于声明一个容器的挂载点

### EXPOSE

- 格式
  - EXPOSE <port> [<port>/<protocol>...]
- 说明
  - 声明容器运行时的监听端口

### WORKDIR

- 格式
  - WORKDIR /path/to/workdir
- 说明
  - 指定 RUN, CMD, ENTRYPOINT, COPY 和 ADD 的工作目录
  - WORKDIR 可以重复使用，使用时会自动创建目录

### USER

- 说明
  - 指定 RUN, CMD, ENTRYPOINT 的用户和组
- 格式
  - USER <user>[:<group>]
  - USER <UID>[:<GID>]

### HEALTHCHECK

```dockerfile
HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1
```

- 说明
  - 设置容器健康检查指令
  - 可以使用 HEALTHCHECK NONE 来清空父层设置的健康检查指令
  - 健康检查命令通过返回值来确定服务是否正常：0成功 1失败 2保留（该状态码保留不使用）
- 格式
  - HEALTHCHECK [OPTIONS] CMD command
    - 支持的选项
      - 设置检查间隔 --interval=30s
      - 设置超时时间 --timeut=30s
      - 开始检查时间 --start-period=0s
      - 重试次数 --retries=3
  - HEALTHCHECK NONE

### ONBUILD

基础镜像 my-node:

```dockerfile
FROM node:slim
RUN mkdir /app
WORKDIR /app
ONBUILD COPY ./package.json /app
ONBUILD RUN [ "npm", "install" ]
ONBUILD COPY . /app/
CMD [ "npm", "start" ]
```

以 my-node 作为基础的镜像 node-app：

```dockerfile
FROM my-node
```

这样当 node-app 构建时， 基础镜像中 ONBUILD 后的指令 就会执行

- 说明
  - 当镜像作为基础镜像时，构建时，基础镜像中 ONBUILD 后的指令会执行
- 格式
  - `ONBUILD <INSTRUCTION>`

