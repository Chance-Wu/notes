参考文章：[Java日志框架那些事儿](https://www.cnblogs.com/chanshuyi/p/something_about_java_log_framework.html#slf4jjdklog)、[logback官网](http://logback.qos.ch/)

>- LogBack 自身实现了 SLF4J 的日志接口，不需要 SLF4J 去做进一步的适配。
>- LogBack 自身是在 Log4J 的基础上优化而成的，其运行速度和效率都比 LOG4J 高。
>- SLF4J + LogBack 支持占位符，方便日志代码的阅读，而 LOG4J 则不支持。
>
>从上面几点来看，**SLF4J + LogBack**是一个较好的选择。
>
>LogBack 被分为3个组件：*logback-core、logback-classic 和 logback-access。*
>
>- **logback-core** 提供了 LogBack 的核心功能，是另外两个组件的基础。
>- **logback-classic** 则实现了 SLF4J 的API，所以当想配合 SLF4J 使用时，需要将 logback-classic 引入依赖中。
>- **logback-access** 是为了集成Servlet环境而准备的，可提供HTTP-access的日志接口。
>
>LogBack的日志记录数据流是从 Class（Package）到 Logger，再从Logger到Appender，最后从Appender到具体的输出终端。
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gn51ddb0noj31060myq41.jpg" style="zoom:60%">

#### 1. logback架构

>Logback 其实可以说是 Log4J 的进化版。==LogBack 除了具备 Log4j 的所有优点之外，还解决了 Log4J 不能使用占位符的问题==。springboot默认使用的是logbcak日志。

>目前，logback分为三个模块：
>
>- logback-core（基础）
>- logback-classic：该模块==本质上实现了SLF4J API==，因此可以轻松地在logback和其他日志框架(如log4j或java.util.logging)之间来回切换。
>- logback-access：该模块与Servlet容器集成，以提供HTTP访问日志功能。

>Logback建立在三个主要的类上：`Logger`，`Appender`和`Layout`。
>
>Logger类是logback-classic模块的一部分。Appender和Layout接口是logback-core的一部分。作为通用模块，logback-core没有记录器的概念。

##### 1.1 Logger context

>==每个单独的记录器(logger)都附加到LoggerContext==，后者负责制造记录器并将它们按树状排列。
>
>记录器是命名的实体。它们的名字区分大小写，并遵循分层命名规则。例如，名为“ com.foo”的记录器是名为“ com.foo.Bar”的记录器的父级。 同样，“ java”是“ java.util”的父代，也是“ java.util.Vector”的祖先。
>
>`root logger`位于日志记录器层次结构的顶部。它的例外之处在于，它是每一个层次结构的一部分。像每个日志程序一样，它可以通过它的名字来检索，如下所示：
>
>```java
>Logger rootLogger = LoggerFactory.getLogger(org.slf4j.Logger.ROOT_LOGGER_NAME);
>```

##### 1.2 有效级别

>可以为logger分配级别。ch.qos.logback.classic.Level类中定义了以下级别：
>
>- TRACE
>- DEBUG
>- INFO
>- WARN
>- ERROR
>
>如果没有为给定的记录器分配一个级别，那么它将从其最接近的祖先那里继承一个已分配的级别。为了确保所有记录器最终都可以继承级别，根记录器始终具有分配的级别。 默认情况下，此级别是DEBUG。

##### 1.3 打印方法和基本选择规则

>打印方法确定记录请求的级别。 如果logger是记录器实例，则语句logger.info（“ ..”）是INFO级别的记录语句。
>
>如果日志记录请求的级别高于或等于记录器的有效级别，则认为该请求已启用。 否则，该请求被称为禁用。
>
>`TRACE < DEBUG < INFO <  WARN < ERROR`
>
>|       | TRACE   | DEBUG   | INFO    | WARN    | ERROR   | OFF    |
>| ----- | ------- | ------- | ------- | ------- | ------- | ------ |
>| TRACE | **YES** | **NO**  | **NO**  | **NO**  | **NO**  | **NO** |
>| DEBUG | **YES** | **YES** | **NO**  | **NO**  | **NO**  | **NO** |
>| INFO  | **YES** | **YES** | **YES** | **NO**  | **NO**  | **NO** |
>| WARN  | **YES** | **YES** | **YES** | **YES** | **NO**  | **NO** |
>| ERROR | **YES** | **YES** | **YES** | **YES** | **YES** | **NO** |

##### 1.4 检索记录器

>以相同的名称调用LoggerFactory.getLogger方法将始终返回对完全相同的记录器对象的引用。
>
>```java
>Logger x = LoggerFactory.getLogger("wombat"); 
>Logger y = LoggerFactory.getLogger("wombat");
>```
>
>x和y是同一个记录器。

##### 1.5 追加器和布局

>Logback允许日志记录请求打印到多个目标。输出目标称为追加器。例如`ConsoleAppender`、`RollingFileAppender`。
>
>```xml
><property name="LOG_PATTERN"
>          value="%d{yyyy-MM-dd HH:mm:ss.SSS} %-4relative [%thread] %-5level %logger{35} - %msg %n"/>
><property scope="context" name="APP_NAME" value="app"/>
><property scope="context" name="LOG_HOME" value="/Users/chance/Downloads/log"/>
>
><appender name="Console" class="ch.qos.logback.core.ConsoleAppender">
>    <encoder>
>        <pattern>${LOG_PATTERN}</pattern>
>    </encoder>
></appender>
>
><appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
>    <file>
>        ${LOG_HOME}/${APP_NAME}.log
>    </file>
>    <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
>        <!--日志文件输出的文件名-->
>        <FileNamePattern>
>            ${LOG_HOME}/${APP_NAME}/${APP_NAME}.%i.log.%d{yyyy-MM-dd}.zip
>        </FileNamePattern>
>        <maxFileSize>50MB</maxFileSize>
>        <!--日志文件保留天数-->
>        <maxHistory>15</maxHistory>
>        <totalSizeCap>10GB</totalSizeCap>
>    </rollingPolicy>
>    <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
>        <pattern>${LOG_PATTERN}</pattern>
>    </encoder>
></appender>
>```
>
>通常，用户不仅希望自定义输出目标，还希望自定义输出格式。通过将布局与附加程序相关联来实现的。布局负责根据用户的需求格式化日志记录请求，而附加程序负责将格式化后的输出发送到其目的地。`PatternLayout`是标准logback发行版的一部分，它使用户可以根据类似于C语言printf函数的转换模式来指定输出格式。
>
>例如，带有转换模式`"%d{yyyy-MM-dd HH:mm:ss.SSS} ％-4relative [％thread]％-5level ％logger{32} - ％msg ％n"`的PatternLayout将输出类似于：
>
>```
>2021-01-29 15:17:27.444 172 [main] DEBUG manual.architecture.HelloWorld2 - Hello world.
>```
>
>- d{yyyy-MM-dd HH:mm:ss.SSS}时间
>
>- ％-4relative自程序启动以来经过的毫秒数
>
>- [％thread]发出日志请求的线程
>- ％-5level日志请求的级别
>- ％logger{32}记录器的名称
>- ％msg文本请求的信息
>- ％n换行

##### 1.6 参数化日志

>鉴于logback-classic的记录器实现了SLF4J的Logger接口，某些打印方法==允许使用多个参数==。这些打印方法变体主要旨在提高性能，同时最大程度地减少对代码可读性的影响。
>
>对于某些Logger记录器，如下
>
>```java
>logger.debug("Entry number: " + i + " is " + String.valueOf(entry[i]));
>```
>
>产生了构造消息参数（将整数i和entry [i]都转换为String并串联中间字符串）的开销。 
>
>（1）避免参数构造成本的一种可能方法是，==将log语句包含在测试中==。
>
>```java
>if(logger.isDebugEnabled()) {
>    logger.debug("Entry number: " + i + " is " + String.valueOf(entry[i]));
>}
>```
>
>这样，如果为记录器禁用了调试，则不会产生参数构造的成本。另一方面，如果为记录器启用了DEBUG级别，则将产生两次评估该记录器是否启用的成本：一次在debugEnabled中，一次在debug中。实际上，这种开销微不足道，因为评估记录器所花费的时间不到实际记录请求所花费的时间的1％。
>
>（2）避免参数构造成本更好的选择
>
>存在一种基于消息格式的便捷替代方法。假设条目是一个对象，可以编写：
>
>```java
>Object entry = new SomeObject(); 
>logger.debug("The entry is {}.", entry);
>```
>
>只有在评估是否要记录，并且决策是肯定的情况下，记录器实现才会==格式化消息并将'{}'对替换为entry的字符串值==。换句话说，当禁用log语句时，这种形式不会产生构造参数的成本。
>
>下面两行将产生完全相同的输出。但是，在禁用日志记录语句的情况下，第二种变体的性能将比第一种变体至少高出30倍。
>
>```java
>logger.debug("The new entry is "+entry+".");
>logger.debug("The new entry is {}.", entry);
>```
>
>两个参数的变体：
>
>```java
>logger.debug("The new entry is {}. It replaces {}.", entry, oldEntry);
>```

##### 1.7 logback运行步骤

>分析当用户调用名为`com.wombat`的记录器的info()方法时，logback采取的步骤。

>**（1）获取过滤器链决策**
>
>如果存在，则调用TurboFilter链。 Turbo过滤器可以设置上下文范围的阈值，或根据与每个日志记录请求关联的信息（例如标记，级别，记录器，消息或Throwable）过滤掉某些事件。如果过滤器链的答复是FilterReply.DENY，则将删除日志记录请求。如果是FilterReply.NEUTRAL，则继续下一步，即步骤2。如果答复是FilterReply.ACCEPT，则跳过下一步，直接跳至步骤3。

>**（2）应用基本选择规则**
>
>在此步骤中，logback==将记录器的有效级别与请求级别进行比较==。如果根据该测试禁用了日志记录请求，则logback将删除该请求，而无需进一步处理。否则，将继续进行下一步。

>**（3）创建一个LoggingEvent对象**
>
>如果请求在以前的过滤器中仍然存在，则logback将创建一个`ch.qos.logback.classic.LoggingEvent`对象，其中包含请求的所有相关参数，例如请求的记录器，请求级别，消息本身，可能已经与请求，当前时间，当前线程以及有关发出日志记录请求的类的各种数据以及MDC一起传递了。请注意，其中某些字段仅在实际需要时才延迟进行初始化。MDC用于用其他上下文信息修饰日志记录请求。

>**（4）调用appenders**
>
>创建LoggingEvent对象后，logback将调用所有适用附加程序的doAppend()方法，即从记录器上下文继承的附加程序。
>
>返回分发附带的所有附加程序都扩展了AppenderBase抽象类，该类在确保线程安全的同步块中实现了doAppend方法。如果存在任何此类过滤器，则AppenderBase的doAppend()方法也会调用附加到附加程序的自定义过滤器。可以动态附加到任何附加程序的自定义过滤器将在单独的章节中介绍。

>**（5）格式化输出**
>
>被调用的附加程序负责格式化日志记录事件。 但是，一些（但不是全部）附加程序将格式化日志记录事件的任务委托给布局。 布局会格式化LoggingEvent实例，并以String形式返回结果。 请注意，某些附加程序（例如SocketAppender）不会将日志记录事件转换为字符串，而是将其序列化。 因此，它们没有布局，也不需要布局。

>**（6）发送LoggingEvent**
>
>日志记录事件完全格式化后，每个附加程序会将其发送到其目的地。
>

#### 2. logback配置

>logback尝试进行自我配置的初始化步骤：
>
>1. logback尝试在类路径中找一个名为`logback-test.xml`的文件。
>2. 如果找不到，则尝试在在类路径下找一个名为`logback.groovy`的文件。
>3. 如果找不到，则在类路径下检查`logback.xml`文件。
>4. 如果还找不到，则查找文件META-INF\services\ch.qos.logback.classic.spi.Configurator，使用服务提供程序加载工具(JDK1.6中引入`java.util.ServiceLoader`)来解析com.qos.logback.classic.spi.Configurator接口的实现。其内容应指定所需的Configurator实现的完全限定的类名。
>5. 如果以上方法都不成功，则logback会==使用BasicConfigurator自动进行自动配置==，这会将日志输出定向到控制台。
>
>最后一步是在没有配置文件的情况下尽力提供默认的日志记录功能。
>
>Joran解析给定的logback配置文件大约需要100毫秒。 要减少应用程序启动时的时间，可以使用服务提供者加载工具（上面的步骤4）来用BasicConfigurator加载自定义的Configurator类。

##### 2.1 自动配置logback

>配置logback的最简单方法是让logback回退到默认配置，让我们来看看如何在一个名为MyApp1的虚构应用程序中完成此操作。
>
>示例：BasicConfigurator用法
>
>```java
>public class MyApp1 {
>
>    final static Logger logger = LoggerFactory.getLogger(MyApp1.class);
>
>    public static void main(String[] args) {
>        logger.info("Entering application.");
>
>        Foo foo = new Foo();
>        foo.doIt();
>        logger.info("Exiting application.");
>    }
>}
>```
>
>此类定义了静态记录器变量。然后，实例化Foo对象。
>
>```java
>public class Foo {
>
>    static final Logger logger = LoggerFactory.getLogger(Foo.class);
>
>    public void doIt() {
>        logger.debug("Did it again!");
>    }
>}
>```
>
>假设不存在配置文件logback-test.xml或logback.xml，则logback将==默认调用BasicConfigurator==，这将设置一个最低配置。 此最小配置由附加到根记录器的==ConsoleAppender==组成。 使用设置为模式`"％d {HH：mm：ss.SSS} [％thread]％-5level％logger {36}-％msg％n"`的PatternLayoutEncoder格式化输出。 此外，默认情况下，为根记录器分配了==DEBUG级别==。
>
>以上输出为：
>
>```
>16:52:29.499 [main] INFO com.chance.logger.logback.MyApp1 - Entering application.
>16:52:29.506 [main] DEBUG com.chance.logger.entity.Foo - Did it again!
>16:52:29.507 [main] INFO com.chance.logger.logback.MyApp1 - Exiting application.
>```

##### 2.2 使用logback-test.xml或logback.xml自动配置

><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gn4o70twvhj30f8070gli.jpg" style="zoom:60%">
>
>如果在类路径上找到`logback-test.xml`或`logback.xml`文件，则logback会尝试自行配置。这是一个等效于我们刚刚看到的BasicConfigurator建立的配置文件。
>
>示例：基本配置文件
>
>```xml
><?xml version="1.0" encoding="UTF-8" ?>
><configuration>
>
>    <property name="LOG_PATTERN" value="%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"/>
>
>    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
>        <!-- encoders are assigned the type
>             ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
>        <encoder>
>            <pattern>${LOG_PATTERN}</pattern>
>        </encoder>
>    </appender>
>
>    <root level="debug">
>        <appender-ref ref="STDOUT"/>
>    </root>
></configuration>
>```
>
>使用上述配置文件的效果与调研`BasicConfigurator`相同。

##### 2.3 出现警告或错误时自动打印状态消息

>在没有警告或错误的情况下，如果仍然希望检查logback的内部状态，则可以通过调用==StatusPrinter类==的`print()`来指示logback打印状态数据。
>
>```java
>public static void main(String[] args) {
>    // 假设 SLF4J 被绑定到当前环境的logback中
>    LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
>    // 打印 logback 的内部状态
>    StatusPrinter.print(lc);
>}
>```
>
>可以在控制台看到以下输出：
>
>```
>20:48:42,040 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Found resource [logback-test.xml] at [file:/Users/chance/IdeaProjects/logger-learn/target/classes/logback-test.xml]
>20:48:42,101 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - debug attribute not set
>20:48:42,108 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - About to instantiate appender of type [ch.qos.logback.core.ConsoleAppender]
>20:48:42,110 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - Naming appender as [STDOUT]
>20:48:42,116 |-INFO in ch.qos.logback.core.joran.action.NestedComplexPropertyIA - Assuming default type [ch.qos.logback.classic.encoder.PatternLayoutEncoder] for [encoder] property
>20:48:42,158 |-INFO in ch.qos.logback.classic.joran.action.RootLoggerAction - Setting level of ROOT logger to DEBUG
>20:48:42,158 |-INFO in ch.qos.logback.core.joran.action.AppenderRefAction - Attaching appender named [STDOUT] to Logger[ROOT]
>20:48:42,158 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - End of configuration.
>20:48:42,160 |-INFO in ch.qos.logback.classic.joran.JoranConfigurator@1ff8b8f - Registering current configuration as safe fallback point
>```

##### 2.4 状态数据(建议开启)

>即使没有错误，也可以知识配置文件转储状态数据。为此，需要设置configuration元素的debug属性。（注意，debug属性只与状态数据有关。）
>
>```xml
><configuration debug="debug">
>
>    <property name="LOG_PATTERN" value="%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"/>
>
>    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
>        <!-- encoders are assigned the type
>             ch.qos.logback.classic.encoder.PatternLayoutEncoder by default -->
>        <encoder>
>            <pattern>${LOG_PATTERN}</pattern>
>        </encoder>
>    </appender>
>
>    <root level="debug">
>        <appender-ref ref="STDOUT"/>
>    </root>
></configuration>
>```
>
>设置==debug="true"==等于安装一个`OnConsoleStatusListener`。下面显示`OnConsoleStatusListener`的安装。
>
>```xml
><statusListener class="ch.qos.logback.core.status.OnConsoleStatusListener"/>
>```

##### 2.5 将默认配置文件的位置指定为系统属性

>java -Dlogback.configurationFile=/path/to/config.xml ...
>
>springboot中指定配置文件位置：`logging.config=classpath:log/logback.xml`

##### 2.6 修改后自动重新加载配置文件

>logback-classic可以扫描器配置文件中的更改，并在配置文件更改时自动重新配置自身。
>
>设置configuration元素的属性==scan="true"==。
>
>默认情况下，配置文件将每分钟扫描一次更改。您可以通过设置元素的*scanPeriod*属性 来指定不同的扫描周期。单位可以是毫秒，秒，分钟或小时。
>
>```xml
><configuration scan = “true” scanPeriod = “30 seconds” >   
>
></configuration>
>```

#### 3. 配置文件语法

>logback配置文件的语法非常灵活。因此，不可能用DTD文件或XML模式指定允许的语法。但是，配置文件的最基本结构可以描述为元素。
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gn4yg5riehj308903zmx1.jpg" style="100%">

##### 3.1 标签名称的大小写敏感性

>- 对于显式规则，标签名时不区分大小写的。例如<logger>和<Logger>和<LOGGER>
>- 隐式规则，除第一个字母外，标签名称区分大小写。隐式规则通常遵循驼峰式命名规则。

##### 3.2 配置`<logger>`

>记录器使用该`<logger>`元素进行配置。一个`<logger>`元素只需要：
>
>- ==一个强制name属性==，
>- ==一个可选的level属性==
>- ==一个可选的additivity属性==，允许值为true或false。
>
>level属性的值允许不区分大小写的字符串值TRACE，DEBUG，INFO，WARN，ERROR，ALL或OFF之一。不区分大小写的特殊值*INHERITED*，或其同义词*NULL*，将强制logger的层次从层次结构中的较高层继承。如果设置logger的级别并稍后决定它应该继承它的级别，这将派上用场。
>
>与`<logger>`元素相似，==`<root>`元素可以包含零个或多个`<appender-ref>`元素==； 如此引用的每个附加程序都会添加到根记录器中。请注意，与log4j不同，在配置根记录程序时，logback-classic不会关闭也不删除任何以前引用的附加程序。

>实例：设置logger或根logger的level。
>
>```xml
><configuration>
>
>    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
>        <!-- encoders 声明了默认类型ch.qos.logback.classic.encoder.PatternLayoutEncoder -->
>        <encoder>
>            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
>        </encoder>
>    </appender>
>
>    <logger name="chapters.configuration" level="INFO"/>
>
>    <!-- level属性不是必须的，默认情况下根logger设置为debug -->
>    <root level="DEBUG">
>        <appender-ref ref="STDOUT" />
>    </root> 
>
></configuration>
>```
>
>[基本选择规则](https://logback.qos.ch/manual/architecture.html#basic_selection)取决于==被调用的logger的有效等级==，而不是附加器附加的logger的等级。Logback将首先确定是否启用日志记录语句，如果启用，则将调用logger层次结构中的appender，而不管其级别如何。

##### 3.3 配置`<appender>`

>appender元素需要两个强制属性：
>
>- name：声明appender的名称。
>- class：指定appender类实例化的全限定名。
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gn4yg5riehj308903zmx1.jpg" style="zoom:100%">
>
>Appender语法
>
>该<layout>元素采用强制性的类属性来指定要实例化的布局类的完全限定名。与<appender>元素一样， <layout>可以包含与布局实例的属性相对应的其他元素。由于这是一个常见的情况，如果布局类是PatternLayout，那么class属性可以省略[默认的类映射](https://logback.qos.ch/manual/onJoran.html#defaultClassMapping) 规则指定。
>
>该<encoder>元素采用指定编码器类的完全限定名实例化的强制类属性。由于这是一个常见的情况，如果编码器类是PatternLayoutEncoder，那么class属性可以省略，如[默认的类映射](https://logback.qos.ch/manual/onJoran.html#defaultClassMapping) 规则所指定的。

##### 3.4 设置上下文名称

>每个记录器都附加到记录器上下文中。默认情况下，记录器上下文称为`“default”`。可以使用`<contextName>`设置不同的名称。一旦设置，记录器上下文名称不能更改。
>
>```xml
><configuration>
>
>    <contextName>myAppName</contextName>
>
>    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
>        <encoder>
>            <pattern>%d %contextName [%t] %level %logger{36} - %msg%n</pattern>
>        </encoder>
>    </appender>
>
>    <root level="debug">
>        <appender-ref ref="STDOUT" />
>    </root>
></configuration>
>```

##### 3.5  变量替换

>变量有一个scope。而且，变量可以在配置文件本身，外部文件，外部资源中定义。甚至可以在运行中进行计算和定义。
>
>变量替换的语法与UNIX shell相似。==${..}之间的字符串被解释为对该属性值的引用==。

##### 3.6 定义变量

>可以在配置文件中定义或者从外部属性文件或外部资源批量加载变量。
>
>示例1：简单变量替换
>
>```xml
><configuration>
>
>    <property name="USER_HOME" value="/home/sebastien" />
>
>    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
>        <file>${USER_HOME}/myApp.log</file>
>        <encoder>
>            <pattern>%msg%n</pattern>
>        </encoder>
>    </appender>
>
>    <root level="debug">
>        <appender-ref ref="FILE" />
>    </root>
>
></configuration>
>```
>
>示例2：系统变量替换
>
>```xml
><configuration>
>
>    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
>        <file>${USER_HOME}/myApp.log</file>
>        <encoder>
>            <pattern>%msg%n</pattern>
>        </encoder>
>    </appender>
>
>    <root level="debug">
>        <appender-ref ref="FILE" />
>    </root>
>
></configuration>
>
>```
>
>示例3：使用单独文件进行变量替换
>
>```xml
><configuration>
>
>    <property file="src/main/java/chapters/configuration/variables1.properties" />
>
></configuration>
>```
>
>示例4：引用类路径上的资源而不是文件
>
>```xml
><configuration>
>
>    <property resource="resource1.properties" />
>
></configuration>
>```

##### 3.7 定义变量范围

>在替换过程中，首先在本地范围内查找属性，在上下文范围内，在系统属性范围内，最后在[OS环境中](http://docs.oracle.com/javase/tutorial/essential/environment/env.html)。
>
>**本地范围**
>
>从配置文件中定义其属性的位置到该配置文件的解释/执行结束，都存在具有局部范围的属性。 因此，每次解析并执行配置文件时，都会重新定义本地作用域中的变量。
>
>**上下文范围**
>
>具有上下文范围的属性会插入到上下文中，并且持续时间与上下文一样长，直到清除为止。 一旦定义，上下文范围内的属性就是上下文的一部分。 这样，它在所有日志记录事件中都可用，包括那些通过序列化发送到远程主机的事件。
>
>**系统范围**
>
>具有系统范围的属性被插入JVM的系统属性中，并且持续时间与JVM一样长，或者直到被清除为止。
>
>`<property>`元素，`<define>`元素或`<insertFromJNDI>`元素的==scope属性==可用于设置属性的范围。 scope属性允许将“==local==”，“==context==”和“==system==”字符串作为可能的值。如果未指定，则范围始终假定为“local”。
>
>示例：在“上下文”范围中定义变量
>
>```xml
><configuration>
>
>    <property scope="context" name="nodeId" value="firstNode" />
>
></configuration>
>```
>
>假设上下文范围中定义了nodeId属性，它将在每个日志事件中都可用，即使是通过序列化发送到远程主机的日志事件。

##### 3.8 变量的默认值

>如果一个变量没有被声明或者它的值为null，那么可能有一个默认值。和Bash shell一样，可以使用 ==:-== 运算符来指定默认值。例如，假设名称为*aName*的变量未定义，将被解释为“golden”。"${aName==:-golden==}"

#### 4. appender

#### 5. encoder

#### 6. layout

#### 7. filters



