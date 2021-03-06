1、环境要求
--
* Java： 1.7+
* Guava：15.0+（Apollo客户端默认会引用Guava 19，如果你的项目引用了其它版本，请确保版本号大于等于15.0）

2、必选设置
--
Apollo客户端依赖于AppId，Apollo Meta Server等环境信息来工作。

#### 2.1 AppId

---

AppId（应用的身份信息，是从服务端获取配置的一个重要信息），有以下几种设置方式，按照优先级从高到低：

> 1. System Property
> Apollo 0.7.0+支持通过System Property传入app.id信息，如
> -Dapp.id=YOUR-APP-ID
>
> 2. 操作系统的System Environment
> Apollo 14.0+支持通过操作系统的System Environment APP_ID来传入app.id信息，如：
> APP_ID=YOUR-APP-ID
>
> 3. Spring Boot application.properties(该配置方式不适用于多个war包部署在同一个tomcat的使用场景)
> Apollo 1.0.0+支持通过Spring Boot的application.properties文件配置，如：
> app.id=YOUR-APP-ID
>
> 4. ==app.properties==
>     ==确保classpath:/META-INF/app.properties文件存在==，内容形如：app.id=YOUR-APP-ID，app.id是用来标识应用身份的唯一id，格式为string。


#### 2.2 Meta Server

---

Apollo Meta Server，Apollo支持应用在不同的环境有不同的配置，所以需要在运行提供给Apollo客户端当前环境的Apollo Meta Server信息。
默认情况下，meta server和config service是部署在同一个JVM进程，所以meta server的地址就是config service的地址。

> 为了实现meta server的高可用，推荐通过SLB（Software Load Balancer）做动态负载均衡。Meta server地址也可以填入IP，如http://1.1.1.1:8080,http://2.2.2.2:8080，不过生产环境还是建议使用域名（走slb），因为机器扩容、缩容等都可能导致IP列表的变化。
>
> 1.0.0版本开始支持以下方式配置apollo meta server信息，按照优先级从高到低分别为：
>
> 1. 通过Java System Property apollo.meta来指定
>    - 在Java程序启动脚本中，可以指定-Dapollo.meta=http://config-service-url
>      如果是运行jar文件，需要注意格式是java -Dapollo.meta=http://config-service-url -jar xxx.jar
>    - 也可以通过程序指定，如System.setProperty("apollo.meta", "http://config-service-url");
>
> 2. 通过Spring Boot的配置文件（该配置方式不适用于多个war包部署在同一个tomcat的使用场景）
>    - 可以在Spring Boot的application.properties或bootstrap.properties中指定apollo.meta=http://config-service-url
>
> 3. 通过操作系统的System EnvironmentAPOLLO_META
>     * 可以通过操作系统的System Environment APOLLO_META来指定
>     * 注意key为全大写，且中间是_分隔
>
> 4. 通过==server.properties==配置文件
>    - 可以在server.properties配置文件中指定apollo.meta=http://config-service-url
>    - 对于Mac/Linux，文件位置为/opt/settings/server.properties
>    - 对于Windows，文件位置为C:\opt\settings\server.properties
>
> 5. 通过==app.properties==配置文件
>     * 可以在classpath:/META-INF/app.properties指定apollo.meta=http://config-service-url
>
> 6. 通过Java system property ${env}_meta
>     * 如果当前env是dev，那么用户可以配置-Ddev_meta=http://config-service-url
>     * 使用该配置方式，那么就必须要正确配置Environment，详见1.2.4.1 Environment
>
> 7. 通过操作系统的System Environment ${ENV}_META (1.2.0版本开始支持)
>     * 如果当前env是dev，那么用户可以配置操作系统的System Environment DEV_META=http://config-service-url
>     * 注意key为全大写
>     * 使用该配置方式，那么就必须要正确配置Environment，详见1.2.4.1 Environment
>
> 8. 通过apollo-env.properties文件
>     * 用户也可以创建一个apollo-env.properties，放在程序的classpath下，或者放在spring boot应用的config目录下
>     * 使用该配置方式，那么就必须要正确配置Environment，详见1.2.4.1 Environment
>     * 文件内容形如：
>     dev.meta=http://1.1.1.1:8080
>     fat.meta=http://apollo.fat.xxx.com
>     uat.meta=http://apollo.uat.xxx.com
>     pro.meta=http://apollo.xxx.com
>
> 如果通过以上各种手段都无法获取到Meta Server地址，Apollo最终会fallback到http://apollo.meta作为Meta Server地址

`自定义Apollo Meta Server地址定位逻辑`

`跳过Apollo Meta Server服务发现`

> **本地缓存路径**
>
> ==Apollo客户端会把从服务端获取到的配置在本地文件系统缓存一份==，用于在遇到服务不可用，或网络不通的时候，依然能从本地恢复配置，不影响应用正常运行。
>
> 本地缓存路径默认位于以下路径，所以请确保`/opt/data`或`C:\opt\data\`目录存在，且应拥有读写权限。
>
> - Mac/Linux: /opt/data/{appId}/config-cacheWindows: C:\opt\data\{appId}\config-cache

本地配置文件会以下面的文件名格式放置于本地缓存路径下：

`{appId}+{cluster}+{namespace}.properties`

* appId就是应用自己的appId，如100004458
* cluster就是应用使用的集群，一般在本地模式下没有做过配置的话，就是default
* namespace就是应用使用的配置namespace，一般是application 

`自定义缓存路径`

3、可选设置
--
>Environment，Environment可以通过以下3种方式的任意一个配置：

2）通过操作系统的System Environment
    * 还可以通过操作系统的System Environment ENV来指定
    * 注意key为全大写

3）通过配置文件
    * 最后一个推荐的方式是通过配置文件来指定env=YOUR-ENVIRONMENT
    * 对于Mac/Linux，文件位置为/opt/settings/server.properties
    * 对于Windows，文件位置为C:\opt\settings\server.properties

 文件内容形如：
 env=DEV

 目前，env支持以下几个值（大小写不敏感）：
 * DEV（Development environment）
 * FAT（Feature Acceptance Test environment）
 * UAT（User Acceptance Test environment）
 * PRO（Production environment）

```            

> Cluster（Apollo支持配置按照集群划分，也就是说对于一个appId和一个环境，对不同的集群可以有不同的配置。）

4、Maven Dependency
--
Apollo的客户端jar包已经上传到中央仓库，应用在实际使用时只需要按照如下方式引入即可。

provider和consumer的pom中引入以下依赖：
```xml
<dependency>
    <groupId>com.ctrip.framework.apollo</groupId>
    <artifactId>apollo-client</artifactId>
    <version>1.7.0</version>
</dependency>
```

5、客户单用法（支持API方式和Spring整合方式）
--

* API方式灵活，配置值实时更新，支持所有Java环境。
* Spring方式接入简单，结合Spring有N种酷炫的玩法，如
    * ==Placeholder方式==：
        * 代码中直接使用，`@Value("${someKeyFromApollo:someDefaultValue}")`
        * 配置文件中使用替换placeholder，`spring.datasource.url: ${someKeyFromApollo:someDefaultValue}`
        * 直接托管spring的配置，`在apollo中直接配置spring.datasource.url=jdbc:mysql://localhost:3306/somedb?characterEncoding=utf8`
    * Spring boot的@ConfigurationProperties方式
    * 从v0.10.0开始的版本支持placeholder在运行时自动更新。 
    * Spring方式也可以结合API方式使用，如注入Apollo的Config对象，就可以照常通过API方式获取配置了：

```text
@ApolloConfig
private Config config;
```

> API使用方式（API方式不依赖Spring框架即可使用）

`默认的namespace配置获取`

```text
// config实例对于每个名称空间都是单例，并且永远不会为null
Config config = ConfigService.getAppConfig();
String someKey = "someKeyFromDefaultNamespace";
String someDefaultValue = "someDefaultValueForTheKey";
String value = config.getProperty(someKey, someDefaultValue);

通过config.getProperty可以获取到某个key对应的实时最新的配置值。另外配置值从内存中获取，所以不需要应用自己做缓存。
```

`监听配置变化事件`
```text
// 监听配置变化事件只在应用真的关心配置变化，需要在配置变化时得到通知时使用，比如：数据库连接串变化后需要重建连接等。
// 如果只是希望每次都取到最新的配置的话，只需要按照上面的例子，调用config.getProperty即可。
Config config = ConfigService.getAppConfig(); 
config.addChangeListener(new ConfigChangeListener() {
    @Override
    public void onChange(ConfigChangeEvent changeEvent) {
        System.out.println("Changes for namespace " + changeEvent.getNamespace());
        for (String key : changeEvent.changedKeys()) {
            ConfigChange change = changeEvent.getChange(key);
            System.out.println(String.format("Found change - key: %s, oldValue: %s, newValue: %s, changeType: %s", change.getPropertyName(), change.getOldValue(), change.getNewValue(), change.getChangeType()));
        }
    }
});
```

`获取公共Namespace的配置`
```text
String somePublicNamespace = "CAT";
Config config = ConfigService.getConfig(somePublicNamespace);
String someKey = "someKeyFromPublicNamespace";
String someDefaultValue = "someDefaultValueForTheKey";
String value = config.getProperty(someKey, someDefaultValue);
```

`获取非properties格式namespace的配置`
```text
// yaml/yml格式的namespace
Config config = ConfigService.getConfig("application.yml");
String someKey = "someKeyFromYmlNamespace";
String someDefaultValue = "someDefaultValueForTheKey";
String value = config.getProperty(someKey, someDefaultValue);

// 非yaml/yml格式的namespace,获取时需要使用ConfigService.getConfigFile接口并指定Format，如ConfigFileFormat.XML。
String someNamespace = "test";
ConfigFile configFile = ConfigService.getConfigFile("test", ConfigFileFormat.XML);
String content = configFile.getContent();
```

> 基于Java配置
>
> 注意==@EnableApolloConfig要和@Configuration一起使用==，不然不会生效。

```text
1.注入默认namespace的配置到Spring中
//这个是最简单的配置形式，一般应用用这种形式就可以了，用来指示Apollo注入application namespace的配置到Spring环境中
@Configuration
@EnableApolloConfig
public class AppConfig {
  @Bean
  public TestJavaConfigBean javaConfigBean() {
    return new TestJavaConfigBean();
  }
}

2.注入多个namespace的配置到Spring中
@Configuration
@EnableApolloConfig
public class SomeAppConfig {
  @Bean
  public TestJavaConfigBean javaConfigBean() {
    return new TestJavaConfigBean();
  }
}
   
//这个是稍微复杂一些的配置形式，指示Apollo注入FX.apollo和application.yml namespace的配置到Spring环境中
@Configuration
@EnableApolloConfig({"FX.apollo", "application.yml"})
public class AnotherAppConfig {}

3.注入多个namespace，并且指定顺序
//这个是最复杂的配置形式，指示Apollo注入FX.apollo和application.yml namespace的配置到Spring环境中，并且顺序在application前面
@Configuration
@EnableApolloConfig(order = 2)
public class SomeAppConfig {
  @Bean
  public TestJavaConfigBean javaConfigBean() {
    return new TestJavaConfigBean();
  }
}
@Configuration
@EnableApolloConfig(value = {"FX.apollo", "application.yml"}, order = 1)
public class AnotherAppConfig {}
```

6、Spring Boot整合方式
--

Spring Boot除了支持上述基于xml配置和基于java配置集成方式以外，还支持通过`application.properties/bootstrap.properties`来配置，该方式能使配置在更早的阶段注入，
比如使用@ConditionalOnProperty的场景或者是有一些spring-boot-starter在启动阶段就需要读取配置做一些事情，所以对于Spring Boot环境建议通过以下方式来接入Apollo。

使用方式很简单，只需要在application.properties/bootstrap.properties中按照如下样例配置即可。
```properties
#1.注入默认application namespace的配置示例
apollo.bootstrap.enabled = true
```
```properties
#2.注入非默认application namespace或多个namespace的配置示例
apollo.bootstrap.enabled = true
# 将在引导阶段注入“application”，“FX.apollo”和“application.yml”名称空间
apollo.bootstrap.namespaces = application,FX.apollo,application.yml
```
```properties
#3.将Apollo配置加载提到初始化日志系统之前
apollo.bootstrap.enabled = true
# 在记录系统初始化之前将apollo初始化
apollo.bootstrap.eagerLoad.enabled=true
```

7、Spring Placeholder的使用
--
Spring应用通常会使用Placeholder来注入配置，使用的格式形如`${someKey:someDefaultValue}`，如`${timeout:100}`。

`建议在实际使用时尽量给出默认值`，以免由于key没有定义导致运行时错误。

从v0.10.0开始的版本支持placeholder在运行时自动更新。

如果需要关闭placeholder在运行时自动更新功能，可以通过以下两种方式关闭：
* 通过设置System Property apollo.autoUpdateInjectedSpringProperties，如启动时传入-Dapollo.autoUpdateInjectedSpringProperties=false

* 通过设置META-INF/app.properties中的apollo.autoUpdateInjectedSpringProperties属性，如
```properties
app.id=SampleApp
apollo.autoUpdateInjectedSpringProperties=false
```

> Java Config使用方式

假设有一个TestJavaConfigBean，通过Java Config的方式还可以使用@Value的方式注入：
```java
public class TestJavaConfigBean {
  @Value("${timeout:100}")
  private int timeout;
  private int batch;
 
  @Value("${batch:200}")
  public void setBatch(int batch) {
    this.batch = batch;
  }
 
  public int getTimeout() {
    return timeout;
  }
 
  public int getBatch() {
    return batch;
  }
}
```

在Configuration类中按照下面的方式使用（假设应用默认的application namespace中有timeout和batch的配置项）：
```java
@Configuration
@EnableApolloConfig
public class AppConfig {
  @Bean
  public TestJavaConfigBean javaConfigBean() {
    return new TestJavaConfigBean();
  }
}
```

> ConfigurationProperties使用方式

Spring Boot提供了@ConfigurationProperties把配置注入到bean对象中。Apollo也支持这种方式，下面的例子会把redis.cache.expireSeconds和redis.cache.commandTimeout分别注入到SampleRedisConfig的expireSeconds和commandTimeout字段中。
```java
@ConfigurationProperties(prefix = "redis.cache")
public class SampleRedisConfig {
  private int expireSeconds;
  private int commandTimeout;

  public void setExpireSeconds(int expireSeconds) {
    this.expireSeconds = expireSeconds;
  }

  public void setCommandTimeout(int commandTimeout) {
    this.commandTimeout = commandTimeout;
  }
}
```

在Configuration类中按照下面的方式使用（假设应用默认的application namespace中有redis.cache.expireSeconds和redis.cache.commandTimeout的配置项）：
```java
@Configuration
@EnableApolloConfig
public class AppConfig {
  @Bean
  public SampleRedisConfig sampleRedisConfig() {
    return new SampleRedisConfig();
  }
}
```

需要注意的是，@ConfigurationProperties如果需要在Apollo配置变化时自动更新注入的值，需要配合使用EnvironmentChangeEvent或RefreshScope。

8、Spring Annotation支持
--
* @ApolloConfig 用来自动注入Config对象
* @ApolloConfigChangeListener   用来自动注册ConfigChangeListener
* @ApolloJsonValue  用来把配置的json字符串自动注入为对象

```java
public class TestApolloAnnotationBean {
  @ApolloConfig
  private Config config; //inject config for namespace application
  @ApolloConfig("application")
  private Config anotherConfig; //inject config for namespace application
  @ApolloConfig("FX.apollo")
  private Config yetAnotherConfig; //inject config for namespace FX.apollo
  @ApolloConfig("application.yml")
  private Config ymlConfig; //inject config for namespace application.yml
 
  /**
   * ApolloJsonValue annotated on fields example, the default value is specified as empty list - []
   * <br />
   * jsonBeanProperty=[{"someString":"hello","someInt":100},{"someString":"world!","someInt":200}]
   */
  @ApolloJsonValue("${jsonBeanProperty:[]}")
  private List<JsonBean> anotherJsonBeans;

  @Value("${batch:100}")
  private int batch;
  
  //config change listener for namespace application
  @ApolloConfigChangeListener
  private void someOnChange(ConfigChangeEvent changeEvent) {
    //update injected value of batch if it is changed in Apollo
    if (changeEvent.isChanged("batch")) {
      batch = config.getIntProperty("batch", 100);
    }
  }
 
  //config change listener for namespace application
  @ApolloConfigChangeListener("application")
  private void anotherOnChange(ConfigChangeEvent changeEvent) {
    //do something
  }
 
  //config change listener for namespaces application, FX.apollo and application.yml
  @ApolloConfigChangeListener({"application", "FX.apollo", "application.yml"})
  private void yetAnotherOnChange(ConfigChangeEvent changeEvent) {
    //do something
  }

  //example of getting config from Apollo directly
  //this will always return the latest value of timeout
  public int getTimeout() {
    return config.getIntProperty("timeout", 200);
  }

  //example of getting config from injected value
  //the program needs to update the injected value when batch is changed in Apollo using @ApolloConfigChangeListener shown above
  public int getBatch() {
    return this.batch;
  }

  private static class JsonBean{
    private String someString;
    private int someInt;
  }
}
```

在Configuration类中按照下面的方式使用：
```java
@Configuration
@EnableApolloConfig
public class AppConfig {
  @Bean
  public TestApolloAnnotationBean testApolloAnnotationBean() {
    return new TestApolloAnnotationBean();
  }
}
```

9、已有配置迁移
--
* 在Apollo为应用新建项目
* 在应用中配置好META-INF/app.properties
* 建议把原先配置先转为properties格式，然后通过Apollo提供的文本编辑模式全部粘帖到应用的application namespace，发布配置
    * 如果原来格式是yml，可以使用YamlPropertiesFactoryBean.getObject转成properties格式
* 如果原来是yml，想继续使用yml来编辑配置，那么可以创建私有的application.yml namespace，把原来的配置全部粘贴进去，发布配置
    * 需要apollo-client是1.3.0及以上版本
* 把原先的配置文件如bootstrap.properties/yml, application.properties/yml从项目中删除
    * 如果需要保留本地配置文件，需要注意部分配置如server.port必须确保本地文件已经删除该配置项

10、Apollo客户端设计
--
* 客户端和服务端保持了一个长连接，从而能第一时间获得配置更新的推送。（通过Http Long Polling实现）
* 客户端还会定时从Apollo配置中心服务端拉取应用的最新配置。
* 这是一个fallback机制，为了防止推送机制失效导致配置不更新
* 客户端定时拉取会上报本地版本，所以一般情况下，对于定时拉取的操作，服务端都会返回304 - Not Modified
* 定时频率默认为每5分钟拉取一次，客户端也可以通过在运行时指定System Property: apollo.refreshInterval来覆盖，单位为分钟。
* 客户端从Apollo配置中心服务端获取到应用的最新配置后，会保存在内存中
* 客户端会把从服务端获取到的配置在本地文件系统缓存一份
* 在遇到服务不可用，或网络不通的时候，依然能从本地恢复配置
* 应用程序可以从Apollo客户端获取最新的配置、订阅配置更新通知

11、本地开发模式
--
Apollo客户端还支持本地开发模式，这个主要用于当开发环境无法连接Apollo服务器的时候，比如在邮轮、飞机上做相关功能开发。

在本地开发模式下，Apollo只会从本地文件读取配置信息，不会从Apollo服务器读取配置。

可以通过下面的步骤开启Apollo本地开发模式。

> 修改环境

修改/opt/settings/server.properties（Mac/Linux）或C:\opt\settings\server.properties（Windows）文件，设置env为Local：

`env=Local`

> 准备本地配置文件

在本地开发模式下，Apollo客户端会从本地读取文件，所以我们需要实现准备好配置文件。
```text
Mac/Linux: /opt/data/{appId}/config-cache
Windows: C:\opt\data\{appId}\config-cache
```
推荐的方式是先在普通模式下使用Apollo，这样Apollo会自动创建该目录并在目录下生成配置文件。

> 本地配置文件

本地配置文件需要按照一定的文件名格式放置于本地配置目录下，文件名格式如下：

`{appId}+{cluster}+{namespace}.properties`
* appId就是应用自己的appId，如100004458
* cluster就是应用使用的集群，一般在本地模式下没有做过配置的话，就是default
* namespace就是应用使用的配置namespace，一般是application 

文件内容以properties格式存储，比如如果有两个key。

> 修改配置

在本地开发模式下，Apollo不会实时监测文件内容是否有变化，所以如果修改了配置，需要重启应用生效。

