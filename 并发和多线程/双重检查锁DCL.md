实现单例模式时，如果未考虑多线程的情况，就容易写出如下错误代码：

```java
public class Singleton {
  private static Singleton uniqueSingleton;
  private Singleton() {
  }

  public Singleton getInstance() {
    if (null == uniqueSingleton) {
      uniqueSingleton = new Singleton();
    }
    return uniqueSingleton;
  }
}
```

考虑有两个线程同时调用getInstance()：

| Time | Thread A                    | Thread B                    |
| ---- | --------------------------- | --------------------------- |
| T1   | 检查到 uniqueSingleton 为空 |                             |
| T2   |                             | 检查到 uniqueSingleton 为空 |
| T3   |                             | 初始化对象A                 |
| T4   |                             | 返回对象A                   |
| T5   | 初始化对象B                 |                             |
| T6   | 返回对象B                   |                             |

如上，uniqueSingleton被实例化了两次并且被不同对象持有。完全违背了单例的初衷。



#### 1. 加锁

---

出现如上情况，第一反应就是加锁：

```java
public class Singleton {
  private static Singleton uniqueSingleton;
  private Singleton() {
  }

  public synchronized Singleton getInstance() {
    if (null == uniqueSingleton) {
      uniqueSingleton = new Singleton();
    }
    return uniqueSingleton;
  }
}
```

这样虽然解决了问题，但是因为用到了synchronized，会导致很大的性能开销，并且加锁其实只需要在第一次初始化的时候用到，之后的调用都没有必要再进行加锁。



#### 2. 双重检查锁

---

双重检查锁（double checked locking）是对上述问题的一种优化。先按判断对象是否已经被初始化，再决定要不要加锁。

>执行双重检查是因为，如果多个线程同时通过了第一次检查，并且其中一个线程首先通过了第二次检查并实例化了对象，那么剩余通过了第一次检查的线程就不会再去实例化对象。

##### 2.1 DCL 错误写法

```java
public class Singleton {
  private static Singleton uniqueSingleton;
  private Singleton() {
  }

  public Singleton getInstance() {
    if (null == uniqueSingleton) {
      synchronized (Singleton.class) {
        if (null == uniqueSingleton) {
          uniqueSingleton = new Singleton();   // error
        }
      }
    }
    return uniqueSingleton;
  }
}
```

这样写，运行顺序就变成了：

1. 检查变量是否被初始化（不去获得锁），如果已被初始化则立即返回。
2. 获取锁。
3. 再次检查变量是否已经被初始化，如果还没被初始化就初始化一个对象。

>上述代码隐患：
>
>实例化对象的那行代码（标记为error的那行），实际上可以分解成以下三个步骤：
>
>1. 分配内存空间
>1. 初始化对象
>1. 将对象指向刚分配的内存空间
>
>但是有些编译器为了性能的原因，可能会将第二步和第三步进行==重排序==，顺序就成了：
>
>1. **分配内存空间**
>1. **将对象指向刚分配的内存空间**
>1. **初始化对象**
>
>现在考虑重排序后，两个线程发生了以下调用：
>
>| Time | Thread A                      | Thread B                                      |
>| ---- | ----------------------------- | --------------------------------------------- |
>| T1   | 检查到uniqueSingleton为空     |                                               |
>| T2   | 获取锁                        |                                               |
>| T3   | 再次检查到uniqueSingleton为空 |                                               |
>| T4   | 为uniqueSingleton分配内存空间 |                                               |
>| T5   | 将uniqueSingleton指向内存空间 |                                               |
>| T6   |                               | 检查到uniqueSingleton不为空                   |
>| T7   |                               | 访问uniqueSingleton（此时对象还未完成初始化） |
>| T8   | 初始化uniqueSingleton         |                                               |
>
>这种情况下，T7时刻 Thread B 对 uniqueSingleton 的访问，访问的是一个初始化未完成的对象。

##### 2.2 DCL 正确写法

```java
public class Singleton {
  private volatile static Singleton uniqueSingleton;
  private Singleton() {
  }

  public Singleton getInstance() {
    if (null == uniqueSingleton) {
      synchronized (Singleton.class) {
        if (null == uniqueSingleton) {
          uniqueSingleton = new Singleton();
        }
      }
    }
    return uniqueSingleton;
  }
}
```

为解决重排序的问题，需要在 uniqueSingleton 前加入volatile。**使用 volatile 关键字，重排序被禁止，所有的【写操作】都将发生在【读操作】之前**。