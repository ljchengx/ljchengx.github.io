---
title: SpringCloud系列之Eureka(二)
date: 2021-02-23 19:00:00
tags:
- SpringCloud 
- Ribbon
categories:
- JAVA
- SpringCloud

---

## Ribbon

### 一、服务提供者

#### 1.Eureka注册中心

注册中心较为简单，这里就不多赘述，不清楚的可以参考上一篇。

#### 2.创建服务提供者

这里我们先创建一个spring-cloud-eureka-client子module

1.pom.xml文件为：

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

3.启动项添加注解@EnableDiscoveryClient

```java
@SpringBootApplication
@EnableDiscoveryClient
public class EurekaClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }
}
```

4.添加controller 这里根据自己需要写方法 

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

5.同理创建第二个服务提供者spring-cloud-eureka-client2

pom.xml

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

yml 端口改变，名称不变

```yaml
server:
  port: 9995
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

启动器和controller基本一致

```java
@SpringBootApplication
@EnableDiscoveryClient
public class EurekaClient2Application {
    public static void main(String[] args) {
        SpringApplication.run(EurekaClient2Application.class, args);
    }
}
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

6.启动两个服务提供者我们在注册中心看看效果

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2865550/1614068959571-ba7ab064-6912-4f14-826f-e3f758702ed4.png?x-oss-process=image%2Fresize%2Cw_1500)

这里两个不同端口的同一个服务的提供者完成。

### 二、服务调用者

上面我们创建了服务提供者，现在我们来创建服务调用者

创建一个子module命名为spring-cloud-eureka-ribbon 因为eureka依赖本身已经带有ribbon，我们在使用中不需要额外引入。

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

2.yml文件

```yaml
server:
  port: 9996
spring:
  application:
    name: spring-cloud-user-ribbon-consumer # 应用名称，应用名称会在Eureka中作为服务名称
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:9991/eureka
    register-with-eureka: true  # ⾃⼰就是服务不需要注册⾃⼰ 集群模式下可以改成true
    fetch-registry: true # ⾃⼰就是服务不需要从Eureka Server获取服务信息,默认为true，集群模式下可以改成true
```

3.启动项

```java
@SpringBootApplication
@EnableDiscoveryClient
public class RibbonApplication {
    public static void main(String[] args) {
        SpringApplication.run(RibbonApplication.class, args);
    }
}
```

4.配置项

由于我们使用的是RestTemplate来完成请求

添加RestTemplateConfiguration，这里需要添加注解@LoadBalanced 使用轮询来请求

```java
@Configuration
public class RestTemplateConfiguration {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return  new RestTemplate();
    }
}
```

然后来处理我们的controller

```java
@RestController
@RequestMapping("/ribbon")
public class RibbonController {

    @Autowired
    private RestTemplate restTemplate;

    @GetMapping("/getResult/{id}")
    public String getResult(@PathVariable("id") Integer id) {

        String url = "http://" + "spring-cloud-order-service-provider" + "/order/getResult/" + id;
        return restTemplate.getForObject(url, String.class);
    }
}
```

这里url就是我们服务提供者的名称来组成的。

5.启动并请求

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2865550/1614069342622-cecf399f-7b06-4c78-9728-bd49b6b680ce.png)

浏览器http://localhost:9996/ribbon/getResult/123456

刷新几次可以看看结果：

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2865550/1614069407022-f6cf1fa5-4ef9-411c-8e7c-9451f99b468e.png)

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2865550/1614069416372-96648268-89fc-4106-89f6-13b102df5663.png)

这里分别出现了两个不同的结果，变化的就是服务提供者的端口，可以说明我们的ribbon使用成功。

代码已上传至[Github](https://github.com/ljchengx/spring-cloud-parent)