BeanFactory接口：

```java
/**
 * 用于访问Spring bean容器的根接口。
 * 这是bean容器的基本客户端视图。
 */
public interface BeanFactory {

	/**
	 * 与由 FactoryBean 创建的bean区别开。
	 * 例如，名为 myJndiObject 的bean是FactoryBean，则获取 ＆myJndiObject 将返回工厂，而不是工厂返回的实例。
	 */
	String FACTORY_BEAN_PREFIX = "&";

	/**
	 * 返回一个实例，该实例可以是指定bean的共享或独立的
	 */
	Object getBean(String name) throws BeansException;

    ...省略代码
    
	/**
	 * 这个bean工厂是否包含给定名称的bean定义或外部注册的单例实例？
	 */
	boolean containsBean(String name);

	/**
	 * 该bean是共享单例吗？也就是说，getBean()是否总是返回相同的实例？
	 */
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

	/**
	 * 这个bean是原型吗？getBean()总是返回独立实例？
	 */
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

	/**
	 * 检查具有给定名称的Bean是否与指定的类型匹配。
	 */
	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

	/**
	 * 同上
	 */
	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

	/**
	 * 获取给定名称的bean的类型。
	 */
	Class<?> getType(String name) throws NoSuchBeanDefinitionException;

	/**
	 * 返回给定bean名称的别名（如果有）
	 */
	String[] getAliases(String name);

}
```

Spring容器的根接口BeanFactory里的一个成员变量：

`String FACTORY_BEAN_PREFIX = "&";`

- BeanFactory首先是Factory，是Spring容器的最顶级的接口。
- FactoryBean是一个接口，用以生成Bean的工厂Bean
- BeanFactory 管理的全局的所有的Bean，而 FactoryBean 只针对一个。

`FACTORY_BEAN_PREFIX`作用是如果在使用beanName获取Bean时，在BeanName前添加这个前缀，那么使用这个BeanName获得的Bean实例是其所在FactoryBean的实例，也就是实现 FactoryBean 接口的那个类的Bean实例。

