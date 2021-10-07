**Application Performance Monitor (*APM*)是应用性能监测软件）**



New Relic目前专注于SaaS和App性能管理业务，它支持支持agent和API传送数据，能够对部署在本地或在云中的web应用程序进行监控、故障修复、诊断、线程分析以及容量计划。

- 端对端事务跟踪：跟踪一个关键事务的性能，这个事务贯穿在整个面向服务应用程序环境。
- 代码级的可见性：深入洞察特定代码段和SQL语句对性能的影响。
- 关键事务：标记你的最关键的事务，当响应时间、调用、错误率等这些表现不佳的时候可以迅速的发现。
- X光会话：通过展示事务跟踪长期分析的结果，来获得对一个关键事务性能更深入的了解。



#### 1. 简单工作原理

---

RPM拥有两种基本的组件：==作为应用程序插件运行的代理==，以及==放置在New Relic数据中心中的服务==。

- 代理会收集性能数据，每分钟都会通过HTTPS或者HTTP协议异步地发送给RPM服务，New Relic那里会存储并处理这些数据。
- New Relic数据中心会完成以下的工作：数据存储、聚集、修正和可视化。我们可以通过浏览器访问性能数据。 New Relic不提供在本地运行服务的方案，服务只运行在他们的数据中心上。



#### 2. 下载安装代理

---

直接下载zip包，或者使用以下命令：

```shell
curl -O https://download.newrelic.com/newrelic/java-agent/newrelic-agent/current/newrelic-java.zip
```

>下载代理后，请按照以下步骤开始 Java 代理安装。
>
>1. 为您的 New Relic 文件创建一个目录，例如`/opt/newrelic`. 在 Windows 上，New Relic 文件必须位于应用程序服务器目录的子目录中，例如`C:\Program Files\Apache Software Foundation\Tomcat 9.0\newrelic`.
>2. 将解压后的`newrelic`目录中的所有 New Relic 文件复制到新目录中。
>3. 用`newrelic.yml`您下载的自定义配置文件替换该文件。

启动应用：

```shell
java -javaagent:FULL_PATH_TO/newrelic.jar -jar app.jar
```



#### 3. 删除监控实例

---

1. 将newrelic.yml配置文件中agent_enabled改成false，或者[卸载代理](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fdocs.newrelic.com%2Fdocs%2Fagents%2Fmanage-apm-agents%2Finstallation%2Funinstall-agent)。
2. 重启应用服务器
3. 等待几分钟（10分钟），看到Application名称变成灰色
4. 删除

