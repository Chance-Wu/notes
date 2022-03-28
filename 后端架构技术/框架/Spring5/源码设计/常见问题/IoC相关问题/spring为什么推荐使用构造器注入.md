#### 1. 前言

>spring的核心特性就是IoC（控制反转）和AOP。我们需要将组件交由spring的IoC管理，将对象的依赖关系由spring控制，避免硬编码所造成的过程过渡耦合。
>
>为什么要使用构造器的注入方式？

#### 2. 常见的三种注入方式(注解方式)

##### 2.1 属性注入

>```java
>@Controller
>public class FooController {
>    @Autowired
>    private FooService fooService;
>
>    public List<Foo> listFoo() {
>        return fooService.list();
>    }
>}
>```
>
>注入方式简单：加入要注入的字段，加上@Autowired注解（默认使用类型注入）。

##### 2.2 构造器注入

>```java
>@Controller
>public class FooController {
>
>    private final FooService fooService;
>
>    @Autowired
>    public FooController(FooService fooService) {
>        this.fooService = fooService;
>    }
>}
>```
>
>在Spring4.x版本中推荐的注入方式就是这种。

##### 2.3 set方法注入

>```java
>@Controller
>public class FooController {
>
>    private FooService fooService;
>
>    @Autowired
>    public void setFooService(FooService fooService) {
>        this.fooService = fooService;
>    }
>}
>```
>
>在Spring3.x刚推出的时候，推荐使用注入的就是这种。

#### 3. 构造器注入的好处

>构造器注入的方式，能够保证注入的==组件不可变==，并且确保需要的==依赖不为空==。此外，构造器注入的依赖总是能够在返回客户端（组件）代码的时候保证==完全初始化的状态==。
>
>- 依赖不可变：其实说的就是final关键字
>- 依赖不为空（省去了为空的检查）：当要实例化FooController的时候，由于自己实现了构造函数，所以不会调用默认构造函数，那么就需要spring容器传入所需要的参数，
>- 完全初始化的状态

#### 4. field注入的缺点

>- 对于IOC容器以外的环境，除了使用反射来提供它需要的依赖之外，无法复用该实现类。而且将一直是个潜在的隐患，因为你不调用将一直无法发现NPE的存在。
>- 还值得一提另外一点是：使用field注入可能会导致循环依赖，即A里面注入B，B里面又注入A。

