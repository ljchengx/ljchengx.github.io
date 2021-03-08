---
title: Docker系列之安装
date: 2021-03-08 19:00:00
tags:
- Docker
categories:
- JAVA
- Docker

---
## Docker之安装

### 安装

> 环境准备

1.Linux服务器 Centos7

2.远程连接

> 环境查看

```sh
#系统内核是3.10以上
[root@VM-0-11-centos ~]# uname -r
3.10.0-1127.19.1.el7.x86_64
```

```sh
#系统详细情况
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```

> 安装

帮助文档：

```shell
# 1、卸载旧文件
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
# 2、需要的安装包
yum install -y yum-utils

# 3、设置镜像的仓库
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo # 默认国外 
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo # 推荐使用阿里云
# 更新yum 索引
yum makecache fast

# 4、安装docker docker-ce 社区版
yum install docker-ce docker-ce-cli containerd.io

# 5、启动docker
systemctl start docker

# 6、使用docker version 查看是否安装成功

```

 ![](https://cdn.jsdelivr.net/gh/ljchengx/PicGo/img/image-20210306164301153.png)

```shell
# 7、run
docker run hello-world
```

 ![](https://cdn.jsdelivr.net/gh/ljchengx/PicGo/img/image-20210306163508989.png)

```sh
# 8、查看下载的hello-word 镜像
docker images
```

 ![](https://cdn.jsdelivr.net/gh/ljchengx/PicGo/img/image-20210306163717655.png)

```shell
# 9、卸载
# 卸载依赖
yum remove docker-ce docker-ce-cli containerd.io

# 删除资源
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```

### 阿里云镜像加速

 1、登录阿里云 容器镜像服务

![image-20210306165025966](https://cdn.jsdelivr.net/gh/ljchengx/PicGo/img/image-20210306165025966.png)

2、找到镜像服务加速地址 左下角位置 找到对应系统

 ![image-20210306165217127](https://cdn.jsdelivr.net/gh/ljchengx/PicGo/img/image-20210306165217127.png)

3、配置

```sh
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ki5mdt9h.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload

sudo systemctl restart docker
```



### 回顾流程

1.开始

2.Docker会在本机寻找镜像

3.判断本机是否有镜像

4.有镜像就运行 没有就去仓库下载

5.找到了下载并运行



### 底层原理

#### docker是如何工作的？

Docker是-个Client Server结构的系统, Docker的守护进程运行在主机上。通过Socket从客户端访问 !

Docker Server接收到Docker-Client的指令,就会执行这个命令!

![](https://cdn.jsdelivr.net/gh/ljchengx/PicGo/img/image-20210306170149393.png)

#### docker为什么比Vm快？

1、docker比虚拟机有着更少的抽象层

 ![image-20210306170310973](https://cdn.jsdelivr.net/gh/ljchengx/PicGo/img/image-20210306170310973.png)

2、docker利用的是宿主机的内核

所以说,新建一个容器的时候 , docker不需要想虚拟机一样重新加载一个操作系统内核 ,避免引导。

虚拟机是加载GuestOS ,分钟级别的,而docker是利用宿主机的操作系统吗,省略了这个复杂的过程,秒级!