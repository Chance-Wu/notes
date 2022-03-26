>BeanFactory实现应尽可能支持标准Bean声明周期接口。全套==初始化==方法及其标准顺序为：

```
BeanNameAware's {@code setBeanName}
BeanClassLoaderAware's {@code setBeanClassLoader}
BeanFactoryAware's {@code setBeanFactory}
EnvironmentAware's {@code setEnvironment}
EmbeddedValueResolverAware's {@code setEmbeddedValueResolver}
ResourceLoaderAware's {@code setResourceLoader}

(仅在应用程序上下文中运行时适用)
ApplicationEventPublisherAware's {@code setApplicationEventPublisher}
MessageSourceAware's {@code setMessageSource}
ApplicationContextAware's {@code setApplicationContext}
ServletContextAware's {@code setServletContext}
(only applicable when running in a web application context)
{@code postProcessBeforeInitialization} methods of BeanPostProcessors
InitializingBean's {@code afterPropertiesSet}
a custom init-method definition
{@code postProcessAfterInitialization} methods of BeanPostProcessors

```

>在关闭bean工厂时，以下生命周期方法适用：

```
{@code postProcessBeforeDestruction} methods of DestructionAwareBeanPostProcessors
DisposableBean's {@code destroy}
自定义 destroy-method
```

