#### 1. 理解Future

当处理一个任务时，总会遇到以下几个阶段：

1. 提交任务
2. 执行任务
3. 任务完成的后置处理

以下简单定义：

- 构造及提交任务的线程为生产者线程；
- 执行任务的线程为消费者线程；
- 任务的后置处理线程为后置消费者线程。

根据任务的特性，会衍生各种各样的线程模型。其中包括==Future模式==。

>示例
>
>```java
>ExecutorService executor = Executors.newFixedThreadPool(3);
>Future<String> future = executor.submit(new Callable<String>() {
>
>    @Override
>    public String call() throws Exception {
>        //do some thing
>        Thread.sleep(100);
>        return "i am ok";
>    }
>});
>System.out.println(future.isDone());
>System.out.println(future.get());
>```
>
>首先创建一个线程池，然后向线程池中提交了一个任务，==submit提交任务后会被立即返回，而不会等到任务实际处理完成才会返回==，==任务提交后返回值便是Future==，通过Future我们可以调用get()方法阻塞式的获取返回结果，也可以使用idDone获取任务是否完成。

生产者线程在提交完任务后，有两个选择：==关注处理结果==和==不关注处理结果==。处理结果包括任务的返回值，也包含任务是否正确完成，中途是否抛出异常等等。

Future 模式提供一种机制，在消费者异步处理生产者提交的任务的情况下，生产者线程也可以拿到消费者线程的处理结果，同时通过 Future 也可以取消掉处理中的任务。

在实际的开发中，我们经常会遇到这种类似需求。任务需要异步处理，同时又关心任务的处理结果，此时使用 Future 是再合适不过了。

##### 1.1 Future如何被构建的

生产者线程提交给消费者线程池任务时，线程池会构造一个实现了Future接口的对象FutureTask。该对象相当于是消费者和生产者的桥梁，==消费者通过FutureTask存储任务的处理结果==，更新任务的状态：未开始、正在处理、已完成等。而生产者拿到的FutureTask被转型为Future接口，可以阻塞式获取任务的处理结果，非阻塞式获取任务处理状态。

##### 1.2 思考

==Java中通过共享对象来进行跨线程通信==，并且提供了各种工具来保证共享对象的线程安全性，Future是一个典型通过共享内存通信的例子，而熟悉Go语言会想到，协程间通信的方式是通过channel。

就像 Go 语言信仰的那句话：不要通过共享内存来通信，而应该通过通信来共享内存。Go 所作的就是通过 channel，通过通信共享了内存，共享了数据。

通过对 Future 的示例，我们了解了 ==Future 在任务生产者和消费者之间的起到的桥梁作==用。

#### 2. 任务结果的处理

有两种可能生产者不需要关注的任务的处理结果：

- 第一种可能，处理结果并不影响后续的业务逻辑。
- 另一种可能，被提交的任务在结束前，会将自身的处理结果上报到其他结构中，例如 MQ，DB，Redis 等等， 这些任务的处理结果会被其他的协调者或者调度者跟踪和监控，不再需要生产者关心。

但另外一种更简单的系统设计，要求生产者需要关心处理结果，根据处理结果执行后续的任务处理， ==CompletableFuture== 正是为此而生。

##### 2.1 Future改造

以上例子获取结果都是使用get阻塞式，实际开发中并不希望生产者线程会被阻塞住，但是又希望当我们提交完任务后可以通过Future处理结果，如何实现呢？

思路：消费者线程在执行Task后，会将处理结果set到Future中，那么我们可以在调用set方法后执行一些列后置处理，这些后置处理是生产者在提交Task时指定的。这样虽然执行后置处理的线程并不是生产者线程，但实际上处理逻辑是由生产者指定的。

==ComparableFuture基于这种机制为我们提供了很多后置处理的执行方式，同时又提供了很多整合多个Future的方法。可以使用ComparableFuture灵活的处理多个Task协同处理的问题。==

##### 2.2 复杂任务的示例说明

某些业务场景中，长任务需要被拆解为很多小任务，而这些小任务有些可以并行处理，有些是有依赖顺序的。假设有如下一个长任务。

TaskA如下子任务：

1. 可并行处理的Task1.1、Task1.2、Task1.3。
2. 依据第一步Task1.1、Task1.2、Task1.3三个任务的结果，执行Task2。
3. 根据Task2的结果，异步的执行Task3.1。

如何保证TaskA中多个子任务的依赖关系，同时保证任务可以得到异步处理呢？

```java
future1.thenCombine(future2， (args1， args2) -> {      ### Task 1
              println(args1);
              println(args2);
              return "3";
          }).thenApply((res) -> {                      ### Task 2
              println(res);
              return "4";
          }).thenApplyAsync((res) -> {                 ### Task 3
              println(res);
              return "5";
          });
```

future1、future2都完成时，执行了==thenCombine==动作，会生成新的Future。新的Future完成后将执行==thenApply==，对合并产生的结果再次处理，==最后再次对结果处理==，而此次处理是异步执行，即后置处理的线程和任务的消费者线程不是同一个线程。

>CompletableFuture 提供了非常多的方法， 其所有方法按照功能分类如下：
>
>1. 对一个或多个 Future 合并操作，生成一个新的 Future， 参考 allOf，anyOf，runAsync， supplyAsync。
>2. 为 Future 添加后置处理动作， thenAccept， `thenApply`， thenRun。
>3. 两个人 Future 任一或全部完成时，执行后置动作：applyToEither， acceptEither， thenAcceptBothAsync， runAfterBoth，runAfterEither 等。
>4. 当 Future 完成条件满足时，异步或同步执行后置处理动作： `thenApplyAsync`， thenRunAsync。所有异步后置处理都会添加 Async 后缀。
>5. 定义 Future 的处理顺序 thenCompose 协同存在依赖关系的 Future，`thenCombine`合并多个 Future的处理结果返回新的处理结果。
>6. 异常处理 exceptionally ，如果任务处理过程中抛出了异常。