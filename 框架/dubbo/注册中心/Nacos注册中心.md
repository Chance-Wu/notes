Nacos 是 Dubbo 生态系统中重要的注册中心实现，其中 `dubbo-registry-nacos` 则是 Dubbo 融合 Nacos 注册中心的实现。

#### 1. 预备工作

当您将 `dubbo-registry-nacos` 整合到您的 Dubbo 工程之前，请==确保后台已经启动 Nacos 服务==。可先行参考 [Nacos 快速入门](https://nacos.io/en-us/docs/quick-start.html)。建议使用 Nacos `1.0.0` 及以上的版本。

#### 2. 快速上手

Dubbo 融合 Nacos 成为注册中心大致步骤可分为“==增加 Maven 依赖==”以及“==配置注册中心==“。

##### 2.1 增加Maven依赖

将 `dubbo-registry-nacos` 的 Maven 依赖添加到您的项目 `pom.xml` 文件中，并且强烈地推荐您使用 Dubbo `2.6.5`：

```xml
<dependencies>

    ...

    <!-- Dubbo Nacos registry dependency -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>dubbo-registry-nacos</artifactId>
        <version>0.0.2</version>
    </dependency>   

    <!-- Keep latest Nacos client version -->
    <dependency>
        <groupId>com.alibaba.nacos</groupId>
        <artifactId>nacos-client</artifactId>
        <version>[0.6.1,)</version>
    </dependency>

    <!-- Dubbo dependency -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>dubbo</artifactId>
        <version>2.6.5</version>
    </dependency>

    <!-- Alibaba Spring Context extension -->
    <dependency>
        <groupId>com.alibaba.spring</groupId>
        <artifactId>spring-context-support</artifactId>
        <version>1.0.2</version>
    </dependency>

    ...

</dependencies>
```

当项目中添加 `dubbo-registry-nacos` 后，无需显式地编程实现服务发现和注册逻辑，实际实现由该三方包提供，接下来配置 Naocs 注册中心。

##### 2.2 配置注册中心

假设Dubbo应用使用Spring Framework装配，将由两种配置方法：==Dubbo Spring外部化配置==以及Spring XML配置文件。

>Dubbo Spring外部化配置
>
>Dubbo Spring 外部化配置是由 Dubbo `2.5.8` 引入的新特性，可==通过 Spring `Environment` 属性自动地生成并绑定 Dubbo 配置 Bean==，实现配置简化，并且降低微服务开发门槛。
>
>假设 Nacos Server 同样运行在服务器 `10.20.153.10` 上，并使用默认 Nacos 服务端口 `8848`，只需将 `dubbo.registry.address` 属性调整如下：
>
>```fallback
>## 其他属性保持不变
>
>## Nacos registry address
>dubbo.registry.address = nacos://10.20.153.10:8848
>...
>```
>
>重启 Dubbo 应用，Dubbo 的服务提供和消费信息在 Nacos 控制台中可以显示：
>
><img src="https://dubbo.apache.org/imgs/blog/dubbo-registry-nacos-1.png" style="zoom:100%">
>
>如图所示，服务名前缀为 `providers:` 的信息为服务提供者的元信息，`consumers:` 则代表服务消费者的元信息。点击“**详情**”可查看服务状态详情：
>
><img src="https://dubbo.apache.org/imgs/blog/dubbo-registry-nacos-2.png" style="zoom:100%">

