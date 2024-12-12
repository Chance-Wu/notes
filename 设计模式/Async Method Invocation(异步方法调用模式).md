### 一、意图

---

通过允许异步调用方法来增强并发性。此模式有助于执行并行任务、减少等待时间并提高系统吞吐量。



### 二、详解

---

允许一个方法在执行完其任务后，不阻塞调用线程，而是通过回调或其他机制通知调用方。提高了程序的并发性，使得程序能够同时处理多个任务，从而提升系统性能。



### 三、实现方式

---

#### 3.1 Future模式

>从JDK1.5开始引入了Future接口(FutureTask类)从异步执行的线程中取得返回值。
>
>Future 表示异步计算的结果，它提供了检查计算是否完成的方法，以等待计算的完成，并获取计算的结果。FutureTask类是Future接口方法的一个基本实现，是一种可以取消的异步计算任务，计算是通过Callable接口来实现的。
>
>1. **get()：阻塞一直等待执行完成拿到结果**。
>2. get(int timeout, TimeUnit timeUnit)：阻塞一直等待执行完成拿到结果，如果在超时时间内，没有拿到抛出异常
>3. isCancelled()：是否被取消
>4. isDone()：是否已经完成
>5. cancel(boolean mayInterruptIfRunning)：试图取消正在执行的任务

- **Callable接口**：定义一个任务，该任务有返回值；
- **Future接口**：表示异步计算的结果。
- **ExecutorService**：执行异步任务的线程池。

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<String> future = executor.submit(() -> {
    try {
        Thread.sleep(2000);
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt(); // 重置中断状态
        throw new RuntimeException("Thread was interrupted", e);
    }
    return "success";
});

// 做其他事情
System.out.println("Doing something else...");

// 阻塞等待结果
try {
    String result = future.get();
    System.out.println(result);
} catch (InterruptedException e) {
    System.err.println("Interrupted while waiting for result: " + e.getMessage());
    // 重置中断状态
    Thread.currentThread().interrupt();
} catch (ExecutionException e) {
    System.err.println("Error occurred during task execution: " + e.getCause().getMessage());
} finally {
    executor.shutdown();
}

// 输出：
// Doing something else...
// success
```

#### 3.2Callback模式

通过传递回调函数，在任务完成后调用回调来处理结果。

- 定义回调接口：定义一个接口，包含回调方法。
- 在异步任务完成后调用回调方法：将回调接口的实例作为参数传递给异步任务。

```java
public static void main(String[] args) {
    performTask(result -> System.out.println("Task completed with result: " + result));
    System.out.println("Main thread continues...");
}

public static void performTask(Callback callback) {
    CompletableFuture.runAsync(() -> {
        try {
            // 模拟耗时操作
            Thread.sleep(1000);
            callback.onComplete("Success");
        } catch (InterruptedException e) {
            callback.onComplete("Failure");
        }
    });
}

interface Callback {
    void onComplete(String result);
}
```

#### 3.3 CompletableFuture模式





































































