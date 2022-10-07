> Feign的主要目标是将Java Http Clients变得简单。



#### 1. 写一个Feign

---

简单的实现一个Feign客户端，首先通过@FeignClient，客户端，其中value为调用其他服务的名称，FeignConfig.class为FeignClient的配置文件，代码如下：

```java
@FeignClient(value = "service-hi",configuration = FeignConfig.class)
public interface SchedualServiceHi {
  @GetMapping(value = "/hi")
  String sayHiFromClientOne(@RequestParam(value = "name") String name);
}
```

其配置文件如下：

```java
@Configuration
public class FeignConfig {

  @Bean
  public Retryer feignRetryer() {
    return new Retryer.Default(100, SECONDS.toMillis(1), 5);
  }

}
```

查看FeignClient的源码，其代码如下：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FeignClient {

  @AliasFor("name")
  String value() default "";

  @Deprecated
  String serviceId() default "";

  String contextId() default "";

  @AliasFor("value")
  String name() default "";

  String qualifier() default "";

  String url() default "";

  boolean decode404() default false;

  Class<?>[] configuration() default {};

  Class<?> fallback() default void.class;

  Class<?> fallbackFactory() default void.class;

  String path() default "";

  boolean primary() default true;

}
```



#### 2. FeignClient的配置

---

##### 2.1 默认的配置类 FeignClientsConfiguration

这个类在spring-cloud-openfeign-core的jar包下，打开这个类，可以发现它是一个配置类，注入了很多的相关配置的bean，包括`feignRetryer`、`FeignLoggerFactory`、`FormattingConversionService`等，其中还包括了`Decoder`、`Encoder`、`Contract`，如果这三个bean在没有注入的情况下，会自动注入默认的配置。

- Decoder feignDecoder：ResponseEntityDecoder(这是对SpringDecoder的封装)
- Encoder feignEncoder：SpringEncoder
- Logger feignLogger：Slf4jLogger
- Contract feignContract：SpringMvcContract
- Feign.Builder feignBuilder：HystrixFeign.Builder

代码如下：

```java
@Configuration
public class FeignClientsConfiguration {

  @Autowired
  private ObjectFactory<HttpMessageConverters> messageConverters;

  @Autowired(required = false)
  private List<AnnotatedParameterProcessor> parameterProcessors = new ArrayList<>();

  @Autowired(required = false)
  private List<FeignFormatterRegistrar> feignFormatterRegistrars = new ArrayList<>();

  @Autowired(required = false)
  private Logger logger;

  @Bean
  @ConditionalOnMissingBean
  public Decoder feignDecoder() {
    return new OptionalDecoder(new ResponseEntityDecoder(new SpringDecoder(this.messageConverters)));
  }

  @Bean
  @ConditionalOnMissingBean
  public Encoder feignEncoder() {
    return new SpringEncoder(this.messageConverters);
  }

  @Bean
  @ConditionalOnMissingBean
  public Contract feignContract(ConversionService feignConversionService) {
    return new SpringMvcContract(this.parameterProcessors, feignConversionService);
  }

  @Bean
  public FormattingConversionService feignConversionService() {
    FormattingConversionService conversionService = new DefaultFormattingConversionService();
    for (FeignFormatterRegistrar feignFormatterRegistrar : feignFormatterRegistrars) {
      feignFormatterRegistrar.registerFormatters(conversionService);
    }
    return conversionService;
  }

  @Configuration
  @ConditionalOnClass({ HystrixCommand.class, HystrixFeign.class })
  protected static class HystrixFeignConfiguration {
    @Bean
    @Scope("prototype")
    @ConditionalOnMissingBean
    @ConditionalOnProperty(name = "feign.hystrix.enabled")
    public Feign.Builder feignHystrixBuilder() {
      return HystrixFeign.builder();
    }
  }

  @Bean
  @ConditionalOnMissingBean
  public Retryer feignRetryer() {
    return Retryer.NEVER_RETRY;
  }

  @Bean
  @Scope("prototype")
  @ConditionalOnMissingBean
  public Feign.Builder feignBuilder(Retryer retryer) {
    return Feign.builder().retryer(retryer);
  }

  @Bean
  @ConditionalOnMissingBean(FeignLoggerFactory.class)
  public FeignLoggerFactory feignLoggerFactory() {
    return new DefaultFeignLoggerFactory(logger);
  }

}
```

##### 2.2 重写配置

可以重写FeignClientsConfiguration中的bean，从而达到自定义配置的目的，比如FeignClientsConfiguration的默认重试次数为Retryer.NEVER_RETRY，即不重试，那么希望做到重写，代码如下：

```java
@Configuration
public class FeignConfig {

  @Bean
  public Retryer feignRetryer() {
    return new Retryer.Default(100, SECONDS.toMillis(1), 5);
  }
}
```

上述代码更改了该FeignClient的重试次数，重试间隔为100ms，最大重试时间为1s，重试次数为5次。



#### 3. Feign 的工作原理

---

feign是一个伪客户端，即它不做任何的请求处理。==Feign通过处理注解生成request==，从而实现简化HTTP API开发的目的，即开发人员可以使用注解的方式定制request api模板，在发送http request请求之前，feign通过处理注解的方式替换掉request模板中的参数，这种实现方式显得更为直接、可理解。

通过包扫描注入FeignClient的bean，该源码在 `FeignClientRegistrar` 类：首先在启动配置上检查是否有@EnableFeignClients注解，如果有该注解，则开启包扫描，扫描被@FeignClient注解接口。代码如下：

```java
private void registerDefaultConfiguration(AnnotationMetadata metadata,
                                          BeanDefinitionRegistry registry) {
  Map<String, Object> defaultAttrs = metadata
    .getAnnotationAttributes(EnableFeignClients.class.getName(), true);

  if (defaultAttrs != null && defaultAttrs.containsKey("defaultConfiguration")) {
    String name;
    if (metadata.hasEnclosingClass()) {
      name = "default." + metadata.getEnclosingClassName();
    }
    else {
      name = "default." + metadata.getClassName();
    }
    registerClientConfiguration(registry, name,
                                defaultAttrs.get("defaultConfiguration"));
  }
}
```

扫描到FeignClient后，将信息取出，以bean的形式注入达到ioc容器中，源码如下：

```java
public void registerFeignClients(AnnotationMetadata metadata,
                                 BeanDefinitionRegistry registry) {
  ClassPathScanningCandidateComponentProvider scanner = getScanner();
  scanner.setResourceLoader(this.resourceLoader);

  Set<String> basePackages;

  Map<String, Object> attrs = metadata
    .getAnnotationAttributes(EnableFeignClients.class.getName());
  AnnotationTypeFilter annotationTypeFilter = new AnnotationTypeFilter(
    FeignClient.class);
  final Class<?>[] clients = attrs == null ? null
    : (Class<?>[]) attrs.get("clients");
  if (clients == null || clients.length == 0) {
    scanner.addIncludeFilter(annotationTypeFilter);
    basePackages = getBasePackages(metadata);
  }
  else {
    final Set<String> clientClasses = new HashSet<>();
    basePackages = new HashSet<>();
    for (Class<?> clazz : clients) {
      basePackages.add(ClassUtils.getPackageName(clazz));
      clientClasses.add(clazz.getCanonicalName());
    }
    AbstractClassTestingTypeFilter filter = new AbstractClassTestingTypeFilter() {
      @Override
      protected boolean match(ClassMetadata metadata) {
        String cleaned = metadata.getClassName().replaceAll("\\$", ".");
        return clientClasses.contains(cleaned);
      }
    };
    scanner.addIncludeFilter(
      new AllTypeFilter(Arrays.asList(filter, annotationTypeFilter)));
  }

  for (String basePackage : basePackages) {
    Set<BeanDefinition> candidateComponents = scanner
      .findCandidateComponents(basePackage);
    for (BeanDefinition candidateComponent : candidateComponents) {
      if (candidateComponent instanceof AnnotatedBeanDefinition) {
        // verify annotated class is an interface
        AnnotatedBeanDefinition beanDefinition = (AnnotatedBeanDefinition) candidateComponent;
        AnnotationMetadata annotationMetadata = beanDefinition.getMetadata();
        Assert.isTrue(annotationMetadata.isInterface(),
                      "@FeignClient can only be specified on an interface");

        Map<String, Object> attributes = annotationMetadata
          .getAnnotationAttributes(
          FeignClient.class.getCanonicalName());

        String name = getClientName(attributes);
        registerClientConfiguration(registry, name,
                                    attributes.get("configuration"));

        registerFeignClient(registry, annotationMetadata, attributes);
      }
    }
  }
}

private void registerFeignClient(BeanDefinitionRegistry registry,
                                 AnnotationMetadata annotationMetadata, Map<String, Object> attributes) {
  String className = annotationMetadata.getClassName();
  BeanDefinitionBuilder definition = BeanDefinitionBuilder
    .genericBeanDefinition(FeignClientFactoryBean.class);
  validate(attributes);
  definition.addPropertyValue("url", getUrl(attributes));
  definition.addPropertyValue("path", getPath(attributes));
  String name = getName(attributes);
  definition.addPropertyValue("name", name);
  String contextId = getContextId(attributes);
  definition.addPropertyValue("contextId", contextId);
  definition.addPropertyValue("type", className);
  definition.addPropertyValue("decode404", attributes.get("decode404"));
  definition.addPropertyValue("fallback", attributes.get("fallback"));
  definition.addPropertyValue("fallbackFactory", attributes.get("fallbackFactory"));
  definition.setAutowireMode(AbstractBeanDefinition.AUTOWIRE_BY_TYPE);

  String alias = contextId + "FeignClient";
  AbstractBeanDefinition beanDefinition = definition.getBeanDefinition();

  boolean primary = (Boolean)attributes.get("primary"); // has a default, won't be null

  beanDefinition.setPrimary(primary);

  String qualifier = getQualifier(attributes);
  if (StringUtils.hasText(qualifier)) {
    alias = qualifier;
  }

  BeanDefinitionHolder holder = new BeanDefinitionHolder(beanDefinition, className,
                                                         new String[] { alias });
  BeanDefinitionReaderUtils.registerBeanDefinition(holder, registry);
}
```

注入bean之后，==通过jdk的代理，当请求Feign Client的方法时会被拦截==，代码在ReflectiveFeign类，代码如下：

```java
public <T> T newInstance(Target<T> target) {
  Map<String, MethodHandler> nameToHandler = targetToHandlersByName.apply(target);
  Map<Method, MethodHandler> methodToHandler = new LinkedHashMap<Method, MethodHandler>();
  List<DefaultMethodHandler> defaultMethodHandlers = new LinkedList<DefaultMethodHandler>();

  for (Method method : target.type().getMethods()) {
    if (method.getDeclaringClass() == Object.class) {
      continue;
    } else if (Util.isDefault(method)) {
      DefaultMethodHandler handler = new DefaultMethodHandler(method);
      defaultMethodHandlers.add(handler);
      methodToHandler.put(method, handler);
    } else {
      methodToHandler.put(method, nameToHandler.get(Feign.configKey(target.type(), method)));
    }
  }
  InvocationHandler handler = factory.create(target, methodToHandler);
  T proxy = (T) Proxy.newProxyInstance(target.type().getClassLoader(),
                                       new Class<?>[] {target.type()}, handler);

  for (DefaultMethodHandler defaultMethodHandler : defaultMethodHandlers) {
    defaultMethodHandler.bindTo(proxy);
  }
  return proxy;
}
```

在SynchronousMethodHandler类进行拦截处理，当被拦截会==根据参数生成RequestTemplate对象==，该对象就是http请求的模板，代码如下：

```java
@Override
public Object invoke(Object[] argv) throws Throwable {
  RequestTemplate template = buildTemplateFromArgs.create(argv);
  Retryer retryer = this.retryer.clone();
  while (true) {
    try {
      return executeAndDecode(template);
    } catch (RetryableException e) {
      try {
        retryer.continueOrPropagate(e);
      } catch (RetryableException th) {
        Throwable cause = th.getCause();
        if (propagationPolicy == UNWRAP && cause != null) {
          throw cause;
        } else {
          throw th;
        }
      }
      if (logLevel != Logger.Level.NONE) {
        logger.logRetry(metadata.configKey(), logLevel);
      }
      continue;
    }
  }
}
```

其中有个executeAndDecode()方法，该方法是==通过RequestTemplate生成Request请求对象，然后根据用client获取response==。

```java
Object executeAndDecode(RequestTemplate template) throws Throwable {
  Request request = targetRequest(template);
  //省略代码
  response = client.execute(request, options);
  //省略代码
}
```



#### 4. Client 组件

---

Client组件是一个非常重要的组件，Feign最终发送request请求以及接收response响应，都是由Client组件完成的，其中Client的实现类，只要有Client.Default，该类==由HttpURLConnnection实现网络请求，另外还支持HttpClient、Okhttp==。

首先来看以下在FeignRibbonClient的自动配置类，FeignRibbonClientAutoConfiguration ，主要在工程启动的时候注入一些bean，其代码如下：

```java
@ConditionalOnClass({ ILoadBalancer.class, Feign.class })
@Configuration
@AutoConfigureBefore(FeignAutoConfiguration.class)
@EnableConfigurationProperties({ FeignHttpClientProperties.class })
//Order is important here, last should be the default, first should be optional
// see https://github.com/spring-cloud/spring-cloud-netflix/issues/2086#issuecomment-316281653
@Import({ HttpClientFeignLoadBalancedConfiguration.class,
         OkHttpFeignLoadBalancedConfiguration.class,
         DefaultFeignLoadBalancedConfiguration.class })
public class FeignRibbonClientAutoConfiguration {

  @Bean
  @Primary
  @ConditionalOnMissingBean
  @ConditionalOnMissingClass("org.springframework.retry.support.RetryTemplate")
  public CachingSpringLoadBalancerFactory cachingLBClientFactory(
    SpringClientFactory factory) {
    return new CachingSpringLoadBalancerFactory(factory);
  }

  @Bean
  @Primary
  @ConditionalOnMissingBean
  @ConditionalOnClass(name = "org.springframework.retry.support.RetryTemplate")
  public CachingSpringLoadBalancerFactory retryabeCachingLBClientFactory(
    SpringClientFactory factory,
    LoadBalancedRetryFactory retryFactory) {
    return new CachingSpringLoadBalancerFactory(factory, retryFactory);
  }

  @Bean
  @ConditionalOnMissingBean
  public Request.Options feignRequestOptions() {
    return LoadBalancerFeignClient.DEFAULT_OPTIONS;
  }
}
```

可以看出导入了 `DefaultFeignLoadBalancedConfiguration` 配置类，代码如下：

```java
@Configuration
class DefaultFeignLoadBalancedConfiguration {
  @Bean
  @ConditionalOnMissingBean
  public Client feignClient(CachingSpringLoadBalancerFactory cachingFactory,
                            SpringClientFactory clientFactory) {
    return new LoadBalancerFeignClient(new Client.Default(null, null),
                                       cachingFactory, clientFactory);
  }
}
```

在缺失配置feignClient的情况下，会自动注入new Client.Default()，跟踪Client.Default()源码，它使用的网络请求框架为 `HttpURLConnection` ，代码如下：

```java
@Override
public Response execute(Request request, Options options) throws IOException {
  HttpURLConnection connection = convertAndSend(request, options);
  return convertResponse(connection, request);
}
```

怎么在feign中使用HttpClient，查看FeignRibbonClientAutoConfiguration的源码，导入了 `HttpClientFeignLoadBalancedConfiguration` 配置类，代码如下：

```java
@Configuration
@ConditionalOnClass(ApacheHttpClient.class)
@ConditionalOnProperty(value = "feign.httpclient.enabled", matchIfMissing = true)
class HttpClientFeignLoadBalancedConfiguration {

  //省略代码...

  @Bean
  @ConditionalOnMissingBean(Client.class)
  public Client feignClient(CachingSpringLoadBalancerFactory cachingFactory,
                            SpringClientFactory clientFactory, HttpClient httpClient) {
    ApacheHttpClient delegate = new ApacheHttpClient(httpClient);
    return new LoadBalancerFeignClient(delegate, cachingFactory, clientFactory);
  }

}
```

从代码@ConditionalOnClass(ApacheHttpClient.class)注解可知道，只需要==在pom文件加上HttpClient的classpath就行了，另外需要在配置文件上加上 feign.httpclient.enabled 为true==，从 @ConditionalOnProperty注解可知，这个可以不写，在默认的情况下就为true。

在pom文件加上：

```xml
<dependency>
  <groupId>io.github.openfeign</groupId>
  <artifactId>feign-httpclient</artifactId>
  <version>11.8</version>
</dependency>
```

同理，如果想要feign使用Okhttp，则只需要在pom文件上加上feign-okhttp的依赖：

```xml
<dependency>
  <groupId>io.github.openfeign</groupId>
  <artifactId>feign-okhttp</artifactId>
  <version>11.8</version>
</dependency>
```



#### 5. feign的负载均衡如何实现

---

通过上述的FeignRibbonClientAutoConfiguration类配置Client的类型(httpurlconnection, okjttp和httpclient)时候，可知最终向容器注入的是`LoadBalancerFeignClient`，即负载均衡客户端。现在来看下LoadBalancerFeignClient的代码：

```java
@Override
public Response execute(Request request, Request.Options options) throws IOException {
  try {
    URI asUri = URI.create(request.url());
    String clientName = asUri.getHost();
    URI uriWithoutHost = cleanUrl(request.url(), clientName);
    FeignLoadBalancer.RibbonRequest ribbonRequest = new FeignLoadBalancer.RibbonRequest(
      this.delegate, request, uriWithoutHost);

    IClientConfig requestConfig = getClientConfig(options, clientName);
    return lbClient(clientName).executeWithLoadBalancer(ribbonRequest,
                                                        requestConfig).toResponse();
  }
  catch (ClientException e) {
    IOException io = findIOException(e);
    if (io != null) {
      throw io;
    }
    throw new RuntimeException(e);
  }
}
```

其中有个executeWithLoadBalancer()方法，即通过负载均衡方式请求。

```java
public T executeWithLoadBalancer(final S request, final IClientConfig requestConfig) throws ClientException {
  LoadBalancerCommand<T> command = buildLoadBalancerCommand(request, requestConfig);

  try {
    return command.submit(
      new ServerOperation<T>() {
        @Override
        public Observable<T> call(Server server) {
          URI finalUri = reconstructURIWithServer(server, request.getUri());
          S requestForServer = (S) request.replaceUri(finalUri);
          try {
            return Observable.just(AbstractLoadBalancerAwareClient.this.execute(requestForServer, requestConfig));
          } 
          catch (Exception e) {
            return Observable.error(e);
          }
        }
      })
      .toBlocking()
      .single();
  } catch (Exception e) {
    Throwable t = e.getCause();
    if (t instanceof ClientException) {
      throw (ClientException) t;
    } else {
      throw new ClientException(e);
    }
  }

}
```

其中服务在submit()方法上，点击submit进入具体的方法，这个方法是LoadBalancerCommand的方法：

```java
// Use the load balancer
Observable<T> o = 
  (server == null ? selectServer() : Observable.just(server))
  .concatMap(new Func1<Server, Observable<T>>() {
    @Override
    // Called for each server being selected
    public Observable<T> call(Server server) {
      context.setServer(server);
      //省略...
    }}                      
```

上述代码中有个selectServe()，该方法是选择服务的进行负载均衡的方法，代码如下：

```java
private Observable<Server> selectServer() {
  return Observable.create(new OnSubscribe<Server>() {
    @Override
    public void call(Subscriber<? super Server> next) {
      try {
        Server server = loadBalancerContext.getServerFromLoadBalancer(loadBalancerURI, loadBalancerKey);   
        next.onNext(server);
        next.onCompleted();
      } catch (Exception e) {
        next.onError(e);
      }
    }
  });
}
```

最终负载均衡交给loadBalancerContext来处理，即之前讲述的Ribbon，在这里不再重复。



#### 6. 总结

---

Feign的源码实现如下：

- 首先通过@EnableFeignClients注解开启FeignClient
- 根据Feign的规则实现接口，并加@FeignClient注解
- 程序启动后，会进行包扫描，扫描所有的@ FeignClient的注解的类，并将这些信息注入到ioc容器中。
- ==当接口的方法被调用，通过jdk的代理，来生成具体的RequesTemplate==
- RequesTemplate再生成Request
- Request交给Client去处理，其中Client可以是HttpUrlConnection、HttpClient也可以是Okhttp
- 最后Client被封装到LoadBalancerClient类，这个类结合类Ribbon做到了负载均衡。
