#### 1. spring-boot-devtools

---

[`spring-boot-devtools`](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-devtools) 是 Spring Boot 提供的开发者工具，它会监控当前应用所在的 classpath 下的文件发生变化，进行自动重启。

注意，`spring-boot-devtools` 并==没有采用热部署的方式，而是一种较快的重启方式==。其官方文档解释如下：

>FROM [《Spring Boot 2.X 中文文档 —— 开发者工具》](https://docshome.gitbooks.io/springboot/content/pages/using-spring-boot.html#using-boot-devtools-restart)
>
>Spring Boot 通过使用两个类加载器来提供了重启技术。
>
>- 不改变的类（例如，第三方 jar）被加载到 **base** 类加载器中。
>- 经常处于开发状态的类被加载到 **restart** 类加载器中。
>
>当应用重启时，**restart** 类加载器将被丢弃，并重新创建一个新的。这种方式意味着应用重启比**冷启动**要快得多，因为省去 **base** 类加载器的处理步骤，并且可以直接使用。
>
>如果您觉得重启还不够快，或者遇到类加载问题，您可以考虑如 ZeroTurnaround 的 [JRebel](https://zeroturnaround.com/software/jrebel/) 等工具。他们是通过在加载类时重写类来加快重新加载。

在项目中的 `pom.xml` 中，引入 `spring-boot-devtools` 依赖如下：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
  <optional>true</optional> <!-- 防止将devtools依赖传递到其他模块中 -->
</dependency>
```

>修改Java代码后，需要重新编译。Build` -> `Build Project
>
>可以使用 `Build Project` 的快捷键：
>
>- Mac：Command + F9
>- Windows：Ctrl + F9

#### 2. IDEA热部署

---

IDEA提供了Hotswap插件，可以实现真正的热部署。

debug模式下修改代码，需要重新编译。