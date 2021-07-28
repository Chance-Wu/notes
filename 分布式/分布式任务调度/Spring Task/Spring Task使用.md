#### 1. 添加Spring Task

```java
@Configuration
@EnableScheduling //开启SpringTask的定时任务能力
public class SpringTaskConfig {
}
```



#### 2. 添加OrderTimeOutCancelTask来执行定时任务

---

```java
@Component
public class OrderTimeOutCancelTask {

  private Logger LOGGER = LoggerFactory.getLogger(OrderTimeOutCancelTask.class);

  /**
   * 每10分钟扫描一次，扫描设定超时时间之前的订单，如果没有支付则取消该订单
   */
  @Scheduled(cron = "0 0/10 * ? * ?")
  private void cancelTimeOutOrder() {
    // do something
    LOGGER.info(">>>>>>取消订单，并根据sku编号释放锁定库存");
  }
}
```

