`Environment`——Configuration配置类中重要的配置项。

### 一、介绍

---

当单独使用Mybatis框架时，需要在配置文件中进行环境配置：

```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC"/> <!--事务管理器-->
    <dataSource type="POOLED"> <!--数据源-->
      <property name="driver" value="com.mysql.jdbc.Driver"/>
      <property name="url" value="jdbc:mysql://localhost:3306/mbtest"/>
      <property name="username" value="root"/>
      <property name="password" value="123456"/>
    </dataSource>
  </environment>
</environments>
```

- `<transactionManager type="JDBC"/>`表示事务管理器是JDBC类型了，这个标签对应Configuration类创建时在其无参构造器中注册的类型别名，通过这个类型别名注册信息可以找到实际的JDBC事务工厂类：`JdbcTransactionFactory`。
- `<dataSource type="POOLED">`表示数据源是POOLED类型，通过**类型别名注册信息**同样可以找到对应的注册信息：`PooledDataSourceFactory`。

```java
public Configuration() {
  typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);
  typeAliasRegistry.registerAlias("MANAGED", ManagedTransactionFactory.class);

  typeAliasRegistry.registerAlias("JNDI", JndiDataSourceFactory.class);
  typeAliasRegistry.registerAlias("POOLED", PooledDataSourceFactory.class);
  typeAliasRegistry.registerAlias("UNPOOLED", UnpooledDataSourceFactory.class);

  typeAliasRegistry.registerAlias("PERPETUAL", PerpetualCache.class);
  typeAliasRegistry.registerAlias("FIFO", FifoCache.class);
  typeAliasRegistry.registerAlias("LRU", LruCache.class);
  typeAliasRegistry.registerAlias("SOFT", SoftCache.class);
  typeAliasRegistry.registerAlias("WEAK", WeakCache.class);

  typeAliasRegistry.registerAlias("DB_VENDOR", VendorDatabaseIdProvider.class);

  typeAliasRegistry.registerAlias("XML", XMLLanguageDriver.class);
  typeAliasRegistry.registerAlias("RAW", RawLanguageDriver.class);

  typeAliasRegistry.registerAlias("SLF4J", Slf4jImpl.class);
  typeAliasRegistry.registerAlias("COMMONS_LOGGING", JakartaCommonsLoggingImpl.class);
  typeAliasRegistry.registerAlias("LOG4J", Log4jImpl.class);
  typeAliasRegistry.registerAlias("LOG4J2", Log4j2Impl.class);
  typeAliasRegistry.registerAlias("JDK_LOGGING", Jdk14LoggingImpl.class);
  typeAliasRegistry.registerAlias("STDOUT_LOGGING", StdOutImpl.class);
  typeAliasRegistry.registerAlias("NO_LOGGING", NoLoggingImpl.class);

  typeAliasRegistry.registerAlias("CGLIB", CglibProxyFactory.class);
  typeAliasRegistry.registerAlias("JAVASSIST", JavassistProxyFactory.class);

  languageRegistry.setDefaultDriverClass(XMLLanguageDriver.class);
  languageRegistry.register(RawLanguageDriver.class);
}
```

>构建Configuration之前：
>
>- 第一步是进行类型别名注册；
>- 第二步就是将Environment配置内容从配置文件中读入Environment类中（这一步是在XMLConfigBuilder中完成的），并将其组合到Configuration类中。



### 二、TypeAliasRegistry类（记录别名alias和类class之间的对应关系）

---

简单来说就是Mybatis的类型别名实现机制就是创建了一个`Map<String,Class<?>>`，解析mybatis配置文件，将alias元素的值作为Map的key，通过反射机制获得type元素对应的类名的类作为Map的value值，在真正**使用时通过别名来获取真正的类**。

```java
// 其实就是一个map结构，用来注册key别名和value具体的类
public class TypeAliasRegistry {

  private final Map<String, Class<?>> typeAliases = new HashMap<>();

  // 创建别名注册器对象时调用无参构造器会自动注册常见类的别名
  public TypeAliasRegistry() {
    registerAlias("string", String.class);

    registerAlias("byte", Byte.class);
    registerAlias("long", Long.class);
    registerAlias("short", Short.class);
    registerAlias("int", Integer.class);
    registerAlias("integer", Integer.class);
    registerAlias("double", Double.class);
    registerAlias("float", Float.class);
    registerAlias("boolean", Boolean.class);

    registerAlias("byte[]", Byte[].class);
    registerAlias("long[]", Long[].class);
    registerAlias("short[]", Short[].class);
    registerAlias("int[]", Integer[].class);
    registerAlias("integer[]", Integer[].class);
    registerAlias("double[]", Double[].class);
    registerAlias("float[]", Float[].class);
    registerAlias("boolean[]", Boolean[].class);

    registerAlias("_byte", byte.class);
    registerAlias("_long", long.class);
    registerAlias("_short", short.class);
    registerAlias("_int", int.class);
    registerAlias("_integer", int.class);
    registerAlias("_double", double.class);
    registerAlias("_float", float.class);
    registerAlias("_boolean", boolean.class);

    registerAlias("_byte[]", byte[].class);
    registerAlias("_long[]", long[].class);
    registerAlias("_short[]", short[].class);
    registerAlias("_int[]", int[].class);
    registerAlias("_integer[]", int[].class);
    registerAlias("_double[]", double[].class);
    registerAlias("_float[]", float[].class);
    registerAlias("_boolean[]", boolean[].class);

    registerAlias("date", Date.class);
    registerAlias("decimal", BigDecimal.class);
    registerAlias("bigdecimal", BigDecimal.class);
    registerAlias("biginteger", BigInteger.class);
    registerAlias("object", Object.class);

    registerAlias("date[]", Date[].class);
    registerAlias("decimal[]", BigDecimal[].class);
    registerAlias("bigdecimal[]", BigDecimal[].class);
    registerAlias("biginteger[]", BigInteger[].class);
    registerAlias("object[]", Object[].class);

    registerAlias("map", Map.class);
    registerAlias("hashmap", HashMap.class);
    registerAlias("list", List.class);
    registerAlias("arraylist", ArrayList.class);
    registerAlias("collection", Collection.class);
    registerAlias("iterator", Iterator.class);

    registerAlias("ResultSet", ResultSet.class);
  }

  // 通过别名找到具体的类
  @SuppressWarnings("unchecked")
  public <T> Class<T> resolveAlias(String string) {
    try {
      if (string == null) {
        return null;
      }
      String key = string.toLowerCase(Locale.ENGLISH);
      Class<T> value;
      if (typeAliases.containsKey(key)) {
        value = (Class<T>) typeAliases.get(key);
      } else {
        value = (Class<T>) Resources.classForName(string);
      }
      return value;
    } catch (ClassNotFoundException e) {
      throw new TypeException("Could not resolve type alias '" + string + "'.  Cause: " + e, e);
    }
  }

  // 通过包名注册类
  public void registerAliases(String packageName) {
    registerAliases(packageName, Object.class);
  }

  // 获得包内的类，除去内部类和接口
  public void registerAliases(String packageName, Class<?> superType) {
    ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<>();
    resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
    Set<Class<? extends Class<?>>> typeSet = resolverUtil.getClasses();
    for (Class<?> type : typeSet) {
      if (!type.isAnonymousClass() && !type.isInterface() && !type.isMemberClass()) {
        registerAlias(type);
      }
    }
  }

  // 注册类
  public void registerAlias(Class<?> type) {
    String alias = type.getSimpleName();
    Alias aliasAnnotation = type.getAnnotation(Alias.class);
    if (aliasAnnotation != null) {
      alias = aliasAnnotation.value();
    }
    registerAlias(alias, type);
  }

  // 注册类包括别名和类
  public void registerAlias(String alias, Class<?> value) {
    if (alias == null) {
      throw new TypeException("The parameter alias cannot be null");
    }
    String key = alias.toLowerCase(Locale.ENGLISH);
    if (typeAliases.containsKey(key) && typeAliases.get(key) != null && !typeAliases.get(key).equals(value)) {
      throw new TypeException("The alias '" + alias + "' is already mapped to the value '" + typeAliases.get(key).getName() + "'.");
    }
    typeAliases.put(key, value);
  }

  // 注册类包括别名和类名
  public void registerAlias(String alias, String value) {
    try {
      registerAlias(alias, Resources.classForName(value));
    } catch (ClassNotFoundException e) {
      throw new TypeException("Error registering type alias " + alias + " for " + value + ". Cause: " + e, e);
    }
  }

  // 获取类型别名
  public Map<String, Class<?>> getTypeAliases() {
    return Collections.unmodifiableMap(typeAliases);
  }
}
```



### 三、Environment类

---

`org.apache.ibatis.mapping.Environment`，整个类被final修饰。

```java
/**
 * 环境
 * 决定加载哪种环境（开发环境、生产环境）
 */
public final class Environment {
  private final String id; //环境id
  private final TransactionFactory transactionFactory; //事务工厂
  private final DataSource dataSource; //数据源

  public Environment(String id, TransactionFactory transactionFactory, DataSource dataSource) {
    if (id == null) {
      throw new IllegalArgumentException("Parameter 'id' must not be null");
    }
    if (transactionFactory == null) {
      throw new IllegalArgumentException("Parameter 'transactionFactory' must not be null");
    }
    this.id = id;
    if (dataSource == null) {
      throw new IllegalArgumentException("Parameter 'dataSource' must not be null");
    }
    this.transactionFactory = transactionFactory;
    this.dataSource = dataSource;
  }

  /**
   * 静态内部类Builder
   * 建造者模式
   * new Environment.Builder(id).transactionFactory(xx).dataSource(xx).build();
   */
  public static class Builder {
    private final String id;
    private TransactionFactory transactionFactory;
    private DataSource dataSource;

    public Builder(String id) {
      this.id = id;
    }

    public Builder transactionFactory(TransactionFactory transactionFactory) {
      this.transactionFactory = transactionFactory;
      return this;
    }

    public Builder dataSource(DataSource dataSource) {
      this.dataSource = dataSource;
      return this;
    }

    public String id() {
      return this.id;
    }

    public Environment build() {
      return new Environment(this.id, this.transactionFactory, this.dataSource);
    }

  }
  // 省略get方法
}
```

通过内部类的方式构建，这个Environment对象的创建会在内部类构建方法build()被显式调用时才会在内存中创建，实现了懒加载。这又有点单例模式的意思在内，**虽然Mybatis中可创建多个Environment环境，但是在正式运行时，只会存在一个环境，确实是使用内部类实现了懒加载的单例模式**。

XMLConfigBuilder构建器中解析构建Configuration类时在解析了Configuration.xml配置文件中environment标签的内容之后，这个位置用于将读自于配置文件的配置信息配置到了Environment对象中。

```java
private void parseConfiguration(XNode root) {
  try {
    propertiesElement(root.evalNode("properties"));
    Properties settings = settingsAsProperties(root.evalNode("settings"));
    loadCustomVfs(settings);
    loadCustomLogImpl(settings);
    typeAliasesElement(root.evalNode("typeAliases"));
    pluginElement(root.evalNode("plugins"));
    objectFactoryElement(root.evalNode("objectFactory"));
    objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
    reflectorFactoryElement(root.evalNode("reflectorFactory"));
    settingsElement(settings);
    environmentsElement(root.evalNode("environments"));
    databaseIdProviderElement(root.evalNode("databaseIdProvider"));
    typeHandlerElement(root.evalNode("typeHandlers"));
    mapperElement(root.evalNode("mappers"));
  } catch (Exception e) {
    throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
  }
}

private void environmentsElement(XNode context) throws Exception {
  if (context != null) {
    if (environment == null) {
      environment = context.getStringAttribute("default");
    }
    for (XNode child : context.getChildren()) {
      String id = child.getStringAttribute("id");
      if (isSpecifiedEnvironment(id)) {
        TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"));
        DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));
        DataSource dataSource = dsFactory.getDataSource();
        // 创建者模式创建Environment对象
        Environment.Builder environmentBuilder = new Environment.Builder(id)
          .transactionFactory(txFactory)
          .dataSource(dataSource);
        configuration.setEnvironment(environmentBuilder.build());
      }
    }
  }
}
```

