#### 1. dubbo官方推荐的异常处理方式

---

dubbo官方文档的[服务化最佳实践](https://dubbo.apache.org/zh/docs/v2.7/user/best-practice/#m-zhdocsv27userbest-practice)中，推荐方式如下：

>- 建议使用异常汇报错误，而不是返回错误码，异常信息能携带更多信息，并且语义更友好。
>- 如果担心性能问题，在必要时，可以通过 override 调异常类的 `fillInStackTrace()` 方法为空方法，使其不拷贝栈信息。
>- ==查询方法不建议抛出 checked 异常==，否则调用方在查询时将过多的 `try...catch`，并且不能进行有效处理。
>- ==服务提供方不应将 DAO 或 SQL 等异常抛给消费方==，应在服务实现中对消费方不关心的异常进行包装，否则可能出现消费方无法反序列化相应异常。



#### 2. dubbo处理异常的逻辑

---

dubbo的异常处理类是com.alibaba.dubbo.rpc.filter.ExceptionFilter 类，归纳下对异常的处理分为下面几类：

1. 如果provider实现了GenericService接口，直接抛出；
2. 如果是checked异常，直接抛出；
3. 在方法签名上有声明，直接抛出；
4. 异常类和接口类在同一jar包里，直接抛出；
5. 是JDK自带的异常，直接抛出；
6. 是Dubbo本身的异常，直接抛出；
7. 否则，包装成RuntimeException抛给客户端。（为了防止客户端反序列化失败）

```java
@Activate(group = CommonConstants.PROVIDER)
public class ExceptionFilter implements Filter, Filter.Listener {
  private Logger logger = LoggerFactory.getLogger(ExceptionFilter.class);

  @Override
  public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
    return invoker.invoke(invocation);
  }

  @Override
  public void onResponse(Result appResponse, Invoker<?> invoker, Invocation invocation) {
    if (appResponse.hasException() && GenericService.class != invoker.getInterface()) {
      try {
        Throwable exception = appResponse.getException();

        // checked异常则直接抛出
        if (!(exception instanceof RuntimeException) && (exception instanceof Exception)) {
          return;
        }
        // 在方法签名上有声明，直接抛出
        try {
          Method method = invoker.getInterface().getMethod(invocation.getMethodName(), invocation.getParameterTypes());
          Class<?>[] exceptionClassses = method.getExceptionTypes();
          for (Class<?> exceptionClass : exceptionClassses) {
            if (exception.getClass().equals(exceptionClass)) {
              return;
            }
          }
        } catch (NoSuchMethodException e) {
          return;
        }

        // 对于未在方法签名中找到的异常，在服务器日志中打印 ERROR 消息。
        logger.error("Got unchecked and undeclared exception which called by " + RpcContext.getContext().getRemoteHost() + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName() + ", exception: " + exception.getClass().getName() + ": " + exception.getMessage(), exception);

        // 异常类和接口类在同一个jar文件中，则直接抛出。
        String serviceFile = ReflectUtils.getCodeBase(invoker.getInterface());
        String exceptionFile = ReflectUtils.getCodeBase(exception.getClass());
        if (serviceFile == null || exceptionFile == null || serviceFile.equals(exceptionFile)) {
          return;
        }
        // JDK异常直接抛出
        String className = exception.getClass().getName();
        if (className.startsWith("java.") || className.startsWith("javax.")) {
          return;
        }
        // dubbo本身的异常直接抛出
        if (exception instanceof RpcException) {
          return;
        }

        // 其余情况，用 RuntimeException 包装并扔回客户端
        appResponse.setException(new RuntimeException(StringUtils.toString(exception)));
      } catch (Throwable e) {
        logger.warn("Fail to ExceptionFilter when called by " + RpcContext.getContext().getRemoteHost() + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName() + ", exception: " + e.getClass().getName() + ": " + e.getMessage(), e);
      }
    }
  }

  @Override
  public void onError(Throwable e, Invoker<?> invoker, Invocation invocation) {
    logger.error("Got unchecked and undeclared exception which called by " + RpcContext.getContext().getRemoteHost() + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName() + ", exception: " + e.getClass().getName() + ": " + e.getMessage(), e);
  }

  public void setLogger(Logger logger) {
    this.logger = logger;
  }
}
```



#### 3. 抛出自定义异常的方式

---

抛出自定义异常就要满足上述前6点中的一个。

- 不继承RuntimeException，以检查时异常抛出（不推荐，正常业务异常应该是运行时异常）

- ==在接口方法上写上throw BusinessException==（不推荐，不符合接口设计原则，且这样是把运行时异常作为检查时异常处理）

  ```java
  public interface DemoService {
    DemoUser getUserInfo(Long userID) throws BusinessException;
  }
  ```

- 把自定义异常类和接口放在同一个包目录下（不推荐，这样绑定了异常类的目录，耦合性变高）

- 改包名，以“java.”或者“javax.”来开头。不推荐，违反了类命名原则。

- 继承Dubbo的RpcException。RpcException也是继承了RuntimeException，所以可以以RuntimeException的方式进行处理。（不推荐，至关于自定义异常属于Dubbo的RpcException，这在程序设计上不合理）

>相对来讲，==自定义异常类和接口放在同一目录下==，以及==继承RpcException==是对于程序侵入性更小的方式。

