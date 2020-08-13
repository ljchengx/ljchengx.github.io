---
title: 记一次关于LitePal数据库的使用
date: 2020-03-28 13:58:19
tags:
- LitePal
- 数据库
categories:
- Android
---

#### 1.原因描述：

项目过程中需要使用本地数据库来保存一些临时数据，方便下次读取使用。发现了LitePal这个库，于是记录下使用的过程，LitePal这个库是郭霖大神写的,下面就是集成知识简单的使用,如果有什么不对的地方望指出。

#### 2.尝试方法：

##### 2.1 LitePal库的集成

```java
 implementation  'org.litepal.android:java:3.0.0'
```

##### 2.2 创建litepal.xml文件

在你的项目**assets** 下添加litepal.xml 这一步主要的是完成数据库相关配置 如名称，版本和表。

![GkdEM8.png](https://s1.ax1x.com/2020/03/28/GkdEM8.png)

##### 2.3 初始化数据库配置

 这里两种方式第一个是让你的**AndroidManifest.xml** 直接使用org.litepal.LitePalApplication 或者在你自己的Application中加上 LitePal.initialize(this);

##### 2.4 建表
![Gkw1kd.md.png](https://s1.ax1x.com/2020/03/28/Gkw1kd.md.png)

这里注意新增的表一定要在litepal.xml文件的list下面添加才可以。

##### 2.5 插入数据

```java
    public void insert(View view) {
        
        ExampleModel exampleModel = new ExampleModel();
        exampleModel.setKey("123");
        exampleModel.setContent("456");
        exampleModel.saveAsync().listen(new SaveCallback() {
            @Override
            public void onFinish(boolean success) {
                if(success){
                    Toast.makeText(MainActivity.this, "数据库保存成功", Toast.LENGTH_SHORT).show();
                }
            }
        });
    }
```

这里使用了异步保存的方法 也可以直接使用save()方法。

##### 2.6 查询数据

```java
    public void select(int id) {

        //根据Id查询
        ExampleModel exampleModel = LitePal.find(ExampleModel.class, id);

        //根据条件查询
        List<ExampleModel> exampleModels = LitePal.where("key  = ? ", "123").find(ExampleModel.class);

    }
```

不同的查询方式，还有更加详细的方式可以看看官方的文档。

##### 2.7 删除数据

```java
    public void delete(int id){

        LitePal.delete(ExampleModel.class, id);
        LitePal.deleteAll(ExampleModel.class, "key = ?" , "350");

    }
```



#### 5.参考链接：

- https://github.com/LitePalFramework/LitePal
