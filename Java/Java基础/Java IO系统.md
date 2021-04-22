#### 1. 输入和输出

>通过继承，从`InputStream`衍生的所有类都拥有名为`read()`的基本方法，用于读取单个字节或者字节数组；从`OutputStream`衍生的所有类都拥有基本方法`write()`，用于写入单个字节或者字节数组。
>
>一般情况下，都是将多个对象重叠在一起，提供自己期望的功能。

##### 1.1 InputStream 的类型

>InputStream的作用是标志那些从不同起源地产生输入的类。这些起源地包括（每个都有一个相关的InputStream子类）：
>
>1. 字节数组
>
>2. String对象
>
>3. 文件
>
>4. “管道”，它的工作原理与现实生活中的管道类似：将一个东西置入一端，它们在另一端出来。
>
>5. 一系列其他流，以便我们将其统一收集到单独一个流内。
>
>6. 其他起源地，如Internet连接等。
>
>FilterInputStream也属于InputStream的一种类型，用它可为“破坏器”类提供一个基础类，以便将属性或者有用的接口同输入流连接到一起。
>
>| InputStream 子类        | 数据源类型                                                   |
>| ----------------------- | ------------------------------------------------------------ |
>| ByteArrayInputStream    | 包含一个内存缓冲区，字节从中取出                             |
>| FileInputStream         | 从文件中获得字节字节                                         |
>| ObjectInputStream用来   | 用来恢复被序列化的对象                                       |
>| PipedInputStream        | 管道输入流，读取管道内容。多和PipedOutStream一起用于多线程通信。 |
>| SequenceInputStream     | 是多种输入流的串联，从第一个输入流读取，直到最后一个输入流   |
>| StringBufferInputStream | 读取的字节由字符串提供                                       |
>

###### 1.1.1 ByteArrayInputStream

>思路：在构造函数中传入`byte[]`，用于初始化其内部成员变量byte[]，然后利用其read()方法，把该流里的数据(buf)输出到指定数组中。
>
>```java
>byte[] buf = "hello".getBytes();
>ByteArrayInputStream in = null;
>try {
>in = new ByteArrayInputStream(buf);
>
>//缓冲容器
>byte[] flush = new byte[20];
>//接收长度
>int len;
>while ((len = in.read(flush)) != -1) {
>String str = new String(flush, 0, len);
>System.out.print(str);
>}
>} catch (IOException e) {
>e.printStackTrace();
>} finally {
>try {
>if (in != null) {
> // 释放资源
> in.close();
>}
>} catch (IOException e) {
>e.printStackTrace();
>}
>}
>```