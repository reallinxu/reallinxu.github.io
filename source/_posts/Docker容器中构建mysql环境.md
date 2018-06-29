---
title: "Docker容器中构建mysql环境"
tags:
  - noname
id: 521
categories:
  - noname
date: 2018-05-28 18:20:31
author: 
  - linxu
---

# Mysql简介 #
# Docker中搭建Mysql环境 #
## Dockerfile文件 ##
将mysql-community-release-el7-5.noarch.rpm放入dockerfile同目录
```
from centos
MAINTAINER reallinxu

yum install mysql
COPY mysql-community-release-el7-5.noarch.rpm /usr/software/mysql-community-release-el7-5.noarch.rpm
RUN rpm -ivh /usr/software/mysql-community-release-el7-5.noarch.rpm
RUN yum install mysql-community-serve
RUN yum install mysql-devel
EXPOSE 3306
```