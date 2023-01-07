### 前言

---

并发编程是为了让程序运行得更快，但是，并不是启动的线程越多就能让程序需大幅度的并发执行。实际中会面临许多问题，比如上下文切换问题、死锁问题，以及受限于硬件和软件资源问题

#### 上下文切换

时间片是CPU分给各个线程的时间，因为时间片非常短，所以CPU将会在各个线程之间来回切换从而让用户感觉多个程序是同时执行的。CPU通过时间片分配算法来循环执行任务，因此需要在各个线程之间进行切换。**从任务的保存到加载过程就称作“上下文切换”**。上下文切换是需要系统开销的。

减少上下文切换的措施：

- 无锁并发编程：多线程竞争锁时，会引起上下文切换，所以**多线程处理数据时，可以用一些方法来避免使用锁，如将数据的ID按照Hash算法取模分段，不同线程处理不同段的数据**。
- CAS算法：Java的Atomic包使用CAS算法来更新数据，不需要加锁。
- 使用最少的线程来完成任务：避免不需要的线程。
- 协程：在单线程里实现多任务的调度，并在单线程里维持多个任务间的切换。

#### 死锁

死锁就是两个或者两个以上的线程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象。

死锁产生的四个必要性：

- 互斥条件
- 不可抢占条件
- 请求和保持条件
- 循环等待条件

避免死锁的几个常见方法：

- 避免获取同一个锁。
- 避免一个线程在锁内同时占用多个资源，尽量保证每个锁只保持一个资源。
- 尝试使用定时锁，**使用tryLock(timeout)来代替使用内部锁机制**。
- 对于数据库锁，加锁和解锁必须在同一个数据库连接中，否则会出现解锁失败的问题。































`volatile`用于标记一个变量“**从主存读写**”。

即每次读取volatile变量，都应该==从主存读取==，而不是从CPU缓存读取。每写入一个volatile变量，应该==写到主存==，而不仅仅写到CPU缓存。

#### 1. 变量可见性问题

>volatile能保证变量修改后，==对各个线程是可见的==。
>
>在一个多线程应用中，线程在操作一个非volatile变量时，处于性能考虑，每个线程可能会将变量从主存拷贝到CPU缓存中。如果有多个CPU，每个线程可能会在不同的CPU中运行。这意味着，每个线程都有可能把变量拷贝到各自CPU的缓存中。
>
><img src="img/008eGmZEgy1gmvp28bnm7j30dy0br0ss.jpg" style="zoom:80%">
>
>JVM并不保证会从主存中读取数据到CPU缓存，或者将CPU缓存中的数据写到主存中。

>假如两个以上的线程访问一个共享对象，这个共享对象包含一个counter变量。
>
>```java
>public class SharedObject {
>
>    public int counter = 0;
>}
>```
>
>- 只有线程1修改了counter变量；
>- 线程1和线程2两个线程都会在某些时刻读取counter变量。
>
>这种没有声明为volatile的情况下，counter的值不保证会从CUP缓存写回到主存中。也就是说，CPU缓存和主存中的counter变量值并不一致。
>
><img src="img/008eGmZEgy1gmvpcpklhej30d00b9jrl.jpg" style="zoom:80%">

#### 2. 可见性保证

>将上述的共享变量counter声明为volatile，则在写入counter变量时，也会同时将变量值写入到主存中。同样，也会直接从主存读取数据。
>
>```java
>public class SharedObject {
>    public volatile int counter = 0;
>}
>```
>
>在上面的场景中，一个线程（T1）修改了counter，另一个线程（T2）读取了counter（但没有修改它），将counter变量声明为volatile，就能保证写入counter变量后，对T2是可见的。然而，==如果T1和T2都修改了counter的值，只是将counter声明为volatile还远远不够==，后面会有更多的说明。

#### 3. 完整volatile可见性保证

>```java
>public class MyClass {
>    private int years;
>    private int months
>        private volatile int days;
>
>
>    public void update(int years, int months, int days){
>        this.years  = years;
>        this.months = months;
>        this.days   = days;
>    }
>}
>```
>
>update()方法写入3个变量，其中只有days变量是volatile。
>
>完整的volatile可见性保证意味着，在写入days变量时，线程中所有可见变量也会写入到主存。也就是说，写入days变量时，years和months也会同时被写入到主存。

>可以==将对volatile变量的读写理解为一个触发刷新的操作==，写入volatile变量时，线程中的所有变量也都会触发写入主存。而读取volatile变量时，也同样会触发线程中所有变量从主存中重新读取。因此，应当尽量将volatile的写入操作放在最后，而将volatile的读取放在最前，这样就能连带将其他变量也进行刷新。

#### 4. 指令重排

>JVM和CPU是允许对程序中的指令进行重排的，只要保证（重排后的）指令语义一致即可。
>
>```java
>public class MyClass {
>    private int years;
>    private int months
>        private volatile int days;
>
>
>    public void update(int years, int months, int days){
>        this.years  = years;
>        this.months = months;
>        this.days   = days;
>    }
>}
>```
>
>一旦update()变量写了days值，则years、months的最新值也会写入主存。但是，如果指令重排了：
>
>```java
>public void update(int years, int months, int days){
>    this.days   = days;
>    this.months = months;
>    this.years  = years;
>}
>```
>
>这样就导致了month、years写入主存的不是最新值。

##### 4.1 解决指令重排的问题 Happens-Before

>原则如下：
>
>- 如果有读写操作发生在写入volatile变量之前，读写其他变量的指令不能重排到写入volatile变量之后。写入一个volatile变量之前的读写操作，对volatile变量是有happens-before保证的。注意，如果是写入volatile之后，有读写其他变量的操作，那么这些操作指令是有可能被重排到写入volatile操作指令之前的。但反之则不成立。即可以把位于写入volatile操作指令之后的其他指令移到写入volatile操作指令之前，而不能把位于写入volatile操作指令之前的其他指令移到写入volatile操作指令之后。
>- 如果有读写操作发生在读取volatile变量之后，读写其他变量的指令不能重排到读取volatile变量之前。注意，如果是读取volatile之前，有读取其他变量的操作，那么这些操作指令是有可能被重排到读取volatile操作指令之后的。但反之则不成立。即可以把位于读取volatile操作指令之前的指令移到读取volatile操作指令之后，而不能把位于读取volatile操作指令之后的指令移到读取volatile操作指令之前。
>
>最终的意思就是保证了volatile变量的读操作在其他变量之前，写操作在其他变量之后。
>
>完整volatile可见性的保证。

#### 5. volatile的问题

>如果线程需要先读取一个volatile变量的值，以此来计算出一个新的值，那么volatile变量就不足够保证正确的可见性。（线程间）读写volatile变量的时间间隔很短，这将导致一个[竞态条件](http://ifeve.com/race-conditions-and-critical-sections/)，多个线程同时读取了volatile变量相同的值，然后以此计算出了新的值，这时各个线程往主存中写回值，则会互相覆盖。
>
>多个线程对counter变量进行自增操作就是这样的情形。

>设想一下，如果线程1将共享变量counter的值0读取到它的CPU缓存，然后自增为1，而还没有将新值写回到主存。线程2这时从主存中读取的counter值依然是0，依然放到它自身的CPU缓存中，然后同样将counter值自增为1，同样也还没有将新值写回到主存。如下图所示：
>
><img src="img/008eGmZEgy1gmw8dbodyyj30cx0b8aa9.jpg" style="zoom:80%">

#### 6. 何时使用volatile

>1. ==多个线程同时读写一个共享变量==
>
>   这种情况下，只使用volatile变量是不够的。应该使用==synchronized==来==保证读写变量是原子的==。
>
>2. ==一个线程对volatile进行读写，其他线程只是读取变量==
>
>   此时，volatile就可以保证其他线程读取到的值是最新值。

