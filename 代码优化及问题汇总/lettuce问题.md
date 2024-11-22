### 一、lettuce间歇性发生RedisCommandTimeoutException

#### 1.1 问题描述

spring boot整合redis

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

- springboot2.0后默认使用lettuce作为操作redis的客户端
- lettuce版本：5.0.5
- bug：导致netty对外内存溢出，netty如果没有指定对外内存，默认使用-Xmx

spring在底层对lettuce和jedis进行了再封装redisTemplate，可以通过排除Lettuce添加Jedis依赖来完成RedisTemplate的默认封装实现。

```java
@Import({ LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class })
public class RedisAutoConfiguration {
```

#### 1.2 原因

那么为何 lettuce 的 Connection 为什么会断呢？而 ConnectionWatchdog为什么没有立即重连呢？又怎么解决这些问题呢？

其实这个问题并不是很重要，因为Socket连接断已经是事实，而且**在分布式环境中，网络分区是必然的**。在网络环境，Redis 服务器主动断掉连接是很正常的，lettuce 的作者也提及 lettuce 一天发生一两次重连是很正常的。

以下情况会导致redis连接断掉：

- Linux内核的**keepalive功能**可能会一直收不到客户端的回应；
- 收到与该连接相关的**ICMP错误信息**；
- 其他网络链路问题等等；

>`tcp dump` 进行抓包。

#### 1.3 lettuce为何没有立即重连

根据ConnectionWatchdog重连的机制（收到netty的**ChannelInactived事件**后启动重连的线程不断进行连接）可以确定，连接是由 Redis 服务端断开的，因为如果是客户端主动断开连接，那么一定能收到ChannelInactived，因此，之所以lettuce要等 15 分钟后才重连，是因为没收到ChanelInactived事件。

那么为什么客户端没有到ChannelInactived事件呢？很多情况都会，例如：

- 客户端没收到服务端 FIN 包；
- 网络链路断了，例如拔网线，断电等等；

日志显示发生RedisCommandTimeoutException后，15 分钟后收到ChannelInactived事件。那么，为什么会大约是 15 分钟而不是别的时间呢？

与 Linux 底层Socket的实现有关--这就是超时重传机制。也就是`/proc/sys/net/ipv4/tcp_retries2`参数。

根据重传机制，发生RedisCommandTimeoutException的命令会重传 tcp_retries2这么多次，刚刚好是 15 分钟左右。

#### 1.4 解决方案

1. 设置 Linux 的 TCP_RETRIES2 参数（全局参数）

2. netty提供了一个参数设置：`TCP_USER_TIMEOUT`，针对单独设置某个应用程序的超时重传的设置。在Spring Boot的auto-configuration中，ClientResources的初始化是默认的ClientResources，因此，可以自定义一个ClientResources。

   ```java
   @Bean
   public ClientResources clientResources(){
     return ClientResources clientResources = ClientResources.builder()
       .nettyCustomizer(new NettyCustomizer() {
         @Override
         public void afterBootstrapInitialized(Bootstrap bootstrap) {
           bootstrap.option(EpollChannelOption.TCP_USER_TIMEOUT, 10);
         }
       })
       .build();
   }
   ```

3. 定制lettuce：增加心跳机制。lettuce提供了`NettyCustomizer`进行扩展，netty提供了心跳机制--`IdleStateHandler`，结合这两者，就很容易在初始化netty时增加心跳机制：

   ```java
   @Bean
   public ClientResources clientResources(){
     NettyCustomizer nettyCustomizer = new NettyCustomizer() {
       @Override
       public void afterChannelInitialized(Channel channel) {
         channel.pipeline().addLast(
           new IdleStateHandler(readerIdleTimeSeconds, writerIdleTimeSeconds, allIdleTimeSeconds));
         channel.pipeline().addLast(new ChannelDuplexHandler() {
           @Override
           public void userEventTriggered(ChannelHandlerContext ctx, Object evt) throws Exception {
             if (evt instanceof IdleStateEvent) {
               ctx.disconnect();
             }
           }
         });
       }
   
       @Override
       public void afterBootstrapInitialized(Bootstrap bootstrap){
       }
     };
     return ClientResources.builder().nettyCustomizer(nettyCustomizer).build();
   }
   ```

   这里由客户端自己做心跳检测，**一旦发现Channel死了，主动关闭ctx.close()**，那么ChannelInactived事件一定会被触发了。但是这个方案有个缺点，增加了客户端的压力。缺点：增加了客户端压力。