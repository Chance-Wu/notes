Springboot在静态方法中调用Service或mapper，初始化后会出现空指针错误：java.lang.NullPointerException。

首先这涉及到代码执行优先级的问题，在一个Java类中，存在着静态代码块，静态方法，构造函数，成员方法等等。不同形式的代码执行顺序不同。

执行顺序优先级：

```
静态代码块
>静态方法
>构造函数（此时通过@Autowired修饰的成员变量为null）
>bean注入
>@PostConstruct注解的init函数
```

| 代码块     | 描述                                             |
| ---------- | ------------------------------------------------ |
| 静态代码块 | 当类被载入时，静态代码块被执行，且只被执行一次   |
| 静态方法   | 在不创建对象的情况下即可执行，可以直接用类名调用 |
| 构造函数   | 创建对象的时候执行                               |
| 成员函数   | 当被对象调用的时候执行                           |



### 一、@PostConstruct

---

1. 在当前类上加入@Component注解，使得可以使用注解注入，并交由Spring容器管理。
2. 同时注入要调用的Service以及同类型的静态变量。
3. 使用java自带注解@PostConstruct注释初始化方法中，并在该初始化方法中将注入的对象赋予静态成员变量。
4. 正常使用静态变量调用Service方法。

```java
@Component
public class MessageUtil {

  @Autowired
  private MessageService messageService;

  public static MessageService service;

  @PostConstruct
  public void init() {
    MessageUtil.service = messageService;
  }
}
```



### 二、spring工具类

---

```java
@Component
public class MessageUtil {

  public static MessageService service = SpringUtils.getBean(MessageService.class);

}
```

```java
/**
 * spring工具类 方便在非spring管理环境中获取bean
 */
@Component
public final class SpringUtils implements BeanFactoryPostProcessor, ApplicationContextAware {
  /** Spring应用上下文环境 */
  private static ConfigurableListableBeanFactory beanFactory;

  private static ApplicationContext applicationContext;

  @Override
  public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException 
  {
    SpringUtils.beanFactory = beanFactory;
  }

  @Override
  public void setApplicationContext(ApplicationContext applicationContext) throws BeansException 
  {
    SpringUtils.applicationContext = applicationContext;
  }

  /**
   * 获取对象
   *
   * @param name
   * @return Object 一个以所给名字注册的bean的实例
   * @throws org.springframework.beans.BeansException
   *
   */
  @SuppressWarnings("unchecked")
  public static <T> T getBean(String name) throws BeansException
  {
    return (T) beanFactory.getBean(name);
  }

  /**
   * 获取类型为requiredType的对象
   *
   * @param clz
   * @return
   * @throws org.springframework.beans.BeansException
   *
   */
  public static <T> T getBean(Class<T> clz) throws BeansException
  {
    T result = (T) beanFactory.getBean(clz);
    return result;
  }

  /**
   * 如果BeanFactory包含一个与所给名称匹配的bean定义，则返回true
   *
   * @param name
   * @return boolean
   */
  public static boolean containsBean(String name)
  {
    return beanFactory.containsBean(name);
  }

  /**
   * 判断以给定名字注册的bean定义是一个singleton还是一个prototype。 如果与给定名字相应的bean定义没有被找到，将会抛出一个异常（NoSuchBeanDefinitionException）
   *
   * @param name
   * @return boolean
   * @throws org.springframework.beans.factory.NoSuchBeanDefinitionException
   *
   */
  public static boolean isSingleton(String name) throws NoSuchBeanDefinitionException
  {
    return beanFactory.isSingleton(name);
  }

  /**
   * @param name
   * @return Class 注册对象的类型
   * @throws org.springframework.beans.factory.NoSuchBeanDefinitionException
   *
   */
  public static Class<?> getType(String name) throws NoSuchBeanDefinitionException
  {
    return beanFactory.getType(name);
  }

  /**
   * 如果给定的bean名字在bean定义中有别名，则返回这些别名
   *
   * @param name
   * @return
   * @throws org.springframework.beans.factory.NoSuchBeanDefinitionException
   *
   */
  public static String[] getAliases(String name) throws NoSuchBeanDefinitionException
  {
    return beanFactory.getAliases(name);
  }

  /**
   * 获取aop代理对象
   * 
   * @param invoker
   * @return
   */
  @SuppressWarnings("unchecked")
  public static <T> T getAopProxy(T invoker)
  {
    return (T) AopContext.currentProxy();
  }

  /**
   * 获取当前的环境配置，无配置返回null
   *
   * @return 当前的环境配置
   */
  public static String[] getActiveProfiles()
  {
    return applicationContext.getEnvironment().getActiveProfiles();
  }

  /**
   * 获取当前的环境配置，当有多个环境配置时，只获取第一个
   *
   * @return 当前的环境配置
   */
  public static String getActiveProfile()
  {
    final String[] activeProfiles = getActiveProfiles();
    return StringUtils.isNotEmpty(activeProfiles) ? activeProfiles[0] : null;
  }
}
```
