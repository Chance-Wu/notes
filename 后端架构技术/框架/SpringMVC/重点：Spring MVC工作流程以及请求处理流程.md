Spring MVC是一个基于请求驱动的Web框架，并且也使用了前端控制器模式来进行设计，再根据请求映射规则分发给相应的页面控制器进行处理。

>主要通过前端控制器controller中的注解来完成请求处理的。前端请求从web.xml中servlet的配置开始，根据servlet拦截的url-pattern，来进行请求转发控制。

#### 1. SpringMVC执行流程

><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gly2qy2amgj310w0megnh.jpg" style="zoom:80%">
>
>1. 浏览器提交请求到中央调度器——`DispatcherServlet`
>2. 中央调度器直接将请求转发给处理器映射器——`HandlerMapping`
>3. 处理器映射器会根据请求，找到处理该请求的处理器，并将其封装为处理器执行链后，返回给中央调度器——`DispatcherServlet`
>4. 中央调度器根据处理器执行链中的处理器，找到能够执行该处理器的处理器适配器——`HandlerAdapter`
>5. 处理器适配器调用执行处理器——`Controller自定义处理器`
>6. 处理器将处理结果及要跳转的视图封装到一个ModelAndView中，并将其返回给处理器适配器
>7. 处理器适配器直接将结果返回给中央调度器
>8. 中央调度器调用视图解析器——`ViewResolver`，将ModelAndView中的视图名称封装为视图对象
>9. 视图解析器将封装了的视图对象返回给中央调度器
>10. 中央调度器调用视图对象——`View`，让其自己进行渲染，即进行数据填充，形成响应对象
>11. 中央调度器响应浏览器
>
><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gly9siluf0j315y0nugn6.jpg" style="zoom:60%">

#### 2. API简要说明

>（1） DispatcherServlet (不需要程序员开发)
>
>中央调度器，也称为前端控制器，在 MVC 架构模式中充当控制器 C，DispatcherServlet是整个流程的控制中心，由它调用诸如处理器映射器、处理器适配器、视图解析器等其它组件处理用户请求。中央调度器的存在降低了组件之间的耦合度。
>
>（2） HandlerMapping (不需要程序员开发)
>
>处理器映射器，负责根据用户请求找到相应的将要执行的 Handler，即处理器。即用于完成将用户请求映射为要处理该请求的处理器，并将处理器封装为处理器执行链传给中央调度器。
>
>（3） HandlAdapter(不需要程序员开发)
>
>处理器适配器，通过 HandlerAdapter 对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。中央调度器会根据不同的处理器自动为处理器选择适配器，以执行处理器。
>
>（4） Handler (需要程序员开发)，即 Controller
>
>处理器，也称为后端控制器，在 DispatcherServlet 的控制下 Handler 调用 Service 层对具体的用户请求进行处理。由于 Handler 涉及到具体的用户业务请求，所以一般情况下需要程序员根据业务需求自己开发 Handler。
>
>（5） ViewResolver (不需要程序员开发)
>
>视图解析器，负责将处理结果生成 View 视图，ViewResolver 首先将逻辑视图名解析为物理视图名，即具体的页面地址，再生成 View 视图对象。最后将处理结果通过页面形式展示给用户。
>
>（6） View (需要程序员开发 jsp 页面) 
>
>SpringMVC 框架提供了很多的 View 视图类型。一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由程序员根据业务需求开发具体的页面。

#### 3. 工作流程

<img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glyvrttl3gj30ws0pa77n.jpg" style="zoom:50%">

>1. ==浏览器发送请求==
>
>2. ==DispatcherServlet接收请求==：首先对对请求进行一个简单判断（判断其为简单请求，还是Multipart请求），然后DispatcherServlet遍历每一个HandlerMapping，将请求交给每一个处理器映射器进行匹配。
>
>3. ==HandlerMapping对请求进行匹配==：对请求进行解析。根据解析结果，找到与请求相对应的处理器对象，并将其包装为处理器执行链 HandlerExecutionChain 对象（包括`Handler对象`以及`Handler对象对应的拦截器`），返回给中央调度器。（处理器映射器顾名思义，就是将请求映射为处理器。）
>
>   ```java
>   mappedHandler = getHandler(processedRequest);
>   ```
>
>   <img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glyvkb89v6j30om07umxi.jpg" style="zoom:60%">
>
>4. ==DispatcherServlet接收HandlerExecutionChain==：根据HandlerExecutionChain中的处理器，查找到与之相应的HandlerAdapter。（当然，此时中央调度器除了找到相应的处理器适配器外，还做了一个工作：执行处理器执行链中的拦截器前端方法。）
>
>   ```java
>   HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
>   ```
>
>   <img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glyvw7bp9bj30u00z7tc6.jpg" style="zoom:50%">
>
>   如果成功获得HandlerAdapter后，此时将开始执行拦截器的`preHandler(...)`
>
>   ```java
>     mappedHandler.applyPreHandle(processedRequest, response)
>         
>         
>         
>     boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
>         HandlerInterceptor[] interceptors = getInterceptors();
>         if (!ObjectUtils.isEmpty(interceptors)) {
>             for (int i = 0; i < interceptors.length; i++) {
>                 HandlerInterceptor interceptor = interceptors[i];
>                 // 调用拦截器的prehandle()方法
>                 if (!interceptor.preHandle(request, response, this.handler)) {
>                     triggerAfterCompletion(request, response, null);
>                     return false;
>                 }
>                 this.interceptorIndex = i;
>             }
>         }
>         return true;
>     }
>   ```
>
>5. ==HandlerAdapter执行Handler==：在执行完HandlerExecutionChain中的拦截器前端方法后，立即调用处理器适配器，让其执行处理器。
>
>6. ==Handler被执行==：提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)。 在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：HttpMessageConveter：将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息数据转换：对请求消息进行数据转换。如String转换成Integer、Double等数据根式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中。Handler执行完成后，向DispatcherServlet 返回一个`ModelAndView对象`；
>
>7. ==DispatcherServlet接收ModelAndView==：接收到处理器适配器发送来的 ModelAndView 后，首先调用执行HandlerExecutionChain中的拦截器后端方法，使后端方法可以修改 ModelAndView。处理器执行链的拦截器后端方法执行完毕后，形成最终的调度结果。中央调度器马上进行调度结果的处理，对处理结果 ModelAndView 进行渲染。而这个渲染的过程，其实是中央调度器遍历所有视图解析器，并根据不同的视图类型由相应的视图解析器形成相应的视图对象的过程。根据返回的ModelAndView，选择一个适合的`ViewResolver`（必须是已经注册到Spring容器中的ViewResolver)返回给DispatcherServlet。
>
>   ```java
>   // 执行处理器执行链中的拦截器后端方法
>   mappedHandler.applyPostHandle(processedRequest, response, mv);
>   ```
>
>8. ==ViewResolver形成View对象==：将视图名称与响应目标定位对象进行绑定，形成View对象返回给DispatcherServlet。
>
>9. ==DispatcherServlet调用View渲染方法渲染View对象==：
>
>   ```java
>   processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
>   
>   // 渲染方法
>   render(mv, request, response);
>   ```
>
>10. View对象进行真正的渲染：
>
>    1. 合并数据 Model；
>    2. 结合视图对象中的响应目标定位对象，准备响应对象 Response；
>    3. 结合合并的数据 Model 与形成的 Response 对象，形成最终的响应视图。`view.render(mv.getModelInternal(), request, response);`
>
>11. ==DispatcherServlet对请求进行响应==：在形成最终的响应视图后，中央调度器执行了收尾工作：执行HandlerExecutionChain的afterCompletion()方法。由 afterCompletion()方法发出对请求的最终响应。
>
>    ```java
>    mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
>    ```
>
>12. 浏览器接收到响应：浏览器接收到由服务端发来的最终的响应 。

#### 4. 示例

##### 4.1 注册中央调度器

```xml
<servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>WEB-INF/springmvc.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>*.do</url-pattern>
</servlet-mapping>
```

>1. 该中央调度器为一个 Servlet，名称为 DispatcherServlet。
>2. 在<servlet/>中添加<Load-on-startup/>的作用是,，标记是否在Web服务器（这里是Tomcat） 启动时会创建这个 Servlet 实例，即是否在 Web 服务器启动时调用执行该 Servlet 的 init()方法，而不是在真正访问时才创建。它的值必须是一个整数。
>   1. 当值大于等于 0 时，表示容器在启动时就加载并初始化这个 servlet，数值越小，该 Servlet的优先级就越高，其被创建的也就越早；
>   2. 当值小于 0 或者没有指定时，则表示该 Servlet 在真正被使用时才会去创建。
>   3. 当值相同时，容器会自己选择创建顺序。
>3. 对于<url-pattern/>，可以写为 / ，建议写为*.do 的形式。

##### 4.2 配置文件位置与名称

>注册完毕后，可直接在服务器上发布运行。此时，默认浏览器页面，及控制台均会抛出 FileNotFoundException 异常。即==默认要从项目根下的 WEB-INF 目录下找名称为'servlet-name'-servlet.xml 的配置文件==。本例配置文件名为 springmvc-servlet.xml。
>
>而一般情况下，该配置文件是放在类路径下，即 src 目录下。所以，在注册中央调度器时，还需要为中央调度器设置查找 SpringMVC 配置文件路径，及文件名。
>
>```xml
><init-param>
>    <param-name>contextConfigLocation</param-name>
>    <param-value>WEB-INF/springmvc.xml</param-value>
></init-param>
>```
>
>打开 DispatcherServlet 的源码，其继承自 FrameworkServlet，而该类中有一个属性contextConfigLocation，用于设置 SpringMVC 配置文件的路径及文件名。该初始化参数的属性就来自于这里。

##### 4.3 创建springmvc.xml文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
</beans>
```

##### 4.4 定义处理器

>该处理器需实现Controller接口
>
>```java
>public class FirstController implements Controller {
>
>    @Override
>    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
>        ModelAndView mv = new ModelAndView();
>        mv.addObject("welcome","Hello SpringMVC World!");
>        mv.setViewName("welcome");
>        return mv;
>    }
>}
>```
>
>ModelAndView中的addObject()方法用于向其model中添加数据。Model 的底层为一个 HashMap。
>
> ==Model 中的数据存储在 request 作用域中==，SringMVC 默认采用转发的方式跳转到视图，本次请求结束，模型中的数据被销毁。

##### 4.5 注册处理器

>在 springmvc.xml 中注册处理器。不过，需要注意==处理器的 id 属性值为一个请求 URI==。表示当客户端提交该请求时，会访问 class 指定的这个处理器。
>
>```xml
><!--注册处理对象
>    id：请求的uri，以 / 开头，表示这是处理器对象
>    class：指定处理器类的全限定名称
>-->
><bean id="/hello.do" class="com.chance.spring.controller.FirstController"/>
>```

##### 4.6 定义目标页面

>在 WEB-INF 目录下新建一个子目录 jsp，在其中新建一个 jsp 页面 welcome.jsp。
>
>```html
><body>
>    服务端信息为：${welcome}
></body>
>```

##### 4.7 注册视图解析器

>SpringMVC 框架为了避免对于请求资源路径与扩展名上的冗余，在视图解析器 InternalResouceViewResolver 中引入了请求的前辍与后辍。而 ==ModelAndView 中只需给出要跳转页面的文件名即可==，对于具体的文件路径与文件扩展名，视图解析器会自动完成拼接。
>
>```xml
><!--注册视图解析器：帮助我们处理视图的路径和扩展名。生成视图对象-->
><!--注册内部资源视图解析器InternalResourceViewResolver-->
><bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
>    <property name="prefix" value="/WEB-INF/jsp/"/>
>    <property name="suffix" value=".jsp"/>
></bean>
>```

##### 4.8 测试

>访问：localhost:8090/hello.do

#### 5. DispatcherServlet 默认配置

>SpringMVC 的程序已经可以正确运行了。但发现一个问题：在流程介绍中所述的重要的处理器映射器、处理器适配器、视图解析器等，都在哪里，不用做配置吗？
>
>当然不是！这些内容均在 ==DispatcherServlet 的默认配置 DispatcherServlet.properties 文件==中被定义。是当 Spring 配置文件中没有指定配置时使用的默认情况。
>
>| 组件名称       | 默认值                                                       |
>| -------------- | ------------------------------------------------------------ |
>| HandlerMapping | BeanNameUrlHandlerMapping<br />DefaultAnnotationHandlerMapping |
>| HandlerAdapter | HttpRequestHandlerAdapter<br />SimpleControllerHandlerAdapter<br />AnnotationMethodHandlerAdapter |
>| ViewResolver   | InternalResourceViewResolver                                 |

#### 6. @RequestMapping定义请求规则

>通过`@RequestMapping`注解可以定义处理器对于请求的映射规则。该注解可以注解在方法上，也可以注解在类上。value 属性值常以“/”开始，也可不加“/”

>队请求提交方式的定义：
>
>@RequestMapping，有一个属性 method，用于对被注解方法所处理请求的提交方式进行限制，即只有满足该 method 属性指定的提交方式的请求，才会执行该被注解方法。
>
>Method 属性的取值为 `RequestMethod` 枚举常量。常用的为 RequestMethod.GET 与RequestMethod.POST。

>对请求中携带参数的定义：
>
>`params`属性中定义了请求中必须携带的参数的要求。以下是几种情况的说明。
>
>- @RequestMapping(value=”/xxx.do”, params={“name”,”age”}) ：要求请求中必须携带请求参数 name 与 age
>
>- @RequestMapping(value=”/xxx.do”, params={“!name”,”age”}) ：要求请求中必须携带请求参数 age，但必须不能携带参数 name
>
>- @RequestMapping(value=”/xxx.do”, params={“name=zs”,”age=23”}) ：要求请求中必须携带请求参数 name，且其值必须为 zs；必须携带参数 age，其其值必须为 23

#### 7. 处理器方法的参数

>处理器方法可以包含以下四类参数，这些参数会在系统调用时由系统自动赋值，即程序员可在方法内直接使用。 
>
>-  `HttpServletRequest` 
>
>-  `HttpServletResponse `
>-  `HttpSession` 
>-  `请求中所携带的请求参数`

##### 7.1 逐个参数接收：

>只要保证请求参数名与该请求处理方法的参数名相同即可。

##### 7.2 请求参数中文乱码问题： 

>对于前面所接收的请求参数，若含有中文，则会出现中文乱码问题。Spring 对于请求参数中的中文乱码问题，给出了专门的字符集过滤器：spring-web-4.3.9.RELEASE.jar 的org.springframework.web.filter 包下的 CharacterEncodingFilter 类。
>
>解决方案：在web.xml中注册字符集过滤器，即可解决Spring的请求参数的中文乱码问题。不过，最好将该过滤器注册在其他过滤器之前。因为过滤器的执行是按照其注册顺序进行的。
>
>```xml
><filter>
>    <!--注册字符集过滤器：解决post请求乱码的问题-->
>    <filter-name>characterEncodingFilter</filter-name>
>    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
>    <!--指定字符集-->
>    <init-param>
>        <param-name>encoding</param-name>
>        <param-value>utf-8</param-value>
>    </init-param>
>    <!--强制request使用字符集encoding-->
>    <init-param>
>        <param-name>forceRequestEncoding</param-name>
>        <param-value>true</param-value>
>    </init-param>
>    <!--强制response使用字符集encoding-->
>    <init-param>
>        <param-name>forceResponseEncoding</param-name>
>        <param-value>true</param-value>
>    </init-param>
></filter>
><filter-mapping>
>    <filter-name>characterEncodingFilter</filter-name>
>    <url-pattern>/*</url-pattern>
></filter-mapping>
>```
>
>`CharacterEncodingFilter`源码分析：
>
><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glyypevkq0j30pu0liq40.jpg" style="zoom:50%">
>
>三个set方法。
>
>字符集设置核心方法：
>
>```java
>protected void doFilterInternal(
>    HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
>    throws ServletException, IOException {
>
>    String encoding = getEncoding();
>    if (encoding != null) {
>        // 强制request使用encoding的字符编码
>        if (isForceRequestEncoding() || request.getCharacterEncoding() == null) {
>            request.setCharacterEncoding(encoding);
>        }
>        // 强制response使用encoding的字符编码
>        if (isForceResponseEncoding()) {
>            response.setCharacterEncoding(encoding);
>        }
>    }
>    filterChain.doFilter(request, response);
>}
>```

##### 7.3 校正请求参数名 @RequestParam

>若请求 URL 所携带的参数名称与处理方法中指定的参数名不相同时，则需在处理方法参数前，添加一个注解@RequestParam(“请求参数名”)，指定请求 URL 所携带参数的名称。该注解是对处理器方法参数进行修饰的。value 属性指定请求参数的名称。

#### 8. 处理器方法的返回值

>使用@Controller 注解的处理器的处理器方法，其返回值常用的有四种类型：
>
>- 第一种：ModelAndView
>
>- 第二种：String
>- 第三种：无返回值 void
>- 第四种：返回自定义类型对象
>
>根据不同的情况，使用不同的返回值。

##### 8.1 返回 ModelAndView

>若处理器方法处理完后，需要跳转到其它资源，且又要在跳转的资源间传递数据，此时处理器方法返回 ModelAndView 比较好。当然，若要返回 ModelAndView，则处理器方法中需要定义 ModelAndView 对象。 
>
>在使用时，若该处理器方法只是进行跳转而不传递数据，或只是传递数据而并不向任何资源跳转（如对页面的 Ajax 异步响应），此时若返回 ModelAndView，则将总是有一部分多余：要么 Model 多余，要么 View 多余。即此时返回 ModelAndView 将不合适。

##### 8.2 返回 String

>==处理器方法返回的字符串可以指定逻辑视图名==，通过视图解析器解析可以将其转换为物理视图地址，返回内部资源逻辑视图名，若要跳转的资源为内部资源，则视图解析器可以使用 InternalResourceViewResolver 内部资源视图解析器。此时处理器方法返回的字符串就是要跳转页面的文件名去掉文件扩展名后的部分。这个字符串与视图解析器中的 prefix、suffix 相结合，即可形成要访问的 URI。

##### 8.3 返回 void

>对于处理器方法返回 void 的应用场景，==AJAX 响应==。 
>
>若处理器对请求处理后，无需跳转到其它任何资源，此时可以让处理器方法返回 void。

##### 8.4 返回对象

>处理器方法也可以返回Object对象。这个Object可以是Integer，String，自定义对象，Map，List等。==返回的对象是作为直接在页面显示的数据出现的==。
>
>==返回对象，需要使用@ResponseBody注解，将转换后的JSON数据放入到响应体中==。



