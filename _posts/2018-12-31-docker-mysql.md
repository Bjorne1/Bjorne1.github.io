---
layout: post
title:      docker-mysql
subtitle:   Communications link failure 
date:       2018-12-31
author:     WCS
header-img: img/2018-12-31-docker-mysql.jpg
catalog: true
tags:
    - mysql
---

# 前言  

趁着元旦放假，学习了下docker相关的知识，在这里记录下学习笔记，以及docker容器中安装mysql的一些注意事项，以及踩的一些坑。  

## 一、什么是docker

对于docker，我的理解就是，docker是一个容器，就像一个矿泉水瓶，里面装着不同颜色的水（即镜像images），然后容器里面的水可以随时拿出来放进去，而不影响。而这个水是可以由我们自己配置的，并给他人使用。



## 二、核心概念

docker主机（Host）：安装了docker程序的机器；

docker客户端（Client）：连接docker主机进行操作；

docker仓库（Registry）：用来保存各种打包好的软件镜像；

docker镜像（Images）：软件打包好的镜像，放在docker仓库中；

docker容器（Container）：镜像启动后的实例称为一个容器，容器是独立运行的一个或多个应用。



## 三、在linux上安装docker

1.检查内核版本，必须是3.10以上；

`uname -r`

2.安装docker

`yum install docker`

> 我在安装完docker的时候遇到，`Job for docker.service failed`的问题，经多方尝试，最终得以解决！只需先`yum update`即可，我的是这个原因导致的，网上还看到有其他各种方式，但对我都没用。

3.启动docker

`sudo service docker start`

> 设置开机启动docker`systemctl enable docker`

4.测试docker是否安装成功

`docker run hello-world`



## 四、docker常用命令操作

1.镜像操作


| 命令                    | 说明                                                   |
| ----------------------- | ------------------------------------------------------ |
| `docker search mysql`   | 从docker hub上搜索mysql镜像                            |
| `docker pull mysql:5.5` | 拉取5.5版本的mysql，:5.5是版本号，不输入则默认最新版本 |
| `docker images`         | 查看本地所有镜像                                       |
| `docker rmi images-id`  | 是删除指定的本地镜像                                   |

2.容器操作

这里以mysql为例：

步骤：

```linux
//1.搜索mysql镜像
docker search mysql
//2.下载镜像，这里默认下载的是最新版本，从而留下好几个坑。。
docker pull mysql
//3.根据镜像启动容器，参照docker hub上的官方文档
docker run --name some-mysql ‐p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql:tag
//说明：
//--name：后面是给容器起别名；
//-p：是映射端口号，将docker的3306端口映射到内部的mysql3306端口上，外部才能访问到；
//MYSQL_ROOT_PASSWORD=root 是设置mysql密码，这里有几个可选参数，必须选一个
//-d：在后台运行

//4.查看运行中的容器
docker ps
//5.查看所有容器
docker ps -a
//6.启动容器
docker start 容器id
//7.删除一个容器
docker rm 容器id
//8.查看容器日志
docker logs 容器id/name
//9.还有其他操作可以查看官方提供的文档https://hub.docker.com/_/mysql

```

## 五、mysql踩坑

1.mysql在docker容器中启动后，外部Navicat访问的时候报`caching-sha2-password`错误，这是由于mysql新版本的密码加密规则不一样导致的，所以需要修改docker中mysql的密码加密规则。

```mysql
//1.63c9e29aelef为容器id
docker exec -it 63c9e29aelef /bin/bash
//2.输入账号密码-u,-p
mysql -uroot --proot 
//输入以下sql即可
ALTER USER 'root' IDENTIFIED WITH mysql_native_password BY 'root';   

```



2.springboot启动的时候，报`com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure `，网上看到说是由于mysql默认配置导致的，所以需要修改mysql中`/etc/mysql/my.cnf`里面的配置。

```
//1.665b4a1e17b6为容器id
docker exec -i -t 665b4a1e17b6 /bin/bash
//2.首先要先安装vim，否则无法编辑my.cnf文件
vim install
//3.进入my.cnf文件夹,打开my.cnf文件
cd /etc/mysql
vim my.cnf
//4.加入以下内容31536000即一年
wait_timeout=31536000
interactive_timeout=31536000
```

问题解决！



## 六、总结

这篇博客写下来，感觉写那些基本操作，核心概念啥的写的意义不大啊，那些命令啥的还不如自己多敲熟悉，写这个花了一个多小时，感觉有点浪费时间，以后重点还是要多放在踩的坑当中！