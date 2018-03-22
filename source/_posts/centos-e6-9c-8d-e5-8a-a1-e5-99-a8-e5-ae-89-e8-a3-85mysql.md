---
title: " centos服务器安装mysql\t\t"
id: 75
categories:
  - CentOS
date: 2018-01-12 16:14:23
tags:
---

1.安装mysql
<pre class="lang:default decode:true ">[root@localhost ~]rpm -qa | grep mysql #搜索指定rpm包是否安装。
[root@localhost ~]yum install mysql #安装客户端
[root@localhost ~]yum install mysql-server #安装服务端(此时安装失败，需要下载)
[root@localhost ~]wget http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
[root@localhost ~]rpm -ivh mysql-community-release-el7-5.noarch.rpm
[root@localhost ~]yum install mysql-community-server #安装server
[root@localhost ~]yum install mysql-devel #服务端安装好后，需要安装 ，包含了所需要的库和文件，如果需要编译其他Mysql客户程序，必须要安装</pre>
2.服务器安装完成后，通过本机DbVisualizer连接时会连接不上，此时需要赋予权限
<pre class="lang:default decode:true ">[root@localhost ~]grant all on *.* to test@'%' identified by '123456';
*.*，表示赋予用户操作服务器上所有数据库所有表的权限。
%，表示所有ip
grant 权限1,权限2,…权限n on 数据库名称.表名称 to 用户名@用户地址 identified by ‘密码’</pre>
3.删除权限
<pre class="lang:default decode:true">[root@localhost ~]grant all on *.* to test@'%' identified by '123456';
*.*，表示赋予用户操作服务器上所有数据库所有表的权限。
%，表示所有ip
revoke 权限1,权限2,…权限n on 数据库名称.表名称 from 用户名@用户地址</pre>
&nbsp;