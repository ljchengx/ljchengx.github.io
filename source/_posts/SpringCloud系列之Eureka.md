---
title: SpringCloud系列之Eureka
date: 2021-02-23 19:00:00
tags:
- SpringCloud 
- Eureka
categories:
- JAVA
- SpringCloud

---

## 一、注册中心

### 1.简单调用

#### 1.Parent项目

主要父级依赖文件详细pom文件为：

```xml
 <properties>
        <spring.cloud-version>Hoxton.SR8</spring.cloud-version>
    </properties>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.8.RELEASE</version>
    </parent>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud-version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```

#### 2.Eureka Server工程

注册中心提供者 

1.创建一个子module在父级项目下 pom文件需要添加netflix-eureka-server

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>
```

2.配置文件yml如下：

```yaml
#eureka server服务端口
server:
  port: 9991
spring:
  application:
    name: spring-cloud-eureka-server # 应用名称，应用名称会在Eureka中作为服务名称

    # eureka 客户端配置（和Server交互），Eureka Server 其实也是一个Client
eureka:
  instance:
    hostname: localhost  # 当前eureka实例的主机名
  client:
    service-url:
      # 配置客户端所交互的Eureka Server的地址（Eureka Server集群中每一个Server其实相对于其它Server来说都是Client）
      # 集群模式下，defaultZone应该指向其它Eureka Server，如果有更多其它Server实例，逗号拼接即可
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
    register-with-eureka: false  # ⾃⼰就是服务不需要注册⾃⼰ 集群模式下可以改成true
    fetch-registry: false # ⾃⼰就是服务不需要从Eureka Server获取服务信息,默认为true，集群模式下可以改成true
```

3.启动项需要添加注解@EnableEurekaServer：

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class,args);
    }
}
```

4.启动效果如下：

[![yL9KXR.png](https://s3.ax1x.com/2021/02/23/yL9KXR.png)](https://imgchr.com/i/yL9KXR)

这里注册服务完成，下面开始处理服务提供者。

#### 3.服务提供者

服务提供者

1.和server工程一样创建一个子module，不过需要添加的依赖不一样

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
    /**
     * 根据用户id获取今日完单数
     * @param id 用户ID
     * @return  完单数
     */
    @GetMapping("/getTodayFinishOrderNum/{id}")
    public Integer getTodayFinishOrderNum(@PathVariable("id") Integer id){
        return port;
    }
}
```

5.启动当前服务提供者

这里在server页面就可以看到已经注册成功。

[![yL9176.png](https://s3.ax1x.com/2021/02/23/yL9176.png)](https://imgchr.com/i/yL9176)

#### 4.服务调用者

调用者整体配置内容和提供者相同，我们按照第三步来修改一下

1.先创建一个子mudole pom.xml一样

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

2.yml修改下名称和端口号

```yaml
server:
  port: 9993
spring:
  application:
    name: spring-cloud-user-service-consumer # 应用名称，应用名称会在Eureka中作为服务名称
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:9991/eureka
    register-with-eureka: true  # ⾃⼰就是服务不需要注册⾃⼰ 集群模式下可以改成true
    fetch-registry: true # ⾃⼰就是服务不需要从Eureka Server获取服务信息,默认为true，集群模式下可以改成true
```

3.启动项也是新增@EnableDiscoveryClient 注解

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

4.这里我们也写一个controller ，在写controller之前写一个RestTemplate

```java
@Configuration
public class RestTemplateConfiguration {
    @Bean
    public RestTemplate getRestTemplate(){
        return  new RestTemplate();
    }
}
```

5.然后我们写一个controller 这里通过服务提供者的名称 来调用第三步的方法

```java
@RestController
@RequestMapping("/user/data")
public class UserCenterController {
    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping("/getTodayStatistic/{id}")
    public Integer getTodayStatistic(@PathVariable("id") Integer id){

        // 使用discoveryClient 类能够与eureka server 交互， getInstances 获取注册到eureka server
        // 的"spring-cloud-order-service-provider" 实例列表

        List<ServiceInstance> instances = discoveryClient.getInstances("spring-cloud-order-service-provider");

        // 获取第一个服务信息
        ServiceInstance instanceInfo = instances.get(0);
        //获取ip
        String ip = instanceInfo.getHost();
        //获取port
        int port = instanceInfo.getPort();
        String url  ="http://"+ip+":"+port+"/order/data/getTodayFinishOrderNum/"+id;
        return restTemplate.getForObject(url, Integer.class);
    }
}
```

6.启动看看

[![yL9JhD.png](https://s3.ax1x.com/2021/02/23/yL9JhD.png)](https://imgchr.com/i/yL9JhD)

这里就出现了两个 我们来试试服务调用者

我们在浏览器页面输入服务调用者的方法http://localhost:9993/user/data/getTodayStatistic/1001

[![yL9N1H.png](https://s3.ax1x.com/2021/02/23/yL9N1H.png)](https://imgchr.com/i/yL9N1H)

这里时候返回的是服务提供者的端口，验证通过，实际上最终方法调用的是9992端口的服务提供者的方法。

以上一个Eureka简单调用方式流程完成。

代码已上传至[Github](https://github.com/ljchengx/spring-cloud-parent)