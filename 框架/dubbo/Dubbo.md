### 一、互联网架构演变趋势

- **单一应用架构**：单体应用，所有功能、模块都耦合在一个应用中。
  - 代表技术SpringMVC、Spring、Mybatis等；
  - 打包成一个独立的单元（war或jar），会以一个进程的方式来运行；
  - 测试成本高、可伸缩性差、可靠性差、迭代困难、跨语言程度差、团队协作难。
- **RPC架构**：远程过程调用。通过网络从远程计算机程序上请求服务，不需要了解底层网络技术的协议。
  - 代表技术：Thrift、Hessian；
  - 应用直接调用服务，服务之间是隔离的；
  - 服务过多时，管理成本高。服务治理，服务注册、发现，服务容错，服务跟踪，服务网关，IP暴露等都是此架构无法避免的问题；
- **SOA架构**：面向服务架构。
  - ==ESB（Enterprise Service Bus）：企业服务总线==，服务中介。主要是提供了一个服务与服务之间的交互；
  - ESB包含的功能如：负载均衡，流量控制，加密处理，服务的监控，异常处理，监控告急等等；
  - 代表技术：Mule、WSO2。
- **微服务架构**：微服务是一种架构风格。一个大型的复杂软件应用，由一个或多个微服务组成。系统中的微服务可被独立部署，各个微服务之间是松耦合的。每个微服务仅关注于完成一件任务并很好的完成该任务。微服务就是一个轻量级的服务治理方案。
  - 对比SOA架构，使用注册中心（zk，eureka，nacos）代替ESB服务总线。注册中心相比服务中线更加轻量级；
  - 代表技术：SpringCloud、Dubbo等；
  - 每个服务之间松耦合，服务内部是高内聚，外部是低耦合；
  - 测试容易、可伸缩性强、可靠性强、跨语言程度更好、团队协作容易、系统迭代容易、运维成本高、分布式系统的复杂性、分布式事务。



### 二、RPC基于RMI的简单实现



#### 2.1 RPC协议（Remote Procedure Call Protocol）

---

RPC协议嘉定某些传输协议的存在，如TCP或UDP，为通信程序之间携带信息数据。在OSI网络通信模型中，RPC跨越了传输层和应用层。RPC是的开发包括网络分布式多程序在内的应用程序更加容易。

RPC采用客户机/服务器模式。请求程序就是一个客户机，而服务提供提供程序就是一个服务器。首先，==客户端调用进程发送一个有进程参数的调用信息到服务进程==，然后等待应答信息。==在服务器端，进程保持睡眠状态直到调用信息到达为止==。当一个调用信息到达，服务器获得进程参数，计算结果，发送答复信息，然后等待下一个调用信息，最后，客户端调用进程接收答复信息，获得进程结果，然后调用执行继续进行。



#### 2.2 RPC框架

---

在单机时代一台电脑运行多个进程，进程之间无法通讯，显然这会浪费很多资源，因此后来出现IPC（Inter process communication-单机中运行的进程之间的相互通信），比如在一台计算机中的A进程写了一个吃饭的方法，那在以前如果在B进程中也要有一个吃饭的方法，必须在B进程中进行创建，但有了RPC后B只需要调用A进程的程序即可完成，再到后来网络时代的出现，大家的电脑都连起来，这时可以调用其他电脑上的进程。严格意义上来讲：Unix的生态系统中RPC可以在同一台电脑上不同进程进行，也可以在不同电脑上进行；而Windows里面同一台电脑不同进程间的通信还可以采用LPC（本地访问）。综上：RPC或LPC是上层建筑，IPC是底层基础。

RPC框架比如：JAVA RMI、Thrift、Dubbo、grpc等。



#### 2.3 RPC与HTTP、TCP/UDP、Socket的区别

---

- TCP/UDP：都是传输协议，主要区别是tcp协议连接需要3次握手，断开需要四次挥手，是通过流来传输的，就是确定连接后，一直发送消息，传完后断开。udp不需要进行连接，直接把信息封装成多个报文，直接发送。所以udp的速度更快写，但是不保证数据的完整性。
- HTTP：超文本传输协议是一种应用层协议，建立在TCP协议之上。
- Socket：在应用程序层面上对TCP/IP协议的封装和应用。==其实是一个调用接口，方便程序员使用TCP/IP协议栈而已==。程序员通过socket来使用tcp/ip协议，但是socket并不一定要使用tcp/ip协议，Socket编程接口在设计的时候，就是希望也能适应其他网络协议。
- RPC：通过网络从远程计算机程序上请求服务，不需要了解底层网络技术的协议，所以==RPC的实现可以通过不同的协议去实现比如可以使用HTTP、RMI等==。



#### 2.4 RPC的运行流程

---

>1. 要解决==通讯==的问题，主要是通过在客户端和服务器之间建立TCP连接，远程过程调用的所有交换的数据都在这个连接里传输。连接可以是按需连接，调用结束后就断掉，也可以是长连接，多个远程调用共享同一个连接。

>2. 要解决==寻址==的问题，A服务器上的应用怎么告诉底层的RPC框架，如何连接到B服务器（如主机或IP地址）以及特定的端口，方法名称是什么，这样才能完成调用。比如基于Web服务协议栈的RPC，就要提供一个endpoint URI，或者从UDD服务商查找。如果是RMI调用的话，还需要一个RMI Registry来注册服务的地址。

>3. 当A服务器上的应用发起远程过程调用时，==方法的参数需要通过底层的网络协议如TCP传递到B服务器==，由于网络协议是基于二进制的，内存中的参数的值要序列化成二进制的形式，也就是序列化（Serialize）或编组（marshal），通过寻址和传输将序列化的二进制发送给B服务器。

>4. B服务器接收到请求后，需要==对参数进行反序列化==（序列化的逆操作），恢复为内存中的表达方式，然后找到对应的方法（寻址的一部分）进行本地调用，然后得到返回值。

>5. 返回值还要发送回服务器A上的应用，也要经过序列化的方式发送，服务器A接收到后，再反序列化，恢复内存中的表达方式，交给服务器A上的应用。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gtim09i4ahj60dw08qjrm02.jpg)



#### 2.5 为什么需要RPC

---

RPC框架的复杂度肯定是高于简单地HTTP接口的。但HTTP接口由于受限于HTTP协议，需要带HTTP请求头，导致传输起来效率或者安全性不如RPC。

>HTTP接口在接口不多、系统与系统交互较少的情况下，解决信息初期常使用的一种通信手段；简单、直接、开发方便。利用现成的http协议进行传输。

>如果是一个大型的网站，内部子系统较多、接口非常多的情况下，RPC架构的好处就显现出来了。
>
>- 首先是==长链接，不必每次通信都要像http一样去3次握手什么的，减少了网络开销==（这个问题在http2.0被解决）；
>- 其次RPC框架一般都有注册中心，有丰富的监控管理；发布、下线接口、动态扩展等，对调用方来说是无感知、统一化的操作；
>- 安全性；
>- 流行的服务化架构、服务化治理。

RPC是一种概念，http也是RPC实现的一种方式，用http交互其实就已经属于RPC了。

>RPC和核心不在于使用什么协议。RPC的目的是让你在本地调用远程的方法，对我们来说这个调用是透明的，并不知道这个调用的方法部署在哪里。通过RPC能解耦服务。
>
>RPC的原理主要用到了动态代理模式。



#### 2.6 RPC基于RMI的简单实现

---

服务暴露：

```java
@Configuration
public class RMIConfig {

  @Autowired
  private UserService userService;

  @Bean
  public RmiServiceExporter rmiServiceExporter() {
    RmiServiceExporter rmiServiceExporter = new RmiServiceExporter();
    rmiServiceExporter.setService(userService);
    rmiServiceExporter.setServiceInterface(UserService.class);
    rmiServiceExporter.setRegistryPort(2002);
    rmiServiceExporter.setServiceName("userService");
    return rmiServiceExporter;
  }
}
```

```java
@Configuration
public class RMIConfig {

  @Bean(name = "userService")
  public RmiProxyFactoryBean rmiProxyFactoryBean() {
    RmiProxyFactoryBean rmiProxyFactoryBean = new RmiProxyFactoryBean();
    rmiProxyFactoryBean.setServiceUrl("rmi://127.0.0.1:2002/userService");
    rmiProxyFactoryBean.setServiceInterface(UserService.class);
    return rmiProxyFactoryBean;

  }
}
```



### 三、Dubbo入门

---

Dubbo是基于定义服务的概念，指定可以通过参数和返回类型远程调用的方法。在服务器端，服务器实现这个接口，并运行一个Dubbo服务器来处理客户端调用。在客户端，客户机有一个存根，它提供与服务器相同的方法。

Dubbo提供三个核心功能：==基于接口的远程调用==、==容错和负载均衡==，以及==服务的自动注册与发现==。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gtjht495qwj60tg0mw3yw02.jpg)



#### 3.1 Multicast注册中心

---

> 不需要启动任何中心节点，只要广播地址一样，就可以互相发现。
>
> ![](https://tva1.sinaimg.cn/large/008i3skNgy1gtjpq4jkfjj60go0503yi02.jpg)
>
> 1. 提供方启动时广播自己的地址；
> 2. 消费方启动时广播订阅请求；
> 3. 提供方收到订阅请求时，单播自己的地址给订阅者，如果设置了unicast=false，则广播给订阅者；
> 4. 消费方收到提供方地址时，连接该地址进行RPC调用。
>
> 组播受网络结构限制，只适合小规模应用或开发阶段使用。组播地址段: `224.0.0.0` - `239.255.255.255`
>
> ```yml
> server:
>   port: 8001
> 
> dubbo:
>   application:
>     name: provider
>   # 注册中心地址
>   registry:
>     address: multicast://224.5.6.7:1234
>     timeout: 6000
>   protocol:
>     name: dubbo
>     port: 20880
>   scan:
>     # 扫描包的位置（需要暴露的服务）
>     base-packages: com.xxxx.provider.service
> ```
>
> ```yml
> server:
>   port: 8002
> 
> dubbo:
>   application:
>     name: consumer
>   registry:
>     address: multicast://224.5.6.7:1234
> ```
>
> 注意：同一台机器使用Multicast广播会出现消费者找不到提供者的错误
>
> 解决方法：
>
> - 删除`/{user_home}/.dubbo`目录
>
> - ```java
>   @DubboReference(version = "1.0", parameters = {"unicast", "false"})
>   private UserService userService;
>   ```



#### 3.2 zk注册中心

---

>zk是一个树型的目录服务，支持变更推送，适合作为 Dubbo 服务的注册中心，工业强度较高，可用于生产环境，并推荐使用。
>
>Dubbo支持curator的zk客户端实现。
>
>zk单机版安装：
>
>```shell
># 创建安装目录
>mkdir -p /usr/local/zookeeper
>
># 将文件解压至该目录
>tar -zxvf apache-zookeeper-3.6.1-bin.tar.gz -C /usr/local/zookeeper/
>
># 创建数据目录data、日志目录log
>mkdir -p /usr/local/zookeeper/apache-zookeeper-3.6.1-bin/data
>mkdir -p /usr/local/zookeeper/apache-zookeeper-3.6.1-bin/logs
>
># 修改配置文件（复制zoo_sample.cfg一份修改名称为zoo.cfg）
>dataDir=/usr/local/zookeeper/apache-zookeeper-3.6.1-bin/data
>logDir=/usr/local/zookeeper/apache-zookeeper-3.6.1-bin/logs
>```
>
>启动zk：
>
>```shell
>./zkServer.sh start
>```



### 四、Dubbo+Zookeeper注册中心示例

> **zk集群配置**
>
> - data目录下创建myid文件；
>
> - zoo.cfg配置修改：
>
>   ```properties
>   # zk-1
>   dataDir=/usr/local/zkCluster/zookeeper-1/data
>   # log目录, 同样可以是任意目录. 如果没有设置该参数, 将使用和#dataDir相同的设置.
>   dataLogDir=/usr/local/zkCluster/zookeeper-1/logs
>         
>   clientPort=2181
>         
>   # 集群的zookeeper应用，格式：serve.n=ip:集群通信端口:集群选举端口
>   server.1=127.0.0.1:2887:3887
>   server.2=127.0.0.1:2888:3888
>   server.3=127.0.0.1:2889:3889
>   ```
>
>   ```properties
>   # zk-2
>   dataDir=/usr/local/zkCluster/zookeeper-2/data
>   dataLogDir=/usr/local/zkCluster/zookeeper-2/logs
>         
>   clientPort=2182
>         
>   # 集群的zookeeper应用，格式：serve.n=ip:集群通信端口:集群选举端口
>   server.1=127.0.0.1:2887:3887
>   server.2=127.0.0.1:2888:3888
>   server.3=127.0.0.1:2889:3889
>   ```
>
>   ```properties
>   # zk-3
>   dataDir=/usr/local/zkCluster/zookeeper-3/data
>   dataLogDir=/usr/local/zkCluster/zookeeper-3/logs
>         
>   clientPort=2183
>         
>   # 集群的zookeeper应用，格式：serve.n=ip:集群通信端口:集群选举端口
>   server.1=127.0.0.1:2887:3887
>   server.2=127.0.0.1:2888:3888
>   server.3=127.0.0.1:2889:3889



```xml
<!--依赖版本管理-->
<dependencyManagement>
  <dependencies>
    <!-- dubbo 依赖 -->
    <dependency>
      <groupId>org.apache.dubbo</groupId>
      <artifactId>dubbo-spring-boot-starter</artifactId>
      <version>${dubbo.version}</version>
      <exclusions>
        <exclusion>
          <groupId>org.slf4j</groupId>
          <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
    <!-- zk 注册中心客户端引入 curator客户端 -->
    <dependency>
      <groupId>org.apache.curator</groupId>
      <artifactId>curator-framework</artifactId>
      <version>${curator.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.curator</groupId>
      <artifactId>curator-recipes</artifactId>
      <version>${curator.version}</version>
    </dependency>
  </dependencies>
</dependencyManagement>
```

provider dubbo配置：

```yaml
dubbo:
  application:
    name: provider
  # 注册中心地址
  registry:
    address: zookeeper://127.0.0.1:2181?backup=127.0.0.1:2182,127.0.0.1:2183
    timeout: 6000
  # 元数据地址
  metadata-report:
    address: zookeeper://127.0.0.1:2181?backup=127.0.0.1:2182,127.0.0.1:2183
  protocol:
    name: dubbo
    port: 20880
  scan:
    # 扫描包的位置（需要暴露的服务）
    base-packages: com.xxxx.provider.service
```

consumer dubbo配置：

```yaml
dubbo:
  application:
    name: consumer
  registry:
    address: zookeeper://127.0.0.1:2181?backup=127.0.0.1:2182,127.0.0.1:2183
```

>如下会拉取注册中心的服务缓存在本地，即使注册中心宕机，之前的服务也能用，但是新的服务无法发现。
>
>```java
>@DubboReference(version = "1.0")
>private UserService userService;
>```



### 五、Dubbo-admin管理中心

maven方式部署

克隆：

```shell
git clone https://hub.fastgit.org/apache/dubbo-admin.git
```

maven打包：

```shell
mvn clean package -Dmaven.test.skip=true
```

运行：

```shell
java -jar dubbo-admin-0.3.0.jar --server.port=8081
```

查看Dubbo Admin管理台界面：http://localhost:8081/（root/root）

服务查询：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gtm1wb0e2lj61ia0te0un02.jpg)

