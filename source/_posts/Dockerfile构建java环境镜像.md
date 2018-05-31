---
title: "Dockerfile构建java环境镜像"
tags:
  - noname
id: 521
categories:
  - noname
date: 2018-05-28 18:20:31
author: 
  - linxu
---

# dockerfile详解 #
Dockerfile是一个文本格式的配置文件，用户可以使用Dockerfile快速构建自定义的镜像。

## dockerfile基本结构 ##
dockerfile一般分为四部分:
1. 基础镜像信息
2. 维护者信息
3. 镜像操作指令
4. 容器启动时执行指令

## dockerfile基本命令 ##
1. FROM命令，选择一个基础镜像，如果有多个，可以使用多个from
	```
	FROM <image> 或 FROM <image>:<tag>
	```
2. MAINTAINER命令，说明作者信息
	```
	MAINTAINER <name> <email>
	```
3. RUN命令，RUN指令将在当前镜像基础上执行指定命令
	```
	RUN <command> 
	RUN ["executable","param1","param2"] 
	```
4. cMD命令，指定启动容器时执行的命令，每个Dockerfile只能有一条CMD命令。如果指定了多条命令，只有最后一条会被执行。如果用户启动容器的时候指定了运行的命令，则会覆盖掉CMD指定的命令
	```
	支持三种格式：
	CMD ["executable","param1","param2"]使用exec执行，推荐方式。
	CMD command param1 prarm2 在/bin/sh中执行，提供给需要交互的应用
	CMD ["prarm1","param2"]提供给ENTRYPOINT的默认参数
	```
5. EXPOSE命令，暴露容器端口
	```
	EXPOSE <port> <port> ...
	```
6. ENV命令，指定环境变量
	```
	ENV <key> <value>
	```
7. ADD命令，复制宿主机文件或目录到容器中，其中src可以是Dockerfile所在目录的一个相对路径(文件或目录)，也可以是一个URL，还可以是一个tar文件
	```
	ADD <src> <dest>
	```
8. COPY命令，复制本地主机的src为容器中的dest，目标路径不存在时，会自动创建
	```
	COPY <src> <dest>
	```
9. ENTRYPOINT命令，配置容器启动后执行的命令，并且不可被docker run 提供的参数覆盖。每个Dockerfile中只能有一个ENTRYPOINT，当指定多个ENTRYPOINT时，只有最后一个生效。
	```	
	ENTRYPOINT ["executable","param1","param2"]
	ENTRYPOING command param1 param2 (shell中执行)
	```
10. VOLUME命令，创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。
	```
	VOLUME ["/data"]
	```
11. USER命令，指定运行容器时的用户名或UID，后续的RUN也会使用指定用户
	```
	USER daemon
	```
12. WORKDIR命令，为后续的RUN、CMD、ENTRYPOINT指令配置工作目录，可以使用多个WORKDIR指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。
	```
	WORKDIR /path/to/workdir
	```
13. ONBUILD命令，配置当所创建的镜像作为其他新创建镜像的基础镜像时，所执行的操作指令
	```
	ONBUILD [INSTRUCTION]
	```

# Dockerfile构建java环境镜像 #  
## 创建Dockerfile文件 ##
```
mkdir /usr/file
vi /usr/file/Dockfile #文件名必须是Dockfile
#以下是文件内容
from centos
MAINTAINER reallinxu

RUN yum install -y wget
RUN mkdir /usr/software
RUN mkdir /usr/jdk
RUN wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" -P /usr/software http://download.oracle.com/otn-pub/java/jdk/8u171-b11/512cd62ec5174c3487ac17c61aaa89e8/jdk-8u171-linux-x64.tar.gz
RUN tar xzf /usr/software/jdk-8u171-linux-x64.tar.gz -C /usr/jdk
ENV JAVA_HOME /usr/jdk/jdk1.8.0_171
ENV PATH $PATH:$JAVA_HOME/bin
EXPOSE 8080
```

## 构建镜像 ##
```
docker build -t="reallinxu/javaimage:v1" .
```

## 运行容器 ##
```
docker run -i -t reallinxu/javaimage:v1 /bin/bash
```

# 构建noname镜像 #
使用本地jdk创建，首先将jdk-8u144-linux-x64.tar.gz，放到Dockerfile同一个文件夹下
```
#Dockerfile文件
from centos
MAINTAINER reallinxu

COPY jdk-8u144-linux-x64.tar.gz /usr/software/jdk-8u144-linux-x64.tar.gz
WORKDIR /usr/software
RUN mkdir /usr/jdk
RUN tar xzf /usr/software/jdk-8u144-linux-x64.tar.gz -C /usr/jdk
ENV JAVA_HOME /usr/jdk/jdk1.8.0_171
ENV PATH $PATH:$JAVA_HOME/bin
EXPOSE 8080

docker build -t="reallinxu/noname" .
```