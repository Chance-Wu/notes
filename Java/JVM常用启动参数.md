在一个java应用启动时，我们可以配置其jvm的启动参数，如：

```sh
java -jar -Xms4096M -Xmx4096M -Xmn1024M -Xss256K hello.jar
```

#### 1. 堆大小设置（堆=年轻代+年老代+持久代）

---

>- **-Xmx**4096m：设置JVM最大可用内存为4096m。（memory max）
>- **-Xms**4096m：设置JVM初始内存为4096m。（memory start）此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。

- -Xss128k：设置每个线程的堆栈大小。（JDK5.0以后每个线程堆栈大小为1M）
- -Xmn1024m：设置年轻代大小为1024m。等效于同时配置下面两个。
  - -XX:NewSize=1024m：设置年轻代初始值为1024M。
  - -XX:MaxNewSize=1024m：设置年轻代最大值为1024M。

#### 2. 垃圾回收器设置（串行收集器、并行收集器、并发收集器）

---

默认情况下JDK5.0以前都是使用串行收集器，如果想使用其他收集器需要在启动时加入相应参数。JDK5.0以后，JVM会根据当前系统配置进行智能判断。

**串行收集器**

- -XX:+UseSerialGC：设置串行收集器。

　　**并行收集器**（吞吐量优先）

- -XX:+UseParallelGC：设置年轻代为并行收集器。（此时年老代仍然为串行）
- -XX:+UseParallelOldGC：配置年老代为并行收集。
- -XX:ParallelGCThreads=20：配置并行收集器的线程数。
- -XX:MaxGCPauseMillis=100：设置每次年轻代垃圾回收的最长时间（单位毫秒）。如果无法满足此时间，JVM会自动调整年轻代大小，以满足此时间。
- -XX:+UseAdaptiveSizePolicy：设置此选项后，并行收集器会自动调整年轻代Eden区大小和Survivor区大小的比例，以达成目标系统规定的最低响应时间或者收集频率等指标。此参数建议在使用并行收集器时，一直打开。

　　**并发收集器**（响应时间优先）

- **-XX:+UseConcMarkSweepGC：**即CMS收集，设置年老代为并发收集。
- -XX:+UseParNewGC：设置年轻代为并发收集。JDK5.0以上JVM会自行设置，无需再设。
- -XX:CMSFullGCsBeforeCompaction=0：每次Full GC后立刻开始压缩和整理内存。
- -XX:+UseCMSCompactAtFullCollection：打开内存空间的压缩和整理，在Full GC后执行。
- -XX:+CMSIncrementalMode：设置为增量收集模式。一般适用于单CPU情况。
- -XX:CMSInitiatingOccupancyFraction=70：表示年老代内存空间使用到70%时就开始执行CMS收集，以确保年老代有足够的空间接纳来自年轻代的对象，避免Full GC的发生。

　　**其它垃圾回收参数**

- -XX:+ScavengeBeforeFullGC：年轻代GC优于Full GC执行。
- **-XX:-DisableExplicitGC：**不响应 System.gc() 代码。
- -XX:+UseThreadPriorities：启用本地线程优先级API。即使 java.lang.Thread.setPriority() 生效，不启用则无效。
- -XX:SoftRefLRUPolicyMSPerMB=0：软引用对象在最后一次被访问后能存活0毫秒（JVM默认为1000毫秒）。
- -XX:TargetSurvivorRatio=90：允许90%的Survivor区被占用（JVM默认为50%）。提高对于Survivor区的使用率。

#### 3. 辅助信息参数设置

---

- -XX:-CITime：打印消耗在JIT编译的时间。
- -XX:ErrorFile=./hs_err_pid.log：保存错误日志或数据到指定文件中。
- -XX:HeapDumpPath=./java_pid.hprof：指定Dump堆内存时的路径。
- -XX:-HeapDumpOnOutOfMemoryError：当首次遭遇内存溢出时Dump出此时的堆内存。
- -XX:OnError=";"：出现致命ERROR后运行自定义命令。
- -XX:OnOutOfMemoryError=";"：当首次遭遇内存溢出时执行自定义命令。
- -XX:-PrintClassHistogram：按下 Ctrl+Break 后打印堆内存中类实例的柱状信息，同JDK的 jmap -histo 命令。
- -XX:-PrintConcurrentLocks：按下 Ctrl+Break 后打印线程栈中并发锁的相关信息，同JDK的 jstack -l 命令。
- -XX:-PrintCompilation：当一个方法被编译时打印相关信息。
- -XX:-PrintGC：每次GC时打印相关信息。
- **-XX:-PrintGCDetails：**每次GC时打印详细信息。
- -XX:-PrintGCTimeStamps：打印每次GC的时间戳。
- -XX:-TraceClassLoading：跟踪类的加载信息。
- -XX:-TraceClassLoadingPreorder：跟踪被引用到的所有类的加载信息。
- -XX:-TraceClassResolution：跟踪常量池。
- -XX:-TraceClassUnloading：跟踪类的卸载信息。