<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gmjtnuhsphj30hw0jo76n.jpg" style="zoom:50%">

### 1. 为什么Redis是单线程的

#### 结论一：Redis并不是纯粹的单进程单线程。

>如下例子：
>
>```
>127.0.0.1:6379> bgsave
>Background saving started
>```
>
>Redis两种持久化方式：RDB、AOF。
>
>拿RDB举例，执行`bgsave`，就意味着fork出一个子进程在后台进行备份。这就是为什么执行完bgsave命令之后，还能对该Redis实例继续其他操作。

#### 结论二：单线程指的是网络请求模块使用了一个线程（所以不考虑并发安全性），即一个线程处理所有网络请求，其他模块仍用了多个线程。

>为什么Redis以单线程的方式处理数据性能会比较高？
>
>**（1）<u>多线程并不一定意味着快</u>**
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gmjs80pnl0j30mx0ci0tm.jpg" style="zoom:100%">
>
>- 一个CPU处理多个请求会导致单个请求响应时间过长；
>
>- 而且==频繁切换线程上下文==会增加性能损耗。

>**（2）<u>那为什么还要使用多线程？</u>**
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gmjse9qwz9j30na0ebgmn.jpg" style="zoom:100%">
>
>大多数任务中都会涉及到网络请求、IO读取（包括文件、数据库）等操作，这些都会产生一定的==阻塞问题==，那么这时候多线程就产生作用了。
>
>- 当线程-3发生阻塞时，如果不进行切换，那就是白白占用CPU资源，此时CPU是空闲的；
>- 反之，切换到其他线程，CPU则执行其他线程的任务，提高CPU利用率。

#### 总结

>官方答案：
>
>```
>Redis是基于内存的操作，CPU不是Redis的瓶颈，Redis的瓶颈最有可能是机器内存的大小或者网络带宽。
>```
>
>详细原因：
>
>==1）不需要各种锁的性能消耗==
>Redis的数据结构并不全是简单的Key-Value，还有list，hash等复杂的结构，这些结构有可能会进行很细粒度的操作，比如在很长的列表后面添加一个元素，在hash当中添加或者删除一个对象。这些操作可能就需要加非常多的锁，导致的结果是同步开销大大增加。
>
>总之，在单线程的情况下，就不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗。
>
>==2）单线程多进程集群方案==
>
>==3）CPU消耗==
>
>采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗 CPU。
>
>但是如果CPU成为Redis瓶颈，或者不想让服务器其他CUP核闲置，那怎么办？
>
>可以考虑多起几个Redis进程，Redis是key-value数据库，不是关系数据库，数据之间没有约束。只要客户端分清哪些key放在哪个Redis进程上就可以了。

### 2. Redis单线程的优势

>优势：
>
>1. 代码更清晰，处理逻辑更简单
>2. 不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗
>3. 不存在多进程或者多线程导致的切换而消耗CPU

>劣势：
>
>1. 无法发挥多核CPU性能，不过可以通过在单机开多个Redis实例来完善。

### 3. IO多路复用技术

>redis 采用网络IO多路复用技术来保证在多连接的时候，系统的高吞吐量。
>
>==多路-指的是多个socket连接，复用-指的是复用一个线程==。多路复用主要有三种技术：select，poll，epoll。`epoll`是最新的也是目前最好的多路复用技术。
>
>采用多路 I/O 复用技术可以让单个线程高效的处理多个连接请求（尽量减少网络IO的时间消耗），且Redis在内存中操作数据的速度非常快（内存内的操作不会成为这里的性能瓶颈），主要以上两点造就了Redis具有很高的吞吐量。

### 4. 总结

>1. Redis是纯内存数据库，一般都是简单的存取操作，线程占用的时间很多，时间的花费主要集中在IO上，所以读取速度快。
>2. 再说一下IO，Redis使用的是非阻塞IO，IO多路复用，使用了单线程来轮询描述符，将数据库的开、关、读、写都转换成了事件，减少了线程切换时上下文的切换和竞争。
>3. Redis采用了单线程的模型，保证了每个操作的原子性，也减少了线程的上下文切换和竞争。
>4. 另外，数据结构也帮了不少忙，Redis全程使用hash结构，读取速度快，还有一些特殊的数据结构，对数据存储进行了优化，如压缩表，对短数据进行压缩存储，再如，跳表，使用有序的数据结构加快读取的速度。
>5. 还有一点，Redis采用自己实现的事件分离器，效率比较高，内部采用非阻塞的执行方式，吞吐能力比较大。

