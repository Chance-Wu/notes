```java
/**
 * 定义一个 factory ，该 factory 在调用时可以返回Object实例
 * （可能是共享的或独立的）
 *
 * <p>此接口通常用于封装通用 factory，
 * 该通用工厂在每次调用时返回某个目标对象的新实例（prototype）。
 *
 * <p>这个接口类似于 {@link FactoryBean} ，
 * 但是后者的实现通常被定义为 {@link BeanFactory} 中的SPI实例，
 * 这个类的实现通常是作为API(通过注入)提供给其他bean的。
 */
@FunctionalInterface
public interface ObjectFactory<T> {

  /**
	 * 返回此工厂管理的对象的实例。
	 * （可能是共享的或独立的）
	 */
  T getObject() throws BeansException;

}
```

