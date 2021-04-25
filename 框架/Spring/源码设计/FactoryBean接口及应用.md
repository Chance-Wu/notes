#### FactoryBean接口

---

>一般情况下，Spring通过反射机制利用的class属性指定实现类实例化Bean，在某些情况下，实例化Bean过程比较复杂，这时采用编码的方式可能会得到一个简单的方案。用户可以通过实现FactoryBean接口定制实例化Bean的逻辑。

FactoryBean它首先==是一个Bean==，其次是==一个能生产或修饰对象生成的工厂Bean==。==spring会在使用getBean()调用获得该bean时，会自动调用该bean的getObject()方法==，所以返回的不是factory这个bean，而是bean.getObject()方法的返回值：

```java
public interface FactoryBean<T> {

  String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

  @Nullable
  T getObject() throws Exception;

  @Nullable
  Class<?> getObjectType();

  default boolean isSingleton() {
    return true;
  }

}
```

> ==SqlMapClientFactoryBean返回的是getObject()中返回的sqlMapClient==，而不是SqlMapClientFactoryBean自己的实例。



#### FactoryBean在Spring中应用

---

Spring中bean会被Spring的IoC容器管理，在AbstractApplicationContext中有一个refresh()方法，项目启动或重启的时候refresh()会调用getBean()初始化所有的Bean，这个getBean()会最终指向AbstractBeanFactory中的getBean()方法。

AbstractBeanFactory中，不管是根据名称还是根据类型，getBean()最终都会调用doGet，在doGetBean()方法中一开始就获取了名称beanName和实例sharedInstance。

```java
String beanName = transformedBeanName(name); //获取真正的名称，会去掉name前面的'&'
Object bean;

Object sharedInstance = getSingleton(beanName); //从容器中的获取这个Bean的实例
```

拿到sharedInstance后，后面的操作做了单例、多例等判断，最终会走到getObjectForBeanInstance()：

```java
protected Object getObjectForBeanInstance(
  Object beanInstance, String name, String beanName, @Nullable RootBeanDefinition mbd) {

  // 判断name是否不为空且以&开头
  if (BeanFactoryUtils.isFactoryDereference(name)) {
    if (beanInstance instanceof NullBean) {
      return beanInstance;
    }
    // 判断beanInstance是否属于FactoryBean或者其子类的实例
    if (!(beanInstance instanceof FactoryBean)) {
      throw new BeanIsNotAFactoryException(beanName, beanInstance.getClass());
    }
  }

  // 如果beanInstance不属于Factory或其子类的实例，或者name是以'&'开头，就直接返回实例对象beanInstance
  if (!(beanInstance instanceof FactoryBean) || BeanFactoryUtils.isFactoryDereference(name)) {
    return beanInstance;
  }

  Object object = null;
  if (mbd == null) {
    object = getCachedObjectForFactoryBean(beanName);
  }
  if (object == null) {
    FactoryBean<?> factory = (FactoryBean<?>) beanInstance;
    if (mbd == null && containsBeanDefinition(beanName)) {
      mbd = getMergedLocalBeanDefinition(beanName);
    }
    boolean synthetic = (mbd != null && mbd.isSynthetic());
   // 核心
    object = getObjectFromFactoryBean(factory, beanName, !synthetic);
  }
  return object;
}
```

进入getObjectFromFactoryBean()方法，再进入

```java
private Object doGetObjectFromFactoryBean(FactoryBean<?> factory, String beanName) throws BeanCreationException {
  Object object;
  try {
    if (System.getSecurityManager() != null) {
      AccessControlContext acc = getAccessControlContext();
      try {
        object = AccessController.doPrivileged((PrivilegedExceptionAction<Object>) factory::getObject, acc);
      }
      catch (PrivilegedActionException pae) {
        throw pae.getException();
      }
    }
    else {
      object = factory.getObject(); //这个factory是传入的beanInstance实例，即获取到的是实现FactoryBean接口定义的getObject方法
    }
  }
  
  // 省略。。。
  return object;
}
```



#### 应用场景

---

==FactoryBean是用来创建比较复杂的bean==，一般的bean直接用xml配置即可，但如果一个bean的创建过程中涉及到很多其他的bean和复杂的逻辑，用xml配置比较困难，这时可以考虑用FactoryBean。

很多开源项目在集成Spring时都使用到FactoryBean，比如Mybatis3提供mybatis-spring项目中的`org.mybatis.spring.SqlSessionFactoryBean`：

```xml
<bean id="tradeSqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="trade" />
  <property name="mapperLocations" value="classpath*:mapper/trade/*Mapper.xml" />
  <property name="configLocation" value="classpath:mybatis-config.xml" />
  <property name="typeAliasesPackage" value="com.bytebeats.mybatis3.domain.trade" />
</bean>
```

org.mybatis.spring.SqlSessionFactoryBean如下：

```java
public class SqlSessionFactoryBean
  implements FactoryBean<SqlSessionFactory>, InitializingBean, ApplicationListener<ApplicationEvent> {

  ......
}
```

另外，阿里开源的分布式服务框架Dubbo中的Consumer也使用到了FactoryBean：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
                           http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd
                           ">

  <!-- 当前应用信息配置 -->
  <dubbo:application name="demo-consumer" />

  <!-- 暴露服务协议配置 -->
  <dubbo:protocol name="dubbo" port="20813" />    

  <!-- 暴露服务配置 -->
  <dubbo:reference id="demoService" interface="com.alibaba.dubbo.config.spring.api.DemoService"  />

</beans>
```

`<dubbo:reference>`对应的Bean是`com.alibaba.dubbo.config.spring.ReferenceBean`类，如下：

```java
public class ReferenceConfig<T> extends ReferenceConfigBase<T> {
	......
}
```



#### 实践

---

当前微服务日趋流行，项目开发中不可避免的需要去调用一些其他系统的接口，这个时候选择一个合适的HTTP client library就很关键了。OkHttp本身的API已经非常好用了，结合Spring以及在其他项目中复用的目的，对OkHttp创建过程做了一些封装，例如超时时间、连接大小、http代理等。

```xml
<dependency>
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>4.9.1</version>
</dependency>
```

首先，是`OkHttpClientFactoryBean`，代码如下：

```java
public class OkHttpClientFactoryBean implements FactoryBean, DisposableBean {

  private int connectTimeout;
  private int readTimeout;
  private int writeTimeout;

  /**
     * http proxy config
     **/
  private String host;
  private int port;
  private String username;
  private String password;

  /**
     * OkHttpClient instance
     **/
  private OkHttpClient client;


  @Override
  public void destroy() throws Exception {
    if (client != null) {
      client.connectionPool().evictAll();
      client.dispatcher().executorService().shutdown();
      client.cache().close();
      client = null;
    }
  }

  @Override
  public Object getObject() throws Exception {
    ConnectionPool pool = new ConnectionPool(5, 10, TimeUnit.SECONDS);

    OkHttpClient.Builder builder = new OkHttpClient.Builder();
    builder.connectTimeout(connectTimeout, TimeUnit.MICROSECONDS)
      .readTimeout(readTimeout, TimeUnit.MILLISECONDS)
      .writeTimeout(writeTimeout, TimeUnit.MILLISECONDS)
      .connectionPool(pool);

    if (StringUtils.isNotBlank(host) && port > 0) {
      Proxy proxy = new Proxy(java.net.Proxy.Type.HTTP, new InetSocketAddress(host, port));
      builder.proxy(proxy);
    }

    if (StringUtils.isNotBlank(username) && StringUtils.isNotBlank(password)) {
      Authenticator proxyAuthenticator = new Authenticator() {
        @Override
        public Request authenticate(Route route, Response response) throws IOException {
          String credential = Credentials.basic(username, password);
          return response.request().newBuilder()
            .header("Proxy-Authorization", credential)
            .build();
        }
      };
      builder.proxyAuthenticator(proxyAuthenticator);
    }

    client = builder.build();
    return client;
  }

  @Override
  public Class<?> getObjectType() {
    return OkHttpClient.class;
  }

  @Override
  public boolean isSingleton() {
    return true;
  }
  
  // 省略getter，setter方法
}

```

测试：

```java
OkHttpClient okHttpClient = (OkHttpClient) applicationContext.getBean("okHttpClientFactoryBean");
```





































