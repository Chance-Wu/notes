### Spring是如何找到@FeignClient标注的接口的

---

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
                                    attributes.get("configuration"));

        registerFeignClient(registry, annotationMetadata, attributes);
      }
    }
  }
}
```













































