> JVM在进行GC时，并非每次都对上面三个内存区域（新生代，老生代；方法区）一起回收的，大部分时候回收的都是指新生代。针对Hotspot VM的实现，它里面的GC按照回收区域又分为两大种类型：
>
> - ==部分收集（Partial GC）==
> - ==整堆收集（Full GC）==

> **部分收集**：不是完整收集整个Java堆的垃圾收集。其中又分为：
>
> - ==新生代收集（Minor GC/Young GC）==
> - ==老年代收集（Major GC/Old GC）==
>   - 目前，只有CMS GC会有单独收集老年代的行为。
>   - 注意，很多时候Major GC会和Full GC混淆使用，需要具体分辨是老年代回收还是整堆回收。
> - ==混合收集（Mixed GC）==
>   - 目前，只有G1 GC会有这种行为

> **整堆收集**：收集整个java堆和方法区的垃圾收集。

> Tips：Major GC 和 Full GC出现STW的时间，是Minor GC的10倍以上。

#### 1. Minor GC

当年轻代空间不足时，就会触发Minor GC，这里的年轻代指的是==Eden满==，==Survivor满不会引发GC==。（每次Minor GC会清理年轻代的内存。）因为Java对象大多都具备朝生夕灭的特性，所以Minor GC非常频繁，一般回收速度也比较快。

Minor GC会引发==STW==，暂停其它用户的线程，等垃圾回收结束，用户线程才恢复运行

> STW：stop the word，在执行垃圾收集算法时，Java应用程序的其他所有线程都被挂起（除了垃圾收集帮助器之外）。Java中一种全局暂停现象，全局停顿，所有Java代码停止，native代码可以执行，但不能与JVM交互；这些现象多半是由于gc引起。

#### 2. Major GC

指发生在老年代的GC，==对象从老年代消失时，我们说 “Major GC” 或 “Full GC” 发生了==。出现了MajorGc，经常会伴随至少一次的Minor GC（但非绝对的，在Parallel Scavenge收集器的收集策略里就有直接进行Major GC的策略选择过程）

- 也就是在老年代空间不足时，会先尝试触发Minor GC。如果之后空间还不足，则触发Major GC。

> Tips：==Major GC的速度一般会比Minor GC慢10倍以上，STW的时间更长，如果Major GC后，内存还不足，就报OOM了==。

#### 3. Full GC

触发Full GC执行的情况有如下五种：

- 调用`System.gc()`时，系统建议执行Full GC，但是不必然执行。
- 老年代空间不足
- 方法区空间不足
- 通过Minor GC后进入老年代的平均大小大于老年代的可用内存
- 由Eden区、survivor space0（From Space）区向survivor space1（To Space）区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小。

说明：Full GC 是开发或调优中尽量要避免的。这样暂时时间会短一些。

#### 4. GC举例

> 不断地创建字符串存放在堆区元空间中，设置JVM启动参数：
>
> `-Xms10m -Xmx10m -XX:+PrintGCDetails`
>
> 打印日志：
>
> [GC (Allocation Failure分配失败) [PSYoungGen: 2048K->480K(2560K)] 2048K->609K(9728K), 0.0015849 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
> [GC (Allocation Failure) [PSYoungGen: 2169K->496K(2560K)] 2298K->1407K(9728K), 0.0016570 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
> [GC (Allocation Failure) [PSYoungGen: 2072K->480K(2560K)] 2983K->2567K(9728K), 0.0015637 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
> [Full GC (Ergonomics) [PSYoungGen: 2097K->0K(2560K)] [ParOldGen: 6695K->5031K(7168K)] 8792K->5031K(9728K), [Metaspace: 3009K->3009K(1056768K)], 0.0047474 secs] [Times: user=0.03 sys=0.00, real=0.00 secs] 
> [GC (Allocation Failure) [PSYoungGen: 0K->0K(2560K)] 5031K->5031K(9728K), 0.0003983 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
> [Full GC (Allocation Failure) [PSYoungGen: 0K->0K(2560K)] [ParOldGen: 5031K->5015K(7168K)] 5031K->5015K(9728K), [Metaspace: 3009K->3009K(1056768K)], 0.0049934 secs] [Times: user=0.02 sys=0.00, real=0.01 secs] 
> Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
> at java.util.Arrays.copyOf(Arrays.java:3332)
> at java.lang.AbstractStringBuilder.ensureCapacityInternal(AbstractStringBuilder.java:124)
> at java.lang.AbstractStringBuilder.append(AbstractStringBuilder.java:448)
> at java.lang.StringBuilder.append(StringBuilder.java:136)
> at com.chance.jvm.gc.GCTest.main(GCTest.java:21)
> Heap
> PSYoungGen      total 2560K, used 81K [0x00000007bfd00000, 0x00000007c0000000, 0x00000007c0000000)
> eden space 2048K, 3% used [0x00000007bfd00000,0x00000007bfd14780,0x00000007bff00000)
> from space 512K, 0% used [0x00000007bff80000,0x00000007bff80000,0x00000007c0000000)
> to   space 512K, 0% used [0x00000007bff00000,0x00000007bff00000,0x00000007bff80000)
> ParOldGen       total 7168K, used 5015K [0x00000007bf600000, 0x00000007bfd00000, 0x00000007bfd00000)
> object space 7168K, 69% used [0x00000007bf600000,0x00000007bfae5f10,0x00000007bfd00000)
> Metaspace       used 3042K, capacity 4556K, committed 4864K, reserved 1056768K
> class space    used 322K, capacity 392K, committed 512K, reserved 1048576K
>
> 触发OOM的时候，一定是进行了一次Full GC，因为只有在老年代空间不足时候，才会爆出OOM异常。