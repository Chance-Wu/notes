当 JVM 内存严重不足时，就会抛出 java.lang.OutOfMemoryError 错误。

### 一、Java heap space

---

当堆内存（Heap Space）没有足够的空间存放新创建的对象时，就会抛出`java.lang.OutOfMemoryError:Javaheap space` 错误（可以对程序日志中的 OutOfMemoryError 配置关键字告警，一经发现，立即处理）。

#### 1.1 原因分析

常见原因分为以下几类：

1. 请求创建一个超大对象，通常是一个大数组。
2. 超出预期的访问量/数据量，通常是上游系统请求流量飙升，可结合业务流量指标排查是否有尖状峰值。
3. 过渡使用终结器（Finalizer），该对象没有立即被GC。
4. 内存泄漏（Memory Leak），大量对象引用没有释放，JVM无法对其自动回收，常见于使用了File等资源没有回收。

#### 1.2 解决方案

针对大部分情况，通常只需要通过 `-Xmx` 参数调高 JVM 堆内存空间即可。如果仍然没有解决，可以参考以下情况做进一步处理：

1. 如果是超大对象，可以检查其合理性，比如是否一次性查询了数据库全部结果，而没有做结果数限制。
2. 如果是业务峰值压力，可以考虑添加机器资源，或者做限流降级。
3. 如果是内存泄漏，需要找到持有的对象，修改代码设计，比如关闭没有释放的连接。



### 二、GC overhead limit exceeded

---

当 Java 进程花费 98% 以上的时间执行 GC，但只恢复了不到 2% 的内存，且该动作连续重复了 5 次，就会抛出 `java.lang.OutOfMemoryError:GC overhead limit exceeded` 错误。简单地说，就是应用程序已经基本耗尽了所有可用内存， GC 也无法回收。

此类问题的原因与解决方案跟 `Java heap space` 非常类似。



### 三、Permgen space

---

该错误表示永久代（Permanent Generation）已用满，通常是因为**加载的 class 数目太多或体积太大**。

#### 3.1 原因分析

永久代存储对象主要包括以下几类：

1. 加载/缓存到内存中的 class 定义，包括类的名称，字段，方法和字节码；
2. 常量池；
3. 对象数组/类型数组所关联的 class；
4. JIT 编译器优化后的 class 信息。

PermGen 的使用量与加载到内存的 class 的数量/大小正相关。

#### 3.2 解决方案

根据 Permgen space 报错的时机，可以采用不同的解决方案，如下所示：

1. 程序启动报错，修改 `-XX:MaxPermSize` 启动参数，调大永久代空间。
1. 应用重新部署时报错，很可能是应用没有重启，导致加载了多份 class 信息，只需重启 JVM 即可解决。
1. 运行时报错，应用程序可能会动态创建大量 class，而这些 class 的生命周期很短暂，但是 **JVM 默认不会卸载 class**，可以设置 `-XX:+CMSClassUnloadingEnabled` 和 `-XX:+UseConcMarkSweepGC`这两个参数允许 JVM 卸载 class。

> 如果上述方法无法解决，可以通过 jmap 命令 dump 内存对象 `jmap -dump:format=b,file=dump.hprof process-id` ，然后利用 Eclipse MAT 功能逐一分析开销最大的 classloader 和重复 class。



### 四、Metaspace

---

JDK 1.8 使用 Metaspace 替换了永久代（Permanent Generation），该错误表示 Metaspace 已被用满，通常是因为加载的 class 数目太多或体积太大。

此类问题的原因与解决方法跟 `Permgenspace` 非常类似。需要特别注意的是调整 Metaspace 空间大小的启动参数为 `-XX:MaxMetaspaceSize`。



### 五、Unable to create new native thread

---

每个 Java 线程都需要占用一定的内存空间，当 JVM 向底层操作系统请求创建一个新的 native 线程时，如果没有足够的资源分配就会报此类错误。

#### 5.1 原因分析

常见的原因包括以下几类：

1. 线程数超过操作系统最大线程数 ulimit 限制；
2. 线程数超过 kernel.pid_max（只能重启）；
3. native 内存不足。

该问题发生的常见过程主要包括以下几步：

1、JVM 内部的应用程序请求创建一个新的 Java 线程；

2、JVM native 方法代理了该次请求，并向操作系统请求创建一个 native 线程；

3、操作系统尝试创建一个新的 native 线程，并为其分配内存；

4、如果操作系统的虚拟内存已耗尽，或是受到 32 位进程的地址空间限制，操作系统就会拒绝本次 native 内存分配；

5、JVM 将抛出 `java.lang.OutOfMemoryError:Unableto createnewnativethread` 错误。

#### 5.2 解决方案

1. 升级配置，为机器提供更多的内存；
2. 降低 Java Heap Space 大小；
3. 修复应用程序的线程泄漏问题；
4. 限制线程池大小；
5. 使用 -Xss 参数减少线程栈的大小；
6. 调高 OS 层面的线程最大数：执行 `ulimit -a` 查看最大线程数限制，使用 `ulimit -u xxx` 调整最大线程数限制。



### 六、Out of swap space

---

该错误表示可用的虚拟内存已被耗尽。虚拟内存（Virtual Memory）由物理内存（Physical Memory）和交换空间（Swap Space）两部分组成。当运行时程序请求的虚拟内存溢出时就会报 `Outof swap space?` 错误。

































