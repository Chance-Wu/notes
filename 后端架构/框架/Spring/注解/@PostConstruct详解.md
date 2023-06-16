### 一、定义

---

@PostConstruct是Java自带的注解，在方法上加该注解会在项目启动的时候执行该方法，也可以理解为在spring容器初始化的时候执行该方法。

从Java EE5规范开始，Servlet中增加了两个影响Servlet生命周期的注解，@PostConstruct和@PreDestroy，这两个注解被用来**修饰一个非静态的void()方法**。现Bean**初始化之前**和**销毁之前**的自定义操作



### 二、用法

---

PostConstruct 注释用于在**依赖关系注入完成之后需要执行的方法上**，以执行任何初始化。此方法必须在将类放入服务之前调用。支持依赖关系注入的所有类都必须支持此注释。即使类没有请求注入任何资源，用 PostConstruct 注释的方法也必须被调用。只有一个方法可以用此注释进行注释。应用 PostConstruct 注释的方法必须遵守以下所有标准：

- 该方法不得有任何参数，除非是在 EJB 拦截器 (interceptor) 的情况下，根据 EJB 规范的定义，在这种情况下它将带有一个 InvocationContext 对象；
- 该方法的返回类型必须为 void；
- 该方法不得抛出已检查异常；
- 应用 PostConstruct 的方法可以是 public、protected、package private 或 private；
- 除了应用程序客户端之外，该方法不能是 static；
- 该方法可以是 final；如果该方法抛出未检查异常，那么不得将类放入服务中，除非是能够处理异常并可从中恢复的 EJB。



### 三、特点

---

1. 只有一个非静态方法能使用此注解
2. 被注解的方法不得有任何参数
3. 被注解的方法返回值必须为void
4. 被注解方法不得抛出已检查异常
5. 此方法只会被执行一次



### 四、执行顺序

---

![img](img/a5b19abe42021d7db2a183d4e4f73dde.jpeg)

服务器加载Servlet -> servlet 构造函数的加载 -> postConstruct ->init（init是在service 中的初始化方法. 创建service 时发生的事件.） ->Service->destory->predestory->服务器卸载serlvet

那么问题：spring中Constructor、@Autowired、@PostConstruct的顺序
Constructor >> @Autowired >> @PostConstruct

依赖注入的字面意思就可以知道，要将对象p注入到对象a，那么首先就必须得生成对象p与对象a，才能执行注入。所以，如果一个类A中有个成员变量p被@Autowired注解，那么@Autowired注入是发生在A的构造方法执行完之后的。



### 五、应用场景

---

如果想在生成对象时候完成某些初始化操作，而偏偏这些初始化操作又依赖于依赖注入，那么就无法在构造函数中实现。为此，可以使用@PostConstruct注解一个方法来完成初始化，**@PostConstruct注解的方法将会在依赖注入完成后被自动调用**。

```java
@Component
public class SystemConstant {

  public static String surroundings;

  @Value("${spring.profiles.active}")
  public String environment;

  @PostConstruct
  public void initialize() {
    System.out.println("初始化环境...");
    surroundings = this.environment;
  }
}
```



```java
@Component
public class RedisUtil {

  private static RedisTemplate<Object, Object> redisTemplates;

  @Autowired
  private RedisTemplate<Object, Object> redisTemplate;

  @PostConstruct
  public void initialize() {
    redisTemplates = this.redisTemplate;
  }

  /**
   * 添加元素
   *
   * @param key
   * @param value
   */
  public static void set(Object key, Object value) {
    if (key == null || value == null) {
      return;
    }
    redisTemplates.opsForValue().set(key, value);
  }
}
```



### 六、注意事项

---

使用此注解时会影响服务启动时间。服务启动时会扫描WEB-INF/classes的所有文件和WEB-INF/lib下的所有jar包。