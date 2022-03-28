>`XmlBeanFactory`继承自`DefaultListableBeanFactory`，而`DefaultListableBeanFactory`是整个bean加载的核心部分，是Spring注册及加载bean的默认实现，而对于`XmlBeanFactory`与`DefaultListableBeanFactory`不同的地方其实是在`XmlBeanFactory`中使用了自定义的XML读取器`XmlBeanDefinitionReader`，实现了个性化的`BeanDefinitionReader`读取。

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gmxng8sssdj31xo0u0q58.jpg" style="zoom:60%">

>- `AliasRegistry`：定义对alias的简单增删改等操作。
>- `SimpleAliasRegistry`：主要使用map作为alias的缓存，并对接口AliasRegistry进行实现。
>- `SingletonBeanRegistry`：定义对单例的注册及获取。
>- `BeanFactory`：定义获取bean及bean的各种属性。
>- `DefaultSingletonBeanRegistry`：对接口SingletonBeanRegistry各函数的实现。
>- `HierarchicalBeanFactory`：继承BeanFactory，也就是在BeanFactory定义的功能的基础上增加了对parentFactory的支持。
>- `BeanDefinitionRegistry`：定义对BeanDefinition的各种增删改操作。
>- `FactoryBeanRegistrySupport`：在DefaultSingletonBeanRegistry基础上增加了对FactoryBean的特殊处理功能。
>- `ConfigurableBeanFactory`：提供配置Factory的各种方法。
>- `ListableBeanFactory`：根据各种条件获取bean的配置清单。
>- `AbstractBeanFactory`：综合FactoryBeanRegistrySupport和ConfigurableBeanFactory的功能。
>- `AutowireCapableBeanFactory`：提供创建bean、自动注入、初始化以及应用bean的后处理器。
>- `AbstractAutowireCapableBeanFactory`：综合AbstractBeanFactory并对接口AutowireCapableBeanFactory进行实现。
>- `ConfigurableListableBeanFactory`：BeanFactory配置清单，指定忽略类型及接口等。
>- `DefaultListableBeanFactory`：综合上面所有功能，主要是对Bean注册后的处理。
>- `XmlBeanFactory`对`DefaultListableBeanFactory`类进行了扩展，主要用于从XML文档中读取`BeanDefinition`，对于注册及获取Bean都是使用从父类`DefaultListableBeanFactory`继承的方法去实现，而唯独与父类不同的个性化实现就是增加了`XmlBeanDefinitionReader`类型的`reader`属性。在`XmlBeanFactory`中主要使用`reader`属性对资源文件进行读取和注册。







