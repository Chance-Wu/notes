Redis要实现分布式锁，需满足以下条件：

- 互斥性（任意时刻只有一个客户端能持有锁）
- 不能死锁（客户端持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁）
- 容错性（只要大部分的Redis节点正常运行，客户端就可以加锁和解锁）



#### 实现

---

>**加锁分析**
>
>```lua
>//获取锁（unique_value可以是UUID等）
>SET resource_name unique_value NX PX 30000
>```
>
>`set()`方法加入：
>
>- `NX`（**N**ot e**X**ists），可以保证如果已有key存在，则函数不会调用成功，<u>保证只有一个客户端持有锁</u>；
>- `PX`，设置自定的到期时间（ms）。<u>保证了不会发生死锁</u>。
>- 将value复制为requestId，用来表示这把锁是属于哪个请求加的，那么在客户端解锁的时候就可以进行校验是否是同一个客户端。

>**解锁分析**
>
>```lua
>//释放锁（lua脚本中，一定要比较value，防止误解锁）
>if redis.call("get",KEYS[1]) == ARGV[1] then
>  return redis.call("del",KEYS[1])
>else
>  return 0
>end
>```
>
>通过lua脚本来==避免CAS模型的并发问题==，因为在释放锁的时候因为涉及到多个Redis操作（利用了==eval命令==执行Lua脚本的原子性）。将Lua代码传到jedis.eval()方法里，并使参数KEYS[1]赋值为lockKey，ARGV[1]赋值为requestId。在执行的时候，首先会获取锁对应的value值，检查是否与requestId相等，如果相等则解锁。

>存在风险：
>
>如果存储锁对应key的那个节点挂了的话，就可能存在丢失锁的风险，导致出现多个客户端持有锁的情况，这样就不能实现资源的独享了。



#### redlock算法

---

假设一个redis集群，有5个redis master实例。然后执行如下如下步骤获取一把锁：

1. 获取当前时间戳（ms）。
2. 轮流用相同的key和随机值在每个master节点上创建锁，比如锁的自动释放时间是10秒钟，那每个节点锁请求的超时时间可能是5-50ms的范围，这个可以防止一个客户端在某个宕掉的master节点上阻塞过长时间，如果一个master节点不可用，应该尽快尝试下一个master节点；
3. 客户端计算第2步中获取锁所花的时间，只有==当客户端在大多数master节点上成功获取了锁（n/2+1），且总共消耗的时间不超过锁释放时间，这个锁就认为是获取成功了==；
4. 如果锁获取成功了，那现在锁自动释放时间就是最初的锁释放时间减去之前获取锁所消耗的时间；
5. 如果锁获取失败，不管是因为获取成功的锁不超过一半还是因为总消耗时间超过了锁释放时间，客户端都会到每个master节点上释放锁，即便是那些他认为没有获取成功的锁。

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gpzhv36vw0j30f908m74a.jpg" style="zoom:100%">



#### Redisson实现

---

在Redis的基础上实现的Java驻内存数据网格。提供了一系列的分布式的Java常用对象，实现了：

- 可重入锁
- 公平锁
- 联锁
- 红锁
- 读写锁

还提供了许多分部署服务。



> **Redisson分布式重入锁用法**
>
> 支持单点模式、主从模式、哨兵模式、集群模式。
>
> 以单点模式为例：
>
> ```java
> private final static String lockKey = "lock";
> private final static int waitTimeout = 5;
> private final static int leaseTime = 10;
> 
> @Test
> void redissonTest() {
>   // 1.构造redis配置
>   Config config = new Config();
>   config.useSingleServer()
>     .setAddress("redis://127.0.0.1:6379")
>     //                .setPassword("")
>     .setDatabase(0);
> 
>   // 2.创建redisson客户端
>   RedissonClient redissonClient = Redisson.create(config);
>   // 3.获取锁对象实例（无法保证是按线程的顺序获取到）
>   RLock rLock = redissonClient.getLock(lockKey);
> 
>   try {
>     /**
>              * 4.尝试获取锁
>              * waitTimeout 尝试获取锁的最大等待时间，超过这个值，则认为获取锁失败
>              * leaseTime   锁的持有时间,超过这个时间锁会自动失效（值应设置为大于业务处理的时间，确保在锁有效期内业务能处理完）
>              */
>     boolean res = rLock.tryLock((long) waitTimeout, (long) leaseTime, TimeUnit.SECONDS);
>     if (res) {
>       log.info("成功获得锁，在这里处理业务。。。");
>     }
>   } catch (Exception e) {
>     throw new RuntimeException("aquire lock fail");
>   } finally {
>     // 5.无论如何, 最后都要解锁
>     rLock.unlock();
>   }
> }
> ```
>
> <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gpziqnv10bj30ms0ikt94.jpg" style="zoom:100%">
>
> <img src="https://tva1.sinaimg.cn/large/008i3skNgy1gpzirdo285j30ks0jhdg7.jpg" style="zoom:100%">
>
> RedissonLock是可重入的，并且考虑了失败次数，可以设置锁的最大等待时间。
>
> 上述没有解决节点挂掉的时候，存在丢失锁的风险问题。所以Redisson提供了实现redlock算法的`RedissonRedLock`，==其真正解决了单点失败的问题，代价是需要额外的为RedissonRedLock搭建Redis环境==。
>
> 如果业务场景可以容忍这种小概率的错误，则推荐使用RedissonLock，否则使用RedissonRedLock。



