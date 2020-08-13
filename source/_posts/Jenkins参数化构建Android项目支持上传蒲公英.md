---
title: Jenkins参数化构建Android项目支持上传蒲公英
date: 2020-07-23 20:20:12
tags:
- jenkins
categories:
- Android
---

#### 一、新建一个自由风格的任务，进入配置项 General选择参数化构建过程

##### 1.新增选择选项参数 

BUILD_TYPE 设置debug release

![UOr2aF.png](https://s1.ax1x.com/2020/07/23/UOr2aF.png)

![UOspsP.png](https://s1.ax1x.com/2020/07/23/UOspsP.png)



##### 2.添加字符参数 

APP_VERSION_NAME  APP_VERSION_CODE IS_JENKINS 如下图

![UOs8JJ.png](https://s1.ax1x.com/2020/07/23/UOs8JJ.png)

![UOsloF.png](https://s1.ax1x.com/2020/07/23/UOsloF.png)

![UOs3i4.png](https://s1.ax1x.com/2020/07/23/UOs3i4.png)



##### 3.项目代码调整

这里设置后如何在项目代码中使用，我们需要在项目的gradle.properties文件下新增刚才添加的参数

```  
BUILD_TYPE  APP_VERSION_NAME  APP_VERSION_CODE IS_JENKINS
```
![UOs6SA.png](https://s1.ax1x.com/2020/07/23/UOs6SA.png)

然后修改app下的build.gradle来使用这些参数 例如：

```
defaultConfig {
        minSdkVersion 19
        targetSdkVersion 29
        versionCode APP_VERSION_CODE as int
        versionName APP_VERSION_NAME
    }
```



回到Jenkins下构建命令修改成使用参数的形式 assemble${BUILD_TYPE}，展开勾选

Pass all job parameters as Project properties

![UOs7Ss.png](https://s1.ax1x.com/2020/07/23/UOs7Ss.png)
![UOsHln.png](https://s1.ax1x.com/2020/07/23/UOsHln.png)

保存完成，开始打包。

![UO6jG4.png](https://s1.ax1x.com/2020/07/23/UO6jG4.png)



#### 二、打包完成自动上传蒲公英

##### 1.安装蒲公英插件

前往插件管理，搜索[ Upload to pgyer](https://plugins.jenkins.io/upload-pgyer) 完成安装。

##### 2.增加构建后操作步骤 新增归档成品 

地址填写自己的打包后的地址(方便执行完成后 可以展示出apk)

app/build/outputs/apk/${BUILD_TYPE}/*.apk

![UORsfS.png](https://s1.ax1x.com/2020/07/23/UORsfS.png)

##### 3.新增upload to pgyer with apiV1 

两个Key根据自己的账号填写

${WORKSPACE}/app/build/outputs/apk/${BUILD_TYPE}

app-${BUILD_TYPE}.apk

![UOWa3F.png](https://s1.ax1x.com/2020/07/23/UOWa3F.png)
![UOWU9U.png](https://s1.ax1x.com/2020/07/23/UOWU9U.png)

##### 4.展示蒲公英下载二维码

安装插件[description setter plugin](https://plugins.jenkins.io/description-setter)

回到增加构建后步骤 set build description

![UOfwPf.png](https://s1.ax1x.com/2020/07/23/UOfwPf.png)

填写如下信息：

```
<a href="${appBuildURL}"><img src="${appQRCodeURL}" width="118px" height="118px"></a>
```

![UOffiV.png](https://s1.ax1x.com/2020/07/23/UOffiV.png)

保存后进入首页系统管理的全局安全管理 修改格式器为safe html 保存，然后重新打包项目。

![UOheSS.png](https://s1.ax1x.com/2020/07/23/UOheSS.png)

![UOhVW8.png](https://s1.ax1x.com/2020/07/23/UOhVW8.png)

打包完成后就能在界面看见下载二维码和构建结果。

![UOhQwn.png](https://s1.ax1x.com/2020/07/23/UOhQwn.png)