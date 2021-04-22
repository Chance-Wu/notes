> BeanFactory本质上是一个bean工厂或者说bean容器，按照我们的要求生产需要的bean，提供给我们使用。只是在生产bean的过程中，需要解决bean之间的依赖问题，才引入依赖注入这种技术。即==依赖注入是beanFactory生产bean时为了解决bean之间的依赖的一种技术而已==。
>
> ==beanFactory会在bean的生命周期的各个阶段中对bean进行各种管理，并且spring将这些阶段通过各种借口暴露给我们==，让我们可以对bean进行各种处理，只要让bean实现对应的接口，那么spring就会在bean的生命周期调用我们实现的接口来处理该bean。
>
> bean容器的启动——bean在实例化之前，必须是在bean容器启动之后，所有有两个阶段：
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

  String FACTORY_BEAN_PREFIX = "&";

  Object getBean(String name) throws BeansException;

  <T> T getBean(String name, Class<T> requiredType) throws BeansException;

  Object getBean(String name, Object... args) throws BeansException;

  <T> T getBean(Class<T> requiredType) throws BeansException;

  <T> T getBean(Class<T> requiredType, Object... args) throws BeansException;

  <T> ObjectProvider<T> getBeanProvider(Class<T> requiredType);

  <T> ObjectProvider<T> getBeanProvider(ResolvableType requiredType);

  boolean containsBean(String name);

  boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

  boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

  boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

  boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

  @Nullable
  Class<?> getType(String name) throws NoSuchBeanDefinitionException;

  @Nullable
  Class<?> getType(String name, boolean allowFactoryBeanInit) throws NoSuchBeanDefinitionException;

  String[] getAliases(String name);

}
```



#### BeanFactory总结

---

spring容器接管了bean的实例化，通过依赖注入达到了松耦合的效果，同时提供了各种扩展接口，来在bean的生命周期的各个阶段插入自己的代码：

- `BeanFactoryPostProcessor接口`（启动阶段）
- 各种`Aware接口`
- `BeanPostProcessor接口`
- `InitializingBean接口`（@PostConstruct，init-method）
- `DisposableBean接口`（@PreDestory，destory-method）





#### FactoryBean接口

---

实现了FactoryBean接口的bean是一类叫做factory的bean。==spring会在使用getBean()调用获得该bean时，会自动调用该bean的getObject()方法==，所以返回的不是factory这个bean，而是bean.getObject()方法的返回值：

```java
public interface FactoryBean<T> {

  String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

  @Nullable
  T getObject() throws Exception;

  @Nullable
  Class<?> getObjectType();

  default boolean isSingleton() {
    return true;
  }

}
```

> 例子：spring与mybatis的结合
>
> ```xml
> <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
>   <property name="dataSource" ref="dataSource" />
>   <property name="configLocation" value="classpath:config/mybatis-config-master.xml" />
>   <property name="mapperLocations" value="classpath*:config/mappers/master/**/*.xml" />
> </bean>
> ```
>
> SqlSessionFactoryBean实现了FactoryBean接口，所以返回的不是SqlSessionFactoryBean的实例，而是它的SqlSessionFactoryBean.getObject()的返回值。

















