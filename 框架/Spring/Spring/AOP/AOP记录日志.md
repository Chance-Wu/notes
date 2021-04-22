#### 1. 前言

>借助spring的AOP功能，可以将AOP应用至*<u>全局异常处理</u>*，*<u>全局请求拦截</u>*、*<u>实现日志记录</u>*等。

```xml
<!-- aop依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>

```

#### 2. 自定义注解

>定义一个方法级别的`@ActionLogAspect`注解，用于标准需要监控的方法：
>
>```java
>/**
> * <p>
> * 操作日志注解
> * <p>
> *
> * @author chance
> * @since 2020-12-15
> */
>@Target(ElementType.METHOD)
>@Retention(RetentionPolicy.RUNTIME)
>public @interface ActionLog {
>
>    String value() default "";
>}
>```

#### 3. 配置AOP切面

>aspectj相关注解的作用：
>
>- `@Aspect`：声明该类为一个注解类；
>- `@Pointcut`：
>- 

#### 3. 保存日志的方法

>为了方便，这里直接使用Spring JdbcTemplate来操作数据库。定义一个SysLogDao接口，包含一个保存操作日志的抽象方法：
>
>```java
>public interface SysLogDao {
>    void saveSysLog(SysLog syslog);
>}
>```
>
>实现方法省略。。。

#### 4. 切面和切点

>前提：启动类上加上`@EnableAspectJAutoProxy(proxyTargetClass=true)`注解。
>
>定义一个ActionLogAspect类，==使用`@Aspect`标注让其成为一个切面==，==切点为使用@ActionLog注解标注的方法==，使用@Around环绕通知：
>
>```java
>@Aspect
>@Component
>public class ActionLogAspect {
>
>    @Autowired
>    private SysLogDao sysLogDao;
>
>    /**
>     * 切点
>     */
>    @Pointcut("@annotation(com.springboot.annotation.ActionLog)")
>    private void pointcut() {
>    }
>
>    @Around("pointcut()")
>    public void around(ProceedingJoinPoint point) {
>        long beginTime = System.currentTimeMillis();
>        try {
>            // 执行方法
>            Object response = point.proceed();
>        } catch (Throwable e) {
>            e.printStackTrace();
>        }
>        // 执行时长(毫秒)
>        long time = System.currentTimeMillis() - beginTime;
>        // 保存日志
>        saveLog(point, time);
>    }
>
>    private void saveLog(ProceedingJoinPoint joinPoint, long time) {
>        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
>        System.out.println();
>        Method method = signature.getMethod();
>        SysLog sysLog = new SysLog();
>        ActionLog logAnnotation = method.getAnnotation(ActionLog.class);
>        if (logAnnotation != null) {
>            // 注解上的描述
>            sysLog.setOperation(logAnnotation.value());
>        }
>        // 请求的方法名
>        String className = joinPoint.getTarget().getClass().getName();
>        String methodName = signature.getName();
>        sysLog.setMethod(className + "." + methodName + "()");
>        // 请求的方法参数值
>        Object[] args = joinPoint.getArgs();
>        // 请求的方法参数名称
>        LocalVariableTableParameterNameDiscoverer u = new LocalVariableTableParameterNameDiscoverer();
>        String[] paramNames = u.getParameterNames(method);
>        if (args != null && paramNames != null) {
>            String params = "";
>            for (int i = 0; i < args.length; i++) {
>                params += "  " + paramNames[i] + ": " + args[i];
>            }
>            sysLog.setParams(params);
>        }
>        // 获取request
>        HttpServletRequest request = HttpContextUtils.getHttpServletRequest();
>        // 设置IP地址
>        sysLog.setIp(IPUtils.getIpAddr(request));
>        // 模拟一个用户名
>        sysLog.setUsername("mrbird");
>        sysLog.setTime((int) time);
>        Date date = new Date();
>        sysLog.setCreateTime(date);
>        // 保存系统日志
>        sysLogDao.saveSysLog(sysLog);
>    }
>}
>```

#### 5. 测试

```java
@RestController
public class TestController {

    @ActionLog("执行方法一")
    @GetMapping("/one")
    public void methodOne(String name) {

    }

    @ActionLog("执行方法二")
    @GetMapping("/two")
    public void methodTwo() throws InterruptedException {
        Thread.sleep(2000);
    }

    @ActionLog("执行方法三")
    @GetMapping("/three")
    public void methodThree(String name, String age) {

    }
}
```



