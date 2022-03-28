>BeanPostProcessor——后置处理器
>
>spring使用模板模式，在bean的创建过程中安插了==许多锚点==，用户寻找对应的锚点，通过重写方法介入到bean的创建过程当中。通过重写这些锚点，掌握使用==BeanPostProcessor==、获取各类==BeanAware==，并且了解bean的生命周期。

>Bean的声明周期大致分为四个阶段：
>
>1. 实例化
>2. 属性赋值
>3. 初始化
>4. 销毁
>
>主要逻辑都在AbstractAutowireCapableBeanFactory类的doCreateBean()方法中。顺序调用以下三个方法：
>
>1. createBeanInstance()——》实例化阶段
>
>2. populateBean()——》属性赋值阶段
>3. initializeBean()——》初始化阶段
>
>至于销毁，是在容器关闭时调用的。

### 常用扩展点

#### 1 第一大类(影响多个Bean的接口)

>实现了这些接口的Bean会切入到多个Bean的生命周期中。Spring内部扩展也经常使用这些接口，例如自动注入以及AOP的实现都与之有关。
>
>- ==BeanPostProcessor==：用于初始化阶段前后。
>- ==InstantiationAwareBeanPostProcessor==：用于实例化阶段前后。
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gn5tkm7db5j30mv0rtq3d.jpg" style="zoom:60%">
>
>`InstantiationAwareBeanPostProcessor`实际上继承了`BeanPostProcessor`接口，严格意义上来看是父子关系，但是从生命周期角度重点关注其特有的对实例化阶段的影响。

>**postProcessBeforeInstantiation调用点：**
>
>```java
>@Override
>protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
>    throws BeanCreationException {
>
>    try {
>        // postProcessBeforeInstantiation方法调用点，这里就不跟进了，
>        // 实际就是for循环调用所有的InstantiationAwareBeanPostProcessor
>        Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
>        if (bean != null) {
>            return bean;
>        }
>    }
>
>    try {   
>        // 上文提到的doCreateBean方法，可以看到
>        // postProcessBeforeInstantiation方法在创建Bean之前调用
>        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
>        if (logger.isTraceEnabled()) {
>            logger.trace("Finished creating instance of bean '" + beanName + "'");
>        }
>        return beanInstance;
>    }
>
>}
>```
>
>可以看到，postProcessBeforeInstantiation在doCreateBean之前调用，也就是在实例化之前调用。==一般情况下是在postProcessAfterInitialization替换代理类==，自定义了TargetSource的情况下在postProcessBeforeInstantiation替换代理类。具体逻辑在AbstractAutoProxyCreator类中。

>**postProcessAfterInstantiation调用点：**
>
>```java
>protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
>
>    // 方法作为属性赋值的前置检查条件，在属性赋值之前执行，能够影响是否进行属性赋值！
>    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
>        for (BeanPostProcessor bp : getBeanPostProcessors()) {
>            if (bp instanceof InstantiationAwareBeanPostProcessor) {
>                InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
>                if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
>                    return;
>                }
>            }
>        }
>    }
>
>    // 忽略后续的属性赋值操作代码
>}
>```
>
>可以看到该方法在属性赋值方法内，但是在真正执行赋值操作之前。其返回值为boolean，返回false时可以阻断属性赋值阶段（ibp.postProcessAfterInstantiation=false）。

#### 2. 第二大类(只调用一次的接口)

>这一大类接口的特点是功能丰富，==常用于用户自定义扩展==。
>
>1. ==Aware类型的接口==
>2. ==生命周期接口==

##### 2.1 无所不知的Aware

>Aware类型的接口的作用就是让我们能够拿到Spring容器中的一些资源。基本都能给个见名知意，Aware之前的名字就表示可以拿到什么资源。例如BeanNameAware可以拿到BeanName。
>
>==Aware接口调用时机：初始化阶段之前==
>
>Aware接口可以分为两组：
>
>1. `BeanName`Aware
>2. `BeanClassLoader`Aware
>3. `BeanFactory`Aware
>
>1. `Environment`Aware
>2. `EmbeddedValueResolver`Aware，实现该接口能够==获取Spring EL解析器==，用户的自定义注解需要支持spel表达式的时候可以使用。
>3. `ApplicationContext`Aware、`ResourceLoader`Aware、`ApplicationEventPublisher`Aware、`MessageSource`Aware) ，实际上这几个接口可以一起记，其返回值实质上都是当前的ApplicationContext对象，因为ApplicationContext是一个复合接口。如下：
>
>```java
>public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
>MessageSource, ApplicationEventPublisher, ResourcePatternResolver {}
>```

##### 2.2 Aware调用时机源码分析

>初始化时调用Aware接口，如下忽略无关代码：
>
>```java
>protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
>
>    // 这里调用的是BeanNameAware、BeanClassLoaderAware、BeanFactoryAware
>    invokeAwareMethods(beanName, bean);
>
>    Object wrappedBean = bean;
>
>    // 这里调用的是ApplicationContextAware、ResourceLoaderAware、ApplicationEventPublisherAware、MessageSourceAware，
>    // 而实质上这里就是前面所说的BeanPostProcessor的调用点！
>    // 也就是说与前面的Aware不同，这里是通过BeanPostProcessor（ApplicationContextAwareProcessor）实现的。
>    wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
>    // 下文即将介绍的InitializingBean调用点
>    invokeInitMethods(beanName, wrappedBean, mbd);
>    // BeanPostProcessor的另一个调用点
>    wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
>
>    return wrappedBean;
>}
>```
>
>并不是所有的Aware接口都使用同样的方式调用。
>
>- BeanxxAware都是在代码中直接调用的；
>- ApplicationContext相关的Aware都是通过BeanPostProcessor#postProcessBeforeInitialization()实现的。
>
>可以看一下`ApplicationContextAwareProcessor`类的源码，就是判断当前创建的bean是否实现了相关的Aware方法，如果实现了会调用回调方法将资源传递给bean。
>
>==BeanPostProcessor的调用时机：在初始化阶段的前后执行。==

#### 3. 简单的两个生命周期接口

>实例化和属性赋值都是spring帮助我们做的，能够自己实现的有==初始化==和==销毁==两个生命周期阶段。
>
>1. **InitializingBean**：对应生命周期的初始化阶段，通过`invokeInitMethods(beanName, wrappedBean, mbd);`方法中调用。
>
>   有一点需要注意，因为Aware方法都是在执行在初始化方法之前，所以可以在初始化方法中放心大胆的使用Aware接口获取的资源，这也是我们自定义扩展spring的常用方式。
>
>   除了==实现InitializingBean接口==之外还能通过==注解==或者==xml配置==的方式指定初始化方法。
>
>2. **DisposableBean**：对应生命周期的销毁阶段，以ConfigurableApplicationContext#close()方法作为入口，实现是通过循环取所有实现了DisposableBean接口的Bean然后调用其destroy()方法 。

#### 4. 扩展：BeanPostProcessor注册时机与执行顺序

##### 4.1 注册时机

>BeanPostProcessor也会注册为Bean，那么是==如何保证BeanPostProcessor在业务Bean之前初始化完成呢？==
>
>查看`AbstractApplicationContext.java`
>
>```java
>@Override
>public void refresh() throws BeansException, IllegalStateException {
>    synchronized (this.startupShutdownMonitor) {
>        // 刷新前的预处理
>        prepareRefresh();
>
>        // 创建DefaultListableBeanFactory，用来加载BeanDefinition
>        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
>
>        // 准备 bean工厂
>        prepareBeanFactory(beanFactory);
>
>        try {
>            // 在上下文子类中对bean工厂进行后处理
>            postProcessBeanFactory(beanFactory);
>
>            // 调用在上下文中注册为bean的 工厂处理器
>            invokeBeanFactoryPostProcessors(beanFactory);
>
>            // 注册拦截bean创建的 BeanPostProcessor
>            registerBeanPostProcessors(beanFactory);
>
>            // 为此上下文初始化消息源
>            initMessageSource();
>
>            // 为此上下文初始化事件多播器
>            initApplicationEventMulticaster();
>
>            // 在特定上下文子类中初始化其他特殊bean
>            onRefresh();
>
>            // 检查侦听器bean并注册它们
>            registerListeners();
>
>            // 实例化所有剩余的单例（非延迟初始化）
>            finishBeanFactoryInitialization(beanFactory);
>
>            // 最后一步：发布相应的事件
>            finishRefresh();
>        }
>
>        catch (BeansException ex) {
>            if (logger.isWarnEnabled()) {
>                logger.warn("Exception encountered during context initialization - " +
>                            "cancelling refresh attempt: " + ex);
>            }
>
>            // 销毁已创建的单例以避免资源悬空。
>            destroyBeans();
>
>            // 重置active标志设置为false
>            cancelRefresh(ex);
>
>            // 将异常传播给调用者
>            throw ex;
>        }
>
>        finally {
>            // 在Spring的核心中重置常见的自省缓存，
>            // 因为我们可能不再需要单例bean的元数据...
>            resetCommonCaches();
>        }
>    }
>}
>```
>
>可以看出，Spring是先执行`registerBeanPostProcessor()`进行BeanPostProcessors的注册，然后再执行`finishBeanFactoryInitialization()`初始化我们的单例非懒加载的Bean。

##### 4.2 执行顺序

>BeanPostProcessor有很多个，而且每个BeanPostProcessor都影响多个Bean，其执行顺序至关重要，必须能够控制其执行顺序才行。关于执行顺序这里需要引用两个排序相关的接口：
>
>- PriorityOrdered是一等公民，首先被执行，PriorityOrdered公民之间通过接口返回值排序
>- Ordered是二等公民，然后执行，Ordered公民之间通过接口返回值排序
>- 都没有实现是三等公民，最后执行。
>
>可以查看registerBeanPostProcessors()方法，其中有注册各种类型BeanPostProcessor的逻辑，根据实现不同排序接口进行分组。优先级高的先加入，优先级低的后加入。
>
>```java
>/**
> * 用于最高优先级值的有用常量
> * @see java.lang.Integer#MIN_VALUE
> */
>int HIGHEST_PRECEDENCE = Integer.MIN_VALUE;
>
>/**
> * 用于最低优先级值的有用常量
> * @see java.lang.Integer#MAX_VALUE
> */
>int LOWEST_PRECEDENCE = Integer.MAX_VALUE;
>```
>
>根据排序接口返回值排序，默认升序排序，==返回值越低优先级越高==。

### 总结

>Spring Bean的生命周期分为==四个阶段==和==多个扩展点==。扩展点又可以分为影响多个Bean和影响单个Bean。
>
>**四个阶段：**
>
>- 实例化
>- 属性赋值
>- 初始化
>- 销毁
>
>**多个扩展点：**
>
>- 影响多个Bean
>  - BeanPostProcessor
>  - InstantiationAwareBeanPostProcessor
>- 影响单个Bean（Aware接口）
>  - 第一组：BeanNameAware、BeanClassLoaderAware、BeanFactoryAware
>  - 第二组：EnvironmentAware、EmbeddedValueResolverAware、ApplicationContextAware(ResourceLoaderAware/ApplicationEventPublisherAware/MessageSourceAware)
>
>**生命周期：**
>
>- InitializingBean
>- DisposableBean

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gn76g6g649j30tw1r0q5t.jpg" style="zoom:50%">

