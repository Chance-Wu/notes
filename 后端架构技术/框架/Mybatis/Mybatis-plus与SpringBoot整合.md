#### jar包

---

```xml
<dependency>
  <groupId>com.baomidou</groupId>
  <artifactId>mybatis-plus-boot-starter</artifactId>
  <version>3.3.2</version>
</dependency>
```



#### 添加配置

---

设置针对的是Mybatis-plus

```yaml
mybatis-plus:
  type-aliases-package: com.chance.entity #为实体类起别名
  mapper-locations: classpath:mapper/*.xml #mapper配置文件位置
  configuration:
    map-underscore-to-camel-case: true
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  type-handlers-package: com.chance.entity.typehandler #自定义类型处理器
```



#### 添加必要的配置类

---

```java
@Configuration
@EnableTransactionManagement
@MapperScan("com.chance.mapper")
public class MybatisPlusConfig {

    /**
     * 分页插件
     * @author fxbin
     * @return com.baomidou.mybatisplus.extension.plugins.PaginationInterceptor
     */
    @Bean
    public PaginationInterceptor paginationInterceptor() {
        return new PaginationInterceptor();
    }
}
```



#### 定义实体

---

```java
@Data
@TableName(value = "ANIMAL")
public class Animal {
  @TableId(value = "ID",type = IdType.AUTO)
  private Integer id;
  @TableField(value = "NAME",exist = true)
  private String name;
  @TableField(value = "TYPE",exist = true)
  private AnimalType type;
  @TableField(value = "SEX",exist = true)
  private AnimalSex sex;
  @TableField(value = "MASTER",exist = true)
  private String master;
}
```



#### Mapper接口

---

```java
public interface AnimalRepository extends BaseMapper<Animal> {
}
```

使用Mybatis-plus后Mapper接口只要继承BaseMapper接口即可，即使不添加XML映射文件也可以实现该接口提供的增删改查功能，还可以配合Wrapper进行条件操作，当然这些操作都仅仅限于单表操作。



#### 乐观锁插件

---

```java
/**
 * 乐观锁插件
 */
@Bean
public OptimisticLockerInterceptor optimisticLockerInterceptor() {
  return new OptimisticLockerInterceptor();
}
```

添加@Version：

```java
@Version
private int version;
```

- @Version支持的数据类型只有int、Integer、long、Long、Date、Timestamp、LocalDateTime；
- 整数类型下newVersion = oldVersion+1;
- newVersion会回写到entity中；
- 仅支持updateById(id)与update(entity, wrapper)方法；
- 在update(entity, wrapper)方法下，wrapper不能复用。



#### 实体主键配置

---

```java
@TableId(value = "ID",type = IdType.AUTO)
private Integer id;
```

```java
@Getter
public enum IdType {
  /**
   * 数据库ID自增
   */
  AUTO(0),
  /**
   * 该类型为未设置主键类型(注解里等于跟随全局,全局里约等于 INPUT)
   */
  NONE(1),
  /**
   * 用户输入ID
   * <p>该类型可以通过自己注册自动填充插件进行填充</p>
   */
  INPUT(2),

  /* 以下3种类型、只有当插入对象ID 为空，才自动填充。 */
  /**
   * 分配ID (主键类型为number或string）,
   * 默认实现类 {@link com.baomidou.mybatisplus.core.incrementer.DefaultIdentifierGenerator}(雪花算法)
   *
   * @since 3.3.0
   */
  ASSIGN_ID(3),
  /**
   * 分配UUID (主键类型为 string)
   * 默认实现类 {@link com.baomidou.mybatisplus.core.incrementer.DefaultIdentifierGenerator}(UUID.replace("-",""))
   */
  ASSIGN_UUID(4),

  private final int key;

  IdType(int key) {
    this.key = key;
  }
}
```