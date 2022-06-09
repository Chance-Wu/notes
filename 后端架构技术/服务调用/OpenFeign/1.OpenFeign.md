#### 1. 如何包含Feign

---

```xml
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableFeignClients
public class Application {

  public static void main(String[] args) {
    SpringApplication.run(Application.class, args);
  }

}
```

```java
@FeignClient("stores")
public interface StoreClient {
  @RequestMapping(method = RequestMethod.GET, value = "/stores")
  List<Store> getStores();

  @RequestMapping(method = RequestMethod.GET, value = "/stores")
  Page<Store> getStores(Pageable pageable);

  @RequestMapping(method = RequestMethod.POST, value = "/stores/{storeId}", consumes = "application/json")
  Store update(@PathVariable("storeId") Long storeId, Store store);

  @RequestMapping(method = RequestMethod.DELETE, value = "/stores/{storeId:\\d+}")
  void delete(@PathVariable Long storeId);
}
```

在 `@FeignClient` 注解中，String值（上面的“stores”）是一个任意的客户端名称，用于创建Spring Cloud LoadBalancer 客户端。还可以使用`url`属性（绝对值或仅主机名）指定 URL。应用程序上下文中 bean 的名称是接口的完全限定名称。可以使用注解的`qualifiers`指定feign客户端别名。

上述的负载均衡器客户端想要发现“store”服务的物理地址。

- 如果应用程序是 Eureka 客户端，那么它将解析 Eureka 服务注册表中的服务。
- 如果不想使用 Eureka，可以使用[`SimpleDiscoveryClient`](https://docs.spring.io/spring-cloud-commons/docs/current/reference/html/#simplediscoveryclient).

Spring Cloud OpenFeign 支持 Spring Cloud LoadBalancer 的阻塞模式可用的所有功能。

>要在 `@Configuration`注解类上使用 `@EnableFeignClients` 注解，请确保指定客户端所在的位置。
>
>`@EnableFeignClients(basePackages = "com.example.clients")` 
>
>或者显式地列出：
>
>`@EnableFeignClients(clients = InventoryServiceFeignClient.class)`



#### 2. 覆写feign默认配置

---

Spring Cloud根据需要使用 `FeignClientsConfiguretion` 为每个命名的客户端创建一个新的整体作为 `ApplicationContext` 。这包含（其他）feign.Decoder，feign.Encoder和feign.Contract。

Spring Cloud允许通过使用@FeignClient声明附加配置（在FeignClientsConfiguration之上）来完全控制feign客户端。例：

```java
@FeignClient（name =“stores”，configuration = FooConfiguration.class）
  public  interface StoreClient {
  // .. 
}
```

>FooConfiguration 不需要用 @Configuration注解。
>
>==name 和 url 属性支持占位符。==
>
>```java
>@FeignClient(name = "${feign.name}", url = "${feign.url}")
>public interface StoreClient {
>  //..
>}
>```

Spring Cloud OpenFeign默认为feign提供以下bean：

- `Decoder` feignDecoder： `ResponseEntityDecoder` (包装了一个 `SpringDecoder`)
- `Encoder` feignEncoder： `SpringEncoder`
- `Logger` feignLogger： `Slf4jLogger`
- `MicrometerCapability` micrometerCapability：如果 feign-micrometer 在类路径上并且 MeterRegistry 可用。
- `CachingCapability` cachingCapability：如果使用了@EnableCaching 注解。可以通过 feign.cache.enabled 禁用。
- `Contract` feignContract： `SpringMvcContract`
- `Feign.Builder` feignBuilder： `FeignCircuitBreaker.Builder`
- `Client` feignClient：如果 Spring Cloud LoadBalancer 在类路径上，则使用 FeignBlockingLoadBalancerClient。如果它们都不在类路径上，则使用默认的 feign 客户端。

##### 2.1 SpringEncoder 配置

在SpringEncoder中，为二进制内容类型设置空字符集，为所有其他内容类型设置UTF-8。

可以修改此行为以从Content-Type标头字符集派生字符集，而不是将 feign.encoder.charset-from-content-type 的值设置为true。



#### 3. 超时处理

---

我们可以在默认客户端和命名客户端上配置超时。OpenFeign 使用两个超时参数：

- `connectTimeout`：防止由于服务器处理时间长而阻塞调用者。
- `readTimeout`：从连接建立时开始应用，在返回响应时间过长时触发。



#### 4. 手动创建Feign客户端

---

使用 Feign Builder API创建客户端。如下示例创建了两个具有相同接口的Feign客户端，但为每个客户端配置了一个单独的请求拦截器。

```java
@Import(FeignClientsConfiguration.class)
class FooController {

  private FooClient fooClient;

  private FooClient adminClient;

  @Autowired
  public FooController(Client client, Encoder encoder, Decoder decoder, Contract contract, MicrometerCapability micrometerCapability) {
    this.fooClient = Feign.builder().client(client)
      .encoder(encoder)
      .decoder(decoder)
      .contract(contract)
      .addCapability(micrometerCapability)
      .requestInterceptor(new BasicAuthRequestInterceptor("user", "user"))
      .target(FooClient.class, "https://PROD-SVC");

    this.adminClient = Feign.builder().client(client)
      .encoder(encoder)
      .decoder(decoder)
      .contract(contract)
      .addCapability(micrometerCapability)
      .requestInterceptor(new BasicAuthRequestInterceptor("admin", "admin"))
      .target(FooClient.class, "https://PROD-SVC");
  }
}
```



#### 5. Feign Spring Cloud断路器支持

---

如果 Spring Cloud CircuitBreaker 在 classpath 上并且 `feign.断路器.enabled=true`，Feign 将使用断路器包装所有方法。

要在每个客户端上禁用 Spring Cloud 断路器 支持，请创建一个具有“`prototype`”范围的 vanilla Feign.Builder，例如：

```java
@Configuration
public class FooConfiguration {
  @Bean
  @Scope("prototype")
  public Feign.Builder feignBuilder() {
    return Feign.builder();
  }
}
```

断路器名称遵循此模式`<feignClientClassName>#<calledMethod>(<parameterTypes>)`。当调用一个@FeignClient带有`FooClient`接口并且被调用的没有参数的接口方法`bar`时，断路器名称将是`FooClient#bar()`。

提供一个CircuitBreakerNameResolver的bean，可以更改断路器名称模式。

```java
@Configuration
public class FooConfiguration {
  @Bean
  public CircuitBreakerNameResolver circuitBreakerNameResolver() {
    return (String feignClientName, Target<?> target, Method method) -> feignClientName + "_" + method.getName();
  }
}
```



#### 6. Feign 断路器回退

---

Spring Cloud 断路器 支持回退的概念：当线路打开或出现错误时执行的默认代码路径。 要为给定的@FeignClient 启用 `fallback`，请==将回退属性设置为实现回退的类名==。 还需要将实现声明为 Spring bean。

```java
@FeignClient(name = "testClientWithFactory",
        fallbackFactory = TestFallbackFactory.class)
protected interface TestClientWithFactory {

  @RequestMapping(method = RequestMethod.GET, value = "/hello")
  Hello getHello();

  @RequestMapping(method = RequestMethod.GET, value = "/hellonotfound")
  String getException();

}

@Component
static class TestFallbackFactory implements FallbackFactory<FallbackWithFactory> {

  @Override
  public FallbackWithFactory create(Throwable cause) {
    return new FallbackWithFactory();
  }

}

static class FallbackWithFactory implements TestClientWithFactory {

  @Override
  public Hello getHello() {
    throw new NoFallbackAvailableException("Boom!", new RuntimeException());
  }

  @Override
  public String getException() {
    return "Fixed response";
  }

}
```

如果需要访问触发回退的原因，可以使用@FeignClient中的`fallbackFactory`属性。

```java
@FeignClient(name = "testClientWithFactory",
             fallbackFactory = TestFallbackFactory.class)
protected interface TestClientWithFactory {

  @RequestMapping(method = RequestMethod.GET, value = "/hello")
  Hello getHello();

  @RequestMapping(method = RequestMethod.GET, value = "/hellonotfound")
  String getException();

}

@Component
static class TestFallbackFactory implements FallbackFactory<FallbackWithFactory> {

  @Override
  public FallbackWithFactory create(Throwable cause) {
    return new FallbackWithFactory();
  }

}

static class FallbackWithFactory implements TestClientWithFactory {

  @Override
  public Hello getHello() {
    throw new NoFallbackAvailableException("Boom!", new RuntimeException());
  }

  @Override
  public String getException() {
    return "Fixed response";
  }

}
```



#### 7. Feign和@Primary

---

将 Feign 与 Spring Cloud 断路器回退一起使用时，==ApplicationContext 中有多个相同类型的 bean。这将导致 @Autowired 不起作用==，因为没有一个 bean 或一个标记为主要的。 为了解决这个问题，Spring Cloud OpenFeign 将所有 Feign 实例标记为 @Primary，因此 Spring Framework 将知道要注入哪个 bean。 在某些情况下，这可能是不可取的。可以将 @FeignClient 的主要属性设置为 false 来关闭此行为。

```java
@FeignClient(name = "hello", primary = false)
public interface HelloClient {
  // methods here
}
```



#### 8. Feign继承支持

---

Feign 通过单继承接口支持样板 API。 这允许将常用操作分组到方便的基本接口中。

**UserService.java**

```java
public interface UserService {

  @RequestMapping(method = RequestMethod.GET, value ="/users/{id}")
  User getUser(@PathVariable("id") long id);
}
```

**UserResource.java**

```java
@RestController
public class UserResource implements UserService {

}
```

**UserClient.java**

```java
@FeignClient("users")
public interface UserClient extends UserService {

}
```



#### 9. Feign 请求/响应压缩

---

可以通过配置以下属性来启动Feign请求启动请求/响应GZIP压缩：

```properties
feign.compression.request.enabled=true
feign.compression.response.enabled=true
```

Feign 请求压缩提供以下属性设置，可以选择==压缩媒体类型==和==最小请求阈值长度==。

```properties
feign.compression.request.enabled=true
feign.compression.request.mime-types=text/xml,application/xml,application/json
feign.compression.request.min-request-size=2048
```



#### 10. Feign 日志

---

为每个 Feign 客户端创建一个logger。默认情况下，记录器的名称是用于创建 Feign 客户端的接口的完整类名。 Feign logging 只响应 DEBUG 级别。

```yaml
logging:
  level:
    com:
      forezp:
        client:
          ProviderClient: DEBUG
```

为每个客户端配置==日志级别==告诉Feign记录日志的模式。有如下选择：

- `NONE`：不记录（默认）
- `BASIC`：仅记录请求方法和URL以及响应状态码和执行时间
- `HEADERS`：记录基本信息以及请求和响应标头
- `FULL`：记录请求和响应头、正文和元数据。

```java
@Configuration
public class FooConfiguration {

  @Bean
  Logger.Level feignLoggerLevel() {
    return Logger.Level.FULL;
  }
}
```



#### 11. Feign Capability 支持

---

Feign capabilities公开了核心 Feign 组件，以便可以修改这些组件。例如，这些功能可以获取客户端，对其进行装饰，然后将装饰后的实例返回给 Feign。对metrics库的支持就是一个很好的现实例子。参阅 [Feign metrics](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/#feign-metrics)。

创建一个或多个 Capability bean 并将它们放在 @FeignClient 配置中，您可以注册它们并修改相关客户端的行为。

```java
@Configuration
public class FooConfiguration {
  @Bean
  Capability customCapability() {
    return new CustomCapability();
  }
}
```



#### 12. Feign 指标

---

如果以下所有条件都为真，则会创建并注册一个 MicrometerCapability bean，以便您的 Feign 客户端将指标发布到 Micrometer：

- feign-micrometer在类路径上
- 一个可用的 MeterRegistry bean
- feign metrics属性设置为true（默认）
  - `feign.metrics.enabled=true` （针对所有客户端）
  - `feign.client.config.feignName.metrics.enabled=true` （针对一个客户端）

如果应用程序已经使用了 Micrometer，那么启用指标就像将 feign-micrometer 放到类路径中一样简单。

可以通过以下任一方式禁用该功能：

- 从类路径中排除feign-micrometer
- 将 feign metrics属性之一设置为 false
  - `feign.metrics.enabled=false`
  - `feign.client.config.feignName.metrics.enabled=false`

可以通过注册自己的bean来自定义 MicrometerCapability。

```java
@Configuration
public class FooConfiguration {
  @Bean
  public MicrometerCapability micrometerCapability(MeterRegistry meterRegistry) {
    return new MicrometerCapability(meterRegistry);
  }
}
```



#### 13. Feign 缓存

---

如果使用@EnableCaching 注解，则会创建并注册一个 CachingCapability bean，以便你的 Feign 客户端识别其接口上的 `@Cache*` 注解：

```java
public interface DemoClient {

    @GetMapping("/demo/{filterParam}")
    @Cacheable(cacheNames = "demo-cache", key = "#keyParam")
    String demoEndpoint(String keyParam, @PathVariable String filterParam);
}
```

可以通过属性 feign.cache.enabled=false 禁用该功能。



#### 14. Feign @QueryMap 支持

---

OpenFeign @QueryMap 注解支持 POJO 用作 GET 参数映射。然而默认的 OpenFeign QueryMap 注解与 Spring 不兼容，因为它缺少 value 属性。

Spring Cloud OpenFeign 提供了等价的 `@SpringQueryMap` 注解，用于==将POJO 或Map 参数注解为查询参数映射==。

例如，Params 类定义了参数 param1 和 param2：

```java
public class Params {
  private String param1;
  private String param2;

  // [Getters and setters omitted for brevity]
}
```

以下 feign 客户端通过使用 @SpringQueryMap 注解来使用 Params 类：

```java
@FeignClient("demo")
public interface DemoTemplate {

  @GetMapping(path = "/demo")
  String demoEndpoint(@SpringQueryMap Params params);
}
```



#### 15. HATEOAS 支持

---

Spring 提供了一些 API 来创建遵循 HATEOAS 原则、Spring Hateoas 和 Spring Data REST 的 REST 表示。

如果项目使用 org.springframework.boot:spring-boot-starter-hateoas 启动器或 org.springframework.boot:spring-boot-starter-data-rest 启动器，则默认启用 Feign HATEOAS 支持。

启用 HATEOAS 支持后，允许 Feign 客户端序列化和反序列化 HATEOAS 表示模型： [EntityModel](https://docs.spring.io/spring-hateoas/docs/1.0.0.M1/apidocs/org/springframework/hateoas/EntityModel.html), [CollectionModel](https://docs.spring.io/spring-hateoas/docs/1.0.0.M1/apidocs/org/springframework/hateoas/CollectionModel.html) 和 [PagedModel](https://docs.spring.io/spring-hateoas/docs/1.0.0.M1/apidocs/org/springframework/hateoas/PagedModel.html)。

```java
@FeignClient("demo")
public interface DemoTemplate {

  @GetMapping(path = "/stores")
  CollectionModel<Store> getStores();
}
```



#### 16. Spring @MatrixVariable 支持

---

Spring Cloud OpenFeign支持Spring的 `@MatrixVariable` 注解。

如果将map作为方法参数传递，@MatrixVariable 路径段是通过使用 = 连接映射中的键值对来创建的。

```javascript
@GetMapping("/objects/links/{matrixVars}")
Map<String, List<String>> getObjects(@MatrixVariable Map<String, List<String>> matrixVars);
```

注意，变量名称和路径段占位符都称为 matrixVars。

```java
@FeignClient("demo")
public interface DemoTemplate {

  @GetMapping(path = "/stores")
  CollectionModel<Store> getStores();
}
```



#### 17. Feign CollectionFormat 支持

---

提供 `@CollectionFormat` 注解来支持 feign.CollectionFormat。可以通过传递所需的 feign.CollectionFormat 作为注解值来注释 Feign 客户端方法（或影响整个类的所有方法）。

在以下示例中，使用 CSV 格式而不是默认的 EXPLODED 来处理该方法。

```java
@FeignClient(name = "demo")
protected interface PageableFeignClient {

  @CollectionFormat(feign.CollectionFormat.CSV)
  @GetMapping(path = "/page")
  ResponseEntity performRequest(Pageable page);

}
```

在将 Pageable 作为查询参数发送时设置 CSV 格式，以便正确编码。



#### 18. 早期初始化错误

---

根据使用 Feign 客户端的方式，可能会在启动应用程序时看到初始化错误。要解决此问题，可以在自动装配客户端时使用 `ObjectProvider`。

```java
@Autowired
ObjectProvider<TestFeignClient> testFeignClient;
```



#### 19. Spring Data 支持

---

可以启用 Jackson 模块以支持 org.springframework.data.domain.Page 和 org.springframework.data.domain.Sort 解码。

```properties
feign.autoconfiguration.jackson.enabled=true
```



#### 20. Spring @RefreshScope 支持

---

如果启用了 Feign 客户端刷新，则每个 feign 客户端都会使用 feign.Request.Options 作为刷新范围的 bean 创建。 这意味着可以通过 POST /actuator/refresh 针对任何 Feign 客户端实例刷新诸如 connectTimeout 和 readTimeout 之类的属性。

默认情况下，Feign 客户端中的刷新行为是禁用的。 使用以下属性启用刷新行为：

```properties
feign.client.refresh-enabled=true
```

>不要使用 @RefreshScope 注解来注释 @FeignClient 接口。



#### 21. 配置属性

---

要查看所有 Spring Cloud OpenFeign 相关配置属性的列表，请查看附录页面。

| Name                                          | Default                                         | Description                                                  |
| :-------------------------------------------- | :---------------------------------------------- | :----------------------------------------------------------- |
| feign.autoconfiguration.jackson.enabled       | `false`                                         | If true, PageJacksonModule and SortJacksonModule bean will be provided for Jackson page decoding. |
| feign.circuitbreaker.alphanumeric-ids.enabled | `false`                                         | If true, Circuit Breaker ids will only contain alphanumeric characters to allow for configuration via configuration properties. |
| feign.circuitbreaker.enabled                  | `false`                                         | If true, an OpenFeign client will be wrapped with a Spring Cloud CircuitBreaker circuit breaker. |
| feign.circuitbreaker.group.enabled            | `false`                                         | If true, an OpenFeign client will be wrapped with a Spring Cloud CircuitBreaker circuit breaker with with group. |
| feign.client.config                           |                                                 |                                                              |
| feign.client.decode-slash                     | `true`                                          | Feign clients do not encode slash `/` characters by default. To change this behavior, set the `decodeSlash` to `false`. |
| feign.client.default-config                   | `default`                                       |                                                              |
| feign.client.default-to-properties            | `true`                                          |                                                              |
| feign.client.refresh-enabled                  | `false`                                         | Enables options value refresh capability for Feign.          |
| feign.compression.request.enabled             | `false`                                         | Enables the request sent by Feign to be compressed.          |
| feign.compression.request.mime-types          | `[text/xml, application/xml, application/json]` | The list of supported mime types.                            |
| feign.compression.request.min-request-size    | `2048`                                          | The minimum threshold content size.                          |
| feign.compression.response.enabled            | `false`                                         | Enables the response from Feign to be compressed.            |
| feign.encoder.charset-from-content-type       | `false`                                         | Indicates whether the charset should be derived from the {@code Content-Type} header. |
| feign.httpclient.connection-timeout           | `2000`                                          |                                                              |
| feign.httpclient.connection-timer-repeat      | `3000`                                          |                                                              |
| feign.httpclient.disable-ssl-validation       | `false`                                         |                                                              |
| feign.httpclient.enabled                      | `true`                                          | Enables the use of the Apache HTTP Client by Feign.          |
| feign.httpclient.follow-redirects             | `true`                                          |                                                              |
| feign.httpclient.hc5.enabled                  | `false`                                         | Enables the use of the Apache HTTP Client 5 by Feign.        |
| feign.httpclient.hc5.pool-concurrency-policy  |                                                 | Pool concurrency policies.                                   |
| feign.httpclient.hc5.pool-reuse-policy        |                                                 | Pool connection re-use policies.                             |
| feign.httpclient.hc5.socket-timeout           | `5`                                             | Default value for socket timeout.                            |
| feign.httpclient.hc5.socket-timeout-unit      |                                                 | Default value for socket timeout unit.                       |
| feign.httpclient.max-connections              | `200`                                           |                                                              |
| feign.httpclient.max-connections-per-route    | `50`                                            |                                                              |
| feign.httpclient.ok-http.read-timeout         | `60s`                                           | {@link OkHttpClient} read timeout; defaults to 60 seconds.   |
| feign.httpclient.time-to-live                 | `900`                                           |                                                              |
| feign.httpclient.time-to-live-unit            |                                                 |                                                              |
| feign.metrics.enabled                         | `true`                                          | Enables metrics capability for Feign.                        |
| feign.oauth2.enabled                          | `false`                                         | Enables feign interceptor for managing oauth2 access token.  |
| feign.oauth2.load-balanced                    | `false`                                         | Enables load balancing for oauth2 access token provider.     |
| feign.okhttp.enabled                          | `false`                                         | Enables the use of the OK HTTP Client by Feign.              |
