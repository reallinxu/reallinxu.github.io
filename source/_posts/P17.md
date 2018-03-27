---
title: " 将本地代码传入github\t\t"
id: 82
categories:
  - CentOS
tags:
---

**第一步：建立git仓库**
cd到你的本地项目根目录下，执行git命令

    git init`</pre>
    **第二步：将项目的所有文件添加到仓库中**
    <pre class="prettyprint">`git <span class="hljs-built_in">add</span> .`</pre>
    **如果想添加某个特定的文件，只需把.换成特定的文件名即可**

    **第三步：将add的文件commit到仓库**
    <pre class="prettyprint">`git <span class="hljs-operator"><span class="hljs-keyword">commit</span> -m <span class="hljs-string">"注释语句"</span></span>