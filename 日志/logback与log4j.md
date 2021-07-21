#### 1. logback

---

logback当前分为三个模块：

- logback-core（核心基础模块）
- logback-classic（是log4j的一个改良版本，完整实现了SLF4J API，可以很方便地更换成其它日志系统）
- logback-access（访问模块与Servlet容器集成提供通过Http来访问日志的功能）

logback要与SLF4J结合使用。



#### 2. logback配置

---

##### 2.1 logger、appender及layout

**logger**作为日志的记录器，把它==关联到应用的对应的context上==后，主要用于存放对象，也可以定义日志类型、级别。

**appender**主要用于==指定日志输出的目的地==，目的地可以是控制台、文件、远程套接字服务器、MySQL和其他数据库、JMS和远程UNIX Syslog守护进程等。Layout负责把事件转换成字符串，格式化的日志信息的输出。

**layout**负责把事件转换成字符串，格式化的日志信息的输出。

##### 2.2 logger context

每个logger都被关联到一个loggerContext，loggerContext负责制造logger，也负责以树结构排列各logger。其他所有logger也通过org.slf4j.LoggerFactory类的静态方法getLogger取得。getLogger方法以logger名称为参数。用同一名字调用LoggerFactory.getLogger方法所得到的永远都是同一个logger对象的引用。

##### 2.3 有效级别及级别的继承

logger可以被分配级别。TRACE < DEBUG < INFO < WARN < ERROR。如果logger没有被分配级别，那么它将从有被分配级别的最近的祖先那里继承级别。root logger默认级别是DEBUG。



#### 3. logback的默认配置

---

如果配置文件`logback-test.xml`和`logback.xml`都不存在，那么==logback默认地会调用BasicConfigurator，创建一个最小化配置==。最小化配置由一个关联到根logger的ConsoleAppender组成。输出用模式为`%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n`的PatternLayoutEncoder进行格式化。rootlogger默认级别是DEBUG。

配置文件的基本结构：以`<configuration>`开头，后面有零个或多个`<appender>`元素，有零个或多个`<logger>`元素，有最多一个`<root>`元素。

>Logback默认配置的步骤
>
>1. 尝试在classpath下查找文件logback-test.xml；
>2. 如果文件不存在，则查找文件logback.xml；
>3. 如果两个文件都不存在，logback用BasicConfigurator自动对自己进行配置，这会导致记录输出到控制台。

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-api</artifactId>
  <version>1.7.25</version>
</dependency>
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-core</artifactId>
  <version>1.1.11</version>
</dependency>
<dependency>
  <groupId>ch.qos.logback</groupId>
  <artifactId>logback-classic</artifactId>
  <version>1.1.11</version>
</dependency>

```

