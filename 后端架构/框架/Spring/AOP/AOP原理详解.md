AOP思想：==横向重复，纵向抽取==

#### 1. AOP简介

>- Aspect Orient Programming——面向切面编程，是面向对象编程OOP的一种补充。
>- 面向对象编程是从静态角度考虑程序的结构，而面向切面编程是从动态角度考虑程序运行过程。
>- AOP底层就是采用动态代理模式实现的。采用了两种代理：
>  - ==JDK的动态代理==（*<u>被代理对象必须要实现接口</u>*，才能产生代理对象）
>  - ==CGLIB的动态代理==（可以对任何类生成代理，代理的原理是*<u>对目标对象的继承代理</u>*，如果目标对象被final修饰，那么该类型无法被cglib代理）

>装成切面，*<u>利用AOP容器的功能将切面织入到主业务逻辑中</u>*。所谓的交叉业务逻辑是指，通用的、与主业务逻辑无关的代码，如安全检查、事物、日志等。
>
>若不使用AOP，则会出现代码纠缠，即交叉逻辑与主业务逻辑或者在一起。这样，会使主业务逻辑变得混杂不清。

#### 2. 面向切面编程术语

1. 切面（Aspect）

   >切面泛指交叉业务逻辑。事务处理、日志处理就可以理解为切面。常用的切面有通知与顾问（实际就是对主业务逻辑的一种增强）。

2. 织入（Weaving）

   >织入就是将切面代码插入到目标对象的过程。把切面连接到其它的应用程序类型或者对象上，并创建一个被通知的对象。这些可以在编译时（例如使用AspectJ编译器），类加载时和运行时完成。Spring和其他纯Java AOP框架一样，在运行时完成织入。

3. 连接点（JoinPoint）

   >连接点指可以被切面织入的方法。通常业务接口中的方法均为连接点。

4. 切入点（Pointcut）

   >切面具体织入的方法。被标记为final的方法是不能作为连接点与切入点的。因为final是不能被修改，不能被增强的。

5. 目标对象（Target）

   >要被增强的对象。

#### 3. 通知（Advice）

>通知是切面的一种实现，可以完成简单织入功能。换个角度说，==通知定了增强代码切入到目标代码的时间点==，是目标方法之前执行，还是之后执行等。通知类型不同，切入的时间不同。
>
>常用通知：
>
>- 前置通知
>- 后置通知
>- 环绕通知
>- 异常处理通知

##### 3.1 通知的用法步骤

>（1）定义目标类
>
>定义将被增强的Bean类。

>（2）定义通知类
>
>通知类是指，实现了相应通知类型接口的类。实现了接口就要实现这些接口中的方法，而这些方法的执行，则是根据不同类型的通知，其执行时机不同。
>
>- ==前置通知==：在目标方法之前执行
>- ==后置通知==：在目标方法之后执行
>- 环绕通知：在目标方法执行之前与之后均执行
>- 异常处理通知：在目标方法执行过程中，若发生指定异常，则执行通知中的方法

>（3）注册目标类

>（4）注册通知切面
>
>即将自定义的切面类交给容器管理。

>（5）注册代理工厂Bean对象==ProxyFactoryBean==
>
>例如：使用`ProxyFactoryBean`类。代理对象的配置，是与JDK的Proxy代理参数一致的，都需要指定三部分：*<u>目标类</u>*、*<u>接口</u>*、*<u>切面</u>*。
>
>```xml
><bean id="studentServiceProxy" 
>      class="org.springframework.aop.framework.ProxyFactoryBean">
>    <property name="target" ref="studentServiceTarget"/>
>    <property name="interfaces" value="com.chance.service.UserService"/>
>    <property name="interceptorNames" value="myMethodBeforeAdvice"/>
></bean>
>```
>
>如果目标对象没有实现业务接口，则可以不设置接口。此时使用的是CGLIB动态代理。

>（6）客户端访问动态代理对象
>
>客户端访问的是动态代理对象，而非原目标对象。因为==代理对象可以将交叉业务逻辑按照通知类型，动态地织入到目标对象的执行中==。
>
>```java
>UserService userService = (UserService)ac.getBean("studentServiceProxy");
>userService.doSome;
>```

##### 3.2 通知详解

>（1）前置通知 MethodBeforeAdvice
>
>前置通知需要实现`MethodBeforeAdvice`接口。需要实现before()方法。特点：
>
>- 在目标方法执行之前先执行
>- 不改变目标方法的执行流程，前置通知代码*<u>不能阻止目标方法的执行</u>*。
>- 不改变目标方法执行的结果。
>
>（2）后置通知`AfterReturningAdvice`
>
>后置通知需要实现`AfterReturningAdvice`接口。需要实现afterReturning()方法。特点：
>
>- 在目标方法执行之后执行
>- 不改变目标方法的执行流程，后置通知代码不能阻止目标方法执行。
>- 不改变目标方法的执行结果。
>
>（3）环绕通知`MethodInterceptor`
>
>需要实现`MethodInteceptor`接口。
>
>- 环绕通知也叫方法拦截器，可以在目标方法调用之前及之后做处理
>- 可以改变目标方法的返回值，
>- 也可以改变程序执行流程。
>
>（4）异常通知`ThrowsAdvice`
>
>需要实现`ThrowsAdvice`接口，该接口的作用是，在目标方法抛出异常后，根据异常的不同做出相应的处理。当该接口处理完异常后，会简单地将异常再次抛出给目标方法。从形式上看，该接口中没有必须要实现的方法。但，这个接口确实有必须实现的方法afterThrowing()。这个方法重载了四种形式。由于使用时，一般只使用其中一种，若要都定义到接口中，则势必要使程序员在使用时必须要实现这四个方法。这是很麻烦的。所以就将该接口定义为`标识接口`（没有方法的接口）。这四个方法在T和rowsAdvice源码的注释上可以看到：
>
><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1glpmo5fdecj31ry068js5.jpg" style="zoom:60%">
>
>常用形式为`public void afterThrowing(自定义的异常类 ex)`
>
>这些方法的执行是在目标方法执行结束后执行的。

##### 3.3 通知的其他用法

>（1）给目标方法织入多个切面
>
>（2）五节课的CGLIB代理生成
>
>若不存在接口，则ProxyFactoryBean会自动采用CGLIB方式生成动态代理。
>
>（3）有接口的CGLIB代理生成-proxyTargetClass属性
>
>```xml
><bean id="studentServiceProxy" 
>      class="org.springframework.aop.framework.ProxyFactoryBean">
>    <property name="target" ref="studentServiceTarget"/>
>    <property name="interfaces" value="com.chance.service.UserService"/>
>    <property name="interceptorNames" value="myMethodBeforeAdvice"/>
>    <!-- 指定是否对类进行代理 -->
>    <property name="proxyTargetClass" value="true">
></bean>
>```
>
>也可指定optimize（优化）的值为true，强制使用CGLIB代理机制。

#### 4. 顾问（Advisor)

>通知是Spring提供的一种切面。其功能过于简单：==只能将切面织入到目标类的所有目标方法中，无法完成将切面织入到指定目标方法中==。
>
>顾问是Spring提供的另一种切面。其可以完成更为复杂的切面织入功能。

>`PointCutAdvisor`是顾问的一种，可以指定具体的切入点。顾问将通知进行了包装，会根据不同的通知类型，在不同的时间点，将切面织入到不同的切入点。
>
>PointcutAdvisor接口有两个常用的实现类：
>
>- `NameMatchMethodPointcutAdvisor`名称匹配方法切入点顾问
>- `RegexpMethodPointcutAdvisor`正则表达式匹配方法切入点顾问

#### 5. 自动代理生成器

>ProxyFactoryBean代理工具类存在如下缺点：
>
>（1）一个代理对象只能代理一个Bean。
>
>（2）客户端获取Bean时，使用的是代理类的id，而非目标对象的Bean的id。形式上不符合逻辑。
>
>Spring提供了自动代理生成器，用于解决ProxyFactoryBean的问题，常用自动代理器如下：
>
>- 默认advisor自动代理生成器——`DefaultAdvisorAutoProxyCreator`
>- Bean名称自动dialing生成器——`BeanNameAutoProxyCreator`
>
>==自动代理生成器均继承自BeanPostProcessor==。容器中所有Bean的初始化时均会自动自动执行Bean后处理器中的方法，故其无需id属性。所以自动代理生成器的Bean也没有id属性，客户类直接使用目标对象bean的id。

##### 5.1 DefaultAdvisorAutoProxyCreator

>==将所有的目标对象与Advisor自动结合，生成代理对象==。无需给生成器做任何的注入配置。注意，==只能与Advisor配合使用==。

##### 5.2 BeanNameAutoProxyCreator

>为每一个目标对象织入所有匹配的Advisor，不具有选择性，且切面只能是顾问Advisor。而BeanNameAutoProxyCreatorde代理方式是，根据Bean的id，来为符合相应名称的类生成相应代理对象，且切面既可以是顾问也已是通知。

#### 6. AspectJ对AOP的实现

>AspectJ也实现了AOP功能，且其实现方式更为简洁，使用方便，支持注解式开发。所以，==Spring又将AspectJ的对于AOP的实现也引入到了自己的框架中==。

##### 6.1 通知类型

>常用五种：
>
>1. 前置
>2. 后置
>3. 环绕
>4. 异常
>5. 最终：无论程序执行是否正常，该通知都会执行。类似于try.catch中的finally代码块

##### 6.2 切入点表达式

>AspectJ定义了专门的表达式用于指定切入点。
>
>```
>execution([访问权限类型]
>		  返回值类型
>		  [全限定性类名]
>		  方法名(参数名)
>		  [抛出异常类型]
>)
>```
>
>切入点表达式要匹配的对象就是目标方法的方法名。表达式中加[]的部分表示可省略部分，各部分用空格分开。在其中可以使用以下符号：
>
>| 符号 | 含义                                                         |
>| ---- | ------------------------------------------------------------ |
>| *    | 0至多个任意字符                                              |
>| ··   | 用在方法参数中，表示任意多个参数用在包名后，表示当前包及其子包路径 |
>| +    | 用在类名后，表示当前类及其子类；用在接口，表示当前接口及其实现类 |

##### 6.3 AspectJ依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

##### 6.4 AspectJ基于注解的AOP实现

>（1）定义业务接口与实现类
>
>（2）定义切面类
>
>其中定义不同的方法，将作为不同的通知方法。
>
>- 切面类上添加@Aspect注解，指定当前类作为切面
>
>（3）在切面类的方法上添加通知注解
>
>如：`@Before()`、`@AfterReturning`
>
>（4）添加`@EnableAspectJAutoProxy`注解
>
>在定义好切面后，需要通知容器，让容器生成“目标类+切面”的代理对象。这个代理是由容器自动生成的。只需在容器中注册一个基于aspectj的自动代理生成器，其就会自动扫描到@Aspect注解，并按照通知类型与切入点，将其织入，并生成代理。
>
>底层是由`AspectJAutoProxyRegistrar`实现的。原理是通过扫描找到@Aspect定义的切面类，再由切面类根据切入点找到目标类的目标方法，再由通知类型找到切入的时间点。

>@Before前置通知-方法有`JoinPoint`参数
>
>通过该参数可以获取切入点表达式、方法签名、目标对象等==。所有通知方法均可包含该参数。

>@AfterReturning后置通知-注解有`returning`属性
>
>由于在目标方法之后执行，可以获取到目标方法的返回值。该注解的returning属性就是用于指定接收方法返回值的变量名的。所以，被注解为后置通知的方法，除了可以包含JoinPoint参数外，还可以包含用于接收返回值的变量。==该变量最好为Object类型==。

>@Around环绕通知-增强方法有`ProceedingJoinPoint`参数
>
>`ProceedingJoinPoint`参数用于执行目标方法。若目标方法有返回值，则该方法的返回值就是目标方法的返回值。该增强方法实际是拦截了目标方法的执行。

>@AfterThrowing异常通知-注解中有`throwing`属性
>
>throwing属性用于指定所发生的异常类对象。被注解为异常通知的方法可以包含一个Throwable参数，参数名称为throwing属性指定的名称。

##### 6.5 @Pointcut定义切入点

>将@Point注解在一个方法之上，以后所有的execution的value属性值均可使用该方法名作为切入点。这个方法使用private标识，即没有实际作用的方法。

#### 7. 示例

```java
@Aspect
@Component
public class ActionLogAspect {

    @Autowired
    private SysLogDao sysLogDao;

    /**
     * 切点
     */
    @Pointcut("@annotation(com.springboot.annotation.ActionLog)")
    private void pointcut() {
    }

    @Around("pointcut()")
    public void around(ProceedingJoinPoint point) {
        long beginTime = System.currentTimeMillis();
        try {
            // 执行方法
            Object response = point.proceed();
        } catch (Throwable e) {
            e.printStackTrace();
        }
        // 执行时长(毫秒)
        long time = System.currentTimeMillis() - beginTime;
        // 保存日志
        saveLog(point, time);
    }

    private void saveLog(ProceedingJoinPoint joinPoint, long time) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        System.out.println();
        Method method = signature.getMethod();
        SysLog sysLog = new SysLog();
        ActionLog logAnnotation = method.getAnnotation(ActionLog.class);
        if (logAnnotation != null) {
            // 注解上的描述
            sysLog.setOperation(logAnnotation.value());
        }
        // 请求的方法名
        String className = joinPoint.getTarget().getClass().getName();
        String methodName = signature.getName();
        sysLog.setMethod(className + "." + methodName + "()");
        // 请求的方法参数值
        Object[] args = joinPoint.getArgs();
        // 请求的方法参数名称
        LocalVariableTableParameterNameDiscoverer u = new LocalVariableTableParameterNameDiscoverer();
        String[] paramNames = u.getParameterNames(method);
        if (args != null && paramNames != null) {
            String params = "";
            for (int i = 0; i < args.length; i++) {
                params += "  " + paramNames[i] + ": " + args[i];
            }
            sysLog.setParams(params);
        }
        // 获取request
        HttpServletRequest request = HttpContextUtils.getHttpServletRequest();
        // 设置IP地址
        sysLog.setIp(IPUtils.getIpAddr(request));
        // 模拟一个用户名
        sysLog.setUsername("mrbird");
        sysLog.setTime((int) time);
        Date date = new Date();
        sysLog.setCreateTime(date);
        // 保存系统日志
        sysLogDao.saveSysLog(sysLog);
    }
}
```



