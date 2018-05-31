---
title: "Docker入门及基础命令"
tags:
  - Docker
id: 101
categories:
  - Docker
date: 2018-05-16 10:20:31
author: 
  - linxu
---
![](https://i.imgur.com/0p0flPx.jpg)
# Docker入门 #  
## Docker介绍 ##
>关于 Docker 是什么，有个著名的隐喻：集装箱。但是它却起了个“码头工人”（docker 的英文翻译）的名字。这无疑给使用者很多暗示：“快来用吧！用了 Docker，就像世界出现了集装箱，这样你的业务就可以随意的、无拘无束的运行在任何地方（Docker 公司的口号：Build，Ship，and Run Any App，Anywhere），于是码头工人就基本都可以下岗了。”但是人们往往忽略了一个问题，隐喻的好处是方便人理解一个不容易理解的概念，但并不能解释其概念本身。 　　　　　　　　　　　　　　　　　　 　　—————————邹立巍《什么是 Docker?》

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。

Docker主要关键词是容器和镜像，可以从仓库中获取所需要的镜像，通过镜像创建容器。每一个容器都相当于一个完全独立的虚拟机(Docker不是虚拟机)，但是所占用内存消耗和启动时间等都远远小于虚拟机。在容器中可以根据所需安装软件，部署应用服务等。

## Docker与Vm对比 ##
![](https://i.imgur.com/Vx4ilht.jpg)

图中可以看出docker不需要Hypervisor实现硬件资源虚拟化，运行在docker容器上的程序直接使用的都是实际物理机的硬件资源。因此在CPU、内存利用率上docker将会在效率上有优势。docker利用的是宿主机的内核，而不需要Guest OS,因此新建一个docker容器只需要几秒钟。docker不需要增加Guest OS，所以在一台物理机上我们可以很容易建立成百上千的容器，而只能建立几个虚拟机。

## 安装Docker ##
docker的安装非常简单，centOs中直接使用  

	yum install docker
即可安装。

## Docker基础命令 ##
1. 启动docker  
	```
	service docker start    	#启动
	service docker stop 		#停止
	service docker restart 	 	#重启
	```
2. 运行创建容器
	```
	sudo docker run -i -t ubuntu /bin/bash
	-i 保证容器中STDIN是开启的。STDIN是标准输入，一般指键盘输入到缓冲区里的东西。
	-t 为创建的容器分配一个伪tty终端。
	```
3. 安装Vim  
	```
	apt-get update && apt-get install vim #容器中执行
	```
4. 退出容器
	```
	exit
	```
5. 查看容器列表
	```
	docker ps     		#列出所有运行中的容器
	docker ps -a  		#列出所有容器
	docker PS -l  		#列出最后一次运行的容器
	docker PS -n x    	#列出最后一次运行的x个容器
	```
6. 创建指定名称的容器
	```
	sudo docker run --name reallinxu -i -t ubuntu /bin/bash
	```
7. 删除容器
	```
	docker rm reallinxu  #可以通过容器名称、CONTAINER ID删除
	```
8. 启动容器
	```
	docker start b74cf5d6a711   #启动一个已停止的容器
	docker restart b74cf5d6a711 #重新启动容器
	```
9. 附着容器会话
	```
	docker attach 48db85aa8845
	```
10. 创建守护容器
	```
	sudo docker run --name daemon_aaa -d ubuntu /bin/sh -c "while true;do echo hello world;sleep 1;done"
	#-d docker会让容器后台运行
	#-c 设置容器CPU权重，在CPU共享场景使用
	```
11. 获取容器日志  
	```
	sudo docker logs daemon_aaa 		#输出最后几条log
	sudo docker logs -f daemon_aaa		#动态监控log
	docker logs --tail 10 daemon_aaa	#获取日志最后10行
	docker logs --tail 0 -f daemon_aaa	#跟踪容器最新log
	```
12. 查看容器内进程
	```
	sudo docker top daemon_aaa  
	```
13. 在容器内部运行进程
	```
	sudo docker exec -d daemon_aaa touch /etc/new_config_file
	#-d 表示需要运行一个后台进程
	sudo docker exec -t -i daemon_aaa /bin/bash
	#创建一个新的bash会话，通过docker exec可以在运行中的容器中进行操作管理
	```
14. 停止守护容器
	```
	sudo docker stop daemon_aaa #向容器发送SIGTERM信号
	sudo docker kill daemon_aaa #向容器发送SIGKILL信号,可以快速停止
	```
15. 自动重启容器
	```
	sudo docker run --restart=always --name daemon_ccc -d ubuntu /bin/sh -c "while true;do echo hello world;sleep 1;done"
	#--restart=always 无论退出代码是什么都会自动重启
	#--restart=on-failure 退出代码非0时重启
	#--restart=on-failure:5 非0时重启，最多重启5次
	```
16. 获取容器信息
	```
	docker inspect daemon_aaa   #查看容器详细信息
	docker inspect --format='{{.State.Running}}' daemon_aaa #查看指定属性，区分大小写
	docker inspect --format='{{.State.Running}} {{.Name}}'  daemon_aaa daemon_bbb #查看多个容器多个属性
	#所有容器都放在/var/lib/docker/containers文件夹下
	```
17. 删除所有容器小技巧
	```
	docker rm `docker ps -a -q`   #返回所有容器id传给rm
	```

## 使用docker镜像和仓库 ##  
1. 拉取指定仓库的镜像，默认为latest
	```
	docker pull ubuntu
	```
2. 查看本地镜像
	```
	docker images
	```
3. 镜像标签
	```
	docker run -t -u --name haha ubuntu:12.04 /bin/bash
	#通过镜像ubuntu:12.04启动一个容器，不指定标签则为latest
	```
4. docker hub上查找镜像
	```
	docker search puppet
	```
5. docker commit方法构建
	dockerhub注册账号：https://hub.docker.com/
	登录dockerhub
	```
	docker login #登录dockerhub,个人认证信息保存在$HOME/.dockercfg
	sudo docker commit 9de0d8b43b0f reallinxu/ubuntu1  #提交一个镜像
	docker commit -m="test" --author="reallinxu" 1d61b09d590d reallinxu/hah:hah
	#-m 提交信息
	#--author 作者
	#:hah 标签
	#docker commit提交的只是创建的容器与容器的当前状态有差异的部分，这使得更新非常轻量
	```
6. dockerfile方法构建
	创建文件
	```
	mkdir static_web
	cd static_web
	touch Dockerfile
	vi Dockerfile
	# Version: 0.0.1
	FROM ubuntu:14.04  #从指定镜像创建容器
	MAINTAINER reallinxu "xxxxx@qq.com" #指定用户名和作者
	RUN apt-get update #更新apt-get
	RUN apt-get install -y nginx  #安装ngnix
	RUN echo 'Hi,I am in your container' >/usr/share/nginx/html/index.html
	EXPOSE 80 #向外部公开80端口，可以公开多个端口
	```
7. docker build构建镜像
	```
	sudo docker build -t="reallinxu/static_web" .	#默认latest标签
	sudo docker build -t="reallinxu/static_web:v1" . #指定标签
	sudo docker build -t="reallinxu/static_web:v1" gitURL  #可以通过git根目录下文件构建
	#build时会按照文件顺序一步步执行
	```
8. 镜像推送到dockerhub
	```
	sudo docker commit 9de0d8b43b0f reallinxu/ubuntu1:test #构建镜像的时候最好加上tag，不加可能会报错
	sudo docker push 9de0d8b43b0f reallinxu/ubuntu1:test #推送到dockerhub
	```
9. 删除镜像
	```
	docker rmi reallinxu/ubuntu1:test
	docker rmi `docker images -a -q`  #小技巧，返回所有镜像ID，再移除
	```

# Docker进阶 #
## 启动阿里云加速 ##
docker的镜像仓库在国外，下载会很慢，启用阿里云加速。在/etc/docker目录下创建daemon.json文件，添加如下内容  
	```
	{
		"registry-mirrors": ["https://almtd3fa.mirror.aliyuncs.com"]
	}
	#https://almtd3fa.mirror.aliyuncs.com为阿里云的加速地址。修改后，重启docker
	systemctl daemon-reload
	service docker restart
	```
## Docker中运行jar ##
1. 下载java8镜像：
	```
	docker pull frolvlad/alpine-oraclejdk8
	```
2. 创建一个简单的spring-boot项目，开放8080端口，生成可执行的jar文件。
3. 将demo-0.0.1-SNAPSHOT.jar放到/usr下，执行命令创建容器并执行jar
	```
	docker run -d -p 8080:8080 -v /usr/demo-0.0.1-SNAPSHOT.jar:/usr/demo-0.0.1-SNAPSHOT.jar --name springboot frolvlad/alpine-oraclejdk8 java -jar /usr/demo-0.0.1-SNAPSHOT.jar
	#-d 容器后台运行
	#-p 主机端口与容器端口映射
	```
4. 浏览器中输入ip:8080验证是否启动成功。

## docker宿主机与容器文件交互 ##  

1. 从主机复制到容器  
	```
	sudo docker cp host_path containerID:container_path
	```
   从容器复制到主机
	```
	sudo docker cp containerID:container_path host_path
	```
2. 挂载数据卷
	```
	docker run -v /path/to/hostdir:/mnt $container(image name or image id)  在容器内拷贝  
	cp /mnt/sourcefile /path/to/destfile 
	```