```
org.apache.ibatis.binding
		BindingException
		MapperMethod
		MapperProxy
		MapperProxyFactory
		MapperRegistry
```

映射器注册器对外，会被Configuration类直接调用，用于将用户自定义的映射器全部注册到注册器中。



### 一、MapperRegistry

---

首先在注册器中会定义一个集合用于保存注册内容：

```java
private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<>();
```

`Class类型`——`MapperProxyFactory类型`

MapperProxyFactory是映射器代理工厂，通过这个工厂类可以获取到对应的映射器代理类MapperProxy，这里只需要保存一个映射器的代理工厂，根据工厂就可以获取到对应的映射器。相应的注册器中必然定义了添加映射器和获取映射器的方法来对外提供服务（供外部调取）。



### 二、addMappers()

---

将给定包名下的所有映射器注册到注册器中。

```java
/**
 * 用于仅指定包名的情况下，扫描包下的每个映射器进行注册
 */
public void addMappers(String packageName) {
  addMappers(packageName, Object.class);
}

/**
 * 将包下满足以superType为超类的映射器注册到注册器中
 */
public void addMappers(String packageName, Class<?> superType) {
  // 查找包下所有是superType的类
  ResolverUtil<Class<?>> resolverUtil = new ResolverUtil<>();
  resolverUtil.find(new ResolverUtil.IsA(superType), packageName);
  Set<Class<? extends Class<?>>> mapperSet = resolverUtil.getClasses();
  for (Class<?> mapperClass : mapperSet) {
    addMapper(mapperClass);
  }
}

/**
 * 将指定类型的映射器添加到注册器中
 */
public <T> void addMapper(Class<T> type) {
  // mapper必须是接口，才会添加
  if (type.isInterface()) {
    if (hasMapper(type)) {
      // 如果重复添加了，报错
      throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
    }
    boolean loadCompleted = false;
    try {
      knownMappers.put(type, new MapperProxyFactory<>(type));
      MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
      parser.parse();
      loadCompleted = true;
    } finally {
      // 如果加载过程中出现异常需要再将这个mapper从mybatis中删除
      if (!loadCompleted) {
        knownMappers.remove(type);
      }
    }
  }
}
```

>核心方法`addMapper(Class<T> type)`：
>
>1. （接口验证）验证要添加的映射器的类型是否是接口，如果不是接口则结束添加，如果是接口则执行下一步；
>2. （重复注册验证）验证注册器集合中是否已存在该注册器，如果已存在则抛出绑定异常，否则执行下一步；
>3. 定义一个boolean值，默认为false；
>4. 执行HashMap的put方法，将该映射器注册到注册器中：**以该接口类型为键，以接口类型为参数调用MapperProxyFactory的构造器创建的映射器代理工厂为值**；
>5. 然后对使用注解方式实现的映射器进行注册（一般不使用）；
>6. 设置第3步的boolean值为true，表示注册完成；
>7. 在finally语句块中对注册失败的类型进行清除。



### 三、getMapper()

---

从集合中获取指定接口类型的映射器代理工厂，然后使用这个代理工厂创建映射器代理实例并返回，那么我们就可以获取到映射器的代理实例：

```java
/**
 * 返回Mapper接口的代理类
 */
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
  final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);
  if (mapperProxyFactory == null) {
    throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
  }
  try {
    return mapperProxyFactory.newInstance(sqlSession);
  } catch (Exception e) {
    throw new BindingException("Error getting mapper instance. Cause: " + e, e);
  }
}
```



### 四、MapperProxyFactory

---

使用这个代理工厂创建映射器代理实例并返回，就可以获取到映射器的代理实例。

```java
public class MapperProxyFactory<T> {

  private final Class<T> mapperInterface;
  private final Map<Method, MapperMethodInvoker> methodCache = new ConcurrentHashMap<>();

  public MapperProxyFactory(Class<T> mapperInterface) {
    this.mapperInterface = mapperInterface;
  }

  public Class<T> getMapperInterface() {
    return mapperInterface;
  }

  public Map<Method, MapperMethodInvoker> getMethodCache() {
    return methodCache;
  }

  @SuppressWarnings("unchecked")
  protected T newInstance(MapperProxy<T> mapperProxy) {
    // （2）用JDK自带的动态代理生成代理映射器
    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);
  }

  public T newInstance(SqlSession sqlSession) {
    // （1）先调用MapperProxy的构造器生成MapperProxy
    final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
  }
}
```

映射方法单独定义，是因为这里并不存在一个真正的类和方法供调用，只是通过反射和代理的原理来实现的假的调用，**映射方法是调用的最小单位（独立个体）**，将映射方法定义之后，它就成为一个实实在在的存在，我们可以将调用过的方法保存到对应的映射器的缓存中，以供下次调用，避免每次调用相同的方法的时候都需要重新进行方法的生成。



### 五、MapperProxy——映射器代理

---

映射器代理类是MapperProxyFactory对应的目标类。

```java
/**
 * 构造器
 * @param sqlSession	session会话，用于指明操作的来源
 * @param mapperInterface	映射器接口，指明操作的目标接口
 * @param methodCache	方法缓存，用于保存具体的操作方法实例
 */
public MapperProxy(SqlSession sqlSession, Class<T> mapperInterface, Map<Method, MapperMethodInvoker> methodCache) {
  this.sqlSession = sqlSession;
  this.mapperInterface = mapperInterface;
  this.methodCache = methodCache;
}
```

```java
/**
 * 映射器代理，代理模式
 */
public class MapperProxy<T> implements InvocationHandler, Serializable {

  private static final long serialVersionUID = -4724728412955527868L;
  private static final int ALLOWED_MODES = MethodHandles.Lookup.PRIVATE | MethodHandles.Lookup.PROTECTED
    | MethodHandles.Lookup.PACKAGE | MethodHandles.Lookup.PUBLIC;
  private static final Constructor<Lookup> lookupConstructor;
  private static final Method privateLookupInMethod;
  private final SqlSession sqlSession;
  private final Class<T> mapperInterface;
  // 定义了一个缓存集合，为了调用MapperProxy的构造器而设，这个缓存集合用于保存当前映射器中的映射方法
  private final Map<Method, MapperMethodInvoker> methodCache;

  static {
    Method privateLookupIn;
    try {
      privateLookupIn = MethodHandles.class.getMethod("privateLookupIn", Class.class, MethodHandles.Lookup.class);
    } catch (NoSuchMethodException e) {
      privateLookupIn = null;
    }
    privateLookupInMethod = privateLookupIn;

    Constructor<Lookup> lookup = null;
    if (privateLookupInMethod == null) {
      // JDK 1.8
      try {
        lookup = MethodHandles.Lookup.class.getDeclaredConstructor(Class.class, int.class);
        lookup.setAccessible(true);
      } catch (NoSuchMethodException e) {
        throw new IllegalStateException(
          "There is neither 'privateLookupIn(Class, Lookup)' nor 'Lookup(Class, int)' method in java.lang.invoke.MethodHandles.",
          e);
      } catch (Exception e) {
        lookup = null;
      }
    }
    lookupConstructor = lookup;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
      // 代理以后，所有Mapper的方法调用时，都会调用这个invoke方法
      // 并不是任何一个方法都需要执行调用代理对象进行执行，如果这个方法是Object中通用的方法（toString等）无需执行
      if (Object.class.equals(method.getDeclaringClass())) {
        return method.invoke(this, args);
      } else {
        // 这里优化了，去缓存中找MapperMethod，执行方法
        return cachedInvoker(method).invoke(proxy, method, args, sqlSession);
      }
    } catch (Throwable t) {
      throw ExceptionUtil.unwrapThrowable(t);
    }
  }

  private MapperMethodInvoker cachedInvoker(Method method) throws Throwable {
    try {
      // 缓存中查找MapperMethod
      MapperMethodInvoker invoker = methodCache.get(method);
      if (invoker != null) {
        return invoker;
      }

      // 缓存中找不到才去创建新的MapperMethod对象
      return methodCache.computeIfAbsent(method, m -> {
        if (m.isDefault()) {
          try {
            if (privateLookupInMethod == null) {
              return new DefaultMethodInvoker(getMethodHandleJava8(method));
            } else {
              return new DefaultMethodInvoker(getMethodHandleJava9(method));
            }
          } catch (IllegalAccessException | InstantiationException | InvocationTargetException
                   | NoSuchMethodException e) {
            throw new RuntimeException(e);
          }
        } else {
          return new PlainMethodInvoker(new MapperMethod(mapperInterface, method, sqlSession.getConfiguration()));
        }
      });
    } catch (RuntimeException re) {
      Throwable cause = re.getCause();
      throw cause == null ? re : cause;
    }
  }

  private MethodHandle getMethodHandleJava9(Method method)
    throws NoSuchMethodException, IllegalAccessException, InvocationTargetException {
    final Class<?> declaringClass = method.getDeclaringClass();
    return ((Lookup) privateLookupInMethod.invoke(null, declaringClass, MethodHandles.lookup())).findSpecial(
      declaringClass, method.getName(), MethodType.methodType(method.getReturnType(), method.getParameterTypes()),
      declaringClass);
  }

  private MethodHandle getMethodHandleJava8(Method method)
    throws IllegalAccessException, InstantiationException, InvocationTargetException {
    final Class<?> declaringClass = method.getDeclaringClass();
    return lookupConstructor.newInstance(declaringClass, ALLOWED_MODES).unreflectSpecial(method, declaringClass);
  }

  interface MapperMethodInvoker {
    Object invoke(Object proxy, Method method, Object[] args, SqlSession sqlSession) throws Throwable;
  }

  private static class PlainMethodInvoker implements MapperMethodInvoker {
    private final MapperMethod mapperMethod;

    public PlainMethodInvoker(MapperMethod mapperMethod) {
      super();
      this.mapperMethod = mapperMethod;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args, SqlSession sqlSession) throws Throwable {
      return mapperMethod.execute(sqlSession, args);
    }
  }

  private static class DefaultMethodInvoker implements MapperMethodInvoker {
    private final MethodHandle methodHandle;

    public DefaultMethodInvoker(MethodHandle methodHandle) {
      super();
      this.methodHandle = methodHandle;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args, SqlSession sqlSession) throws Throwable {
      return methodHandle.bindTo(proxy).invokeWithArguments(args);
    }
  }
}
```

>MapperProxy是使用JDK动态代理实现的代理功能，其重点就在`invoke()方法`中:
>
>- 首先过滤掉Object类的方法；
>- 然后从先从缓存中获取指定的方法；
>- 如果缓存中不存在则新建一个MapperMethod实例并将其保存在缓存中，如果缓存中存在这个指定的方法实例，则直接获取执行。
>
>这里使用缓存进行流程优化，极大的提升了MyBatis的执行速率。



### 六、MapperMethod

---

映射器方法是最底层的被调用者，同时也是binding模块中最复杂的内容。它是Mybatis中对SqlSession会话操作的封装，意味着这个类可以看作是整个Mybatis中数据库操作的枢纽，所有的数据库操作都需要经过它来得以实现。

SqlSession会作为参数在从MapperRegistry中的getMapper()中一直传递到MapperMethod中的execute()方法中，然后在这个方法中进行执行。

```java
// 都是以静态内部内的方式定义
// sql命令
private final SqlCommand command;
// 接口中的方法
private final MethodSignature method;
```

```java
public MapperMethod(Class<?> mapperInterface, Method method, Configuration config) {
  this.command = new SqlCommand(config, mapperInterface, method);
  this.method = new MethodSignature(config, mapperInterface, method);
}
```

>核心方法：
>
>- 对SqlSession的增、删、改操作使用了rowCountResult()方法进行封装，这个方法对SQL操作的返回值类型进行验证检查，保证返回数据的安全。
>- 针对SqlSession的查询操作分多种情况：
>  - 有结果处理器的情况，就不需要本方法进行结果处理，自然有指定的结果处理器来进行处理，所以其result返回值设置为null；
>  - 针对返回多条记录的情况，内部调用SqlSession的`selectList(String statement, Object parameter, RowBounds rowBounds)`方法；
>  - 针对返回Map的情况，内部调用SqlSession的`selectMap(String statement, Object parameter, String mapKey, RowBounds rowBounds)`方法；
>  - 针对返回一条记录的情况：执行SqlSession的`selectOne(String statement, Object parameter)`方法
>- Mybatis提供了`RowBounds`来进行分页设置，在需要分页的情况下，直接将设置好的分页实例传到SqlSession中即可。==这种方式的分页属于内存分页（不适合大数据量）==。==实际会使用分页插件（直接修改SQL脚本来实现查询级的分页）==。

```java
/**
 * 执行（核心方法）
 * 封装了对sqlSession的操作（增删改查）
 */
public Object execute(SqlSession sqlSession, Object[] args) {
  Object result;
  // 4种情况，insert|update|delete|select，分别调用sqlSession的4大类方法
  switch (command.getType()) {
    case INSERT: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.insert(command.getName(), param));
      break;
    }
    case UPDATE: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.update(command.getName(), param));
      break;
    }
    case DELETE: {
      Object param = method.convertArgsToSqlCommandParam(args);
      result = rowCountResult(sqlSession.delete(command.getName(), param));
      break;
    }
    case SELECT:
      // 如果有结果处理器
      if (method.returnsVoid() && method.hasResultHandler()) {
        executeWithResultHandler(sqlSession, args);
        result = null;
      } else if (method.returnsMany()) { //如果结果有多条记录
        result = executeForMany(sqlSession, args);
      } else if (method.returnsMap()) { //如果结果是map
        result = executeForMap(sqlSession, args);
      } else if (method.returnsCursor()) { //如果结果是游标
        result = executeForCursor(sqlSession, args);
      } else { //否则就是一条记录
        Object param = method.convertArgsToSqlCommandParam(args);
        result = sqlSession.selectOne(command.getName(), param);
        if (method.returnsOptional()
            && (result == null || !method.getReturnType().equals(result.getClass()))) {
          result = Optional.ofNullable(result);
        }
      }
      break;
    case FLUSH:
      result = sqlSession.flushStatements();
      break;
    default:
      throw new BindingException("Unknown execution method for: " + command.getName());
  }
  if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
    throw new BindingException("Mapper method '" + command.getName()
                               + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");
  }
  return result;
}
```

静态内部类：

```java
/**
 * SQL命令
 * 
 */
public static class SqlCommand {

  // SQL命令的名称（全限定名.方法名称）
  private final String name;
  // SQL命令的类型（UNKNOWN, INSERT, UPDATE, DELETE, SELECT, FLUSH）
  private final SqlCommandType type;

  public SqlCommand(Configuration configuration, Class<?> mapperInterface, Method method) {
    final String methodName = method.getName();
    final Class<?> declaringClass = method.getDeclaringClass();
    MappedStatement ms = resolveMappedStatement(mapperInterface, methodName, declaringClass,
                                                configuration);
    if (ms == null) {
      if (method.getAnnotation(Flush.class) != null) {
        name = null;
        type = SqlCommandType.FLUSH;
      } else {
        throw new BindingException("Invalid bound statement (not found): "
                                   + mapperInterface.getName() + "." + methodName);
      }
    } else {
      name = ms.getId();
      type = ms.getSqlCommandType();
      if (type == SqlCommandType.UNKNOWN) {
        throw new BindingException("Unknown execution method for: " + name);
      }
    }
  }

  public String getName() {
    return name;
  }

  public SqlCommandType getType() {
    return type;
  }

  private MappedStatement resolveMappedStatement(Class<?> mapperInterface, String methodName,
                                                 Class<?> declaringClass, Configuration configuration) {
    String statementId = mapperInterface.getName() + "." + methodName;
    if (configuration.hasStatement(statementId)) {
      return configuration.getMappedStatement(statementId);
    } else if (mapperInterface.equals(declaringClass)) {
      return null;
    }
    for (Class<?> superInterface : mapperInterface.getInterfaces()) {
      if (declaringClass.isAssignableFrom(superInterface)) {
        MappedStatement ms = resolveMappedStatement(superInterface, methodName,
                                                    declaringClass, configuration);
        if (ms != null) {
          return ms;
        }
      }
    }
    return null;
  }
}
```

```java
public static class MethodSignature {

  // 是否返回多个多条记录
  private final boolean returnsMany;
  // 是否返回Map
  private final boolean returnsMap;
  // 是否返回void
  private final boolean returnsVoid;
  // 是否返回游标
  private final boolean returnsCursor;
  // 返回可选
  private final boolean returnsOptional;
  // 返回类型
  private final Class<?> returnType;
  // @mapKey注解指定的键值
  private final String mapKey;
  // 结果处理器参数在方法参数列表中的位置下标
  private final Integer resultHandlerIndex;
  // 分页配置参数在方法参数列表中的位置下标
  private final Integer rowBoundsIndex;
  // 参数名解析器
  private final ParamNameResolver paramNameResolver;

  public MethodSignature(Configuration configuration, Class<?> mapperInterface, Method method) {
    Type resolvedReturnType = TypeParameterResolver.resolveReturnType(method, mapperInterface);
    if (resolvedReturnType instanceof Class<?>) {
      this.returnType = (Class<?>) resolvedReturnType;
    } else if (resolvedReturnType instanceof ParameterizedType) {
      this.returnType = (Class<?>) ((ParameterizedType) resolvedReturnType).getRawType();
    } else {
      this.returnType = method.getReturnType();
    }
    this.returnsVoid = void.class.equals(this.returnType);
    this.returnsMany = configuration.getObjectFactory().isCollection(this.returnType) || this.returnType.isArray();
    this.returnsCursor = Cursor.class.equals(this.returnType);
    this.returnsOptional = Optional.class.equals(this.returnType);
    this.mapKey = getMapKey(method);
    this.returnsMap = this.mapKey != null;
    this.rowBoundsIndex = getUniqueParamIndex(method, RowBounds.class);
    this.resultHandlerIndex = getUniqueParamIndex(method, ResultHandler.class);
    this.paramNameResolver = new ParamNameResolver(configuration, method);
  }

  // 将方法中的参数转换成SQL脚本命令中的参数形式，其实就是将参数位置作为键，具体参数作为值保存到一个Map集合中，SQL脚本中用键#{1}通过集合技能得到具体的参数
  public Object convertArgsToSqlCommandParam(Object[] args) {
    return paramNameResolver.getNamedParams(args);
  }

  // 省略
}
```

