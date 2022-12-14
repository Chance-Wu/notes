**Jackson作为springMVC默认的MessageConverter（消息序列化工具）**，经常在项目中使用，如果熟悉Jackson常用的使用方法，特性化机制，就会事半功倍，极大提高前后端数据交互的灵活性。

maven依赖：

```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>
  <version>2.9.5</version>
</dependency>
```

使用jackson需要三个jar包，jackson-databind、jackson-core和jackson-annotations，添加一个依赖 **jackson-databind** 就可以拥有这三个jar包。



### 一、基础功能

---

jackson使用的最多的就是jackson-databind包，官方文档地址是[https://github.com/FasterXML/jackson-databind](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FFasterXML%2Fjackson-databind)，使用jackson序列化对象使用ObjectMapper类，如下：

```java
ObjectMapper objectMapper = new ObjectMapper();
User user = new User("zhangsan", "123456");
// class to json
String userStr = objectMapper.writeValueAsString(user);
// json to class
User user2 = objectMapper.readValue(userStr, User.class);
```



### 二、常用注解

---

jackson注解主要涉及到的jar包为jackson-annotations.jar，官方文档地址[https://github.com/FasterXML/jackson-annotations/wiki/Jackson-Annotations](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2FFasterXML%2Fjackson-annotations%2Fwiki%2FJackson-Annotations)

#### 2.1 @JsonProperty

用在属性或者方法上面，用于改变序列化时字段的名称。

```java
public class User {
  @JsonProperty("user_name")
  public String username;
  public String password;
}

// 序列化为如下格式username变成了user_name
{"password":"123456","user_name":"zhangsan"}
```

#### 2.2 @JsonIgnoreProperties

用在类上面，用于序列化时忽略指定字段。

```java
@JsonIgnoreProperties({"username", "price"})
public class Pet {

  private String username;
  private String password;
  private Date birthday;
  private Double price;
}

// 指定序列化时忽略username、price字段
{"password":"123456","birthday":1533887811261}
```

#### 2.3 @JsonIgnore

用在属性上面，用于序列化时忽略该属性。

```java
public class Pet {

  @JsonIgnore
  private String username;
  private String password;
  private Date birthday;
  private Double price;
}

// @JsonIgnore加在属性上面，使序列化时忽略该字段
{"password":"123456","birthday":1533888026016,"price":0.6}
```

#### 2.4 @JsonFormat

用在Date时间类型属性上面，用于序列化时间为需要的格式。

```java
public class Pet {
  private String username;
  private String password;
  @JsonFormat(pattern="yyyy-MM-dd HH:mm:ss", timezone="GMT+8")
  private Date birthday;
  private Double price;
}

// @JsonFormat加在属性上面，用于jackson对时间格式规定，注意要指定中国时区为+8
{"username":"哈哈","password":"123456","birthday":"2018-08-10 16:17:51","price":0.6}
```

#### 2.5 @JsonInclude

用在类上面，用于声明在序列化时忽略一些没有意义的字段，例如：属性为NULL的字段。

```java
@JsonInclude(Include.NON_NULL)
public class Pet {
  private String username;
  private String password;
  private Date birthday;
  private Double price;
}

// @JsonInclude加在类上面，jackson序列化时会忽略无意义的字段，例如username和price是空值，那么就不序列化这两个字段
{"password":"123456","birthday":1533890045175}
```

#### 2.6 @JsonSerialize

用在类或属性上面，用于指定序列化时使用的JsonSerialize类。

一般会使用自定义的序列化器，例如自定义MyJsonSerializer，用来处理Double类型序列化时保留两位小数。

```java
public class MyJsonSerializer extends JsonSerializer<Double>{
  @Override
  public void serialize(Double value, JsonGenerator gen, SerializerProvider serializers)
    throws IOException, JsonProcessingException {
    if (value != null)
      gen.writeString(BigDecimal.valueOf(value).
                      setScale(2, BigDecimal.ROUND_HALF_UP).toString());
  }
}
```

使用方式：

```java
public class Pet {
  private String username;
  private String password;
  private Date birthday;
  @JsonSerialize(using=MyJsonSerializer.class)
  private Double price;
}

// 指定序列化price属性时使用自定义MyJsonSerializer，对Double类型进行自定义处理，保留两位小数
{"username":"哈哈","password":"123456","birthday":1533892290795,"price":"0.60"}
```



### 三、Jackson和SpringBoot整合使用

---

jackson作为spring-boot的默认MessageConverter工具，他被包含在spring-boot-starter-web依赖中，这里主要说明spring-boot如何对jackson进行个性化配置，重写WebMvcConfigurerAdapter类的configureMessageConverters方法即可，如下

```java
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {
  @Override
  public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    ObjectMapper objectMapper = new ObjectMapper();
    // 设置Date类型字段序列化方式
    objectMapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.SIMPLIFIED_CHINESE));

    // 指定BigDecimal类型字段使用自定义的CustomDoubleSerialize序列化器
    SimpleModule simpleModule = new SimpleModule();
    simpleModule.addSerializer(BigDecimal.class, new CustomDoubleSerialize());
    objectMapper.registerModule(simpleModule);

    MappingJackson2HttpMessageConverter converter = new MappingJackson2HttpMessageConverter(objectMapper);
    converters.add(converter);
  }
}
```

可以看出，spring-boot对于jackson的使用更加灵活。