### 一、意图

---

业务委托模式是一种结构设计模式，它在表示层和业务层之间添加了一个抽象层。可以实现层之间的松散耦合，并封装有关如何定位、连接和与构成应用程序的业务对象交互的知识。



### 二、详解

---

业务委托充当从表示层调用业务对象的适配器。

#### 2.1 参与者

- **Client（客户端/表示层）：** 发起请求，需要访问业务服务的对象。
- **Business Delegate（业务委托类）：** 负责与业务层交互的中间类，封装了业务逻辑和查找服务等操作。
- **Business Service（业务服务）：** 实际提供业务逻辑的组件，例如 EJB、POJO 等。
- **Lookup Service（查找服务）：** 负责查找和定位业务服务的组件，例如 JNDI。

#### 2.2 工作原理

1. 客户端通过业务委托类发起请求。
2. 业务委托类使用查找服务查找并获取相应的业务服务。
3. 业务委托类调用业务服务的方法，执行业务逻辑。
4. 业务服务将结果返回给业务委托类。
5. 业务委托类将结果返回给客户端。



### 三、代码实现

---

#### 3.1 业务服务接口

```java
public interface VideoStreamingService {

    /**
     * 执行视频流处理
     * 该方法负责处理视频流数据，可以包括但不限于视频数据的接收、处理和转发
     * 具体处理逻辑取决于服务的实现，可能涉及视频格式的转换、视频内容的分析等操作
     */
    void doProcessing();
}
```

#### 3.2 业务服务实现

```java
@Slf4j
public class NetflixService implements VideoStreamingService {

    @Override
    public void doProcessing() {
        log.info("NetflixService is now processing");
    }
}
```

```java
@Slf4j
public class YouTubeService implements VideoStreamingService {

    @Override
    public void doProcessing() {
        log.info("YouTubeService is now processing");
    }
}
```

#### 3.3 查找服务

```java
public class BusinessLookup {

    /**
     * Netflix 服务实例，用于访问 Netflix 的功能
     */
    private NetflixService netflixService;

    /**
     * YouTube 服务实例，用于访问 YouTube 的功能
     */
    private YouTubeService youTubeService;

    public void setNetflixService(NetflixService netflixService) {
        this.netflixService = netflixService;
    }

    public void setYouTubeService(YouTubeService youTubeService) {
        this.youTubeService = youTubeService;
    }

    /**
     * 根据电影名称选择视频流服务。
     *
     * @param movie 电影名称
     * @return 返回选定的视频流服务实例
     */
    public VideoStreamingService getBusinessService(String movie) {
        // 根据电影名称是否包含 "die hard" 来选择视频流服务
        if (movie.toLowerCase().contains("die hard")) {
            return netflixService;
        } else {
            return youTubeService;
        }
    }
}
```

#### 3.4 业务委托类

用于封装对业务服务的请求，它通过一个业务查找对象来定位并激活相应的业务服务。

```java
public class BusinessDelegate {

    /**
     * 业务查找对象，用于获取具体的业务服务
     */
    private BusinessLookup businessLookup;

    public void setBusinessLookup(BusinessLookup businessLookup) {
        this.businessLookup = businessLookup;
    }

    /**
     * 播放电影的方法
     * 根据电影名称查找并激活相应的视频流服务进行电影播放
     *
     * @param movie 电影名称，用以确定使用哪个视频流服务
     */
    public void playbackMovie(String movie) {
        // 根据电影名称获取对应的视频流服务实例
        VideoStreamingService videoStreamingService = businessLookup.getBusinessService(movie);
        // 调用视频流服务的处理方法，开始播放电影
        videoStreamingService.doProcessing();
    }
}
```

#### 3.5 客户端

```java
public class MobileClient {

    /**
     * 业务委托对象，用于处理具体的业务逻辑操作。
     */
    private final BusinessDelegate businessDelegate;

    /**
     * 构造函数，初始化 MobileClient 实例。
     *
     * @param businessDelegate 业务委托对象，用于处理具体的业务逻辑操作。
     */
    public MobileClient(BusinessDelegate businessDelegate) {
        this.businessDelegate = businessDelegate;
    }

    /**
     * 发起电影播放请求。
     * 该方法将播放电影的任务委托给业务委托对象，使得客户端可以在不了解具体实现细节的情况下发起电影播放。
     *
     * @param movie 要播放的电影名称。
     */
    public void playbackMovie(String movie) {
        businessDelegate.playbackMovie(movie);
    }
}
```

#### 3.6 测试

```java
// 准备对象实例
// 初始化业务代理和业务查找对象，为后续设置服务做准备
BusinessDelegate businessDelegate = new BusinessDelegate();
BusinessLookup businessLookup = new BusinessLookup();

// 设置具体的业务服务实现
// 分别设置NetflixService和YouTubeService作为具体的业务服务实现
businessLookup.setNetflixService(new NetflixService());
businessLookup.setYouTubeService(new YouTubeService());

// 将业务查找对象设置到业务代理中
// 使得业务代理可以通过业务查找对象找到对应的服务实现
businessDelegate.setBusinessLookup(businessLookup);

// 创建客户端并使用业务代理
// 创建一个移动客户端，并将业务代理传递给它，以便客户端可以通过业务代理访问业务服务
MobileClient client = new MobileClient(businessDelegate);

// 使用客户端播放电影
// 播放两部电影，以演示客户端通过业务代理调用不同服务的能力
client.playbackMovie("Die Hard 2");
client.playbackMovie("Maradona: The Greatest Ever");
```



### 四、优缺点

---

#### 4.1 优点

- 松耦合：降低了表示层和业务层之间的耦合度。
- 简化客户端代码：客户端代码更加简洁，易于维护。
- 提高可维护性和可扩展性：业务层的修改不会影响到表示层。
- 提高性能：可以通过缓存等技术提高性能。

#### 4.2 缺点

- 增加代码的复杂性：引入了额外的委托类。



### 五、适用场景

---

#### 5.1 需要解耦表示层和业务层的应用

最主要的应用场景。当表示层（例如 UI 或 Web 页面）需要访问业务逻辑时，如果直接与业务层交互，会导致表示层和业务层高度耦合。这意味着业务层的任何修改都可能影响到表示层，反之亦然。使用业务委托模式可以在两者之间引入一个中间层——委托类，从而**解耦表示层和业务层**。表示层只需要与委托类交互，而无需知道业务层的具体实现。

示例：一个 Web 应用，用户在页面上提交订单。如果不使用委托模式，处理订单的 Servlet 或 Controller 可能直接调用订单服务。如果订单服务的接口或实现发生变化，Servlet 或 Controller 也需要相应修改。使用委托模式后，Servlet 或 Controller 只需调用订单委托类，订单委托类负责与订单服务交互。这样，即使订单服务发生变化，也只需修改订单委托类，而无需修改 Servlet 或 Controller。

#### 5.2 业务逻辑比较复杂需要封装的应用

当业务逻辑比较复杂时，如果直接在表示层处理这些逻辑，会导致表示层代码过于臃肿，难以维护。使用业务委托模式可以将这些复杂的业务逻辑封装在委托类中，使表示层代码更加简洁，专注于用户界面展示。

示例：一个金融应用，需要进行复杂的风险评估。风险评估的逻辑可能涉及到多个步骤和多个业务服务。使用委托模式可以将这些复杂的逻辑封装在一个风险评估委托类中，表示层只需调用该委托类的一个方法即可完成风险评估，而无需关心其内部的复杂实现。

#### 5.3 需要隐藏业务服务的实现细节的应用

有些情况下，我们可能需要隐藏业务服务的实现细节，例如防止客户端直接访问底层的 EJB 或其他组件。使用业务委托模式可以将这些实现细节隐藏在委托类中，客户端只能通过委托类访问业务服务。

#### 5.4 需要提高性能的应用

委托类可以缓存业务服务的查找结果，避免重复查找，提高性能。例如，如果需要频繁访问同一个业务服务，委托类可以在第一次查找后将该服务缓存起来，后续的请求直接从缓存中获取，而无需再次查找。



### 六、Spring中委托模式实例

---

#### 6.1 DispatcherServlet的请求处理委托

DispatcherServlet 是 Spring MVC 的核心组件，它负责接收所有的 HTTP 请求，并将请求委托给不同的 HandlerMapping、HandlerAdapter 和 ViewResolver 进行处理。这就是典型的委托模式的应用。

```java
// DispatcherServlet 的 doDispatch 方法（简化）
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
  HttpServletRequest processedRequest = request;
  HandlerExecutionChain mappedHandler = null;
  boolean multipartRequestParsed = false;

  WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

  try {
    ModelAndView mv = null;
    Exception dispatchException = null;

    try {
      processedRequest = checkMultipart(request);
      multipartRequestParsed = (processedRequest != request);

      // 1. 根据请求查找 HandlerExecutionChain（包含 Handler 和 Interceptor）
      mappedHandler = getHandler(processedRequest);
      if (mappedHandler == null) {
        noHandlerFound(processedRequest, response);
        return;
      }

      // 2. 获取 HandlerAdapter，用于执行 Handler
      HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

      // Process last-modified header, if supported by the handler.
      String method = request.getMethod();
      boolean isGet = "GET".equals(method);
      if (isGet || "HEAD".equals(method)) {
        long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
        if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
          return;
        }
      }

          // 3. 执行 Handler 前的 Interceptor
      if (!mappedHandler.applyPreHandle(processedRequest, response)) {
        return;
      }

      // 4. 执行 Handler
      mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

      if (asyncManager.isConcurrentHandlingStarted()) {
        return;
      }

      applyDefaultViewName(processedRequest, mv);
          // 5. 执行 Handler 后的 Interceptor
      mappedHandler.applyPostHandle(processedRequest, response, mv);
    }
    catch (Exception ex) {
      dispatchException = ex;
    }
    catch (Throwable err) {
      // As of 4.3, we're processing Errors thrown from handler methods as well,
      // making them available for @ExceptionHandler methods and other scenarios.
      dispatchException = new NestedServletException("Handler dispatch failed", err);
    }
        // 6. 处理 View
    processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
  }

  // ...
}
```

>在这个过程中，DispatcherServlet将请求处理的各个环节委托给了不同的组件：
>
>- **getHandler()：** 委托给 HandlerMapping 查找合适的 Handler。
>- **getHandlerAdapter()：** 委托给 HandlerAdapter 执行 Handler。
>- **applyPreHandle() 和 applyPostHandle()：** 委托给 Interceptor 执行拦截器逻辑。
>- **processDispatchResult()：** 委托给 ViewResolver 解析视图。
>
>通过这种委托方式，Dispatcherservlet将复杂的请求处理流程分解成多个独立的步骤，每个步骤由专门的组件负责，提高了代码的模块化和可维护性。

#### 6.2 BeanDefinitionParserDelegate的Bean定义解析委托

在 Spring 的 XML 配置中，`BeanDefinitionParserDelegate` 负责解析 Bean 的定义。它根据不同的 XML 元素，将解析任务委托给不同的 `BeanDefinitionParser` 实现类。

```java
// BeanDefinitionParserDelegate 的 parseCustomElement 方法（简化）
public BeanDefinition parseCustomElement(Element ele, @Nullable BeanDefinition containingBd) {
  String namespaceUri = getNamespaceURI(ele);
  if (namespaceUri == null) {
    return null;
  }
  NamespaceHandler handler = this.readerContext.getNamespaceHandlerResolver().resolve(namespaceUri);
  if (handler == null) {
    error("Unable to locate Spring NamespaceHandler for XML schema namespace [" + namespaceUri + "]", ele);
    return null;
  }
  return handler.parse(ele, new ParserContext(this.readerContext, this, containingBd));
}
```

BeanDefinitionParserDelegate 将解析任务委托给了 NamespaceHandler，而 NamespaceHandler 又会将具体的解析任务委托给相应的 BeanDefinitionParser。例如，`<context:component-scan>` 元素的解析就委托给了 ComponentScanBeanDefinitionParser。

通过这种委托方式，Spring 可以灵活地扩展 XML 配置的功能，添加自定义的 XML 元素和解析逻辑。

#### 6.3 ApplicationEventMulticaster 的事件广播委托

Spring 的事件机制使用 ApplicationEventMulticaster 将事件广播给所有注册的监听器。ApplicationEventMulticaster 本身并不处理事件，而是将事件委托给不同的 ApplicationListener 进行处理。

```java
@Override
public void multicastEvent(ApplicationEvent event) {
  multicastEvent(event, resolveDefaultEventType(event));
}

@Override
public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
  ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
  Executor executor = getTaskExecutor();
  for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
    if (executor != null) {
      executor.execute(() -> invokeListener(listener, event));
    }
    else {
      invokeListener(listener, event);
    }
  }
}

protected void invokeListener(ApplicationListener<?> listener, ApplicationEvent event) {
  ErrorHandler errorHandler = getErrorHandler();
  if (errorHandler != null) {
    try {
      doInvokeListener(listener, event);
    }
    catch (Throwable err) {
      errorHandler.handleError(err);
    }
  }
  else {
    doInvokeListener(listener, event);
  }
}
```

ApplicationEventMulticaster 将事件委托给了 ApplicationListener 的 onApplicationEvent 方法进行处理。

通过这种委托方式，Spring 可以支持多种类型的事件和监听器，并灵活地处理事件的广播和分发。