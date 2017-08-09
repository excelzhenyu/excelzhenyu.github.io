---
layout: post
title: docker部署mysql
category : docker
tagline: "Supporting tagline"
tags : [docker, mysql]
---
{% include JB/setup %}
# docker部署mysql
---

最近做一个项目用docker部署mysql，记录下自己的过程跟大家分享。
1.首先，从远程仓库pull最新版的mysql，

```
docker pull mysql
```
默认pull mysql:latest
2.接下来就要run mysql了，
这里说一下，docker部署的mysql有个问题，就是重启后数据就都没有了，假如你想要把数据持久话到本地，需要挂载来映射主机的一个目录


```
docker run -d -p 3306:3306 -v /your/local/path:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=your password mysql:latest
```
这里可以用端口映射也可以设置--net=host来设置网络连接使mysql能被外部环境访问。
3.进入容器内部

```
docker exec -it {containerId} /bin/bash
```
4.修改mysql配置
进入容器内部的话是没有vi的，apt-get install也不行，不知道是不是我电脑的原因。我的解决方案是
1>先apt-get update,在apt-get install vim，就可以了。
2>挂载外部my.cnf配置文件，命令如下

```
docker run --name some-mysql -v /my/custom:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=myPassword -d mysql:latest
```
5.如果想同时既挂载外部配置文件又要存储数据到本地,使用两个-v命令即可

6.更多信息请访问官方文档 [点击查看官方文档](https://hub.docker.com/_/mysql/)

