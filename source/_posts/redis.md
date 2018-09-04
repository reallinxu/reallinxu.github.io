---
title: " redis入门"
id: 102
categories:
  - Redis
date: 2018-01-24 18:53:33
tags:
---

# 安装redis #
	[root@linxu ~]# wget http://download.redis.io/releases/redis-4.0.6.tar.gz
	[root@linxu ~]# tar xzf redis-4.0.6.tar.gz 
	[root@linxu ~]# mkdir /usr/local/redis
	[root@linxu ~]# mv redis-4.0.6 /usr/local/redis
	[root@linxu ~]# cd /usr/local/redis/redis-4.0.6  
	[root@linxu redis-4.0.6]# make
	[root@linxu redis]# cd src
	[root@linxu src]# ./redis-server  #运行 
	[root@linxu src]# ./redis-server redis.conf  #使用指定配置文件运行
# 测试redis #
	第一个窗口:
	[root@linxu ~]# cd /usr/local/redis/redis-4.0.6/src
	[root@linxu src]# ./redis-server redis.conf  #使用指定配置文件运行
	第二个窗口:
	[root@linxu ~]# cd /usr/local/redis/redis-4.0.6/src
	[root@linxu src]# ./redis-cli
	127.0.0.1:6379>set test hahaah  
	OK
	127.0.0.1:6379>get test
	"hahaah"

# 利用jedis测试java连接redis #
	//pom.xml中添加
	<dependency>
    	<groupId>redis.clients<groupId>
    	<artifactId>jedis<artifactId>
    	<version&gt;2.9.0<version>
	<dependency>
测试代码：
	package com.redis.demo;

	import redis.clients.jedis.Jedis;

	public class TestRedis {
    	public static void main(String[] args) {
        	//连接本地的 Redis 服务
        	Jedis jedis = new Jedis("192.168.43.163",6379);
        	System.out.println("连接成功");
        	//查看服务是否运行
        	System.out.println("服务正在运行: "+jedis.ping());
    	}
	}
运行时报异常：  

	Exception in thread "main" redis.clients.jedis.exceptions.JedisDataException: DENIED Redis is running in protected mode because protected mode is enabled, no bind address was specified, no authentication password is requested to clients. In this mode connections are only accepted from the loopback interface. If you want to connect from external computers to Redis you may adopt one of the following solutions: 1) Just disable protected mode sending the command 'CONFIG SET protected-mode no' from the loopback interface by connecting to Redis from the same host the server is running, however MAKE SURE Redis is not publicly accessible from internet if you do so. Use CONFIG REWRITE to make this change permanent. 2) Alternatively you can just disable the protected mode by editing the Redis configuration file, and setting the protected mode option to 'no', and then restarting the server. 3) If you started the server manually just for testing, restart it with the '--protected-mode no' option. 4) Setup a bind address or an authentication password. NOTE: You only need to do one of the above things in order for the server to start accepting connections from the outside.
此时需要修改配置文件redis.conf，修改如下
	1.注释掉bind
	#bind 127.0.0.1
	2.protected-mode yes
	改为
	protected-mode no
	重新启动redis即可
