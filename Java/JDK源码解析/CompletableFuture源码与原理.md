### 一、JDK官方文档解释

---

CompletableFuture是JDK8中的新特性，主要用于对JDK5中加入的Future的补充。CompletableFuture实现了CompletionStage和Future接口。

CompletableFuture类的官方API文档解释：

1. CompletableFuture是一个在完成时可以触发相关方法和操作的Future，并且它可以视作为CompletableStage。
2. 除了直接操作状态和结果的这些方法和相关方法外（CompletableFuture API提供的方法），CompletableFuture还实现了以下的CompletionStage的相关策略： ① 非异步方法的完成，可以由当前CompletableFuture的线程提供，也可以由其他调用完方法的线程提供。 ② 所有没有显示使用Executor的异步方法，会使用ForkJoinPool.commonPool()（那些并行度小于2的任务会创建一个新线程来运行）。为了简化监视、调试和跟踪异步方法，所有异步任务都被标记为CompletableFuture.AsynchronouseCompletionTask。 ③ 所有CompletionStage方法都是独立于其他公共方法实现的，因此一个方法的行为不受子类中其他方法的覆盖影响。
3. CompletableFuture还实现了Future的以下策略 ① 不像FutureTask，因CompletableFuture无法直接控制计算任务的完成，所以CompletableFuture的取消会被视为异常完成。调用cancel()方法会和调用completeExceptionally（）方法一样，具有同样的效果。isCompletedEceptionally()方法可以判断CompletableFuture是否是异常完成。 ② 在调用get()和get(long, TimeUnit)方法时以异常的形式完成，则会抛出ExecutionException，大多数情况下都会使用join()和getNow(T)，它们会抛出CompletionException。

小结：

1. Concurrent包中的Future在获取结果时会发生阻塞，而CompletableFuture则不会，它可以通过触发异步方法来获取结果。
2. 在CompletableFuture中，如果没有显示指定的Executor的参数，则会调用默认的ForkJoinPool.commonPool()。
3. 调用CompletableFuture的cancel()方法和调用completeExceptionally()方法的效果一样。

在JDK5中，使用Future来获取结果时都非常的不方便，只能通过get()方法阻塞线程或者通过轮询isDone()的方式来获取任务结果，这种阻塞或轮询的方式会无畏的消耗CPU资源，而且还不能及时的获取任务结果，因此JDK8中提供了CompletableFuture来实现异步的获取任务结果。



### 二、CompletableFuture的API的使用

---

CompletableFuture类提供了非常多的方法供我们使用，包括了runAsync()、supplyAsync()、thenAccept()等方法。

#### 2.1 runAsync()

异步运行

```java
@Test
public void runAsyncExample() throws Exception {
  ExecutorService executorService = Executors.newSingleThreadExecutor();
  CompletableFuture cf = CompletableFuture.runAsync(() -> {
    try {
      Thread.sleep(2000);
    } catch (Exception e) {
      e.printStackTrace();
    }
    System.out.println(Thread.currentThread().getName());
  }, executorService);
  System.out.println(Thread.currentThread().getName());
  while (true) {
    if (cf.isDone()) {
      System.out.println("CompletedFuture...isDown");
      break;
    }
  }
}
```

运行结果：

```
main
pool-1-thread-1
CompletedFuture...isDown
```

这里调用的runAsync()方法没有使用FrokJoinPool的线程，而是使用了Executors.newSingleThreadExecutor()中的线程。**runAsync()其实效果跟单开一个线程一样**。

#### 2.2 supplyAsync()

supply有供应的意思，supplyAsync就可以理解为异步供应，查看supplyAsync()方法入参可以知道，其有两个入参：

- `Supplier<U> supplier`
- `Executor executor`

这里先简单介绍下Supplier接口，Supplier接口是JDK8引入的新特性，它也是用于创建对象的，只不过调用Supplier的get()方法时，才会去通过构造方法去创建对象，并且每次创建出的对象都不一样。Supplier常用语法为：

```java
Supplier<MySupplier> sup= MySupplier::new;
```

再讲一个thenAccept()方法，可以发现thenAccept()方法的入参如下：

- `Comsumer<? super T>`

Comsumer接口同样是java8新引入的特性，它有两个重要接口方法：

1. **accept()**
2. **andThen()**

thenAccept()可以理解为接收CompletableFuture的结果然后再进行处理。

下面看下supplyAsync()和thenAccept()的例子：

```java
public void thenApply() throws Exception {
  ExecutorService executorService = Executors.newFixedThreadPool(2);
  CompletableFuture cf = CompletableFuture.supplyAsync(() -> { //实现了Supplier的get()方法
    try {
      Thread.sleep(2000);
    } catch (Exception e) {
      e.printStackTrace();
    }
    System.out.println("supplyAsync " + Thread.currentThread().getName());
    return "hello ";
  },executorService).thenAccept(s -> { //实现了Comsumper的accept()方法
    try {
      thenApply_test(s + "world");
    } catch (Exception e) {
      e.printStackTrace();
    }
  });

  System.out.println(Thread.currentThread().getName());
  while (true) {
    if (cf.isDone()) {
      System.out.println("CompletedFuture...isDown");
      break;
    }
  }
}
```

运行结果：

```
main
supplyAsync pool-1-thread-1
thenApply_test hello world
thenApply_test pool-1-thread-1
CompletedFuture...isDown
```

从代码逻辑可以看出，thenApply_test等到了pool-1-thread-1线程完成任务后，才进行的调用，并且拿到了supplye()方法返回的结果，而main则异步执行了，这就避免了Future获取结果时需要阻塞或轮询的弊端。

#### 2.3 exceptionally()

当任务在执行过程中报错了咋办？exceptionally()方法很好的解决了这个问题，当报错时会去调用exceptionally()方法，它的入参为：Function<Throwable, ? extends T> fn，fn为执行任务报错时的回调方法，下面看看代码示例：

```java
public void exceptionally() {
  ExecutorService executorService = Executors.newSingleThreadExecutor();
  CompletableFuture cf = CompletableFuture.supplyAsync(() -> {
    try {
      Thread.sleep(3000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    if (1 == 1) {
      throw new RuntimeException("测试exceptionally...");
    }
    return "s1";
  }, executorService).exceptionally(e -> {
    System.out.println(e.getMessage());
    return "helloworld " + e.getMessage();
  });
  cf.thenAcceptAsync(s -> {
    System.out.println("thenAcceptAsync: " + s);
  });
  System.out.println("main: " + Thread.currentThread().getName());
  while (true) {}
}
```

运行结果：

```
main: main
java.lang.RuntimeException: 测试exceptionally...
CompletableFuture is Down...helloworld java.lang.RuntimeException: 测试exceptionally...
thenAcceptAsync: helloworld java.lang.RuntimeException: 测试exceptionally...
```

从代码以及运行结果来看，**当任务执行过程中报错时会执行exceptionally()中的代码，thenAcceptAsync()会获取抛出的异常并输出到控制台**，不管CompletableFuture()执行过程中报错、正常完成、还是取消，都会被标示为**已完成**，所以最后CompletableFuture.isDown()为true。

在Java8中，新增的ForkJoinPool.commonPool()方法，这个方法可以获得一个公共的ForkJoin线程池，这个公共线程池中的所有线程都是Daemon线程，意味着如果主线程退出，这些线程无论是否执行完毕，都会退出系统。



### 三、源码分析

---

CompletableFuture类实现了Future接口和CompletionStage接口，Future大家都经常遇到，但是这个CompletionStage接口就有点陌生了，这里的CompletionStage实际上是一个任务执行的一个“阶段”，CompletionStage详细的内容在下文有介绍。

```java
public class CompletableFuture<T> implements Future<T>, CompletionStage<T> {
  volatile Object result;       // CompletableFuture的结果值或者是一个异常的报装对象AltResult
  volatile Completion stack;    // 依赖操作栈的栈顶
  ...
    // CompletableFuture的方法
    ... 
    // Unsafe mechanics
    private static final sun.misc.Unsafe UNSAFE;
  private static final long RESULT;
  private static final long STACK;
  private static final long NEXT;
  static {
    try {
      final sun.misc.Unsafe u;
      UNSAFE = u = sun.misc.Unsafe.getUnsafe();
      Class<?> k = CompletableFuture.class;
      RESULT = u.objectFieldOffset(k.getDeclaredField("result")); //计算result属性的位偏移量
      STACK = u.objectFieldOffset(k.getDeclaredField("stack")); //计算stack属性的位偏移量
      NEXT = u.objectFieldOffset 
        (Completion.class.getDeclaredField("next"));  //计算next属性的位偏移量
    } catch (Exception x) {
      throw new Error(x);
    }
  }
}
```

https://github.com/coderbruis/JavaSourceCodeLearning/blob/master/note/JDK/%E6%B7%B1%E5%85%A5%E8%A7%A3%E8%AF%BBCompletableFuture%E6%BA%90%E7%A0%81%E4%B8%8E%E5%8E%9F%E7%90%86.md





































