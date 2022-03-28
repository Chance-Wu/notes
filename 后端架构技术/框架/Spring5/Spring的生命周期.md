##### 1. Spring Bean 生命周期

<img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gkywhy1sg2j319u0u0gqx.jpg" style="zoom:60%">

> 1. 在Spring Bean准备使用之前当然是调用构造器,所以Constructor是创建Spring Bean的第一步。
> 2. 通过Setter方法完成依赖注入。
> 3. 依赖注入一旦结束，`BeanNameAware.setBeanName()`会被调用，设置该bean在BeanFactory的名称。
> 4. 接下来调用`BeanClassLoaderAware.setBeanClassLoader() `，为 bean 实例提供类加载器，我们知道所有类都是要通过类加载器加载到上下文的。
> 5. 然后`BeanFactoryAware.setBeanFactory()` 会被调用为 bean 实例提供其所拥有的 factory。

##### 2. Aware说明

> Spring 的依赖注入最大亮点就是所有的 Bean 对 Spring 容器的存在是没有意识的，这时候要让 Bean 主动意识到 Spring 容器的存在，才能调用 Spring 所提供的资源，这就是 Spring Aware。==其实 Spring Aware 是 Spring 设计为框架内部使用的，若使用了，你的 Bean 将会和 Spring 框架耦合==。

| Aware子接口                    | 描述                                        |
| ------------------------------ | ------------------------------------------- |
| BeanNameAware                  | 获取容器中Bean的名称                        |
| BeanFactoryAware               | 获取当前BeanFactory，这样可以调用容器的服务 |
| ApplicationContextAware        | 同上                                        |
| MessageSourceAware             | 获取Message Source相关文本信息              |
| ApplicationEventPublisherAware | 发布事件                                    |
| ResourceLoaderAware            | 获取资源加载器，这样获取外部资源文件        |

例如：公共组件中的SecurityContext（安全上下文）。实现了ApplicationContextAware接口

```java
@Component
public class SecurityContext implements ApplicationContextAware {
	private static ThreadLocal<Client>  clientThreadLocal = new InheritableThreadLocal<>();
	private static ThreadLocal<User> userThreadLocal = new InheritableThreadLocal<>();
	private static ThreadLocal<String> tokenLocal = new InheritableThreadLocal<>();

	private static ApplicationContext applicationContext;
	
	private SecurityContext() {

	}

	public static String getUserName() {
		return getUserPrincipal() != null ? getUserPrincipal().getLoginName() : null;
	}

	public static String getName() {
		return getUserPrincipal() != null ? getUserPrincipal().getName() : null;
	}

	public static User getUserPrincipal() {
		User user = userThreadLocal.get();
		if (null != user) {
			return user;
		}
		String authToken = getToken();
		if (StringUtils.isBlank(authToken)) {
			authToken = getTokenFromRequest();
		}

		if (StringUtils.isBlank(authToken)) {
			return null;
		}
		String key = CommonConstant.REDIS_KEY_SSO_USER + authToken;

		RedisUtils redisUtils = applicationContext.getBean(RedisUtils.class);
		user = (User)redisUtils.get(key);


		if (null != user) {
			userThreadLocal.set(user);
		}
		return user;
	}
	public static Client getClientPrincipal() {
		Client client = clientThreadLocal.get();
		if (null != client) {
			return client;
		}
		String authToken = getToken();

		if (StringUtils.isBlank(authToken)) {
			authToken = getTokenFromRequest();
		}

		if (StringUtils.isBlank(authToken)) {
			return null;
		}
		String key = CommonConstant.REDIS_KEY_SSO_CLIENT + authToken;

		RedisUtils redisUtils = applicationContext.getBean(RedisUtils.class);
		client = (Client)redisUtils.get(key);

		if (null != client) {
			clientThreadLocal.set(client);
		}
		return client;
	}

	public static String getClientName() {
		return getClientPrincipal() != null ? getClientPrincipal().getClientName() : null;
	}

	public static String getClientCode() {
		return getClientPrincipal() != null ? getClientPrincipal().getClientCode() : null;
	}

	public static void setUserPrincipal(User user) {
		userThreadLocal.set(user);
	}
	public static void setClientPrincipal(Client client){clientThreadLocal.set(client);}
	public static String getToken() {
		return tokenLocal.get();
	}

	public static void setToken(String token) {
		tokenLocal.set(token);
	}

	public static void remove() {
		userThreadLocal.remove();
		tokenLocal.remove();
		clientThreadLocal.remove();
	}

	private static String getTokenFromRequest() {
		ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder
				.getRequestAttributes();
		HttpServletRequest request = attributes.getRequest();
		if (null == attributes) return null;

		return request.getHeader("token");
	}

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
	    this.applicationContext = applicationContext;
	}

	public static String getRequestHeader(String name) {
		ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder
				.getRequestAttributes();
		HttpServletRequest request = attributes.getRequest();
		if (null == attributes) return null;

		return request.getHeader(name);
	}
}
```

