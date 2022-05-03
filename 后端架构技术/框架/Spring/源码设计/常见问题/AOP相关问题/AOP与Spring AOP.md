>aop——编程需要实现的目标。是面向对象编程的一种补充，主要用于处理一些==具有横切性质的系统服务==，如日志收集、事务管理、安全检查、缓存、对象池管理等。
>
>当我们需要==为分散的对象引入公共行为==的时候，OOP显得无能为力。即OOP允许你定义从上到下的关系，但并不适合定义从左到右的关系。例如日志功能，日志代码往往水平地散布在所有对象层次中，而与它所散布到的对象的核心功能毫无关系。==这种散布在各处的无关的代码被称为横切（cross-cutting）代码==，在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。
>
>- 面向对象解决了层次的问题，比如下级继承上级来增强上级等等。
>- AOP解决了左右级问题，比如分别给一下不同的对象的某些方法加入不同的额外功能等等。
>
>spring aop——aop的实现。
>
>AspectJ同样是aop的实现手段。

#### 1. spring aop使用场景分析

>- 日志记录
>- 异常处理
>- 事务管理
>- 权限验证
>- 效率检查

#### 2. Spring AOP概念

>1. **Aspect（切面）**：切面是一个横切关注点的模块化。一个切面能够包含同一个类型的不同增强方法，比如说事务处理和日志处理可以理解为两个切面。==切面由切入点和通知组成，它既包含了横切逻辑的定义，也包括了切入点的定义==。
>
>   ```java
>   @Aspect
>   @Component
>   public class LogAspect { // 日志切面
>   
>   }
>   ```
>
>2. **Join point（连接点）**：程序执行过程中的一点。在Spring AOP中，==连接点始终代表方法的执行==。连接点由两个信息确定：
>
>   - 方法（表示程序执行点，即在哪个目标方法）
>   - 相对点（表示方位，即目标方法的什么位置，比如调用前，后等）
>
>   ```java
>   @Before("pointCut()")
>   public void log(JoinPoint joinPoint) { //这个JoinPoint参数就是连接点
>   }
>   ```
>
>3. **Pointcut（切入点）**：切入点是对连接点进行拦截的条件定义。切入点表达式如何和链接点匹配是AOP的核心，spring默认使用AspectJ切入点语法。一般认为，所有的方法都可以认为是连接点，但是我们并不希望在所有的方法上都添加通知，而切入点的作用就是提供一组规则来匹配连接点。
>
>   ```java
>   @Pointcut("execution(* com.chance.aop.service..*(..))")
>   public void pointCut() {
>   }
>   ```
>
>4. **Target（目标对象）**：指将要被增强的对象。
>
>5. **Advice（通知）**：==通知是拦截到连接点之后要执行的代码==，包括了”aroud“，”before“，”after“等不同类型的通知。Spring AOP框架以拦截器来实现通知模型，并维护一个以连接点为中心的拦截器链。
>
>6. **Weaving（织入）**：织入是将切面和业务逻辑对象连接起来，并创建通知代理的过程。织入可以在编译时，类加载时和运行时完成。==在编译时进行织入就是静态代理，而在运行时进行织入则是动态代理==。
>
>7. **Advisor（增强器）**：切面的另外一种实现，能够将通知以更为复杂的方式织入到目标对象中，是将通知包装为更复杂切面的装配器。Advisor由切入点和Advice组成。 Advisor这个概念来自于Spring对AOP的支撑，在AspectJ中是没有等价的概念的。Advisor就像是一个小的自包含的切面，这个切面只有一个通知。切面自身通过一个Bean表示，并且必须实现一个默认接口。
>
>   ```java
>   // AbstractPointcutAdvisor是默认接口
>   public class LogAdvisor extends AbstractPointcutAdvisor {
>       private Advice advice; // Advice
>       private Pointcut pointcut; // 切入点
>   
>       @PostConstruct
>       public void init() {
>           // AnnotationMatchingPointcut是依据修饰类和方法的注解进行拦截的切入点。
>           this.pointcut = new AnnotationMatchingPointcut((Class) null, Log.class);
>           // 通知
>           this.advice = new LogMethodInterceptor();
>       }
>   }
>   ```

#### 3. 



































