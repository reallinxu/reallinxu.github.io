---
title: " 网站同时支持http和https访问"
id: 133
categories:
  - WordPress
date: 2018-01-29 15:08:53
tags:
---

1.在腾讯云申请SSL证书https://buy.cloud.tencent.com/ssl?fromSource=ssl，此时选择了个人配置DNS，具体使用方法请参照官网说明。

2.证书申请成功后，下载证书上传到服务器，将crt和key两个文件放到/usr/local/httpscrt文件夹中

3.修改nginx配置文件如下：  

	vi /etc/nginx/conf.d/virtual.conf     #没有该文件，直接新建，内容如下
	#转发给wordpress网站
	server {
    	listen 80;
    	listen 443 ssl;     #此处监听443端口，https默认443，下面注释ssl on同时支持http和https
    	server_name www.reallinxu.com;        
    	access_log /var/log/nginx/aaa/access.log;   #access_log属于ngx_http_log_module的设置, 缺省level为info
    	error_log /var/log/nginx/aaa/error.log;     #error_log属于core module, 缺省的level是error 
    	#ssl on;    #此处注释
    	ssl_certificate /usr/local/httpscrt/1_www.reallinxu.com_bundle.crt;  #证书位置
    	ssl_certificate_key /usr/local/httpscrt/2_www.reallinxu.com.key;   #key位置

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

4.重启nginx即可。