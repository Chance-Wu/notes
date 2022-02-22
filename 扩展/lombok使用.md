lombok能够自动嵌入到IDE编辑器和编译工具中。



#### 1. 安装插件和配置依赖

---

在IDEA中安装Lombok插件，这样在使用Lombok的时候就不会编译报错。

maven依赖配置，在使用Maven打包的时候能自动生成需要的代码。

```xml
<dependency>
  <groupId>org.projectlombok</groupId>
  <artifactId>lombok</artifactId>
  <version>1.16.20</version>
  <scope>provided</scope>
</dependency>
```



#### 2. 使用注解简化代码

---

- **@NoArgsConstructor**：用来生成一个默认的无参构造方法。
- **@RequiredArgsConstructor**：使用类中所有带有`@NonNull`注解和`final`修饰的字段生成对应的构造方法。
- **@Data**：等同于以下几个注解合集：@Getter、@Setter、@RequiredArgsConstructor、@ToString、@EqualsAndHashCode
- **@NonNull**：用于字段的非空检查，如果传入到set方法中的值为空，则抛出空指针异常，该注解也会生成一个默认的构造方法。

