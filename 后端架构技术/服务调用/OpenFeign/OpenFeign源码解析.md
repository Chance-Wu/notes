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
  AnnotationConfigApplicationContext context = getContext(name);
  if (BeanFactoryUtils.beanNamesForTypeIncludingAncestors(context,
                                                          type).length > 0) {
    return context.getBean(type);
  }
  return null;
}
```

































