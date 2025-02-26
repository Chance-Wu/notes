<img src="img/0081Kckwgy1gkam0ali3aj30om0h4q4n.jpg" style="zoom:80%">

#### 1. jdk介绍

> - JDK提供了编译、运行Java程序所需的各种资源和工具。包括包括Java编译器，Java运行时环境【JRE】；开发工具包括编译工具(javac.exe) 打包工具(jar.exe)等。
>
> - JRE：即JAVA运行时环境，JVM就是包括在JRE中，以及常用的JAVA类库等；
>
> - SDK：SDK是基于JDK进行扩展的，是解决企业级开发的工具包。如JSP、JDBC、EJB等就是由SDK提供的 ；

#### 2. JVM

>JVM 是可运行 Java 代码的假想计算机 ，包括==一套字节码指令集==、==一组寄存器==、==一个栈==、==一个垃圾回收==，==堆==和==一个存储方法域==。JVM 是运行在操作系统之上的，它与硬件没有直接的交互。

>① Java 源文件—->编译器—->字节码文件
>
>② 字节码文件—->JVM—->机器码
>
>==每一种平台的解释器是不同的==，但是实现的虚拟机是相同的，这也就是 Java 为什么能够跨平台的原因了，==当一个程序从开始运行，这时虚拟机就开始实例化了，多个程序启动就会存在多个虚拟机实例==。程序退出或者关闭，则虚拟机实例消亡，多个虚拟机实例之间数据不能共享。

<img src="img/0081Kckwgy1gm72yov3i0j30uj0u040s.jpg" style="zoom:60%">

##### 2.1 内存结构还是运行时数据区

>从某一角度来说，**Java 虚拟机的内存结构 == 运行时数据区**，在《Java 虚拟机规范》中用的是【运行时数据区】术语的。

##### 2.2 运行时数据区

>JVM被分为三个主要的子系统：*<u>类加载器子系统</u>*、<u>*运行时数据区*</u>和*<u>执行引擎</u>* 。
>
>Java 虚拟机规范中，定义了五种运行时数据区，分别是 ==Java 堆==、==方法区==、==虚拟机栈==、==本地方法区==、==程序计数器==。
>
>*<u>方法区中包含了运行时常量池</u>*。
>
><img src="img/0081Kckwgy1gm72z927tpj30qg0eqgmr.jpg" style="zoom:60%">