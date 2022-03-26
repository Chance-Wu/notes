##### 1. Jackson核心模块组成

>- **jackson-core**，核心包，提供基于"流模式"解析的相关 API，它包括 JsonPaser 和 JsonGenerator。Jackson 内部实现正是通过高性能的流模式 API 的 JsonGenerator 和 JsonParser 来生成和解析 json。
>- **jackson-annotations**，注解包，提供标准注解功能；
>- **jackson-databind**，数据绑定包， 提供基于"对象绑定" 解析的相关 API （ ObjectMapper ） 和"树模型" 解析的相关 API （JsonNode）；基于"对象绑定" 解析的 API 和"树模型"解析的 API 依赖基于"流模式"解析的 API。

>`jackson-databind` 依赖 jackson-core 和 jackson-annotations，当添加 jackson-databind 之后， jackson-core 和 jackson-annotations 也随之添加到 Java 项目工程中。
>
>```xml
><dependency>
>    <groupId>com.fasterxml.jackson.core</groupId>
>    <artifactId>jackson-databind</artifactId>
>    <version>2.11.3</version>
></dependency>
>```

##### 2. ObjectMapper的使用

>最常用的 API 就是==基于"对象绑定" 的 ObjectMapper==。
>
>```java
>ObjectMapper objectMapper = new ObjectMapper();
>Person person = new Person();
>person.setName("Tom");
>person.setWeigth(40);
>String jsonString = objectMapper.writerWithDefaultPrettyPrinter()
>    .writeValueAsString(person);
>Person deserializedPerson = objectMapper.readValue(jsonString, Person.class);
>```
>
>`ObjectMapper`通过*<u>writeValue系列方法</u>*将java对象序列化为json，并将json存储成不同的格式：
>
>- String（writeValueAsString）
>- Byte Array（writeValueAsBytes）
>- Writer
>- File
>- OutStream 和 DataOutput。
>
>`ObjectMapper`通过 *<u>readValue 系列方法</u>*从不同的数据源（如 String，Byte，Array，Reader，File，URL，InputStream ）将 json 反序列化为 java 对象。

##### 3. 信息配置

>在调用writeValue或调用readValue方法之前，往往需要设置ObjectMapper的相关配置信息。这些配置信息应用Java对象的所有属性上。
>
>```java
>ObjectMapper objectMapper = new ObjectMapper();
>// 设置ObjectMapper的相关配置信息
>// 在反序列化是忽略在 json 中存在但 Java对象不存在的属性
>objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
>// 在序列化是日期格式默认为 yyyy-MM-dd'T'HH:mm:ss.SSSZ
>objectMapper.configure(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS, false);
>// 在序列化时忽略值为 null 的属性
>objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
>// 忽略值为默认值的属性
>objectMapper.setDefaultPropertyInclusion(JsonInclude.Include.NON_DEFAULT);
>```

##### 4. Jackson 的注解的使用

>Jackson 根据它的默认方式序列化和反序列化 java 对象，*<u>若根据实际需要，灵活的调整它的默认方式，可以使用 Jackson 的注解</u>*。常用的注解及用法如下。
>
>| 注解               | 用法                                                         |
>| ------------------ | ------------------------------------------------------------ |
>| **@JsonProperty**  | 把属性的名称序列化时转换为另外一个名称。示例：  @JsonProperty("birth_ d ate")  private Date birthDate; |
>| **@JsonFormat**    | 用于属性或者方法，把属性的格式序列化时转换成指定的格式。示例： @JsonFormat(timezone = "GMT+8", pattern = "yyyy-MM-dd HH:mm")  public Date getBirthDate() |
>| @JsonPropertyOrder | 用于类，指定属性在序列化时 json 中的顺序 ，示例： @JsonPropertyOrder({ "birth_Date", "name" })  public class Person |
>| @JsonCreator       | 用于构造方法，和 @JsonProperty 配合使用，适用有参数的构造方法。 示例：@JsonCreator  public Person(@JsonProperty("name")String name) {…} |
>| **@JsonAnySetter** | 用于属性或者方法，设置未反序列化的属性名和值作为键值存储到 map 中  @JsonAnySetter  public void set(String key, Object value) {  map.put(key, value);  } |
>| **@JsonAnyGetter** | 用于方法 ，获取所有未序列化的属性  public Map<String, Object> any() { return map; } |

##### 6. 从Reader读取对象

```java
ObjectMapper objectMapper = new ObjectMapper();

// 从Reader中读取对象
String carJson = "{\"brand\":\"Mercedes\",\"doors\":4}";
StringReader reader = new StringReader(carJson);
Car car = objectMapper.readValue(reader, Car.class);
```

##### 7. 从File中读取对象

```java
File file = new File("src/main/resources/car.json");
System.out.println(file.getAbsolutePath());
System.out.println(System.getProperty("user.dir"));
Car car2 = objectMapper.readValue(file, Car.class);
```

##### 8. 从URL中读取对象

```java
URL url = new URL("file:/Users/chance/IdeaProjects/java-basis/src/main/resources/car.json");
Car car3 = objectMapper.readValue(url, Car.class);
```

##### 9. 从InputStream读取对象

```java
InputStream inputStream = new FileInputStream("src/main/resources/car.json");
Car car4 = objectMapper.readValue(inputStream, Car.class);
```

##### 10. 从字节数组中读取对象

```java
byte[] bytes = carJson.getBytes("UTF-8");
Car car5 = objectMapper.readValue(bytes, Car.class);
```

##### 11. 从JSON数组字符串中读取对象数组

```java
String jsonArray = "[{\"brand\":\"ford\"}, {\"brand\":\"Fiat\"}]";
Car[] carArray = objectMapper.readValue(jsonArray, Car[].class);
```

##### 12. 从JSON数组字符串中读取对象列表

```java
List<Car> carList = objectMapper.readValue(jsonArray, new TypeReference<List<Car>>() {
});
```

##### 13. 从JSON字符串中读取映射为map

```java
String jsonObject = "{\"brand\":\"ford\",\"doors\":5}";
Map<String, Object> jsonMap = objectMapper.readValue(jsonObject,
                                                     new TypeReference<Map<String, Object>>() {
                                                     });
```

##### 14. 树模型

```java
JsonNode jsonNode = objectMapper.readValue(carJson, JsonNode.class);
```

>JSON字符串被解析为`JsonNode`对象而不是`Car`对象。只需将`JsonNode.class`作为第二个参数传递给`readValue()`方法。
>
>该`ObjectMapper`也有一个特殊的`readTree()`方法，它总是返回一个 `JsonNode`。以下是`JsonNode`使用该`ObjectMapper` 的 `readTree()`方法将JSON解析为jsonNode的示例：
>
>```java
>JsonNode jsonNode1 = objectMapper.readTree(carJson);
>```

>**JsonNode类**
>
>```java
>JsonNode brandNode = jsonNode.get("brand");
>String brand = brandNode.asText();
>System.out.println("brand = " + brand);
>
>JsonNode doorsNode = jsonNode.get("doors");
>int doors = doorsNode.asInt();
>System.out.println("doors = " + doors);
>```

##### 15. 将Object转换为JsonNode

```java
ObjectMapper objectMapper = new ObjectMapper();
Car car = new Car();
car.setBrand("Cadillac");
car.setDoors(4);

JsonNode jsonNode = objectMapper.valueToTree(car);
```

##### 16. 将JsonNode抓换为Object

```java
ObjectMapper objectMapper = new ObjectMapper();

String carJson = "{ \"brand\" : \"Mercedes\", \"doors\" : 5 }";
JsonNode carJsonNode = objectMapper.readTree(carJson);

Car car = objectMapper.treeToValue(carJsonNode, Car.class);
```

##### 17. 使用Jackson ObjectMapper读取和编写

###### 17.1 yaml字符串和对象的互转

>```java
>ObjectMapper objectMapper = new ObjectMapper(new YAMLFactory());
>
>Employee employee = new Employee("John Doe", "john@doe.com");
>
>String yamlString;
>try {
>    // 序列化为yaml格式的字符串对象
>    yamlString = objectMapper.writeValueAsString(employee);
>    System.out.println(yamlString);
>} catch (JsonProcessingException e) {
>    e.printStackTrace();
>}
>```
>
>该`yamlString`变量包含`Employee`在执行此代码后序列化为YAML数据格式的对象。
>
><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gld1lses5aj30ca03qweb.jpg" style="zoom:40%">
>
>以下是`Employee`再次将YAML文本读入对象的示例：
>
>```java
>try {
>    // 反序列化为Employee对象
>    Employee employee2 = objectMapper.readValue(yamlString, Employee.class);
>    System.out.println(employee2);
>} catch (IOException e) {
>    e.printStackTrace();
>}
>```

###### 17.2 创建要读取的Employee.yml文件

>```yaml
>name: test
>email: test@qq.com
>```
>
>创建要写入的EmployeeYamlOutput.yml（空文件）
>
>```java
>public class YamlJacksonExample {
>
>    public static void main(String[] args) {
>        try {
>            //从yaml文件读取数据
>            reaedYamlToEmployee();
>            //写入yaml文件
>            reaedEmployeeToYaml();
>        } catch (Exception e) {
>            e.printStackTrace();
>        }
>    }
>
>    /**
>     * 从yaml文件读取数据
>     *
>     * @throws IOException
>     */
>    private static void reaedYamlToEmployee() throws IOException {
>        ObjectMapper mapper = new ObjectMapper(new YAMLFactory());
>        Employee employee = mapper.readValue(new File("src/main/resources/EmployeeYaml.yml"), Employee.class);
>        System.out.println(employee.getName() + "********" + employee.getEmail());
>
>    }
>
>    /**
>     * 写入yaml文件
>     *
>     * @throws IOException
>     */
>    private static void reaedEmployeeToYaml() throws IOException {
>        //去掉三个破折号
>        ObjectMapper mapper = new ObjectMapper(new YAMLFactory().disable(YAMLGenerator.Feature.WRITE_DOC_START_MARKER));
>        //禁用掉把时间写为时间戳
>        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
>
>        Employee employee = new Employee("test2", "999@qq.com");
>        mapper.writeValue(new File("src/main/resources/EmployeeYamlOutput.yml"), employee);
>    }
>}
>```

##### 18. Jackson工具类

>```java
>package com.chance.toolkit.utils;
>
>import com.fasterxml.jackson.core.JsonProcessingException;
>import com.fasterxml.jackson.databind.*;
>import com.fasterxml.jackson.databind.util.JSONPObject;
>import org.apache.commons.lang.StringUtils;
>import org.slf4j.Logger;
>import org.slf4j.LoggerFactory;
>
>import java.io.IOException;
>import java.text.DateFormat;
>import java.text.SimpleDateFormat;
>
>import static com.fasterxml.jackson.annotation.JsonInclude.Include;
>
>/**
> * @Description: JacksonBundle
> * @Author: chance
> * @Date: 12/5/20 4:22 PM
> * @Version 1.0
> */
>public class JacksonBundle {
>
>    private static Logger logger = LoggerFactory.getLogger(JacksonBundle.class);
>
>    private ObjectMapper mapper;
>
>    /**
>     * 构造方法
>     */
>    public JacksonBundle() {
>        this(null);
>    }
>
>
>    /**
>     * 根据Jackson的 {@link Include}类创建ObjectMapper
>     *
>     * @param include
>     */
>    public JacksonBundle(Include include) {
>        mapper = new ObjectMapper();
>        if (include != null) {
>            mapper.setSerializationInclusion(include);
>        }
>        mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
>    }
>
>    /**
>     * 创建序列化属性为所有对象认为可能是空值的ObjectMapper
>     *
>     * <p>
>     * 默认是的空值判断有以下类型:
>     * <ul>
>     * <li>
>     * 如果是 {@link java.util.Collections} 或 {@link java.util.Map} 。根据 <code>isEmpty()</code>方法返回值判断是否序列化
>     * </li>
>     * <li>
>     * 如果是Java数组，长度等于0都为空值的情况
>     * </li>
>     * <li>
>     * 如是String类型根据 <code>length()</code>方法返回值判断是否序列化,如果为0表示该值为空
>     * </li>
>     * </ul>
>     * </p>
>     * 其他类型的默认处理为：空值将一样加入。
>     * <p>
>     * 提示：其他类型的默认处理可以自定义{@link JsonSerializer}的{@link JsonSerializer#isEmpty(Object)}如果被重写，
>     * 序列化json会调用该方法，根据返回值判断是否序列化
>     * </p>
>     *
>     * @return {@link JacksonBundle}
>     */
>    public static JacksonBundle nonEmptyMapper() {
>
>        return new JacksonBundle(Include.NON_EMPTY);
>    }
>
>    /**
>     * 创建不管任何值都序列化成json的ObjectMapper
>     *
>     * @return {@link JacksonBundle}
>     */
>    public static JacksonBundle alwaysMapper() {
>        return new JacksonBundle(Include.ALWAYS);
>    }
>
>    /**
>     * 创建序列化属性为非空(null)值的ObjectMapper
>     *
>     * @return {@link JacksonBundle}
>     */
>    public static JacksonBundle nonNullMapper() {
>        return new JacksonBundle(Include.NON_NULL);
>    }
>
>    /**
>     * 创建只输出初始值被改变的属性到Json字符串的ObjectMapper, 最节约的存储方式。
>     *
>     * @return
>     */
>    public static JacksonBundle nonDefaultMapper() {
>        return new JacksonBundle(Include.NON_DEFAULT);
>    }
>
>    /**
>     * 将json字符串转换为对象.
>     * <pre>
>     * 如果json字符串为null或"null"字符串,返回null. 如果json字符串为"[]",返回空集合.
>     * 如需读取集合如List/Map,且不是List&lt;String&gt;这种简单类型时使用如下语句:
>     * List&lt;MyBean&gt; beanList = binder.getMapper().readValue(listString, new TypeReference&lt;List&lt;MyBean&gt;&gt;() {});
>     * </pre>
>     *
>     * @param json        json字符串
>     * @param tragetClass 转换对象的Class
>     * @return <T>
>     */
>    public <T> T fromJson(String json, Class<T> tragetClass) {
>        if (StringUtils.isEmpty(json)) {
>            return null;
>        }
>
>        try {
>            return mapper.readValue(json, tragetClass);
>        } catch (IOException e) {
>            logger.warn("parse json string error:" + json, e);
>            return null;
>        }
>    }
>
>    /**
>     * 反序列化复杂Collection如List<Bean>, 先使用函数createCollectionType构造类型,然后调用本函数.
>     *
>     * @see #createCollectionType(Class, Class...)
>     */
>    public <T> T fromJson(String jsonString, JavaType javaType) {
>        if (StringUtils.isEmpty(jsonString)) {
>            return null;
>        }
>
>        try {
>            return (T) mapper.readValue(jsonString, javaType);
>        } catch (IOException e) {
>            logger.warn("parse json string error:" + jsonString, e);
>            return null;
>        }
>    }
>
>    /**
>     * 构造泛型的Collection Type如:
>     * <p>
>     * ArrayList<MyBean>, 则调用constructCollectionType(ArrayList.class,MyBean.class)
>     * HashMap<String,MyBean>, 则调用(HashMap.class,String.class, MyBean.class)
>     * </p>
>     */
>    public JavaType createCollectionType(Class<?> collectionClass, Class<?>... elementClasses) {
>        return mapper.getTypeFactory().constructParametricType(collectionClass, elementClasses);
>    }
>
>    /**
>     * 当JSON里只含有Bean的部分属性时，更新一个已存在Bean，只覆盖部分属性.
>     */
>    public <T> T update(String jsonString, T object) {
>        try {
>            return (T) mapper.readerForUpdating(object).readValue(jsonString);
>        } catch (JsonProcessingException e) {
>            logger.warn("update json string:" + jsonString + " to object:" + object + " error.", e);
>        } catch (IOException e) {
>            logger.warn("update json string:" + jsonString + " to object:" + object + " error.", e);
>        }
>        return null;
>    }
>
>    /**
>     * 将对象转换成json字符串,如果对象为Null,返回"null". 如果集合为空集合,返回"[]".
>     *
>     * @param target 转换为json的对象
>     */
>    public String toJson(Object target) {
>
>        try {
>            return mapper.writeValueAsString(target);
>        } catch (IOException e) {
>            logger.warn("write to json string error:" + target, e);
>            return null;
>        }
>    }
>
>    /**
>     * 设置转换日期类型的时间科室,如果不设置默认打印Timestamp毫秒数.
>     *
>     * @param pattern 时间格式化字符串
>     */
>    public void setDateFormat(String pattern) {
>        if (!StringUtils.isEmpty(pattern)) {
>            DateFormat dateFormat = new SimpleDateFormat(pattern);
>            mapper.getSerializationConfig().with(dateFormat);
>            mapper.getDeserializationConfig().with(dateFormat);
>        }
>    }
>
>    /**
>     * 将对象转换成JSONP格式字符串
>     *
>     * @param function jsonp回调方法名
>     * @param target   转换为jsonp的对象
>     */
>    public String toJsonP(String function, Object target) {
>        return toJson(new JSONPObject(function, target));
>    }
>
>    /**
>     * 设置是否使用Enum的toString函数来读取Enum,为false时使用Enum的name()函数类读取Enum, 默认为false.
>     */
>    public void enableEnumUseToString() {
>        mapper.enable(SerializationFeature.WRITE_ENUMS_USING_TO_STRING);
>        mapper.enable(DeserializationFeature.READ_ENUMS_USING_TO_STRING);
>    }
>
>    /**
>     * 取出Mapper做进一步的设置或使用其他序列化API.
>     *
>     * @return {@link ObjectMapper}
>     */
>    public ObjectMapper getMapper() {
>        return mapper;
>    }
>}
>```

