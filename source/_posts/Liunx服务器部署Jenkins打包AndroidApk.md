---
title: Liunx服务器部署Jenkins打包Android Apk
date: 2020-07-21 20:20:12
tags:
- jenkins
categories:
- Android
---



## Liunx服务器部署Jenkins打包Android Apk

### 一.安装JAVA环境

#### 1.下载JDK

​		根据情况下载JDK版本 目前本教程使用的jdk8 也可以自行前往下载页面: https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html.  这里要看下自己的服务器系统是32位还是64位，按照所需下载即可，建议下载后缀为tar.gz压缩格式的安装包。下载后上传到服务器上，通过命令 tar -zxvf jdk-8u261-linux-i586.tar.gz 即可。

#### 2.配置环境变量

```
vim /etc/profile
```

打开到最后加上JAVA_HOME环境变量

```
export JAVA_HOME=/usr/local/java/jdk1.8.0_261
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

执行命令使环境变量生效

```
source /etc/profile
```



### 二.安装GRADLE环境

#### 1.下载GRADLE

​		根据项目中使用的gradle版本下载对应文件 下载地址:https://services.gradle.org/distributions/

这里使用的是 gradle-5.4.1-all.zip，下载后上传到服务器 这里放的是/usr/local/gradle 路径下(根据自己的情况来)，解压 。

```
unzip gradle-5.4.1-all.zip
```

#### 2.配置环境变量

打开文件

```
vim /etc/profile
```

文件最后加上(可以和JAVA_HOME放一起)

```
export GRADLE_HOME=/usr/local/gradle/gradle-5.4.1
export PATH=${GRADLE_HOME}/bin:${PATH}
```

然后执行生效

```
source /etc/profile
```



### 三.安装AndroidSDK

#### 1.下载AndroidSDK

​		打开网址：https://developer.android.google.cn/studio

找到Command line tools only 标题下的SDK tools package 下载Linux包

![U7DjG6.png](https://s1.ax1x.com/2020/07/22/U7DjG6.png)

下载完成后上传服务器 /opt/android(路径根据自己情况来 可以和gradle放一起)。完成解压 unzip xxx

![UHSl7D.png](https://s1.ax1x.com/2020/07/22/UHSl7D.png)

正常解压后只有这一个文件夹，OK 我们这个时候先去配置下环境

打开 

```
vim /etc/profile
```

和上面一样

```
export ANDROID_HOME=/opt/android/sdk
export PATH=$PATH:$ANDROID_HOME:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools:$ANDROID_HOME/tools/bin
```

配置Android_home

执行

```
source /etc/profile
```

这个时候来到刚才只有tools这一层文件夹

执行

```
sdkmanager --list
```

应该会出现下图这样的表格

![UHpkKP.png](https://s1.ax1x.com/2020/07/22/UHpkKP.png)

这个时候就根据当前项目需要的版本来进行安装就可以。

比如当前项目需要 build-tools:29.0.2 ，platforms;android-28 ，ndk;21.3.6528147

例如执行：

```
sdkmanager "platform-tools"
sdkmanager "build-tools:29.0.2"
sdkmanager "platforms;android-28"
sdkmanager "ndk;21.3.6528147"
```

等需要的版本都安装好以后当前文件夹下就会出现如上面那个图。

到此需要的环境变量全部安装完成。

### 四.安装Jenkins

#### 1.下载安装包

​		前往：http://pkg.jenkins-ci.org/redhat-stable/ 例如下载：jenkins-2.222.1-1.1.noarch.rpm 上传到服务器上

```
rpm -ivh jenkins-2.222.1-1.1.noarch.rpm
```

完成安装。

#### 2.修改配置

```
vim /etc/sysconfig/jenkins
```

可以根据自己的需要改变端口，原来的是8080 。

基本上安装完成了可以访问下服务器ip:8080是不是通了，

如果没有通查看问题

```
systemctl status jenkins
```

如果是java环境问题可以替换自己的路径

修改java配置： vi /etc/init.d/jenkins 修改为自己java路径

然后启动service jenkins start

第一次打来jenkins的时候，有个初始化密码，需要你输入。

密码在服务器的 cd /var/lib/jenkins/secrets

### 五.开始打包

1.打开jenkins系统管理下的系统配置，完成以下三个参数的配置

ANDROID_HOME

/usr/share/android-sdk

GRADLE_USER_HOME

/usr/local/gradle/gradle-5.4.1

JAVA_HOME

/usr/local/java/jdk1.8.0_261

这三个路径在上面服务器的环境变量里保持一致

![UHe6C6.png](https://s1.ax1x.com/2020/07/22/UHe6C6.png)

2.安装Gradle插件 成功后进入系统管理的全局工具配置

[![UHmaJP.png](https://s1.ax1x.com/2020/07/22/UHmaJP.png)](https://imgchr.com/i/UHmaJP)

这里地址也是上面服务器的环境变量配置。

3.新建一个任务

[![UHnKTs.png](https://s1.ax1x.com/2020/07/22/UHnKTs.png)](https://imgchr.com/i/UHnKTs)

源码管理根据自己的需要 默认有git 

[![UHnfAA.png](https://s1.ax1x.com/2020/07/22/UHnfAA.png)](https://imgchr.com/i/UHnfAA)

没有svn的去插件管理安装下subversion

然后新增构建步骤

![UHungK.png](https://s1.ax1x.com/2020/07/22/UHungK.png)

[![UHuwDg.png](https://s1.ax1x.com/2020/07/22/UHuwDg.png)](https://imgchr.com/i/UHuwDg)

1位置选择刚才设置的全局参数 2位置选择你的执行命令 可以空格分开

点击保存回到首页就可以构建了。