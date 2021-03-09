---
title: Docker系列之常用命令
date: 2021-03-09 19:00:00
tags:
- Docker
categories:
- JAVA
- Docker

---
## 帮助命令

```shell
docker version # 显示版本信息
docker info	   # 系统信息
docker 命令 --help #万能命令

```

帮助文档地址：https://docs.docker.com/engine/reference

## 镜像命令

**docker images** 查看所有本地的主机上的镜像

```shell
[root@VM-0-11-centos ~]#  docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   10 hours ago   13.3kB

# 解释
REPOSITORY 镜像的仓库源
TAG		   镜像的标签
IMAGE ID   镜像的Id
CREATED    镜像的创建时间
SIZE	   镜像的大小

#可选项
-a 全部镜像
-q 只显示Id
```

**docker search** 搜索镜像

```shell
[root@VM-0-11-centos ~]# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   10578     [OK]       
mariadb                           MariaDB Server is a high performing open sou…   3957      [OK]  

# 可选项 docker serach --help
--filter=STARS=5000  # 结果是STARS大于5000
[root@VM-0-11-centos ~]# docker search mysql  --filter=STARS=5000
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation…   10578     [OK]        
```

**docker pull** 下载镜像

```shell
# 下载镜像 docker pull 镜像名称 可以指定Tag 版本号
[root@VM-0-11-centos ~]# docker pull mysql
Using default tag: latest           # 如果不写tag 默认latest
latest: Pulling from library/mysql
a076a628af6f: Pull complete         # 分层下载  docker images核心  联合文件系统
f6c208f3f991: Pull complete 
88a9455a9165: Pull complete 
406c9b8427c6: Pull complete 
7c88599c0b25: Pull complete 
25b5c6debdaf: Pull complete 
43a5816f1617: Pull complete 
1a8c919e89bf: Pull complete 
9f3cf4bd1a07: Pull complete 
80539cea118d: Pull complete 
201b3cad54ce: Pull complete 
944ba37e1c06: Pull complete 
Digest: sha256:feada149cb8ff54eade1336da7c1d080c4a1c7ed82b5e320efb5beebed85ae8c # 签名信息
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest # 真实地址 

# 等价于它 
docker pull mysql 
docker pull docker.io/library/mysql:latest

# 指定版本
[root@VM-0-11-centos ~]# docker pull mysql:5.7
5.7: Pulling from library/mysql
a076a628af6f: Already exists 
f6c208f3f991: Already exists 
88a9455a9165: Already exists 
406c9b8427c6: Already exists 
7c88599c0b25: Already exists 
25b5c6debdaf: Already exists 
43a5816f1617: Already exists 
1831ac1245f4: Pull complete 
37677b8c1f79: Pull complete 
27e4ac3b0f6e: Pull complete 
7227baa8c445: Pull complete 
Digest: sha256:b3d1eff023f698cd433695c9506171f0d08a8f92a0c8063c1a4d9db9a55808df
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7

```

 ![image-20210306173329396](https://cdn.jsdelivr.net/gh/ljchengx/PicGo/img/image-20210306173329396.png)



**docker rmi** 删除命令

```shell
docker rmi -f 镜像Id # 删除指定的镜像 
docker rmi -f $(docker images -aq) # 删除全部镜像

```





## 容器命令

说明：我们有了镜像才可以创建容器，linux 下载一个 centos镜像来测试学习

```shell
docker pull centos
```

**新建容器并启动**

```shell
docker run [可选参数] image
 
# 参数说明
--name="Name" 容器名字 tomcat01 tomcat02
-d			  后台方式运行
-it			  使用交互方式运行 进入容器查看内容
-P			  指定容器的端口 -p 8080:8080
	-p 主机端口:容器端口 （常用）
	-p 容器端口
-p		      随机指定端口 

# 测试 启动并进入容器
[root@VM-0-11-centos ~]# docker run -it centos /bin/bash
[root@17ad4c340398 /]# ls  查看容器内的centos 基础版本 很多命令不完善 
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

# 从容器中退回主机
[root@17ad4c340398 /]# exit 
exit
[root@VM-0-11-centos /]# ls
bin  boot  data  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

```

**列出所有运行的容器**

```shell
# docker ps
-a   # 列出当前正在运行的容器 +带出历史运行过的容器
-n=? # 显示最近创建的容器
-q   # 只显示容器的编号
[root@VM-0-11-centos /]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@VM-0-11-centos /]# docker ps -a
17ad4c340398   centos         "/bin/bash"   4 minutes ago   Exited (0) 2 minutes ago         
e684eccc0807   d1165f221234   "/hello"      2 days ago      Exited (0) 2 days ago
```

**退出容器**

```shell
exit # 退出容器
Ctrl + P + Q # 退出不停止
```

**删除容器**

```shell
docker rm 容器id                 # 删除指定容器  不能删除正在运行的容器   
docker rm -f $(docker ps -aq)    # 删除所有的容器 
docker ps -a -q|xargs docker rm  # 删除所有的容器
```

**启动和停止容器的操作**

```shell
docker start 容器id    # 启动
docker restart 容器id  # 重启
docker stop 容器id 	 # 停止	
docker kill 容器id     # 杀进程
```

## **常用其他命令**

**后台启动容器**

```shell
# 通过 docker run -d 镜像名 后台启动
[root@VM-0-11-centos ~]# docker run -d centos
d269c3553f1b40ff62ea5a23635ff330cb28f857c2d8c10073c95545b56dffcc

# 通过docker ps 发现停止了

# 常见坑 容器使用后台运行 就必须要有一个前台进程 docker发现没有应用 就会自动停止
# nginx 容器启动后 发现自己没有服务 就会立即停止 就是没有程序了

```

**常看日志命令**

```shell
# docker logs 日志命令  
docker logs -tf --tail 10 容器 没有日志

# 编写脚本
 [root@VM-0-11-centos ~]# docker run -d centos /bin/sh -c "while true;do echo ljchengx;sleep 2;done"

# 查看
[root@VM-0-11-centos ~]# docker ps
CONTAINER ID   IMAGE     
c6d3be37906b   centos    

# 查看日志 指定行数
-tf 
-- tail count  显示的行数
[root@VM-0-11-centos ~]# docker logs -tf --tail 10 c6d3be37906b
 
# 显示全部
[root@VM-0-11-centos ~]# docker logs -tf c6d3be37906b

```

**查看进程信息**

```shell
# top 命令
[root@VM-0-11-centos ~]# docker top c6d3be37906b
UID                 PID                 PPID                C                   STIME             
root                17058               17035               0                   10:07          
root                31240               17058               0                   10:17          

```

**查看元数据

```shell
# docker inspect 容器Id

# 测试
[root@VM-0-11-centos ~]# docker inspect c6d3be37906b
[
    {
        "Id": "c6d3be37906be8467a4cbb8f4d25f48176b406f9e9161ab85a738e3e9a7674fe",
        "Created": "2021-03-09T02:07:54.08862076Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while true;do echo ljchengx;sleep 2;done"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 17058,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-03-09T02:07:54.429353159Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
        "ResolvConfPath": "/var/lib/docker/containers/c6d3be37906be8467a4cbb8f4d25f48176b406f9e9161ab85a738e3e9a7674fe/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/c6d3be37906be8467a4cbb8f4d25f48176b406f9e9161ab85a738e3e9a7674fe/hostname",
        "HostsPath": "/var/lib/docker/containers/c6d3be37906be8467a4cbb8f4d25f48176b406f9e9161ab85a738e3e9a7674fe/hosts",
        "LogPath": "/var/lib/docker/containers/c6d3be37906be8467a4cbb8f4d25f48176b406f9e9161ab85a738e3e9a7674fe/c6d3be37906be8467a4cbb8f4d25f48176b406f9e9161ab85a738e3e9a7674fe-json.log",
        "Name": "/gallant_jemison",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/d5315d260b6f52455014e1a6bc5275a9f81ca40586b33bdf30bb25f6a38835aa-init/diff:/var/lib/docker/overlay2/93fbbcebc66cf30fe7524759b101b0ec71ee5de3c3e12c884988e10ea8b94a0a/diff",
                "MergedDir": "/var/lib/docker/overlay2/d5315d260b6f52455014e1a6bc5275a9f81ca40586b33bdf30bb25f6a38835aa/merged",
                "UpperDir": "/var/lib/docker/overlay2/d5315d260b6f52455014e1a6bc5275a9f81ca40586b33bdf30bb25f6a38835aa/diff",
                "WorkDir": "/var/lib/docker/overlay2/d5315d260b6f52455014e1a6bc5275a9f81ca40586b33bdf30bb25f6a38835aa/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "c6d3be37906b",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "while true;do echo ljchengx;sleep 2;done"
            ],
            "Image": "centos",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20201204",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "79bdc9c626a1158ae89f85850a2e1f1388c9263d3a1da8567d04e470dce27a9f",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/79bdc9c626a1",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "13c3c223789749fc5b64c10fc35992d1e0df64dfa9cb73691eea8f6d51317522",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "9a1d22b0ff0846b37b84fd7179511a6a6065f1c6becd2833f9a21b74f28d6bf2",
                    "EndpointID": "13c3c223789749fc5b64c10fc35992d1e0df64dfa9cb73691eea8f6d51317522",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]

 
```

**进入当前容器的命令**

```shell
# 命令 进入
docker exec -it 容器Id baseshell

# 测试
[root@VM-0-11-centos ~]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED       STATUS       PORTS     NAMES
c6d3be37906b   centos    "/bin/sh -c 'while t…"   4 hours ago   Up 4 hours             gallant_jemison
[root@VM-0-11-centos ~]# docker exec -it c6d3be37906b /bin/bash
[root@c6d3be37906b /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@c6d3be37906b /]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 02:07 ?        00:00:01 /bin/sh -c while true;do echo ljchengx;sleep 2;done
root      6656     0  0 05:49 pts/0    00:00:00 /bin/bash
root      6682     1  0 05:50 ?        00:00:00 /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 2
root      6683  6656  0 05:50 pts/0    00:00:00 ps -ef

# 方式二
docker attach 容器Id

# 进入正在执行的
[root@VM-0-11-centos ~]# docker attach c6d3be37906b

# 区别
exec 新终端
attach 打开正在运行的终端

```

**容器内拷贝文件到主机**

```shell
# docker cp 容器Id:路径/文件名 /主机路径

[root@VM-0-11-centos home]# docker attach ecf8bbccec70
[root@ecf8bbccec70 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@ecf8bbccec70 /]# cd /home
[root@ecf8bbccec70 home]# touch test.java    
[root@ecf8bbccec70 home]# ls
test.java
[root@ecf8bbccec70 home]# exit
exit
[root@VM-0-11-centos home]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS                      PORTS     NAMES
ecf8bbccec70   centos    "/bin/bash"   7 minutes ago   Exited (0) 30 seconds ago             practical_panini
[root@VM-0-11-centos home]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@VM-0-11-centos home]# docker cp ecf8bbccec70:/home/test.java /home
[root@VM-0-11-centos home]# ls
ljchengx.java  test.java

# 拷贝属于手动过程 
```

## **总结**

 ![image-20210309141555377](https://cdn.jsdelivr.net/gh/ljchengx/PicGo/img/image-20210309141555377.png)



```shell
docker version 查看docker的版本号，包括客户端、服务端、依赖的Go等
docker info 查看系统(docker)层面信息，包括管理的images, containers数等
docker search <image> 在docker index中搜索image
docker pull <image> 从docker registry server 中下拉image
docker push <image|repository> 推送一个image或repository到registry
docker push <image|repository>:TAG 同上，指定tag
docker inspect <image|container> 查看image或container的底层信息
docker images TODO filter out the intermediate image layers (intermediate image layers 是什么)
docker images -a 列出所有的images
docker ps 默认显示正在运行中的container
docker ps -l 显示最后一次创建的container，包括未运行的
docker ps -a 显示所有的container，包括未运行的
docker logs <container> 查看container的日志，也就是执行命令的一些输出
docker rm <container...> 删除一个或多个container
docker rm `docker ps -a -q` 删除所有的container
docker ps -a -q | xargs docker rm 同上, 删除所有的container
docker rmi <image...> 删除一个或多个image
docker start/stop/restart <container> 开启/停止/重启container
docker start -i <container> 启动一个container并进入交互模式
docker attach <container> attach一个运行中的container
docker run <image> <command> 使用image创建container并执行相应命令，然后停止
docker run -i -t <image> /bin/bash 使用image创建container并进入交互模式, login shell是/bin/bash
docker run -i -t -p <host_port:contain_port> 将container的端口映射到宿主机的端口
docker commit <container> [repo:tag] 将一个container固化为一个新的image，后面的repo:tag可选
docker build <path> 寻找path路径下名为的Dockerfile的配置文件，使用此配置生成新的image
docker build -t repo[:tag] 同上，可以指定repo和可选的tag
docker build - < <dockerfile> 使用指定的dockerfile配置文件，docker以stdin方式获取内容，使用此配置生成新的image
docker port <container> <container port> 查看本地哪个端口映射到container的指定端口，或者用docker ps 也可以看到。
```