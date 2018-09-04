---
title: "Docker容器中搭建Redis集群"
tags:
  - docker
id: 523
categories:
  - Docker
date: 2018-05-30 18:00:00
author: 
  - linxu
---

# Redis简介 #
Redis为传说中的内存数据库的一种，运行在内存中，性能强大，还可以用作缓存和消息中间件。Redis支持多种数据结构的存储，提供了大部分平台的客户端，使用方便。

# Redis优缺点 #
## 优点 ##
1. 性能极高：Redis能读的速度是110000次/s,写的速度是81000次/s。
2. 丰富的数据类型：Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
3. 原子：Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
4. 丰富的特性：Redis还支持 publish/subscribe, 通知, key 过期等等特性。

## 缺点 ##
1. Redis不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的IP才能恢复。
2. 主机宕机，宕机前有部分数据未能及时同步到从机，切换IP后还会引入数据不一致的问题，降低了系统的可用性。
3. redis的主从复制采用全量复制，从机新加入集群或者从机和主机网络断开重连时都会进行，复制过程中主机会fork出一个子进程对内存做一份快照，并将内存快照保存为文件发送给从机，需要保证主机的内存足够。若快照文件较大，将影响集群的服务能力。
4. Redis较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。所以运维人员在系统上线时必须确保有足够的空间，这对资源造成了很大的浪费。

# Docker中搭建Redis集群 #
## Dockerfile文件 ##
将redis-4.0.6.tar.gz,ruby-2.5.0.tar.gz,openssl-1.1.1-pre7.tar.gz文件放入Dockerfile同目录
```
from centos
MAINTAINER reallinxu

COPY redis-4.0.6.tar.gz /usr/software/redis-4.0.6.tar.gz
RUN mkdir /usr/redis
RUN tar xzf /usr/software/redis-4.0.6.tar.gz -C /usr/redis
RUN yum -y install gcc automake autoconf libtool make
RUN cd /usr/redis/redis-4.0.6 && make
COPY ruby-2.5.0.tar.gz /usr/software/ruby-2.5.0.tar.gz
RUN mkdir /usr/ruby
RUN tar xzf /usr/software/ruby-2.5.0.tar.gz -C /usr/ruby
RUN cd /usr/ruby/ruby-2.5.0 && ./configure && ls Makefile && make && make install
RUN yum install -y rubygems
RUN yum install -y zlib-devel
COPY openssl-1.1.1-pre7.tar.gz /usr/software/openssl-1.1.1-pre7.tar.gz
RUN mkdir /usr/openssl
RUN tar xzf /usr/software/openssl-1.1.1-pre7.tar.gz -C /usr/openssl
RUN cd /usr/openssl/openssl-1.1.1-pre7 && ./config -fPIC --prefix=/usr/local/openssl enable-shared && ./config -t && make && make install
RUN cd /usr/ruby/ruby-2.5.0/ext/zlib && ruby ./extconf.rb && sed -i 's/zlib.o: $(top_srcdir)\/include\/ruby.h/zlib.o: ..\/..\/include\/ruby.h/' Makefile && make && make install 
RUN cd /usr/ruby/ruby-2.5.0/ext/openssl && ruby extconf.rb  --with-openssl-include=/usr/local/openssl/include/ --with-openssl-lib=/usr/local/openssl/lib && sed -i 's/$(top_srcdir)/..\/../g' Makefile && make && make install
RUN gem install redis
RUN sed -i 's/daemonize no/daemonize yes/' /usr/redis/redis-4.0.6/redis.conf
RUN sed -i 's/dir .\//dir \/usr\/redis\//' /usr/redis/redis-4.0.6/redis.conf
RUN sed -i 's/# cluster-enabled yes/cluster-enabled yes/' /usr/redis/redis-4.0.6/redis.conf
RUN sed -i 's/# cluster-config-file nodes-6379.conf/cluster-config-file nodes-6379.conf/' /usr/redis/redis-4.0.6/redis.conf
RUN sed -i 's/# cluster-node-timeout 15000/cluster-node-timeout 15000/' /usr/redis/redis-4.0.6/redis.conf
RUN sed -i 's/appendonly no/appendonly yes/' /usr/redis/redis-4.0.6/redis.conf
RUN rm -rf /usr/software/redis-4.0.6.tar.gz
RUN rm -rf /usr/software/ruby-2.5.0.tar.gz
RUN rm -rf /usr/software/openssl-1.1.1-pre7.tar.gz
EXPOSE 6379
```

## 构建镜像 ##
```
docker build -t="reallinxu/redis:v1" .
```
## 创建容器 ##
redis集群需要最少6个节点，3+3模式,所以用以下命令创建6个容器
```
docker run -i -t --name redis1 reallinxu/redis:v1
docker run -i -t --name redis2 reallinxu/redis:v1
docker run -i -t --name redis3 reallinxu/redis:v1
docker run -i -t --name redis4 reallinxu/redis:v1
docker run -i -t --name redis5 reallinxu/redis:v1
docker run -i -t --name redis6 reallinxu/redis:v1
```
## 构建集群 ##
分别对六台做以下操作：
1. 查看容器ip
```
cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2      263989ddf7e2
```
2. 修改redis配置
```
vi /usr/redis/redis-4.0.6/redis.conf
找到 bind 127.0.0.1
改为 bind 172.17.0.2 #对应容器的ip
```
3. 启动redis
```
cd /usr/redis/redis-4.0.6/src
./redis-server ../redis.conf
```

启动六台后实现集群
```
./redis-trib.rb create --replicas 1 172.17.0.2:6379 172.17.0.3:6379 172.17.0.4:6379 172.17.0.5:6379 172.17.0.6:6379 172.17.0.7:6379  
```
## 测试redis集群 ##
```
./redis-cli  -c -h 172.17.0.3 -p 6379  
172.17.0.3:6379> set a 1
-> Redirected to slot [15495] located at 172.17.0.4:6379
OK
172.17.0.3:6379> get a
-> Redirected to slot [15495] located at 172.17.0.4:6379
"1"
172.17.0.4:6379> exit
```

## 遇到的错误 ##
1. -bash: make: command not found
解决：安装gcc 
```
yum -y install gcc automake autoconf libtool make
```
2. ERROR:  Loading command: install (LoadError) cannot load such file -- zlib
解决：需要依赖zlib工具
```
yum install -y zlib-devel
```
3. Could not create Makefile due to some reason, probably lack of necessary
解决：zlib需要安装到本地运行库
```
cd /usr/ruby/ruby-2.5.0/ext/zlib 
ruby ./extconf.rb
make
make install 
```
4. make: *** No rule to make target `/include/ruby.h', needed by `zlib.o'.  Stop
解决：修改makefile文件
```
sed -i 's/zlib.o: $(top_srcdir)\/include\/ruby.h/zlib.o: ..\/..\/include\/ruby.h/' Makefile 
make
make install
```
5. while executing gem ...(Gem:Exception) Unable to require openssl
解决：需要安装openssl
```
wget https://www.openssl.org/source/openssl-1.1.1-pre7.tar.gz
RUN mkdir /usr/openssl
RUN tar xzf /usr/software/openssl-1.1.1-pre7.tar.gz -C /usr/openssl
RUN cd /usr/openssl/openssl-1.1.1-pre7
./config -fPIC --prefix=/usr/local/openssl enable-shared  
./config -t  
make  
make install
还需要安装到本地库
cd /usr/ruby/ruby-2.5.0/ext/openssl
ruby extconf.rb  --with-openssl-include=/usr/local/openssl/include/ --with-openssl-lib=/usr/local/openssl/lib
make
make install
```
6. make: *** No rule to make target `/include/ruby.h', needed by `ossl.o'.  Stop
解决：修改makefile文件
```
sed -i 's/$(top_srcdir)/..\/../g' Makefile
make
make install
```