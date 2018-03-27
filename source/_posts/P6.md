---
title: " 更新本地代码到github\t\t"
id: 93
categories:
  - github
date: 2018-01-23 16:28:47
tags:
---

**第一步：查看当前的git仓库状态，可以使用git status**

<pre class="lang:default decode:true">git status</pre>

**第二步：更新全部**

<pre class="lang:default decode:true">git add *</pre>

**第三步：接着输入git commit -m "更新说明"**

<pre class="lang:default decode:true">git commit -m "更新说明"</pre>

**第四步：先git pull,拉取当前分支最新代码**

<pre class="lang:default decode:true">git pull</pre>

**第五步：push到远程master分支上**

<pre class="lang:default decode:true  ">git push origin master</pre>