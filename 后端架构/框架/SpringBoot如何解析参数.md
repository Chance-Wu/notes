**以下讨论几种传参方式，看看它们是如何被解析并应用到方法参数上的。**



#### 1. HTTP请求处理流程

---

无论在SpringBoot还是SpringMVC中，一个HTTP请求会被DispatcherServlet类接收，它本质是一个Servlet，因为它继承自HttpServlet。在这里，Spring负责解析请求，匹配到Controller类上的方法，解析参数并执行方法，最后处理返回值并渲染视图。

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gusufwb04aj60x80i60tm02.jpg" style="zoom:80%;" />

解析参数对应上图的目标方法这一步骤。针对不同类型的参数，有不同的解析器。Spring已经帮助我们注册了很多解析器。

org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter#getDefaultArgumentResolvers

```java
/**
	* 返回要使用的参数解析器列表，
	* 包括通过 {@link #setCustomArgumentResolvers} 提供的内置解析器和自定义解析器。
 */
private List<HandlerMethodArgumentResolver> getDefaultArgumentResolvers() {
  List<HandlerMethodArgumentResolver> resolvers = new ArrayList<>(30);

  // 基于注释的参数解析
  resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), false));
  resolvers.add(new RequestParamMapMethodArgumentResolver());
  resolvers.add(new PathVariableMethodArgumentResolver());
  resolvers.add(new PathVariableMapMethodArgumentResolver());
  resolvers.add(new MatrixVariableMethodArgumentResolver());
  resolvers.add(new MatrixVariableMapMethodArgumentResolver());
  resolvers.add(new ServletModelAttributeMethodProcessor(false));
  resolvers.add(new RequestResponseBodyMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
  resolvers.add(new RequestPartMethodArgumentResolver(getMessageConverters(), this.requestResponseBodyAdvice));
  resolvers.add(new RequestHeaderMethodArgumentResolver(getBeanFactory()));
  resolvers.add(new RequestHeaderMapMethodArgumentResolver());
  resolvers.add(new ServletCookieValueMethodArgumentResolver(getBeanFactory()));
  resolvers.add(new ExpressionValueMethodArgumentResolver(getBeanFactory()));
  resolvers.add(new SessionAttributeMethodArgumentResolver());
  resolvers.add(new RequestAttributeMethodArgumentResolver());

  // 基于类型的参数解析
  resolvers.add(new ServletRequestMethodArgumentResolver());
  resolvers.add(new ServletResponseMethodArgumentResolver());
  resolvers.add(new HttpEntityMethodProcessor(getMessageConverters(), this.requestResponseBodyAdvice));
  resolvers.add(new RedirectAttributesMethodArgumentResolver());
  resolvers.add(new ModelMethodProcessor());
  resolvers.add(new MapMethodProcessor());
  resolvers.add(new ErrorsMethodArgumentResolver());
  resolvers.add(new SessionStatusMethodArgumentResolver());
  resolvers.add(new UriComponentsBuilderMethodArgumentResolver());
  if (KotlinDetector.isKotlinPresent()) {
    resolvers.add(new ContinuationHandlerMethodArgumentResolver());
  }

  // 自定义参数
  if (getCustomArgumentResolvers() != null) {
    resolvers.addAll(getCustomArgumentResolvers());
  }

  // 囊括所有
  resolvers.add(new PrincipalMethodArgumentResolver());
  resolvers.add(new RequestParamMethodArgumentResolver(getBeanFactory(), true));
  resolvers.add(new ServletModelAttributeMethodProcessor(true));

  return resolvers;
}
```

>它们有一个共同的接口`HandlerMethodArgumentResolver`。`supportsParameter`用来判断方法参数是否可以被当前解析器解析，如果可以就调用resolveArgument去解析。
>
>```java
>public interface HandlerMethodArgumentResolver {
>
>  // 判断方法参数是否可以被当前解析器解析
>  boolean supportsParameter(MethodParameter parameter);
>
>  // 解析参数
>  @Nullable
>  Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
>                         NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;
>
>}
>```



#### 2. @RequestParam

---

> 在Controller方法中，如果参数标注了`@RequestParam`注解，或者是一个简单数据类型。
>
> 以下请求路径为http://localhost:8080/test1?t1=Jack&t2=Java

```java
@RequestMapping("/test1")
@ResponseBody
public String test1(String t1, @RequestParam(name = "t2",required = false) String t2,HttpServletRequest request){
  logger.info("参数:{},{}",t1,t2);
  return "Java";
}
```

>这里对应的参数解析器是`RequestParamMethodArgumentResolver`。拿到参数名称后，直接从Request中获取值。
>
>```java
>protected Object resolveName(String name, MethodParameter parameter, 
>                             NativeWebRequest request) throws Exception {
>
>  HttpServletRequest servletRequest = request.getNativeRequest(HttpServletRequest.class);
>  //...省略部分代码...
>  if (arg == null) {
>    String[] paramValues = request.getParameterValues(name);
>    if (paramValues != null) {
>      arg = paramValues.length == 1 ? paramValues[0] : paramValues;
>    }
>  }
>  return arg;
>}
>```



#### 3. @RequestBody

---

如果我们需要前端传输更多的参数内容，那么通过一个POST请求，将参数放在Body中传输是更好的方式。当然，比较友好的数据格式当属JSON。可以在Controller方法中可以通过@RequestBody注解来接收它，并自动转换为合适的Java Bean对象。

```java
@ResponseBody
@RequestMapping("/test2")
public String test2(@RequestBody SysUser user){
  logger.info("参数信息:{}",JSONObject.toJSONString(user));
  return "Hello";
}
```

>在没有Spring的情况下，我们考虑一下如何解决这一问题呢？
>
>首先呢，还是要依靠Request对象。对于Body中的数据，我们可以通过`request.getReader()`方法来获取，然后读取字符串，最后通过JSON工具类再转换为合适的Java对象。
>
>```java
>@RequestMapping("/test2")
>@ResponseBody
>public String test2(HttpServletRequest request) throws IOException {
>  BufferedReader reader = request.getReader();
>  StringBuilder builder = new StringBuilder();
>  String line;
>  while ((line = reader.readLine()) != null){
>    builder.append(line);
>  }
>  logger.info("Body数据：{}",builder.toString());
>  SysUser sysUser = JSONObject.parseObject(builder.toString(), SysUser.class);
>  logger.info("转换后的Bean：{}",JSONObject.toJSONString(sysUser));
>  return "Java";
>}
>```
>
>实际场景中，@RequestBody注解的参数会由RequestResponseBodyMethodProcessor类来负责解析。它的解析由父类==AbstractMessageConverterMethodArgumentResolver==负责。整个过程分为三个步骤来看：
>
>1. **获取请求辅助信息（readWithMessageConverters(HttpInputMessage, MethodParameter, Type)）**
>
>   比如HTTP请求的数据格式，上下文Class信息、参数类型Class、HTTP请求方法类型等。
>
>2. **确定消息转换器**
>
>   确定一个消息转换器。消息转换器有很多，它们的共同接口是HttpMessageConverter。在这里，Spring帮我们注册了很多转换器，所以需要循环它们，来确定使用哪一个来做消息转换。如果是JSON数据格式的，会选择`MappingJackson2HttpMessageConverter`来处理。它的构造函数正是指明了这一点。
>
>3. **解析**
>
>   既然确定了消息转换器，那么剩下的事就很简单了。通过Request获取Body，然后调用转换器解析就好了。
>
>   ```java
>   protected <T> Object readWithMessageConverters(NativeWebRequest webRequest, MethodParameter parameter,
>                                                  Type paramType) throws IOException, HttpMediaTypeNotSupportedException, HttpMessageNotReadableException {
>   
>     HttpInputMessage inputMessage = createInputMessage(webRequest);
>     return readWithMessageConverters(inputMessage, parameter, paramType);
>   }
>   ```
>
>   再往下就是Jackson包的内容了，不再深究。实际上主要就是为了寻找两个东西：
>
>   - 方法解析器RequestResponseBodyMethodProcessor
>   - 消息转换器MappingJackson2HttpMessageConverter



#### 4. GET请求参数转换Bean

---

在Controller方法上用Java Bean接收。

```java
@RequestMapping("/test3")
@ResponseBody
public String test3(SysUser user){
  logger.info("参数：{}",JSONObject.toJSONString(user));
  return "Java";
}
```

然后用GET方法请求：http://localhost:8080/test3?id=1001&name=Jack&password=1234&address=北京市海淀区

URL后面的参数名称对应Bean对象里面的属性名称，也可以自动转换。那么，这里它又是怎么做的呢 ？

>首先想到Java的反射机制；其次Java还有一种==内省机制==可以完成这件事。可以获取目标类的属性描述符对象，然后拿到它的Method对象， 通过invoke来设置。
>
>```java
>private void setProperty(Object target,String key,String value) {
>  try {
>    PropertyDescriptor propDesc = new PropertyDescriptor(key, target.getClass());
>    Method method = propDesc.getWriteMethod();
>    method.invoke(target, value);
>  } catch (Exception e) {
>    e.printStackTrace();
>  }
>}
>```
>
>在上面的循环中，可以调用这个方法来实现。
>
>```java
>while (iterator.hasNext()){
>  Map.Entry<String, String[]> next = iterator.next();
>  String key = next.getKey();
>  String value = next.getValue()[0];
>  setProperty(userInfo,key,value);
>}
>```
>
>为什么要说到内省机制呢？因为Spring在处理这件事的时候，最终也是靠它处理的。
>
>简单来说，它是通过`BeanWrapperImpl`来处理的。关于BeanWrapperImpl有个很简单的使用方法：
>
>```java
>SysUser user = new SysUser();
>BeanWrapper wrapper = new BeanWrapperImpl(user.getClass());
>
>wrapper.setPropertyValue("id","20001");
>wrapper.setPropertyValue("name","Jack");
>
>Object instance = wrapper.getWrappedInstance();
>```
>
>wrapper.setPropertyValue最后就会调用到BeanWrapperImpl#BeanPropertyHandler.setValue()方法。它的setValue方法和我们上面的setProperty方法大致相同。
>
>```java
>private class BeanPropertyHandler extends PropertyHandler {
>  //属性描述符
>  private final PropertyDescriptor pd;
>  public void setValue(@Nullable Object value) throws Exception {
>    //获取set方法
>    Method writeMethod = this.pd.getWriteMethod();
>    ReflectionUtils.makeAccessible(writeMethod);
>    //设置
>    writeMethod.invoke(BeanWrapperImpl.this.getWrappedInstance(), value);
>  }
>}
>```
>
>通过以上方式，就完成了GET请求参数到Java Bean对象的自动转换。

>回过头来，再看Spring。Spring中处理这种参数的解析器是`ServletModelAttributeMethodProcessor`。它的解析过程在其父类ModelAttributeMethodProcessor.resolveArgument()方法。整个过程，也可以分为三个步骤来看。
>
>1. 获取目标类的构造函数
>
>   根据参数类型，先生成一个目标类的构造函数，以供后面绑定数据的时候使用。
>
>2. 创建数据绑定器WebDataBinder
>
>   WebDataBinder继承自DataBinder。而DataBinder主要的作用，简言之就是利用BeanWrapper给对象的属性设值。
>
>3. 绑定数据到目标类，并返回
>
>   在这里，又把WebDataBinder转换成ServletRequestDataBinder对象，然后调用它的bind方法。
>
>   接下来将request中的参数转换为MutablePropertyValues pvs对象。接下来就是循环pvs，调用setPropertyValue设置属性。当然了，最后调用的其实就是`BeanWrapperImpl#BeanPropertyHandler.setValue()`。
>
>   ```java
>   //模拟Request参数
>   Map<String,Object> map = new HashMap();
>   map.put("id","1001");
>   map.put("name","Jack");
>   map.put("password","123456");
>   map.put("address","北京市海淀区");
>   
>   //将request对象转换为MutablePropertyValues对象
>   MutablePropertyValues propertyValues = new MutablePropertyValues(map);
>   SysUser sysUser = new SysUser();
>   //创建数据绑定器
>   ServletRequestDataBinder binder = new ServletRequestDataBinder(sysUser);
>   //bind数据
>   binder.bind(propertyValues);
>   System.out.println(JSONObject.toJSONString(sysUser));
>   ```



#### 5. 自定义参数解析器

---

我们说所有的消息解析器都实现了HandlerMethodArgumentResolver接口。我们也可以定义一个参数解析器，让它实现这个接口就好了。

首先，可以定义一个ParametersConverter注解。

```java
@Target({ElementType.TYPE, ElementType.METHOD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface ParametersConverter {

  String name() default "";

  boolean required() default false;

  String defaultValue() default "default";
}
```

然后是实现了HandlerMethodArgumentResolver接口的解析器类。

```java
public class CustomHandlerMethodArgumentResolver implements HandlerMethodArgumentResolver {

  private static final Logger LOGGER = LoggerFactory.getLogger(CustomHandlerMethodArgumentResolver.class);

  public CustomHandlerMethodArgumentResolver() {
  }

  @Override
  public boolean supportsParameter(MethodParameter parameter) {
    return parameter.hasParameterAnnotation(ParametersConverter.class);
  }

  @Override
  public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) throws Exception {
    return this.handleParameterNames(parameter, webRequest);
  }

  private Object handleParameterNames(MethodParameter parameter, NativeWebRequest webRequest) {
    LOGGER.info(">>>>>>>>>> Convert request parameters|start <<<<<<<<<<");
    ParametersConverter annotation = parameter.getParameterAnnotation(ParametersConverter.class);
    String name = annotation.name();
    //从Request中获取参数值
    String param = webRequest.getParameter(name);
    return "HaHa，" + param;
  }
}
```

需要配置一下：

```java
@Configuration
public class PortalConfig implements WebMvcConfigurer {

  @Override
  public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
    resolvers.add(new CustomHandlerMethodArgumentResolver());
  }
}
```



#### 6. 总结

---

无论参数如何变化，参数类型再怎么复杂。它们都是通过HTTP请求发送过来的，那么就可以通过`HttpServletRequest`来获取到一切。Spring做的就是通过注解，尽量适配大部分应用场景。