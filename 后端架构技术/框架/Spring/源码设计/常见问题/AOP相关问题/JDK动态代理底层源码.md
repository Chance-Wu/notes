>当spring没有手动开启Cglib动态代理，即：`<aop:aspectj-autoproxy proxy-target-class="true"/>`或`@EnableAspectJAutoProxy(proxyTargetClass = true)`，默认使用的就是Jdk动态代理。
>
>以下将模拟`@Cacheable`，即缓存在动态代理中的应用进行讲解。需要注意的是，==Jdk动态代理相比起cglib动态代理，Jdk动态代理的对象必须实现接口，否则将报错。我们也将带着这个问题在源码分析中寻找答案。==
>
>当@Cacheable注解在方法上时：
>
>1. 在方法执行前，将调用JDK动态代理有限查找Redis缓存。
>2. 当缓存不存在时，执行方法，例如查询数据库
>3. 在方法执行后，再次调用JDK动态代理，将结果缓存到Redis中。

#### 1. 使用

>1. 创建接口UserService
>
>   ```java
>   public interface UserService {
>       public String getUserByName(String name);
>   }
>   ```
>
>2. 创建实现类UserServiceImpl
>
>   ```java
>   public class UserServiceImpl implements UserService {
>       @Override
>       public String getUserByName(String name) {
>           System.out.println("从数据库中查询到:" + name);
>           return name;
>       }
>   }
>   ```
>
>3. 创建Jdk动态代理JdkCacheHandler
>
>```java
>public class JdkCacheHandler implements InvocationHandler {
>
>    /**
>     * 目标对象
>     */
>    private Object target;
>
>    public JdkCacheHandler(Object target) {
>        this.target = target;
>    }
>
>    /**
>     * 创建JDK代理
>     */
>    public Object createJDKProxy() {
>        Class clazz = target.getClass();
>        // 创建JDK代理需要3个参数，目标类加载器、目标类接口、代理类对象(即本身)
>        return Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(), this);
>    }
>
>    @Override
>    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
>        System.out.println("查找数据库前，在缓存中查找是否存在:" + args[0]);
>        // 触发目标类方法
>        Object result = method.invoke(target, args);
>        System.out.printf("查找数据库后，将%s加入到缓存中\r\n", result);
>        return result;
>    }
>}
>```
>
>4. 测试
>
>   ```java
>   @Test
>   void testJdkProxy() {
>       UserService userService = new UserServiceImpl();
>       JdkCacheHandler jdkCacheHandler = new JdkCacheHandler(userService);
>       UserService proxy = (UserService) jdkCacheHandler.createJDKProxy();
>   
>       System.out.println("==========================");
>       proxy.getUserByName("bugpool");
>       System.out.println("==========================");
>   
>       System.out.println(proxy.getClass());
>   }
>   ```
>
>```
>==========================
>查找数据库前，在缓存中查找是否存在:bugpool
>从数据库中查询到:bugpool
>查找数据库后，将bugpool加入到缓存中
>==========================
>class com.sun.proxy.$Proxy84
>```

#### 2. 调用机制

##### 2.1 查看$Proxy代码

>当经过JDK动态代理以后，生成的proxy已经不再是UserService类型了，而是$Poxy85类型，了解其调用机制，得先获取proxy类的代码。
>
>修改JVM运行参数，添加`-Dsun.misc.ProxyGenerator.saveGeneratedFiles=true`
>
>可以在com.sun.proxy查看到生成的代理类，经过反编译后的代码如下：
>
>```java
>public final class $Proxy84 extends Proxy implements UserService {
>    private static Method m1;
>    private static Method m3;
>    private static Method m2;
>    private static Method m0;
>
>    public $Proxy84(InvocationHandler var1) throws  {
>        super(var1);
>    }
>
>    public final boolean equals(Object var1) throws  {
>        try {
>            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
>        } catch (RuntimeException | Error var3) {
>            throw var3;
>        } catch (Throwable var4) {
>            throw new UndeclaredThrowableException(var4);
>        }
>    }
>
>    public final String getUserByName(String var1) throws  {
>        try {
>            return (String)super.h.invoke(this, m3, new Object[]{var1});
>        } catch (RuntimeException | Error var3) {
>            throw var3;
>        } catch (Throwable var4) {
>            throw new UndeclaredThrowableException(var4);
>        }
>    }
>
>    public final String toString() throws  {
>        try {
>            return (String)super.h.invoke(this, m2, (Object[])null);
>        } catch (RuntimeException | Error var2) {
>            throw var2;
>        } catch (Throwable var3) {
>            throw new UndeclaredThrowableException(var3);
>        }
>    }
>
>    public final int hashCode() throws  {
>        try {
>            return (Integer)super.h.invoke(this, m0, (Object[])null);
>        } catch (RuntimeException | Error var2) {
>            throw var2;
>        } catch (Throwable var3) {
>            throw new UndeclaredThrowableException(var3);
>        }
>    }
>
>    static {
>        try {
>            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
>            m3 = Class.forName("com.chance.aop.service.UserService").getMethod("getUserByName", Class.forName("java.lang.String"));
>            m2 = Class.forName("java.lang.Object").getMethod("toString");
>            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
>        } catch (NoSuchMethodException var2) {
>            throw new NoSuchMethodError(var2.getMessage());
>        } catch (ClassNotFoundException var3) {
>            throw new NoClassDefFoundError(var3.getMessage());
>        }
>    }
>}
>```

##### 2.2 调用机制

>1. 从proxy调用开始
>
>   ```java
>   proxy.getUserByName("bugpool");
>   ```
>
>2. proxy是`$Proxy84`类型，因此进入`$Proxy84`的getUserByName方法
>
>   ```java
>   public final class $Proxy84 extends Proxy implements UserService {
>       //...
>   
>       // 构造器，传入JdkCacheHandler类的对象，正是下方调用的super.h属性 
>       public $Proxy84(InvocationHandler var1) throws  {
>           super(var1);
>       }
>   
>       public final String getUserByName(String var1) throws  {
>           try {
>               /**
>                * 调用父类的h属性的invoke方法
>                */
>               return (String)super.h.invoke(this, m3, new Object[]{var1});
>           } catch (RuntimeException | Error var3) {
>               throw var3;
>           } catch (Throwable var4) {
>               throw new UndeclaredThrowableException(var4);
>           }
>       }
>   
>       //...
>   }
>   ```
>
>3. 因此==`h.invoke`实际调用的正是JdkCacheHandler类的invoke方法==。
>
>   ```java
>   public class JdkCacheHandler implements InvocationHandler {
>       //...
>       @Override
>       public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
>   
>           System.out.println("查找数据库前，在缓存中查找是否存在:" + args[0]);
>           // 触发目标类方法
>           Object result = method.invoke(target, args);
>           System.out.printf("查找数据库后，将%s加入到缓存中\r\n", result);
>           return result;
>       }
>   }
>   ```
>
>4. 而`method.invoke(target, args)`中的method=getUserByName，target=构造函数传进来的UserServiceImpl对象，args="bugpool"
>
>   ```java
>   public class UserServiceImpl implements UserService {
>       @Override
>       public String getUserByName(String name) {
>           System.out.println("从数据库中查询到:" + name);
>           return name;
>       }
>   }
>   ```

#### 3. 源码分析

##### 3.1 原理

>了解完Jdk动态代理的调用机制，所有核心问题都落在了$Proxy84类的对象proxy是如何生成的？即如下代理，先了解一下java的运行机制：
>
>1. 所有的.java文件经过编译生成.class文件。
>2. 通过类加载器ClassLoader将.class中的字节码加载到JVM中。
>3. 运行。
>
>```java
>// 创建JDK代理
>public Object createJDKProxy() {
>    Class clazz = target.getClass();
>    // 创建JDK代理需要3个参数
>    // 目标类加载器：用于加载生成的字节码
>    // 目标类接口：用于生成字节码，也就是说$Proxy的生产仅仅需要接口数组就可以完成
>    // 代理类对象(即本身)：用于回调invoke方法，实现方法的增强
>    return Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(), this);
>}
>```
>
>1. 通过`clazz.getInterfaces()`获取到所有接口，通过接口可以生成类似以下字节码（注意以下给出的是代码），细细观察会发现其实各个接口方法生成的代码都是一样的，只有`(String)super.h.invoke(this, m3, new Object[]{var1}`的m3和参数有可能是不同的。所以其实==想生成$Proxy字节码，只需要接口数组就已经完全足够了==。
>
>```java
>public static Object newProxyInstance(ClassLoader loader,
>                                      Class<?>[] interfaces,
>                                      InvocationHandler h)
>    throws IllegalArgumentException
>{
>    Objects.requireNonNull(h);
>
>    final Class<?>[] intfs = interfaces.clone();
>    final SecurityManager sm = System.getSecurityManager();
>    if (sm != null) {
>        checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
>    }
>
>    // 当缓存中存在代理类则直接获取，否则生成代理类
>    // ①②步骤，核心代码，即生成代理类字节码以及加载都在这里进行
>    Class<?> cl = getProxyClass0(loader, intfs);
>
>    try {
>        if (sm != null) {
>            checkNewProxyPermission(Reflection.getCallerClass(), cl);
>        }
>
>        // 获取代理类的构造函数
>        final Constructor<?> cons = cl.getConstructor(constructorParams);
>        final InvocationHandler ih = h;
>        if (!Modifier.isPublic(cl.getModifiers())) {
>            AccessController.doPrivileged(new PrivilegedAction<Void>() {
>                public Void run() {
>                    cons.setAccessible(true);
>                    return null;
>                }
>            });
>        }
>        // 通过反射调用代理类的构造函数
>        return cons.newInstance(new Object[]{h});
>    ...
>}
>```
>
>2. 此时已经获取到`$Proxy84.class`的字节码，但是==此处的字节码还未加载到JVM中，因此需要调用`clazz.getClassLoader()`传进来的类加载器进行加载==，并得到对应的class，也就是`$Proxy84`类。
>
>3. 获取$Proxy84类的构造函数，该构造函数有一个重要的参数h
>
>   ```java
>   public $Proxy84(InvocationHandler var1) throws  {
>       super(var1);
>   }
>   ```
>
>4. 通过反射调用`$Proxy84`类的构造函数，`cons.newInstance(new Object[]{h});`构造函数的h正是传入的this，即JdkCacheHandler类的对象。
>
>5. 将反射获取到的`$Proxy84`对象返回。

##### 3.2 源码分析

>1. 从Proxy.newProxyInstance开始跟踪代码。
>
>2. 跟踪代码newProxyInstance，这里需要注意`Class<?> cl = getProxyClass0(loader, intfs);`结束后，==cl变量一直都只是class，即$Proxy84类，并未生成对应的对象，这里不要混淆类和对象==。
>
>3. 跟踪`Class<?> cl = getProxyClass0(loader, intfs);`
>
>4. 跟踪`proxyClassCache.get(loader, interfaces);`（注：Jdk动态代理对已经生成加载过的代理类进行了缓存以提高性能，缓存的相关代码不是我们关心的重点，可以跳过相关代码），主要关心`V value = supplier.get();`其中==supplier本质是factory，通过`new Factory(key, parameter, subKey, valuesMap)`创建==。
>
>5. 跟踪`V value = supplier.get();`即Factory类的get方法，这里大部分的工作还是在做校验和缓存，我们只关心核心逻辑`valueFactory.apply(key, parameter);`其中valueFactory是上一个步骤传入的ProxyClassFactory。
>
>   ```java
>   // Factory.java    
>   public synchronized V get() { // 序列化访问
>       // 再次检查，supplier是否是当前对象
>       Supplier<V> supplier = valuesMap.get(subKey);
>       if (supplier != this) {
>           return null;
>       }
>   
>       // create new value
>       V value = null;
>       try {
>           // valueFactory 是前序传进来的 new ProxyClassFactory()
>           // 核心逻辑，调用valueFactory.apply生成对应代理类并加载
>           value = Objects.requireNonNull(valueFactory.apply(key, parameter));
>       } finally {
>           if (value == null) {
>               valuesMap.remove(subKey, this);
>           }
>       }
>       // 唯一到达此处的路径是非空值
>       assert value != null;
>   
>       // 用CacheValue包装值（WeakReference）
>       CacheValue<V> cacheValue = new CacheValue<>(value);
>   
>       // 放入reverseMap
>       reverseMap.put(cacheValue, Boolean.TRUE);
>   
>       // 尝试用CacheValue代替我们（这应该总是成功的）
>       if (!valuesMap.replace(subKey, this, cacheValue)) {
>           throw new AssertionError("Should not reach here");
>       }
>   
>       return value;
>   }
>   ```
>
>6. 跟踪核心逻辑
>
>   1. Jdk动态代理通过拼凑的方式拼凑出`$Proxy`的全类名：`com.sun.proxy.$Proxy84.class`
>   2. ==生产字节码`byte[] proxyClassFile = ProxyGenerator.generateProxyClass( proxyName, interfaces, accessFlags);`可以看出Jdk动态代理需要interfaces接口数组进行生成字节码，这也是为什么Jdk动态代理必须实现接口的原因==。同时从参数也可以看出需要生成字节码其实只需要接口数组，不需要其他信息。其实实现原理大概也可以猜出，Jdk动态代理通过遍历所有接口方法，为方法生成对应的`return (String)super.h.invoke(this, m0~n, new Object[]{var1});`代码
>   3. 加载字节码：在上面获取到了字节码的字节数组，接下去就是调用classLoader将所有的字节码读入到JVM中
>
>   ```java
>   return defineClass0(loader, proxyName,
>                       proxyClassFile, 0, proxyClassFile.length);
>   ```

