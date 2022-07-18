#### 1. 引入起步依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

#### 2. application.yml文件中引入RabbitMQ基本的配置信息。

```yaml
spring:
  rabbitmq:
    host: 127.0.0.1
    port: 5672
    username: guest
    password: guest
```

#### 3. 编写RabbitConfig类，类里面设置很多个EXCHANGE，QUEUE，ROUTINGKEY，是为了接下来的不同使用场景。

- Broker：它提供一种传输服务,它的角色就是维护一条从生产者到消费者的路线，保证数据能按照指定的方式进行传输。
- Exchange：消息交换机，它指定消息按什么规则,路由到哪个队列。
- Queue：消息的载体，每个消息都会被投到一个或多个队列。
- Binding：绑定，它的作用就是把exchange和queue按照路由规则绑定起来。
- Routing Key：路由关键字，exchange根据这个关键字进行消息投递。
- vhost：虚拟主机，一个broker里可以有多个vhost，用作不同用户的权限分离。
- Producer：消息生产者，就是投递消息的程序。
- Consumer：消息消费者，就是接受消息的程序。
- Channel：消息通道，在客户端的每个连接里，可建立多个channel。

```java
package com.chance.rabbitmq.demo.component;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Scope;

/**
 * Broker:它提供一种传输服务,它的角色就是维护一条从生产者到消费者的路线，保证数据能按照指定的方式进行传输,
 * Exchange：消息交换机,它指定消息按什么规则,路由到哪个队列。
 * Queue:消息的载体,每个消息都会被投到一个或多个队列。
 * Binding:绑定，它的作用就是把exchange和queue按照路由规则绑定起来.
 * Routing Key:路由关键字,exchange根据这个关键字进行消息投递。
 * vhost:虚拟主机,一个broker里可以有多个vhost，用作不同用户的权限分离。
 * Producer:消息生产者,就是投递消息的程序.
 * Consumer:消息消费者,就是接受消息的程序.
 * Channel:消息通道,在客户端的每个连接里,可建立多个channel.
 *
 * @Description: RabbitConfig
 * @Author: chance
 * @Date: 3/6/21 4:52 PM
 * @Version 1.0
 */
@Configuration
public class RabbitConfig {

    private final Logger logger = LoggerFactory.getLogger(this.getClass());

    /**
     * RabbitMQ 主机
     */
    @Value("${spring.rabbitmq.host}")
    private String host;

    /**
     * RabbitMQ 端口
     */
    @Value("${spring.rabbitmq.port}")
    private int port;

    /**
     * 登录用户以对代理进行身份验证
     */
    @Value("${spring.rabbitmq.username}")
    private String username;

    /**
     * 登录以针对代理进行身份验证
     */
    @Value("${spring.rabbitmq.password}")
    private String password;


    /**
     * Exchange 消息交换机：指定消息按什么规则，路由到那些个队列
     */
    public static final String EXCHANGE_A = "my-mq-exchange_A";
    public static final String EXCHANGE_B = "my-mq-exchange_B";
    public static final String EXCHANGE_C = "my-mq-exchange_C";

    /**
     * Queue 消息的载体：每个消息都会被投到一个或多个队列
     */
    public static final String QUEUE_A = "QUEUE_A";
    public static final String QUEUE_B = "QUEUE_B";
    public static final String QUEUE_C = "QUEUE_C";

    /**
     * Routing Key 路由关键字：exchange根据这个关键字进行消息投递
     */
    public static final String ROUTINGKEY_A = "spring-boot-routingKey_A";
    public static final String ROUTINGKEY_B = "spring-boot-routingKey_B";
    public static final String ROUTINGKEY_C = "spring-boot-routingKey_C";


    @Bean
    public ConnectionFactory connectionFactory() {
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory(host, port);
        connectionFactory.setUsername(username);
        connectionFactory.setPassword(password);
        connectionFactory.setVirtualHost("/");
        connectionFactory.setPublisherConfirms(true);
        return connectionFactory;
    }

    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE) //必须是prototype类型
    public RabbitTemplate rabbitTemplate() {
        RabbitTemplate template = new RabbitTemplate(connectionFactory());
        return template;
    }
}
```

#### 4. 编写消息的生产者

```java
@Component
public class MsgProducer implements RabbitTemplate.ConfirmCallback {

    private final Logger logger = LoggerFactory.getLogger(MsgProducer.class);

    /**
     * 由于rabbitTemplate的scope属性设置为ConfigurableBeanFactory.SCOPE_PROTOTYPE，所以不能自动注入
     */
    private RabbitTemplate rabbitTemplate;

    /**
     * 构造方法注入rabbitTemplate
     */
    @Autowired
    public MsgProducer(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
        // rabbitTemplate如果为单例的话，那回调就是最后设置的内容
        rabbitTemplate.setConfirmCallback(this);
    }

    public void sendMsg(String content) {
        CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
        // 把消息放入ROUTINGKEY_A对应的队列当中去，对应的是队列A
        rabbitTemplate.convertAndSend(RabbitConfig.EXCHANGE_A, RabbitConfig.ROUTINGKEY_A, content, correlationData);
    }

    /**
     * 回调
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        logger.info("回调id：" + correlationData);
        if (ack) {
            logger.info("消息成功消费");
        } else {
            logger.info("消息消费失败：" + cause);
        }
    }
}
```

#### 5. 把交换机，队列，通过路由关键字进行绑定，写在RabbitConfig类当中

```java
/**===============================把交换机，队列，通过路由关键字进行绑定======================================*/

/**
     * 针对消费者配置
     * 1）设置交换机类型
     * 2）将队列绑定到交换机
     *
     * FanoutExchange：将消息分发到所有的绑定队列，无routingkey的概念
     * HeadersExchange：通过添加属性key-value匹配
     * DirectExchange：按照routingkey分发到指定队列
     * TopicExchange：多关键字匹配
     */
@Bean
public DirectExchange defaultExchange() {
    return new DirectExchange(EXCHANGE_A);
}

/**
     * 获取队列A
     */
@Bean
public Queue queueA() {
    //队列持久
    return new Queue(QUEUE_A, true);
}

/**
     * 获取队列B
     */
@Bean
public Queue queueB() {
    //队列持久
    return new Queue(QUEUE_B, true);
}

/**一个交换机可以绑定多个消息队列，即消息通过一个交换机，可以分发到不同的队列当中去*/
@Bean
public Binding binding() {
    return BindingBuilder.bind(queueA()).to(defaultExchange()).with(RabbitConfig.ROUTINGKEY_A);
}

@Bean
public Binding bindingB() {
    return BindingBuilder.bind(queueB()).to(defaultExchange()).with(RabbitConfig.ROUTINGKEY_B);
}
```

#### 6.编写消息的消费者，这一步是最复杂的，因为可以编写除很多不同的需求出来，写法也有很多不同。

- 比如一个生产者，一个消费者。

```java
@Component
@RabbitListener(queues = RabbitConfig.QUEUE_A)
public class MsgReceiver {

    private final Logger logger = LoggerFactory.getLogger(MsgReceiver.class);

    @RabbitHandler
    public void process(String content) {
        logger.info("接收处理队列A当中的消息：" + content);
    }
}
```

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1goas0q6nfaj30co04cq3f.jpg" style="zoom:80%">

- 比如一个生产者，多个消费者，可以写多个消费者，并且他们的分发是负载均衡的。

```java
@Component
@RabbitListener(queues = RabbitConfig.QUEUE_A)
public class MsgReceiverConsumerTwo {

    private final Logger logger = LoggerFactory.getLogger(MsgReceiverConsumerTwo.class);

    @RabbitHandler
    public void process(String content) {
        logger.info("处理器two接收处理队列A当中的消息：" + content);
    }
}
```

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1goas0q6nfaj30co04cq3f.jpg" style="zoom:80%">

- 另外一种消息处理机制的写法如下，在RabbitMQConfig类里面增加bean：

```java
@Bean
public SimpleMessageListenerContainer messageContainer() {
    //加载处理消息A的队列
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory());
    //设置接收多个队列里面的消息，这里设置接收队列A
    //假如想一个消费者处理多个队列里面的信息可以如下设置：
    //container.setQueues(queueA(),queueB(),queueC());
    container.setQueues(queueA());
    container.setExposeListenerChannel(true);
    //设置最大的并发的消费者数量
    container.setMaxConcurrentConsumers(10);
    //最小的并发消费者的数量
    container.setConcurrentConsumers(1);
    //设置确认模式手工确认
    container.setAcknowledgeMode(AcknowledgeMode.MANUAL);
    container.setMessageListener((ChannelAwareMessageListener) (message, channel) -> {
        // 通过basic.qos方法设置prefetch_count=1，这样RabbitMQ就会使得每个Consumer在同一个时间点最多处理一个Message，
        // 换句话说,在接收到该Consumer的ack前,它不会将新的Message分发给它
        channel.basicQos(1);
        byte[] body = message.getBody();
        logger.info("接收处理队列A当中的消息:" + new String(body));
        // 为了保证永远不会丢失消息，RabbitMQ支持消息应答机制。
        // 当消费者接收到消息并完成任务后会往RabbitMQ服务器发送一条确认的命令，然后RabbitMQ才会将消息删除。
        channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
    });
    return container;
}
```

下面是当一个消费者，处理多个队列里面的信息打印的log：

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1goaslm6jw4j30e004taar.jpg" style="zoom:80%">

#### 7. Fanout Exchange

Fanout就是我们熟悉的广播模式，给Fanout交换机发送消息，绑定了这个交换机的所有队列都收到这个消息。

```java
//配置fanout_exchange
@Bean
FanoutExchange fanoutExchange() {
    return new FanoutExchange(RabbitConfig.FANOUT_EXCHANGE);
}

//把所有的队列都绑定到这个交换机上去
@Bean
Binding bindingExchangeA(Queue queueA,FanoutExchange fanoutExchange) {
    return BindingBuilder.bind(queueA).to(fanoutExchange);
}
@Bean
Binding bindingExchangeB(Queue queueB, FanoutExchange fanoutExchange) {
    return BindingBuilder.bind(queueB).to(fanoutExchange);
}
@Bean
Binding bindingExchangeC(Queue queueC, FanoutExchange fanoutExchange) {
    return BindingBuilder.bind(queueC).to(fanoutExchange);
}
```

消息发送，这里不设置routing_key，因为设置了也无效，发送端的routing_key写任何字符都会被忽略。

```java
public void sendAll(String content) {
    rabbitTemplate.convertAndSend("fanoutExchange","", content);
}
```

消息处理的结果如下所示：

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1goat1okixkj30fp0c6jtp.jpg" style="zoom:80%">