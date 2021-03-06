#### 1. 堆空间的参数设置

> `-XX:+PrintFlagsInitial`：查看所有的参数的默认初始值
>
> 
>
> `-XX:+PrintFlagsFinal`：查看所有的参数的最终值（可能会存在修改，不再是初始值）
>
> 
>
> `-Xms`：初始堆空间内存（默认为物理内存的1/64）
>
> 
>
> `-Xmx`：最大堆空间内存（默认为物理内存的1/4）
>
> 
>
> `-Xmn`：设置新生代的大小。（初始值及最大值）
>
> 
>
> `-XX:NewRatio`：配置新生代与老年代在堆结构的占比
>
> 
>
> `-XX:SurvivorRatio`：设置新生代中Eden和S0/S1空间的比例
>
> 
>
> `-XX:MaxTenuringThreshold`：设置新生代垃圾的最大年龄
>
> 
>
> `-XX:+PrintGCDetails`：输出详细的GC处理日志
>
> 
>
> 打印GC简要信息：①`-XX:+PrintGC `② `- verbose:gc`
>
> 

> 在发生Minor GC之前，虚拟机会*<u>检查老年代最大可用的连续空间是否大于新生代所有对象的总空间</u>*。

> Tips： 在JDK6 Update24之后，HandlePromotionFailure参数不会再影响到虚拟机的空间分配担保策略，JDK6 Update 24之后的规则变为==只要老年代的连续空间大于新生代对象总大小或者历次晋升的平均大小就会进行Minor GC，否则将进行FullGC==。