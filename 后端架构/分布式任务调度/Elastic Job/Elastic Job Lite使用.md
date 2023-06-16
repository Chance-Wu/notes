Elastic-Job-Lite定位为轻量级无中心化解决方案，使用jar包的形式提供分布式任务的协调服务，外部依赖仅Zookeeper。



#### 1. 引入Maven依赖

---

```xml
<dependency>
  <groupId>org.apache.shardingsphere.elasticjob</groupId>
  <artifactId>elasticjob-lite-core</artifactId>
  <version>${latest.release.version}</version>
</dependency>
```



#### 2. 配置ZK注册中心

---

在使用的时候，首先要配置zk注册中心。

```java
@Slf4j
@Configuration
@ConditionalOnExpression("'${regCenter.serverList}'.length() >0")
public class JobRegistryCenterConfig {

    @Value("${regCenter.serverList}")
    private String serverList;

    @Value("${regCenter.namespace}")
    private String namespace;

    @Bean(initMethod = "init")
    public ZookeeperRegistryCenter regCenter() {
        log.info("serverLists is {}", serverList);
        return new ZookeeperRegistryCenter(new ZookeeperConfiguration(serverList, namespace));
    }
}
```



#### 2. 实现一个有业务逻辑的Job

---

实现SimpleJob接口：

```java
public class MyJob implements SimpleJob {

  @Override
  public void execute(ShardingContext context) {
    switch (context.getShardingItem()) {
      case 0: 
        // do something by sharding item 0
        break;
      case 1: 
        // do something by sharding item 1
        break;
      case 2: 
        // do something by sharding item 2
        break;
        // case n: ...
    }
  }
}
```



#### 3. 作业配置

---

```java
@Configuration
public class MyJobConfig {

  /**
   * cron表达式，用于控制作业触发时间
   */
  private final String cron = "0/5 * * * * ?";

  /**
   * 作业分片总数
   */
  private final int shardingTotalCount = 3;

  /**
   * 分片序列号和参数用等号分隔，多个键值对用逗号分隔
   * 分片序列号从0开始，不可大于或等于作业分片总数
   * 如：
   * 0=a,1=b,2=c
   */
  private final String shardingItemParameters = "0=A,1=B,2=C";

  /**
   * 作业自定义参数
   * 作业自定义参数，可通过传递该参数为作业调度的业务方法传参，用于实现带参数的作业
   * 例：每次获取的数据量、作业实例从数据库读取的主键等。
   */
  private final String jobParameters = "parameter";

  @Autowired
  private ZookeeperRegistryCenter regCenter;

  @Bean
  public SimpleJob stockJob() {
    return new MySimpleJob();
  }

  @Bean(initMethod = "init")
  public JobScheduler simpleJobScheduler(final SimpleJob simpleJob) {
    return new SpringJobScheduler(simpleJob, regCenter, getLiteJobConfiguration(simpleJob.getClass(), cron, shardingTotalCount, shardingItemParameters, jobParameters));
  }

  private LiteJobConfiguration getLiteJobConfiguration(final Class<? extends SimpleJob> jobClass,
                                                       final String cron,
                                                       final int shardingTotalCount,
                                                       final String shardingItemParameters,
                                                       final String jobParameters) {
    // 定义作业核心配置
    JobCoreConfiguration simpleCoreConfig = JobCoreConfiguration.newBuilder(jobClass.getName(), cron, shardingTotalCount).shardingItemParameters(shardingItemParameters).jobParameter(jobParameters).build();
    // 定义SIMPLE类型配置
    SimpleJobConfiguration simpleJobConfig = new SimpleJobConfiguration(simpleCoreConfig, jobClass.getCanonicalName());
    // 定义Lite作业根配置
    LiteJobConfiguration simpleJobRootConfig = LiteJobConfiguration.newBuilder(simpleJobConfig).overwrite(true).build();
    return simpleJobRootConfig;
  }
}
```



#### 4. 源码学习

---

通过上面创建的demo,整体熟悉一下elastic-job-lite的代码流。

入口:SpringJobScheduler
参数:(ElasticJob, CoordinatorRegistryCenter, LiteJobConfiguration)
可以看到分别传入了:
自己实现的SpringBootSimpleJob、zookeeperRegistryCenter、getLiteJobConfiguration()


getLiteJobConfiguration()方法对我们传入的job信息进行了配置,最后返回了LiteJobConfiguration。
上面的DEMO只传入了少量的参数cron,shardingTotalCount,shardingItemParameters
还有更多的参数可以配置,可以参考JobCoreConfiguration这个类里面的字段信息。


SpringJobScheduler extends JobScheduler:
直接看 JobScheduler 里面

     /**
     * 初始化作业.
     */
    public void init() {
    
        //去ZK 保存或者更新 job的信息  (ZookeeperRegistryCenter)
        LiteJobConfiguration liteJobConfigFromRegCenter = schedulerFacade.updateJobConfiguration(liteJobConfig);
    
        //单列模式实现的一个缓存 (当前分片总数)
        JobRegistry.getInstance().setCurrentShardingTotalCount(liteJobConfigFromRegCenter.getJobName(), liteJobConfigFromRegCenter.getTypeConfig().getCoreConfig().getShardingTotalCount());
    
        //初始化 JobScheduleController
        JobScheduleController jobScheduleController = new JobScheduleController(
                createScheduler(), createJobDetail(liteJobConfigFromRegCenter.getTypeConfig().getJobClass()), liteJobConfigFromRegCenter.getJobName());
    
        JobRegistry.getInstance().registerJob(liteJobConfigFromRegCenter.getJobName(), jobScheduleController, regCenter);
    
        //注册作业的启动信息
        schedulerFacade.registerStartUpInfo(!liteJobConfigFromRegCenter.isDisabled());
    
        //启动Quartz调度
        jobScheduleController.scheduleJob(liteJobConfigFromRegCenter.getTypeConfig().getCoreConfig().getCron());
    }


JobScheduleController初始化:
创建Quartz的调度器
指定Quartz调度时候执行任务的具体实现类LiteJob


registerStartUpInfo注册作业的启动信息:
利用ZK选举当前作业的主节点
持久化作业服务器上线信息
持久化作业运行实例上线相关信息


scheduleJob()加入Quartz调度并且进行调度


上面已经知道了具体在调度中执行的类是LiteJob,继续看LiteJob中的execute()

    @Override
    public void execute(final JobExecutionContext context) throws JobExecutionException {
        JobExecutorFactory.getJobExecutor(elasticJob, jobFacade).execute();
    }

getJobExecutor:
获取作业执行器,我们使用的是SimpleJobExecutor

execute()方法会走到AbstractElasticJobExecutor中,进行作业的执行。

暂时只关注三个流程
1-获取当前作业服务器的分片上下文.

ShardingContexts shardingContexts = jobFacade.getShardingContexts();

getShardingContexts()方法中做的事情:

判断是否需要分片,如果需要就进行分片(调用了shardingService.shardingIfNecessary()方法进行判断)
在shardingIfNecessary()中会判断当前节点是否是主节点,如果是子节点 在这里 等待分片的完成!!!  不然不能执行后续代码。
进行具体分片：调用JobShardingStrategy.sharding()方法进行分片,有几种实现类,先不管。
分片结束会进程将分片信息保存到ZK:
curatorTransactionFinal.create().forPath(jobNodePath.getFullPath(ShardingNode.getInstanceNode(shardingItem)), entry.getKey().getJobInstanceId().getBytes()).and();
entry.getKey().getJobInstanceId()可以简单理解为一个服务器
大致保存的内容可以理解为：
服务器1 中  有 [0,1,2]分片
服务器2 中  有 [3,4,5]分片


2-分片完成后,获取当前机器需要执行的分片任务!!! (当前作业的主节点会进行分片,子节点会等待主节点分片完成, 最后都会走分片后的逻辑。)

List<Integer> shardingItems = shardingService.getLocalShardingItems();

 public List<Integer> getShardingItems(final String jobInstanceId) {
        JobInstance jobInstance = new JobInstance(jobInstanceId);
        if (!serverService.isAvailableServer(jobInstance.getIp())) {
            return Collections.emptyList();
        }
        List<Integer> result = new LinkedList<>();
        int shardingTotalCount = configService.load(true).getTypeConfig().getCoreConfig().getShardingTotalCount();
        for (int i = 0; i < shardingTotalCount; i++) {
            if (jobInstance.getJobInstanceId().equals(jobNodeStorage.getJobNodeData(ShardingNode.getInstanceNode(i)))) {
                result.add(i);
            }
        }
        return result;
    }

可以看到同样是根据getJobInstanceId  来获取 分片任务的,和之前呼应。


3-真正的执行分片的作业
具体代码不展示了
最后会调用抽象方法protected abstract void process(ShardingContext shardingContext);

之前已经知道了具体的实现类:SimpleJobExecutor:

    @Override
    protected void process(final ShardingContext shardingContext) {
        simpleJob.execute(shardingContext);
    }


可以看到这里真正调用了我们的 simpleJob (具体实现类SpringBootSimpleJob)  的  execute方法。