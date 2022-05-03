##### 如果一个单例的 Bean 依赖一个原型的 Bean，原型的Bean每次获取的是同一个对象吗？

- 如果一个单例的 Bean 依赖一个单例的 Bean，那么被依赖的Bean作用域还是单例的。

- 如果一个单例的 Bean 依赖一个原型的 Bean，原型的Bean每次获取的都是同一个对象。

> 原因：Spring在启动的时候，会实例化所有的Bean，懒加载除外，实例化完毕的Bean，会被Spring缓存起来，等到我们调用Spring容器的API获取Bean的时候，就会检查这个Bean是否已经实例化过，也就是在缓存中查找，如果找到，就直接返回。
>
> 单例（懒加载除外） Bean，在 Spring 启动的时候就已经实例化完成，接下来只会从缓存中读取，而我们单例 Bean 中，不管是依赖了单例 Bean，还是原型 Bean，都是在 Spring 容器启动时实例化好了的，所以，我们看到两次打印 hashCode 的结果是一模一样的。

由于单例bean只实例化了一次，那么它只有一个机会为它的依赖bean去设置属性。

##### 如果想依赖一个原型的Bean，并且想每次调用都返回新的实例怎么办呢？

> 方式一：**实现ApplicationContextAware**接口
>
> 在大多数应用程序场景中，容器中的大多数bean都是单例的。当一个单例bean需要与另一个单例bean协作，或者一个非单例bean需要与另一个非单例bean协作时，通常通过将一个bean定义为另一个bean的属性来处理依赖关系。当bean的生命周期不同时，就会出现问题。假设单例bean A需要使用非单例(原型)bean B，可能是在A上的每个方法调用上。容器只创建单例bean A一次，因此只有一次机会设置属性。容器不能每次需要bean B的新实例时都向bean A提供一个。
>
> 您可以通过==实现ApplicationContextAware接口==，*<u>以及在bean A每次需要bean B实例时对容器发出getBean(“B”)调用来让bean A知道容器</u>*。下面是这种方法的一个例子:
>
> ```java
> @Repository
> public class OrderDao implements ApplicationContextAware {
>  
> 	private ApplicationContext applicationContext;
>  
> 	@Autowired
> 	public IndexDao indexDao;
>  
> 	public void order(){
> 		System.out.println("orderDao:"+this.hashCode());
> 		indexDao = (IndexDao) applicationContext.getBean("indexDao");
> 		System.out.println("indexDao:"+indexDao.hashCode());
> 	}
>  
> 	@Override
> 	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
> 		this.applicationContext = applicationContext;
> 	}
> }
> ```
>
> 这种方式严重依赖于Spring的API，侵入性太强。
>
> 方式二：**使用@Lookup注解**
>
> lookup method注入
>
> 指容器覆盖容器管理bean上的方法，返回容器中另一个已命名bean的查找结果的能力。查找通常涉及原型bean。Spring框架通*<u>过CGLIB 动态地生成覆盖该方法的子类来实现这种方法注入</u>*。
>
> ```java
> 
> @Repository
> public abstract class OrderDao {
>  
> 	@Lookup("indexDao")
> 	public abstract IndexDao getIndexDao();
>  
> 	public void order(){
> 		System.out.println("orderDao:"+this.hashCode());
> 		System.out.println("indexDao:"+getIndexDao().hashCode());
> 	}
> }
> ```

