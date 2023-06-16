官方参考文档 [Spring JavaConfig](https://docs.spring.io/spring-javaconfig/docs/1.0.0.M4/reference/html/) 

在 Spring3.0 开始，Spring 提供了 JavaConfig 的方式，允许我们使用 Java 代码的方式，进行 Spring Bean 的创建。示例代码如下：

```java
@Configuration
public class DemoConfiguration {
  
  @Bean
  public void object() {
    return new Object();
  }
}
```

- 通过在类上添加`@Configuration`注解，声明这是一个Spring配置类。
- 通过在方法上添加`@Bean`注解，声明该方法创建一个Spring Bean。

