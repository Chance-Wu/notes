```
企业级spring-boot案例
|----Spring Boot整合actuator,实现服务监控管理 spring-boot-actuator
|----Spring Boot集成事件发布及监听 spring-boot-applicationevent
|----Spring Boot整合异步线程池 spring-boot-async
|----Spring Boot整合自定义banner spring-boot-banner
|----Spring Boot整合本地缓存caffeine spring-boot-cache-caffeine
|----Spring Boot整合验证码captcha spring-boot-captcha
|----Spring Boot整合cors跨域资源共享 spring-boot-cors
|----Spring Boot整合jpa实现增删改查 spring-boot-data-jpa
|----Spring Boot整合PostgreSQL spring-boot-data-postgresql
|----Spring Boot整合Redis spring-boot-data-redis
|----Spring Boot整合Docker spring-boot-docker
|----Spring Boot设计模式之工厂模式 spring-boot-dp-factory
|----Spring Boot整合Druid数据库连接池 spring-boot-druid
|----Spring Boot整合dubbo spring-boot-dubbo
|----Spring Boot整合EasyExcel,实现Excel导入导出 spring-boot-easyexcel
|----Spring Boot整合邮件发送 spring-boot-email
|----Spring Boot整合全局异常处理,接口统一响应对象 spring-boot-exception-handler
|----Spring Boot整合flyway,数据库版本控制 spring-boot-flyway
|----Spring Boot整合git插件,将项目打包为tar.gz,并带上git版本号,然后通过sh脚本快速部署 spring-boot-git-commit-id-plugin
|----spring-boot快速入门-HelloWorld spring-boot-helloworld
|----Spring Boot整合HikariCP数据库连接池 spring-boot-hikaricp
|----Spring Boot整合https spring-boot-https
|----Spring Boot整合Jackson,实现数据脱敏 spring-boot-json-desensitization
|----Spring Boot整合Kafka spring-boot-kafka
|----Spring Boot整合Knife4j-API接口文档 spring-boot-knife4j
|----Spring Boot整合log4j2日志 spring-boot-log4j2
|----Spring Boot整合logback日志 spring-boot-logback
|----Spring Boot整合MyBatis spring-boot-mybatis
|----Spring Boot整合MyBatis,使用注解方式 spring-boot-mybatis-annotation
|----Spring Boot整合MyBatis逆向工程 spring-boot-mybatis-generator
|----Spring Boot整合通用mapper spring-boot-mybatis-mapper
|----Spring Boot整合MyBatis多数据源 spring-boot-mybatis-multi-datasource
|----Spring Boot整合MyBatis,使用pagehelper进行分页 spring-boot-mybatis-pagehelper
|----Spring Boot整合MyBatis脱敏插件,实现手机号等信息脱敏 spring-boot-mybatis-plugin-sensitive
|----Spring Boot整合Mybatis-Plus spring-boot-mybatis-plus
|----Spring Boot整合MyBatis Plus代码生成器 spring-boot-mybatis-plus-generator
|----Spring Boot整合MyBatis Plus多数据源 spring-boot-mybatis-plus-multi-datasource
|----Spring Boot整合pf4j,进行插件式编程拓展 spring-boot-pf4j
|----Spring Boot加载配置文件 spring-boot-properties
|----Spring Boot整合接口限流-guava单体方式 spring-boot-ratelimit-guava
|----Spring Boot整合接口限流-redis集群方式 spring-boot-ratelimit-redis
|----Spring Boot整合RestTemplate,实现服务间调用 spring-boot-resttemplate
|----Spring Boot整合retrofit,支持通过接口的方式发起HTTP请求 spring-boot-retrofit
|----Spring Boot 启动时的运行方法 spring-boot-runner
|----Spring Boot整合定时任务scheduler spring-boot-scheduler
|----Spring Boot整合Screw,一键生成数据库文档 spring-boot-screw
|----Spring Boot整合Shiro spring-boot-shiro
|----Spring Boot整合Swagger3-API接口文档 spring-boot-swagger3
|----Spring Boot整合模板引擎Thymeleaf spring-boot-thymeleaf
|----Spring Boot整合undertow spring-boot-undertow
|----Spring Boot项目打包成war包 spring-boot-war
|----Spring Boot整合zip,压缩和解压文件 spring-boot-zip
```

#### 为什么需要在spring boot启动时运行代码

出于多种原因，我们需要在应用程序启动时运行方法，例如：

- 记录重要的事情或消息说应用程序已启动
- 处理数据库或文件、索引、创建缓存等。
- 启动后台进程，如发送通知、从某个队列中获取数据等。

#### 1. 使用CommandLineRunner接口

---

`org.springframework.boot.CommandLineRunner` 是Spring Boot提供的一个接口，实现该接口并将之注入Spring IoC容器后，应用启动时就会执行其`run`方法。

实现 CommandLineRunner 接口的示例：

```java
@Slf4j
@Component
public class MyCommandLineRunner implements CommandLineRunner {

  @Override
  public void run(String... args) throws Exception {
    log.info("启动预加载数据(MyCommandLineRunner)...{}", Arrays.toString(args));
  }
}
```

创建CommandLineRunner接口的bean示例：

```java
@Slf4j
@SpringBootApplication
public class SpringBootRunnerApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringBootRunnerApplication.class, args);
    log.info("应用启动...");
  }

  @Bean
  public CommandLineRunner CommandLineRunnerBean() {
    return (args) -> {
      log.info("启动预加载数据(In CommandLineRunnerImpl)...{}", Arrays.toString(args));
    };
  }
}
```

使用命令行或 IDE 运行应用程序。例如，使用“-status=running”作为参数运行应用程序时

```shell
mvn spring-boot:run -Dspring-boot.run.arguments="--spring.profile.active=test, --foo=bar, --foo=baz, --debug, runner"
```

或者

```shell
mvn package 
java -jar target/<FILENAME.JAR HERE> --spring.profile.active=test, --foo=bar, --foo=baz, --debug, runner
```

或者

```shell
java -jar spring-boot-runner-1.0.0.jar --spring.profile.active=test, --foo=bar, --foo=baz, --debug, runner
```

或者在 IDE 运行应用程序指定参数

![](https://tva1.sinaimg.cn/large/008i3skNgy1gx4ekhgy02j30tw0iy0u2.jpg)

这将产生以下日志输出：

```
2021-12-04 10:24:04.525  INFO 6964 --- [           main] c.j.s.boot.runner.MyCommandLineRunner    : 启动预加载数据(MyCommandLineRunner)...[--spring.profile.active=test, --foo=bar, --foo=baz, --debug, runner]
2021-12-04 10:24:04.526  INFO 6964 --- [           main] c.j.s.boot.SpringBootRunnerApplication   : 启动预加载数据(In CommandLineRunnerImpl)...[--spring.profile.active=test, --foo=bar, --foo=baz, --debug, runner]
2021-12-04 10:24:04.527  INFO 6964 --- [           main] c.j.s.boot.SpringBootRunnerApplication   : 应用启动...
```

正如我们所看到的，参数没有被解析，而是被解释为单个值"–spring.profile.active=test, --foo=bar, --foo=baz, --debug, runner"。

要以解析格式访问命令行参数，我们需要使用 ApplicationRunner 接口。

Spring Boot 在启动过程中添加了 CommandLineRunner 接口。因此在 commandlinerRunner 中抛出异常将强制 Spring boot 中止启动。

一个Spring Boot可以存在多个`CommandLineRunner`的实现，当存在多个时，你可以实现`Ordered`接口控制这些实现的执行顺序(**Order 数值越小优先级越高**)。

```java
@Component
@Order(1)
public class CommandLineRunnerImpl implements CommandLineRunner {
  ........
}
```

#### 2. 使用ApplicationRunner接口

---

要访问解析的参数，我们需要使用 `ApplicationRunner` 接口，在**Spring Boot 1.3.0**引入了一个和`CommandLineRunner`功能一样的接口。`CommandLineRunner`接收可变参数`String... args`，而`ApplicationRunner` 接收一个封装好的对象参数`ApplicationArguments`。除此之外它们功能完全一样，甚至连方法名都一样。

它提供了访问参数的不同方法，如下所示

| 方法                                | 说明                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| String[] GetSourceArgs()            | 被传递给应用程序的原始参数，返回这些参数的字符串数组         |
| Set getOptionNames()                | 获取选项名称的`Set`字符串集合，选项参数的前面都有"–"。如 `--spring.profiles.active=dev --debug` 将返回`["spring.profiles.active","debug"]` 。 |
| List getNonOptionArgs()             | 用来获取所有的无选项参数，不带"–"的参数                      |
| boolean containsOption(String name) | 用来判断是否包含某个选项的名称                               |
| List getOptionValues(String name)   | 通过名称来获取该名称对应的选项值，我们可以在命令行中多次使用相同的键。如`--foo=bar --foo=baz` 将返回`["bar","baz"]`。 |

```java
@Slf4j
@Order(1)
@Component
public class MyApplicationRunner implements ApplicationRunner {

  @Override
  public void run(ApplicationArguments args) throws Exception {
    log.info("MyApplicationRunner run方法");
    boolean b1 = args.containsOption("spring.profile.active");
    boolean b2 = args.containsOption("abc");
    log.info("containsOption[spring.profile.active]:{},containsOption[abc]:{}", b1, b2);

    log.info("SourceArgs:{}", Arrays.toString(args.getSourceArgs()));
    log.info("NonOptionArgs:{}", args.getNonOptionArgs());
    log.info("OptionNames:{}", args.getOptionNames());

    log.info("Printing key and value in loop:");
    for (String key : args.getOptionNames()) {
      log.info("key:{}", key);
      log.info("val:{}", args.getOptionValues(key));
    }
  }
}
```

使用"–spring.profile.active=test, --foo=bar, --foo=baz, --debug, runner"参数运行程序

```java
2021-12-04 11:02:53.584  INFO 15260 --- [           main] c.j.s.boot.runner.MyApplicationRunner    : MyApplicationRunner run方法
2021-12-04 11:02:53.584  INFO 15260 --- [           main] c.j.s.boot.runner.MyApplicationRunner    : containsOption[spring.profile.active]:true,containsOption[abc]:false
2021-12-04 11:02:53.586  INFO 15260 --- [           main] c.j.s.boot.runner.MyApplicationRunner    : SourceArgs:[--spring.profile.active=test, --foo=bar, --foo=baz, --debug, runner]
2021-12-04 11:02:53.586  INFO 15260 --- [           main] c.j.s.boot.runner.MyApplicationRunner    : NonOptionArgs:[runner]
2021-12-04 11:02:53.586  INFO 15260 --- [           main] c.j.s.boot.runner.MyApplicationRunner    : OptionNames:[spring.profile.active, debug, foo]
2021-12-04 11:02:53.586  INFO 15260 --- [           main] c.j.s.boot.runner.MyApplicationRunner    : Printing key and value in loop:
2021-12-04 11:02:53.586  INFO 15260 --- [           main] c.j.s.boot.runner.MyApplicationRunner    : key:spring.profile.active
2021-12-04 11:02:53.587  INFO 15260 --- [           main] c.j.s.boot.runner.MyApplicationRunner    : val:[test]
2021-12-04 11:02:53.587  INFO 15260 --- [           main] c.j.s.boot.runner.MyApplicationRunner    : key:debug
2021-12-04 11:02:53.587  INFO 15260 --- [           main] c.j.s.boot.runner.MyApplicationRunner    : val:[]
2021-12-04 11:02:53.587  INFO 15260 --- [           main] c.j.s.boot.runner.MyApplicationRunner    : key:foo
2021-12-04 11:02:53.587  INFO 15260 --- [           main] c.j.s.boot.runner.MyApplicationRunner    : val:[bar, baz]
2021-12-04 11:02:53.587  INFO 15260 --- [           main] c.j.s.boot.runner.MyCommandLineRunner    : MyCommandLineRunner run方法...[--spring.profile.active=test, --foo=bar, --foo=baz, --debug, runner]
2021-12-04 11:02:53.587  INFO 15260 --- [           main] c.j.s.boot.SpringBootRunnerApplication   : bean注解创建CommandLineRunner,参数[--spring.profile.active=test, --foo=bar, --foo=baz, --debug, runner]
2021-12-04 11:02:53.588  INFO 15260 --- [           main] c.j.s.boot.SpringBootRunnerApplication   : 应用启动...
```

CommandLineRunner 和 ApplicationRunner 具有类似的功能，例如

- run() 方法中的异常将中止应用程序启动
- 可以使用 Ordered 接口或 @Order 注解对多个 ApplicationRunner 进行排序

需要注意的最重要的一点是，CommandLineRunners 和 ApplicationRunners 之间共享 Order。这意味着 commandlinerRunner 和 applicationRunner 之间的执行顺序可能会混合。

`CommandLineRunner` 和 `ApplicationRunner` 区别

- 参数不一样。两个接口的实现方法一样，参数不一样，设置命令行参数：–spring.profile.active=test，ApplicatonRunner接口的方法参数ApplicationArguments（是个对象）比CommandLineRunner接口的方法参数（是个可以接收多个string的参数）功能更强大。ApplicatonRunner接口的方法参数ApplicationArguments既可以获取参数的字符串，也可以直接获取key；CommandLineRunner接口的方法参数只能获取参数的字符串。
- 方法执行顺序不一样。ApplicationRunner接口的实现方法比CommandLineRunner接口的实现方法前执行（当然也可以设置@Order的值来决定谁先执行）

执行顺序使用源码分析

说到执行顺序，那么再进一步了解一下这两个方法是在什么时候执行的。这两个接口的实现执行的时机在于SpringApplication初始化之后，调用的run方法中被调用的。

```java
public ConfigurableApplicationContext run(String... args) {
  // 创建 StopWatch 对象，用于统计 run 方法启动时长。
  StopWatch stopWatch = new StopWatch();
  // 启动统计。
  stopWatch.start();
  ConfigurableApplicationContext context = null;
  Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
  // 配置 headless 属性。
  configureHeadlessProperty();
  // 获得 SpringApplicationRunListener 数组，
  // 该数组封装于 SpringApplicationRunListeners 对象的 listeners 中。
  SpringApplicationRunListeners listeners = getRunListeners(args);
  // 启动监听，遍历 SpringApplicationRunListener 数组每个元素，并执行。
  listeners.starting();
  try {
    //创建 ApplicationArguments 对象
    ApplicationArguments applicationArguments = new DefaultApplicationArguments(
      args);
    // 加载属性配置，包括所有的配置属性（如：application.properties 中和外部的属性配置）
    ConfigurableEnvironment environment = prepareEnvironment(listeners,
                                                             applicationArguments);
    configureIgnoreBeanInfo(environment);
    // 打印 Banner
    Banner printedBanner = printBanner(environment);
    // 创建容器
    context = createApplicationContext();

    // 异常报告器
    exceptionReporters = getSpringFactoriesInstances(
      SpringBootExceptionReporter.class,
      new Class[]{ConfigurableApplicationContext.class}, context);
    // 准备容器，组件对象之间进行关联
    prepareContext(context, environment, listeners, applicationArguments,
                   printedBanner);
    // 初始化容器
    refreshContext(context);
    // 初始化操作之后执行，默认实现为空。
    afterRefresh(context, applicationArguments);
    // 停止时长统计
    stopWatch.stop();
    // 打印启动日志
    if (this.logStartupInfo) {
      new StartupInfoLogger(this.mainApplicationClass)
        .logStarted(getApplicationLog(), stopWatch);
    }
    // 通知监听器：容器启动完成。
    listeners.started(context);
    // 调用 ApplicationRunner 和 CommandLineRunner 的运行方法。
    callRunners(context, applicationArguments);
  } catch (Throwable ex) {
    // 异常处理
    handleRunFailure(context, ex, exceptionReporters, listeners);
    throw new IllegalStateException(ex);
  }

  try {
    // 通知监听器：容器正在运行。
    listeners.running(context);
  } catch (Throwable ex) {
    // 异常处理
    handleRunFailure(context, ex, exceptionReporters, null);
    throw new IllegalStateException(ex);
  }
  return context;
}
```

在try方法的最后，会执行一个callRunners的方法，在此方法中会对实现这两个接口的实现类进行调用。

```java
private void callRunners(ApplicationContext context, ApplicationArguments args) {
  List<Object> runners = new ArrayList<>();
  runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
  runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
  AnnotationAwareOrderComparator.sort(runners);
  for (Object runner : new LinkedHashSet<>(runners)) {
    if (runner instanceof ApplicationRunner) {
      callRunner((ApplicationRunner) runner, args);
    }
    if (runner instanceof CommandLineRunner) {
      callRunner((CommandLineRunner) runner, args);
    }
  }
}

private void callRunner(ApplicationRunner runner, ApplicationArguments args) {
  try {
    (runner).run(args);
  } catch (Exception ex) {
    throw new IllegalStateException("Failed to execute ApplicationRunner", ex);
  }
}

private void callRunner(CommandLineRunner runner, ApplicationArguments args) {
  try {
    (runner).run(args.getSourceArgs());
  } catch (Exception ex) {
    throw new IllegalStateException("Failed to execute CommandLineRunner", ex);
  }
}
```

通过以上代码，我们也就了解到这两个接口的实现类的执行时机了。

#### 3. Spring Boot中的应用事件

---

Spring 框架在不同的情况下会触发不同的事件。它还在启动过程中触发了许多事件。我们可以使用这些事件来执行我们的代码，例如 ApplicationReadyEvent 可以用于在 Spring Boot 应用程序启动后执行代码。

如果我们不需要命令行参数，这是在应用程序启动后执行代码的最佳方式。

```java
@Slf4j
@Component
public class RunAfterStartup {

  @EventListener(ApplicationReadyEvent.class)
  public void runAfterStartup() {
    log.info("Spring Boot中的应用事件,Yaaah, I am running........");
  }
}
```

输出：

```
2021-12-04 11:08:23.434  INFO 7824 --- [           main] c.j.spring.boot.event.RunAfterStartup    : Spring Boot中的应用事件,Yaaah, I am running........
```

Spring Boot 中一些最重要的事件

- **ApplicationContextInitializedEvent** ：在准备好 `ApplicationContext` 并调用`ApplicationContextInitializers` 之后但在加载 `bean` 定义之前触发
- **ApplicationPreparedEvent** ：加载 `bean` 定义后触发
- **ApplicationStartedEvent** ：在刷新上下文之后，但在调用命令行和应用程序运行程序之前触发
- **ApplicationReadyEvent** ：在调用应用程序和命令行运行程序后触发
- **ApplicationFailedEvent** : 如果启动时出现异常则触发

可以创建多个 `ApplicationListeners`。可以使用 `@Order` 注释或 `Ordered` 接口对它们进行排序。

该顺序与其他相同类型的 `ApplicationListeners` 共享，但不与 `ApplicationRunners` 或 `CommandLineRunners` 共享。

#### 4. 方法上的@Postconstruct注解

---

一个方法可以用@PostConstruct 注解来标记。每当一个方法被这个注解标记时，它会在依赖注入后立即被调用。

@PostConstruct 方法链接到特定的类，因此它应该只用于特定于类的代码。每个类只能有一个带有 postConstruct 注释的方法。

```java
@Slf4j
@Component
public class PostContructImpl {

  public PostContructImpl() {
    log.info("PostContructImpl构造器");
  }

  @PostConstruct
  public void runAfterObjectCreated() {
    log.info("PostContruct runAfterObjectCreated方法");
  }
}
```

输出：

```
2021-12-04 11:18:52.563  INFO 14928 --- [           main] c.j.s.b.postconstruct.PostContructImpl   : PostContructImpl构造器
2021-12-04 11:18:52.564  INFO 14928 --- [           main] c.j.s.b.postconstruct.PostContructImpl   : PostContruct runAfterObjectCreated方法
```

需要注意的是，如果类被标记为惰性，则意味着该类是在请求时创建的，之后将执行标有@postConstruct 注解的方法。

用 postConstruct 注解标记的方法可以有任何名称，但不能有任何参数。它必须是空的，不应该是静态的。

请注意，@postConstruct 注解是 Java EE 模块的一部分，它在 Java 9 中被标记为已弃用并在 Java 11 中删除。我们仍然可以通过将 java.se.ee 添加到应用程序中来使用它。

#### 5. InitializingBean接口

---

InitializingBean 的工作方式与 postConstruct 注释完全相似。我们必须实现 InitializingBean 接口，而不是使用注解。然后我们需要覆盖 void afterPropertiesSet() 方法。InitializingBean 是 org.springframework.beans.factory 包的一部分。

```java
@Slf4j
@Component
public class InitializingBeanImpl implements InitializingBean {

  public InitializingBeanImpl() {
    log.info("InitializingBeanImpl Constructor构造器");
  }

  @Override
  public void afterPropertiesSet() throws Exception {
    log.info("InitializingBeanImpl afterPropertiesSet方法");
  }
}
```

输出：

```
2021-12-04 11:23:15.843  INFO 8012 --- [           main] c.j.s.b.i.InitializingBeanImpl           : InitializingBeanImpl Constructor构造器
2021-12-04 11:23:15.843  INFO 8012 --- [           main] c.j.s.b.i.InitializingBeanImpl           : InitializingBeanImpl afterPropertiesSet方法
```

如果我们同时使用 @PostConstruct 注释和 InitializingBean 会发生什么。那么在这种情况下，@PostConstruct 方法将在 InitializingBean 的 afterPropertiesSet() 方法之前被调用。

>**@PostConstruct` 和 `afterPropertiesSet` 区别**
>
>1. afterPropertiesSet，顾名思义「在属性设置之后」，调用该方法时，该 bean 的所有属性已经被 Spring 填充。如果我们在某些属性上使用 `@Autowired`（常规操作应该使用构造函数注入），那么 Spring 将在调用`afterPropertiesSet` 之前将 bean 注入这些属性。但 `@PostConstruct` 并没有这些属性填充限制
>2. 所以 `InitializingBean.afterPropertiesSet` 解决方案比使用 `@PostConstruct` 更安全，因为如果我们依赖尚未自动注入的 `@Autowired` 字段，则 `@PostConstruct` 方法可能会遇到 NullPointerExceptions

#### 6. @Bean注解的init属性

---

使用@Bean 注释中的 initMethod 属性提供一个方法。这个方法会在 bean 初始化后调用。

initMethod 中提供的方法必须是 void 并且不应该有任何参数。这种方法甚至可以是私有的。

```java
@Slf4j
public class BeanInitMethodImpl {

  public void runAfterObjectCreated() {
    log.info("BeanInitMethodImpl runAfterObjectCreated方法,yooooooooo......... someone called me");
  }
}

@Slf4j
@SpringBootApplication
public class SpringBootRunnerApplication {

  public static void main(String[] args) {
    SpringApplication.run(SpringBootRunnerApplication.class, args);
    log.info("应用启动...");
  }

  @Bean(initMethod = "runAfterObjectCreated")
  public BeanInitMethodImpl getFunnyBean() {
    return new BeanInitMethodImpl();
  }
}
```

输出：

```
2021-12-04 11:26:09.865  INFO 2884 --- [           main] c.j.s.b.b.BeanInitMethodImpl             : BeanInitMethodImpl runAfterObjectCreated方法,yooooooooo......... someone called me
```

如果同一个类有InitializingBean 实现和@Bean 注解的initMethod 属性，则InitializingBean 的afterPropertiesSet 方法会在initMethod 之前被调用。

#### 总结

---

有时可能需要组合多个选项。然后他们将按以下顺序执行，

- Constructor
- PostContruct method
- afterPropertiesSet method
- Bean init Method
- ApplicationStartedEvent
- ApplicationRunner Or CommandLineRunner depends on Order
- ApplicationReadyEvent