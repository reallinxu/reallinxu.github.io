---
title: " Spring-batch restart重拉机制"
id: 64
categories:
  - Spring-batch
date: 2017-12-22 14:27:28
tags:
---

对于Spring-batch中的Reader-processor-writer模式，例如   

	<batch:step id="test-01">
		<batch:tasklet>
			<batch:chunk reader="testR" processor="testP" commit-interval="1">
				<batch:writer>
					<bean class="com.test.TestW">
				<batch:writer>
			<batch:chunk>
		<batch:tasklet>
	<batch:step>
如果reader读取为两条数据，第二条数据异常断批，由于commint-interval为1，第一条会执行processor和writer，且成功commit，第二条Processor会回滚，writer不会执行。当restart时，会从第二条数据重新处理，第一条已处理不会再重复处理。

如果此时将  

	<batch:chunk reader="testR" processor="testP" commit-interval="1">
改为  

	<batch:chunk reader="testR" processor="testP" commit-interval="2">
这时，reader读取两条数据，第二条数据异常断批，则两条数据都会回滚掉，restart时会重新执行。

以上，目前已验证Reader-processor-writer模式重拉不会导致数据重复处理。