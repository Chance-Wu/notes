前期准备：
* jdk1.8+
* maven
* zookeeper（作为dubbo中服务的注册中心）选择稳定版本3.4.14/3.6.1

1、新建maven父项目
--
在父项目中不需要编写代码，所以删除父项目中src文件夹。

2、新建api模块
--
该模块用来统一组织项目中的pai，将由后面的provider模块来提供具体的逻辑实现。

因为api模块中只需要提供接口所以只需要新建一个普通的maven项目即可，该模块不需要新建为Springboot项目

在api模块中，新建包com.chance.dubbo.api并声明一个接口文件。

```java
public interface DemoService {

    String sayHello(String name);
}
```

3、新建provider模块
--
> 在该模块中我们要为刚刚在api模块中声明的service提供具体实现，并通过dubbo的@Service注解将实现的服务发布到dubbo中。

provider模块是dubbo中用来提供服务的角色，在此新建一个springboot项目。

> 添加dubbo起步依赖，同时把之前建好的api模块通过依赖方式引入进来：
```xml
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>2.7.8</version>
</dependency>

<!--引入新建的api模块-->
<dependency>
    <groupId>com.chance.dubbo</groupId>
    <artifactId>api</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>

<!--引入zookeeper-->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.2.0</version>
</dependency>

<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.2.0</version>
</dependency>
```

> 在provider配置文件中，进行一些简单的配置
```yaml
spring:
  application:
    name: provider
server:
  port: 9090

#指定当前应用/服务名字，相同的服务应同名
dubbo:
  application:
    name: provider
  protocol:
    name: dubbo
    port: 20880
  registry:
    address: zookeeper://127.0.0.1:2181 #注册地址
  provider:
    timeout: 1000

#自定义属性，用来规定dubbo服务的版本号
demo:
  service:
    version: 1.0.0
```

> 在provider的启动文件中增加一个`@EnableDubbo注解`以开启Springboot对dubbo的支持。
```java
@EnableDubbo
@SpringBootApplication
public class ProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }

}
```

> 接下来，新建包及实现类来实现api模块中声明的接口方法，在service实现类上加上`@DubboService注解`，并指定版本
```java
@DubboService(version = "${demo.service.version}")
public class DemoServiceImpl implements DemoService {

    @Override
    public String sayHello(String name) {
        return "Hello " + name + ", I'am provider!";
    }
}
```

4、新建consumer模块
--
> 和provider模块一样，引入dubbo起步依赖和api模块的依赖：
```xml
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
    <version>2.7.8</version>
</dependency>

<dependency>
    <groupId>com.chance.dubbo</groupId>
    <artifactId>api</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>

<!--引入zookeeper-->
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>4.2.0</version>
</dependency>

<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>4.2.0</version>
</dependency>
```

> 配置文件
```yaml
spring:
  application:
    name: consumer
server:
  port: 9999

#指定当前应用/服务名字，相同的服务应同名
dubbo:
  application:
    name: consumer
  registry:
    address: zookeeper://127.0.0.1:2181

#自定义属性，用来规定dubbo服务的版本号
demo:
  service:
    version: 1.0.0
```

> 同样在consumer的启动类前增加@EnableDubbo注解
```java
@EnableDubbo
@SpringBootApplication
public class ConsumerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

}
```

> 服务提供者provider对内提供服务，服务消费者consumer消费provider提供的服务，同时consumer对外提供服务，所以我们在consumer中提供controller来增加外部访问的入口。
```java
@RestController
@RequestMapping("/demo")
public class DemoController {
    // 从dubbo引入service
    @DubboReference(version = "${demo.service.version}")
    private DemoService demoService;

    /**
    * 对外访问接口
    * @param name
    * @return 
    */
    @GetMapping("/say/{name}")
    public String sayHello(@PathVariable("name") String name) {
        return demoService.sayHello(name);
    }
}
```

