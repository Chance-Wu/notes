`ContextualDeserializer` 和 `ContextualSerializer` 是jackson提供的，当我们需要根据编码解码的字段类型决定解码器编码器的时候，利用ContextualDeserializer和ContextualSerializer 提供的方式，**实现替代的作用**。



### 一、ContextualSerializer使用场景

---

注解结合 `继承 JsonSerialize 实现 ContextualSerializer`，通过注解显式地声明序列化方式，实现返回的对象进行转译。

`@JacksonAnnotationsInside`： jackson元注解之一，一般用于将其他的注解一起打包成"组合"注解。

`@JsonSerialize`：**自定义序列化**。

`ContextualSerializer`：JsonSerializer可以实现的附加接口，以获取回调，该回调可用于创建序列化程序的上下文实例，以用于处理受支持类型的属性。这对于可以通过注释配置的序列化程序很有用，或者根据要序列化的属性类型，序列化程序应该具有不同的行为







































