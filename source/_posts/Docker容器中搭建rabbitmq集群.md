---
title: "Docker容器中搭建rabbitmq集群"
tags:
  - noname
id: 522
categories:
  - noname
date: 2018-05-29 18:00:00
author: 
  - linxu
---

# RabbitMq简介 #
>AMQP，即Advanced Message Queuing Protocol，高级消息队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，反之亦然。AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。

RabbitMQ是一个开源的AMQP实现，服务器端用Erlang语言编写，支持多种客户端。使用Rabbitmq可以使程序间不再需要彼此调用，只需要将消息放入消息队列中，通过对队列的监听，对消息进行对应处理，而且消息队列在当消息接收方服务忙或不可用时，提供了一个消息暂存的功能，等服务可用时，继续对消息进行处理。

# RabbitMq几个基本概念 #
1. Connection、Channel
	Connection是RabbitMQ的socket链接，它封装了socket协议相关部分逻辑。Channel是可以定义Queue、定义Exchange、绑定Queue与Exchange、发布消息等的最重要的一个接口。
2. Queue
	Queue（队列）是RabbitMQ的内部对象，生产者将消息投递到queue中，消费者从queue中获取消息。多个消费者可以订阅同一个queue，这时queue的消息会均摊给多个消费者。
3. Message acknowledgment
	消费者收到queue消息后，为了避免没有处理完成就宕机导致消息丢失，在消费者完成后会发送一个回执给rabbitmq，rabbit收到消息回执(ack)才会移除消息,如果rabbitmq检测到消费者已断开但是仍未收到回执，则会将消息发送给其他消费者。
4. Message durability
	如果需要在rabbitmq重启时也不丢失消息，可以将queue和message设置为可持久化的。这种情况可能存在还没有持久化rabbitmq就停的小概率事件，这就需要rabbitmq的事务。
5. Prefetch count
	设置prefetchCount来限制Queue每次发送给每个消费者的消息数，比如我们设置prefetchCount=1，则Queue每次给每个消费者发送一条消息，消费者处理完这条消息后Queue会再给该消费者发送一条消息。
6. Exchange
	生产者通过Exchange将消息投入queue中，由Exchange将消息路由到一个或多个Queue中(或者丢弃)。
7. routing key
	生产者可以在发送消息给Exchange时，通过指定routing key来决定消息流向哪里。
8. Binding
	RabbitMQ中通过Binding将Exchange与Queue关联起来。
9. Binding key
	binding key与routing key相匹配时，消息将会被路由到对应的Queue中。
10. Exchange Types
	fanout：所有发送到该Exchange的消息路由到所有与它绑定的Queue中。
	direct：消息路由到那些binding key与routing key完全匹配的Queue中。
	topic：topic与direct相似，可以使用*与#做模糊匹配，*匹配一个单词，#匹配多个单词（可以是零个）
	headers：不依赖于key的规则，而是根据发送的消息内容中的headers属性进行匹配。在绑定Queue与Exchange时指定一组键值对，当消息发送到Exchange时，RabbitMQ会取到该消息的headers（也是一个键值对的形式），对比其中的键值对是否完全匹配Queue与Exchange绑定时指定的键值对。
11. RPC
	客户端发送请求消息时，在消息的属性（MessageProperties）中设置两个值replyTo（一个Queue名称，用于告诉服务器处理完成后将通知我的消息发送到这个Queue中）和correlationId（此次请求的标识号，服务器处理完成后需要将此属性返还，客户端将根据这个id了解哪条请求被成功执行了或执行失败）。

# Docker容器搭建rabbitmq集群 #
## Dockerfile文件 ##
将rabbitmq-signing-key-public.asc，rabbitmq-server-3.3.5-1.noarch.rpm放入Dockerfile同目录
```
from centos
MAINTAINER reallinxu

RUN rpm -ivh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
RUN yum install -y erlang
COPY rabbitmq-signing-key-public.asc /usr/software/rabbitmq-signing-key-public.asc
COPY rabbitmq-server-3.3.5-1.noarch.rpm /usr/software/rabbitmq-server-3.3.5-1.noarch.rpm
RUN rpm --import /usr/software/rabbitmq-signing-key-public.asc
RUN rpm -ivh /usr/software/rabbitmq-server-3.3.5-1.noarch.rpm  --force --nodeps
RUN rabbitmq-plugins enable rabbitmq_management
RUN chkconfig --level 3 rabbitmq-server on
RUN sed -i 's/PID_FILE=\/var\/run\/rabbitmq\/pid/&\nexport RABBITMQ_MNESIA=\/rabbitmq\/mnesia/' /etc/init.d/rabbitmq-server
RUN chmod 644 /etc/rabbitmq/enabled_plugins
RUN rm -rf /usr/software/rabbitmq-signing-key-public.asc
RUN rm -rf /usr/software/rabbitmq-server-3.3.5-1.noarch.rpm
EXPOSE 8080 15672 5672
```
## 构建镜像 ##
```
docker build -t="reallinxu/rabbit:v1" .
```
## 创建容器 ##
### 创建第一个容器 ###
```
docker run -p 15672:15672 -i -t --name rabbit1 reallinxu/rabbit:1 
#查看hosts
cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.3      c0543dbe04fe
```

### 创建后续容器 ###
因为要搭建集群，后面的hosts文件需要配置第一台的配置，这时候我们使用-v进行挂载容器，将/etc/hosts指定使用宿主机的/user/hosts文件。
```
#/user/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.3      c0543dbe04fe
```
创建容器
```
docker run -i -t -p 15670:15672 --name rabbit2 -v /user/hosts:/etc/hosts reallinxu/rabbit:1 
```

### 实现集群 ###
第一个容器中启动rabbitmq
```
rabbitmq-server -detached # -detached后台运行
rabbitmqctl add_user admin admin
rabbitmqctl set_user_tags admin administrator
```
将/var/lib/rabbitmq的.erlang.cookie文件拷贝到宿主机中
```
sudo docker cp c0543dbe04fe:/var/lib/rabbitmq /user
```
将第一个容器中/root下cookie覆盖
```
docker cp /user/rabbitmq/.erlang.cookie c0543dbe04fe:/root
```
将宿主机的cookie文件覆盖到其他容器中：
```
docker cp /user/rabbitmq/.erlang.cookie 780f23c07ef7:/root
docker cp /user/rabbitmq/.erlang.cookie 780f23c07ef7:/var/lib/rabbitmq
```
更改.erlang.cookie的访问权限为400
```
chmod 400 /var/lib/rabbitmq/.erlang.cookie
```
更改.erlang.cookie的文件所有人为rabbitmq
```
chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie
```
启动后续容器并做如下操作：
```
rabbitmq-server -detached
rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@c0543dbe04fe
rabbitmqctl start_app
```
查看rabbitmq后台界面，可以发现集群搭建成功。不过因为直接挂载，导致容器自身映射端口并没有生效，所以在后面需要将自己的hosts配置进去，在hosts文件后面追加下面语句
```
172.17.0.3	780f23c07ef7
```
容器ID请按照自己创建容器ID来替换。

# 小知识 #
## sed命令 ##
sed -i 's/指定的字符/要插入的字符&/'  文件  #前插
sed -i 's/指定的字符/&要插入的字符/'  文件  #后插
sed 's/原字符串/替换字符串/' 文件			#替换
sed 's/原字符串/替换字符串/g' 文件			#替换所有
\n 换行符  \转义

## 查看容器端口映射 ##
docker port <NAME>

## 失败解决 ##
安装rpm包如果出现“Exiting on user Command”的错误，请在yum命令后加上“-y”选项

## systemctl ##
不可以使用，尽量避免，网上说可以CMD或者entrypoint设置为/usr/sbin/init，但是创建容器时一直没有反应，所以放弃了