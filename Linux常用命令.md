### kill

---

用于删除执行中的程序或工作。

kill 可将指定的信息送至程序。预设的信息为 SIGTERM(15)，可将指定程序终止。若仍无法终止该程序，可使用 SIGKILL(9) 信息尝试强制删除程序。程序或工作的编号可利用 ps 指令或 jobs 指令查看。

```shell
kill [-s <信息名称或编号>][程序]　或　kill [-l <信息编号>]
```

- -l <信息编号> 　若不加<信息编号>选项，则 -l 参数会列出全部的信息名称。
- -s <信息名称或编号> 　指定要送出的信息。
- [程序] 　[程序]可以是程序的PID或是PGID，也可以是工作编号。

```shell
$ kill -l
HUP INT QUIT ILL TRAP ABRT EMT FPE KILL BUS SEGV SYS PIPE ALRM TERM URG STOP TSTP CONT CHLD TTIN TTOU IO XCPU XFSZ VTALRM PROF WINCH INFO USR1 USR2
```

最常用的信号是：

- 1 (HUP)：重新加载进程。
- 9 (KILL)：杀死一个进程。
- 15 (TERM)：正常停止一个进程。

>linux 的 **kill** 命令是向进程发送信号，**kill** 不是杀死的意思，**-9** 表示无条件退出，但由进程自行决定是否退出，这就是为什么 **kill -9** 终止不了系统进程和守护进程的原因。



### more

---

类似 cat ，不过会**以一页一页的形式显示**，而最基本的指令就是按空白键（space）就往下一页显示，按 b 键就会往回（back）一页显示，而且还有搜寻字串的功能（与 vi 相似），使用中的说明文件，请按 h 。



### nohup /root/test.sh > test.log 2>&1 &

---

#### nohup

在系统后台不挂断地运行命令，退出终端不会影响程序的运行。在默认情况下（非重定向时），会输出一个名叫 `nohup.out` 的文件到当前目录下，如果当前目录的 nohup.out 文件不可写，输出重定向到 $HOME/nohup.out 文件中。

#### &

在命令的后面加一个&的作用是，将这个任务放到后台执行。会显示进程ID，可以用来暂停、恢复或者终止对应的进程。

#### 2>&1

将标准错误 2 重定向到标准输出 &1。标准输出 &1 再被重定向输入到 test.log 文件中。

- 0 – stdin (standard input，标准输入)
- 1 – stdout (standard output，标准输出)
- 2 – stderr (standard error，标准错误输出)



### jobs

---

显示Linux中的任务列表及任务状态，包括后台运行的任务。该命令可以显示任务号及其对应的进程号。

```
-l：显示进程号；
-p：仅任务对应的显示进程号；
-n：显示任务状态的变化；
-r：仅输出运行状态（running）的任务；
-s：仅输出停止状态（stoped）的任务。
```



### 查看进程下的存活线程

---

#### pstree -p 进程id

查看进程树之间的关系。

```shell
chenyang@chances-MacBook-Pro ~ % pstree -p 410
-+= 00001 root /sbin/launchd
 \-+= 00410 chenyang /Applications/Google Chrome.app/Contents/MacOS/Google Chro
   |--- 00469 chenyang /Applications/Google Chrome.app/Contents/Frameworks/Goog
   |--- 00475 chenyang /Applications/Google Chrome.app/Contents/Frameworks/Goog
```

可以看出每个进程的pid，所属用户。

#### cat /proc/进程id/status

```shell
cat /proc/27402/status
```

#### Top -p 进程id，然后按H

比如某台服务器的CPU使用率飙升，通过top命令查看是gitlab程序占用的cpu比较大，"ps -ef|grep gitlab"发现有很多个gitlab程序，现在需要查询gitlab各个进程下的线程数情况。批量查询命令如下：

```shell
# for pid in $(ps -ef|grep -v grep|grep gitlab|awk '{print $2}');do echo ${pid} > /root/a.txt ;cat /proc/${pid}/status|grep Threads > /root/b.txt;paste /root/a.txt /root/b.txt;done|sort -k3 -rn
```

1. `for pid in $(ps -ef|grep -v grep|grep gitlab|awk '{print $2}')`
2. 定义${pid}变量为gitlab进程的pid号
3. echo ${pid} > /root/a.txt
4. 将http进程的pid号都打印到/root/a.txt文件中
5. cat /proc/${pid}/status|grep Threads > /root/b.txt
6. 将各个pid进程号下的线程信息打印到/root/b.txt文件中
7. `paste /root/a.txt /root/b.txt`
8. 以列的形式展示a.txt和b/txt文件中的信息
9. sort -k3 -rn
   1. -k3 表示以第三列进行排序
   2. -rn 表示降序



### 设置或显示环境变量

---

在shell中执行程序时，shell会提供一组环境变量。export可新增，修改或删除环境变量，供后续执行的程序使用。export的效力仅限于该次登录操作。

```shell
export [-fnp][变量名称]=[变量设置值]
```

- -f 　代表[变量名称]中为函数名称。
- -n 　删除指定的变量。变量实际上并未删除，只是不会输出到后续指令的执行环境中。
- -p 　列出所有的shell赋予程序的环境变量。

`export -p`：列出当前的环境变量值。

`export MYENV=7`：定义环境变量赋值。

### 查看日志

#### tail

-n显示行号；相当于nl命令（number of line）

`tail -100f test.log` 实时监控100行日志；

`tail -n 10 test.log` 查询日志尾部最后10行的日志；

`tail -n +10 test.log` 查询10行之后的所有日志；

#### head

和tail相反

`head -n 10 test.log` 查询日志文件中的头10行日志；

`head -n -10 test.log` 查询日志文件除了最后10行的其他所有日志；

#### cat

`cat -n test.log |grep "debug"` 查询关键字的日志；

`cat -n test.log | grep "debug" | more` 分页打印，通过点击空格键翻页。

使用 >xxx.txt 将其保存到文件中,到时可以拉下这个文件分析

#### sed

sed -n '/2014-12-17 16:17:20/,/2014-12-17 16:17:36/p' test.log







### jps

---

> 是jdk提供的一个==查看当前java进程==的小工具，可以看作是JavaVirtual Machine Process Status Tool的缩写。
>
> 命令格式：**jps [options ] [ hostid ]** 

#### 查看当前java进程

```shell
$ jps
```

#### 输出主类或者jar的完全路径

```shell
$ jps -l
```

#### 输出jvm参数

```shell
$ jps -v
```

#### 仅仅显示java进程号

```shell
$ jps -q
```

> 注意：如果需要查看其他机器上的jvm进程，需要在待查看机器上启动jstatd。

### jstat（JVM的统计监测工具）

---

> JVM statistics Monitoring，用于监视虚拟机运行时状态信息的命令，它可以显示出虚拟机进程中的类装载、内存、垃圾收集、JIT编译等运行数据。
>
> <img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gk0sbb0mgbj30tj09h74l.jpg" style="zoom:80%">
>
> ==堆内存== = 年轻代 + 年老代 + 永久代 + 元数据区
> 年轻代 = Eden区 + 两个Survivor区（From和To）
>
> 命令格式如下：**jstat [-命令选项] [vmid] [间隔时间/毫秒] [查询次数]**

#### 垃圾回收统计

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

#### 总结垃圾回收统计

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

#### 堆内存统计

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

#### 元数据空间统计

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

#### 新生代/老年代内存空间统计

> ```shell
> $ jstat -gcnewcapacity pid 
> ```
>
> ```shell
> $ jstat -gcoldcapacity pid
> ```

#### 查看JVM内存使用详情

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

#### 使用hprof二进制形式，输出jvm的heap内容到文件xxx

> ```shell
> $ jmap -dump:live,format=b,file=myjmapfile.txt 19570
> ```
>
> live子选项是可选的，假如指定live选项,那么只输出活的对象到文件。

#### 打印正等候回收的对象的信息

> ```shell
> $ jmap -finalizerinfo 3772
> ```

#### 打印heap的概要信息，GC使用的算法，heap（堆）的配置及JVM堆内存的使用情况

> ```shell
> $ jmap -heap 19570
> ```

#### 打印每个class的实例数目,内存占用,类全名信息

> ```shell
> $ jmap -histo:live 19570
> ```
>
> VM的内部类名字开头会加上前缀”*”。 如果live子参数加上后,只统计活的对象数量。
>
> 采用`jmap -histo pid>a.log`日志将其保存，在一段时间后，使用文本对比工具，可以对比出GC回收了哪些对象。
>
> jmap -dump:format=b,file=outfile 3024可以将3024进程的内存heap输出出来到outfile文件里，再配合MAT（内存分析工具）。

#### 打印classload和jvm heap长久层的信息

> ```shell
> $ jmap -permstat 19570
> ```
>
> 包含每个classloader的名字，活泼性，地址，父classloader和加载的class数量。 另外，内部String的数量和占用内存数也会打印出来。





















