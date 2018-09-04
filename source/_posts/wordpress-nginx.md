---
title: " WordPress从Apache迁移到nginx"
id: 130
categories:
  - WordPress
date: 2018-01-29 15:02:48
tags:
---

1.停掉httpd当前服务，禁用重启  

	service httpd stop
	#开机重启后，apache服务不再启动p 
	chkconfig httpd off
2.安装nginx  

	yum -y install nginx
3.安装php-fpm  

	yum install php-fpm
	/etc/init.d/php-fpm start  #该目录下没有执行文件，采用直接下一步设置开机自启
	chkconfig php-fpm on
4.编辑nginx配置文件  

	vi /etc/nginx/conf.d/virtual.conf     #没有该文件，直接新建，内容如下
	#转发给wordpress网站
	server {
    	listen 80;
    	server_name www.reallinxu.com;        
    	access_log /var/log/nginx/aaa/access.log;   #access_log属于ngx_http_log_module的设置, 缺省level为info
    	error_log /var/log/nginx/aaa/error.log;     #error_log属于core module, 缺省的level是error 

    	location / {
             root /data/www/www.reallinxu.com;
             index index.php index.html index.htm;     #由于是PHP类型的动态页面为主，所以把index.php放在前面效率会更高些
             try_files $uri $uri/ /index.php?$args;   #普通php网站因为没有rewrite的话，这个不需要
    	}

    	error_page 404 /404.html;         #error_page errcode uri (也就是说出现了404错误，会请求/404.html)
    	location = /404.html {            #这是一个典型的location
             root /data/www/www.reallinxu.com;
    	}

    	error_page 500 502 503 504 /50x.html;
    	location = /50x.html {
             root /data/www/www.reallinxu.com;
    	}

    	# 这种写法可以防止把恶意程序伪装成.jpg之类的攻击，（其实有个更简单的方法，就是把php.ini中的cgi.fix_pathinfo=0，但有时候简单的修改cgi.fix_pathinfo会造成有的php脚本出错)
    	location ~ [^/]\.php(/|$) {
             root /data/www/www.reallinxu.com;
             fastcgi_split_path_info ^(.+?\.php)(/.*)$;
             if (!-f $document_root$fastcgi_script_name) {
                     return 404;
             }
             #try_files $uri =404;         #这个try_files说明：对于.php文件，直接执行$uri, 如果找不到这个$uri,直接给出404错误，（和 location / 定义不同！），主要是为了防止 伪装成图片的攻击  (目前看，最安全的方式，是用上面那一句话，官方推荐的）
             # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
             fastcgi_pass 127.0.0.1:9000;
             fastcgi_index index.php;
             fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
             include fastcgi_params;
    	}
    	location ~ /\.ht {
             deny all;
    	}

    	rewrite /wp-admin$ $scheme://$host$uri/ permanent;
 	}

5.新建日志目录并赋权限  

	mkdir -p /var/log/nginx/aaa
	chown -R nginx:adm /var/log/nginx/aaa
6.安装最新的php-mysql  

	yum update
	yum install php-mysql
7.启动nginx  

	service nginx start
8.此时访问可能会出现数据库错误  

	vi wp-config.php
	#修改以下部分
	define( 'WP_DEBUG', true );
再进行访问时，会出现具体错误，此时发现是数据库连接错误，本人是需要修改配置文件如下  

	define( 'DB_HOST', '127.0.0.1' );   #原为localhost
9.修改线程数，优化内存  

	vi /etc/php-fpm.d/www.conf   #修改如下
	最大线程数
	pm.max_children = 3
	初始线程数
	pm.start_servers = 1
	最小空余线程数
	pm.min_spare_servers = 1
	最大空余线程数
	pm.max_spare_servers = 1
	php_admin内存最大限制
	php_admin_value[memory_limit] = 128M
