# 设置或显示环境变量

在shell中执行程序时，shell会提供一组环境变量。export可新增，修改或删除环境变量，供后续执行的程序使用。export的效力仅限于该次登录操作。

```shell
export [-fnp][变量名称]=[变量设置值]
```

- -f 　代表[变量名称]中为函数名称。
- -n 　删除指定的变量。变量实际上并未删除，只是不会输出到后续指令的执行环境中。
- -p 　列出所有的shell赋予程序的环境变量。

`export -p`：列出当前的环境变量值。

`export MYENV=7`：定义环境变量赋值。

# 查看日志

## tail

-n显示行号；相当于nl命令（number of line）

`tail -100f test.log` 实时监控100行日志；

`tail -n 10 test.log` 查询日志尾部最后10行的日志；

`tail -n +10 test.log` 查询10行之后的所有日志；

## head

和tail相反

`head -n 10 test.log` 查询日志文件中的头10行日志；

`head -n -10 test.log` 查询日志文件除了最后10行的其他所有日志；

## cat

`cat -n test.log |grep "debug"` 查询关键字的日志；

`cat -n test.log | grep "debug" | more` 分页打印，通过点击空格键翻页。

使用 >xxx.txt 将其保存到文件中,到时可以拉下这个文件分析

## sed

sed -n '/2014-12-17 16:17:20/,/2014-12-17 16:17:36/p' test.log







# jps

> 是jdk提供的一个==查看当前java进程==的小工具，可以看作是JavaVirtual Machine Process Status Tool的缩写。
>
> 命令格式：**jps [options ] [ hostid ]** 

## 查看当前java进程

```shell
$ jps
```

## 输出主类或者jar的完全路径

```shell
$ jps -l
```

## 输出jvm参数

```shell
$ jps -v
```

## 仅仅显示java进程号

```shell
$ jps -q
```

> 注意：如果需要查看其他机器上的jvm进程，需要在待查看机器上启动jstatd。

# jstat（JVM的统计监测工具）

> JVM statistics Monitoring，用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。
>
> <img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gk0sbb0mgbj30tj09h74l.jpg" style="zoom:80%">
>
> ==堆内存== = 年轻代 + 年老代 + 永久代 + 元数据区
> 年轻代 = Eden区 + 两个Survivor区（From和To）
>
> 命令格式如下：**jstat [-命令选项] [vmid] [间隔时间/毫秒] [查询次数]**

## 垃圾回收统计

> ```shell
> $ jstat -gc pid
> ```
>
> - S0C：第一个幸存区的大小
> - S1C：第二个幸存区的大小
> - S0U：第一个幸存区的使用大小
> - S1U：第二个幸存区的使用大小
> - EC：伊甸园区的大小
> - EU：伊甸园区的使用大小
> - OC：老年代大小
> - OU：老年代使用大小
> - MC：方法区大小
> - MU：方法区使用大小
> - CCSC:压缩类空间大小
> - CCSU:压缩类空间使用大小
> - YGC：年轻代垃圾回收次数
> - YGCT：年轻代垃圾回收消耗时间
> - FGC：老年代垃圾回收次数
> - FGCT：老年代垃圾回收消耗时间
> - GCT：垃圾回收消耗总时间

## 总结垃圾回收统计

> ```shell
> $ jstat -gcutil pid
> ```
>
> S0：幸存1区当前使用比例
> S1：幸存2区当前使用比例
> E：伊甸园区使用比例
> O：老年代使用比例
> M：元数据区使用比例
> CCS：压缩使用比例
> YGC：年轻代垃圾回收次数
> FGC：老年代垃圾回收次数
> FGCT：老年代垃圾回收消耗时间
> GCT：垃圾回收消耗总时间

## 堆内存统计

> ```shell
> $ jstat -gccapacity pid
> ```
>
> NGCMN：新生代最小容量
> NGCMX：新生代最大容量
> NGC：当前新生代容量
> S0C：第一个幸存区大小
> S1C：第二个幸存区的大小
> EC：伊甸园区的大小
> OGCMN：老年代最小容量
> OGCMX：老年代最大容量
> OGC：当前老年代大小
> OC:当前老年代大小
> MCMN:最小元数据容量
> MCMX：最大元数据容量
> MC：当前元数据空间大小
> CCSMN：最小压缩类空间大小
> CCSMX：最大压缩类空间大小
> CCSC：当前压缩类空间大小
> YGC：年轻代gc次数
> FGC：老年代GC次数

## 元数据空间统计

> ```shell
> jstat -gcmetacapacity pid
> ```
>
> MCMN:最小元数据容量
> MCMX：最大元数据容量
> MC：当前元数据空间大小
> CCSMN：最小压缩类空间大小
> CCSMX：最大压缩类空间大小
> CCSC：当前压缩类空间大小
> YGC：年轻代垃圾回收次数
> FGC：老年代垃圾回收次数
> FGCT：老年代垃圾回收消耗时间
> GCT：垃圾回收消耗总时间

## 新生代/老年代内存空间统计

> ```shell
> $ jstat -gcnewcapacity pid 
> ```
>
> ```shell
> $ jstat -gcoldcapacity pid
> ```

## 查看JVM内存使用详情

> jmap命令是一个可以输出所有内存中对象的工具，甚至可以将VM 中的heap，以二进制输出成文本。打印出某个java进程（使用pid）内存内的，所有‘对象’的情况（如：产生那些对象，及其数量）。
>
> 命令格式：
>
> jmap [option] <pid>	连接到正在运行的进程
>
> jmap [option] <executable <core>	连接到核心文件
>
> jmap [option] [server_id@]<remote server IP or hostname>	连接到远程调试服务

> pid:    目标进程的PID，进程编号，可以采用ps -ef | grep java 查看java进程的PID;
> executable:     产生core dump的java可执行程序;
> core:     将被打印信息的core dump文件;
> remote-hostname-or-IP:     远程debug服务的主机名或ip;
> server-id:     唯一id,假如一台主机上多个远程debug服务;

## 使用hprof二进制形式，输出jvm的heap内容到文件xxx

> ```shell
> $ jmap -dump:live,format=b,file=myjmapfile.txt 19570
> ```
>
> live子选项是可选的，假如指定live选项,那么只输出活的对象到文件。

## 打印正等候回收的对象的信息

> ```shell
> $ jmap -finalizerinfo 3772
> ```

## 打印heap的概要信息，GC使用的算法，heap（堆）的配置及JVM堆内存的使用情况

> ```shell
> $ jmap -heap 19570
> ```

## 打印每个class的实例数目,内存占用,类全名信息

> ```shell
> $ jmap -histo:live 19570
> ```
>
> VM的内部类名字开头会加上前缀”*”。 如果live子参数加上后,只统计活的对象数量。
>
> 采用`jmap -histo pid>a.log`日志将其保存，在一段时间后，使用文本对比工具，可以对比出GC回收了哪些对象。
>
> jmap -dump:format=b,file=outfile 3024可以将3024进程的内存heap输出出来到outfile文件里，再配合MAT（内存分析工具）。

## 打印classload和jvm heap长久层的信息

> ```shell
> $ jmap -permstat 19570
> ```
>
> 包含每个classloader的名字，活泼性，地址，父classloader和加载的class数量。 另外，内部String的数量和占用内存数也会打印出来。





















