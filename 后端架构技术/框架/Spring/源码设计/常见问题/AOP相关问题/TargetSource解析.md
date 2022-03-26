>TargetSource是被代理的target实例的来源。
>
>TargetSource被用于获取当前MethodInvocation（方法调用）所需要的target（目标对象），这个target通过反射的方式被调用（如：method.invoke(target,args)）。
>
>`proxy`代理的不是`target`，而是`TargetSource`，这点非常重要！！！

>**为什么SpringAOP代理不直接代理target，而需要通过代理TargetSource（target的来源，其内部持有target），间接代理target呢？**
>
>通常情况下，一个代理对象只能代理一个target，每次方法调用的目标也是唯一固定的target。但是，如果让proxy代理TargetSource，可以使得每次方法==调用的target实例都不同(当然也可以相同，这取决于TargetSource实现)==。这种机制使得方法调用变得灵活，可以扩展出很多高级功能，如：单例，原型，本地线程，目标对象池、运行时目标对象热替换目标源等等。

>TargetSource组件本身与SpringIoC容器无关，即命周期不一定是受spring容器管理的，我们以往的XML中的AOP配置，只是对受容器管理的bean而言的，我们当然可以手动创建一个target，同时使用Spring的AOP框架（而不使用IoC容器）。如下：
>
>```java
>/**
> * 被代理的目标类
> */
>public class TargetBean {
>
>    public static void main(String[] args) {
>        System.out.println("show");
>    }
>}
>
>public class AOPTest {
>
>    @Test
>    public void testAop() {
>        TargetBean target = new TargetBean();
>        SingletonTargetSource targetSource = new SingletonTargetSource(target);
>        // 使用spring aop框架的代理工厂创建代理对象
>        TargetBean proxy = (TargetBean) ProxyFactory.getProxy(targetSource);
>        System.out.println(proxy.getClass().getName());
>    }
>}
>// com.chance.debug.aop.TargetBean$$EnhancerBySpringCGLIB$$1db7be23
>```
>
>这个例子只是创建target的代理对象，并没有添加任何增强逻辑。从输出可以看到目标类已经被CGLIB增强。

#### 1. TargetSource功能定义如下

>```java
>public interface TargetSource extends TargetClassAware {
>
>    /**
>	 * 返回当前目标源的目标类型
>	 * 可以返回null值，如：EmptyTargetSource（未知类会使用这个目标源）
>	 */
>    @Override
>    @Nullable
>    Class<?> getTargetClass();
>
>    /**
>	 * 当前目标源是否是静态的。
>	 * 如果为false，则每次方法调用结束后会调用releaseTarget()释放目标对象；
>	 * 如果为true，则目标对象不可变，也就没必要释放了。
>	 */
>    boolean isStatic();
>
>    /**
>	 * 获取一个目标对象。
>	 * 在每次MethodInvocation方法调用执行之前获取。
>	 */
>    @Nullable
>    Object getTarget() throws Exception;
>
>    /**
>	 * 释放指定的目标对象
>	 */
>    void releaseTarget(Object target) throws Exception;
>
>}
>```

##### 1.1 TargetSource的四个简单实现

>| Target实现               | 描述                                                         |
>| ------------------------ | ------------------------------------------------------------ |
>| EmptyTargetSource        | 静态目标源，当不存在target目标对象，或者甚至连targetClass目标类都不存在时（或未知），使用此类实例。 |
>| HotSwappableTargetSource | 动态目标源，支持热替换的目标源，支持spring应用运行时替换目标对象。 |
>| JndiObjectTargetSource   | spring对JNDI管理bean的支持，static属性可配置。               |
>| SingletonTargetSource    | 静态目标源，单例目标源。Spring的AOP框架默认为受IoC容器管理的bean创建此目标源。换句话说，SingletonTargetSource、proxy与目标bean三者的生命周期均相同。如果bean被配置为prototype，则spring会在每次getBean时创建新的SingletonTargetSource实例。 |

##### 1.2 三大类实现

>| 实现类                               | 描述                                                         |
>| ------------------------------------ | ------------------------------------------------------------ |
>| AbstractBeanFactoryBasedTargetSource | 此类目标源基于IoC容器实现，也就是说target目标对象可以通过beanName从容器中获取。 |
>| AbstractRefreshableTargetSource      | 可刷新的目标源。此类实现可根据配置的刷新延迟时间，在每次获取目标对象时自动刷新目标对象。 |
>| AbstractLazyCreationTargetSource     | 此类实现在调用getTarget()获取时才创建目标对象。              |

#### 2. 总结

>SpringAOP的自动代理（*AutoProxyCreator），只会为受SpringIoC容器管理的bean创建SingletonTargetSource。其他实现类均在手动代理时按需使用。

