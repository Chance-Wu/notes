### 一、target和source

---

在 MapStruct 中，@Mapping(target = …, source = …) 是用于定义源对象（Source）到目标对象（Target）之间属性映射关系的注解。

target: target 属性指定了目标对象中的一个属性。这个属性将接收从源对象中转换或映射过来的值。例如：

```java
@Mapping(target = "username", ...)
```

这里表示我们将映射一个值到目标对象的 username 属性上。

source: source 属性指定了源对象中的一个属性或方法。这个属性或方法的值将被转换或映射到目标对象的指定属性上。例如：

```java
@Mapping(target = "username", source = "name")
```

这里表示我们将源对象的 name 属性的值映射到目标对象的 username 属性上。

如果源和目标属性的名字相同，你可以省略 source 属性，MapStruct 会自动匹配。例如：

```java
@Mapping(target = "username")
```

这里表示我们将源对象的 username 属性的值映射到目标对象的 username 属性上，因为它们的名字相同，所以可以省略 source 属性。

示例：创建一个 UserMapper 接口来定义这些属性之间的映射关系：

```java
@Mapper
public interface UserMapper {
  UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);
  @Mapping(target = "username", source = "name")
  @Mapping(target = "userAge", source = "age")
  UserDto toUserDto(User user);
}
```

在这个例子中，我们指定了两个映射关系：

target = “username” 和 source = “name” 表示将 User 对象的 name 属性映射到 UserDto 对象的 username 属性。

target = “userAge” 和 source = “age” 表示将 User 对象的 age 属性映射到 UserDto 对象的 userAge 属性。



### 二、dateformat

---

dateformat用来格式化时间类型字符串。

```java
@Mapping(source = "createDate", target = "createDate", dateFormat = "yyyy/MM/dd")
ProductDto toProductDto(Product product);
```



### 三、expression使用

---

expression 属性值是一个字符串，它**会被解析为 Java 表达式**，并在运行时执行。

使用expression时，其中必须为java()格式。

应用场景：在转换属性时，可能需要调用java里的甚至jar包中的各种工具类或者自定义方法产生新的数据：

```java
@Mapper
public interface ProductMapper {

  @Mapping(target = "idBool",
           expression = "java( cn.hutool.core.util.StrUtil.isNotBlank( product.getId() ))")
  @Mapping(target = "price", 
           expression = "java( java.math.BigDecimal.valueOf( product.getPrice() ))")
  ProductDto toProductDto(Product product);
}
```

想要ProductDto 对象中idBool显示product的Id是否存在，price字段转为BigDecimal类型。

查看生成的代码：

```java
productDto.setPrice( java.math.BigDecimal.valueOf( product.getPrice() ) );
productDto.setIdBool( cn.hutool.core.util.StrUtil.isNotBlank( product.getId() ) );
```



### 四、defaultExpression属性

---

当且仅当source属性为null的时候，将target指定的属性值设置为该表达式的值。不能同时和expression、defaultValue以及constant属性一起使用！



### 五、qualifiedByName和conditionQualifiedByName

---

```java
@Mapper
public interface ProductMapper {

  ProductMapper INSTANCE = Mappers.getMapper(ProductMapper.class);

  @Mapping(target = "id", source = "id", qualifiedByName = "toId")
  ProductDto toProductDto(Product product);

  @Named("toId")
  static String toId(String id) {
    return "1";
  }
}
```

查看生成代码：

```java
productDto.setId( ProductMapper.toId( product.getId() ) );
```

```java
@Mapper
public interface ProductMapper {
  ProductMapper INSTANCE = Mappers.getMapper(ProductMapper.class);

  @Condition
  @Named("showId")
  static boolean showId(String value) {
    return "333333".equals(value);
  }

  @Mapping(target = "id", source = "id", conditionQualifiedByName = "showId")
  ProductDto toProductDto(Product product);

  @Named("toId")
  static String toId(String id) {
    return "1";
  }
}
```

此时当只有ID为333333才能正常copy。