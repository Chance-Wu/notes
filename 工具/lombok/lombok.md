lombok注解在java进行编译时进行代码的构建，对于java对象的创建工作它 更优雅，不需要写多余的重复代码。

@Builder

对象的创建工作提供了Builder方法，它提供在设计数据实体时，对外保持private setter，而对属性的复制采用Builder的方式，这种方式最优雅，也更符合封装的原则，不对外公开属性的写操作。

此注解声明实体，表示可以进行Builder方式初始化。

```java
UserInfo userInfo = UserInfo.builder()
  .name(“wcy”)
  .email(“wcy@sina.com”)
  .build();
```



