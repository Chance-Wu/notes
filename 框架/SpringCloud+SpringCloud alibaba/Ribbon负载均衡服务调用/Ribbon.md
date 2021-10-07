### 一、ribbon介绍

Spring Cloud Ribbon是一个基于HTTP和TCP的客服端==负载均衡==工具，它是基于Netflix Ribbon实现的一个工具类框架。

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-loadbalancer -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-loadbalancer</artifactId>
  <version>3.0.3</version>
</dependency>
```

功能：

- LB（负载均衡）——将用户的请求平摊地分配到多个服务上，从而达到系统的HA（高可用）。
  - 集中式LB：即在服务的消费方和提供方之间使用独立的LB设施（可以是硬件F5，也可以是软件Nginx），由该设施负责把访问请求通过某种策略转发至服务的提供方；
  - 进程内LB：==将LB的逻辑集成到消费方==（比如Ribbon），消费方从服务注册中心获知哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。

>Ribbon本地负载均衡客户端 VS Nginx服务端负载均衡
>
>- Nginx负载均衡，客户端所有请求都会交给nginx，然后由nginx实现转发请求。即是由服务端实现的。
>- ==Ribbon本地负载均衡，在调用微服务接口时候，会在注册中心上获取信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程服务调用==。



**负载均衡 + RestTemplate调用**



### 二、负载均衡和Rest调用



#### 2.1 引入pom依赖

---

eureka整合了ribbon，所以只要引入eureka的依赖即可。

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```



#### 2.2 主启动类打开`@EnableEurekaClient`注解

---

```java
@SpringBootApplication
@EnableEurekaClient
public class OrderMain80
{
  public static void main(String[] args) {
    SpringApplication.run(OrderMain80.class, args);
  }
}
```

这里ribbon已经开始负载均衡（即@EnableEurekaClient已经开启了ribbon），但是默认的负载算法是轮询。



#### 2.3 修改默认的负载均衡配置

---

新建配置类，这个类一定不能在主启动类同级别或主启动类级别的包下面（官网有明确要求）。

```java
@Configurable
public class RibbonConfiguration {

  @Bean
  public IRule ribbonRule(IClientConfig clientConfig) {
    // 注册别的默认负载均衡方法
    return new WeightedResponseTimeRule(); 
  }
}
```

同时在主启动类级别的包下添加另外一个配置类，给RestTemplate添加负载均衡功能。



























