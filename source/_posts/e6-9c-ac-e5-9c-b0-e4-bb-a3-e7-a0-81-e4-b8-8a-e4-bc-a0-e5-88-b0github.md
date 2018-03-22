---
title: " 本地代码上传到github\t\t"
id: 89
categories:
  - github
date: 2018-01-23 16:22:40
tags:
---

**第一步：建立git仓库**
cd到你的本地项目根目录下，执行git命令
<pre class="lang:default decode:true">git init</pre>
**第二步：将项目的所有文件添加到仓库中**
<pre class="lang:default decode:true">git add .</pre>
**如果想添加某个特定的文件，只需把.换成特定的文件名即可**

**第三步：将add的文件commit到仓库**
<pre class="lang:default decode:true">git commit -m "注释语句"</pre>

* * *

**第四步：去github上创建自己的Repository**

**第五步：重点来了，将本地的仓库关联到github上**
<pre class="lang:default decode:true">git remote add origin https://github.com/reallinxu/spring-boot-dubbo</pre>
**后面的https链接地址换成你自己的仓库url地址**

**第六步：上传github之前，要先pull一下，执行如下命令：**
<pre class="lang:default decode:true">git pull origin master</pre>
**第七步，也就是最后一步，上传代码到github远程仓库**
<pre class="lang:default decode:true ">git push -u origin master</pre>