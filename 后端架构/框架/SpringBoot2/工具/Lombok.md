#### 1. 实现原理

---

Lombok 的实现原理，基于 [JSR269(Pluggable Annotation Processing API)](https://jcp.org/en/jsr/detail?id=269) 规范，自定义编译器注解处理器，用于在 Javac 编译阶段时，扫描使用到 Lombok 定义的注解的类，进行自定义的代码生成。

想要进一步深入了解可以阅读如下文章：

- [《注解处理器是干嘛的》](http://www.iocoder.cn/Fight/What-does-the-annotation-handler-do/?self)
- [《JSR269 插件化注解API》](https://blog.whatakitty.com/JSR269插件化注解API.html)

#### 2. 安装Lombok

---

IDEA安装插件，项目中引入依赖：

```xml
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <optional>true</optional>
</dependency>
```

#### 3. Lombok注解一览

---

[`@Getter`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/Getter.java) 注解，添加在**类**或**属性**上，生成对应的 get 方法。

[`@Setter`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/Setting.java) 注解，添加在**类**或**属性**上，生成对应的 set 方法。

[`@ToString`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/ToString.java) 注解，添加在**类**上，生成 toString 方法。

[`@EqualsAndHashCode`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/EqualsAndHashCode.java) 注解，添加在**类**上，生成 equals 和 hashCode 方法。

`@AllArgsConstructor`、`@RequiredArgsConstructor`、`@NoArgsConstructor` 注解，添加在**类**上，为类自动生成对应参数的构造方法。

[`@Data`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/Data.java) 注解，添加在**类**上，是 5 个 Lombok 注解的组合。

- 为所有属性，添加 `@Getter`、`@ToString`、`@EqualsAndHashCode` 注解的效果
- 为非 `final` 修饰的属性，添加 `@Setter` 注解的效果
- 为 `final` 修改的属性，添加 `@RequiredArgsConstructor` 注解的效果

`@Value` 注解，添加在**类**上，和 `@Data` 注解类似，区别在于它会把所有属性默认定义为 `private final` 修饰，所以不会生成 set 方法。

[`@CommonsLog`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/extern/apachecommons/CommonsLog.java)、[`@Flogger`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/extern/flogger/Flogger.java)、[`@Log`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/extern/java/Log.java)、[`@JBossLog`](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/extern/jbosslog/JBossLog.java)、[@Log4j](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/extern/log4j/Log4j.java)、[@Log4j2](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/extern/log4j/Log4j2.java)、[@Slf4j](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/Slf4j.java)、[@Slf4jX](https://github.com/rzwitserloot/lombok/blob/master/src/core/lombok/Slf4jX.java) 注解，添加在**类**上，自动为类添加对应的日志支持。

`@NonNull` 注解，添加在**方法参数**、**类属性**上，用于自动生成 `null` 参数检查。若确实是 `null` 时，抛出 NullPointerException 异常。

`@Cleanup` 注解，添加在方法中的**局部变量**上，在作用域结束时会自动调用 `#close()` 方法，来释放资源。例如说，使用在 Java IO 流操作的时候。

`@Builder` 注解，添加在**类**上，给该类加个构造者模式 Builder 内部类。

`@Synchronized` 注解，添加在**方法**上，添加同步锁。

`@SneakyThrows` 注解，添加在**方法**上，给该方法添加 `try catch` 代码块。

`@Accessors` 注解，添加在**方法**或**属性**上，并设置 `chain = true`，实现链式编程。

下面，我们在 Spring Boot 示例项目中，使用下 `@Data` 和 `@Slf4j`、`@NonNull` 这三个 Lombok 常用注解。