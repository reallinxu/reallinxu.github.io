---
title: " Spring-batch restart重拉机制\t\t"
id: 64
categories:
  - Spring-batch
date: 2017-12-22 14:27:28
tags:
---

对于Spring-batch中的Reader-processor-writer模式，例如
<pre class="lang:default decode:true">&lt;batch:step id="test-01"&gt;
	&lt;batch:tasklet&gt;
		&lt;batch:chunk reader="testR" processor="testP" commit-interval="1"&gt;
			&lt;batch:writer&gt;
				&lt;bean class="com.test.TestW" /&gt;
			&lt;/batch:writer&gt;
		&lt;/batch:chunk&gt;
	&lt;/batch:tasklet&gt;
&lt;/batch:step&gt;</pre>
如果reader读取为两条数据，第二条数据异常断批，由于commint-interval为1，第一条会执行processor和writer，且成功commit，第二条Processor会回滚，writer不会执行。当restart时，会从第二条数据重新处理，第一条已处理不会再重复处理。

如果此时将
<pre class="lang:default decode:true ">&lt;batch:chunk reader="testR" processor="testP" commit-interval="1"&gt;</pre>
改为
<pre class="lang:default decode:true ">&lt;batch:chunk reader="testR" processor="testP" commit-interval="2"&gt;</pre>
这时，reader读取两条数据，第二条数据异常断批，则两条数据都会回滚掉，restart时会重新执行。

以上，目前已验证Reader-processor-writer模式重拉不会导致数据重复处理。