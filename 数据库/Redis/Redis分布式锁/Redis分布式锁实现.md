> `锁`：解决多个执行线程访问共享资源错误或数据不一致问题的工具。
>
> `分布式锁`：解决分布式应用之间访问共享资源的并发问题。

#### 1. 分布式锁应用场景

>1. ==避免不同节点重复相同的工作==：比如用户执行了某个操作有可能不同节点会发送多封邮件；
>
>2. ==避免破坏数据的正确性==：如果两个节点在同一条数据上同时进行操作，可能会造成数据错误或不一致的情况出现；

#### 2. 常见实现方式

>锁的本质是`同一时间只允许一个用户操作`。
>
>1. **基于MySQL中的锁**：mysql本身有自带的悲观锁`for update`关键字。也可以自己实现悲观锁/乐观锁来达到目的。
>2. **基于Zookeeper有序节点**：创建临时节点的`有序节点`，这样客户端获取节点列表时，就能够根据当前子节点列表中的序号判断是否能够获得锁。
>3. **基于Redis的单线程**：由于`Redis是单线程`，所以命令会以串行的方式执行，并且本身提供了像SETNX（set if not exists）这样的指令，本身具有互斥性。

#### 3. Redis分布式锁的问题

##### 3.1 锁超时

>假设现在有两台平行的服务A、B，其中A服务在获取锁之后由于位置神秘力量突然挂了，那么B服务就永远无法获取到锁了：
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmo5qclbs3j30n409aaa7.jpg" style="zoom:80%">
>
>此时，需要设置一个超时时间，来保证服务的可用性。

>另一个问题随之而来：==如果在加锁和释放锁之间的逻辑执行得太长，以至于超出了锁的超时限制==，也会出现问题。因为这时候第一个线程持有锁过期了，而临界区的逻辑还没有执行完，与此同时第二个线程就提前拥有了这把锁，导致临界区的代码不能得到严格的串行执行。

##### 3.2 Java GC可能引发的安全问题

>在GC的时候会发生STW（Stop-The-World），这本身是为了保障垃圾回收器的正常执行，但可能会引发如下的问题：
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEly1gmo6cd9d3nj30n40dg0td.jpg" style="zoom:80%">
>
>服务A获取了锁并设置了超时时间，但是服务A出现了STW且时间较长，导致了==分布式锁进行了超时释放==，在这个期间服务B获取到了锁，待服务A STW结束之后又恢复了锁，这就导致了服务A和服务B同时获取到了锁，这个时候分布式锁就不安全了

##### 3.3 单点、多点问题

>- 如果 Redis 采用单机部署模式，当 Redis 故障了，就会导致整个服务不可用。
>
>- 如果采用主从模式部署，我们想象一个这样的场景：服务A申请到一把锁之后，如果作为主机的Redis宕机了，那么服务B在申请锁的时候就会从从机那里获取到这把锁，为了解决这个问题，Redis 作者提出了一种==RedLock红锁==的算法(Redission同Jedis)。

#### 4. 实现

##### 4.1 使用redisson实现分布式锁

>假设有5个完全独立的redis主服务器。
>
>1. 获取当前时间戳
>
>2. client尝试==按照顺序使用相同的key，value获取所有redis服务的锁==，在获取锁的过程中的获取时间比锁过期时间短很多，这是为了不要过长时间等待已经关闭的redis服务。并且试着获取下一个redis实例。
>
>   ```
>   比如：TTL为5s，设置获取锁最多1s，所以如果1s内无法获取锁，就放弃获取这个锁，从而尝试获取下个锁。
>   ```
>
>3. client通过获取所有能获取的锁后的时间减去第1步的时间，这个时间要小于TTL时间并且至少有3个redis实例成功获取锁，才算真正的获取锁成功。
>
>4. 如果成功获取锁，则锁的真正有效时间是TTL减去第3步的时间差。
>
>   ```
>   比如：TTL是5s，获取锁用了2s，则真正有效时间为3s
>   ```
>
>5. 如果client由于某些原因获取锁失败，便会开始解锁所有redis实例。

> redisson maven依赖：
>
> ```xml
> <dependency>
>     <groupId>org.redisson</groupId>
>     <artifactId>redisson</artifactId>
>     <version>3.11.5</version>
> </dependency>
> ```
>
> 































































































