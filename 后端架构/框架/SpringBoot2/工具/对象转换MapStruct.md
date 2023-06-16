#### 1. 概述

---

为了让应用的代码更易维护，我们往往会将项目进行分层。在[《阿里巴巴 Java 开发手册》](https://github.com/alibaba/p3c/blob/master/阿里巴巴Java开发手册（华山版）.pdf)中，推荐分层如下图：

![](https://tva1.sinaimg.cn/large/008i3skNgy1gxrmzi48lhj30dm0bp74e.jpg)

分层之后，每一层都有自己的领域模型，即不同类型的 Bean：

- **DO**（Data Object）：与数据库表结构一一对应，通过DAO层向上传输数据源对象。
- **DTO**（Data Transfer Object）：数据传输对象，Service或Manager向外传输的对象。
- **BO**（Business Object）：业务对象。由Service层输出的封装业务逻辑的对象。
- 等等…

那么，进行就需要这些**对象的转换**。例如说：

```java
// 从数据库中查询用户
UserDO userDO = userMapper.selectBy(id);

// 对象转换
UserBO userBO = new UserBO();
userBO.setId(userDO.getId());
userBO.setUsername(userDO.getUsername());
// ... 还有其它属性
```

显然，**手动**进行对象的转换，虽然**执行性能**很高，但是**开发效率**非常低下，且可能会存在漏写的情况。因此，我们会选择借助框架或是工具来实现对象的转换，例如说：

- Spring BeanUtils
- Apache BeanUtils
- Dozer
- Orika
- MapStruct
- ModelMapper
- JMapper

 [MapStruct](https://mapstruct.org/)，基于 [JSR 269 的 Java 注解处理器](https://jcp.org/en/jsr/detail?id=269)，**自动生成**对象的代码，使用便捷，性能优秀。

通过创建一个 MapStruct **Mapper** 接口，并定义一个转换接口方法，后续交给 MapStruct 自动生成对象转换的代码即可。

#### 2. 使用

---

```xml
<dependencies>
  <dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.3.1.Final</version>
  </dependency>
</dependencies>

<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.8.1</version>
      <configuration>
        <source>${java.version}</source>
        <target>${java.version}</target>
        <annotationProcessorPaths>
          <path>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct-processor</artifactId>
            <version>1.3.1.Final</version>
          </path>
        </annotationProcessorPaths>
      </configuration>
    </plugin>
  </plugins>
</build>
```

>一定要在maven-compiler-plugin插件中，声明mapstruct-processor为JSR 269的Java注解处理器。

```java
public class UserDO {
  private Integer id;
  private String username;
  private String password;
  // ... 省略 setter/getter 方法
}
```

```java
public class UserBO {
  private Integer id;
  private String username;
  private String password;
  // ... 省略 setter/getter 方法
}
```

创建UserConvert接口，作为User相关的Bean的转换器。

```java
@Mapper // <1> 该注解声明它是一个MapStruct Mapper映射器
public interface UserConvert {

  // <2>调用 Mappers 的 #getMapper(Class<T> clazz) 方法，获得 MapStruct 帮我们自动生成的 UserConvert 实现类的对象。
  UserConvert INSTANCE = Mappers.getMapper(UserConvert.class);

  // <3>定义 #convert(UserDO userDO) 方法，声明 UserDO 转换成 UserBO
  UserBO convert(UserDO userDO);

}
```

#### 3. 集成Lombok

---

MapStruct 自动生成的对象转换的代码，也是依赖 setter、getter 方法的，因此两者在一起使用时，需要进行相应的配置。如下图所示：

```xml
<dependencies>
  <!-- 引入 mapstruct 依赖 -->
  <dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>${mapstruct.version}</version>
  </dependency>

  <!-- 引入 lombok 依赖 -->
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>${lombok.version}</version>
    <scope>provided</scope>
  </dependency>
</dependencies>

<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.8.1</version>
      <configuration>
        <source>${java.version}</source>
        <target>${java.version}</target>
        <annotationProcessorPaths>
          <!-- 引入 mapstruct-processor -->
          <path>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct-processor</artifactId>
            <version>${mapstruct.version}</version>
          </path>
          <!-- 引入 lombok-processor -->
          <path>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
          </path>
        </annotationProcessorPaths>
      </configuration>
    </plugin>
  </plugins>
</build>
```

```java
@Data
@Accessors(chain = true)
public class UserDO {
  private Integer id;
  private String username;
  private String password;
}
```

```java
@Data
@Accessors(chain = true)
public class UserDO {
  private Integer id;
  private String username;
  private String password;
}
```

#### 4. @Mapping

---

在对象转换时，可能会存在属性不是完全映射的情况，例如说属性名不同。此时，可以使用@Mapping注解，配置相应的映射关系。

```java
@Mappings({
  @Mapping(source = "id", target = "userId")
})
UserDetailBO convertDetail(UserDO userDO);
```

`@Mapping` 注解还支持多个对象转换为一个对象。

```java
@Mappings({
  @Mapping(source = "adminBO.id", target = "id"),
  @Mapping(source = "adminBO.name", target = "name"),
  @Mapping(source = "accessTokenBO.id", target = "token.accessToken"),
  @Mapping(source = "accessTokenBO.refreshToken", target = "token.refreshToken"),
  @Mapping(source = "accessTokenBO.expiresTime", target = "token.expiresTime"),
})
AdminsOAuth2AuthenticateResponse convert(AdminBO adminBO, OAuth2AccessTokenbo accessTokenBO);
```

#### 5. IDEA MapStruct插件

---

MapStruct 提供了 [IDEA MapStruct Support](https://plugins.jetbrains.com/plugin/10036-mapstruct-support/) 插件。