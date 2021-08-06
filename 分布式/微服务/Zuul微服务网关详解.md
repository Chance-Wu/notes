Zuul是spring cloud中的微服务网关。网关：是一个网络整体系统中的前置门户入口。请求首先通过网关，进行路径的路由，定位到具体的服务节点上。

> - Zuul首先是一个==微服务==，会在Eureka注册中心进行服务的注册和发现。
> - 也是一个==网关==，请求应该通过Zuul来进行路由。



#### 1. Zuul网关的作用

---

网关作用：

- **统一入口**：为全部服务提供一个唯一的入口，网关起到外部和内部==隔离==的作用，保障了==后台服务==的安全性。
- **鉴权校验**：==识别==每个请求的==权限==，拒绝不符合要求的请求。
- **动态路由**：动态的将请求==路由==到不同的后端集群中。
- 减少客户端与服务端的==耦合==：服务可以独立发展，通过网关层来做映射。

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gse4sy5kulj30po0je0v9.jpg" style="zoom:50%;" />



#### 2. Zuul网关的应用

---

##### 2.1 网关访问方式

通过zuul访问服务的，URL地址默认格式为：http://zuulHostIp:port/服务名称/服务中的URL。

服务名称：配置文件中的spring.application.name。

服务的URL：就是对应的服务对外提供的URL路径监听。

##### 2.2 网关依赖注入

```xml
<!-- spring cloud Eureka Client 启动器 -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-eureka</artifactId>
</dependency>

<!-- zuul -->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>

<!-- zuul网关的重试机制，不是使用ribbon内置的重试机制
   是借助spring-retry组件实现的重试
   开启zuul网关重试机制需要增加下述依赖
 -->
<dependency>
  <groupId>org.springframework.retry</groupId>
  <artifactId>spring-retry</artifactId>
</dependency>
```

##### 2.3 网关启动

```java
@SpringBootApplication
@EnableZuulProxy
public class ZuulApplication {
  public static void main(String[] args) {
    SpringApplication.run(ZuulApplication.class, args);
  }
}
```

`@EnableZuulProxy`开启Zuul网关。当前应用是一个Zuul微服务网关。会在Eureka注册中心注册当前服务。

##### 2.4 URL路径匹配

```yaml
# 参数key结构：zuul.routes.customName.path=xxx
# 其中customName自定义。通常使用要调用的服务名称，方便后期管理

# 可使用的通配符有：
# ? 单个字符
# * 任意多个字符，不包含多级路径
# ** 任意多个字符，包含多级路径
zuul:
  routes:
    eureka-application-service:
      path: /api/**

# 参数key结构：zuul.routes.customName.url=xxx
# url用于配置符合path的请求路径路由到的服务地址。
zuul:
  routes:
    eureka-application-service:
      url: http://127.0.0.1:8080/
```

##### 2.5 服务名称匹配

```yaml
# service id pattern 通过服务名称路由
# key结构 ：zuul.routes.customName.path=xxx
# 路径匹配规则

# key结构 ：zuul.routes.customName.serviceId=xxx
# serviceId用于配置符合path的请求路径路由到的服务名称。
zuul:
  routes:
    eureka-application-service:
      path: /api/**
      serviceId: eureka-application-service
```

##### 2.6 路由排除配置

```yaml
# ignored service id pattern
# 配置不被zuul管理的服务列表。多个服务名称使用逗号','分隔。
# 配置的服务将不被zuul代理。
zuul:
  ignored-services: eureka-application-service
# 此方式相当于给所有新发现的服务默认排除zuul网关访问方式，只有配置了路由网关的服务才可以通过zuul网关访问

# 通配方式配置排除列表。
zuul:
  ignored-services: *

# 使用服务名称匹配规则配置路由列表，相当于只对已配置的服务提供网关代理。
zuul:
  routes:
    eureka-application-service:
      path: /api/**

# 通配方式配置排除网关代理路径。所有符合ignored-patterns的请求路径都不被zuul网关代理。
zuul:
  ignored-patterns: /**/test/**
zuul:
  routes:
    eureka-application-service:
      path: /api/**
```

##### 2.7 路由前缀配置

```yaml
# prefix URL pattern 前缀路由匹配
# 配置请求路径前缀，所有基于此前缀的请求都由zuul网关提供代理。
zuul:
  prefix: /api
# 使用服务名称匹配方式配置请求路径规则。
# 这里的配置将为：http://ip:port/api/appservice/**的请求提供zuul网关代理，可以将要访问服务进行前缀分类。
# 并将请求路由到服务eureka-application-service中。
zuul:
  routes:
    eureka-application-service:
      path: /appservice/**
```

zuul网关其==底层使用ribbon来实现请求的路由==，并内置Hystrix，可选择性提供网关fallback逻辑。使用zuul的时候，并不推荐使用Feign作为application client端的开发实现。毕竟Feign技术是对ribbon的再封装，使用Feign本身会提高通讯消耗，降低通讯效率，只在服务相互调用的时候使用Feign来简化代码开发就够了。商业开发中，使用==Ribbon+RestTemplate==来开发的比例更高。



#### 3. Zuul网关过滤器

---

Zuul提供了过滤器定义，可以用来过滤代理请求，提供额外功能逻辑。如：权限验证，日志记录等。

Zuul提供的过滤器是一个父类==ZuulFilter==。通过父类中定义的抽象方法==filterType==，来决定当前的Filter种类是什么。

- **前置过滤**：请求进入Zuul之后，立刻执行的过滤逻辑。
- **路由后过滤**：请求进入Zuul之后，并Zuul实现了请求路由后执行的过滤逻辑，路由后过滤，是在远程服务调用之前过滤的逻辑。
- **后置过滤**：远程服务调用结束后执行的过滤逻辑。
- **异常过滤**：是任意一个过滤器发生异常或远程服务调用无结果反馈的时候执行的过滤逻辑。无结果反馈，就是远程服务调用超时。

##### 3.1 过滤器实现方式

继承父类ZuulFilter。四个抽象方法如下：

- **filterType()**：方法返回字符串数据，代表当前过滤器的类型。可选值有
  - ==pre==：前置过滤器，在请求被路由前执行，通常用于处理身份认证，日志记录等；
  - ==route==：在路由执行后，服务调用前被调用；
  - ==error==：任意一个filter发生异常的时候执行或远程服务调用没有反馈的时候执行（超时），通常用于处理异常；
  - ==post==：在route或error执行后被调用，一般用于收集服务信息。
- **filterOrder()**：返回int数据，用于为同filterType的多个过滤器定制执行顺序，返回值越小，执行顺序越优先。
- **shouldFilter()**：返回boolean数据，代表当前filter是否生效。
- **run()**：具体的过滤执行逻辑。如pre类型的过滤器，可以通过对请求的验证来决定是否将请求路由到服务上；如post类型的过滤器，可以对服务响应结果做加工处理（如为每个响应增加footer数据）。

##### 3.2 过滤器的生命周期

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsfahwmro1j312m0qi0x1.jpg" style="zoom:80%;" />



#### 4. Zuul网关的容错（与Hystrix的无缝结合）

---

在spring cloud中，Zuul启动器中包含了Hystrix相关依赖，在Zuul网关工程中，默认是提供了Hystrix Dashboard服务监控数据的(hystrix.stream)，但是不会提供监控面板的界面展示。

##### 4.1 Zuul中的服务降级

==Zuul的fallback容错处理逻辑，只针对timeout异常处理，当请求被Zuul路由后，只要服务有返回（包括异常），都不会触发Zuul的fallback容错逻辑==。



#### 5. Zuul网关的限流保护

---

Zuul网关组件提供了限流保护。当请求并发达到阀值，自动触发限流保护，返回错误结果。只要提供error错误处理机制即可。

Zuul的限流保护需要额外依赖`spring-cloud-zuul-ratelimit`组件。

```xml
<dependency>
  <groupId>com.marcosbarbero.cloud</groupId>
  <artifactId>spring-cloud-zuul-ratelimit</artifactId>
</dependency>
```

>全局限流配置：
>
>```yaml
># 开启限流保护
>zuul:
>  ratelimit:
>    enabled: true
>    default-policy:
>      limit: 3 # 60s内请求超过3次，服务端就抛出异常，60s后可以恢复正常请求
>      refresh-interval: 60
>      type: origin # 针对IP进行限流，不影响其他IP

>局部限流配置：
>
>```yaml
># 开启限流保护
>zuul:
>  ratelimit:
>    enabled: true
>    policies:
>      hystrix-application-client:
>        limit: 3 # hystrix-application-client服务60s内请求超过3次，服务抛出异常。
>        refresh-interval: 60
>        type: origin # 针对IP限流。
>```



#### 6. Zuul网关性能调优：网关的两层超时调优

---

当请求通过zuul网关路由到服务，并等待服务返回响应，这个过程中zuul也有超时控制。zuul的底层使用的是Hystrix+ribbon来实现请求路由。

zuul中的==Hystrix内部使用线程池隔离机制提供请求路由实现==。

- 如果Hystrix超时，直接返回超时异常。
- ==如果ribbon超时，同时Hystrix未超时，ribbon会自动进行服务集群轮询重试，直到Hystrix超时为止==。
- 如果Hystrix超时时长小于ribbon超时时长，ribbon不会进行服务集群轮询重试。

```yaml
zuul:
  retryable: true # 开启zuul网关重试

# hystrix超时时间设置
# 线程池隔离，默认超时时间1000ms
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 8000

# ribbon超时时间设置：建议设置比Hystrix小
ribbon:
  ConnectTimeout: 5000 # 请求连接的超时时间: 默认5000ms
  ReadTimeout: 5000 # 请求处理的超时时间: 默认5000ms
  MaxAutoRetries: 1 # 重试次数：表示访问服务集群下原节点（同路径访问）；
  MaxAutoRetriesNextServer: 1 # 表示访问服务集群下其余节点（换台服务器）
  OkToRetryOnAllOperations: true # 开启重试
```

Spring-cloud中的zuul网关重试机制是使用spring-retry实现的。

```xml
<dependency>
  <groupId>org.springframework.retry</groupId>
  <artifactId>spring-retry</artifactId>
</dependency>
```

