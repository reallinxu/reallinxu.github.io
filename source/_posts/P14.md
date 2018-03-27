---
title: " vi文件Found\_a\_swap\_file\_by\_the\_name\_\t\t"
id: 105
categories:
  - 杂项
date: 2018-01-24 18:59:56
tags:
---

在vi文件时出现  Found a swap file by the name <span class="string">".Test.java.swp"</span>   提示，是因为之前vi异常中断.

ls -a可以查看隐藏文件*.swap

rm *.swap即可