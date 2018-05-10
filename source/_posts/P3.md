---
title: " Dubbo+Zookeeper+Spring-boot+dubbo-simple 搭建入门"
id: 80
categories:
  - Dubbo
date: 2018-01-23 16:16:32
tags:
---

# 安装Zookeeper #
	[root@linxu ~]#  wget http://mirrors.shu.edu.cn/apache/zookeeper/zookeeper-3.5.3-beta/zookeeper-3.5.3-beta.tar.gz
	[root@linxu ~]#  tar -zxvf zookeeper-3.5.3-beta.tar.gz  #此时可能会报错使用 
	[root@linxu ~]#  tar -xvf zookeeper-3.5.3-beta.tar.gz 
	[root@linxu ~]#  mv zookeeper-3.5.3-beta /usr/local/zookeeper #没有目录请先mkdir
	[root@linxu ~]#  cd /usr/local/zookeeper/zookeeper-3.5.3-beta/conf
	[root@linxu conf]# cp zoo_sample.cfg zoo.cfg
	[root@linxu conf]# vi zoo.cfg 
	修改配置对应项
	dataDir=/usr/local/zookeeper/zookeeper-3.5.3-beta/data
	dataLogDir=/usr/local/zookeeper/zookeeper-3.5.3-beta/logs
	[root@linxu conf]# vi /etc/profile
	末尾添加
	export ZOOKEEPER_HOME=/usr/local/zookeeper/zookeeper-3.5.3-beta/
	export PATH=$ZOOKEEPER_HOME/bin:$PATH
	export PATH
	[root@linxu conf]# source /etc/profile #使生效
	[root@linxu conf]# zkServer.sh start   #启动
	[root@linxu conf]# zkServer.sh status  #查看状态
	[root@linxu conf]# zkServer.sh stop    #停止
	[root@linxu conf]# zkServer.sh restart #重启


# 部署dubbo-admin #

从github上下载dubbo源码，编译dubbo-admin(最新版已没有dubbo-admin，下载历史版本)

dubbo地址：https://github.com/alibaba/dubbo/tree/dubbo-2.5.8

编译：mvn package -Dmaven.test.skip=true

编译成功后将target目录下的war包放到tomcat下webapps中，启动tomcat即可访问http://localhost:8080/dubbo-admin-2.5.8/

# 代码实现 #
本地使用spring-boot创建提供者（provider）和消费者(consume) 

代码：https://github.com/reallinxu/spring-boot-dubbo

分别启动provider和consume，查看dubbo主页，可以看到提供者和消费者。

PS：provider和consume的接口位置必须一致

# 搭建dubbo-simple监控 #

从github上下载dubbox源码，编译dubbo-monitor-simple

dubbox地址：https://github.com/reallinxu/dubbox

编译：mvn package -Dmaven.test.skip=true

编译成功后将dubbo-monitor-simple-2.8.4-assembly.tar.gz放入服务器  

	[root@linxu dubbo-simple]# tar -zxvf dubbo-monitor-simple-2.8.4-assembly.tar.gz
	[root@linxu bin]# cd /usr/local/dubbo-simple/dubbo-monitor-simple-2.8.4/conf
	修改dubbo.properties
	dubbo.container=log4j,spring,registry,jetty
	dubbo.application.name=simple-monitor
	dubbo.application.owner=
	#dubbo.registry.address=multicast://224.5.6.7:1234
	dubbo.registry.address=zookeeper://192.168.43.163:2181 #Zookeeper地址
	#dubbo.registry.address=redis://127.0.0.1:6379
	#dubbo.registry.address=dubbo://127.0.0.1:9090
	dubbo.protocol.port=7070
	dubbo.jetty.port=7072  #jetty端口，避免冲突
	dubbo.jetty.directory=${user.home}/monitor
	dubbo.charts.directory=${dubbo.jetty.directory}/charts
	dubbo.statistics.directory=${user.home}/monitor/statistics
	dubbo.log4j.file=logs/dubbo-monitor-simple.log
	dubbo.log4j.level=WARN</pre>
	启动dubbo-simple
	[root@linxu ~]# cd /usr/local/dubbo-simple/dubbo-monitor-simple-2.8.4/bin
	[root@linxu bin]# sh start.sh
启动成功后登陆访问http://192.168.43.163:7072 (端口为jetty端口)

如果查看不到监控提供者

com.alibaba.dubbo.monitor.MonitorService

请核对dubbo.properties zookeeper地址和端口是否正确
