---
title: 部署jar文件到Docker上
date: 2018-07-14 
tags: 
- Docker
author: Faushine
layout: post
---
最近签证通过了，这就意味着职业生活也要告一段落。Docker不是最近接触的东西了，早在今年年初就有想法系统学习一下，可总是三天打鱼两天晒网的状态。最近因为新项目部署的问题不得不使用docker，果然是用过才知道好，这篇文章就暂且把用过的命令大概记录一下。手动部署和用Dockerfile都是一个原理，就是把命令一行一行的执行，所以如果你是第一次部署的话最好记录一下操作步骤，之后方便写进Dockerfile里面。

> 手动部署
 
1. 下载镜像文件到本机：
```sh
# docker run -it openjdk:8-jre
```
2. 查看镜像文件
```sh
# docker images
```
3. 可以在列表里看到刚刚下载的镜像
![alt text](/img/in-post/2018-07-14/1.png)
4. 进入运行中的container：
```sh
# docker exec -it fe9f7b1e4fa0      
```
5. 列出最近创建的容器
```sh
# docker ps -l
```
 ![alt text](/img/in-post/2018-07-14/2.png)
6. （如果容器的状态是exit）启动容器
```sh
# docker container restart 76b2b4bdd94b
```
7. 把包复制到容器里面：
```sh
# docker cp D:\velo-prototype\target\velo-standard-alone.war 76b2b4bdd94b:/fanyuxin
```
8. 运行container：
```bash
# docker exec -it 76b2b4bdd94b java -jar /fanyuxin/velo-standard-alone.jar
```
 ![alt text](/img/in-post/2018-07-14/3.png)

> 使用dockerfile部署：

Dockerfile 是一个文本文件，其内包含了一条条的**指令(Instruction)**，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。使用Dockerfile我们可以把每一层修改、安装、构建、操作的命令都写入一个文件中，用这个文件来构建想要的镜像。

1. 创建Dockerfile
``` sh
FROM openjdk:8

ADD ./target/velo-standard-alone.jar /opt/velo-prototype/velo-standard-alone.jar
ADD ./start.sh /opt/velo-prototype/start.sh

WORKDIR /opt/velo-prototype

EXPOSE 8080

ENTRYPOINT ["/bin/bash", "start.sh"]
```
2. 进入Dockerfile所在的目录，build之后docker会按照dockerfile的内容去创建一个name为demo，tag为1的镜像。
```sh
# docker build -t demo:1
```
 ![alt text](/img/in-post/2018-07-14/4.png)
 ![alt text](/img/in-post/2018-07-14/5.png)
3. 运行镜像:
为了能使用localhost访问，我把docker的8080端口映射到了本机的80端口
```sh
# docker run -d -p 80:8080 00c84ecfc3f3
```
 ![alt text](/img/in-post/2018-07-14/6.png)
 ![alt text](/img/in-post/2018-07-14/7.png)
