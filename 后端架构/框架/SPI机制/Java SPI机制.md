#### SPI在实际项目中的应用

在mysql-connector-java-xxx.jar中发现了`META-INF\services\java.sql.Driver`文件，里面只有两行记录：

```
com.mysql.jdbc.Driver
com.mysql.fabric.jdbc.FabricMySQLDriver
```

由上可以分析出，java.sql.Driver是一个规范接口，`com.mysql.jdbc.Driver`，`com.mysql.fabric.jdbc.FabricMySQLDriver`则是mysql-connector-java-xxx.jar对这个规范的实现接口。

在jcl-over-slf4j-xxxx.jar中发现了`META-INF\services\org.apache.commons.logging.LogFactory`文件，里面只有一行记录：

```
org.apache.commons.logging.impl.SLF4JLogFactory
```

>补充：class.forName(“com.mysql.jdbc.Driver”)到底做了什么事？
>
>class.forName与类加载机制有关，会触发执行com.mysql.jdbc.Driver类中的静态方法，从而使主类加载数据库驱动。如果再追问，为什么它的静态块没有自动触发？可答：因为数据库驱动类的特殊性质，JDBC规范中明确要求Driver类必须向DriverManager注册自己，导致其必须由class.forName手动触发，这可以在java.sql.Driver中得到解释。完美了吗？还没，来到最新的DriverManager源码中，可以看到这样的注释：
>
>`DriverManager`类的方法 `getConnection` 和 `getDrivers` 已经得到提高以支持 Java Standard Edition [Service Provider](http://tool.oschina.net/uploads/apidocs/technotes/guides/jar/jar.html#Service Provider) 机制。 JDBC 4.0 Drivers 必须包括 `META-INF/services/java.sql.Driver` 文件。此文件包含 `java.sql.Driver` 的 JDBC 驱动程序实现的名称。例如，要加载 `my.sql.Driver` 类，`META-INF/services/java.sql.Driver` 文件需要包含条目：`my.sql.Driver`
>
>应用程序不再需要使用 `Class.forName()` 显式地加载 JDBC 驱动程序。当前使用 `Class.forName()` 加载 JDBC 驱动程序的现有程序将在不作修改的情况下继续工作。
>
>可以发现Class.forName()已经被弃用了，所以，这道题目的最佳回答，是回答Java中的SPI机制，进而聊聊加载驱动的演变历史。

#### SPI在扩展方面的应用

---

SPI不仅仅是为厂商指定的标准，同样也为框架扩展提供了一个思路。框架可以预留出SPI接口，这样可以在不侵入代码的前提下，通过增删依赖来扩展框架。前提是，框架得预留出核心接口，剩下的适配工作便留给了开发者。