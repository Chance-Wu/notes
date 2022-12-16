### 一、市面上的日志框架以及日志抽象层类

---

日志框架分为3部分：**日志门面**、**日志适配器**、**日志库**。

1. 日志门面：JCL、SLF4j、jboss-logging这些框架都是日志门面。门面设计模式是面向对象的一种设计模式，类似JDBC，也就是说这些货本身自己不干活，就是一套接口规范，让调用者不需要关心日志底层具体是什么框架在干活
2. 日志库：log4j、logback、JUL、log4j2都是日志库，也就是真实干活的人
3. 日志适配器：它是解决日志门面和日志库接口不兼容的，一般配套的都是兼容的

所以我们的目标是挑选一个日志门面，再挑选一个日志库，搭配使用即可，Log4j2是Log4j升级版、Log4j2是Apache写的一套框架，由于太优秀太过复杂，因此也比较小众。JUL太简陋了基本没人用。



### 方案一：slf4j+Logback

---

在springboot中，**默认抽象接口层使用slf4j**，**实现层用logback**，创建了一个demo项目，当我们引入spring-boot-starter的时候，就默认帮我们引入logback、slf4j。

![image-20221216133134071](img/image-20221216133134071.png)

当我们创建一个boot项目，我们没有配置任何其它配置，就看到在控制台下打印日志，Logback默认打印debug级别日志，但是我们注意到debug级别的日志没有记录下来，那是因为**Spring Boot为Logback提供了默认的配置文件base.xml**，另外Spring Boot 提供了两个输出端的配置文件console-appender.xml和file-appender.xml，base.xml引用了这两个配置文件。所以Springboot默认日志级别是info，可以看到base.xml引用两个配置。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<included>
  <include resource="org/springframework/boot/logging/logback/defaults.xml" />
  <property name="LOG_FILE" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log}"/>
  <include resource="org/springframework/boot/logging/logback/console-appender.xml" />
  <include resource="org/springframework/boot/logging/logback/file-appender.xml" />
  <root level="INFO">
    <appender-ref ref="CONSOLE" />
    <appender-ref ref="FILE" />
  </root>
</included>
```

在这里，可以看到 Spring boot已通过将根记录器设置为INFO来覆盖 Logback 的默认日志记录级别，这是我们在上面的示例中没有看到调试消息的原因。

#### 第一种：简单配置

小项目可以直接在application.yml中对logback简单配置，如下。

```yaml
logging:
  level:
    com:
      javayihao: trace
  path: /spring/log
  pattern:
    console: '%d{yyyy-MM-dd HH:mm:ss.SSS}+++[%thread] %-5level %logger{50} - %msg%n'
    file: '%d{yyyy-MM-dd HH:mm:ss.SSS}===[%thread] %-5level %logger{50} - %msg%n'
```

#### 第二种：通过logback专有的xml配置文件详细配置

通过application.yml文件配置Logback，对于大多数Spring Boot应用来说已经足够了，但是对于一些大型的企业应用来说似乎有一些相对复杂的日志需求。在Spring Boot中你可以在logback.xml或者在logback-spring.xml中对Logback进行配置，相对于 `logback.xml`，`logback-spring.xml` 更加被偏爱。如果是其他名字，只需在application.properties中配置logging.config=classpath:logback-boot.xml使用自己的日志配置文件即可。

```xml
<configuration scan="true" scanPeriod="60 seconds" debug="false">  
  <property name="glmapper-name" value="glmapper-demo" /> 
  <contextName>${glmapper-name}</contextName> 

  <appender>
    //xxxx
  </appender>   

  <logger>
    //xxxx
  </logger>

  <root>             
    //xxxx
  </root>  
</configuration>
```

**`<configuration/>`标签**

- scan：当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。
- scanPeriod：设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。
- debug：当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。

**`<property/>`标签：用来定义变量值的标签，有两个属性**

- name的值是变量的名称
- value的值时变量定义的值。

通过property定义的值会被插入到logger上下文中。定义变量后，可以使“${name}”来使用变量。如上面的xml所示。

**`<contextName/>`标签**

每个logger都关联到logger上下文，默认上下文名称为“default”。但可以使用contextName标签设置成其他名字，用于区分不同应用程序的记录。

**`<appender/>`标签**

appender种类：

- ConsoleAppender：把日志添加到控制台
- FileAppender：把日志添加到文件
- RollingFileAppender：滚动记录文件，先将日志记录到指定文件，当符合某个条件时，将日志记录到其他文件。它是FileAppender的子类

**`<filter/>`子标签**

filter是appender里面的子元素。它作为过滤器存在，执行一个过滤器会有返回DENY，NEUTRAL，ACCEPT三个枚举值中的一个。appender 有多个过滤器时，按照配置顺序执行。

- DENY：日志将立即被抛弃不再经过其他过滤器
- NEUTRAL：有序列表里的下个过滤器过接着处理日志
- ACCEPT：日志会被立即处理，不再经过剩余过滤器

filter的class有ThresholdFilter以及LevelFilter



























