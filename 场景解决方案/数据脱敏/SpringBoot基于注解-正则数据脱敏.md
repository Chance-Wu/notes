### 一、自定义注解

---

```java
@JacksonAnnotationsInside
@JsonSerialize(using = MyJsonSerializer.class)
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
public @interface CustomSerializer {

  /**
   * 脱敏规则处理类
   * @return
   */
  Class<? extends BaseRule> value() default DefaultRule.class;

  /**
   * 正则，pattern和format必需同时有值。如果都有值时，优先使用正则进行规则替换
   * @return
   */
  String pattern() default "";

  String format() default "";

}
```



### 二、注解检查转换类

---

```java
@Slf4j
public class MyJsonSerializer extends JsonSerializer<String> implements ContextualSerializer {

  /**
   * 脱敏规则
   */
  private BaseRule rule;

  @Override
  public void serialize(String value, JsonGenerator gen, SerializerProvider serializers) throws IOException {
    gen.writeString(rule.apply(value));
  }

  @Override
  public JsonSerializer<?> createContextual(SerializerProvider prov, BeanProperty property) throws JsonMappingException {
    //获取对象属性上的自定义注解
    CustomSerializer customSerializer = property.getAnnotation(CustomSerializer.class);
    if (null != customSerializer) {
      try {
        //根据注解的配置信息，创建对应脱敏规则处理类
        this.rule = customSerializer.value().newInstance();
        //如果正则信息不为空，则使用注解上的正则初始化到对应的脱敏规则处理类中
        if (isNotBlank(customSerializer.pattern()) && isNotBlank(customSerializer.format())) {
          this.rule.setRule(new RuleItem()
                            .setRegex(customSerializer.pattern())
                            .setFormat(customSerializer.format()));
        }
        return this;
      } catch (Exception e) {
        log.error("json转换处理异常", e);
      }
    }
    return prov.findValueSerializer(property.getType(), property);
  }

  private boolean isNotBlank(String str) {
    return null != str && str.trim().length() > 0;
  }
}
```



### 三、数据脱敏基类

---

```java
@Data
public abstract class BaseRule implements Function<String, String> {
  /**
   * 脱敏规则对象
   */
  private RuleItem rule;

  @Override
  public String apply(String str) {
    if (null == str) {
      return null;
    }
    //初始化脱敏规则
    initRule();
    if (null == rule || null == rule.getRegex() || null == rule.getFormat()) {
      return str;
    }
    //正则替换
    return str.replaceAll(rule.getRegex(), rule.getFormat());
  }
  abstract void initRule();
}
```



### 四、注解参数以及默认处理类

---

```java
@Data
@Accessors(chain = true)
public class RuleItem {

  /**
   * 正则
   */
  private String regex;

  /**
   * 格式化显示
   */
  private String format;
}
```

```java
public class DefaultRule extends BaseRule {
  @Override
  void initRule() {
  }
}
```



### 五、数据脱敏基类实现类

---

#### 5.1 手机号脱敏

---

```java
public class PhoneRule extends BaseRule {
  /**
   * 仅显示前3位和后4位
   */
  @Override
  void initRule() {
    setRule(new RuleItem()
            .setRegex("(\\d{3})\\d*(\\d{4})")
            .setFormat("$1****$2"));
  }
}
```

#### 5.2 身份证号码脱敏

```java
public class IdCardRule extends BaseRule {
  /**
   * 仅显示前6位和后4位
   */
  @Override
  void initRule() {
    setRule(new RuleItem()
            .setRegex("(\\d{6})\\d*(\\w{4})")
            .setFormat("$1********$2"));
  }
}
```

#### 5.3 用户名脱敏

```java
public class UserNameRule extends BaseRule {
  /**
   * 仅显示最后一个汉字
   */
  @Override
  void initRule() {
    setRule(new RuleItem()
            .setRegex("\\S*(\\S)")
            .setFormat("**$1"));
  }
}
```

#### 5.4 实体类属性上面加注解

```java
@CustomSerializer(value = PhoneRule.class)
private String phonenumber;
```

这样子返回的数据，就会脱敏返回。