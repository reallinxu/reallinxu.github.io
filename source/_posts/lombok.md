---
title: "lombok"
tags:
  - lombok
id: 520
categories:
  - lombok
date: 2018-09-04 10:20:31
author: 
  - linxu
---
# 前言 #
“发明起源于懒”，当你在java开发中需要不停地重复编写set，get，tostring方法时，当你觉得这些代码很浪费时间，影响美观时，lombok你需要了解一下。

# 简介 #
lombok可以通过简单注解实现bean的set，get，tostring等方法，不再需要额外编写。
lombok常用注解如下：  

+ @Data 注解在类上；提供类所有属性的 getting 和 setting 方法，此外还提供了equals、canEqual、 hashCode、toString 方法
+ @Setter ：注解在属性上；为属性提供 setting 方法
+ @Setter ：注解在属性上；为属性提供 getting 方法
+ @Log4j ：注解在类上；为类提供一个 属性名为log 的 log4j 日志对象
+ @NoArgsConstructor ：注解在类上；为类提供一个无参的构造方法
+ @AllArgsConstructor ：注解在类上；为类提供一个全参的构造方法
+ @Cleanup : 可以关闭流
+ @Builder ： 被注解的类加个构造者模式
+ @Synchronized ： 加个同步锁
+ @SneakyThrows : 等同于try/catch 捕获异常
+ @NonNull : 如果给参数加个这个注解 参数为null会抛出空指针异常
+ @Value : 注解和@Data类似，区别在于它会把所有成员变量默认定义为private final修饰，并且不会生成set方法。

# IDE使用lombok #
## eclipse ##
1. 下载lombok，[click](https://projectlombok.org/download)
2. 双击lombok.jar直接安装，如果安装不了，将lombok放入到eclipse安装目录，编辑eclipse.ini，在最后一行加上 -javaagent:lombok.jar，重启eclipse。
3. 添加maven配置：  
  
``` java
	<dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<version>1.16.10</version>
	</dependency>
```

## idea ##
1. 安装lombok插件，File-->Settings-->plugins-->Browse Repositories，搜索lombok plugin，选择install，重启idea。
2. 如果上面下载不了，或者太慢，且本地有shadowsocks翻墙支持，可以在当前窗口进入HTTP Proxy Settings，选择Manual proxy configuration，设置如图：
	![Http Proxy](/imgs/lombokhttpproxy.JPG)
3. 添加maven配置：  
 
``` java
	<dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<version>1.16.10</version>
	</dependency>
```

