>System类的构造器由private修饰，不允许被实例化。因此，类中的方法也都是static修饰的静态方法。

#### 1. System类简介

---

==System类代表当前Java程序运行的平台==，提供了对应的一些系统属性信息和系统操作。

```java
/**
 * System类包含了几个有用的类字段和方法。他不能被实例化。
 * 
 * <p>System类提供的功能包括标准输入，标准输出，错误输出流；
 * 访问外部定义的属性和环境变量；
 * 加载文件和库的方法；
 * 一个工具方法，能够快速复制一个数组的一部分。
 *
 * @author  unascribed
 * @since   JDK1.0
 */
public final class System 
```

System类提供了代表**标准输入**、**标准输出**和**错误输出**的类变量，并提供了一些静态方法用于访问**环境变量**、**系统属性**的方法，还提供了**加载文件和动态链接库**的方法。

#### 2. 构造方法

---

```java
/** 这个类不能初始化*/
private System() {
}
```

#### 3. 与流有关的变量和方法

---

in，out，err

```java
/**
 * 标准输入流。这个流已经代开，并且准备好提供输入数据。
 * 典型地，这个流对应于键盘输入或者由主机环境或用户指定的其他输入来源。
 */
public final static InputStream in = null;

/**
 * 标准输出流。这个流已经打开并且准备好接受输出数据。
 * 典型地，这个流对应显式输出（console）或者由主机环境或用户指定的其他输出地点。
 * <p>
 * 对于一个普通的独立的java程序，写出一行输出数据的典型方式是
 * <blockquote><pre>
 *     System.out.println(data)
 * </pre></blockquote>
 * <p>
 * 
 * 看PrintStream类的println方法
 */
public final static PrintStream out = null;

/**
 * 典型的错误输出流。这个流已经打开并且准备好接受输出数据。
 * <p>
 * 典型地，这个流对应显式输出（console）或者由主机环境或用户指定的其他输出地点。
 * 按照惯例，这个输出流用于展示错误信息或者其他即使在主要输出流，但是用户应该快速注意到的信息。
 * 变量out的值，应该被重定向到一个文件或者其他目的地，通常不被持续监控
 */
public final static PrintStream err = null;
```

#### 4. 数组复制

---

很多jdk的类都是用这个方法。

从指定的源头数组复制一个数组，以特定的位置开始，到结果数组的特定位置。
 * 数组成员的一个子序列被复制，从源头数组src到结果数组dest。
 * 被复制的成员的个数与参数length相同。
 * 源头数组的成员，从srcPos的位置到srcPos+length-1的位置，被复制到
 * 结果数组的位置，从destPos到destPos+length-1。

```java
public static native void arraycopy(Object src,  int  srcPos,
                                    Object dest, int destPos,
                                    int length);
```

```java
int[] src = {1,2,3,4,5,6};
int[] dest = new int[3];
System.arraycopy(src,0,dest,0,3);
```

#### 5. 获取当前系统时间

---

System类还有两个获取系统当前时间的方法：System.currentTimeMillis()；System.nanoTime()；它们都返回一个long型的整数，前者以毫秒为单位，后者以纳秒为单位。必须指出的是，这两个方法的时间粒度取决于底层的操作系统。

#### 6. identityHashCode()

---

System类还提供了一个identityHashCode(Object x)方法，该方法返回指定对象的精确hashCode值，也就是根据该对象的地址计算得到的hashCode值。当某个类的hashCode()方法被重写后，该类实例的hashCode()方法就不能唯一地标识该对象；但通过identityHashCode()方法返回的hashCode值，依然是根据该对象的地址计算得到的hashCode值。所以，如果两个对象的identityHashCode值相等，则两个对象绝对是同一个对象。

#### 7. gc()

---

调用 gc 方法暗示着 Java 虚拟机做了一些努力来回收未用对象或失去了所有引用的对象，以便能够快速地重用这些对象当前占用的内存。当控制权从方法调用中返回时，虚拟机已经尽最大努力从所有丢弃的对象中回收了空间。

#### 8. exit()

---

exit(int)方法终止当前正在运行的 Java 虚拟机，参数解释为状态码。根据惯例，非 0 的状态码表示异常终止。这是唯一一个能够退出程序并不执行finally的情况。说明：退出虚拟机会直接终止整个程序，这时的程序已经不是从代码的层面来终止程序。

