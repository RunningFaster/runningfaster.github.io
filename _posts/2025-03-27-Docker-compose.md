---
layout: post
title: Docker-compose手册
subtitle: 
cover-img: 
thumbnail-img: /assets/img/docker-compose.png
share-img: 
tags: [install]
author: Jamie
---

## Dockerfile 文件参数

```dockerfile
# 第一条指令必须为FROM指令，如果在同一个Dockerfile中创建多个镜像时，可以使用多个FROM指令
FROM python:3.7.2

# 如果有Dockerfile基于当前镜像，则会自动执行ONBUILD指令内容，等价于在后边添加了两条指令
ONBUILD ADD . /app/src

# 开发者信息
MAINTAINER Jamie <ren@126.com>

# 在容器内创建文件夹，RUN指令将对镜像执行跟随的命令，每运行一条RUN指令，镜像添加新的一层，并提交。命令较长时，可以使用 \ 来换行
# RUN 格式为 RUN <command> 或者 RUN ["executable", "param1", "param2"]
# 前者将在shell终端中运行命令，即 /bin/sh -c; 后者则使用exec执行
# 指定使用其他终端可通过第二种方式实现，例如 RUN ["/bin/bash", "-c", "echo hello"]
RUN mkdir -p /var/www/html/docker_django

# 暴露端口号
export 8080

# 指定一个环境变量，会被后续RUN指令使用，并在容器运行时保持
ENV ENV VERSION=1.0 DEBUG=on \
 NAME="Happy Feet"

# 设置容器内的工作目录，为后续的 RUN CMD ENTRYPOINT 指令配置工作目录
# 可以使用多个 WORKDIR 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径
WORKDIR /var/www/html/docker_django
WORKDIR docker_django
> 最终路径为 /var/www/html/docker_django/docker_django

# 将当前目录内的文件拷贝的容器的工作目录中，<src> 可以是一个相对路径，可以是一个URL，还可以是个tar文件（自动解压为目录）
ADD . /var/www/html/docker_django

# 复制本地主机的<src>（为Dockerfile所在目录的相对路径）到容器中，党使用本地目录为源目录时，推荐使用COPY
COPY . /var/www/html/docker_django

# 更新pip版本 和 安装依赖
RUN pip install -r requirement.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

# 设置start.sh文件的可执行权限
RUN chmod +x ./start.sh

# 启动服务时的操作命令
CMD python manage.py runserver 0.0.0.0:8080

# 配置容器启动后执行的命令，并且不可被docker run提供的参数覆盖
# 与CMD的区别：
ENTRYPOINT python manage.py runserver 0.0.0.0:8080
```

## docker-compose YML 文件参数列表

```yml
version: "3"

services:
  web:
    image: ubuntu # 指定为镜像名称或者镜像ID
    build: ./docker_django # 使用docker_django目录下得Dockerfile
    command: python manage.py runserver 0.0.0.0:8080 # 覆盖容器启动后默认执行的命令
    link: # 链接到其他服务中的容器，使用 服务名称或者 服务名称:服务别名 都可以
        - db
        - db:database
        - redis
    external_links: # 链接到 docker-compose.yml 外部的容器
        - redis1
    ports: # 映射端口
      - "3000"
      - "8080:8080"
      - "22110:22"
      - "0.0.0.0:8080:8080"
    export: # 暴露端口，但不映射到宿主机，只能被连接的服务访问
      - "8000"
    volumes: # 卷挂载路径设置，可以设置宿主机路径，或者加上访问模式（HOST:CONTAINER:ro）
      - ./docker_django:/var/www/html/docker_django # 挂载项目代码
    volumes_from:
      - service_name 从另一个服务或者容器挂载他的所有卷
    environment: # 设置环境变量，只给定名称的变量会自动获取它在Compose主机上的值，可以用来放置泄露不必要的数据
      - DEBUG=False
      - SESSION_SECRET
    env_file: # 从文件中获取环境变量，可以为单独的文件路径或列表
      - ./common.env
    pid: "host" # 跟主机系统共享进程命名空间，打开该选项的容器可以相互通过进程ID来访问和操作
    dns: # 可以是一个值， 也可以是一个列表
      - 8.8.8.8
    extends: # 基于已有的服务进行拓展，后者会自动继承common.yml中的webapp服务及相关变量
      file: common.yml
    restart: always
    tty: true
    stdin_open: true
```