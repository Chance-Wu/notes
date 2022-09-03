### 一、介绍

---

数据源模块位于`org.apache.ibatis.datasource`包下：

```
org.apache.ibatis.datasource
		.jndi
				JndiDataSourceFactory.java
		.pooled
				PooledConnection.java
				PooledDataSource.java
				PooledDataSourceFactory.java
				PoolState.java
		.unpooled
				UnpooledDataSource.java
				UnpooledDataSourceFactory.java
		DataSourceException.java
		DataSourceFactory.java
```

数据源模块采用了工厂模式。



### 二、数据源工厂接口DataSourceFactory

---

数据源接口沿用JDK中给出的javax.sql包下的`DataSource`。

```java
public interface DataSource  extends CommonDataSource, Wrapper {
  Connection getConnection() throws SQLException;
  Connection getConnection(String username, String password)
    throws SQLException;
}
```

定义了两个获取数据库连接Connection的方法，可见数据源的目的就是为了对外提供数据库连接的获取接口。

数据源工厂接口如下：

```java
public interface DataSourceFactory {

  // 设置属性，需要在构建Configuration配置类时由XMLConfigBuilder来进行调用
  // 将配置文件中的数据源属性填充到DataSource中（再填充到Environment，再填充到Configuration中
  void setProperties(Properties props);

  // 获取DataSource实例
  DataSource getDataSource();
}
```

```xml
<dataSource type="POOLED">
  <property name="driver" value="com.mysql.jdbc.Driver"/>
  <property name="url" value="jdbc:mysql://localhost:3306/mbtest"/>
  <property name="username" value="root"/>
  <property name="password" value="123456"/>
</dataSource>
```

这一段配置内容中`<dataSource>`标签编码这是数据源配置信息，type属性表示这个数据源是POOLED类型的数据源（对应`PooledDataSourceFactory.class`），然后在其内部设置property子标签用于指定数据源具体配置信息。

>Connection与DataSource的关系
>
>Connection（连接）是包含在DataSource（数据源）之内的，我们可以通过DataSource来得到其中的Connection，所以需要==先创建DataSource实例，以此来获取Connection数据库连接==，DataSource是基础，Connection是目的。



### 三、Mybatis数据源类型

---

- `unpooled`：非池型数据源，作为基础存在，一般不会直接使用。
- `pooled`：池型数据源，常用，以非池行数据源为基础。
- `jndi`：托管型，采用外部的数据源。

#### 3.1 非池型数据源（UNPOOLED——UnpooledDataSourceFactory.class）

非池型数据源工厂：

```java
public class UnpooledDataSourceFactory implements DataSourceFactory {

  private static final String DRIVER_PROPERTY_PREFIX = "driver.";
  private static final int DRIVER_PROPERTY_PREFIX_LENGTH = DRIVER_PROPERTY_PREFIX.length();

  protected DataSource dataSource;

  // 无参构造器
  public UnpooledDataSourceFactory() {
    this.dataSource = new UnpooledDataSource();
  }

  // 设置属性（重点）

  @Override
  public void setProperties(Properties properties) {
    // 创建局部变量，用于存放参数属性列表中经过过滤的属性
    Properties driverProperties = new Properties();
    // 调用Mybatis提供的元对象MetaObject来生成针对dataSource实例的元实例metaDataSource
    // 元实例：是针对具体实例的一批装饰器，在包装器核心是具体实例，外围是多个包装器，用于增强功能。
    MetaObject metaDataSource = SystemMetaObject.forObject(dataSource);
    // 遍历获取属性列表Properties
    for (Object key : properties.keySet()) {
      String propertyName = (String) key;
      // 属性名为driver.为前缀的属性，获取其值，保存在driverProperties中以备用
      if (propertyName.startsWith(DRIVER_PROPERTY_PREFIX)) {
        String value = properties.getProperty(propertyName);
        driverProperties.setProperty(propertyName.substring(DRIVER_PROPERTY_PREFIX_LENGTH), value);
        // 属性名在dataSource实例中存在set方法的属性，获取其值，经过转化后保存到metaDataSource元实例中
        // 配置文件中的driver、url、username、password都是以此种方式保存到dataSource实例
      } else if (metaDataSource.hasSetter(propertyName)) {
        String value = (String) properties.get(propertyName);
        Object convertedValue = convertValue(metaDataSource, propertyName, value);
        metaDataSource.setValue(propertyName, convertedValue);
      } else {
        throw new DataSourceException("Unknown DataSource property: " + propertyName);
      }
    }
    // 对driverProperties进行null判断，即判断有无通过前缀方式配置的属性，如果有则将这些配置同样保存到metaDataSource元实例中
    // 这样到最后其实所有的配置信息都保存到了metaDataSource元实例中
    // 其实最后通过MetaObject的setValue()方法，将所有这些经过过滤的属性设置保存到了元实例的核心：dataSource中了
    if (driverProperties.size() > 0) {
      metaDataSource.setValue("driverProperties", driverProperties);
    }
  }

  // 获取之前在构造器中创建的数据源实例
  @Override
  public DataSource getDataSource() {
    return dataSource;
  }
}
```

非池型数据源`UnpooledDataSource`：保持一个数据库连接实例的数据源

UnpooledDataSource实现了DataSource接口，实现了其中的所有抽象方法。类中字段如下：

```java
// 数据库的驱动类加载器：用于从磁盘加载数据库驱动类到内存
private ClassLoader driverClassLoader;
// 驱动器属性，保存手动设置的数据库驱动器属性，属性一般以driver.为前缀
private Properties driverProperties;
// 数据库驱动注册器，内部保存着所有已注册的数据库驱动类实例
private static Map<String, Driver> registeredDrivers = new ConcurrentHashMap<>();

private String driver;
private String url;
private String username;
private String password;

// 是否自动提交
private Boolean autoCommit;
// 默认事务级别
private Integer defaultTransactionIsolationLevel;
// 默认网络超时
private Integer defaultNetworkTimeout;
```

静态代码块：

```java
static {
  Enumeration<Driver> drivers = DriverManager.getDrivers();
  while (drivers.hasMoreElements()) {
    Driver driver = drivers.nextElement();
    registeredDrivers.put(driver.getClass().getName(), driver);
  }
}
```

DriverManager是JDK提供的驱动器管理类，在该类加载时会执行其中的静态块，静态块中调用了一个loadInitialDrivers()方法，这是一个加载原始驱动器的方法，**将JDK中的原始配置jdbc.drivers中的驱动器名表示的驱动类加载到内存**，其会调用本地方法进行驱动类进行加载，然后调用DriverManager中的registerDriver()方法**将驱动类实例一一保存到DriverManager中的registeredDrivers集合中**。

我们这里调用DriverManager中getDrivers()方法，将会获取DriverManager中在集合registeredDrivers中的保存的驱动器实例，在获取的时候会进行类加载验证，验证的目的是确保使用本类加载器获取的驱动器类与在registeredDrivers中保存的对应驱动类实例的类是同一类型（==）。最后获取到的是驱动实例的枚举。

对这个枚举进行遍历，将其中所有驱动器实例保存到我们当前的UnpooledDataSource中的静态集合registeredDrivers（区别于DriverManager中的同名集合，二者类型都不同）中。

这样保证在该类加载的时候就将默认的驱动器实例加载到静态集合中以备用。该静态代码块的作用就是为registeredDrivers集合赋值。

最重要的方法：

```java
@Override
public Connection getConnection() throws SQLException {
  return doGetConnection(username, password);
}

@Override
public Connection getConnection(String username, String password) throws SQLException {
  return doGetConnection(username, password);
}
```

这是两个获取数据源连接的方法，二者内部都调用了同一个方法`doGetConnection()`，这是真正执行数据库连接并获取这个连接的方法。

```java
private Connection doGetConnection(String username, String password) throws SQLException {
  Properties props = new Properties();
  if (driverProperties != null) {
    props.putAll(driverProperties);
  }
  if (username != null) {
    props.setProperty("user", username);
  }
  if (password != null) {
    props.setProperty("password", password);
  }
  return doGetConnection(props);
}
```

此时props里保存的是有关数据源连接的基础信息。

调用重载的同名方法：

```java
private Connection doGetConnection(Properties properties) throws SQLException {
  initializeDriver();
  Connection connection = DriverManager.getConnection(url, properties);
  configureConnection(connection);
  return connection;
}
```

首先调用`initializeDriver()`方法进行驱动器初始化：

```java
private synchronized void initializeDriver() throws SQLException {
  if (!registeredDrivers.containsKey(driver)) {
    Class<?> driverType;
    try {
      if (driverClassLoader != null) {
        driverType = Class.forName(driver, true, driverClassLoader);
      } else {
        driverType = Resources.classForName(driver);
      }
      Driver driverInstance = (Driver) driverType.getDeclaredConstructor().newInstance();
      DriverManager.registerDriver(new DriverProxy(driverInstance));
      registeredDrivers.put(driver, driverInstance);
    } catch (Exception e) {
      throw new SQLException("Error setting driver on UnpooledDataSource. Cause: " + e);
    }
  }
}
```

DriverProxy，这是一个驱动代理，这个类是以静态代理类的方式定义的，其实现了Driver接口，实现了Driver接口中的所有抽象方法，是一个真正的代理类，代理Driver真正的实现类，即真正起作用的驱动类实例，代理类将驱动类的所有方法全部保护起来。

接着通过调用DriverManager的getConnection方法来获取数据库连接connection，最后调用configureConnection()方法进行数据库连接的最后配置，配置的内容如下：

```java
private void configureConnection(Connection conn) throws SQLException {
  if (defaultNetworkTimeout != null) {
    conn.setNetworkTimeout(Executors.newSingleThreadExecutor(), defaultNetworkTimeout);
  }
  if (autoCommit != null && autoCommit != conn.getAutoCommit()) {
    conn.setAutoCommit(autoCommit);
  }
  if (defaultTransactionIsolationLevel != null) {
    conn.setTransactionIsolation(defaultTransactionIsolationLevel);
  }
}
```

> 配置内容为：自动提交，事务级别

#### 3.2 池型数据源（POOLED——PooledDataSourceFactory.class）

Java项目中多采用池型数据源，C3P0，DBCP之类的也都提供了池型数据源，在MyBatis中也自定义了一种池型数据源`PooledDataSource`。

池型数据源工厂：

```java
public class PooledDataSourceFactory extends UnpooledDataSourceFactory {

  // 获取数据源实例
  public PooledDataSourceFactory() {
    this.dataSource = new PooledDataSource();
  }

}
```

该类继承了UnPooledDataSourceFactory工厂类，即其也用于非池型数据源工厂类中的方法，最主要继承的功能就是`setProperties(Properties properties)`的功能，**将被读取到内存中的数据源配置信息设置到数据源实例中**。

辅助类：`PooledConnection`

```java
class PooledConnection implements InvocationHandler {
```

这是一个JDK动态代理，说明**PooledConnection池连接就是为了创建一个真实连接的代理**，使用连接代理来调用真实连接进行其他操作。

同时这个代理对连接的功能进行了扩充，将其转化为池型连接，即池型的概念与实现有一部分就在这个类中进行，**PooledConnection类将一个普通的Connection包装成为一个池型化的连接**，使其适用于MyBatis自定义的池型数据源。

```java
// 以下为池型连接的属性
private static final String CLOSE = "close";
private static final Class<?>[] IFACES = new Class<?>[] { Connection.class };

private final int hashCode;
// 池型数据源，为了方便调用其内部定义的部分方法来辅助完成池型连接的一些功能判断
private final PooledDataSource dataSource;
// 真正的连接，属于被代理的对象
private final Connection realConnection;
// 代理的连接
private final Connection proxyConnection;
// 数据库连接被检出的时间戳
private long checkoutTimestamp;
// 数据库连接被创建的时间戳
private long createdTimestamp;
// 连接最后被使用的时间戳
private long lastUsedTimestamp;
// 数据库连接的类型编码，格式：url+username+password
private int connectionTypeCode;
// 连接是否可用的逻辑值
private boolean valid;
```

构造器：

```java
public PooledConnection(Connection connection, PooledDataSource dataSource) {
  this.hashCode = connection.hashCode();
  this.realConnection = connection;
  this.dataSource = dataSource;
  this.createdTimestamp = System.currentTimeMillis();
  this.lastUsedTimestamp = System.currentTimeMillis();
  this.valid = true;
  // 调用Proxy的newProxyInstance()方法来生成代理连接实例
  this.proxyConnection = (Connection) Proxy.newProxyInstance(Connection.class.getClassLoader(), IFACES, this);
}
```

代理中最重要的方法：

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  String methodName = method.getName();
  // 如果调用close的话，忽略它，反而将这个connection加入到池中
  if (CLOSE.equals(methodName)) {
    dataSource.pushConnection(this);
    return null;
  }
  try {
    if (!Object.class.equals(method.getDeclaringClass())) {
      // 除了toString()方法，其他方法调用之前要检查connection是否合法的，不合法要抛出SQLException
      checkConnection();
    }
    // 其他的方法，则交给真正的connection去调用
    return method.invoke(realConnection, args);
  } catch (Throwable t) {
    throw ExceptionUtil.unwrapThrowable(t);
  }

}
```

- 首先获取要调用方法的名称methodName，然后对这个名称进行验证，如果是close方法的话，那么忽略关闭的实现，转而将这个连接推入连接池中（`pushConnection`），这是因为在池型数据源中，当连接不再使用后是要返回池中备用的，而不是直接被关闭销毁。
- 如果调用的方法是不是close时，则首先进行该方法申明处的判断，如果这个方法不是来自Object类（剔除toString()方法），那么对当前的连接的可用性进行判断，如果不可用（valid值为false），则抛出SqlException异常，否则继续执行下一步，由真实连接进行方法调用。
- 总得来说就是实现方法调用（这也是代理的目的所在），外面看起来是是由代理类执行方法，其实内部是由真实连接类来执行方法。

池状态类：`PoolState`

用于描述连接池的状态属性。这个属性不同于池连接的属性，这是属于连接池的属性，作用于整个连接池，而连接池中包含有限个连接，一定要明白其中的关系。

在这个类中定义了许多属性，多用于统计信息的输出。

```java
public class PoolState {

  protected PooledDataSource dataSource;

  // 空闲的连接
  protected final List<PooledConnection> idleConnections = new ArrayList<>();
  // 活动的连接
  protected final List<PooledConnection> activeConnections = new ArrayList<>();

  /*--------------------以下是一些统计信息--------------------------*/
  // 请求次数
  protected long requestCount = 0;
  // 总请求时间
  protected long accumulatedRequestTime = 0;
  protected long accumulatedCheckoutTime = 0;
  protected long claimedOverdueConnectionCount = 0;
  protected long accumulatedCheckoutTimeOfOverdueConnections = 0;
  // 总等待时间
  protected long accumulatedWaitTime = 0;
  // 要等待的次数
  protected long hadToWaitCount = 0;
  // 坏的连接次数
  protected long badConnectionCount = 0;

  // 构造器
  public PoolState(PooledDataSource dataSource) {
    this.dataSource = dataSource;
  }
}
```

池型数据源：`PooledDataSource`

同步的线程安全的数据库连接池

- **获取池连接（推出池连接）**
- *收回池连接（推入池连接）**
- 关闭池连接
- 池连接的可用性判断

```java
public class PooledDataSource implements DataSource {

  private static final Log log = LogFactory.getLog(PooledDataSource.class);

  // 池状态
  private final PoolState state = new PoolState(this);

  // 有一个非池型数据源
  private final UnpooledDataSource dataSource;

  // 可选配置字段
  protected int poolMaximumActiveConnections = 10;
  protected int poolMaximumIdleConnections = 5;
  protected int poolMaximumCheckoutTime = 20000;
  protected int poolTimeToWait = 20000;

  protected int poolMaximumLocalBadConnectionTolerance = 3;
  // 发送到数据的侦测查询，用来验证连接是否正常工作，并且准备接受请求。默认是"NO PING QUERY SET"，这会引起许多数据
  protected String poolPingQuery = "NO PING QUERY SET";
  // 开启或禁用侦测查询
  protected boolean poolPingEnabled;
  // 用来配置poolPingQuery 多次时间被用一次
  protected int poolPingConnectionsNotUsedFor;

  private int expectedConnectionTypeCode;
}
```

- `int poolMaximumActiveConnections = 10`：连接池中**最多可拥有的活动连接数**，池中保存的活动连接数不能超过这个值（10个），当要超过时，在没有空闲连接的基础下，不能再新建连接，而是从活动链接中取最老的那个连接进行使用。（这个发生在推出连接时）
- `int poolMaximumIdleConnections = 5`：连接池中**最多可拥有的空闲连接数**，池中保存的空闲连接数不能超过这个值（5个），当要超过时，将多出的真实连接直接关闭，池连接置为无效。（这个发生在推入连接时）
- `int poolMaximumCheckoutTime = 20000`：连接池最大检出时间（可以理解为**验证时间**），如果一个连接验证时间超过设定值，则将这个连接设置为过期（发生在推出连接时）
- `int poolTimeToWait = 20000`：池等待时间，当需要从池中获取一个连接时，如果空闲连接数量为0，而活动连接的数量也达到了最大值，那么就针对那个最早取出的连接进行检查验证（check out），如果验证成功（即在上面**poolMaximumCheckoutTime**限定的时间内验证通过），说明这个连接还处于使用状态，这时取出操作暂停，线程等待限定时间，这个限定时间就是这个参数的使用位置。
- `String poolPingQuery = "NO PING QUERY SET"`：在**验证连接是否有效**的时候，对数据库执行查询，查询内容为该设置内容。整个目的就是为了得知这个数据库连接还是否能够使用（未关闭，并处于正常状态），这是一个侦测查询。
- `boolean poolPingEnabled = false`：这是一个开关，表示**是否打开侦测查询功能**，默认为false，表示关闭该功能。
- `int poolPingConnectionsNotUsedFor = 0`：如果一个连接在限定的时间内一直未被使用，那么就要对该连接进行验证，以确定这个连接是否处于可用状态（即进行侦测查询），这个限定的时间就使用**poolPingConnectionsNotUsedFor**来设定，默认值为0。

```java
// 构造器
public PooledDataSource() {
  dataSource = new UnpooledDataSource();
}

public PooledDataSource(UnpooledDataSource dataSource) {
  this.dataSource = dataSource;
}

public PooledDataSource(String driver, String url, String username, String password) {
  dataSource = new UnpooledDataSource(driver, url, username, password);
  expectedConnectionTypeCode = assembleConnectionTypeCode(dataSource.getUrl(), dataSource.getUsername(), dataSource.getPassword());
}

public PooledDataSource(String driver, String url, Properties driverProperties) {
  dataSource = new UnpooledDataSource(driver, url, driverProperties);
  expectedConnectionTypeCode = assembleConnectionTypeCode(dataSource.getUrl(), dataSource.getUsername(), dataSource.getPassword());
}

public PooledDataSource(ClassLoader driverClassLoader, String driver, String url, String username, String password) {
  dataSource = new UnpooledDataSource(driverClassLoader, driver, url, username, password);
  expectedConnectionTypeCode = assembleConnectionTypeCode(dataSource.getUrl(), dataSource.getUsername(), dataSource.getPassword());
}

public PooledDataSource(ClassLoader driverClassLoader, String driver, String url, Properties driverProperties) {
  dataSource = new UnpooledDataSource(driverClassLoader, driver, url, driverProperties);
  expectedConnectionTypeCode = assembleConnectionTypeCode(dataSource.getUrl(), dataSource.getUsername(), dataSource.getPassword());
}
```

池型数据源的根本还是非池型数据源，池型就是在非池型的基础上加上池型的概念与实现。

在创建池型数据源实例的时候首先会创建一个非池型数据源的实例并将其赋值给参数。五种构造器也是根据非池型数据源的五种构造器而来，一一对应，只是在后四个构造器中增加了连接类型编码的组装。这个连接类型编码适用于区分连接种类的。

```java
// 获取连接的方法中调用popConnection()方法
@Override
public Connection getConnection() throws SQLException {
  return popConnection(dataSource.getUsername(), dataSource.getPassword()).getProxyConnection();
}

@Override
public Connection getConnection(String username, String password) throws SQLException {
  return popConnection(username, password).getProxyConnection();
}

private PooledConnection popConnection(String username, String password) throws SQLException {
  boolean countedWait = false;
  PooledConnection conn = null;
  long t = System.currentTimeMillis();
  int localBadConnectionCount = 0;

  // 最外面是while死循环，如果一直拿不到connection，
  while (conn == null) {
    synchronized (state) {
      // 连接池中如果有可用的连接的话
      if (!state.idleConnections.isEmpty()) {
        // 删除空闲列表里第一个，返回
        conn = state.idleConnections.remove(0);
      } else {
        // 连接池中如果没有空闲连接的话
        // 如果activeConnections太少，那就new一个PooledConnection
        if (state.activeConnections.size() < poolMaximumActiveConnections) {
          conn = new PooledConnection(dataSource.getConnection(), this);
        } else {
          // 如果activeConnections太多，那就不能再new了
          // 取得activeConnection列表的第一个（最老的）连接
          PooledConnection oldestActiveConnection = state.activeConnections.get(0);
          long longestCheckoutTime = oldestActiveConnection.getCheckoutTime();
          // 如果检出时间过长，则这个connection标记为overdue（过期）
          if (longestCheckoutTime > poolMaximumCheckoutTime) {
            // 将连接置为过期连接
            state.claimedOverdueConnectionCount++;
            state.accumulatedCheckoutTimeOfOverdueConnections += longestCheckoutTime;
            state.accumulatedCheckoutTime += longestCheckoutTime;
            state.activeConnections.remove(oldestActiveConnection);
            if (!oldestActiveConnection.getRealConnection().getAutoCommit()) {
              try {
                oldestActiveConnection.getRealConnection().rollback();
              } catch (SQLException e) {
              }
              // 删掉最老的连接，然后再new一个新连接
              conn = new PooledConnection(oldestActiveConnection.getRealConnection(), this);
              conn.setCreatedTimestamp(oldestActiveConnection.getCreatedTimestamp());
              conn.setLastUsedTimestamp(oldestActiveConnection.getLastUsedTimestamp());
              oldestActiveConnection.invalidate();
            } else {
              // 如果检出时间不长，则等待
              try {
                if (!countedWait) {
                  // 统计信息：等待+1
                  state.hadToWaitCount++;
                  countedWait = true;
                }
                long wt = System.currentTimeMillis();
                // wait一段时间
                state.wait(poolTimeToWait);
                state.accumulatedWaitTime += System.currentTimeMillis() - wt;
              } catch (InterruptedException e) {
                break;
              }
            }
          }
        }
        // 如果已经拿到连接，则返回
        if (conn != null) {
          // ping服务器并检查连接是否有效
          if (conn.isValid()) {
            if (!conn.getRealConnection().getAutoCommit()) {
              conn.getRealConnection().rollback();
            }
            conn.setConnectionTypeCode(assembleConnectionTypeCode(dataSource.getUrl(), username, password));
            // 记录检出时间
            conn.setCheckoutTimestamp(System.currentTimeMillis());
            conn.setLastUsedTimestamp(System.currentTimeMillis());
            state.activeConnections.add(conn);
            state.requestCount++;
            state.accumulatedRequestTime += System.currentTimeMillis() - t;
          } else { // 如果没拿到连接
            // 统计信息：坏连接+1
            state.badConnectionCount++;
            localBadConnectionCount++;
            conn = null;
            // 如果好几次都拿不到，就放弃了，抛出异常
            if (localBadConnectionCount > (poolMaximumIdleConnections + poolMaximumLocalBadConnectionTolerance)) {
              throw new SQLException("PooledDataSource: Could not get a good connection to the database.");
            }
          }
        }
      }

    }

    if (conn == null) {
      throw new SQLException("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");
    }

    return conn;
  }
```

1. 这是个同步方法，线程安全。但是将synchronized锁同步放置到循环内部，而不是循环之外的原因是因为：如果将同步锁放置在循环之外，当多个线程执行到锁的位置，其中一个线程获得锁然后开始执行循环，如果发生问题导致无限循环，那么这个锁将是一直被这个线程所持有，导致其他线程永久处于等待锁的状态，程序无法执行下去。而将锁放置到循环内部，当多个线程来到锁之前，其中一个线程获得锁，执行循环内部代码，当执行完成一次循环，无论成功失败，都会释放锁，而其他线程就可以获得锁进而执行。
2. 首先验证空闲连接集合是否为空（验证是否还有空闲连接备用），如果存在空闲连接，那么直接获取这个空闲连接，将这个连接从空闲连接集合中删除。
3. 如果没有空闲连接，那么就验证活动连接集合中连接的数量是否达到最大值（poolMaximumActiveConnections），如果未达到最大值，这时，我们可以直接创建一个新的池型连接（需要一个真实连接于与一个池型数据源实例作为参数）
4. 如果活动连接集合中的连接数目已经达到最大值（poolMaximumActiveConnections），那么就针对最早的那个活动连接（即在集合中排在0位的那个连接实例）进行验证。并获取其验证时间间隔值（该连接上一次记录验证时间戳到当前时间的间隔），将其与池连接的最大验证时限（poolMaximumCheckoutTime）进行比较，如果前者大，说明针对这个连接距上一次记录验证时间戳的时间超过了限定时限，这时将这个老连接从活动连接集合中删除，并新建一个池连接，还以老连接所代理的真实连接为真实连接（实际上就是创建一个新的代理），并将老的池连接设为无效。
5. 如果验证时间与显示时间比较结果为验证时间小于限定时限（这个限定时限的设置需要根据项目实际情况来设置，或通过经验来设置，确保在这个时间之内连接的数据库操作执行完毕，不然贸然将连接关闭会导致原本的数据库操作失败），说明这个连接还可能处于使用状态，这时候只有等待一途，这里将线程设置等待限定秒数(poolTimeToWait)，线程进入等待状态，那么就会释放同步锁，此时其他线程就能获得锁来进行执行。当前线程在等待N秒之后自动进入准备状态准备重新获得锁。
6. 然后就获得的连接进行判断，如果连接不为空，那么验证连接是否可用（isValid），如果连接可用则设置连接类型编码，并记录验证时间戳（setCheckoutTimestamp）与最后一次使用时间戳(setLastUsedTimestamp)，这两个时间戳可用于计算该连接的验证时间与最后一次使用时间，在前面会使用到这些值进行判断。再然后将该链接添加到活动连接集合中。
7. 如果获取的连接为空，或者说没有获取到连接，则坏连接数加1，将连接置null，并验证坏连接数值，如果比当前空闲连接数量+3都大的话，那么就放弃获取连接，并抛出SqlException，（抛出异常也就意味着执行的终止，这个线程将不再执行循环操作）

```java
// 推入连接的方法
protected void pushConnection(PooledConnection conn) throws SQLException {

  synchronized (state) {
    // 先从可用连接中删除此连接
    state.activeConnections.remove(conn);
    if (conn.isValid()) {
      // 如果空闲的连接太少
      if (state.idleConnections.size() < poolMaximumIdleConnections && conn.getConnectionTypeCode() == expectedConnectionTypeCode) {
        state.accumulatedCheckoutTime += conn.getCheckoutTime();
        if (!conn.getRealConnection().getAutoCommit()) {
          conn.getRealConnection().rollback();
        }
        // new一个新的连接，加入到idle列表
        PooledConnection newConn = new PooledConnection(conn.getRealConnection(), this);
        state.idleConnections.add(newConn);
        newConn.setCreatedTimestamp(conn.getCreatedTimestamp());
        newConn.setLastUsedTimestamp(conn.getLastUsedTimestamp());
        conn.invalidate();
        // 通知其他线程可以来抢connection了
        state.notifyAll();
      } else {
        // 空闲连接已经足够了
        state.accumulatedCheckoutTime += conn.getCheckoutTime();
        if (!conn.getRealConnection().getAutoCommit()) {
          conn.getRealConnection().rollback();
        }
        // 那就将connection关闭就可以了
        conn.getRealConnection().close();
        conn.invalidate();
      }
    } else {
      state.badConnectionCount++;
    }
  }
}
```

8. 这个方法同样是一个同步方法，拥有同步锁，以池状态实例为锁。
9. 首先我们将当前要推入的连接实例从活动连接中删除，表示其不再处于使用状态。
10. 然后对连接额可用性进行（valid）判断，如果还处于可用状态，则验证空闲连接集合中的空闲连接数量是否小于设置的限定值（poolMaximumIdleConnections）和当前连接实例的类型编码是否与当前池型数据源中的连接类型编码一致，如果上面两点都满足，则进行下一步：
11. 新建一个池型连接实例并将其添加到空闲连接集合中，这个池型连接实例是以之前要推入的连接为基础重新创建的，也就是说是针对那个要推入的池型连接的真实连接重新创建一个池型连接代理（只改变外包装，实质不改变），并将原池型连接的时间戳设置统统设置到新的连接中，保持连接的持续性，然后将原池型连接置为无效。
12. 然后唤醒所有沉睡线程notifyAll()。
13. 如果第(3)点中的判断中有一个不成立（空闲连接数量达到最大值或者连接的类型编码不一致）那么直接将该连接的真实连接关闭，池连接置为无效即可。

```java
// 验证某一个连接是否仍然可用，它会被PooledConnection类中的isValid()方法调用
protected boolean pingConnection(PooledConnection conn) {
  boolean result = true;

  try {
    result = !conn.getRealConnection().isClosed();
  } catch (SQLException e) {
    result = false;
  }

  if (result && poolPingEnabled && poolPingConnectionsNotUsedFor >= 0
      && conn.getTimeElapsedSinceLastUse() > poolPingConnectionsNotUsedFor) {
    try {
      Connection realConn = conn.getRealConnection();
      try (Statement statement = realConn.createStatement()) {
        statement.executeQuery(poolPingQuery).close();
      }
      if (!realConn.getAutoCommit()) {
        realConn.rollback();
      }
      result = true;
    } catch (Exception e) {
      log.warn("Execution of ping query '" + poolPingQuery + "' failed: " + e.getMessage());
      try {
        conn.getRealConnection().close();
      } catch (Exception e2) {
      }
      result = false;
    }
  }
}
return result;
}
```

14. 首先创建一个局部变量result用于保存判断结果，默认为true
15. 然后将当前池型连接包裹的真实连接的开闭状态值的非值赋值给result（当真实连接处于关闭状态时，result值为false，当真实连接处于开启状态时，result值为true），如果赋值过程出现了异常，则直接将result置false
16. 判断result的值，如果result值为true，则判断poolPingEnabled的值，这是侦测查询的开关，如果这个值为true，表示开启侦测查询，那么就可以执行以下内容。
17. 判断poolPingConnectionsNotUsedFor的值是否大于等于0（这个判断的意思是判断是否设置了正确的poolPingConnectionsNotUsedFor值），并且判断该连接的自最后一次使用以来的时间间隔是否大于设定的poolPingConnectionsNotUsedFor值（验证该连接是否到了需要进行侦测查询的时间，如果小于设置时间则不进行侦测查询）
18. 如果上述条件均满足，则进行一次侦测查询，这个侦测查询就是针对这个连接的一个测试查询，看看整个查询过程是否通畅，若通畅（没有任何异常出现），则将result置为true，一旦测试过程出现了异常，则将该连接的真实连接关闭，并将result置为false

#### 3.3 托管型数据源（JNDI--JndiDataSourceFactory.class）

Mybatis中专门提供的一个数据源工厂——`JndiDataSourceFactory`，与事务模块中MANAGED类型的事务一致，也属于托管型，它用于在使用诸如与Spring容器整合的场合，这时为了便于使这些外部定义的数据源整合到Mybatis的环境中，就需要使用这个JNDI数据源工厂来进行获取。

这里只定义了数据源工厂，真正的数据源由外部来提供，这还是纯种的抽象工厂模式。

```java
/**
* 这个数据源的实现是为了使用如Spring或应用服务器这类容器，容器可以集中或在外部配置数据源，然后放置一个JNDI上下文的引用
*/
public class JndiDataSourceFactory implements DataSourceFactory {

  public static final String INITIAL_CONTEXT = "initial_context";
  public static final String DATA_SOURCE = "data_source";
  // 和其他数据源相似，可以通过名为"env."的前缀直接向初始时上下文发送属性
  public static final String ENV_PREFIX = "env.";

  private DataSource dataSource;

  @Override
  public void setProperties(Properties properties) {
    try {
      InitialContext initCtx;
      Properties env = getEnvProperties(properties);
      if (env == null) {
        initCtx = new InitialContext();
      } else {
        initCtx = new InitialContext(env);
      }

      if (properties.containsKey(INITIAL_CONTEXT) && properties.containsKey(DATA_SOURCE)) {
        Context ctx = (Context) initCtx.lookup(properties.getProperty(INITIAL_CONTEXT));
        dataSource = (DataSource) ctx.lookup(properties.getProperty(DATA_SOURCE));
      } else if (properties.containsKey(DATA_SOURCE)) {
        dataSource = (DataSource) initCtx.lookup(properties.getProperty(DATA_SOURCE));
      }

    } catch (NamingException e) {
      throw new DataSourceException("There was an error configuring JndiDataSourceTransactionPool. Cause: " + e, e);
    }
  }

  @Override
  public DataSource getDataSource() {
    return dataSource;
  }

  private static Properties getEnvProperties(Properties allProps) {
    final String PREFIX = ENV_PREFIX;
    Properties contextProperties = null;
    for (Entry<Object, Object> entry : allProps.entrySet()) {
      String key = (String) entry.getKey();
      String value = (String) entry.getValue();
      // 和其他数据源配置相似，可以通过名为"env."的前缀直接向初始时上下文发送属性
      if (key.startsWith(PREFIX)) {
        if (contextProperties == null) {
          contextProperties = new Properties();
        }
        contextProperties.put(key.substring(PREFIX.length()), value);
      }
    }
    return contextProperties;
  }

}
```

- JNDI数据源工厂类定了一个继承自DataSourceFactory的获取数据源的方法`getDataSource()方法`，用于**获取外部创建的数据源实例**。
- 设置属性的方法，与非池型数据源工厂中类似，JNDI数据源也可以通过前缀的方式设置一些数据源的属性来传递到数据源中，用来设置数据源的基本信息。
- JNDI数据源就是通过在外部数据源上覆盖一个上下文，即将数据源添加到某个上下文中，将这个上下文传递到工厂，由工厂从上下文中获取这个数据源，并通过getDataSource()被获取。

针对Mybatis整合的程序开发，只要针对这个工厂来开发就行了。

至此，`数据源模块`与之前解析的`事务模块`为组装`Environment环境`的两个重要的唯二的模块，而Environment又是构建Configuration配置类的首要模块。
