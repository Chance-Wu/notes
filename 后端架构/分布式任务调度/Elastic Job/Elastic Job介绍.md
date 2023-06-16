Elastic-Job是一个分布式调度解决方案，由两个相互独立的子项目Elastic-Job-Lite和Elastic-Job-Cloud组成。

- Elastic-Job-Lite定位为轻量级无中心化解决方案，使用jar包的形式提供分布式任务的协调服务；
- Elastic-Job-Cloud采用自研的Mesos Framework的解决方案，额外提供资源治理、应用分发以及进程隔离等功能。

 

#### 1. 任务调度

---

思考一下业务场景的解决方案：

- 某电商系统需要在每天上午10点，下午3点，晚上8点发放一批优惠券。
- 某银行系统需要在信用卡到期还款日的前三天进行短信提醒。

以上场景就是任务调度所需要解决的问题。

任务调度是指系统为了自动完成特定任务，在约定的特定时刻去执行任务的过程。



#### 2. 任务调度如何实现

---

##### 2.1 多线程方式实现

开启一个线程，每sleep一段时间，就去检查是否已到预期执行时间。参考以下代码：

```java
public static void main(String[] args) {
  // 每隔10s执行任务
  final long timeInterval = 10000;
  Runnable runnable = new Runnable() {
    @Override
    public void run() {
      while (true) {
        try {
          Thread.sleep(timeInterval);
          System.out.println(">>>>>>>>do something");
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
      }
    }
  };
  Thread thread = new Thread(runnable);
  thread.start();
}
```

##### 2.2 Timer方式实现

Timer的优点在于简单易用，每个Timer对应一个线程，因此可以同时启动多个Timer并行执行多个任务，同一个Timer中的任务是串行执行的。

```java
public static void main(String[] args) {
  Timer timer = new Timer();
  timer.schedule(new TimerTask() {
    @Override
    public void run() {
      System.out.println(">>>>>>>>>do something");
    }
  },1000,2000);
  // 1s后开始调度，每2s执行一次
}
```

##### 2.3 ScheduledExecutor方式实现

线程池设计的ScheduledExecutor，其设计思想是，每一个被调度的任务都会由线程池中的一个线程去执行，因此任务是并发的，相互之间不会受到干扰。

```java
public static void main(String[] args) {
  ScheduledExecutorService service = Executors.newScheduledThreadPool(10);
  service.scheduleAtFixedRate(new Runnable() {
    @Override
    public void run() {
      System.out.println(">>>>>>>>>do something");
    }
  }, 1, 2, TimeUnit.SECONDS);
}
```

>Timer和ScheduledExecutor都仅能提供基于开始时间与重复间隔的任务调度，不能胜任更加复杂的调度需求。比如，设置每月第一天凌晨1点执行任务、复杂调度任务的管理，任务间传递数据等等。

##### 2.4 Quartz方式实现

Quartz是一个任务调度框架，可以满足更多更复杂的调度需求，核心类包括`Scheduler`，`Job`以及`Trigger`。

- Job负责定义需要执行的任务；
- Trigger负责设置调度策略；
- Scheduler将二者组装在一起，并触发任务开始执行。

Quartz支持简单的按时间间隔调度、还支持按日历调度方式，通过设置CronTrigger表达式（包括：秒、分、时、日、月、周、年）进行任务调整。

```xml
<dependency>
  <groupId>org.quartz-scheduler</groupId>
  <artifactId>quartz</artifactId>
  <version>2.3.2</version>
</dependency>

<dependency>
  <groupId>org.quartz-scheduler</groupId>
  <artifactId>quartz-jobs</artifactId>
  <version>2.3.2</version>
</dependency>
```

编写job：

```java
public class DemoJob implements Job {

  @Override
  public void execute(JobExecutionContext jobExecutionContext) throws JobExecutionException {
    String name = jobExecutionContext.getJobDetail().getJobDataMap().getString("name");
    System.out.println("hello" + name);
  }
}
```

在controller文件里调用：

```java
@RestController
@RequestMapping("/quartz")
public class QuartzController {

  @RequestMapping("/demo")
  public void work() throws SchedulerException {
    //定时器对象
    Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();
    //定义一个工作对象 设置工作名称与组名
    JobDetail job = JobBuilder.newJob(DemoJob.class).withIdentity("job1", "group1").build();

    job.getJobDataMap().put("name", "hello Job");

    //定义一个任务调度的Trigger 设置工作名称与组名 每2秒执行一次
    Trigger trigger = TriggerBuilder.newTrigger().withIdentity("trigger1", "group1").withSchedule(CronScheduleBuilder.cronSchedule("*/2 * * * * ?")).build();
    //设置工作 与触发器
    scheduler.scheduleJob(job, trigger);
    //开始定时任务
    scheduler.start();
  }
}
```

运行，在浏览器中输入localhost:8080/quartz/demo即可在控制台看到每隔2秒钟输出一次。



#### 3. 分布式任务调度

---

通常任务调度的程序是集成在应用中的，比如：优惠券服务中包括了定时发送优惠券的调度程序，结算服务中包括了定期生成报表的任务调度程序，==由于采用分布式架构，一个服务往往会部署多个冗余实例来运行我们的业务，在这种分布式系统环境下运行任务调度，称之为分布式任务调度==。

##### 3.1 分布式调度要实现的目标

不管是任务调度程序集成在应用程序中，还是单独构建的任务调度系统，如果采用分布式调度任务的方式就相当于将任务调度程序分布式构建，这样就可以具有分布式系统的特点，并且提高任务的调度处理能力：

1. 并行任务调度

   并行任务调度实现靠着多线程。如果将任务调度程序分布式部署，每个节点还可以部署为集群，这样就可以让多台计算机共同去完成任务调度。

2. 高可用

   若某一个实例宕机，不影响其他实例来执行任务。

3. 弹性扩容

   当集群中增加实例就可以提高并执行任务的处理效率。

4. 任务管理与检测

   对系统中存在的所有定时任务进行统一的管理及监测。让开发人员及运维人员能够时刻了解任务执行情况，从而做出快速地应急处理响应。

5. 避免任务重复执行

   需要控制相同的任务在多个运行实例上只执行一次，考虑采用下边的方法：

   - 分布式锁，多个实例在任务执行前首先需要获取锁，如果获取失败那么就证明有其他服务已经在运行。如果获取成功那么证明没有服务在运行定时任务，则执行。（实现方式：redis等）
   - Zookeeper选举，利用Zookeeper对Leader实例执行定时任务，有其他业务已经使用了ZK，那么执行定时任务的时候判断自己是否是leader，如果不是则不执行。



#### 4. elastic-job介绍

---

市场上的分布式任务调度产品：

- Elastic-Job：当当网基于quartz二次开发的弹性分布式任务调度系统，功能丰富强大，采用zookeeper实现分布式协调，实现任务高可用以及分片。
- Saturn：唯品会开源的一个分布式任务调度平台，可以全域统一配置，统一监控，任务高可用以及分片并发处理。它是在elastic-job基础之上改良出来的。
- xxl-job：大众点评的分布式任务调度平台，是一个轻量级分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。
- TBSchedule：淘宝的一款高性能分布式调度框架，目前被应用于阿里、京东、支付宝、国美等很多互联网企业的流程调度系统中。

>Elastic-Job由两个相互独立的子项目Elastic-Job-Lite和Elastic-Job-Cloud组成，使用`Elastic-Job`可以快速实现分布式任务调度。


