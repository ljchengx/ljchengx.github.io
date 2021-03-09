---
title: Docker系列之练习
date: 2021-03-09 19:00:00
tags:
- Docker
categories:
- JAVA
- Docker

---
## 一、使用Docker部署Nginx

> 开始

```shell
# 第一步 建议去docker hub搜索
[root@VM-0-11-centos /]# docker search nginx

# 第二步 下载
[root@VM-0-11-centos /]# docker pull nginx

# 启动 
# 这里使用容器的80对应给主机的3344端口 重命名为nginx01
# --name 重命名
# -p 宿主机端口 
[root@VM-0-11-centos /]# docker run -d --name nginx01 -p 3344:80 nginx 

# 查看
# 通过curl 来验证访问主机的3344端口 这里直接返回了nginx默认页面
[root@VM-0-11-centos /]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                  NAMES
933ee17088da   nginx     "/docker-entrypoint.…"   55 seconds ago   Up 54 seconds   0.0.0.0:3344->80/tcp   nginx01
[root@VM-0-11-centos /]# curl localhost:3344
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```



![image-20210309143957874](https://cdn.jsdelivr.net/gh/ljchengx/PicGo/img/image-20210309143957874.png)

```shell
# 进入容器
[root@VM-0-11-centos /]# docker exec -it nginx01 /bin/bash
root@933ee17088da:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx

```

思考：

每次修改配置文件 需要进入容器

达到容器外部修改文件 内部完成修改

-v 数据卷技术

## 二、使用Docker安装Tomcat

```shell
# 官方的使用
$ docker run -it --rm tomcat:9.0

# 之前的启动属于后台 停止后 容器还能查到 使用官方命令是用完即删

# 自己使用 先下载 后启动
[root@VM-0-11-centos /]# docker pull tomcat

# 启动
[root@VM-0-11-centos /]# docker run -d -p 3355:8080 --name tomcat01 tomcat

# 测试访问没有问题

# 进入容器
[root@VM-0-11-centos /]# docker exec -it tomcat01 /bin/bash

# 发现问题 linux命令少了 webapps下没有文件  默认最小镜像 所有不必要的文件去除 保证最小可运行的环境

```

## 三、使用Docker部署elasticsearch

```shell
# es 暴露的端口很多
# es 十分耗内存 数据需要挂载

# --net somenetwork 网络
# 启动
$ docker run -d --name elasticsearch --net somenetwork -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.10.1

# 启动直接卡死  

# 查看 docker stats 

# -e 环境配置修改
docker run -d --name elasticsearch01 -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-xms64m -xmx512m" elasticsearch:7.10.1
```