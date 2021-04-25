> BeanFactory本质上是一个bean工厂或者说bean容器，按照要求生产需要的bean，提供给我们使用。只是在生产bean的过程中，需要解决bean之间的依赖问题，才引入依赖注入这种技术。即==依赖注入是beanFactory生产bean时为了解决bean之间的依赖的一种技术而已==。
>
> ==beanFactory会在bean的生命周期的各个阶段中对bean进行各种管理，并且spring将这些阶段通过各种借口暴露给我们==，让我们可以对bean进行各种处理，只要让bean实现对应的接口，那么spring就会在bean的生命周期调用我们实现的接口来处理该bean。
>
> 1. bean容器的启动阶段；
> 2. 容器中bean的实例化阶段。

#### BeanFactory容器职责——对象注册与依赖绑定

---

BeanFactory就是生成Bean的工厂：

- 业务对象的注册；
- 对象间依赖关系的绑定；

```java
/**
 * 该接口由包含多个bean定义的对象实现，每个定义均由String名称唯一标识。
 * 根据bean的定义，工厂将返回所包含对象的独立实例（Prototype设计模式），
 * 或单个共享实例（Singleton设计模式的替代方案，在这种情况下，实例是工厂范围内的Singleton）。
 * 返回哪种类型的实例取决于bean工厂的配置。
 */
public interface BeanFactory {

  // 用于取消引用FactoryBean实例，并将其与FactoryBean创建的bean区分开来。
  // 例如，如果名为myJndiObject的bean是FactoryBean，则获取＆myJndiObject将返回工厂，而不是工厂返回的实例。
  String FACTORY_BEAN_PREFIX = "&";

  Object getBean(String name) throws BeansException;

  <T> T getBean(String name, Class<T> requiredType) throws BeansException;
  
  // 省略
}
```

>Bean工厂实现应尽可能支持标准bean生命周期接口。 完整的初始化方法及其标准顺序是：
>
> * `BeanNameAware`.setBeanName()
> * `BeanClassLoaderAware`.setBeanClassLoader()
> * `BeanFactoryAware`.setBeanFactory()
> * `EnvironmentAware`.setEnvironment()
> * `EmbeddedValueResolverAware`.setEmbeddedValueResolver()
> * `ResourceLoaderAware`.setResourceLoader()  仅适用于在应用程序上下文中运行时
> * `ApplicationEventPublisherAware`.setApplicationEventPublisher() 仅适用于在应用程序上下文中运行时
> * `MessageSourceAware`.setMessageSource() 仅适用于在应用程序上下文中运行时
> * `ApplicationContextAware`.setApplicationContext() 仅适用于在应用程序上下文中运行时
> * `ServletContextAware`.setServletContext() 仅适用于在应用程序上下文中运行时
> * `BeanPostProcessors`.postProcessBeforeInitialization()和InitializingBean.afterPropertiesSet() 自定义初始化方法定义



#### BeanFactory总结

---

spring容器接管了bean的实例化，通过依赖注入达到了松耦合的效果，同时提供了各种扩展接口，来在bean的生命周期的各个阶段插入自己的代码：

- `BeanFactoryPostProcessor接口`（启动阶段）
- 各种`Aware接口`
- `BeanPostProcessor接口`
- `InitializingBean接口`（@PostConstruct，init-method）
- `DisposableBean接口`（@PreDestory，destory-method）

