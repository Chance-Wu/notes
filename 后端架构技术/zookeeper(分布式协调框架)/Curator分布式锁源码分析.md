#### 1. Apache Curator

>Curator是为ZooKeeper开发的一套Java客户端类库，它是一个分布式协调服务。
>
>包含：
>
>- 一套高级API框架
>- 工具类
>
>解决了以下问题：
>
>1. 封装ZooKeeper client与ZooKeeper server之间的==连接处理==；
>2. 提供了一套==Fluent风格的操作API==；
>3. 提供例如ZooKeeper各种==应用场景的抽象封装==。（分布式锁，选举）

>Curator主要包含了以下模块：
>
><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gmdoyu9ky3j30ca0ig74g.jpg" style="zoom:60%">
>
>- curator-client：封装操作原生ZooKeeper的底层，包括连接，重试，集群等。
>- curator-framework：提供高层抽象API简化ZooKeeper的使用，封装了管理到ZooKeeper集群的连接以及重试机制的复杂性，是Curator对外提供的接口。
>- curator-recipes：构建在Curator Framework框架之上，实现了原生ZooKeeper中的分布式工具特征recipes。
>- curator-examples：提供了Curator使用例子，包括服务发现、选举、分布式锁，缓存等。

#### 2. 使用Curator

##### 2.1 引入curator依赖

>```xml
><!--注意与zookeeper的版本对应-->
><dependency>
>    <groupId>org.apache.curator</groupId>
>    <artifactId>curator-recipes</artifactId>
>    <version>4.3.0</version>
></dependency>
>```
>
><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gmdplwyaf9j30r8084aac.jpg" style="zoom:50%">

##### 2.2 创建CuratorFramework

>```java
>@Configuration
>public class CuratorConfig {
>
>    @Value("${zkCluster.ip}")
>    private String zkClusterIp;
>
>    /**
>     * 将 Zookeeper 框架式客户端交给Spring管理
>     * 并设置一个初始化方法 start
>     */
>    @Bean(initMethod = "start")
>    public CuratorFramework curatorFramework() {
>        // 重试策略
>        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
>        // 创建具有默认会话超时(60m)和默认连接超时(15m)的新客户端
>        CuratorFramework client = CuratorFrameworkFactory.newClient(zkClusterIp, retryPolicy);
>        //client.start();
>        return client;
>    }
>}
>```

##### 2.3 创建分布式锁

>```java
>@RestController
>public class ProductController {
>
>    @Value("${server.port}")
>    private String port;
>
>    @Autowired
>    private OrderService orderService;
>
>    /**注入Curator客户端*/
>    @Autowired
>    private CuratorFramework curatorFramework;
>
>    @PostMapping("/stock/deduct")
>    public Object reduceStock(Integer id) throws Exception {
>
>        // 创建互斥锁
>        InterProcessMutex interProcessMutex = new InterProcessMutex(curatorFramework, "/product" + id);
>
>        try {
>            // 加锁，acquire()：获取互斥锁，直到可用为止。
>            interProcessMutex.acquire();
>
>            orderService.reduceStock(id);
>        } catch (Exception e) {
>            if (e instanceof RuntimeException) {
>                throw e;
>            }
>        }finally {
>            // 释放互斥锁
>            interProcessMutex.release();
>        }
>        return "ok:" + port;
>    }
>}
>```

#### 3. 源码分析

><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gmehtsh89oj30d40fy0ti.jpg" style="zoom:60%">
>
>1. 请求进来，直接在/lock节点下创建一个临时顺序节点
>2. 判断自己是不是lock节点下，最小的节点。
>   1. 是最小的，获得锁
>   2. 不是，对前面的节点进行监听（watch）
>3. 获得锁，处理完释放锁，即delete节点，然后后继第一个节点将收到通知，重复第2步判断。

##### 3.1 实例化InterProcessMutex

>```java
>public InterProcessMutex(CuratorFramework client, String path)
>{
>    this(client, path, new StandardLockInternalsDriver());
>}
>
>public InterProcessMutex(CuratorFramework client, String path, LockInternalsDriver driver)
>{
>    this(client, path, LOCK_NAME, 1, driver);
>}
>```
>
>两个构造函数共同的入参：
>
>- client：curator实现的zookeeper客户端
>- path：要在zookeeper加锁的路径，即后面创建临时节点的父节点
>
>上面两个构造函数中，其实第一个也是在调用第二个构造函数，它传入了一个默认的`StandardLockInternalsDriver对象`，即标准的==锁驱动类==。就是说InterProcessMutex也支持你传入自定义的锁驱动类来扩展。
>
>```java
>// 代码进入：InterProcessMutex.java
>InterProcessMutex(CuratorFramework client, String path, String lockName, int maxLeases, LockInternalsDriver driver)
>{
>    // 验证入参的合法性 
>    basePath = PathUtils.validatePath(path);
>    // 实例化了一个LockInternals对象
>    internals = new LockInternals(client, driver, path, lockName, maxLeases);
>}
>// 代码进入：LockInternals.java
>LockInternals(CuratorFramework client, LockInternalsDriver driver, String path, String lockName, int maxLeases)
>{
>    this.driver = driver;
>    this.lockName = lockName;
>    this.maxLeases = maxLeases;
>
>    this.client = client.newWatcherRemoveCuratorFramework();
>    this.basePath = PathUtils.validatePath(path);
>    this.path = ZKPaths.makePath(path, lockName);
>}
>```
>
>做了2件事：验证入参的合法性 & 实例化了一个LockInternals对象。

##### 3.2 加锁方法acquire

>实例化完InterProcessMutex对象，开始调用acquire()方法来尝试加锁：
>
>```java
>@Override
>public void acquire() throws Exception
>{
>    if ( !internalLock(-1, null) )
>    {
>        throw new IOException("Lost connection while trying to acquire lock: " + basePath);
>    }
>}
>
>@Override
>public boolean acquire(long time, TimeUnit unit) throws Exception
>{
>    return internalLock(time, unit);
>}
>```
>
>- `acquire()` ：调用该方法后，会一直堵塞，直到抢夺到锁资源，或者zookeeper连接中断后，上抛异常。
>- `acquire(long time, TimeUnit unit)`：传入超时时间以及单位，抢夺时，如果出现堵塞，会在超过该时间后，返回false。
>
>推荐后者，==传入超时时间，避免出现大量的临时节点累积以及线程堵塞的问题==。

##### 3.3 锁的可重入

>```java
>private boolean internalLock(long time, TimeUnit unit) throws Exception
>{
>    //注意并发：给定的lockData实例只能由单个线程执行，因此不需要锁定
>    Thread currentThread = Thread.currentThread();
>
>    // 查找当前线程
>    LockData lockData = threadData.get(currentThread);
>    if ( lockData != null )
>    {
>        // 重新进入（可重入锁）
>        lockData.lockCount.incrementAndGet();
>        return true;
>    }
>
>    // 加锁
>    String lockPath = internals.attemptLock(time, unit, getLockNodeBytes());
>    // 加锁成功
>    if ( lockPath != null )
>    {
>        LockData newLockData = new LockData(currentThread, lockPath);
>        threadData.put(currentThread, newLockData);
>        return true;
>    }
>
>    return false;
>}
>```
>
>上述代码，实现了锁的可重入。每个InterProcessMutex实例，都会持有一个`ConcurrentHashMap`类型的threadData对象，以线程对象作为key，以LockData作为value值。
>
>通过判断当前线程threadData是否有值：
>
>- 如果有，则表示线程可以重入该锁，于是将lockData的lockCount进行累加；
>- 如果没有，则进行锁的抢夺。
>
>`internals.attemptLock`方法返回lockPath!=null时，表明了该线程已经成功持有了这把锁，于是乎LockData对象被new了出来，并存放到threadData中。

##### 3.4 抢夺锁

>`attemptLock`方法就是核心部分：
>
>```java
>String attemptLock(long time, TimeUnit unit, byte[] lockNodeBytes) throws Exception
>{
>    final long      startMillis = System.currentTimeMillis();
>    final Long      millisToWait = (unit != null) ? unit.toMillis(time) : null;
>    final byte[]    localLockNodeBytes = (revocable.get() != null) ? new byte[0] : lockNodeBytes;
>    int             retryCount = 0;
>
>    String          ourPath = null;
>    boolean         hasTheLock = false;
>    boolean         isDone = false;
>    while ( !isDone )
>    {
>        isDone = true;
>
>        try
>        {
>            // 创建临时顺序节点
>            ourPath = driver.createsTheLock(client, path, localLockNodeBytes);
>            hasTheLock = internalLockLoop(startMillis, millisToWait, ourPath);
>        }
>        catch ( KeeperException.NoNodeException e )
>        {
>            // 当它找不到锁节点时，由StandardLockInternalsDriver抛出
>            // 这可能发生在会话过期等情况下。所以，如果重试允许，就再试一次
>            if ( client.getZookeeperClient().getRetryPolicy().allowRetry(retryCount++, System.currentTimeMillis() - startMillis, RetryLoop.getDefaultRetrySleeper()) )
>            {
>                isDone = false;
>            }
>            else
>            {
>                throw e;
>            }
>        }
>    }
>
>    if ( hasTheLock )
>    {
>        return ourPath;
>    }
>
>    return null;
>}
>```
>
>1. while循环
>
>   正常情况下，这个循环会在下一次结束。但是当出现NoNodeException异常时，会根据zookeeper客户端的重试策略，进行有限次数的重新获取锁。
>
>2. driver.createsTheLock(client, path, localLockNodeBytes);
>
>   createTheLock就是在创建这个锁，即在zookeeper指定路径上，创建一个临时顺序节点。（注意：此时只是纯粹的创建了一个节点，不是说线程已经持有了锁）
>
>   ```java
>   @Override
>   public String createsTheLock(CuratorFramework client, String path, byte[] lockNodeBytes) throws Exception
>   {
>       String ourPath;
>       if ( lockNodeBytes != null )
>       {
>           // 创建一个容器节点作为父节点，创建临时的顺序节点
>           ourPath = client.create().creatingParentContainersIfNeeded().withProtection().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath(path, lockNodeBytes);
>       }
>       else
>       {
>           ourPath = client.create().creatingParentContainersIfNeeded().withProtection().withMode(CreateMode.EPHEMERAL_SEQUENTIAL).forPath(path);
>       }
>       return ourPath;
>   }
>   ```
>
>3. internalLockLoop
>
>   判断自身是否能够持有锁。如果不能，进入wait，等待被唤醒。
>
>   ```java
>   private boolean internalLockLoop(long startMillis, Long millisToWait, String ourPath) throws Exception
>   {
>       boolean     haveTheLock = false;
>       boolean     doDelete = false;
>       try
>       {
>           if ( revocable.get() != null )
>           {
>               client.getData().usingWatcher(revocableWatcher).forPath(ourPath);
>           }
>   
>           // 如果一开始使用无参的acquire方法，那么此处的循环可能就是一个死循环。当zookeeper客户端启动时，并且当前线程还没有成功获取到锁时，就会开始新的一轮循环。
>           while ( (client.getState() == CuratorFrameworkState.STARTED) && !haveTheLock )
>           {
>               // 获取到所有子节点列表，并且从小到大根据节点名称后10位数字进行排序。
>               List<String>        children = getSortedChildren();
>               String              sequenceNodeName = ourPath.substring(basePath.length() + 1); // +1 to include the slash
>   
>               // 判断是否可以持有锁
>               PredicateResults    predicateResults = driver.getsTheLock(client, children, sequenceNodeName, maxLeases);
>               if ( predicateResults.getsTheLock() )
>               {
>                   haveTheLock = true;
>               }
>               else
>               {
>                   String  previousSequencePath = basePath + "/" + predicateResults.getPathToWatch();
>   
>                   // 代码块在争夺锁失败以后的逻辑中。那么此处线程应该做什么呢？
>                   synchronized(this)
>                   {
>                       try
>                       {
>                           // 使用getData（）而不是exist（）以避免留下不必要的观察者，这是一种资源泄漏
>                           client.getData().usingWatcher(watcher).forPath(previousSequencePath);
>                           if ( millisToWait != null )
>                           {
>                               millisToWait -= (System.currentTimeMillis() - startMillis);
>                               startMillis = System.currentTimeMillis();
>                               if ( millisToWait <= 0 )
>                               {
>                                   doDelete = true;    // timed out - delete our node
>                                   break;
>                               }
>   
>                               wait(millisToWait);
>                           }
>                           else
>                           {
>                               wait();
>                           }
>                       }
>                       catch ( KeeperException.NoNodeException e )
>                       {
>                           // it has been deleted (i.e. lock released). Try to acquire again
>                       }
>                   }
>               }
>           }
>       }
>       catch ( Exception e )
>       {
>           ThreadUtils.checkInterrupted(e);
>           doDelete = true;
>           throw e;
>       }
>       finally
>       {
>           if ( doDelete )
>           {
>               deleteOurPath(ourPath);
>           }
>       }
>       return haveTheLock;
>   }
>   ```
>
>   1. driver.getsTheLock
>
>      判断是否可以持有锁，判断规则：当前创建的节点是否在上一步获取到的子节点列表的首位。
>      如果是，说明可以持有锁，那么getsTheLock = true，封装进PredicateResults返回。
>      如果不是，说明有其他线程早已先持有了锁，那么getsTheLock = false，此处还需要获取到自己前一个临时节点的名称pathToWatch。（注意这个pathToWatch后面有比较关键的作用）
>
>   2. synchronized(this)
>
>      这块代码在争夺锁失败以后的逻辑中。那么此处该线程应该做什么呢？
>      首先添加一个watcher监听，而监听的地址正是上面一步返回的pathToWatch进行basePath + "/" 拼接以后的地址。也就是说当前线程会监听自己前一个节点的变动，而不是父节点下所有节点的变动。然后华丽丽的...wait(millisToWait)。线程交出cpu的占用，进入等待状态，等到被唤醒。
>      接下来的逻辑就很自然了，如果自己监听的节点发生了变动，那么就将线程从等待状态唤醒，重新一轮的锁的争夺。
>
>至此，完成了整个锁的抢夺过程。

##### 3.5 释放锁

>```java
>@Override
>public void release() throws Exception
>{
>    // 注意并发：一个给定的lockData实例只能由单个线程进行操作，所以不需要锁定
>    Thread currentThread = Thread.currentThread();
>    LockData lockData = threadData.get(currentThread);
>    if ( lockData == null )
>    {
>        throw new IllegalMonitorStateException("You do not own the lock: " + basePath);
>    }
>
>    int newLockCount = lockData.lockCount.decrementAndGet();
>    if ( newLockCount > 0 )
>    {
>        return;
>    }
>    if ( newLockCount < 0 )
>    {
>        throw new IllegalMonitorStateException("Lock count has gone negative for lock: " + basePath);
>    }
>    try
>    {
>        internals.releaseLock(lockData.lockPath);
>    }
>    finally
>    {
>        threadData.remove(currentThread);
>    }
>}
>```
>
>- 减少重入锁的计数，直到变成0。
>- 释放锁，即移除 Watchers & 删除创建的节点。
>- 从threadData中，删除自己线程的缓存。

#### 4. 锁驱动类

>`StandardLockInternalsDriver`——标准锁驱动类
>
>StandardLockInternalsDriver类提供的功能接口：
>
>```java
>// 判断是否获取到了锁
>public PredicateResults getsTheLock(CuratorFramework client, List<String> children, String sequenceNodeName, int maxLeases);
>
>// 在zookeeper的指定路径上，创建一个临时顺序节点
>public String createsTheLock(CuratorFramework client, String path, byte[] lockNodeBytes);
>
>// 修复排序，在StandardLockInternalsDriver的实现中，即获取到临时节点的最后序列数，进行排序。
>public String fixForSorting(String str, String lockName);
>```
>
>借助于这个类，我们可以尝试实现自己的锁机制，比如判断锁获得的策略可以做修改，比如获取子节点列表的排序方案可以自定义。

#### 5. InterProcessMutex原理总结

>InterProcessMutex通过在zookeeper的某路径节点下创建临时序列节点来实现分布式锁，即==每个线程（跨进程的线程）获取同一把锁前，都需要在同样的路径下创建一个节点，节点名字由uuid + 递增序列组成==。
>
>通过对比自身的序列数是否在所有子节点的第一位，来判断是否成功获取到了锁。
>
>当获取锁失败时，它会添加watcher来监听前一个节点的变动情况，然后进行等待状态。直到watcher的事件生效将自己唤醒，或者超时时间异常返回。



