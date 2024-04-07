>- **kill -15** ：kill 指令默认就是 - 15，知识发送一个`SIGTERM`信号通知进程终止，由进程`自行决定`怎么做，即**进程不一定终止**。一般不直接使用 kill -15，不一定能够终止进程。
>- **kill -9**：强制终止进程，进程会被立刻终止。kill -9 **过于暴力**，往往会出现事务执行、业务处理中断的情况，导致数据库中存在`脏数据`、系统中存在残留文件等情况。如果要使用 kill -9，尽量先使用 kill -15 给进程一个处理善后的机会。`该命令可以模拟一次系统宕机，系统断电等极端情况。`
>- **kill -2**：类似 Ctrl + C 退出，会先保存相关数据再终止进程。kill -2 立刻`终止正在执行的代码`->`保存数据`->`终止进程`，只是在进程终止之前会保存相关数据，依然会出现事务执行、业务处理中断的情况，做不到优雅停机。

### springboot的优雅停机

---

在最新版的`Spring Boot 2.3`中终于集成了优雅退出（Graceful shutdown），在官方文档中可以看到内置的 web 服务器（`Jetty、Reactor Netty、Tomcat 和 Undertow`）以及反应式和基于 Servlet 的 web 应用程序都支持优雅退出功能。当`server.shutdown=graceful`启用时，在 web 容器关闭时，web 服务器将不再接收新请求，并将等待活动请求完成的缓冲期。缓冲期 `timeout-per-shutdown-phase` 配置
默认时间为 20s, 意味着最大等待 20s，超时无论线程任务是否执行完毕都会停机处理，一定要合理设置缓冲期大小。

使用方式很简单，只需要配置一下`yml`文件即可：

```yaml
server:
  shutdown: graceful #开启优雅停机，默认是立即停机IMMEDIATE
spring:
  lifecycle:
    timeout-per-shutdown-phase: 20s #缓冲器即最大等待时间
```

#### 优雅停机的目的

- 优雅停机是指为确保应用关闭时，通知应用进程**释放**所占用的资源。
-  `线程池`，shutdown（不接受新任务等待处理完）还是 shutdownNow（调用 Thread.interrupt 进行中断）。
- socket 链接，比如：netty、`jmq、fmq`。（需要着重处理）
- 告知注册中心快速下线，比如`jsf`。（需要着重处理）
- 清理临时文件。
- 各种堆内堆外内存释放。

总之，进程**强行终止**会带来**数据丢失**或者终端无法恢复到正常状态，在分布式环境下可能导致数据不一致的情况。

#### jvm如何接受处理linux信号量的

当jvm启动时就加载了自定义SignalHandler，关闭jvm时触发对应的handle。

```java
public interface SignalHandler {
  SignalHandler SIG_DFL = new NativeSignalHandler(0L);
  SignalHandler SIG_IGN = new NativeSignalHandler(1L);

  void handle(Signal var1);
}
```

```java
class Terminator {
  private static SignalHandler handler = null;

  Terminator() {
  }
  //jvm设置SignalHandler，在System.initializeSystemClass中触发
  static void setup() {
    if (handler == null) {
      SignalHandler var0 = new SignalHandler() {
        public void handle(Signal var1) {
          //调用Shutdown.exit
          Shutdown.exit(var1.getNumber() + 128);
        }
      };
      handler = var0;
      try {
        //中断时
        Signal.handle(new Signal("INT"), var0);
      } catch (IllegalArgumentException var3) {

      }
      try {
        //终止时
        Signal.handle(new Signal("TERM"), var0);
      } catch (IllegalArgumentException var2) {

      }
    }
  }
}
```

**Runtime.addShutdownHook**。在了解`Shutdown.exit`之前，先看`Runtime.getRuntime().addShutdownHook(shutdownHook)`；则是为 jvm 中增加一个关闭的钩子，当 jvm`关闭`的时候调用。

```java
public class Runtime {
  public void addShutdownHook(Thread hook) {
    SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
      sm.checkPermission(new RuntimePermission("shutdownHooks"));
    }
    ApplicationShutdownHooks.add(hook);
  }
}
```

```java
class ApplicationShutdownHooks {
  /* 已注册钩子的集合 */
  private static IdentityHashMap<Thread, Thread> hooks;

  static {
    try {
      Shutdown.add(1 /* 关闭钩子调用顺序 */,
                   false /* 如果正在关闭，则不注册 */,
                   new Runnable() {
                     public void run() {
                       runHooks();
                     }
                   }
                  );
      hooks = new IdentityHashMap<>();
    } catch (IllegalStateException e) {
      // 如果正在进行关闭，则不能添加应用程序关闭钩子
      hooks = null;
    }
  }

  static synchronized void add(Thread hook) {
    if(hooks == null)
      throw new IllegalStateException("Shutdown in progress");

    if (hook.isAlive())
      throw new IllegalArgumentException("Hook already running");

    if (hooks.containsKey(hook))
      throw new IllegalArgumentException("Hook previously registered");

    hooks.put(hook, hook);
  }
}
```

```java
//它含数据结构和逻辑管理虚拟机关闭序列
class Shutdown {
  /* Shutdown 系列状态*/
  private static final int RUNNING = 0;
  private static final int HOOKS = 1;
  private static final int FINALIZERS = 2;
  private static int state = RUNNING;
  /* 是否应该运行所以finalizers来exit? */
  private static boolean runFinalizersOnExit = false;
  // 系统关闭钩子注册一个预定义的插槽.
  // 关闭钩子的列表如下:
  // (0) Console restore hook
  // (1) Application hooks
  // (2) DeleteOnExit hook
  private static final int MAX_SYSTEM_HOOKS = 10;
  private static final Runnable[] hooks = new Runnable[MAX_SYSTEM_HOOKS];
  // 当前运行关闭钩子的钩子的索引
  private static int currentRunningHook = 0;
  /* 前面的静态字段由这个锁保护 */
  private static class Lock { };
  private static Object lock = new Lock();

  /* 为native halt方法提供锁对象 */
  private static Object haltLock = new Lock();

  static void add(int slot, boolean registerShutdownInProgress, Runnable hook) {
    synchronized (lock) {
      if (hooks[slot] != null)
        throw new InternalError("Shutdown hook at slot " + slot + " already registered");

      if (!registerShutdownInProgress) {//执行shutdown过程中不添加hook
        if (state > RUNNING)//如果已经在执行shutdown操作不能添加hook
          throw new IllegalStateException("Shutdown in progress");
      } else {//如果hooks已经执行完毕不能再添加hook。如果正在执行hooks时，添加的槽点小于当前执行的槽点位置也不能添加
        if (state > HOOKS || (state == HOOKS && slot <= currentRunningHook))
          throw new IllegalStateException("Shutdown in progress");
      }

      hooks[slot] = hook;
    }
  }
  /* 执行所有注册的hooks
     */
  private static void runHooks() {
    for (int i=0; i < MAX_SYSTEM_HOOKS; i++) {
      try {
        Runnable hook;
        synchronized (lock) {
          // 获取锁以确保在关闭期间注册的钩子在这里可见。
          currentRunningHook = i;
          hook = hooks[i];
        }
        if (hook != null) hook.run();
      } catch(Throwable t) {
        if (t instanceof ThreadDeath) {
          ThreadDeath td = (ThreadDeath)t;
          throw td;
        }
      }
    }
  }
  /* 关闭JVM的操作
     */
  static void halt(int status) {
    synchronized (haltLock) {
      halt0(status);
    }
  }
  //JNI方法
  static native void halt0(int status);
  // shutdown的执行顺序：runHooks > runFinalizersOnExit
  private static void sequence() {
    synchronized (lock) {
      if (state != HOOKS) return;
    }
    runHooks();
    boolean rfoe;
    synchronized (lock) {
      state = FINALIZERS;
      rfoe = runFinalizersOnExit;
    }
    if (rfoe) runAllFinalizers();
  }
  //Runtime.exit时执行，runHooks > runFinalizersOnExit > halt
  static void exit(int status) {
    boolean runMoreFinalizers = false;
    synchronized (lock) {
      if (status != 0) runFinalizersOnExit = false;
      switch (state) {
        case RUNNING:       /* Initiate shutdown */
          state = HOOKS;
          break;
        case HOOKS:         /* Stall and halt */
          break;
        case FINALIZERS:
          if (status != 0) {
            /* Halt immediately on nonzero status */
            halt(status);
          } else {
            /* Compatibility with old behavior:
                     * Run more finalizers and then halt
                     */
            runMoreFinalizers = runFinalizersOnExit;
          }
          break;
      }
    }
    if (runMoreFinalizers) {
      runAllFinalizers();
      halt(status);
    }
    synchronized (Shutdown.class) {
      sequence();
      halt(status);
    }
  }
  //shutdown操作，与exit不同的是不做halt操作(关闭JVM)
  static void shutdown() {
    synchronized (lock) {
      switch (state) {
        case RUNNING:       /* Initiate shutdown */
          state = HOOKS;
          break;
        case HOOKS:         /* Stall and then return */
        case FINALIZERS:
          break;
      }
    }
    synchronized (Shutdown.class) {
      sequence();
    }
  }
}
```

#### spring中如何实现优雅停机

以Spring5.2.9在spring中通过 `ContextClosedEvent` 事件来触发一些动作，主要通过LifecycleProcessor.onClose来做stopBeans。由此可见spring也基于jvm做了扩展。

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader {
  public void registerShutdownHook() {
    if (this.shutdownHook == null) {
      // No shutdown hook registered yet.
      this.shutdownHook = new Thread() {
        @Override
        public void run() {
          doClose();
        }
      };
      Runtime.getRuntime().addShutdownHook(this.shutdownHook);
    }
  }
  protected void doClose() {
    boolean actuallyClose;
    synchronized (this.activeMonitor) {
      actuallyClose = this.active && !this.closed;
      this.closed = true;
    }

    if (actuallyClose) {
      if (logger.isInfoEnabled()) {
        logger.info("Closing " + this);
      }

      LiveBeansView.unregisterApplicationContext(this);

      try {
        //发布应用内的关闭事件
        publishEvent(new ContextClosedEvent(this));
      }
      catch (Throwable ex) {
        logger.warn("Exception thrown from ApplicationListener handling ContextClosedEvent", ex);
      }

      // 停止所有的Lifecycle beans.
      try {
        getLifecycleProcessor().onClose();
      }
      catch (Throwable ex) {
        logger.warn("Exception thrown from LifecycleProcessor on context close", ex);
      }

      // 销毁spring 的 BeanFactory可能会缓存单例的 Bean.
      destroyBeans();

      // 关闭当前应用上下文（BeanFactory）
      closeBeanFactory();

      // 执行子类的关闭逻辑
      onClose();

      synchronized (this.activeMonitor) {
        this.active = false;
      }
    }
  } 
}
```

```java
public interface LifecycleProcessor extends Lifecycle {
  /**
  * 上下文刷新通知，例如用于自动启动组件。
  */
  void onRefresh();

  /**
  * 上下文关闭阶段的通知，例如用于自动停止组件。
  */
  void onClose();
}
```

#### tomcat和spring的关系

-  `Tomcat`和`jettey`是 **HTTP 服务器**和 **Servlet 容器**，负责给类似 Spring 这种 servlet 提供一个运行的环境，其中：Http 服务器与 Servlet 容器的功能界限是：可以把 HTTP 服务器想象成`前台`的接待，**负责网络通信和解析请求**，Servlet 容器是`业务`部门，**负责处理业务请求**。
- Tomcat 和 Servlet 作为 **Web 服务器**和 **Servlet 容器**的结合，可以接受**网络 http 请求解析**为 Servlet 规范的请求对象和响应对象。比如，**HttpServletRequest** 对象是 Tomcat 提供的，Servlet 是规范，**Tomcat** 是实现规范的 Servlet 容器，**SpringMVC** 是处理 Servlet 请求的应用，其中 DispatcherServlet 实现了 Servlet 接口，Tomcat 负责加载和调用 DispatcherServlet。同时，DispatcherServlet 有自己的容器（SpringMVC）容器，这个容器负责管理 SpringMVC 相关的 bean，比如 Controler 和 ViewResolver 等。同时，Spring 中还有其他的 Bean 比如 Service 和 DAO 等，这些由全局的 Spring IOC 容器管理，因此，Spring 有两个 IOC 容器。
- 如果只是使用 spring (不包含 springmvc)，那么是 tomcat 容器解析 xml 文件，通过反射实例化对应的类，根据这些 servlet 规范实现类，触发对应的代码处理逻辑，这个时候 tomcat 负责 http 报文的解析和 servlet 调度的工作。
- 如果使用 spring mvc，那么 tomcat 只是解析 http 报文，然后将其转发给 dispatchsetvlet，然后由 springmvc 根据其配置，实例对应的类，执行对应的逻辑，然后返回结果给 dispatchservlet，最后由它转发给 tomcat, 由 tomcat 负责构建 http 报文数据。

### 其他资源的正确关闭和释放

---

在Spring Boot应用优雅关闭过程中，确保**数据库连接**、**线程池和其他网络IO资源**能够被正确关闭和释放是非常重要的。以下是一些建议和实践步骤：

1. 数据库连接：
   - Spring Boot集成了HikariCP、Tomcat JDBC Pool等数据源连接池，在应用上下文关闭时，如果正确配置，这些连接池通常会自动关闭并回收所有连接。确保在你的配置中没有禁用这些行为。
   - 可以在`DataSource` bean的生命周期回调方法中添加额外的关闭逻辑，例如实现`DisposableBean`接口并在`destroy()`方法中关闭或清空连接池。
2. 线程池：
   - Spring框架中的`ThreadPoolTaskExecutor`等线程池组件在应用上下文关闭时也会尝试停止和清理线程池。但为了确保长时间运行的任务有机会完成，你可以调整线程池的配置，使其在接收到关闭信号时拒绝新的任务，并等待现有任务结束。
   - 实现`SmartLifecycle`接口，并设置`stop(Runnable callback)`方法，以便在应用关闭时得到通知，并在此处采取相应措施（如取消线程池中的任务）。
3. 网络IO资源：
   - 对于非阻塞I/O，比如NIO通道和Selector，应在适当的资源管理类中监听应用关闭事件，及时关闭和释放这些资源。
   - 对于Socket通信，确保在接收到关闭信号后，关闭所有的客户端或服务器Socket连接，避免资源泄露。
4. 使用Spring Boot Actuator：
   - 利用`ApplicationListener`监听`ContextClosedEvent`事件，当应用上下文关闭时执行资源清理逻辑。
   - 若使用了Spring Boot Actuator，其生命周期管理会帮助协调很多资源的关闭。
5. 配置资源的关闭顺序：
   - 使用`@PreDestroy`注解在服务类的方法上，用于指定在Spring Bean销毁前执行的清理操作。

示例代码片段可能如下：

```java
@Configuration
public class ResourceCleanupConfig {

  @Autowired
  private DataSource dataSource;

  @Autowired
  private ThreadPoolTaskExecutor executor;

  @PreDestroy
  public void onShutdown() {
    // 清理数据库连接池
    ((HikariDataSource) dataSource).close();

    // 停止并清理线程池
    executor.shutdown();
    try {
      if (!executor.awaitTermination(30, TimeUnit.SECONDS)) {
        executor.shutdownNow();
      }
    } catch (InterruptedException ex) {
      executor.shutdownNow();
      Thread.currentThread().interrupt();
    }

    // ... 进行其他资源清理操作
  }
}
```

总之，确保资源被正确关闭的关键在于遵循良好的编程实践和Spring框架提供的生命周期管理机制，以及根据特定资源类型的具体需求进行定制化处理。

#### mq资源释放

`mq`通过添加`hook`在停机时调用`pause`先停止该应用的消费，防止出现上线期间`mq`中线程池的`线程中断`的情况发生。

```java
@Component
@Slf4j
public class ShutDownHook {

  @Value("${shutdown.waitTime:10}")
  private int waitTime;

  @Resource
  com.jdjr.fmq.client.consumer.MessageConsumer fmqMessageConsumer;

  @Resource
  com.jd.jmq.client.consumer.MessageConsumer jmqMessageConsumer;

  @PreDestroy
  public void destroyHook() {
    try {
      log.info("ShutDownHook destroy");
      jmqMessageConsumer.pause();
      fmqMessageConsumer.pause();

      int i = 0;
      while (i < waitTime) {
        try {
          Thread.sleep(1000);
          log.info("距离服务关停还有{}秒", waitTime - i++);
        } catch (Throwable e) {
          log.error("异常", e);
        }
      }
    } catch (Throwable e) {
      log.error("异常", e);
    }
  }
}
```

在优雅停机时需要先把生产者下线，并预留一定时间消费完毕，行云部署有相关 stop.sh 脚本，项目中通过在 shutdown 中编写方法实现。

```java
@Component
@Lazy(value = false)
public class ShutDown implements ApplicationContextAware {
  private static Logger logger = LoggerFactory.getLogger(ShutDown.class);

  @Value("${shutdown.waitTime:60}")
  private int waitTime;

  @Resource
  com.jdjr.fmq.client.consumer.MessageConsumer fmqMessageConsumer;

  @PostConstruct
  public void init() {
    logger.info("ShutDownHook init");
  }

  private ApplicationContext applicationContext = null;

  @PreDestroy
  public void destroyHook() {
    try {
      logger.info("ShutDownHook destroy");
      destroyJsfProvider();
      fmqMessageConsumer.pause();

      int i = 0;
      while (i < waitTime) {
        try {
          Thread.sleep(1000);
          logger.info("距离服务关停还有{}秒", waitTime - i++);
        } catch (Throwable e) {
          logger.error("异常", e);
        }
      }

    } catch (Throwable e) {
      logger.error("异常", e);
    }

  }
  private void destroyJsfProvider() {
    logger.info("关闭所有JSF生产者");
    if (null != applicationContext) {
      String[] providerBeanNames = applicationContext.getBeanNamesForType(ProviderBean.class);
      for (String name : providerBeanNames) {
        try {
          logger.info("尝试关闭JSF生产者" + name);
          ProviderBean bean=(ProviderBean)applicationContext.getBean(name);
          bean.destroy();
          logger.info("关闭JSF生产者" + name + "成功");
        } catch (BeanCreationNotAllowedException re){
          logger.error("JSF生产者" + name + "未初始化，忽略");
        } catch (Exception e) {
          logger.error("关闭JSF生产者失败", e);
        }
      }
    }
    logger.info("所有JSF生产者已关闭");
  }

  @Override
  public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
    this.applicationContext = applicationContext;
    ((AbstractApplicationContext)applicationContext).registerShutdownHook();
  }

}
```





参考：https://www.cnblogs.com/kukuxjx/p/17578770.html