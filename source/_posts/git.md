---
title: "git常用命令"
id: 93
categories:
  - Git
date: 2018-01-23 16:28:47
tags:
---
# Note #
此处为git使用过程中的一些常用命令记录汇总。

## 初始化绑定github并提交 ##
** 第一步：建立git仓库 **  
cd到你的本地项目根目录下，执行git命令  

	git init  

** 第二步：将项目的所有文件添加到仓库中 **  

	git add .
	如果想添加某个特定的文件，只需把.换成特定的文件名即可**

** 第三步：将add的文件commit到仓库 **  

	git commit -m "注释语句"
 
** 第四步：重点来了，将本地的仓库关联到github上 **  

	git remote add origin https://github.com/reallinxu/spring-boot-dubbo
	后面的https链接地址换成你自己的仓库url地址  

** 第五步：同步仓库代码(新初始仓库则不需要这步) **  

	git pull origin master  

** 第六步，也就是最后一步，上传代码到github远程仓库 **  

	git push -u origin master

## 更新代码到github ##
** 第一步：查看当前的git仓库状态，可以使用git status **

	git status

** 第二步：更新全部 **

	git add *

** 第三步：接着输入git commit -m "更新说明" **

	git commit -m "更新说明"

** 第四步：先git pull,拉取当前分支最新代码 **

	git pull

** 第五步：push到远程master分支上 **

	git push origin master

## 创建本地分支，合并到master分支
** 第一步：本地创建新分支 **
  
	git branch test  

** 第二步：切换到本地分支 **  
  
	git checkout test

** 第三步：改动后commit **
  
	git add .
	git commit -m 'test'

** 第四步：切换回master分支 **
  
	git checkout master

** 第五步：merge **
	
	git merge test

** 第六步：同步master **
	
	git pull

** 第七步:解决冲突 **
	pull时冲突
	git stash 暂存本地未提交的修改内容
	git stash list 查看保存在栈中的版本信息,stash@{0}就是刚才保存的标记
	git pull   合并远程代码
	手动修改冲突文件，重新add，commit
	
	merge时冲突
	merge后手动修改冲突文件，重新add，commit

** 第八步:提交到仓库 **
	
	git push

## git日常笔记 ##
+ 查看当前commit记录
	git cherry -v
+ 合并其他分支commit记录
	git cherry-pick <commit SHA>
+ git撤销到某次commit
	git reset --hard <commit_id> （commit_id为撤销到的id）
+ git撤销某次commit
	git revert <commit_id>