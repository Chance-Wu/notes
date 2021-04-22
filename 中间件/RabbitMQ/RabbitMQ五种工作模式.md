#### 1. 简单队列

一个生产者对应一个消费者。

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gob8yxttypj30bd03v744.jpg" style="zoom:100%">

```java
/**
 * 〈简述〉<br>
 * 〈连接RabbitMQ的工具类〉
 */
public class ConnectionUtil {
    public static Connection getConnection() throws Exception {
        return getConnection(new Properties());
    }

    private static Connection getConnection(Properties properties) throws Exception {
        return getConnection(properties.getHost(),
                             properties.getPort(),
                             properties.getvHost(),
                             properties.getUserName(),
                             properties.getPassWord());
    }

    public static Connection getConnection(String host, int port, String vHost, String userName, String passWord) throws Exception {
        //1、定义连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //2、设置服务器地址
        factory.setHost(host);
        //3、设置端口
        factory.setPort(port);
        //4、设置虚拟主机、用户名、密码
        factory.setVirtualHost(vHost);
        factory.setUsername(userName);
        factory.setPassword(passWord);
        //5、通过连接工厂获取连接
        Connection connection = factory.newConnection();
        return connection;
    }

    public static class Properties implements Serializable {
        String host = "192.168.1.103";
        int port = 5672;
        String vHost =  "/";
        String userName = "guest";
        String passWord = "guest";

        public Properties() {
        }

        public Properties(String host, int port, String vHost, String userName, String passWord) {
            this.host = host;
            this.port = port;
            this.vHost = vHost;
            this.userName = userName;
            this.passWord = passWord;
        }

        public String getHost() {
            return host;
        }

        public Properties setHost(String host) {
            this.host = host;
            return self();
        }

        public int getPort() {
            return port;
        }

        public Properties setPort(int port) {
            this.port = port;
            return self();
        }

        public String getvHost() {
            return vHost;
        }

        public Properties setvHost(String vHost) {
            this.vHost = vHost;
            return self();
        }

        public String getUserName() {
            return userName;
        }

        public Properties setUserName(String userName) {
            this.userName = userName;
            return self();
        }

        public String getPassWord() {
            return passWord;
        }

        public Properties setPassWord(String passWord) {
            this.passWord = passWord;
            return self();
        }

        private Properties self(){
            return this;
        }
    }
}
```

```java
/**
 * 〈简述〉<br> 
 * 〈简单队列——消息生产者〉
 */
public class Producer {
    private final static String QUEUE_NAME = QueueName.test_simple_queue.toString();

    public static void main(String[] args) throws Exception {
        sendMessage();
    }

    public static void sendMessage() throws Exception {
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection();
        //2、声明信道
        Channel channel = connection.createChannel();
        //3、声明(创建)队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //4、定义消息内容
        String message = "hello rabbitmq ";
        //5、发布消息
        channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
        System.out.println("[x] Sent'" + message + "'");
        //6、关闭通道
        channel.close();
        //7、关闭连接
        connection.close();
    }
}
```

```java
/**
 * 〈简述〉<br>
 * 〈消息消费者〉
 */
public class Customer {

    private final static String QUEUE_NAME = QueueName.test_simple_queue.toString();

    public static void main(String[] args) throws Exception {
        getMessage();

    }

    public static void getMessage() throws Exception {
        //1、获取连接
        Connection connection = ConnectionUtil.getConnection();
        //2、声明通道
        Channel channel = connection.createChannel();
        //3、声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //4、定义队列的消费者
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope,
                                       AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msgString = new String(body, "utf-8");
                System.out.println("接收的消息：" + msgString);
            }
        };
        //5、监听队列
        /*
   true:表示自动确认，只要消息从队列中获取，无论消费者获取到消息后是否成功消费，都会认为消息已经成功消费
   false:表示手动确认，消费者获取消息后，服务器会将该消息标记为不可用状态，等待消费者的反馈，
    如果消费者一直没有反馈，那么该消息将一直处于不可用状态，并且服务器会认为该消费者已经挂掉，不会再给其
    发送消息，直到该消费者反馈。
   */
        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}
```

这里消费者有==自动确认消息==和==手动确认消息==两种模式。

#### 2. work模式

一个生产者对应多个消费者，但是一条消息只能有一个消费者获得消息。

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gob91vm5huj30cb045wed.jpg" style="zoom:100%">

轮询分发：

```java
/**
 * 〈简述〉<br>
 * 〈轮询分发——生产者〉
 */
public class Send {
    private static final String QUEUE_NAME = QueueName.test_work_queue.toString();

    public static void main(String args[]) throws Exception {
        //获取连接
        Connection connection = ConnectionUtil.getConnection();
        //获取channel
        Channel channel = connection.createChannel();
        //声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        for (int i = 0; i < 50; i++) {
            String msg = "hello " + i;
            System.out.println("[mq] send:" + msg);
            channel.basicPublish("", QUEUE_NAME, null, msg.getBytes());
            Thread.sleep(i * 20);
        }
        channel.close();
        connection.close();
    }

}
```

这里创建两个消费者：

- 消费者1：每接收一条消息红颜休眠1秒

```java
/**
 * 〈简述〉<br>
 * 〈接收者〉
 */
public class Receive1 {
    private static final String QUEUE_NAME = QueueName.test_work_queue.toString();

    public static void main(String args[]) throws Exception {
        //获取连接
        Connection connection = ConnectionUtil.getConnection();
        //获取channel、
        Channel channel = connection.createChannel();
        //声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //定义一个消费这
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "utf-8");
                System.out.println("[1] Receive1 msg:" + msg);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[1] done");
                }
            }
        };
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, consumer);
    }
}
```

- 消费者2：每接收一条消息后休眠2秒

```java
/**
 * 〈简述〉<br>
 * 〈接收者〉
 */
public class Receive2 {
    private static final String QUEUE_NAME = QueueName.test_work_queue.toString();

    public static void main(String args[]) throws Exception {
        //获取连接
        Connection connection = ConnectionUtil.getConnection();
        //获取channel
        Channel channel = connection.createChannel();
        //声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //定义一个消费这
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "utf-8");
                System.out.println("[2] Receive2 msg:" + msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[2] done");
                }
            }
        };
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, consumer);
    }
}
```

==轮询分发==就是将消息队列中的消息，依次发送给所有消费者。一个消息只能被一个消费者获取。

==公平分发==：消费者关闭自动应答，开启手动回执。

```java
/**
 * 〈简述〉<br>
 * 〈接收者〉
 */
public class Receive2 {
    private static final String QUEUE_NAME = QueueName.test_work_queue.toString();

    public static void main(String args[]) throws Exception {
        //获取连接
        Connection connection = ConnectionUtil.getConnection();
        //获取channel
        Channel channel = connection.createChannel();
        //声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        channel.basicQos(1);//保证一次只分发一个消息
        //定义一个消费这
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "utf-8");
                System.out.println("[2] Receive2 msg:" + msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[2] done");
                    //手动回执
                    channel.basicAck(envelope.getDeliveryTag(),false);
                }
            }
        };
        boolean autoAck = false;//自动应答
        channel.basicConsume(QUEUE_NAME, autoAck, consumer);
    }

}
```

手动回执：消费者完成业务接口方法后可以告知消息队列处理完成，消息队列中取一条消息发送给消费者。

能者多劳：效率高的消费者消费消息多。

#### 3. 发布/订阅模式

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gobmonh2vpj30d705v3yg.jpg" style="zoom:80%">

一个消费者将消息首先发送到交换器，交换器绑定到多个队列，然后被监听该队列的消费者所接收并消费。

X表示交换器，在RabbitMQ中，交换器主要有四种类型：==direct==、==fanout==、==topic==、==headers==，这里的交换器是fanout。

##### 3.1 生产者

```java
/**
 * 〈简述〉<br>
 * 〈订阅模式——生产者〉
 */
public class Send {
    private static final String EXCHANGE_NAME = MqName.exchange_fanout.toString();

    public static void main(String args[]) throws Exception {
        //获取连接
        Connection connection = ConnectionUtil.getConnection();
        //获取channel
        Channel channel = connection.createChannel();
        //声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME,"fanout");//分发
        //发送消息
        String msg = "hello exchange";
        System.out.println("[mq] send:" + msg);
        channel.basicPublish(EXCHANGE_NAME, "", null, msg.getBytes());
        channel.close();
        connection.close();
    }
}
```

##### 3.2 消费者

注意：两个消费者绑定不同的队列，绑定相同的交换机；

消费者1：绑定队列名 = queue_fanout_email1

```java
/**
 * 〈简述〉<br>
 * 〈接收者〉
 */
public class Receive1 {
    private static final String QUEUE_NAME = MqName.queue_fanout_email.toString() + "1";
    private static final String EXCHANGE_NAME = MqName.exchange_fanout.toString();

    public static void main(String args[]) throws Exception {
        //获取连接
        Connection connection = ConnectionUtil.getConnection();
        //获取channel
        Channel channel = connection.createChannel();
        //声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //绑定到交换机
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "");
        //定义一个消费这
        Consumer consumer = new DefaultConsumer(channel) {
            /**
             * No-op implementation of {@link Consumer#handleDelivery}.
             *
             * @param consumerTag
             * @param envelope
             * @param properties
             * @param body
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "utf-8");
                System.out.println("[1] Receive1 msg:" + msg);
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[1] done");
                }
            }
        };
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, consumer);
    }

}
```

消费者2：绑定队列名=queue_fanout_email2

```java
/**
 * 〈简述〉<br>
 * 〈接收者〉
 */
public class Receive2 {
    private static final String QUEUE_NAME = MqName.queue_fanout_email.toString() + "2";
    private static final String EXCHANGE_NAME = MqName.exchange_fanout.toString();

    public static void main(String args[]) throws Exception {
        //获取连接
        Connection connection = ConnectionUtil.getConnection();
        //获取channel
        Channel channel = connection.createChannel();
        //声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //绑定到交换机
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, "");
        //定义一个消费这
        Consumer consumer = new DefaultConsumer(channel) {
            /**
             * No-op implementation of {@link Consumer#handleDelivery}.
             *
             * @param consumerTag
             * @param envelope
             * @param properties
             * @param body
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "utf-8");
                System.out.println("[2] Receive2 msg:" + msg);
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    System.out.println("[2] done");
                }
            }
        };
        boolean autoAck = false;
        channel.basicConsume(QUEUE_NAME, autoAck, consumer);
    }
}
```

以上两个消费者获得了同一条消息。即就是，一个消息从交换机同时发送给了两个队列中，监听这两个队列的消费者消费了这个消息。

==如果没有队列绑定交换机，则消息将丢失。因为交换机没有存储能力，消息只能存储在队列中==。

#### 4. 路由模式

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gobo7a1nj1j30es055aa1.jpg" style="zoom:80%">

生产者将消息发送到direct交换器，在绑定队列和交换器的时候有一个路由key，生产者发送的消息会指定一个路由key，那么消息只会发送到相应key相同的队列，接着监听该队列的消费者消费消息。

##### 4.1 生产者

```java
/**
 * 〈简述〉<br> 
 * 〈路由模式-消息发送者〉
 */
public class Send {

    public static final String EXCHANGE_NAME = MqName.exchange_routing.toString();
    public static final String ROUTING_KEY = MqName.routing_world.toString();

    public static void main(String args[]) throws Exception {
        //获取连接
        Connection connection = ConnectionUtil.getConnection();
        //获取channel
        Channel channel = connection.createChannel();
        //声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, "direct");
        String msg = "route message ->" + ROUTING_KEY;
        System.out.println("对 " + ROUTING_KEY + " 发送消息：" + msg);
        channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, null, msg.getBytes());
        //关闭连接
        channel.close();
        connection.close();
    }
}
```

##### 4.2 消费者

注意：两个消费者，绑定相同的交换机，不同的队列，不一样的路由。

消费者1：路由=routing_hello

```java
/**
 * 〈简述〉<br>
 * 〈接收消息1〉
 */
public class Receive1 {

    public static final String QUEUE_NAME = MqName.queue_routing_001.toString();
    public static final String EXCHANGE_NAME = MqName.exchange_routing.toString();
    public static final String ROUTING_KEY_hello = MqName.routing_hello.toString();

    public static void main(String args[]) throws Exception {
        //获取连接
        Connection connection = ConnectionUtil.getConnection();
        //获取channel
        Channel channel = connection.createChannel();
        //声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //设置预读取数
        channel.basicQos(1);
        //绑定交换机
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY_hello);
        Consumer consumer = new DefaultConsumer(channel) {
            /**
             * No-op implementation of {@link Consumer#handleDelivery}.
             *
             * @param consumerTag
             * @param envelope
             * @param properties
             * @param body
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "utf-8");
                System.out.println(envelope.getRoutingKey() + " [1] Receive1 msg:" + msg);
            }
        };
        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}
```

消费者2：绑定两个路由（路由1=routing_world 路由2=routing_hello）

```java
/**
 * 〈简述〉<br>
 * 〈接收消息2〉
 */
public class Receive2 {

    public static final String QUEUE_NAME = MqName.queue_routing_002.toString();
    public static final String EXCHANGE_NAME = MqName.exchange_routing.toString();
    public static final String ROUTING_KEY_world = MqName.routing_world.toString();
    public static final String ROUTING_KEY_hello = MqName.routing_hello.toString();

    public static void main(String args[]) throws Exception {
        //获取连接
        Connection connection = ConnectionUtil.getConnection();
        //获取channel
        Channel channel = connection.createChannel();
        //声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //设置预读取数
        channel.basicQos(1);
        //绑定交换机和路由器，可以绑定多个路由
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY_world);
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY_hello);
        //定义消息消费者
        Consumer consumer = new DefaultConsumer(channel) {
            /**
             * No-op implementation of {@link Consumer#handleDelivery}.
             *
             * @param consumerTag
             * @param envelope
             * @param properties
             * @param body
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "utf-8");
                System.out.println(envelope.getRoutingKey() + " [2] Receive1 msg:" + msg);
            }
        };
        //接收消息
        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}
```

**测试结果：**

生产者发送：routing_world

消费者1：没有接收到

消费者2：接收到了

生产者发送：routing_hello

消费者1：接收到了

消费者2：接收到了

路由模式，是以路由规则为导向，引导消息存入符合规则的队列中。再由队列的消费者进行消费的。

#### 5. 主题模式

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gobr6bijv4j30fe0520sq.jpg" style="zoom:80%">

上面的路由模式是根据路由key进行完整的匹配（完全相等才发送消息），这里的通配符模式通俗的来讲就是模糊匹配。

符号”`#`“表示匹配一个或多个词，符号”`*`“表示匹配一个词。

##### 5.1 生产者

```java
/**
 * 〈简述〉<br> 
 * 〈主题模式-消息发送者〉
 */
public class Send {

    public static final String EXCHANGE_NAME = MqName.exchange_topic.toString();
    public static final String ROUTING_KEY = MqName.routing_goods.toString();

    public static void main(String args[]) throws Exception {
        //获取连接
        Connection connection = ConnectionUtil.getConnection();
        //获取channel
        Channel channel = connection.createChannel();
        //声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, "topic");
        //        String routingKey = ROUTING_KEY + ".add";
        String routingKey = ROUTING_KEY + ".publish";
        //        String routingKey = ROUTING_KEY + ".update";

        String msg = "route message ->" + routingKey;
        channel.basicPublish(EXCHANGE_NAME, routingKey, null, msg.getBytes());
        System.out.println("对 " + routingKey + " 发送消息：" + msg);
        //关闭连接
        channel.close();
        connection.close();
    }
}
```

##### 5.2 消费者

注意两个消费者，路由的不同

消费者1：路由1 = routing_goods.add、路由2 = routing_goods.update

```java
/**
 * 〈简述〉<br>
 * 〈接收消息1〉
 */
public class Receive1 {

    public static final String QUEUE_NAME = MqName.queue_topic_001.toString();
    public static final String EXCHANGE_NAME = MqName.exchange_topic.toString();
    public static final String ROUTING_KEY = MqName.routing_goods.toString();

    public static void main(String args[]) throws Exception {
        //获取连接
        Connection connection = ConnectionUtil.getConnection();
        //获取channel
        Channel channel = connection.createChannel();
        //声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //设置预读取数
        channel.basicQos(1);
        //绑定交换机
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY + ".add");
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY + ".update");
        Consumer consumer = new DefaultConsumer(channel) {
            /**
             * No-op implementation of {@link Consumer#handleDelivery}.
             *
             * @param consumerTag
             * @param envelope
             * @param properties
             * @param body
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "utf-8");
                System.out.println(envelope.getRoutingKey() + " [1] Receive1 msg:" + msg);
            }
        };
        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}
```

消费者2：路由 = routing_goods.*

```java
/**
 * 〈简述〉<br>
 * 〈接收消息2〉
 */
public class Receive2 {

    public static final String QUEUE_NAME = MqName.queue_routing_002.toString();
    public static final String EXCHANGE_NAME = MqName.exchange_topic.toString();
    public static final String ROUTING_KEY = MqName.routing_goods.toString();

    public static void main(String args[]) throws Exception {
        //获取连接
        Connection connection = ConnectionUtil.getConnection();
        //获取channel
        Channel channel = connection.createChannel();
        //声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //设置预读取数
        channel.basicQos(1);
        //绑定交换机和路由器，可以绑定多个路由
        channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY + ".*");
        //定义消息消费者
        Consumer consumer = new DefaultConsumer(channel) {
            /**
             * No-op implementation of {@link Consumer#handleDelivery}.
             *
             * @param consumerTag
             * @param envelope
             * @param properties
             * @param body
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String msg = new String(body, "utf-8");
                System.out.println(envelope.getRoutingKey() + " [2] Receive1 msg:" + msg);
            }
        };
        //接收消息
        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}
```

测试结果：

消费者1只能接收到.add 和 .update的消息

消费者2可以接收到.add .publish 和 .update的消息

与路由模式相似，但是，主题模式是一种模糊的匹配方式。

#### 6. 工作模式总结

这五种工作模式，可以归为三类：

1. 生产者，消息队列，一个消费者；
2. 生产者，消息队列，多个消费者；
3. 生产者，交换机，多个消息队列，多个消费者；

#### 7. 四种交换器

1. direct 如果路由键完全匹配的话，消息才会被投放到相应的队列。
2. fanout 当发送一条消息到fanout交换器上时，它会把消息投放到所有附加在此交换器上的队列。
3. topic　设置模糊的绑定方式，“*”操作符将“.”视为分隔符，匹配单个字符；“#”操作符没有分块的概念，它将任意“.”均视为关键字的匹配部分，能够匹配多个字符。
4. header headers 交换器允许匹配 AMQP 消息的 header 而非路由键，除此之外，header 交换器和 direct 交换器完全一致，但是性能却差很多，因此基本上不会用到该交换器。

