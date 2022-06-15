### Spring是如何找到@FeignClient标注的接口的

---

#### 源码分析

我们在Consumer服务启动类中加了一个注解：@EnableFeignClients

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Import(FeignClientsRegistrar.class)
public @interface EnableFeignClients {
  //省略
}
```

可以看到该注解向容器中导入了 `FeignClientsRegistrar.class` 类，跟踪进入该类：

```java
class FeignClientsRegistrar implements ImportBeanDefinitionRegistrar,
ResourceLoaderAware, EnvironmentAware {

}
```

该类实现了 `ImportBeanDefinitionRegistrar` 接口，该接口有个向spring容器注册组件的方法：

```java
@Override
public void registerBeanDefinitions(AnnotationMetadata metadata,
                                    BeanDefinitionRegistry registry) {
  registerDefaultConfiguration(metadata, registry);
  registerFeignClients(metadata, registry);
}
```

调试发现metadata就是主启动类的信息：

![image-20220608094451246](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/image-20220608094451246.png)

先跟踪第一个方法：==registerDefaultConfiguration(metadata, registry);==

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

上述代码通过主启动类获取 `@EnableFeignClients` 注解的`defaultConfiguration`属性，同时创建一个名叫name的变量，值为：`default.+主启动类的全限定类名`。（default.com.forezp.NacosConsumerApplication）

继续跟进去：

```java
private void registerClientConfiguration(BeanDefinitionRegistry registry, Object name,
                                         Object configuration) {
  BeanDefinitionBuilder builder = BeanDefinitionBuilder
    .genericBeanDefinition(FeignClientSpecification.class);
  builder.addConstructorArgValue(name);
  builder.addConstructorArgValue(configuration);
  registry.registerBeanDefinition(
    name + "." + FeignClientSpecification.class.getSimpleName(),
    builder.getBeanDefinition());
}
```

上述代码第7行==向Spring容器中注册了一个FeignClientSpecification类==，beanName为default.com.forezp.NacosConsumerApplication.FeignClientSpecification

```java
class FeignClientSpecification implements NamedContextFactory.Specification {

  private String name;

  private Class<?>[] configuration;
}
```

@EnableFeignClients注解的defaultConfiguration属性没有配置，所以FeignSpecification的属性值是一个空数组。

接着分析第二个方法：==registerFeignClients(metadata, registry);==

```java
public void registerFeignClients(AnnotationMetadata metadata,
                                 BeanDefinitionRegistry registry) {
  ClassPathScanningCandidateComponentProvider scanner = getScanner();//创建一个扫描器
  scanner.setResourceLoader(this.resourceLoader);

  Set<String> basePackages;

  Map<String, Object> attrs = metadata
    .getAnnotationAttributes(EnableFeignClients.class.getName());//获取EnableFeignClients注解的元素据，因为该注解包含要扫描的路径
  AnnotationTypeFilter annotationTypeFilter = new AnnotationTypeFilter(
    FeignClient.class);//注解过滤器，代表要过滤含有FeignClient.class的类
  final Class<?>[] clients = attrs == null ? null
    : (Class<?>[]) attrs.get("clients");
  if (clients == null || clients.length == 0) {
    scanner.addIncludeFilter(annotationTypeFilter);//扫描器将注解过滤器新增进来了，这个是重点
    basePackages = getBasePackages(metadata);//EnableFeignClients注解的value，basePackages，basePackageClasses属性有值，就扫描这些配置的包名，如果没设置就扫描主启动类的包名，此处配置了"com/forezp/client"，所以这里就扫描com/forezp/client
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

  //遍历所有要扫描的包名
  for (String basePackage : basePackages) {
    Set<BeanDefinition> candidateComponents = scanner
      .findCandidateComponents(basePackage);//通过扫描器，读取包名下所有的类，然后看看哪些是有FeignClient.class注解的，将这些类转成beanDefinition对象
    for (BeanDefinition candidateComponent : candidateComponents) {
      if (candidateComponent instanceof AnnotatedBeanDefinition) {//这些BeanDefinition 都含有FeignClient.class注解
        //验证带注解的类是一个接口
        AnnotatedBeanDefinition beanDefinition = (AnnotatedBeanDefinition) candidateComponent;
        AnnotationMetadata annotationMetadata = beanDefinition.getMetadata();
        Assert.isTrue(annotationMetadata.isInterface(),
                      "@FeignClient can only be specified on an interface");

        Map<String, Object> attributes = annotationMetadata
          .getAnnotationAttributes(
          FeignClient.class.getCanonicalName());

        String name = getClientName(attributes);//获取FeignClient.class的name值，我这里配置的是@FeignClient(name ="nacos-provider" )，nacos-provider
        registerClientConfiguration(registry, name,
                                    attributes.get("configuration"));//这个跟之前分析的第一个方法逻辑是一样的：向容器注入了一个name=nacos-provider.feignClientSpecification 类型feignClientSpecification的bean

        registerFeignClient(registry, annotationMetadata, attributes);//注册所有的带有@FeignClient(name ="nacos-provider" )注解的bean
      }
    }
  }
}
```

> 上述代码总结，主要做了：
>
> 1. 创建了一个扫描器ClassPathScanningCandidateComponentProvider scanner，并将AnnotationTypeFilter过滤器传入扫描器中（scanner.addIncludeFilter(annotation)），用于过滤扫描到的带有@FeignClient注解的类。
>
> 2. 获取要扫描的包路径：basePackages = getBasePackages(metadata);
>
>    ![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200526154449851-379401932.png)
>
> 3. 遍历包名获取候选的带有FeignClient注解的类的信息 `Set<BeanDefinition> candidateComponents = scanner.findCandidateComponents(basePackage);`
>
>    ![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200526163726579-1469328701.png)
>
>    因为扫描器之前加了有个注解过滤器，这里isCandidateComponent(metadataReader)的逻辑是使用注解过滤器来过滤出带有FeignClient注解的类。
>
>    ![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200526164008297-65241780.png)
>
> 4. 遍历扫描到的BeanDefinition，向ioc容器中注册对应的bean，这里每个带有@FeignClient的类都会注册2个bean。
>
>    ![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200526164508576-770195555.png)
>
>    registerClientConfiguration(registry, name,attributes.get("configuration")); 向容器中注册了name=product.feignClientSpecification 类型feignClientSpecification的bean
>
>    重点是： `registerFeignClient(registry, annotationMetadata, attributes);`方法
>
>    ![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200526165316175-893622153.png)
>
>     所以，该方法向spring容器注入的bean，名字为：com.forezp.client.ProviderClient，类型为==FeignClientFactoryBean==，而FactoryBean都会提供一个getObject方法获取对应的实例
>

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200526172820932-1762044602.png)

### springboot自动装配机制

---

springboot启动时会加载类路径下**/META-INF/spring.factories**中key为`org.springframework.boot.autoconfigure.EnableAutoConfiguration`的类。

![image-20220609165001071](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/image-20220609165001071.png)

![image-20220609165435729](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/image-20220609165435729.png)

有四个相关配置类会被加载，关注 `FeignRibbonClientAutoConfiguration` 和 `FeignAutoConfiguration`。

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200526175522397-1838922478.png)

再看看DefaultFeignLoadBalancedConfiguration这个类：

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200526175741400-1768409166.png)

之后看看这个bean: FeignAutoConfiguration

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200526180003166-91113935.png)

#### 自动装配机制注入的bean

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200526181141287-1353391899.png)

![1365950-20200526181237761-375257599](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200526181237761-375257599.png)



### 获取第三方服务实例

---

每个带有@FeignClient注解的接口都会被注入到spring容器，名字为接口的==全限定类名==，类型为 `FeignClientFactoryBean` 。

```java
@FeignClient(name = "nacos-provider", fallback = ProviderClientBack.class)
public interface ProviderClient {//该接口注入到spring容器中，类型变成了FeignClientFactoryBean，beanName为其全限定类名

  @GetMapping("/hi")
  @Cacheable(cacheNames = "hi-cache", key = "#name")
  String hi(@RequestParam(value = "name", defaultValue = "forezp", required = false) String name);
}
```

debug调试，从容器中根据beanName获取对应的实例看看，我这里的beanName=com.forezp.client.ProviderClient。尝试从容器中获取该对象。

```java
@RestController
public class ConsumerController {

  //    @Autowired
  //    ProviderClient providerClient;
  //先别注入ProviderService，不然下面获取的bean就是从缓存拿了，看不到实际的创建效果
  @Autowired
  private ApplicationContext context;

  @GetMapping("/hi-feign")
  public String hiFeign() {
    Object bean = context.getBean("com.forezp.client.ProviderClient");
    System.out.println(bean.getClass());
    return null;
  }
}
```

跟进该方法：

```java
@Override
public Object getBean(String name) throws BeansException {
  assertBeanFactoryActive();
  return getBeanFactory().getBean(name);
}
```

```java
@Override
public Object getBean(String name) throws BeansException {
  return doGetBean(name, null, null, false);
}
```

```java
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
                          @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {

  final String beanName = transformedBeanName(name);
  Object bean;

  // Eagerly check singleton cache for manually registered singletons.
  Object sharedInstance = getSingleton(beanName);
  if (sharedInstance != null && args == null) {
    if (logger.isTraceEnabled()) {
      if (isSingletonCurrentlyInCreation(beanName)) {
        logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +
                     "' that is not fully initialized yet - a consequence of a circular reference");
      }
      else {
        logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
      }
    }
    bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
  }

```

对于factoryBean，==如果beanName前面带上“&”符号，获取的bean就是原始的factoryBean==，==如果没带该符号，获取的是factoryBean.getObject()方法返回的bean==，这次调试是没带上该符号的。

```java
protected Object getObjectForBeanInstance(
  Object beanInstance, String name, String beanName, @Nullable RootBeanDefinition mbd) {

  // 判断那么是否带上“&”符号作为前缀，这次没有带上
  if (BeanFactoryUtils.isFactoryDereference(name)) {
    if (beanInstance instanceof NullBean) {
      return beanInstance;
    }
    if (!(beanInstance instanceof FactoryBean)) {
      throw new BeanIsNotAFactoryException(transformedBeanName(name), beanInstance.getClass());
    }
  }

  if (!(beanInstance instanceof FactoryBean) || BeanFactoryUtils.isFactoryDereference(name)) {
    return beanInstance;
  }

  Object object = null;
  if (mbd == null) {
    //先从缓存中获取
    object = getCachedObjectForFactoryBean(beanName);
  }
  if (object == null) {
    FactoryBean<?> factory = (FactoryBean<?>) beanInstance;
    if (mbd == null && containsBeanDefinition(beanName)) {
      mbd = getMergedLocalBeanDefinition(beanName);
    }
    boolean synthetic = (mbd != null && mbd.isSynthetic());
    // 第一次缓存还没有，所以调用该方法
    object = getObjectFromFactoryBean(factory, beanName, !synthetic);
  }
  return object;
}
```

```java
protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {
  if (factory.isSingleton() && containsSingleton(beanName)) {
    synchronized (getSingletonMutex()) {
      Object object = this.factoryBeanObjectCache.get(beanName);
      if (object == null) {
        object = doGetObjectFromFactoryBean(factory, beanName);
        
```

```java
private Object doGetObjectFromFactoryBean(final FactoryBean<?> factory, final String beanName)
  throws BeanCreationException {

  Object object;
  try {
    if (System.getSecurityManager() != null) {
      AccessControlContext acc = getAccessControlContext();
      try {
        object = AccessController.doPrivileged((PrivilegedExceptionAction<Object>) factory::getObject, acc);
      }
      catch (PrivilegedActionException pae) {
        throw pae.getException();
      }
    }
    else {
      //这里看到，最终都是调用了factoryBean的getObject()方法获取对应的实例
      object = factory.getObject();
    }
  }
  
```

至此，我们可以知道，所有带有@FeignClient注解的接口获取实例，最终都是调用FeignClientFactoryBean的getObject方法，该方法才是我们的重点。

`FeignClientFactoryBean的getObject()`方法的讲解：

```java
@Override
public Object getObject() throws Exception {
  return getTarget();
}
```

```java
<T> T getTarget() {
  FeignContext context = applicationContext.getBean(FeignContext.class);//springboot自动装配的时候，该FeignContext被注入了spring容器，所以才能从容器中取出来
  Feign.Builder builder = feign(context);

  if (!StringUtils.hasText(this.url)) {
    if (!this.name.startsWith("http")) {
      url = "http://" + this.name;
    }
    else {
      url = this.name;
    }
    url += cleanPath();
    return (T) loadBalance(builder, context, new HardCodedTarget<>(this.type,
                                                                   this.name, url));
  }
  if (StringUtils.hasText(this.url) && !this.url.startsWith("http")) {
    this.url = "http://" + this.url;
  }

```

分析：Feign.Builder builder = feign(context)

```java
protected Feign.Builder feign(FeignContext context) {
  FeignLoggerFactory loggerFactory = get(context, FeignLoggerFactory.class);
  Logger logger = loggerFactory.create(this.type);

  Feign.Builder builder = get(context, Feign.Builder.class)
    // required values
    .logger(logger)
    .encoder(get(context, Encoder.class))
    .decoder(get(context, Decoder.class))
    .contract(get(context, Contract.class));

  configureFeign(context, builder);

  return builder;
}
```

上面很多bean都是调用get(Context.xxx.class)方法获取的，而context就是FeignContext，在springBoot自动装配时注入了spring容器，那么我们跟踪第一个方法看看：

```java
protected <T> T get(FeignContext context, Class<T> type) {
  T instance = context.getInstance(this.contextId, type);
  if (instance == null) {
    throw new IllegalStateException("No bean found of type " + type + " for "
                                    + this.contextId);
  }
  return instance;
}
```

```java
public <T> T getInstance(String name, Class<T> type) {
  AnnotationConfigApplicationContext context = getContext(name);//通过一个服务名获取一个spring容器
  if (BeanFactoryUtils.beanNamesForTypeIncludingAncestors(context,
                                                          type).length > 0) {
    return context.getBean(type);//从容器中获取对应的bean
  }
  return null;
}
```

这里有两个疑问，这个容器是怎么创建的，为何能通过该容器获取对应的bean

```java
protected AnnotationConfigApplicationContext getContext(String name) {
  if (!this.contexts.containsKey(name)) {
    synchronized (this.contexts) {
      if (!this.contexts.containsKey(name)) {
        this.contexts.put(name, createContext(name));//先从一个叫contexts的map中根据服务名获取spring子容器，获取不到，就创建
      }
    }
  }
  return this.contexts.get(name);
}
```

由于上面的获取子容器的方法是在FeignContext中的，所以我们要分析下FeignContext的创建过程：

```java
public class FeignContext extends NamedContextFactory<FeignClientSpecification> {

  public FeignContext() {
    super(FeignClientsConfiguration.class, "feign", "feign.client.name");
  }

}
```

FeignContext创建时调用了父类的有参构造器

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200527111416030-2049294024.png)

此时再回到，通过服务名称获取子容器的方法，获取不到就创建一个子容器：

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200527112327650-71920523.png)

由图可知，子容器注入了一个FeignClientsConfiguration配置类，该类是由FeignContext初始化时，调用父类有参构造器传入的，所以分析下该配置类：

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

通过该配置类，可以知道，子容器中注入了Decoder，Encoder，Contract，FormattingConversionService，Retryer，FeignLoggerFactory，Module

此时我们再看看父容器和子容器注入的主要类：

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200527114516898-1274808589.png)

有了上面的分析，我们回到之前的方法继续分析：

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200527114748217-598895779.png)

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200527115659510-1807179178.png)

我们看到该方法：configureUsingProperties(properties.getConfig().get(this.contextId),builder);，通过一个ContextId也就是服务名，我这里是product，获取该服务的专有配置，说明每一个服务都可以拥有自己的特殊配置；
看看这个FeignClientProperties类：

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200527120702728-471820477.png)

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200527115813637-592059065.png)

再次回到FactoryBean的getObject()方法进行分析：

```java
<T> T getTarget() {
		FeignContext context = applicationContext.getBean(FeignContext.class);
		Feign.Builder builder = feign(context);

		if (!StringUtils.hasText(this.url)) {
			if (!this.name.startsWith("http")) {
				url = "http://" + this.name;
			}
			else {
				url = this.name;
			}
			url += cleanPath();
			return (T) loadBalance(builder, context, new HardCodedTarget<>(this.type,
					this.name, url));//ProviderClient接口被封装成一个HardCodTarget类，传入了类型，名字，url，其中url=http://nacos-provider
		}
  
```

```java
protected <T> T loadBalance(Feign.Builder builder, FeignContext context,
                s            HardCodedTarget<T> target) {
  Client client = getOptional(context, Client.class);
  if (client != null) {
    builder.client(client);
    Targeter targeter = get(context, Targeter.class);
    return targeter.target(this, builder, context, target);
  }

  throw new IllegalStateException(
    "No Feign Client for loadBalancing defined. Did you forget to include spring-cloud-starter-netflix-ribbon?");
}
```

上述代码中可以看出，从容器中获取了一个client和targeter，这2个bean是在父容器中获取的。

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200527121546363-443752105.png)

继续跟进：

```java
class HystrixTargeter implements Targeter {

  @Override
  public <T> T target(FeignClientFactoryBean factory, Feign.Builder feign, FeignContext context,
                      Target.HardCodedTarget<T> target) {
    if (!(feign instanceof feign.hystrix.HystrixFeign.Builder)) {
      return feign.target(target);//由于没有配置hystrix相关配置，走到这个分支
    }
    feign.hystrix.HystrixFeign.Builder builder = (feign.hystrix.HystrixFeign.Builder) feign;
    SetterFactory setterFactory = getOptional(factory.getName(), context,
                                              SetterFactory.class);
    if (setterFactory != null) {
      builder.setterFactory(setterFactory);
    }
    Class<?> fallback = factory.getFallback();
    if (fallback != void.class) {
      return targetWithFallback(factory.getName(), context, target, builder, fallback);
    }
    Class<?> fallbackFactory = factory.getFallbackFactory();
    if (fallbackFactory != void.class) {
      return targetWithFallbackFactory(factory.getName(), context, target, builder, fallbackFactory);
    }

    return feign.target(target);
  }
```

```java
public <T> T target(Target<T> target) {
  return build().newInstance(target);
}
```

```java
public Feign build() {
  SynchronousMethodHandler.Factory synchronousMethodHandlerFactory =
    new SynchronousMethodHandler.Factory(client, retryer, requestInterceptors, logger,
                                         logLevel, decode404, closeAfterDecode, propagationPolicy);
  ParseHandlersByName handlersByName =
    new ParseHandlersByName(contract, options, encoder, decoder, queryMapEncoder,
                            errorDecoder, synchronousMethodHandlerFactory);
  return new ReflectiveFeign(handlersByName, invocationHandlerFactory, queryMapEncoder);
}
```

可见是调用了ReflectiveFeign的newInstance(target);方法。

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200527141214470-194594823.png)

至此，我们可以发现，最终是调用了JDK动态代理机制来生成代理类，下面再来分析上面2个方法：

- `Map<String, MethodHandler> nameToHandler = targetToHandlersByName.apply(target);`
- `InvocationHandler handler = factory.create(target, methodToHandler);`

先看第一个方法

```java
public Map<String, MethodHandler> apply(Target key) {
      List<MethodMetadata> metadata = contract.parseAndValidatateMetadata(key.type());
      Map<String, MethodHandler> result = new LinkedHashMap<String, MethodHandler>();
```

```java
@Override
public List<MethodMetadata> parseAndValidatateMetadata(Class<?> targetType) {
  //做一些校验，必须是一个接口
  checkState(targetType.getTypeParameters().length == 0, "Parameterized types unsupported: %s",
             targetType.getSimpleName());
  checkState(targetType.getInterfaces().length <= 1, "Only single inheritance supported: %s",
             targetType.getSimpleName());
  if (targetType.getInterfaces().length == 1) {
    checkState(targetType.getInterfaces()[0].getInterfaces().length == 0,
               "Only single-level inheritance supported: %s",
               targetType.getSimpleName());
  }
  Map<String, MethodMetadata> result = new LinkedHashMap<String, MethodMetadata>();
  for (Method method : targetType.getMethods()) {
    if (method.getDeclaringClass() == Object.class ||
        (method.getModifiers() & Modifier.STATIC) != 0 ||
        Util.isDefault(method)) {
      continue;
    }
    //遍历ProviderClient所有的方法，对业务方法调用
    MethodMetadata metadata = parseAndValidateMetadata(targetType, method);
    checkState(!result.containsKey(metadata.configKey()), "Overrides unsupported: %s",
               metadata.configKey());
    result.put(metadata.configKey(), metadata);
  }
  return new ArrayList<>(result.values());
}
```

这里只有一个方法：

```java
@FeignClient(name = "nacos-provider", fallback = ProviderClientBack.class)
public interface ProviderClient {

  @GetMapping("/hi")
  @Cacheable(cacheNames = "hi-cache", key = "#name")
  String hi(@RequestParam(value = "name", defaultValue = "forezp", required = false) String name);
}
```

所以跟进去：

```java
@Override
public MethodMetadata parseAndValidateMetadata(Class<?> targetType, Method method) {
  this.processedMethods.put(Feign.configKey(targetType, method), method);
  MethodMetadata md = super.parseAndValidateMetadata(targetType, method);

  RequestMapping classAnnotation = findMergedAnnotation(targetType,
                                                        RequestMapping.class);
  if (classAnnotation != null) {
    // produces - use from class annotation only if method has not specified this
    if (!md.template().headers().containsKey(ACCEPT)) {
      parseProduces(md, method, classAnnotation);
    }

    // consumes -- use from class annotation only if method has not specified this
    if (!md.template().headers().containsKey(CONTENT_TYPE)) {
      parseConsumes(md, method, classAnnotation);
    }

    // headers -- class annotation is inherited to methods, always write these if
    // present
    parseHeaders(md, method, classAnnotation);
  }
  return md;
}
```

可见，校验是对RequestMapping注解的校验

```java
protected MethodMetadata parseAndValidateMetadata(Class<?> targetType, Method method) {
      MethodMetadata data = new MethodMetadata();
      data.returnType(Types.resolve(targetType, targetType, method.getGenericReturnType()));
      data.configKey(Feign.configKey(targetType, method));

      if (targetType.getInterfaces().length == 1) {
        processAnnotationOnClass(data, targetType.getInterfaces()[0]);
      }
      processAnnotationOnClass(data, targetType);//对ProviderClient上的@RequestMapping注解处理


      for (Annotation methodAnnotation : method.getAnnotations()) {
        processAnnotationOnMethod(data, methodAnnotation, method);//对方法上的@RequestMapping注解处理
      }
```

分析对方法上的处理：

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200527143204770-836017566.png)

之后我们再分析`InvocationHandler handler = factory.create(target, methodToHandler);`方法：

```java
static final class Default implements InvocationHandlerFactory {

  @Override
  public InvocationHandler create(Target target, Map<Method, MethodHandler> dispatch) {
    return new ReflectiveFeign.FeignInvocationHandler(target, dispatch);//ProviderClient的所有方法都被封装到dispatch中
  }
}
```

```java
FeignInvocationHandler(Target target, Map<Method, MethodHandler> dispatch) {
  this.target = checkNotNull(target, "target");
  this.dispatch = checkNotNull(dispatch, "dispatch for %s", target);
}
```

对上述分析做个总结：

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200527145706199-1343236557.png)



### 如何通过feign调用到第三方服务的

在这里就是如何通过Consumer服务调用到provider服务。

调用任何服务都要知道==ip和端口==，因此，如何获取ip和端口，如果服务提供者有多个，如何进行负载均衡，debug调试下：

```java
static class FeignInvocationHandler implements InvocationHandler {

    private final Target target;
    private final Map<Method, MethodHandler> dispatch;//被InvocationHandler拦截了，ProviderClient的所有业务方法都封装到该类了

    FeignInvocationHandler(Target target, Map<Method, MethodHandler> dispatch) {
      this.target = checkNotNull(target, "target");
      this.dispatch = checkNotNull(dispatch, "dispatch for %s", target);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      if ("equals".equals(method.getName())) {
        try {
          Object otherHandler =
              args.length > 0 && args[0] != null ? Proxy.getInvocationHandler(args[0]) : null;
          return equals(otherHandler);
        } catch (IllegalArgumentException e) {
          return false;
        }
      } else if ("hashCode".equals(method.getName())) {
        return hashCode();
      } else if ("toString".equals(method.getName())) {
        return toString();
      }

      return dispatch.get(method).invoke(args);//获取对应的方法进行调用
    }
```

```java
@Override
  public Object invoke(Object[] argv) throws Throwable {
    RequestTemplate template = buildTemplateFromArgs.create(argv);
    Retryer retryer = this.retryer.clone();
    while (true) {
      try {
        return executeAndDecode(template);
```

```java
Object executeAndDecode(RequestTemplate template) throws Throwable {
    Request request = targetRequest(template);//此时请求的url还没换成ip和端口

    if (logLevel != Logger.Level.NONE) {
      logger.logRequest(metadata.configKey(), logLevel, request);
    }

    Response response;
    long start = System.nanoTime();
    try {
      response = client.execute(request, options);//调用的客户端，springboot自动装配时注入容器的
```

```java
@Override
	public Response execute(Request request, Request.Options options) throws IOException {
		try {
			URI asUri = URI.create(request.url());
			String clientName = asUri.getHost();//要调用的服务名字，这里我们调用nacos-provider服务
			URI uriWithoutHost = cleanUrl(request.url(), clientName);
			FeignLoadBalancer.RibbonRequest ribbonRequest = new FeignLoadBalancer.RibbonRequest(
					this.delegate, request, uriWithoutHost);

			IClientConfig requestConfig = getClientConfig(options, clientName);//获取nacos-provider相关配置
			return lbClient(clientName).executeWithLoadBalancer(ribbonRequest,
					requestConfig).toResponse();
		}
```

```java
public T executeWithLoadBalancer(final S request, final IClientConfig requestConfig) throws ClientException {
        LoadBalancerCommand<T> command = buildLoadBalancerCommand(request, requestConfig);

        try {
            return command.submit(
                new ServerOperation<T>() {
                    @Override
                    public Observable<T> call(Server server) {
                        URI finalUri = reconstructURIWithServer(server, request.getUri());
                        S requestForServer = (S) request.replaceUri(finalUri);//这里最终会传一个server过来，该server就含有对应的ip和端口了，然后就可以替换url变成最终的形式
                        try {
                            return Observable.just(AbstractLoadBalancerAwareClient.this.execute(requestForServer, requestConfig));//这里用了RXJAVA的技术
                        } 
                        catch (Exception e) {
                            return Observable.error(e);
                        }
                    }
                })
                .toBlocking()
                .single();
```

看看server是如何获取到的，这里用到ribbon进行负载均衡，点击command.submit

```java
public Observable<T> submit(final ServerOperation<T> operation) {
        final ExecutionInfoContext context = new ExecutionInfoContext();
        
        if (listenerInvoker != null) {
            try {
                listenerInvoker.onExecutionStart();
            } catch (AbortExecutionException e) {
                return Observable.error(e);
            }
        }

        final int maxRetrysSame = retryHandler.getMaxRetriesOnSameServer();
        final int maxRetrysNext = retryHandler.getMaxRetriesOnNextServer();

        // Use the load balancer
  			//server为null，所以会调用选择server方法
        Observable<T> o = 
                (server == null ? selectServer() : Observable.just(server))
                .concatMap(new Func1<Server, Observable<T>>() {
                    @Override
                    // Called for each server being selected
                    public Observable<T> call(Server server) {
                        context.setServer(server);
```

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

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200527152727101-1205452518.png)

上图是ribbon通过负载均衡获取单个服务实例的过程。

```java
public T executeWithLoadBalancer(final S request, final IClientConfig requestConfig) throws ClientException {
        LoadBalancerCommand<T> command = buildLoadBalancerCommand(request, requestConfig);

        try {
            return command.submit(
                new ServerOperation<T>() {
                    @Override
                    public Observable<T> call(Server server) {
                        URI finalUri = reconstructURIWithServer(server, request.getUri());//请求url
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
```

继续跟踪到达了

```java
@Override
public RibbonResponse execute(RibbonRequest request, IClientConfig configOverride)
  throws IOException {
  Request.Options options;
  if (configOverride != null) {
    RibbonProperties override = RibbonProperties.from(configOverride);
    options = new Request.Options(
      override.connectTimeout(this.connectTimeout),
      override.readTimeout(this.readTimeout));
  }
  else {
    options = new Request.Options(this.connectTimeout, this.readTimeout);
  }
  Response response = request.client().execute(request.toRequest(), options);
  return new RibbonResponse(request.getUri(), response);
}
```

```java
@Override
public Response execute(Request request, Options options) throws IOException {
  HttpURLConnection connection = convertAndSend(request, options);
  return convertResponse(connection, request);
}
```

最终是使用HttpURLConnection发起请求：

```java
@FeignClient(name = "nacos-provider", fallback = ProviderClientBack.class)
public interface ProviderClient {

  @GetMapping("/hi")
  @Cacheable(cacheNames = "hi-cache", key = "#name")
  String hi(@RequestParam(value = "name", defaultValue = "forezp", required = false) String name);
}
```

![img](OpenFeign%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.assets/1365950-20200527154109772-1871052808.png)
