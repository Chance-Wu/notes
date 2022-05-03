#### bean文件中注入的是实现类，但是通过getBean()取出的时候要强转成接口类，而不是实现类？

---

这个问题应该和spring中配置的代理模式相关，即到底是使用JDK动态代理还是Cglib代理。

- JDK动态代理的代理对象不需要实现接口，但是目标对象一定要实现接口；
- 如果目标对象只是一个单独的对象，并没有实现任何的接口，这个时候就可以使用Cglib代理；

使用Cglib代理的时候，通过getBean()取出的注入对象既可以是普通对象，也可以是接口，通过JDK动态代理就只能使用接口。而一般代码习惯中使用JDK动态代理还是更常见的。

别的可能原因：

一般在使用多态的时候都是这样用的：

UserService userService = new UserServiceImpl();

//将一个接口对象实例化成一个它的实现类对象。

那么在spring中为什么强转成了接口对象而不是子类对象呢：

`UserService userService = (UserService) ctx.getBean("userService");`

这是因为在applicationContext.xml容器加载时已经对id为“userService”的bean进行了实例化。而这个对象就是实现类对象的实例。而在后台代码中我们只是将他取出来而不是再一次实例化。这一点从getBean这个方法名中就能看出。因此我们其实是将一个实现类的实例对象强转成了其父类接口的对象，因此多态在这里能够正常运行。

UserService userService = (UserService) ctx.getBean("userService"); 

 

#### 为什么通过JDK动态代理就只能使用接口？

---

如果只是单纯注入是可以用实现类接收注入对象的，但是往往开发中会对实现类做增强，如事务，日志等，实现增强的AOP技术是通过JDK动态代理实现的，而JDK动态代理对实现类对象做增强得到的增强类与实现类是兄弟关系，所以不能用实现类接收增强类对象，只能用接口接收。

Cglib代理类和实现类之间是父子关系，自然可以用实现类去接收代理类对象，但是一般这样做是没有意义的。

让Spring强制使用Cglib代理：

`<aop:aspectj-autoproxy proxy-target-class="true"/>`

