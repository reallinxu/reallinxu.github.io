---
title: "nginx入门"
tags:
  - nginx
id: 1201
categories:
  - nginx
date: 2018-12-01 10:20:31
author: 
  - linxu
---

# nginx入门 #

## 简介 ##
Nginx是一个很强大的高性能Web和反向代理服务。由俄罗斯的程序设计师lgor Sysoev(塞索耶夫)所开发，供俄国大型的入口网站及搜索引擎Rambler使用。其特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好。
nginx的崛起要感谢wordpress(WordPress是使用PHP语言开发的博客平台，用户可以在支持PHP和MySQL数据库的服务器上架设属于自己的博客网站)，2008年，WordPress需要搬到软件上，这样才符合其开源哲学理念。当时选择的是全球最有名的Web服务器——Apache，当工程师开始修改软件安装设置的时候，Apache竟然死机了，尤其是在WordPress最繁忙的时候。所以他们最后撤除了不稳定的Apache，将公司放在一个那时候根本不起眼的一个叫作Nginx的开源项目的赌注上。五年之后，WordPress仍然在Nginx上安稳运行，这也使后来很多其它公司都如法炮制。

## 优点 ##
* 高并发连接：官方测试能够支撑5W并发连接，在实际生产中可跑到2~3W并发连接数。
* 内存消耗少：在3万并发连接下，开启的10个Nginx 进程才消耗150M内存 （15M*10=150M）。
* 配置文件非常简单：风格跟程序一样通俗易懂。
* 成本廉价：Nginx为开源软件，可以免费使用。
* 支持Rwrite重写：能够根据域名、URL的不同，将HTTP请求分发到不同的后端服务器群
* 内置健康检查功能：在3万并发连接下，如果Nginx Proxy后端的某台Web服务器宕机了，不会影响前端的访问。
* 节省带宽：支持GZIP压缩，可以添加浏览器本地缓存的Header头
* 稳定性高 ：用于反向代理，宕机的概率微乎其微。

## 主要功能 ##
1. 反向代理
* 正向代理：某些情况下，代理我们用户去访问服务器，需要用户手动的设置代理服务器的ip和端口号。
* 反向代理：是用来代理服务器的，代理我们要访问的目标服务器。代理服务器接受请求，然后将请求转发给内部网络的服务器(集群化)，并将从服务器上得到的结果返回给客户端，此时代理服务器对外就表现为一个服务器。  

Nginx在反向代理上，提供灵活的功能，可以根据不同的正则采用不同的转发策略将不同的请求代理到不同的服务。

2. 负载均衡
高并发情况下将数据流量分摊到多个服务器执行，减轻每台服务器的压力，多台服务器(集群)共同完成工作任务，从而提高了数据的吞吐量。Nginx可使用的负载均衡策略有：轮询（默认）、权重、ip_hash、url_hash(第三方)、fair(第三方)

3. 动静分离
Nginx提供的动静分离是指把动态请求和静态请求分离开，合适的服务器处理相应的请求，使整个服务器系统的性能、效率更高。Nginx可以根据配置对不同的请求做不同转发，这是动态分离的基础。静态请求对应的静态资源可以直接放在Nginx上做缓冲，更好的做法是放在相应的缓冲服务器上。动态请求由相应的后端服务器处理。

## 配置示例 ##
``` config
server {
    listen       443 ssl;
    server_name  anxin.sitproduct.techbao.cn;
    ssl on;  #开启ssl证书
	ssl_certificate "/home/project/httpscrt/1_anxin.sitproduct.techbao.cn_bundle.crt";  #证书位置
     ssl_certificate_key "/home/project/httpscrt/2_anxin.sitproduct.techbao.cn.key";   #key位置

    location / {
    proxy_pass http://127.0.0.1:28081;  #转发到本地28081端口
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}

server {
	listen 8081;
    server_name blog.techbao.cn;
    root /home/project/bbs/upload;   #静态网站目录
	location / {
			
            try_files $uri @redirect;  # 找不到uri(/index等)则通过@redirect转发
    	}      
    }

	location @redirect{
     	rewrite ^/(.*)$   http://www.baidu.com;
    }
}
```

## 小经验 ##
1.nginx监听不能监听服务器已经使用的端口
2.如果某个端口下只有一个server_name的时候，所有访问该端口的请求，不管server_name是什么，都由该server处理。


## jenkins简介 ##
Jenkins是一个开源软件项目，是基于Java开发的一种持续集成工具，用于监控持续重复的工作，旨在提供一个开放易用的软件平台，使软件的持续集成变成可能。 
可以新建maven任务或者流水线任务等，maven任务通过插件通过maven来进行打包测试等，流水线任务可以通过自己的脚本，自由的定义命令执行顺序，完全由脚本控制。