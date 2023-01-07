```java
public class ThreadPoolExecutor extends AbstractExecutorService {

  // ... 其他代码省略

  private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
  private static final int COUNT_BITS = Integer.SIZE - 3;
  private static final int CAPACITY   = (1 << COUNT_BITS) - 1;

  private static final int RUNNING    = -1 << COUNT_BITS;
  private static final int SHUTDOWN   =  0 << COUNT_BITS;
  private static final int STOP       =  1 << COUNT_BITS;
  private static final int TIDYING    =  2 << COUNT_BITS;
  private static final int TERMINATED =  3 << COUNT_BITS;

  private static int runStateOf(int c)     { return c & ~CAPACITY; }
  private static int workerCountOf(int c)  { return c & CAPACITY; }
  private static int ctlOf(int rs, int wc) { return rs | wc; }

  private static boolean runStateLessThan(int c, int s) {
    return c < s;
  }

  private static boolean runStateAtLeast(int c, int s) {
    return c >= s;
  }

  private static boolean isRunning(int c) {
    return c < SHUTDOWN;
  }

  // ... 其他代码省略
}
```

首先看下ctlOf()方法，入参是int rs（线程池里线程的状态）和int wc（线程数），基于这两个点我们进行位运算分析。

首先看线程变量：

```java
private static final int COUNT_BITS = Integer.SIZE - 3;
private static final int RUNNING    = -1 << COUNT_BITS;
```

Integer.SIZE = 32，所以COUNT_BITS = 29，这里RUNNING就是-1的二进制位左移29位，得到的结果就是（提示：-1的二进制是: 1111 1111 1111 1111 ... 三十二位全是1）（在计算机中，负数采用补码的形式储存）

```
1110 0000 0000 0000 0000 0000 0000 0000
```

这就是RUNNING的二进制值。 同理我们可以分别得到SHUTDOWN、STOP、TIDYING、TERMINATED的二进制值。

```
0000 0000 0000 0000 0000 0000 0000 0000 // SHUTDOWN
0010 0000 0000 0000 0000 0000 0000 0000 // STOP
0100 0000 0000 0000 0000 0000 0000 0000 // TIDYING
0110 0000 0000 0000 0000 0000 0000 0000 // TERMINATED
```

这里其实已经可以看出作者的用意了，就是让**高3位作为线程池的状态，低29位用来表示线程数量**。对于

```java
private static int ctlOf(int rs, int wc) { return rs | wc; }
// 位运算“或”，遇1得1，否则为0
```

所以ctlOf就表示将rs代表的线程状态和wc代表的线程数计算在同一个32位二进制中，互相不影响。 所以如下：

```java
private final AtomicInteger ctl = new AtomicInteger(ctlOf(RUNNING, 0));
// 1110 0000 0000 0000 0000 0000 0000 0000
```

接着，再来分析下另外两个方法：runStateOf()、workerCountOf()，这两个方法都喝CAPACITY有关，先看下CAPACITY属性

```java
private static final int CAPACITY   = (1 << COUNT_BITS) - 1;
// 1 << 29 => 0010 0000 0000 0000 0000 0000 0000 0000
// 1 << 29 - 1 => 0001 1111 1111 1111 1111 1111 1111 1111

private static int runStateOf(int c)     { return c & ~CAPACITY; }
// ~CAPACITY => 1110 0000 0000 0000 0000 0000 0000 0000
// 运算“与”表示11得1，否则为0，所以 c & ~CAPACITY实际上就只能操作高三位，
// 也就是只能计算线程状态，并且~CAPACITY表示的是RUNNING时的状态

private static int workerCountOf(int c)  { return c & CAPACITY; }
// CAPACITY => 0001 1111 1111 1111 1111 1111 1111 1111
// 所以 c & CAPACITY 就表示只能操作低29位，所以workerCountOf就只能操作线程数
```

这里需要注意的是，runStateOf()和workerCountOf()传入的数字都是需要由：ctlOf()计算返回的，否则计算会出错。