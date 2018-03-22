---
title: " WordPress TwentyFourteen页面不居中问题解决\t\t"
id: 60
categories:
  - WordPress
date: 2017-12-22 14:12:35
tags:
---

选择  仪表盘--&gt;外观--&gt;编辑

选择style.css，找到如下代码：
<pre class="lang:default decode:true">html, body, div, span, applet, object, iframe, h1, h2, h3, h4, h5, h6, p, blockquote, pre, a, abbr, acronym, address, big, cite, code, del, dfn, em, font, ins, kbd, q, s, samp, small, strike, strong, sub, sup, tt, var, dl, dt, dd, ol, ul, li, fieldset, form, label, legend, table, caption, tbody, tfoot, thead, tr, th, td {
	border: 0;
	font-family: inherit;
	font-size: 100%;
	font-style: inherit;
	font-weight: inherit;
	margin: 0;
	outline: 0;
	padding: 0;
	vertical-align: baseline;
}</pre>
将
<pre class="lang:default decode:true  ">	margin: 0;</pre>
改为
<pre class="lang:default decode:true ">	margin: auto;</pre>
更新文件即可居中。