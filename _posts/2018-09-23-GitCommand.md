---
layout: post
title:      Git Commands
subtitle:   Commands used frequently in git
date:       2018-09-23
author:     WCS
header-img: img/2018-09-23-GitCommand.jpg
catalog: true
tags:
    - Git
---

## 1.常见命令

初始化一个Git仓库  

`git init`  

添加文件夹到暂存区  

`git add src/`  

添加文件到暂存区  

`git add pom.xml`  

版本提交(将暂存区的改动提交到版本控制中)   

`git commit -m "comment"`  

回退到上一个版本，HEAD表示当前版本  

`git reset --hard HEAD^`  

撤销xx.txt文件的修改    

`git checkout -- xx.txt`  

## 2.添加远程仓库

Git与Github是通过SSH加密的所以需要一些设置如下：  

**第一步**：创建SSHkey。

`ssh-keygen -t rsa -C "youremail@example.com"`  

然后就能在c:/user/用户名/.shh中看到`id_rsa`和`id_rsa.pub`  

**第二步**：登陆GitHub，打开“Account settings”，“SSH Keys”页面：
然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴`id_rsa.pub`文件的内容。  

**第三步**：先在github中建立一个空的仓库springBootDemo。  

**第四步**：然后在git Bush中添加远程仓库。  

`git remote add origin git@github.com:Bjorne1/springBootDemo.git`  

**第五步**：把本地仓库推送到远程仓库。  

`git push -u origin master`  

(我们第一次推送master分支时，加上了`-u`参数，Git不但会把本地的master分支内容推送的远程新的master分支，
还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。)  


## 3.从远程仓库克隆

先在github中建立一个仓库。  

`git clone git@github.com:Bjorne1/springBootDemo.git`

## 4.git忽略文件`.gitignore`

由于window平台不能直接创建`.`开头的文件名，于是需要通过git bash来创建。  

**第一步**：在`git bash`中`vim .gitignore`或者`touch .gitignore`来创建.gitignore文件。  

**第二步**：`i`进入到编辑模式。  

**第三步**：`.ignore`提交规则。  
```
1：以#开头的为注释  
2：*.text 表示忽略所有.text 结尾的文件  
3：*.[ab] 表示忽略*.a和*.b结尾的文件
4：!a.text 表示但a.text除外  
5：.idea/ 表示忽略.idea目录  
6：.idea 表示忽略.idea目录和文件  
7：/pom.xml 表示只忽略当前目录下的pom.xml
```

## 5.git在idea中使用的一些tips

1：先在idea设置中设置好git、github的目录账号等。  

2：本地文件在修改后`commit`，即本地git仓库的更新提交。  

3：在git中，先`pull`再`push`，github仓库就与本地仓库同步了。  

## 6.版本回退

**第一步**：先在git-show history找到你要回退版本的版本号oldVersion以及当前版本号newVersion。  

**第二步**：Git->Repository->Reset HEAD 中输入oldVersion,Reset Type:*Hard。  

**第三步**：此时将代码push到github会提示冲突，则取消提交。  

**第四步**：Git->Repository->Reset HEAD 中输入newVersion,Reset Type:*Mixed。  

**第五步**：`commit`之后再`push`,就不再有冲突了。

