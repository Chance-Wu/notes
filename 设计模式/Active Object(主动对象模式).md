### 一、意图

---

主动对象模式为 Java 中的异步处理提供了一种可靠的方法，可确保应用程序响应迅速并实现高效的线程管理。它通过将任务封装在具有自己的线程和消息队列的对象中来实现这一点。这种分离可使主线程保持响应，并避免直接线程操作或共享状态访问等问题。



### 二、解释

---

将方法执行与方法调用分离，以提高多线程应用程序中的并发性和响应能力。

> 主动对象设计模式将方法执行与对象的方法调用解耦，每个对象都驻留在自己的控制线程中。目标是通过使用异步方法调用和调度程序来处理请求，从而引入并发性。
>
> 该模式由六个元素组成：
>
> - 代理，它向客户端提供具有可公开访问方法的接口。
> - 定义活动对象上的方法请求的接口。
> - 来自客户的待处理请求列表。
> - 调度程序，决定下一步执行哪个请求。
> - 活动对象方法的实现。
> - 客户端接收结果的回调或变量。



### 三、代码示例

---

基于线程池和主动对象模式的日志记录实现设计。

#### 3.1 日志记录任务（命令对象）

命令对象封装日志的写入逻辑。

```java
/**
 * 日志记录任务（命令对象）
 * 该类实现了{@link Runnable}接口，用于执行日志记录任务
 * 它封装了日志消息，并在run方法中定义了如何记录日志
 *
 * @author chance
 * @date 2024/12/4 13:04
 * @since 1.0
 */
public class LogTask implements Runnable {

    /**
     * 日志消息
     */
    private final String message;

    /**
     * 构造方法，创建一个LogTask实例
     *
     * @param message 要记录的日志消息
     */
    public LogTask(String message) {
        this.message = message;
    }

    /**
     * 实现Runnable接口的run方法
     * 该方法定义了日志记录的操作
     * 在这个例子中，它模拟了日志写入操作，实际应用中可以替换为写入日志文件或数据库等操作
     */
    @Override
    public void run() {
        // 模拟日志写入（可以是写文件、写数据库等）
        System.out.println(Thread.currentThread().getName() + ": " + message);
    }
}
```

#### 3.2 调度器（Scheduler）

调度器从任务队列中取出任务，并使用线程池执行。

```java
/**
 * 日志调度器（Scheduler）
 * 负责管理日志处理任务的调度与执行
 *
 * @author chance
 * @date 2024/12/4 13:10
 * @since 1.0
 */
public class LogScheduler {

    /**
     * 任务队列，用于存放待执行的日志处理任务
     */
    private final BlockingQueue<Runnable> taskQueue;

    /**
     * 线程池，用于执行日志处理任务
     */
    private final ExecutorService threadPool;

    /**
     * 运行状态标志，用于控制日志调度器的启动与停止
     */
    private volatile boolean running = true;

    /**
     * 构造方法，初始化日志调度器
     *
     * @param taskQueue      任务队列，用于存放待执行的任务
     * @param threadPoolSize 线程池大小，决定同时执行任务的数量
     */
    public LogScheduler(BlockingQueue<Runnable> taskQueue, int threadPoolSize) {
        this.taskQueue = taskQueue;
        // 创建固定大小的线程池
        this.threadPool = Executors.newFixedThreadPool(threadPoolSize);
    }

    /**
     * 启动日志调度器
     * 在一个新线程中循环检查任务队列，并提交任务到线程池执行
     */
    public void start() {
        new Thread(() -> {
            while (running) {
                try {
                    // 从队列中取出任务
                    Runnable task = taskQueue.take();
                    // 提交到线程池执行
                    threadPool.submit(task);
                } catch (InterruptedException e) {
                    // 捕获中断异常，恢复中断状态
                    Thread.currentThread().interrupt();
                }
            }
        }).start();
    }

    /**
     * 停止日志调度器
     * 设置running状态为false，并关闭线程池
     */
    public void stop() {
        running = false;
        threadPool.shutdown();
    }
}
```

#### 3.3 日志代理（Proxy）

代理负责接收客户端请求，将日志写入任务封装为LogTask，并提交到命令队列。

```java
/**
 * 日志代理（Proxy）
 * 该类用于将日志记录请求委托给专门的日志调度器，以异步方式处理日志记录任务
 * 它通过一个阻塞队列来缓存日志记录任务，然后由日志调度器根据需要处理这些任务
 *
 * @author chance
 * @date 2024/12/4 13:06
 * @since 1.0
 */
public class LogProxy {

    /**
     * 任务队列，用于缓存待处理的日志记录任务
     */
    private final BlockingQueue<Runnable> taskQueue;

    /**
     * 日志调度器，负责根据需要处理日志记录任务
     */
    private final LogScheduler logScheduler;

    /**
     * 构造方法，初始化LogProxy实例
     *
     * @param threadPoolSize 线程池大小，决定同时处理日志任务的最大线程数
     */
    public LogProxy(int threadPoolSize) {
        this.taskQueue = new LinkedBlockingQueue<>();
        this.logScheduler = new LogScheduler(taskQueue, threadPoolSize);
    }

    /**
     * 记录日志的方法，将日志记录任务提交到任务队列中
     *
     * @param message 日志信息，需要记录的消息
     */
    public void log(String message) {
        try {
            // 将任务放入队列
            taskQueue.put(new LogTask(message));
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }

    /**
     * 启动日志调度器，开始处理日志记录任务
     */
    public void start() {
        logScheduler.start();
    }

    /**
     * 停止日志调度器，不再处理新的日志记录任务
     */
    public void stop() {
        logScheduler.stop();
    }
}
```

#### 3.4 测试

```java
// 创建日志代理，线程池大小为3
LogProxy logProxy = new LogProxy(3);
logProxy.start();

// 模拟多线程提交日志
for (int i = 0; i < 10; i++) {
    final int index = i;
    new Thread(() -> logProxy.log("Log message " + index)).start();
}

// 停止日志系统
try {
    // 等待日志写入完成
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
}
logProxy.stop();
```



### 四、主动对象模式的性能特点

---

#### 4.1 额外开销

- 主动对象模式引入了命令队列和调度器，增加了队列操作（如 `put` 和 `take`）的开销。
- 需要额外的线程（调度线程）从队列中取任务并提交给线程池，增加了一层间接操作。

#### 4.2 吞吐量

对于单个任务的延迟稍高，但在高并发场景下，任务队列有助于平滑流量峰值，防止线程池被瞬间打满。

#### 4.3 任务管理

- 任务在队列中排队，线程池只需专注于执行任务，而不是直接处理突发流量。
- 更适合高频次、快速完成的小任务（如日志记录），在高并发下性能相对稳定。

#### 4.4 建议

1. 如果系统对性能极为敏感且并发量可控，**直接使用线程池**是更轻量的选择。
2. 如果需要处理高并发、任务量大、流量波动明显的场景，采用**主动对象模式**更适合，尤其当日志任务需要一定的可靠性和稳定性时。