>从SpringMVC的配置上，谈一谈组合模式。

>组合模式（Composite）：==将对象组合成树形结构==以表示“部分-整体”的层次结构，组合模式使得用户对单个对象和组合对象的使用具有一致性。
>
>通俗的来说，就是将一系列的对象组合在一个整体的对象中，用户在对这个组合对象操作使用时就能跟操作一个对象一样。
>
><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gm01skr7joj30rc0kymz9.jpg" style="zoom:60%">
>
>Component为组合中的对象声明接口，在适当情况下，实现所有类共有接口的默认行为。声明一个接口用于访问和管理Component的子部件。

>在使用Java注解对SpringMVC进行配置时，通常使用以下方式：
>
>```java
>// 自己的配置通过继承自WebMvcConfigurerAdapter类，重写方法来进行springMVC的配置，这边WebMvcConfigurerAdapter是WebMvcConfigurer的一个适配类，提供了一系列可配置的接口方法
>public class MyConfiguration extends WebMvcConfigurerAdapter {
>
>    @Override
>    public void addFormatters(FormatterRegistry formatterRegistry) {
>        formatterRegistry.addConverter(new MyConverter());
>    }
>
>    @Override
>    public void configureMessageConverters(List<HttpMessageConverter> converters) {
>        converters.add(new MyHttpMessageConverter());
>    }
>
>    // More overridden methods ...
>}
>```
>
>那么SpringMVC是怎么探测到你这些重写的方法并把配置的结果告诉SpringMVC本身的呢？
>
>这边涉及到两个比较重要的部分，`DelegatingWebMvcConfiguration`类和`EnableWebMvc`注解。

#### 1. 组合模式应用——SpringMVC中的Java配置

>@EnableWebMvc 注解：
>
>```java
>@Retention(RetentionPolicy.RUNTIME)
>@Target(ElementType.TYPE)
>@Documented
>@Import(DelegatingWebMvcConfiguration.class)
>public @interface EnableWebMvc {
>}
>```
>
>该注解最主要的作用就是导入`DelegatingWebMvcConfiguration`类：
>
>```java
>@Configuration
>public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
>
>	private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();
>
>
>    // 注入configure集合类
>	@Autowired(required = false)
>	public void setConfigurers(List<WebMvcConfigurer> configurers) {
>		if (!CollectionUtils.isEmpty(configurers)) {
>            // WebMvcConfigurerComposite类的方法
>			this.configurers.addWebMvcConfigurers(configurers);
>		}
>	}
>
>
>	@Override
>	protected void configurePathMatch(PathMatchConfigurer configurer) {
>		this.configurers.configurePathMatch(configurer);
>	}
>
>	@Override
>	protected void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
>		this.configurers.configureContentNegotiation(configurer);
>	}
>
>	@Override
>	protected void configureAsyncSupport(AsyncSupportConfigurer configurer) {
>		this.configurers.configureAsyncSupport(configurer);
>	}
>
>	// 等等一系列方法。详情可以参考WebMvcConfigurationSupport类
>    // 它是springmvc实现Java配置的核心类
>    // 定义了一系列默认的和待提供的配置方法
>    // 并将这些配置告诉springmvc本身
>
>}
>```
>
>DelegatingWebMvcConfiguration类中可以看出，==通过@Autowired注解，自动的导入WebMvcConfigurer类的集合==。这里实际上就完成了对WebMvcConfigurer对象的探测。WebMvcConfigurer是一个接口，springMVC的Java配置类大都源自于它，例如WebMvcConfigurerAdapter。
>
>1. ==我们的配置类继承了WebMvcConfigurerAdapter类，==
>2. ==并添加Configuration注解后，它就能被DelegatingWebMvcConfiguration类探测并使用==。
>
>并且，`DelegatingWebMvcConfiguration`类维护了一个私有的`WebMvcConfigurerComposite`对象，它与WebMvcConfigurer息息相关，可以说它和WebMvcConfigurer一起，通过组合模式，实现了对不同配置对象的管理。
>
>==WebMvcConfigurerComposite体现了组合模式树枝节点用Composite结尾，里面包含了树叶节点，树枝和树叶都实现了相同的抽象类或接口WebMvcConfigurer。==
>
>```java
>class WebMvcConfigurerComposite implements WebMvcConfigurer {
>
>    private final List<WebMvcConfigurer> delegates = new ArrayList<WebMvcConfigurer>();
>
>
>    public void addWebMvcConfigurers(List<WebMvcConfigurer> configurers) {
>        if (!CollectionUtils.isEmpty(configurers)) {
>            this.delegates.addAll(configurers);
>        }
>    }
>
>    @Override
>    public void addInterceptors(InterceptorRegistry registry) {
>        for (WebMvcConfigurer delegate : this.delegates) {
>            delegate.addInterceptors(registry);
>        }
>    }
>
>    // 重写了一系列方法，参考WebMvcConfigurer接口
>}
>```
>
>WebMvcConfigurerComposite中，每个方法的参数的类型都是别的类，并且每一个方法都将这些类配置到WebMvcConfigurer类中。
>
>==例如addInterceptors方法，它将注册的拦截器加入到WebMvcConfigurer中，最终通过WebMvcConfigurationSupport类提供给springMVC==。

>通过了组合模式，springMVC将不同的配置（配置同时也是类本身）整合在了同一个整体类中（也就是WebMvcConfigurer）。
>总而言之，springMVC通过组合模式，使得用户或者说框架本身在进行配置时，就通过操作WebMvcConfigurer类及其衍生类这个整体就行了。

>可以说springMVC把组合模式用在实现Java配置上是很明智和合理的，因为框架的总体配置与相关子配置本来就是整体与部分的关系。
>我们在日常的开发中，如果也找到了这些整体-部分的对应关系，那么使用该设计模式可以很好的简化代码量和解耦合。

