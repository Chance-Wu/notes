`ContextualDeserializer` 和 `ContextualSerializer` 是jackson提供的，当我们需要根据编码解码的字段类型决定解码器编码器的时候，利用ContextualDeserializer和ContextualSerializer 提供的方式，**实现替代的作用**。



### 一、ContextualSerializer使用场景

---

注解结合 `继承 JsonSerialize 实现 ContextualSerializer`，通过注解显式地声明序列化方式，实现返回的对象进行转译。

`@JacksonAnnotationsInside`： jackson元注解之一，一般用于将其他的注解一起打包成"组合"注解。

`@JsonSerialize`：**自定义序列化**。

`ContextualSerializer`：JsonSerializer可以实现的附加接口，以获取回调，该回调可用于创建序列化程序的上下文实例，以用于处理受支持类型的属性。这对于可以通过注释配置的序列化程序很有用，或者根据要序列化的属性类型，序列化程序应该具有不同的行为。

1. 当前实现仅针对于接口返回时返回对象转译，返回对象必须是自定义序列化的对象，否则不生效。

   ```java
   @GetMapping("/test")
   public R<Test> test(ZfxtInfo zfxtInfo) {
     Test test = new Test();
     test.setAge("1");
     test.setName("张三");
     test.setSex("0");
     return R.ok(test);
   }	
   
   //注解使用demo
   entity对象：
   @EmbedTrans(dictType = "sys_user_sex", fieldName = "sexText111",dicMetohd = DicTypeEnum.NORMAL_CODE_TRANS_NAME)
     private String sex;
   ```

2. 自定义注解：

   ```java
   
   /**
    * 嵌入式转译注解
    * 生效点：request请求完成返回时，进行序列化过滤，并嵌入实现
    * 失效点： service业务处理层是不会生效的，序列化对象转Json,或者其他格式对象后，也是无法过滤到的
    */
   @JacksonAnnotationsInside
   @JsonSerialize(
     using = DicTransContextualSerializer.class
   )
   @Target({ElementType.METHOD, ElementType.FIELD})
   @Retention(RetentionPolicy.RUNTIME)
   @Documented
   public @interface EmbedTrans {
     /**
      * dic类型
      */
     @NotBlank
     String dictType();
   
     /**
      * 转译输出字段
      */
     String fieldName() default "";
   
     /**
      * 需要调用查询字典的方法
      */
     DicTypeEnum dicMetohd() default DicTypeEnum.NORMAL_CODE_TRANS_NAME;
   }
   ```

3. 实现ContextualSerializer获取回调

   ```java
   public class DicTransContextualSerializer extends JsonSerializer<Objects> implements ContextualSerializer {
     @Override
     public JsonSerializer<?> createContextual(SerializerProvider serializerProvider, BeanProperty beanProperty) throws JsonMappingException {
       if (beanProperty != null) {
         if (Objects.equals(beanProperty.getType().getRawClass(), int.class)
             || Objects.equals(beanProperty.getType().getRawClass(), String.class)) {
           EmbedTrans t = beanProperty.getAnnotation(EmbedTrans.class);
           if (t != null) {
             String beanFieldName = beanProperty.getName();
             if (StringUtils.hasText(t.fieldName())) {
               beanFieldName = t.fieldName();
             }
             DicTypeEnum dicTypeEnum = DicTypeEnum.NORMAL_CODE_TRANS_NAME;
             if (t.dicMetohd() != null) {
               dicTypeEnum = t.dicMetohd();
             }
             if (Objects.equals(beanProperty.getType().getRawClass(), int.class)) {
               return new DicTransIntegerSerializer(t.dictType(), beanFieldName + "Text", dicTypeEnum);
             } else {
               return new DicTransStringSerializer(t.dictType(), beanFieldName + "Text", dicTypeEnum);
             }
           }
         }
         return serializerProvider.findValueSerializer(beanProperty.getType(), beanProperty);
       }
       return serializerProvider.findNullValueSerializer(beanProperty);
     }
   
     @Override
     public void serialize(Objects s, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
   
     }
   ```

4. 自定义序列化对象

   ```java
   public class DicTransIntegerSerializer extends JsonSerializer<Integer> {
     private String dictType;
     private String fieldName;
     private DicTypeEnum dicTypeEnum;
   
     public DicTransIntegerSerializer(String dictType, String fieldName, DicTypeEnum dicTypeEnum) {
       this.dictType = dictType;
       this.fieldName = fieldName;
       this.dicTypeEnum = dicTypeEnum;
     }
   
     public DicTransIntegerSerializer() {
     }
   
     @Override
     public void serialize(Integer integer, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
       jsonGenerator.writeObject(integer);
       jsonGenerator.writeFieldName(fieldName);
   
       String val = String.valueOf(integer);
       String getval = DicUtil.getval(dicTypeEnum, dictType, val);
   
       jsonGenerator.writeString(getval);
     }
   }
   ```