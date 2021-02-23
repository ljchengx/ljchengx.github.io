---
title: SpringCloud系列之Eureka(三)
date: 2021-02-23 19:00:00
tags:
- SpringCloud 
- Feign
categories:
- JAVA
- SpringCloud

---

## Feign

我们在进行微服务项目的开发的时候，经常会遇到一个问题，比如A服务是一个针对用户的服务，里面有用户的增删改查的接口和方法，而现在我有一个针对产品的服务B服务中有一个查找用户的需求，这个时候我们可以在B服务里再写一个查找用户的接口，可是就为了一个接口就得从控制层到持久层都写一遍怎么看都不值当，最关键的是这个接口在别的服务里面还有，这就更不应该做了，所以springCloud提供了服务调用的方法——feign。

### 一、Feign

#### 1.Eureka注册中心

注册中心较为简单，这里就不多赘述，不清楚的可以参考系列第一篇。

#### 2.服务提供者

pom.xml文件如下：

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

2.yml文件需要修改配置

```yaml
server:
  port: 9992
spring:
  application:
    name: spring-cloud-order-service-provider
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:9991/eureka //这里是server工程地址
    register-with-eureka: true  # ⾃⼰就是服务不需要注册⾃⼰ 集群模式下可以改成true
    fetch-registry: true # ⾃⼰就是服务不需要从Eureka Server获取服务信息,默认为true，集群模式下可以改成true
```

3.启动项需要添加的注解和server不一样，使用@EnableDiscoveryClient

```java
@SpringBootApplication
@EnableDiscoveryClient
public class EurekaClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }
}
```

4.我们这边写一个方法，等下使用，新建一个controller文件夹，创建一个controller.

```java
@RestController
@RequestMapping("/order/data")
public class OrderStatisticServiceController {
    @Value("${server.port}")
    private Integer port;
    
    @GetMapping("/getResult/{id}")
    public String getResult(@PathVariable("id") Integer id) {
        return "getResult = " + id;
    }
}
```

5.启动当前服务提供者

这里在server页面就可以看到已经注册成功。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2865550/1614061469859-f3f011d2-1401-4597-a6e5-67eda4705dcf.png?x-oss-process=image%2Fresize%2Cw_1500)

#### 3.服务调用者

Feign依赖于Eureka环境，所以我们在上面的基础上新增一个子module即可

1.pom.xml文件如下,相比上面简单调用的第四步多一个关于Feign的依赖

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
    </dependencies>
```

2.yml配置文件

```yaml
server:
  port: 9994
spring:
  application:
    name: spring-cloud-user-feign-consumer # 应用名称，应用名称会在Eureka中作为服务名称
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:9991/eureka
    register-with-eureka: true  # ⾃⼰就是服务不需要注册⾃⼰ 集群模式下可以改成true
    fetch-registry: true # ⾃⼰就是服务不需要从Eureka Server获取服务信息,默认为true，集群模式下可以改成true
```

3.启动项相比上面的调用者多个@EnableFeignClients注解

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class FeignApplication {
    public static void main(String[] args) {
        SpringApplication.run(FeignApplication.class, args);
    }
}
```

4.我们使用Feign需要创建一个接口 这里举个简单的例子

```java
@FeignClient(name = "spring-cloud-order-service-provider") //这里是服务提供者在Eureka中的名称
public interface IResultInterface {
    @RequestMapping(value = "/order/getResult/{id}") //服务提供者的接口路由
    String getResult(@PathVariable("id") Integer id);
}
```



5.我们创建当前module的Controller调用的接口

```java
@RestController
@RequestMapping("/order")
public class ResultController {

    @Autowired
    IResultInterface iResultInterface; //这里使用接口来完成方法的请求

    @RequestMapping("/getResult/{id}")
    public String getResult(@PathVariable("id") Integer id) {
        return iResultInterface.getResult(id);
    }
}
```

6.启动并访问看看效果

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2865550/1614065389227-7b9190f8-7c00-4e23-a09a-d68b20e3f6f2.png?x-oss-process=image%2Fresize%2Cw_1500)



![image.png](https://cdn.nlark.com/yuque/0/2021/png/2865550/1614065405766-60e085ba-21cf-4f5f-9089-02fe7dcba2b6.png)

这里看到我们请求的是9994的新的服务调用者工程 但是返回值的服务提供者的结果，说明这里Feign使用成功。



代码已上传至[Github](https://github.com/ljchengx/spring-cloud-parent)