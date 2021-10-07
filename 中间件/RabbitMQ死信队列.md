为了保证订单业务的消息数据不丢失，需要使用到RabbitMQ的死信队列机制，当消息消费发生异常时，将消息投入死信队列中。

#### 1. 死信队列是什么

---

Dead Letter

1. 消息被否定确认，使用`channel.basicNack`或`channel.basicReject`，并且此时requeue属性被设置为false。
2. 消息在队列的存活时间超过设置的TTL时间。
3. 消息队列的消息数量已经超过最大队列长度。

满足以上其中一条的消息称为“死信”。“死信”消息会被RabbitMQ进行特殊处理，==如果配置了死信队列信息，那么该消息将会被丢进死信队列中，如果没有配置，则该消息将会被丢弃==。



#### 2. 配置死信队列

---

1. 配置业务队列，绑定到业务交换机上
2. ==为业务队列配置死信交换机和路由key==
3. 为业务队列==配置死信队列==

注意，并不是直接声明一个公共的死信队列，然后所以死信消息就自己跑到死信队列里去了。而是为每个需要使用死信的业务队列配置一个死信交换机，这里同一个项目的死信交换机可以共用一个，然后为每个业务队列分配一个单独的路由key。

有了死信交换机和路由key后，接下来，就像配置业务队列一样，配置死信队列，然后绑定在死信交换机上。也就是说，死信队列并不是什么特殊的队列，只不过是绑定在死信交换机上的队列。死信交换机也不是什么特殊的交换机，只不过是用来接受死信的交换机，所以可以为任何类型【Direct、Fanout、Topic】。==一般来说，会为每个业务队列分配一个独有的路由key，并对应的配置一个死信队列进行监听==。



> 创建Rabbit配置类：
>
> 声明两个Exchange，一个业务Exchange，一个死信Exchange，业务Exchange下绑定两个业务队列dead.letter.demo.simple.business.queuea，dead.letter.demo.simple.business.queueb，两个业务队列都绑定同一个死信Exchange，分别配置路由key，在死信Exchange下绑定了两个死信队列，设置的路由key分别为业务队列里配置的路由key。
>
> ```java
> @Configuration
> public class RabbitMQConfig {
>   public static final String BUSINESS_EXCHANGE_NAME = "dead.letter.demo.simple.business.exchange";
>     public static final String BUSINESS_QUEUEA_NAME = "dead.letter.demo.simple.business.queuea";
>     public static final String BUSINESS_QUEUEB_NAME = "dead.letter.demo.simple.business.queueb";
>     public static final String DEAD_LETTER_EXCHANGE = "dead.letter.demo.simple.deadletter.exchange";
>     public static final String DEAD_LETTER_QUEUEA_ROUTING_KEY = "dead.letter.demo.simple.deadletter.queuea.routingkey";
>     public static final String DEAD_LETTER_QUEUEB_ROUTING_KEY = "dead.letter.demo.simple.deadletter.queueb.routingkey";
>     public static final String DEAD_LETTER_QUEUEA_NAME = "dead.letter.demo.simple.deadletter.queuea";
>     public static final String DEAD_LETTER_QUEUEB_NAME = "dead.letter.demo.simple.deadletter.queueb";
> 
>     /**
>      * 声明业务Exchange
>      */
>     @Bean("businessExchange")
>     public FanoutExchange businessExchange() {
>         return new FanoutExchange(BUSINESS_EXCHANGE_NAME);
>     }
> 
>     /**
>      * 声明死信Exchange
>      */
>     @Bean("deadLetterExchange")
>     public DirectExchange deadLetterExchange() {
>         return new DirectExchange(DEAD_LETTER_EXCHANGE);
>     }
> 
>     /**
>      * 声明业务队列A
>      */
>     @Bean("businessQueueA")
>     public Queue businessQueueA() {
>         Map<String, Object> args = new HashMap<>(2);
>         // 声明当前队列绑定的死信交换机
>         args.put("x-dead-letter-exchange", DEAD_LETTER_EXCHANGE);
>         // 声明当前队列的私信路由key
>         args.put("x-dead-letter-routing-key", DEAD_LETTER_QUEUEA_ROUTING_KEY);
>         return QueueBuilder.durable(BUSINESS_QUEUEA_NAME).withArguments(args).build();
>     }
> 
>     /**
>      * 声明业务队列B
>      */
>     @Bean("businessQueueB")
>     public Queue businessQueueB() {
>         Map<String, Object> args = new HashMap<>(2);
>         // 声明当前队列绑定的死信交换机
>         args.put("x-dead-letter-exchange", DEAD_LETTER_EXCHANGE);
>         // 声明当前队列的私信路由key
>         args.put("x-dead-letter-routing-key", DEAD_LETTER_QUEUEB_ROUTING_KEY);
>         return QueueBuilder.durable(BUSINESS_QUEUEB_NAME).withArguments(args).build();
>     }
> 
>     /**
>      * 声明死信队列A
>      */
>     @Bean("deadLetterQueueA")
>     public Queue deadLetterQueueA() {
>         return new Queue(DEAD_LETTER_QUEUEA_NAME);
>     }
> 
>     /**
>      * 声明死信队列B
>      */
>     @Bean("deadLetterQueueB")
>     public Queue deadLetterQueueB() {
>         return new Queue(DEAD_LETTER_QUEUEB_NAME);
>     }
> 
>     /**
>      * 声明业务队列A绑定关系
>      */
>     @Bean
>     public Binding businessBindingA(@Qualifier("businessQueueA") Queue queue,
>                                     @Qualifier("businessExchange") FanoutExchange exchange) {
>         return BindingBuilder.bind(queue).to(exchange);
>     }
> 
>     /**
>      * 声明业务队列B绑定关系
>      */
>     @Bean
>     public Binding businessBindingB(@Qualifier("businessQueueB") Queue queue,
>                                     @Qualifier("businessExchange") FanoutExchange exchange) {
>         return BindingBuilder.bind(queue).to(exchange);
>     }
> 
>     /**
>      * 声明死信队列A绑定关系
>      */
>     @Bean
>     public Binding deadLetterBindingA(@Qualifier("deadLetterQueueA") Queue queue,
>                                       @Qualifier("deadLetterExchange") DirectExchange exchange) {
>         return BindingBuilder.bind(queue).to(exchange).with(DEAD_LETTER_QUEUEA_ROUTING_KEY);
>     }
> 
>     /**
>      * 声明死信队列B绑定关系
>      */
>     @Bean
>     public Binding deadLetterBindingB(@Qualifier("deadLetterQueueB") Queue queue,
>                                       @Qualifier("deadLetterExchange") DirectExchange exchange) {
>         return BindingBuilder.bind(queue).to(exchange).with(DEAD_LETTER_QUEUEB_ROUTING_KEY);
>     }
> }

>配置文件application.yml
>
>```yaml
>spring:
>  rabbitmq:
>    host: localhost
>    password: guest
>    username: guest
>    listener:
>      type: simple
>      simple:
>          default-requeue-rejected: false
>          acknowledge-mode: manual
>```



**业务队列消费**

```java
@Slf4j
@Component
public class BusinessMessageReceiver {

  /**
   * 监听业务队列A
   */
  @RabbitListener(queues = RabbitMQConfig.BUSINESS_QUEUEA_NAME)
  public void receiveA(Message message, Channel channel) throws IOException {
    String msg = new String(message.getBody());
    log.info(">>>>>>>>>>>>>收到的业务消息A：{}", msg);

    boolean ack = true;
    Exception exception = null;
    try {
      if (msg.contains("deadletter")) {
        throw new RuntimeException("dead letter exception");
      }
    } catch (Exception e) {
      ack = false;
      exception = e;
    }
    if (!ack) {
      log.error(">>>>>>>>>>>>>消息消费发生异常，error msg：{}", exception.getMessage(), exception);
      channel.basicNack(message.getMessageProperties().getDeliveryTag(), false, false);
    } else {
      channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
    }
  }

  /**
   * 监听消费队列B
   */
  @RabbitListener(queues = RabbitMQConfig.BUSINESS_QUEUEB_NAME)
  public void receiveB(Message message, Channel channel) throws IOException {
    System.out.println("收到业务消息B：" + new String(message.getBody()));
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
  }
}
```

```java
@Slf4j
@Component
public class DeadLetterMessageReceiver {

  /**
   * 监听死信队列A
   */
  @RabbitListener(queues = RabbitMQConfig.DEAD_LETTER_QUEUEA_NAME)
  public void receiveA(Message message, Channel channel) throws IOException {
    log.info(">>>>>>>>>>>>>>>>收到死信消息A：" + new String(message.getBody()));
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
  }

  /**
   * 监听死信队列B
   */
  @RabbitListener(queues = RabbitMQConfig.DEAD_LETTER_QUEUEB_NAME)
  public void receiveB(Message message, Channel channel) throws IOException {
    log.info(">>>>>>>>>>>>>>>>收到死信消息B：" + new String(message.getBody()));
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
  }
}
```

简单消息生产者：

```java
@Component
public class BusinessMassageSender {

  @Autowired
  private RabbitTemplate rabbitTemplate;

  public void sendMsg(String msg) {
    rabbitTemplate.convertSendAndReceive(RabbitMQConfig.BUSINESS_EXCHANGE_NAME, "", msg);
  }
}
```

测试：

```java
@RequestMapping("rabbitmq")
@RestController
public class RabbitMQMsgController {

  @Autowired
  private BusinessMassageSender sender;

  @RequestMapping("sendmsg")
  public void sendMsg(String msg) {
    sender.sendMsg(msg);
  }
}
```

启动，可以看到RabbitMQ的管理后台中看到一共四个队列，除默认的Exchange外还声明了两个Exchange。

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gu7tp20cogj60t10fhwfa02.jpg" style="zoom:67%;" />

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gu7tn3d69kj60p80h1q3l02.jpg" style="zoom:67%;" />

访问localhost:8080/rabbitmq/sendmsg?msg=hello

日志：

```
>>>>>>>>>>>>>收到的业务消息A：hello
>>>>>>>>>>>>>收到的业务消息B：hello
```

表示两个Consumer都正常收到了消息。这代表正常消费的消息，ack后正常返回。然后测试nck的消息。

访问localhost:8080/rabbitmq/sendmsg?msg=deadletter，将会触发业务队列A的NCK，按照预期，消息被NCK后，会抛到死信队列中，因此死信队列将会出现这个消息，日志如下：

```
>>>>>>>>>>>>>收到的业务消息A：deadletter
>>>>>>>>>>>>>消息消费发生异常，error msg：dead letter exception
java.lang.RuntimeException: dead letter exception
...
>>>>>>>>>>>>>>>>收到死信消息A：deadletter
```

可以看到，死信队列的Consumer接受到了这个消息。



#### 3. 死信消息的变化

---

“死信”被丢到死信队列中后，会发生什么变化呢？

如果队列配置了参数==x-dead-letter-routing-key==的话，“死信”的路由key将会被替换成该参数对应的值。如果没有设置，则保留该消息原有的路由key。

> 例如：
>
> 原有消息的路由key是testA，被发送到业务Exchage中，然后被投递到业务队列QueueA中，如果该队列没有配置参数`x-dead-letter-routing-key`，则该消息成为死信后，将保留原有的路由key testA，如果配置了该参数，并且值设置为testB，那么该消息成为死信后，路由key将会被替换为testB，然后被抛到死信交换机中。
>
> 另外，由于被抛到了死信交换机，所以消息的Exchange Name也会被替换为死信交换机的名称。消息Header中，也会添加很多字段，添加日志输出：`log.info(">>>>>>>>>>>>>>>>死信消息properties：{}", message.getMessageProperties());`即可得到死信消息Header中被添加的信息：
>
> ```
> >>>>>>>>>>>>>>>>死信消息properties：MessageProperties [headers={x-first-death-exchange=dead.letter.demo.simple.business.exchange, x-death=[{reason=rejected, count=1, exchange=dead.letter.demo.simple.business.exchange, time=Tue Sep 07 10:18:23 CST 2021, routing-keys=[], queue=dead.letter.demo.simple.business.queuea}], x-first-death-reason=rejected, x-first-death-queue=dead.letter.demo.simple.business.queuea}, correlationId=1, replyTo=amq.rabbitmq.reply-to.g1h2AA1yZXBseUA5MjQ4ODg0AABEHAAAAAJhKmML.Qe9B1Vmdlov9mwBSM4B0ig==, contentType=text/plain, contentEncoding=UTF-8, contentLength=0, receivedDeliveryMode=PERSISTENT, priority=0, redelivered=false, receivedExchange=dead.letter.demo.simple.deadletter.exchange, receivedRoutingKey=dead.letter.demo.simple.deadletter.queuea.routingkey, deliveryTag=1, consumerTag=amq.ctag-wtsfIZ_1dd5T353B868h5A, consumerQueue=dead.letter.demo.simple.deadletter.queuea]
> ```
>
> | 字段名                 | 含义                                                         |
> | ---------------------- | ------------------------------------------------------------ |
> | x-first-death-exchange | 第一次被抛入的死信交换机的名称                               |
> | x-first-death-reason   | 第一次成为死信的原因，<br />`rejected`：消息在重新进入队列时被队列拒绝，由于`default-requeue-rejected` 参数被设置为`false`。<br />`expired` ：消息过期。`maxlen` ： 队列内消息数量超过队列最大容量 |
> | x-first-death-queue    | 第一次成为死信前所在队列名称                                 |
> | x-death                | 历次被投入死信交换机的信息列表，同一个消息每次进入一个死信交换机，这个数组的信息就会被更新 |



#### 4. 死信队列应用场景

---

一般用在较为重要的业务队列中，确保未被正确消费的消息不被丢弃，一般发生消费异常可能原因主要是由于消息本身存在错误导致处理异常，处理过程中参数校验异常，或者因网络波动导致的查询异常等等，当发生异常时，当然不能每次通过日志获取原消息，然后让运维帮忙重新投递消息。==通过配置死信队列，可以让未正确处理的消息暂存到另一个队列中，待后续排查清楚问题后，编写相应的处理代码来处理死信消息，这样比手工恢复数据要好太多了==。



#### 5. 总结

---

死信消息的生命周期：

1. 业务消息被投入业务队列
2. 消费者消费业务队列的消息，由于处理过程中发生异常，于是进行了==nck==或者==reject==操作
3. 被nck或reject的消息由RabbitMQ投递到死信交换机中
4. 死信交换机将消息投入相应的死信队列
5. 死信队列的消费者消费死信消息

死信消息是RabbitMQ为我们做的一层保证，其实我们也可以不使用死信队列，而是在消息消费异常时，将消息主动投递到另一个交换机中，当你明白了这些之后，这些Exchange和Queue想怎样配合就能怎么配合。比如从死信队列拉取消息，然后发送邮件、短信、钉钉通知来通知开发人员关注。或者==将消息重新投递到一个队列然后设置过期时间，来进行延时消费==。