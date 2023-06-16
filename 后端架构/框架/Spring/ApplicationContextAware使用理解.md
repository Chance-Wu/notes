> 在Web应用中，Spring容器通常采用声明式方式配置产生：*<u>开发者只要在web.xml中配置一个Listener，该Listener将会负责初始化Spring容器，MVC框架可以直接调用Spring容器中的Bean，无需访问Spring容器本身</u>*。在这种情况下，容器中的Bean处于容器管理下，*<u>无需主动访问容器</u>*，只需接受容器的依赖注入即可。

> 但在某些特殊的情况下，Bean需要实现某个功能，但该功能必须借助于Spring容器才能实现，此时就必须让该Bean先获取Spring容器，然后借助于Spring容器实现该功能。==为了让Bean获取它所在的Spring容器==，可以让该Bean实现ApplicationContextAware接口。

##### 1.实现ApplicationContextAware接口

```java
@Component
public class SpringContextHolder implements ApplicationContextAware {

    /**声明一个静态变量用来保存应用上下文*/
    private static ApplicationContext applicationContext;

    /**
     * 实现ApplicationContextAware接口，注入Context到静态变量中
     */
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("applicationContext正在初始化，application："+applicationContext);
        SpringContextHolder.applicationContext = applicationContext;
    }

    /**
     * 从静态变量applicationContext中得到Bean，自动转型为所赋值对象的类型
     * @param requiredType
     * @param <T>
     * @return
     */
    public static <T> T getBean(Class<T> requiredType){
        if(context==null){
            System.out.println("applicationContext是空的");
        }else{
            System.out.println("applicationContext不是空的");
        }
        return context.getBean(requiredType);
    }

    /**
     * 获取静态变量中的ApplicationContext
     * @return
     */
    public static ApplicationContext getApplicationContext(){
        return applicationContext;
    }

}
```

> Spring容器会检测容器中的所有Bean，如果发现某个Bean实现了ApplicationContextAware接口，Spring容器会在创建该Bean之后，自动调用该Bean的`setApplicationContextAware(ApplicationContext applicationContext)`方法，调用该方法时，会将容器本身作为参数传给该方法-该方法中的实现部分将Spring传入的参数(容器本身)赋给该类对象的applicationContext实例变量，因此接下来可以通过该applicationContext实例变量来访问容器本身。

##### 2. 应用场景

> 一般的应用场景即是上面的SpringContextHolder。我们可以通过SpringContextHolder类的静态方法SpringContextHolder.getAgetBean(Class<T> requiredType)来获取bean实例。==这使得在一些无法注入的地方我们可以获取到bean（比如工具类中）==。

##### 3. 源码分析

> 跟踪Spring源码，看看具体是怎么执行的。
>
> 进入AbstractApplicationContext的refresh方法：
>
> ```java
> @Override
> public void refresh() throws BeansException, IllegalStateException {
>     synchronized (this.startupShutdownMonitor) {
>         // 准备对此上下文进行刷新
>         prepareRefresh();
> 
>         // 告诉子类刷新内部bean工厂
>         ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
> 
>         // 准备在上下文中使用bean工厂
>         prepareBeanFactory(beanFactory);
> 
>         try {
>             // 允许在上下文子类中对bean工厂进行后处理
>             postProcessBeanFactory(beanFactory);
> 
>             // 调用在上下文中注册为bean的工厂处理器。
>             invokeBeanFactoryPostProcessors(beanFactory);
> 
>      ...    
> }
> ```
>
> 进入`prepareBeanFactory(beanFactory)`
>
> ```java
> protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
>     // 告诉内部bean工厂使用上下文的类加载器等
>     beanFactory.setBeanClassLoader(getClassLoader());
>     beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
>     beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));
> 
>     // 使用上下文回调配置Bean工厂
>     beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
>     beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
>     beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
>     beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
>     beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
>     ...
> }
> ```
>
> 可以看到Spring自动给添加上了一个BeanPostProcessor——ApplicationContextAwareProcessor，再看看ApplicationContextAwareProcessor：
>
> ```java
> class ApplicationContextAwareProcessor implements BeanPostProcessor {
> 
> 	private final ConfigurableApplicationContext applicationContext;
> 
> 	private final StringValueResolver embeddedValueResolver;
> 
> 
> 	/**
> 	 * 为给定上下文创建一个新的ApplicationContextAwareProcessor。
> 	 */
> 	public ApplicationContextAwareProcessor(ConfigurableApplicationContext applicationContext) {
> 		this.applicationContext = applicationContext;
> 		this.embeddedValueResolver = new EmbeddedValueResolver(applicationContext.getBeanFactory());
> 	}
> 
> 
> 	@Override
> 	public Object postProcessBeforeInitialization(final Object bean, String beanName) throws BeansException {
> 		AccessControlContext acc = null;
> 
> 		if (System.getSecurityManager() != null &&
> 				(bean instanceof EnvironmentAware || bean instanceof EmbeddedValueResolverAware ||
> 						bean instanceof ResourceLoaderAware || bean instanceof ApplicationEventPublisherAware ||
> 						bean instanceof MessageSourceAware || bean instanceof ApplicationContextAware)) {
> 			acc = this.applicationContext.getBeanFactory().getAccessControlContext();
> 		}
> 
> 		if (acc != null) {
> 			AccessController.doPrivileged(new PrivilegedAction<Object>() {
> 				@Override
> 				public Object run() {
> 					invokeAwareInterfaces(bean);
> 					return null;
> 				}
> 			}, acc);
> 		}
> 		else {
> 			invokeAwareInterfaces(bean);
> 		}
> 
> 		return bean;
> 	}
> 
> 	private void invokeAwareInterfaces(Object bean) {
> 		if (bean instanceof Aware) {
> 			if (bean instanceof EnvironmentAware) {
> 				((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
> 			}
> 			if (bean instanceof EmbeddedValueResolverAware) {
> 				((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
> 			}
> 			if (bean instanceof ResourceLoaderAware) {
> 				((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
> 			}
> 			if (bean instanceof ApplicationEventPublisherAware) {
> 				((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
> 			}
> 			if (bean instanceof MessageSourceAware) {
> 				((MessageSourceAware) bean).setMessageSource(this.applicationContext);
> 			}
> 			if (bean instanceof ApplicationContextAware) {
> 				((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
> 			}
> 		}
> 	}
> 
> 	@Override
> 	public Object postProcessAfterInitialization(Object bean, String beanName) {
> 		return bean;
> 	}
> 
> }
> ```
>
> ApplicationContextAwareProcessor确实实现了BeanPostProcessor接口，查看`invokeAwareInterfaces(Object bean)`方法，*<u>它会判断bean的类型，如果bean是ApplicationContextAware接口，那么就调用这个对象的setApplicationContext方法</u>*。

> 那么BeanPostProcessor是何时调用的呢？
>
> 代码流程：
>
> - AbstractApplicationContext.refresh -> 
> - AbstractApplicationContext.finishBeanFactoryInitialization -> 
> - ConfigurableListableBeanFactory(DefaultListableBeanFactory).preInstantiateSingletons -> 
> - DefaultListableBeanFactory.getBean -> 
> - DefaultListableBeanFactory.doGetBean -> 
> - DefaultListableBeanFactory.createBean -> 
> - AbstractAutowireCapableBeanFactory.doCreateBean -> 
> - AbstractAutowireCapableBeanFactory.initializeBean
>
> 看下最末尾的AbstractAutowireCapableBeanFactory.initializeBean方法：
>
> ```java
> protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {
>     if (System.getSecurityManager() != null) {
>         AccessController.doPrivileged(new PrivilegedAction<Object>() {
>             @Override
>             public Object run() {
>                 invokeAwareMethods(beanName, bean);
>                 return null;
>             }
>         }, getAccessControlContext());
>     }
>     else {
>         invokeAwareMethods(beanName, bean);
>     }
> 
>     Object wrappedBean = bean;
>     if (mbd == null || !mbd.isSynthetic()) {
>         wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
>     }
> 
>     try {
>         invokeInitMethods(beanName, wrappedBean, mbd);
>     }
>     catch (Throwable ex) {
>         throw new BeanCreationException(
>             (mbd != null ? mbd.getResourceDescription() : null),
>             beanName, "Invocation of init method failed", ex);
>     }
>     if (mbd == null || !mbd.isSynthetic()) {
>         wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
>     }
>     return wrappedBean;
> }
> ```
>
> 其中applyBeanPostProcessorsBeforeInitialization会对该bean对象应用所有的beanPostProcessors：
>
> ```java
> @Override
> public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
>     throws BeansException {
> 
>     Object result = existingBean;
>     for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
>         result = beanProcessor.postProcessBeforeInitialization(result, beanName);
>         if (result == null) {
>             return result;
>         }
>     }
>     return result;
> }
> ```

