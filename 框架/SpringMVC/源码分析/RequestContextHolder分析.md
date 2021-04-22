>`RequestContextHolder`：==持有上下文的Request容器==。

#### 1. 使用

>通过RequestContextHolder的静态方法可以取到当前请求的==request对象==。
>
>```java
>//两个方法在没有使用JSF的项目中是没有区别的
>RequestAttributes requestAttributes = RequestContextHolder.currentRequestAttributes();
>//RequestContextHolder.getRequestAttributes();
>//从session里面获取对应的值
>HttpServletRequest request = ((ServletRequestAttributes)requestAttributes).getRequest();
>HttpServletResponse response = ((ServletRequestAttributes)requestAttributes).getResponse();
>```

#### 2. request和response怎么和当前请求挂钩

>首先分析RequestContextHolder这个类，里面==有两个ThreadLocal保存当前线程下的request==。
>
>```java
>// 得到存储进去的request
>private static final ThreadLocal<RequestAttributes> requestAttributesHolder =
>    new NamedThreadLocal<>("Request attributes");
>
>// 可被子线程继承的request
>private static final ThreadLocal<RequestAttributes> inheritableRequestAttributesHolder =
>    new NamedInheritableThreadLocal<>("Request context");
>```
>
>再看`getRequestAttributes()`方法，相当于直接获取ThreadLocal里面的值，这样就保证了每一次获取到的Request是该请求的request。
>
>```java
>@Nullable
>public static RequestAttributes getRequestAttributes() {
>    RequestAttributes attributes = requestAttributesHolder.get();
>    if (attributes == null) {
>        attributes = inheritableRequestAttributesHolder.get();
>    }
>    return attributes;
>}
>```

#### 3. request和response等是什么时候设置进去的

><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gnz1b8s6vbj31040h8mxw.jpg" style="zoom:80%">
>
>左边1是==Servlet的接口和实现类==。
>
>右边2是使得SpringMVC具有Spring的一些==环境变量和Spring容器==。类似XXXAware接口就是对该类提供Spring感知，简单来说就是如果想使用Spring的XXXX就要实现XXXAware，spring会把需要的东西传过来。
>
>分析以下3个类的源码：
>
>1. `HttpServletBean`进行初始化工作
>2. `FrameworkServlet`初始化WebApplicationContext，并提供了service方法预处理请求
>3. `DispatcherServlet`具体分发处理
>
>可以在FrameworkServlet查看到该类重写了`service()`，`doget()`，`doPost()`...等方法，这些实现里面都有一个预处理方法`processRequest(request,response);`。
>
>具体实现分为3步：
>
>1. 获取上一个请求的参数
>2. 重新建立新的参数
>3. ==设置到RequestContextHolder==
>4. 父类的service()处理请求
>5. 恢复request
>6. 发布事件
>
>```java
>protected final void processRequest(HttpServletRequest request, HttpServletResponse response)
>    throws ServletException, IOException {
>
>    long startTime = System.currentTimeMillis();
>    Throwable failureCause = null;
>
>    // 获取上一个请求保存的LocaleContext
>    LocaleContext previousLocaleContext = LocaleContextHolder.getLocaleContext();
>    // 建立新的LocaleContext
>    LocaleContext localeContext = buildLocaleContext(request);
>
>    // 获取上一个请求保存的RequestAttributes
>    RequestAttributes previousAttributes = RequestContextHolder.getRequestAttributes();
>    // 建立新的RequestAttributes
>    ServletRequestAttributes requestAttributes = buildRequestAttributes(request, response, previousAttributes);
>
>    WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
>    asyncManager.registerCallableInterceptor(FrameworkServlet.class.getName(), new RequestBindingInterceptor());
>
>    // 具体设置的方法
>    initContextHolders(request, localeContext, requestAttributes);
>
>    try {
>        doService(request, response);
>    }
>    catch (ServletException | IOException ex) {
>        failureCause = ex;
>        throw ex;
>    }
>    catch (Throwable ex) {
>        failureCause = ex;
>        throw new NestedServletException("Request processing failed", ex);
>    }
>
>    finally {
>        // 恢复
>        resetContextHolders(request, previousLocaleContext, previousAttributes);
>        if (requestAttributes != null) {
>            requestAttributes.requestCompleted();
>        }
>        logResult(request, response, failureCause, asyncManager);
>        // 发布事件
>        publishRequestHandledEvent(request, response, startTime, failureCause);
>    }
>}
>```
>
>再看`initContextHolders(request, localeContext, requestAttributes)`方法，==把新的RequestAttributes设置进LocalThread==，实际上保存的类型为`ServletRequestAttributes`，这也是为什么在使用的时候可以把RequestAttributes强转为ServletRequestAttributes。
>
>```java
>private void initContextHolders(HttpServletRequest request,
>                                @Nullable LocaleContext localeContext, @Nullable RequestAttributes requestAttributes) {
>
>    if (localeContext != null) {
>        LocaleContextHolder.setLocaleContext(localeContext, this.threadContextInheritable);
>    }
>    if (requestAttributes != null) {
>        RequestContextHolder.setRequestAttributes(requestAttributes, this.threadContextInheritable);
>    }
>}
>```

#### 4. 子线程等异步情况下使用

>我们有的时候会在service层获取request填充一些诸如用户名和IP地址等信息，这个时候如果不想从Controller层传request，可以在service直接使用
>
>```java
>HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
>```
>
>但是，如果service层的函数是异步的话，是获取不到request的。
>
>==通常RequestContextHolder.getRequestAttributes()无法在子线程等异步情况下使用，如果非要获取request里的属性，只能先预置一个函数填充数据后再启动异步函数。==