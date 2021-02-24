---
title: SpringCloud系列之Eureka(四)
date: 2021-02-24 19:00:00
tags:
- SpringCloud 
- Hystrix
categories:
- JAVA
- SpringCloud

---

## Hystrix

熔断器：

**优点**：

1.快速失效 不用等业务超时时间 返回默认值

2.服务正常，自动连接到业务

**三种状态**：关闭 打开 半开

1.不是调用失败立即打开 

2.调用次数达到阀值  服务出异常 打开断路器

3.过一个时间窗口 重新调用

如果服务调用失败 就是调用FallBack

### 1.服务提供者

这里使用的还是之前的提供者模块spring-cloud-eureka-client

1.pom.xml

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
```



2.yml配置文件

```yaml
server:
  port: 9992
spring:
  application:
    name: spring-cloud-order-service-provider
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:9991/eureka
    register-with-eureka: true  # ⾃⼰就是服务不需要注册⾃⼰ 集群模式下可以改成true
    fetch-registry: true # ⾃⼰就是服务不需要从Eureka Server获取服务信息,默认为true，集群模式下可以改成true
```



3.controller逻辑代码

```java
@RestController
@RequestMapping("/order")
public class OrderStatisticServiceController {
    @Value("${server.port}")
    private Integer port;

    @GetMapping("/getResult/{id}")
    public String getResult(@PathVariable("id") Integer id) {
        return "getResult = " + port + "--" + +id;
    }
}
```

4.启动Eureka服务，启动服务提供者

[![yOEkXF.png](https://s3.ax1x.com/2021/02/24/yOEkXF.png)](https://imgtu.com/i/yOEkXF)

### 2.服务调用者

这里服务调用者我们使用Feign的方式来处理 项目参考Feign的配置

我们新建一个子module命名为spring-cloud-user-hystrix-consumer

1.pom.xml

这里注意一下 我们在Feign的基础上需要添加Hystrix的依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
    </dependencies>
```

2.yml配置文件

换个名称和端口即可

```yaml
server:
  port: 9997
spring:
  application:
    name: spring-cloud-user-hystrix-consumer # 应用名称，应用名称会在Eureka中作为服务名称
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:9991/eureka
    register-with-eureka: true  # ⾃⼰就是服务不需要注册⾃⼰ 集群模式下可以改成true
    fetch-registry: true # ⾃⼰就是服务不需要从Eureka Server获取服务信息,默认为true，集群模式下可以改成true
```

3.Feign的接口

使用Feign来完成服务调用

```java
@FeignClient(name = "spring-cloud-order-service-provider")
public interface IResultInterface {
    @RequestMapping(value = "/order/getResult/{id}")
    String getResult(@PathVariable("id") Integer id);
}
```

4.Controller

这里需要注意 我们在写自己调用方法的同时，就需要引入熔断器的fallback，当我们的服务提供者无法访问的时候，

触发熔断器则进入自定义写的类似回调方法

```java
@RestController
@RequestMapping("/order")
public class ResultController {

    @Autowired
    IResultInterface iResultInterface;

    @RequestMapping("/getResult/{id}")
    @HystrixCommand(fallbackMethod = "fallbackGetResult")
    public String getResult(@PathVariable("id") Integer id) {
        return iResultInterface.getResult(id);
    }

    //触发熔断器的返回值
    public String fallbackGetResult(Integer id){
        return "这里进入熔断器的默认值";
    }
}
```

5.启动项目

[![yOEEm4.png](https://s3.ax1x.com/2021/02/24/yOEEm4.png)](https://imgtu.com/i/yOEEm4)

启动完成，我们来验证一下；浏览器地址输入：http://localhost:9997/order/getResult/1234567

这个时候返回的结果是：

[![yOEmkR.png](https://s3.ax1x.com/2021/02/24/yOEmkR.png)](https://imgtu.com/i/yOEmkR)

说明当前通过Feign来调用9992端口的服务提供者方法成功

然后我们把服务提供者的端口停掉，刷新请求看看效果

[![yOEQ1K.png](https://s3.ax1x.com/2021/02/24/yOEQ1K.png)](https://imgtu.com/i/yOEQ1K)

这里发现进入我们的fallback的自定义方法里了，说明熔断器触发成功。

代码已上传至[Github](https://github.com/ljchengx/spring-cloud-parent)