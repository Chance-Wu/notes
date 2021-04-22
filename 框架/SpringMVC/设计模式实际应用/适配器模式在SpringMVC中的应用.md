>HandlerAdapter使用了适配器模式。
>
>以下是前端控制器`DispatcherServlet`源码：
>
>```java
>public class DispatcherServlet extends FrameworkServlet {
>    //DispatcherServlet类中的核心方法
>    protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
>        HttpServletRequest processedRequest = request;//将request交给HttpServletRequest
>        HandlerExecutionChain mappedHandler = null;
>        boolean multipartRequestParsed = false;
>        …………(省略部分代码)
>            //通过request可以得到请求对应的控制器，即controller/handler
>            mappedHandler = getHandler(processedRequest);
>        …………(省略部分代码)
>            //根据handler得到对应的适配器 
>            HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
>        …………(省略部分代码)
>            //通过适配器去调用对应controller的方法，并返回ModelAndView
>            mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
>    }
>```
>
>适配器接口`HandlerAdapter`的源码：
>
>```java
>@Override
>public final ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
>    throws Exception {
>
>    return handleInternal(request, response, (HandlerMethod) handler);
>}
>```
>
>适配器接口的具体实现类如下：
>
><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glxv5nwp5xj31ds07g3z9.jpg" style="zoom:50%">
>
>其中一个具体的实现类HttpRequestHandlerAdapter的源码，可以看到无非就是重写了接口中的三个方法，其中supports方法就是判断handler是否对应这个适配器实现类，返回一个布尔值；而handle方法就是去调用具体的controller方法，并返回ModelAndView。
>
>```java
>public class HttpRequestHandlerAdapter implements HandlerAdapter {
>
>    @Override
>    public boolean supports(Object handler) {
>        return (handler instanceof HttpRequestHandler);
>    }
>
>    @Override
>    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
>        throws Exception {
>
>        ((HttpRequestHandler) handler).handleRequest(request, response);
>        return null;
>    }
>
>    @Override
>    public long getLastModified(HttpServletRequest request, Object handler) {
>        if (handler instanceof LastModified) {
>            return ((LastModified) handler).getLastModified(request);
>        }
>        return -1L;
>    }
>}
>```
>
>SpringMVC中的HandlerAdapter用到的适配器模式的流程如下：
>
>1. 将request交给HttpServletRequest。
>2. 通过request可以得到请求对应的控制器，或者叫处理器：即controller/handler。
>3. 根据handler得到对应的适配器。
>4. 通过适配器是调用对应controller的方法，并返回ModelAndView（==表面看执行的是适配器的handle方法，实际上适配器执行的就是对应controller的方法==）。

>有了适配器模式，最终是通过具体的适配器类来调用controller中的方法，那么为什么不直接去调用controller中的方法，而要通过适配器来调用呢，这不是多此一举吗？
>
>- 可以看到处理器的类型不同，有多重实现方式，那么调用方式就不是确定的，==如果需要直接调用 Controller 方法，需要调用的时候就得不断是使用 if else 来进行判断是哪一种子类然后执行。那么如果后面要扩展 Controller，就得修改原来的代码，这样违背了 OCP 原则==。
>- 另外，把每个controller的方法差异封装到各个 HandlerAdapter的实现类里，对外只暴露统一的handle方法。
>- 前面在类适配器模式我们提到，由于适配器类继承了被适配器类的方法，因此在适配器类中我们可以会直接调用被适配器类的方法，这样就使得被适配器类的方法在 Adapter 中会暴露出来，增加了使用的成本；而SpringMVC中的HandlerAdapter使用了适配器模式后就解决了这个问题。









