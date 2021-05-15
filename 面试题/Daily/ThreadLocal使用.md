>官方解释：
>
>“该类提供了==线程局部变量==。这些变量不同于它们的普通对应物，因为访问某个变量的每个线程都有自己的局部变量，它独立于变量的初始化副本。==ThreadLocal 实例通常是类中的 private static 字段==，它们希望将状态与某一个线程（例如，用户 ID 或事务 ID）相关联。”

>即ThreadLocal类允许我们创建只能被同一个线程读写的变量。

#### 1. 内部结合

>1. 每个Thread线程内部都有一个Map（ThreadLocalMap）。
>2. Map里面存储==ThreadLocal对象==（key）和==线程的变量副本==（value）。
>3. Thread内部的Map是由ThreadLocal维护的，由ThreadLocal负责向map获取和设置线程的变量值。
>4. 对应不同的线程，每次获取副本值时，别的线程并不能获取到当前线程的副本值，形成了副本的隔离，互不干扰。
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gmxy73d7jnj30m10m8ab2.jpg" style="zoom:100%">
>
>ThreadLocal内部都有一个Map来存储当前线程的共享变量，只能当前线程访问，从而保证不会影响其他线程的变量。

#### 2. 源码分析

>ThreadLocal主要方法：
>
>```java
>public T get(); // 方法用于获取当前线程的副本变量值。
>public void set(T value); // 方法用于保存当前线程的副本变量值。
>protected T initialValue(); // 为当前线程初始副本变量值。
>public void remove(); // 方法移除当前前程的副本变量值。
>```

##### 2.1 initialValue方法

>为变量设置初始值，默认实现如下：
>
>```java
>protected T initialValue() {
>    return null;
>}
>```
>
>如果想要给该变量设置一个初始值，只需重写方法即可，例如：
>
>```java
>@Override
>protecgted String initialValue() {
>    return "hello world";
>}
>```
>
>`ThreadLocal`对象中很多方法的实现会调用该方法来设置初始值。

##### 2.2 get方法分析

>了解ThreadLocal的运作机制，可以从get方法作为切入点分析：
>
>```java
>public T get() {
>    Thread t = Thread.currentThread();
>    // 获取当前线程对象t对应的ThreadLocalMap
>    ThreadLocalMap map = getMap(t);
>    if (map != null) {
>        // 以当前ThreadLocal对象为键查找值
>        ThreadLocalMap.Entry e = map.getEntry(this);
>        if (e != null) {
>            @SuppressWarnings("unchecked")
>            T result = (T)e.value;
>            return result;
>        }
>    }
>    return setInitialValue();
>}
>```
>
>1. 首先获取当前线程对象，根据该线程对象找到对应的ThreadLocalMap。
>2. 如果找到了该线程对象的ThreadLocalMap，则通过当前ThreadLocal对象作为键查找Map中对应的Entry（键值对）对象，如果查找结果不为null，则返回Entry对象的value。
>3. 否则调用setInitialValue()方法将当前ThreadLocal对象(this)和变量作为键值对存入ThreadlocalMap并返回变量。
>
>==每个`Thread`对象会持有一个`ThreadLocalMap`，当我们创建一个ThreadLocal对象时，该ThreadLocal对象本身作为键，变量作为值存入该ThreadLocalMap中，来实现线程的私有变量。==

##### 2.3 setInitialValue方法实现

>```java
>private T setInitialValue() {
>    T value = initialValue();
>    Thread t = Thread.currentThread();
>    ThreadLocalMap map = getMap(t);
>    if (map != null)
>        map.set(this, value);
>    else
>        createMap(t, value);
>    return value;
>}
>
>void createMap(Thread t, T firstValue) {
>    t.threadLocals = new ThreadLocalMap(this, firstValue);
>}
>```
>
>1. 首先调用initialValue方法初始化变量，
>2. 然后获取当前Thread对象所属的ThreadLocalMap（即线程的成员变量threadLocals），
>3. 如果threadLocals不为null，则将当前ThreadLocal对象及其变量存入Map，否则会调用createMap方法为当前Thread对象新建一个ThreadLocalMap并存入当前ThreadLocal对象及其变量。

##### 2.4 ThreadLocal内部类：ThreadLocalMap

>ThreadLocalMap负责存储==ThreadLocal==及其==变量==。
>
>ThreadLocalMap持有一个内部类Entry，类似于HashMap.Node类，负责保存键值对。
>
>```java
>static class Entry extends WeakReference<ThreadLocal<?>> {
>    /** 与此ThreadLocal关联的值 */
>    Object value;
>
>    Entry(ThreadLocal<?> k, Object v) {
>        super(k);
>        value = v;
>    }
>}
>```

>**为什么Entry要继承WeakReference？**
>
>```
>WeakReference标志性的特点是：reference实例不会影响到被应用对象的GC回收行为（即只要对象被除WeakReference对象之外所有的对象解除引用后，该对象便可以被GC回收），只不过在被对象回收之后，reference实例想获得被引用的对象时程序会返回null。
>```
>
>既然ThreadLocal声明为弱引用，自然会联想到和GC有关。
>
>- 如果是强引用的话，在线程运行过程中，我们不再使用threadLocal对象了，将其置为null，但threadLocal在Thread对象成员变量里还有引用，导致其无法被GC回收，引发内存泄漏。
>- 而Entry声明为WeakReference，userInfoLocal置为null后，线程的threadLocalMap就不算强引用了，userInfoLocal就可以被GC回收了。

#### 3. 使用场景

>- 保存线程上下文信息，在任意需要的地方可以获取。
>- 线程安全的，避免某些情况需要考虑线程安全必须同步带来的性能损失。

##### 3.1 保存线程上下文

>比如Spring的事务管理，用ThreadLocal存储Connection，从而各个DAO可以获取同一Connection，可以进行事务回滚，提交等操作。

##### 3.2 避免某些情况需要考虑线程安全必须同步带来的性能损失

>每个线程在ThreadLocal中读写数据是线程隔离，互相之间不会影响，由于不需要共享信息，自然就不存在竞争问题了，从而保证了某些情况下线程的安全，以及避免了某些情况需要考虑线程安全必须同步带来的性能损失。

