#### 1. 什么是延时队列

---

首先它是一种队列，队列意味着内部的元素是有序的，元素出队和入队是有方向性的，元素从一端进入，从另一端取出。

其次，延时队列，最重要的特性就体现在它的延时属性上，跟普通的队列不一样的是，普通队列中的元素总是等着希望被早点取出处理，而延时队列中的元素则是希望被在指定时间得到取出和处理，所以==延时队列中的元素是都是带时间属性的==，通常来说是需要被处理的消息或者任务。



#### 2. 延时队列使用场景

---

1. 订单在十分钟之内未支付则自动取消。
2. 新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒。
3. 账单在一周内未支付，则自动结算。
4. 用户注册成功后，如果三天内没有登录则进行短信提醒。
5. 用户发起退款，如果三天内没有得到处理则通知相关运营人员。
6. 预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议。

这些场景都有一个特点，需要在某个事件发生之后或者之前的指定时间点完成某一项任务。



#### 3. RabbitMQ中的TTL

---

RabbitMQ的高级特性TTL（Time to Live）。

TTL是RabbitMQ中一个消息或者队列的属性，表明==一条消息或者该队列中的所有消息的最大存活时间==，单位是==毫秒==。换句话说，如果一条消息设置了TTL属性或者进入了设置TTL属性的队列，那么==这条消息如果在TTL设置的时间内没有被消费，则会成为“死信”==。如果同时配置了队列的TTL和消息的TTL，那么较小的那个值将会被使用。



>**设置TTL**
>
>方式一：在创建队列的时候设置队列的==x-message-ttl==属性
>
>```java
>Map<String, Object> args = new HashMap<String, Object>();
>args.put("x-message-ttl", 6000);
>channel.queueDeclare(queueName, durable, exclusive, autoDelete, args);
>```
>
>方式二：针对每条消息设置TTL
>
>```java
>AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
>builder.expiration("6000");
>AMQP.BasicProperties properties = builder.build();
>channel.basicPublish(exchangeName, routingKey, mandatory, properties, "msg body".getBytes());
>```
>
>区别：
>
>- 如果设置了队列的TTL属性，那么一旦消息过期，就会被队列丢弃；
>- 而第二种方式，消息即使过期，也不一定会被马上丢弃，因为消息是否过期是在即将投递到消费者之前判定的，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间。



如果不设置TTL，表示消息永远不会过期，如果将TTL设置为0，则表示除非此时可以直接投递该消息到消费者，否则该消息将会被丢弃。



#### 4. 利用RabbitMQ实现延时队列

---

延时队列，不就是想要消息延迟多久被处理吗，TTL则刚好能让消息在延迟多久之后成为死信，另一方面，成为死信的消息都会被投递到死信队列里，这样只需要消费者一直消费死信队列里的消息就万事大吉了，因为里面的消息都是希望被立即处理的消息。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gu81qmvnolj61i00iaaaq02.jpg)

生产者生产一条延时消息，根据需要延时时间的不同，利用不同的routingkey将消息路由到不同的延时队列，每个队列都设置了不同的TTL属性，并绑定在同一个死信交换机中，消息过期后，根据routingkey的不同，又会被路由到不同的死信队列中，消费者只需要监听对应的死信队列进行处理即可。



声明交换机、队列以及他们的绑定关系：

```java
@Component
public class RabbitMQDelayConfig {

  public static final String DELAY_EXCHANGE_NAME = "delay.queue.demo.business.exchange";
  public static final String DELAY_QUEUEA_NAME = "delay.queue.demo.business.queuea";
  public static final String DELAY_QUEUEB_NAME = "delay.queue.demo.business.queueb";
  public static final String DELAY_QUEUEA_ROUTING_KEY = "delay.queue.demo.business.queuea.routingkey";
  public static final String DELAY_QUEUEB_ROUTING_KEY = "delay.queue.demo.business.queueb.routingkey";
  public static final String DEAD_LETTER_EXCHANGE = "delay.queue.demo.deadletter.exchange";
  public static final String DEAD_LETTER_QUEUEA_ROUTING_KEY = "delay.queue.demo.deadletter.delay_10s.routingkey";
  public static final String DEAD_LETTER_QUEUEB_ROUTING_KEY = "delay.queue.demo.deadletter.delay_60s.routingkey";
  public static final String DEAD_LETTER_QUEUEA_NAME = "delay.queue.demo.deadletter.queuea";
  public static final String DEAD_LETTER_QUEUEB_NAME = "delay.queue.demo.deadletter.queueb";

  // 声明延时Exchange
  @Bean("delayExchange")
  public DirectExchange delayExchange() {
    return new DirectExchange(DELAY_EXCHANGE_NAME);
  }

  // 声明死信Exchange
  @Bean("deadLetterExchange")
  public DirectExchange deadLetterExchange() {
    return new DirectExchange(DEAD_LETTER_EXCHANGE);
  }

  // 声明延时队列A 延时10s
  // 并绑定到对应的死信交换机
  @Bean("delayQueueA")
  public Queue delayQueueA() {
    Map<String, Object> args = new HashMap<>(2);
    // x-dead-letter-exchange    这里声明当前队列绑定的死信交换机
    args.put("x-dead-letter-exchange", DEAD_LETTER_EXCHANGE);
    // x-dead-letter-routing-key  这里声明当前队列的死信路由key
    args.put("x-dead-letter-routing-key", DEAD_LETTER_QUEUEA_ROUTING_KEY);
    // x-message-ttl  声明队列的TTL
    args.put("x-message-ttl", 6000);
    return QueueBuilder.durable(DELAY_QUEUEA_NAME).withArguments(args).build();
  }

  // 声明延时队列B 延时 60s
  // 并绑定到对应的死信交换机
  @Bean("delayQueueB")
  public Queue delayQueueB() {
    Map<String, Object> args = new HashMap<>(2);
    // x-dead-letter-exchange    这里声明当前队列绑定的死信交换机
    args.put("x-dead-letter-exchange", DEAD_LETTER_EXCHANGE);
    // x-dead-letter-routing-key  这里声明当前队列的死信路由key
    args.put("x-dead-letter-routing-key", DEAD_LETTER_QUEUEB_ROUTING_KEY);
    // x-message-ttl  声明队列的TTL
    args.put("x-message-ttl", 60000);
    return QueueBuilder.durable(DELAY_QUEUEB_NAME).withArguments(args).build();
  }

  // 声明死信队列A 用于接收延时10s处理的消息
  @Bean("deadLetterQueueA")
  public Queue deadLetterQueueA() {
    return new Queue(DEAD_LETTER_QUEUEA_NAME);
  }

  // 声明死信队列B 用于接收延时60s处理的消息
  @Bean("deadLetterQueueB")
  public Queue deadLetterQueueB() {
    return new Queue(DEAD_LETTER_QUEUEB_NAME);
  }

  // 声明延时队列A绑定关系
  @Bean
  public Binding delayBindingA(@Qualifier("delayQueueA") Queue queue,
                               @Qualifier("delayExchange") DirectExchange exchange) {
    return BindingBuilder.bind(queue).to(exchange).with(DELAY_QUEUEA_ROUTING_KEY);
  }

  // 声明业务队列B绑定关系
  @Bean
  public Binding delayBindingB(@Qualifier("delayQueueB") Queue queue,
                               @Qualifier("delayExchange") DirectExchange exchange) {
    return BindingBuilder.bind(queue).to(exchange).with(DELAY_QUEUEB_ROUTING_KEY);
  }

  // 声明死信队列A绑定关系
  @Bean
  public Binding deadLetterBindingA(@Qualifier("deadLetterQueueA") Queue queue,
                                    @Qualifier("deadLetterExchange") DirectExchange exchange) {
    return BindingBuilder.bind(queue).to(exchange).with(DEAD_LETTER_QUEUEA_ROUTING_KEY);
  }

  // 声明死信队列B绑定关系
  @Bean
  public Binding deadLetterBindingB(@Qualifier("deadLetterQueueB") Queue queue,
                                    @Qualifier("deadLetterExchange") DirectExchange exchange) {
    return BindingBuilder.bind(queue).to(exchange).with(DEAD_LETTER_QUEUEB_ROUTING_KEY);
  }
}
```

创建两个消费者，分别对两个死信队列的消息进行消费：

```java
@Slf4j
@Component
public class DeadLetterQueueConsumer {

  @RabbitListener(queues = DEAD_LETTER_QUEUEA_NAME)
  public void receiveA(Message message, Channel channel) throws IOException {
    String msg = new String(message.getBody());
    log.info("当前时间：{},死信队列A收到消息：{}", new Date().toString(), msg);
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
  }

  @RabbitListener(queues = DEAD_LETTER_QUEUEB_NAME)
  public void receiveB(Message message, Channel channel) throws IOException {
    String msg = new String(message.getBody());
    log.info("当前时间：{},死信队列B收到消息：{}", new Date().toString(), msg);
    channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
  }
}
```

消息生产者：

```java
public enum DelayTypeEnum {

  DELAY_6S(6000L, "延时6s"),
  DELAY_60S(60000L, "延时60s");

  private Long ttl;
  private String desc;

  DelayTypeEnum(Long ttl, String desc) {
    this.ttl = ttl;
    this.desc = desc;
  }

  public static DelayTypeEnum getDelayTypeEnumByValue(Integer delayType) {
    if (delayType == 1) {
      return DELAY_6S;
    } else if (delayType == 2) {
      return DELAY_60S;
    } else {
      return null;
    }
  }
}

```

```java
@Component
public class DelayMessageSender {

  @Autowired
  private RabbitTemplate rabbitTemplate;

  public void sendMsg(String msg, DelayTypeEnum type){
    switch (type){
      case DELAY_6S:
        rabbitTemplate.convertAndSend(DELAY_EXCHANGE_NAME, DELAY_QUEUEA_ROUTING_KEY, msg);
        break;
      case DELAY_60S:
        rabbitTemplate.convertAndSend(DELAY_EXCHANGE_NAME, DELAY_QUEUEB_ROUTING_KEY, msg);
        break;
    }
  }
}
```

测试：

```java
@Slf4j
@RequestMapping("rabbitmq")
@RestController
public class RabbitMQDelayMsgController {

  @Autowired
  private DelayMessageSender delayMessageSender;

  @RequestMapping("sendmsg")
  public void sendMsg(String msg, Integer delayType) {
    log.info("当前时间：{},收到请求，msg:{},delayType:{}", new Date(), msg, delayType);
    delayMessageSender.sendMsg(msg, Objects.requireNonNull(DelayTypeEnum.getDelayTypeEnumByValue(delayType)));
  }
}
```

发送几条消息，http://localhost:8080/rabbitmq/sendmsg?msg=testMsg1&delayType=1 http://localhost:8080/rabbitmq/sendmsg?msg=testMsg2&delayType=2

日志如下：

```
当前时间：Tue Sep 07 16:06:34 CST 2021,收到请求，msg:testMsg1,delayType:1
当前时间：Tue Sep 07 16:06:40 CST 2021,死信队列A收到消息：testMsg1
当前时间：Tue Sep 07 16:08:14 CST 2021,收到请求，msg:testMsg2,delayType:2
当前时间：Tue Sep 07 16:09:14 CST 2021,死信队列B收到消息：testMsg2
```

第一条消息在6s后变成了死信消息，然后被消费者消费掉，第二条消息在60s之后变成了死信消息，然后被消费掉。

>注意：如果这样使用的话，岂不是每增加一个新的时间需求，就要新增一个队列，这里只有6s和60s两个时间选项，如果需要一个小时后处理，那么就需要增加TTL为一个小时的队列。



#### 5. RabbitMQ延时队列优化

---

针对上述的问题，需要一种更通用的方案才能满足需求，只能将TTL设置在消息属性里。增加一个延时队列，用于接收设置为任意延时时长的消息，增加一个相应的死信队列和routingkey。

```java
public static final String DELAY_QUEUEC_NAME = "delay.queue.demo.business.queuec";
public static final String DELAY_QUEUEC_ROUTING_KEY = "delay.queue.demo.business.queuec.routingkey";
public static final String DEAD_LETTER_QUEUEC_ROUTING_KEY = "delay.queue.demo.deadletter.delay_anytime.routingkey";
public static final String DEAD_LETTER_QUEUEC_NAME = "delay.queue.demo.deadletter.queuec";

// 声明延时队列C 不设置TTL
// 并绑定到对应的死信交换机
@Bean("delayQueueC")
public Queue delayQueueC(){
  Map<String, Object> args = new HashMap<>(3);
  // x-dead-letter-exchange    这里声明当前队列绑定的死信交换机
  args.put("x-dead-letter-exchange", DEAD_LETTER_EXCHANGE);
  // x-dead-letter-routing-key  这里声明当前队列的死信路由key
  args.put("x-dead-letter-routing-key", DEAD_LETTER_QUEUEC_ROUTING_KEY);
  return QueueBuilder.durable(DELAY_QUEUEC_NAME).withArguments(args).build();
}

// 声明死信队列C 用于接收延时任意时长处理的消息
@Bean("deadLetterQueueC")
public Queue deadLetterQueueC(){
  return new Queue(DEAD_LETTER_QUEUEC_NAME);
}

// 声明延时列C绑定关系
@Bean
public Binding delayBindingC(@Qualifier("delayQueueC") Queue queue,
                             @Qualifier("delayExchange") DirectExchange exchange){
  return BindingBuilder.bind(queue).to(exchange).with(DELAY_QUEUEC_ROUTING_KEY);
}

// 声明死信队列C绑定关系
@Bean
public Binding deadLetterBindingC(@Qualifier("deadLetterQueueC") Queue queue,
                                  @Qualifier("deadLetterExchange") DirectExchange exchange){
  return BindingBuilder.bind(queue).to(exchange).with(DEAD_LETTER_QUEUEC_ROUTING_KEY);
}
```

生产者发送延时消息方法：

```java
@Component
public class DelayMessageSender {

  @Autowired
  private RabbitTemplate rabbitTemplate;

  public void sendDelayMsg(String msg, String delayTime) {
    rabbitTemplate.convertSendAndReceive(DELAY_EXCHANGE_NAME, DELAY_QUEUEB_ROUTING_KEY, msg, new MessagePostProcessor() {
      @Override
      public Message postProcessMessage(Message message) throws AmqpException {
        message.getMessageProperties().setExpiration(delayTime);
        return message;
      }
    });
  }
}
```

测试：

```java
@Slf4j
@RequestMapping("rabbitmq")
@RestController
public class RabbitMQDelayMsgController {

  @Autowired
  private DelayMessageSender delayMessageSender;

  @RequestMapping("delayMsg")
  public void delayMsg(String msg, String delayTime) {
    log.info("当前时间：{},收到请求，msg:{},delayType:{}", new Date(), msg, delayTime);
    delayMessageSender.sendDelayMsg(msg, delayTime);
  }
}
```

访问：http://localhost:8080/rabbitmq/delayMsg?msg=testMsg1delayTime=5000 来生产消息。

```
当前时间：Wed Sep 08 08:36:55 CST 2021,收到请求，msg:testMsg1,delayTime:5000
当前时间：Wed Sep 08 08:37:00 CST 2021,死信队列B收到消息：testMsg1
```



>注意：如果使用在消息属性上设置TTL的方式，消息可能并不会按时“死亡”，因为RabbitMQ之后检查第一个消息是否过期，如果过期则丢到死信队列，索引如果第一个消息的延时时长很长，而第二个消息的延时时长很短，则第二个消息并不会优先得到执行。
>
>先发了一个延时时长为20s的消息，然后发了一个延时时长为2s的消息，结果显示，第二个消息会在等第一个消息成为死信后才会“死亡“。



#### 6. 利用RabbitMQ插件实现延迟队列

---

以上如果不能实现在消息粒度上添加TTL，并使其在设置的TTL时间及时死亡，就无法设计成一个通用的延时队列。

安装一个插件即可：https://www.rabbitmq.com/community-plugins.html ，下载rabbitmq_delayed_message_exchange插件，然后解压放置到RabbitMQ的插件目录。接下来，进入RabbitMQ的安装目录下的sbin目录，执行下面命令让该插件生效，然后重启RabbitMQ。

```shell
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

再声明几个Bean：

```java
@Configuration
public class DelayedRabbitMQConfig {
  public static final String DELAYED_QUEUE_NAME = "delay.queue.demo.delay.queue";
  public static final String DELAYED_EXCHANGE_NAME = "delay.queue.demo.delay.exchange";
  public static final String DELAYED_ROUTING_KEY = "delay.queue.demo.delay.routingkey";

  @Bean
  public Queue immediateQueue() {
    return new Queue(DELAYED_QUEUE_NAME);
  }

  @Bean
  public CustomExchange customExchange() {
    Map<String, Object> args = new HashMap<>();
    args.put("x-delayed-type", "direct");
    return new CustomExchange(DELAYED_EXCHANGE_NAME, "x-delayed-message", true, false, args);
  }

  @Bean
  public Binding bindingNotify(@Qualifier("immediateQueue") Queue queue,
                               @Qualifier("customExchange") CustomExchange customExchange) {
    return BindingBuilder.bind(queue).to(customExchange).with(DELAYED_ROUTING_KEY).noargs();
  }
}
```

controller层再添加一个入口：

```java
@RequestMapping("delayMsg2")
public void delayMsg2(String msg, Integer delayTime) {
  log.info("当前时间：{},收到请求，msg:{},delayTime:{}", new Date(), msg, delayTime);
  sender.sendDelayMsg(msg, delayTime);
}
```

消息生产者的代码也需要修改：

```java
public void sendDelayMsg(String msg, Integer delayTime) {
  rabbitTemplate.convertAndSend(DELAYED_EXCHANGE_NAME, DELAYED_ROUTING_KEY, msg, a ->{
    a.getMessageProperties().setDelay(delayTime);
    return a;
  });
}
```

最后再创建一个消费者：

```java
@RabbitListener(queues = DELAYED_QUEUE_NAME)
public void receiveD(Message message, Channel channel) throws IOException {
  String msg = new String(message.getBody());
  log.info("当前时间：{},延时队列收到消息：{}", new Date().toString(), msg);
  channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
}
```