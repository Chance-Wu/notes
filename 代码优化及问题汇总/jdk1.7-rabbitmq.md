## 一、依赖（RabbitMQ Java Client）

**推荐版本（支持 Java 1.7）：**

```
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>4.12.0</version>
</dependency>
```

> ❗ 注意：
>  RabbitMQ Java Client **5.x 以上需要 Java 8**，所以 Java 7 必须用 4.x 或更低。

------

## 二、连接 RabbitMQ 基本配置

```
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class RabbitMQConnection {

    public static Connection getConnection() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");      // RabbitMQ 地址
        factory.setPort(5672);              // 默认端口
        factory.setUsername("guest");
        factory.setPassword("guest");
        factory.setVirtualHost("/");

        return factory.newConnection();
    }
}
```

------

## 三、生产者（发送消息）

```
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;

public class Producer {

    private static final String QUEUE_NAME = "test_queue";

    public static void main(String[] args) throws Exception {
        Connection connection = RabbitMQConnection.getConnection();
        Channel channel = connection.createChannel();

        // 声明队列（不存在才会创建）
        channel.queueDeclare(
                QUEUE_NAME,
                false,  // durable
                false,  // exclusive
                false,  // autoDelete
                null
        );

        String message = "Hello RabbitMQ (Java 1.7)";
        channel.basicPublish(
                "",           // 默认交换机
                QUEUE_NAME,   // routingKey = 队列名
                null,
                message.getBytes("UTF-8")
        );

        System.out.println("发送消息：" + message);

        channel.close();
        connection.close();
    }
}
```

注意：

1. **zhu字符集要显式指定 UTF-8**
1. **一定要用 amqp-client 4.x**

生产环境建议：

- durable = true
- 手动 ACK
- 连接池 / 单例 Connection