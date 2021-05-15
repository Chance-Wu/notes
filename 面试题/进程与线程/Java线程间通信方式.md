- 同步
- while轮询
- wait/notify机制
- 管道通信



#### 1. 同步

---

这里的同步是指多个线程通过`synchronized`关键字方式来实现线程间的通信。

```java
public class MyObject {

  synchronized public void methodA() {
    System.out.println("执行A方法");
  }

  synchronized public void methodB() {
    System.out.println("执行B方法");
  }
}
```

```java
public class ThreadA extends Thread {
  private MyObject object;

  @Override
  public void run() {
    super.run();
    object.methodA();
  }

  public ThreadA(MyObject object) {
    this.object = object;
  }
}
```

```java
public class ThreadB extends Thread {

  private MyObject object;

  @Override
  public void run() {
    super.run();
    object.methodB();
  }

  public ThreadB(MyObject object) {
    this.object = object;
  }
}

```

```java
public class TestSynchronized {

  public static void main(String[] args) {
    MyObject object = new MyObject();

    // 线程A和线程B持有 同一个MyObject类的对象
    ThreadA threadA = new ThreadA(object);
    ThreadB threadB = new ThreadB(object);
    threadA.start();
    threadB.start();
  }
}
```

> 由于线程A和线程B持有同一个MyObject类的对象object，尽管这两个线程需要调用不同的方法，但是它们是同步执行的，比如：==线程B需要等待线程A执行完了methodA()方法之后，它才能执行methodB()方法。这样，线程A和线程B就实现了通信==。
>
> 这种方式，本质上就是“共享内存”式的通信。==多个线程需要访问同一个共享变量，谁拿到了锁（获得了访问权限），谁就可以执行==。



#### 2. while轮询（poll）

---

>线程A不断地改变条件，线程B不停地通过while语句检测这个条件(list.size()==5)是否成立，从而实现了线程间的通信。但是这种方式会浪费CPU资源。
>
>因为JVM调度器将CPU交给线程B执行时，它没做啥“有用”的工作，只是在不断地测试某个条件是否成立。这种方式还存在另外一个问题：==轮询的条件的可见性问题==。（线程都是先把变量读取到本地线程栈空间，然后再去再去修改的本地变量。因此，如果线程B每次都在取本地的 条件变量，那么尽管另外一个线程已经改变了轮询的条件，它也察觉不到，这样也会造成死循环。）



#### 3. wait/notify机制

---

```java
public class MyList {

  private static List<String> list = new ArrayList<>();

  public static void add() {
    list.add("anyString");
  }

  public static int size() {
    return list.size();
  }
}
```

```java
public class ThreadA extends Thread {

  private Object lock;

  public ThreadA(Object lock) {
    super();
    this.lock = lock;
  }

  @Override
  public void run() {
    try {
      synchronized (lock) {
        if (MyList.size() != 5) {
          System.out.println("wait begin "
                             + System.currentTimeMillis());
          lock.wait();
          System.out.println("wait end  "
                             + System.currentTimeMillis());
        }
      }
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

```java
public class ThreadB extends Thread {

  private Object lock;

  public ThreadB(Object lock) {
    super();
    this.lock = lock;
  }

  @Override
  public void run() {
    try {
      synchronized (lock) {
        for (int i = 0; i < 10; i++) {
          MyList.add();
          if (MyList.size() == 5) {
            lock.notify();
            System.out.println("已经发出了通知");
          }
          System.out.println("添加了" + (i + 1) + "个元素!");
          Thread.sleep(1000);
        }
      }
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

```java
public class Run {

  public static void main(String[] args) {
    try {
      Object lock = new Object();

      ThreadA a = new ThreadA(lock);
      a.start();

      Thread.sleep(50);

      ThreadB b = new ThreadB(lock);
      b.start();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```

wait begin 1620895727262
添加了1个元素!
添加了2个元素!
添加了3个元素!
添加了4个元素!
已经发出了通知
添加了5个元素!
添加了6个元素!
添加了7个元素!
添加了8个元素!
添加了9个元素!
添加了10个元素!
wait end  1620895737345

>线程A要等待某个条件满足时（list.size() != 5），才执行操作。线程B则向list中添加元素，改变list的size。这里用到了Object类的`wait()`和`notify()`方法。
>
>- 当条件未满足时(list.size() !=5)，线程A调用wait() 放弃CPU，并进入阻塞状态。---不像②while轮询那样占用CPU；
>- 当条件满足时，线程B调用 notify()通知 线程A，所谓通知线程A，就是唤醒线程A，并让它进入可运行状态。
>
>CPU的利用率提高了。
>
>缺点：比如，线程B先执行，一下子添加了5个元素并调用了notify()发送了通知，而此时线程A还执行；当线程A执行并调用wait()时，那它永远就不可能被唤醒了。因为，线程B已经发了通知了，以后不再发通知了。这说明：**通知过早，会打乱程序的执行逻辑。**



#### 4. 管道通信

---

使用`java.io.PipedInputStream` 和 `java.io.PipedOutputStream`进行通信

分布式系统中说的两种通信机制：==共享内存机制==和==消息通信机制==。

- 感觉前面的①中的synchronized关键字和②中的while轮询 “属于” 共享内存机制，由于是轮询的条件使用了volatile关键字修饰时，这就表示它们通过判断这个“共享的条件变量“是否改变了，来实现进程间的交流。
- ==管道通信，更像消息传递机制，也就是说：通过管道，将一个线程中的消息发送给另一个==。

