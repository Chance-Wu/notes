控制反转IoC又叫依赖注入(DI)。==描述了对象的定义和依赖的一个过程==。依赖的对象通过*<u>构造参数</u>*、*<u>工厂方法参数</u>*或者*<u>属性注入</u>*，当对象实例化后依赖的对象才被创建，当创建bean后容器注入这些依赖对象。这个过程基本上是反向的，因此命名为控制反转(IoC)，它==通过直接使用构造类来控制实例化==，或者定义它们之间的依赖关系，或者类似于服务定位模式的一种机制。 

> Spring提供了两种类型的IOC容器实现。
>
> - **BeanFacotry接口**：IOC容器的基本实现。
> - **ApplicationContext接口**：提供了更多的高级特性，是BeanFactory的子接口。

#### 1. 容器概述

> `org.springframework.context.ApplicationContext`接口代表了Spring Ioc容器。
>
> - 它负责*<u>实例化</u>*、*<u>配置</u>*、*<u>组装Bean</u>*。
> - 容器通过读取配置元数据获取对象的实例化、配置和组装的描述信息。
> - 它配置的元数据用*<u>xml</u>*、<u>*Java注解*</u>或*<u>Java代码</u>*表示。 
>
> Spring提供几个开箱即用的 `ApplicationContext接口`的实现类。在独立应用程序中通常创建一 个 `ClassPathXmlApplicationContext`或`FileSystemXmlApplicationContext`实例对象。 
>
> *<u>在大多数应用场景中，用户不需要显式实例化一个或多个Spring IoC容器的实例</u>*。比如在web应用场景中，在web.xml中简单的8行（或多点）样板式的xml配置文件就可以搞定（参见第3.15.4 节“Web应用程序的便利的ApplicationContext实例化”）。 
>
> ```xml
> <?xml version="1.0" encoding="UTF-8"?>
> <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
>       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>       xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
>       version="4.0">
>  
>  <context-param>
>      <param-name>contextConfigLocation</param-name>
>      <param-value>/WEB-INF/applicationContext.xml</param-value>
>  </context-param>
>  <listener>
>      <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
>  </listener>
>  
> </web-app>
> ```
>
> 下图是Spring如何工作的高级展示。你应用中所有的类都由元数据组装到一起。
>
> <img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gkuhqrc0nzj30p60g0mx9.jpg" style="zoom:50%">
>
> 所以当 ApplicationContext 创建和实例化后，你就有了一个完全可配置和可执行的系统或应用。

#####  1.1 配置元数据

如上图，Spring容器使用了一种配置元数据的形式。表示应用程序的开发人员告诉Spring容器怎样去实例化、配置和装配你应用中的对象。

> 配置元数据传统上以简单直观的`XML格式`提供。还可以使用`基于Java的配置`、`基于注解配置`。
>

> 通常会定义*<u>服务层对象</u>*，*<u>数据层对象（DAO）</u>*，*<u>展现层对象</u>*（如Struts的Action实例），*<u>底层对象</u>*（比如Hibernate的SessionFactories）。==通常在容器中不定义细粒度的域对象，因为一般是由DAO层或者业务逻辑处理层负责创建和加载这些域对象。但是，你可以使用Spring集成Aspectj来配置IOC容器管理之外所创建的对象==。

- class：bean的全类名，通过反射的方式在IOC容器中创建Bean，所以要求Bean中必须有无参数的构造器。
- id：标识容器中的bean，id唯一。若id没有指定，Spring自动将全限定类名作为Bean的名字。（如：com.chance.spring.beans.HelloWorld）id属性值可以被依赖对象引用。

##### 1.2 实例化容器

提供给`ApplicationContext`构造函数的位置路径是资源字符串，这些资源字符串使容器可以从各种外部资源（例如本地文件系统，Java CLASSPATH等）加载配置元数据。

```java
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");
```

Spring的`Resources`抽象提供了一种方便的机制，用于从URI语法中定义的位置读取InputStream。资源路径被用于构建应用程序上下文。

###### 1.2.1 组成基于XML的配置元数据

> 通常，每个单独的XML配置文件都代表体系借个中的逻辑层或模块。加载XML配置的两种方式：
>
> - ClassPathXmlApplicationContext(String... configLocations)此函数具有多个位置。
> - 或者，使用一个或多个<import/>标签将其他xml文件整合到`applicationContext.xml`。
>
> ```xml
> <beans>
> 	<import resource="resources/services.xml"/>
>  	<import resource="/resources/daos.xml"/>
>  
> </beans>
> ```
>

> Tips：
>
> - 鉴于这些路径是相对的，最好不要使用任何斜线
> - 不建议使用相对的“../”路径引用父目录中的文件。这样做会创建对当前应用程序外部文件的依赖。尤其对于运行时解析过程选择“最近的”类路径根目录然后查找其父目录的classpath:URL，不建议使用此引用。
> - 可以使用完全限定的资源定位符位置来代替相对路径：如`file:C:/config/services.xml`或`classpath:/config/services.xml`。但是这样将应用程序的配置耦合到特定的绝对位置。==通常最好为这样的绝对位置保留一个间接寻址（如在运行时针对JVM系统属性解析“${...}”占位符）==。

##### 1.3 使用容器

###### 1.3.1 ApplicationContext

> ApplicationContext是一个*<u>维护bean定义以及相互依赖</u>*的注册表的高级工厂的接口。通过ApplicationContext，可以读取bean定义并访问它们，如下所示：
>
> ```java
> // 创建和配置bean
> ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext");
> // 检索配置的bean实例
> HelloWorld helloWorld = ctx.getBean("helloWorld", HelloWorld.class);
> // 配置实例
> helloWorld.hello();
> ```

###### 1.3.2 GenericApplicationContext

> `GenericApplicationContext`是为通用目的设计的，它能加载各种配置文件，例如xml，properties等等。
>
> - 内部使用的`DefaultListableBeanFactory`的实例，提供了一些方法来配置该实例，例如是否允许bean定义的覆盖、是否允许bean之间的循环应用等等。
> - 该类实现了`BeanDefinitionRegistry`，bean的定义注册。以便能通过`BeanDefinitionReader`读取bean的配置，并注册。`BeanDefinitionRegistry`接口的实现是直接使用内部的`DefaultListableBeanFactory`的实例。
> - 为了能够注册bean的定义，`refresh()`只允许调用一次。
>
> ```java
> GenericApplicationContext ctx = new GenericApplicationContext();
> XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(ctx);
> xmlBeanDefinitionReader.loadBeanDefinitions("applicationContext.xml");
> ctx.refresh();
> HelloWorld helloWorld = ctx.getBean("helloWorld3", HelloWorld.class);
> ```
>
> ==应用程序代码应该不调用getBean()方法，因此完全不依赖于Spring API==。例如，Spring与Web框架的集成为各种Web框架组件（例如控制器和JSF管理的Bean）提供了依赖项注入，可以通过元数据（例如自动装配注释）声明对特定Bean的依赖项。

#### 2. Bean

> bean是使用我们提供给容器的配置元数据创建的。在容器本身，这些bean定义表示为`BeanDefinition对象`。
>
> <img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gkzi7af4xkj30u607c0so.jpg" style="zoom:50%">
>
> 这些对象包含（除其他信息外）以下元数据：
>
> - **全限定类名**
> - Bean行为配置元素，用于声明Bean在容器中的行为（**作用域，生命周期回调等**）。
> - 引用该bean完成其工作所需的其他bean。这些引用也称为协作者或**依赖项**。
> - 要在新创建的对象中设置的其他配置设置，例如：池的大小限制或要在管理连接池的bean中使用的连接数。
>
> 该元数据转换为构成每个bean定义的一组属性。下表描述了这些属性：
>
> | Property                 | 在...中解释  |
> | ------------------------ | ------------ |
> | Class                    | 实例化Bean   |
> | Name                     | Bean的命名   |
> | Scope                    | Bean的作用域 |
> | Constructor arguments    | 依赖注入     |
> | Properties               | 依赖注入     |
> | Autowiring mode          | Bean的装配   |
> | Lazy initialization mode |              |
> | Initialization method    |              |
> | Destruction method       |              |
>
> 除了包含有关如何创建特定bean定义之外，这些ApplicationContext实现还允许注册在容器外部（由用户）创建的现有对象。
>
> - 通过`ApplicationContext`的`getBeanFactory()`方法来访问`BeanFactory`的。该方法返回BeanFactory的默认实现`DefaultListableBeanFactory`。
> - 通过`registerSingleton()`和`registerBeanDefinition()`方法支持此注册。

##### 2.1 Bean的命名

bean id唯一性由容器强制执行。一个bean通常只有一个标识符。但是，如果需要多个，则可以将多余的视为*<u>别名</u>*。

> 基于XML配置文件：
> - `id属性`可以精确的指定一个ID。
> - `name属性`可以指定别名，可使用逗号，分号，或空格分隔。
>
> 如果不显示指定，则容器将为该bean生成一个唯一的名称。但是，如果您希望通过使用ref元素或服务定位器样式查找通过名称引用该bean，则必须提供一个名称。

> *<u>通过在类路径中进行组件扫描</u>*，Spring会按照规则为未命名的组件生成Bean名称：
>
> - 采用简单的类名称并将其初始字符转换为小写。
> - 特殊情况下，如果有多个字符并且第一个和第二个字符均为大写字母，则会保留原始大小。这些规则与`java.beans.Introspector.decapitalize`定义的规则相同。

> **在Bean定义之外设置别名**
>
> 在基于XML的配置元数据中，可以使用`<alias/>`元素来完成此任务。
>
> ```xml
> <alias name="fromName" alias="toName"/>
> ```
>
> 在这种情况下，(在同一个容器中)名为fromName的bean在使用了这个别名定义之后，也可以被称为toName。
>

##### 2.2 实例化Bean

> Bean的定义实质上是创建一个或多个对象的方法。
>
> `<bean/>`元素的class属性指定要实例化的对象的类型。可以通过以下两种方式之一使用该属性：
>
> 1. 通常，在容器本身*<u>通过反射调用其构造函数直接创建Bean</u>*的情况下，指定要构造的Bean类，这在某种程度上等同于使用`new`运算符的Java代码。
> 2. 指定包含用于创建对象的静态工厂方法的实际类，*<u>容器将在类上调用静态工厂方法创建Bean</u>*。

> 内部类名：如果想要为静态内部类配置Bean定义，则必须使用内部类的二进制名称。
>
> 例如：如果在com.example包中有一个名为SomeThing的类，并且此SomeThing类具有一个名为OtherThing的静态内部类，则bean定义上的class属性的值为`com.example.SomeThing$OtherThing`。*<u>名称中使用$字符将内部类的类名与外部类名分开</u>*。

###### 2.3.1 使用构造函数实例化

> Spring IoC容器实际上可以管理您希望它管理的任何类。它并不局限于管理真正的javabean。大多数Spring用户更喜欢实际的javabean，它*<u>只有一个默认的(无参数的)构造函数和适当的setter和getter方法</u>*，这些方法是根据容器中的属性建模的。
>
> ```xml
><bean id="exampleBean" class="examples.ExampleBean"/>
> <bean name="anotherExample" class="examples.ExampleBeanTwo"/>
>```

###### 2.3.2 用静态工厂方法实例化

> 定义使用静态工厂方法创建的bean时，使用class属性指定包含静态工厂方法的类，并使用名为`factory-method的属性`指定工厂方法本身的名称。
>
> 以下bean定义通过调用工厂方法来创建bean。该定义不指定返回对象的类型，而*<u>仅指定包含工厂方法的类</u>*。此示例中，*<u>createInstance()方法必须是静态方法</u>*。
>
> ```xml
> <bean id="clientService"
>  class="examples.ClientService"
>  factory-method="createInstance"/>
> ```
>
> ClientService：
>
> ```java
> public class ClientService {
>     private static ClientService clientService = new ClientService();
>     private ClientService() {}
> 
>     public static ClientService createInstance() {
>         return clientService;
>     }
> }
> ```

###### 2.3.3 使用实例工厂方法实例化

> 使用实例工厂方法进行实例化会从容器中<u>*调用现有bean的非静态方法来创建新bean*</u>。
>
> - 将class属性保留为空
> - 在`factory-bean属性`中，指定包含要创建该对象的实例方法的bean的名称。
> - 使用`factory-method属性`设置工厂方法本身的名称。
>
> ```xml
> <!-- 工厂bean, 包含一个方法createInstance()方法 -->
> <bean id="serviceLocator" class="examples.DefaultServiceLocator">
>  <!-- 注入此bean所需的所有依赖项 -->
> </bean>
> 
> <!-- 通过工厂bean创建bean -->
> <bean id="clientService"
>  factory-bean="serviceLocator"
>  factory-method="createClientServiceInstance"/>
> ```
>
> 工厂类DefaultServiceLocator：
>
> ```java
> public class DefaultServiceLocator {
> 
>     private static ClientService clientService = new ClientServiceImpl();
> 
>     public ClientService createClientServiceInstance() {
>         return clientService;
>     }
> }
> ```
>
> 一个工厂类也可以包含一个以上的工厂方法，如下：
>
> ```xml
> <bean id="serviceLocator" class="examples.DefaultServiceLocator">
>  <!-- inject any dependencies required by this locator bean -->
> </bean>
> 
> <bean id="clientService"
>  factory-bean="serviceLocator"
>  factory-method="createClientServiceInstance"/>
> 
> <bean id="accountService"
>  factory-bean="serviceLocator"
>  factory-method="createAccountServiceInstance"/>
> ```
>
> ```java
> public class DefaultServiceLocator {
> 
>     private static ClientService clientService = new ClientServiceImpl();
> 
>     private static AccountService accountService = new AccountServiceImpl();
> 
>     public ClientService createClientServiceInstance() {
>         return clientService;
>     }
> 
>     public AccountService createAccountServiceInstance() {
>         return accountService;
>     }
> }
> ```
>
> 这种方法表明*<u>可以通过依赖项注入(DI)来管理和配置工厂bean本身</u>*。

> Tips：
>
> - “`factory bean`”：在Spring容器中配置的bean，它通过实例或静态工厂方法创建对象。
> - `FactoryBean`：是指特定于Spring的FactoryBean实现类。

###### 2.3.4 确定Bean的运行时类型

> Bean元数据定义中指定的类只是一个初始类引用，
>
> - 可能与声明的工厂方法结合，
> - 或者作为FactoryBean类，这可能导致bean的不同运行时类型，
> - 或者在实例化工厂方法的情况下根本不设置（而是通过指定的FactoryBean的名称解析）。
> - 此外，AOP代理可以用基于接口的代理包装一个bean实例，并且有限地公开目标bean的实际类型(仅仅是它实现的接口)。
>
> 了解特定bean的实际运行时类型的推荐方法是使用`BeanFactory.getType()`。这将考虑到上述所有情况，返回与BeanFactory.getBean()`调用返回相同的对象的类型。

#### 3. 依赖关系

##### 3.1 DI（依赖注入）

> DI是一个过程，通过该过程，对象仅通过构造函数参数，工厂方法的参数或在构造或创建对象实例后在对象实例上设置的属性来定义其依赖和关系。从工厂方法返回。然后，容器在创建bean时注入那些依赖项。从根本上讲，*<u>此过程是通过使用类的直接构造或服务定位器模式来控制bean自身依赖关系的实例化或位置的bean本身的逆过程</u>*（因此称为Control Inversion）。
>
> 使用DI原理，代码更加简洁，当为对象提供依赖项时，去耦会更有效。该对象不查找其依赖项，并且不知道依赖项的位置或类。结果，您的类变得更易于测试，尤其是当依赖项依赖于接口或抽象基类时，它们允许在单元测试中使用存根或模拟实现。
>
>  DI存在两个主要变体：==基于构造函数的依赖注入==和==基于Setter的依赖注入==。

###### 3.1.1 基于构造函数的依赖注入

> 基于构造函数的DI是通过容器*<u>调用具有多个参数的构造函数</u>*来完成的，*<u>每个参数表示一个依赖项</u>*。 与调用带有特定参数的静态工厂方法来构造Bean几乎是等效的。 以下示例显示了只能通过构造函数注入进行依赖项注入的类：
>
> ```java
> public class SimpleMovieLister {
> 
>     // SimpleMovieLister 依赖于 MovieFinder
>     private MovieFinder movieFinder;
> 
>     // 一个构造函数，以便Spring容器可以注入MovieFinder
>     public SimpleMovieLister(MovieFinder movieFinder) {
>         this.movieFinder = movieFinder;
>     }
> 
> }
> ```
>
> 注意：该类没有什么特别的。它是一个不依赖于特定容器的接口，基类或注解的POJO。

**构造函数参数解析**

> 构造函数参数解析匹配通过使用参数的类型进行。如果Bean定义的构造函数参数中没有潜在的歧义，则在实例化Bean时，在Bean定义中定义构造函数参数的顺序就是将这些参数提供给适当的构造函数的顺序。参考以下类：
>
> ```java
> public class ThingOne {
> 
>     public ThingOne(ThingTwo thingTwo, ThingThree thingThree) {
>         // ...
>     }
> }
> ```
>
> 假设ThingTwo和ThingThree类没有通过继承关联，则不存在潜在的歧义。 因此，以下配置可以正常运行，并且您无需在`<constructor-arg />`元素中显式指定构造函数参数索引或类型。
>
> ```xml
> <beans>
>     <bean id="beanOne" class="x.y.ThingOne">
>         <constructor-arg ref="beanTwo"/>
>         <constructor-arg ref="beanThree"/>
>     </bean>
> 
>     <bean id="beanTwo" class="x.y.ThingTwo"/>
> 
>     <bean id="beanThree" class="x.y.ThingThree"/>
> </beans>
> ```
>
> 当引用另一个bean时，类型是已知的，并且可以发生匹配。当使用简单类型（例如<value> true </ value>）时，Spring无法确定值的类型，因此在没有帮助的情况下无法按类型进行匹配。 考虑以下类别：
>
> ```java
> public class ExampleBean {
> 
>     private int years;
> 
>     private String ultimateAnswer;
> 
>     public ExampleBean(int years, String ultimateAnswer) {
>         this.years = years;
>         this.ultimateAnswer = ultimateAnswer;
>     }
> }
> ```
>
> - **构造函数参数类型匹配**：在上述情况下，如果通过使用type属性显式指定构造函数参数的类型，则容器可以使用简单类型的类型匹配。如下所示：
> - **构造函数参数索引**（==索引从0开始==）：可以使用index属性来显示指定构造函数参数的索引。
> - **构造函数参数名称**：您还可以使用构造函数参数名称来消除歧义。
>
> ```xml
> <bean id="exampleBean" class="examples.ExampleBean">
>     <constructor-arg type="int" value="7500000"/>
>     <constructor-arg type="java.lang.String" value="42"/>
>     <constructor-arg index="0" value="7500000"/>
>     <constructor-arg index="1" value="42"/>
>     <constructor-arg name="years" value="7500000"/>
>     <constructor-arg name="ultimateAnswer" value="42"/>
> </bean>
> ```

###### 3.1.2 基于Setter的依赖注入

> 通过使用无参构造函数或无参静态工厂方法实例化您的bean之后，容器通过在bean上调用setter方法完成基于setter的DI。
>
> ```java
> public class SimpleMovieLister {
> 
>     // SimpleMovieLister 依赖于 MovieFinder
>     private MovieFinder movieFinder;
> 
>     // setter方法，以便Spring容器可以注入MovieFinder
>     public void setMovieFinder(MovieFinder movieFinder) {
>         this.movieFinder = movieFinder;
>     }
> }
> ```
>
> `ApplicationContext`管理的bean支持==基于构造函数==和==基于setter的DI==。
>
>  在已经通过构造函数方法注入了某些依赖项之后，它还支持基于setter的DI。可以以`BeanDefinition`的形式配置依赖项，并与`PropertyEditor`实例结合使用，以将属性从一种格式转换为另一种格式。但是，大多数Spring用户并不直接（即以编程方式）使用这些类，而是使用XML bean定义，带注释的组件（即以 `@Component`，`@Controller` 等进行注释的类）或`@Bean`方法来处理这些类。基于Java的 `@Configuration`类。 然后将这些源在内部转换为BeanDefinition实例，并用于加载整个Spring IoC容器实例。

> 基于构造函数或基于setter的DI？ 
>
> 由于可以混合使用基于构造函数的DI和基于setter的DI，因此将构造函数用于强制性依赖项并将setter方法或配置方法用于可选依赖性是一个很好的经验法则。注意，*<u>在setter方法上使用@Required批注可以使该属性成为必需的依赖项</u>*。但是，最好使用带有参数的程序验证的构造函数注入。
>
> - Spring团队通常提倡构造函数注入，因为它使您可以将应用程序组件实现为不可变对象，并确保所需的依赖项不为null。此外，注入构造函数的组件始终以完全初始化的状态返回到客户端（调用）代码。附带说明一下，大量的构造函数自变量是一种不好的代码，这表明该类可能承担了太多的职责，应将其重构以更好地解决关注点分离问题。 
> - Setter注入主要应仅用于可以在类中分配合理的默认值的可选依赖项。否则，必须在代码使用依赖项的任何地方执行非空检查。 setter注入的一个好处是，setter方法使该类的对象在以后可以重新配置或重新注入。例如，如果第三方类未公开任何setter方法，则构造函数注入可能是DI的唯一可用形式。

###### 3.1.3 依赖性解析过程

> 容器执行bean依赖项解析，如下所示：
>
> - 使用描述所有bean的配置元数据创建和初始化ApplicationContext。（XML、Java代码或注释配置元数据）
> - 对于每个bean，其依赖项都以属性，构造函数参数或static-factory方法的形式表示。实际创建Bean时，会将这些依赖项提供给Bean。
> - 每个*<u>属性或构造器参数</u>*都要设置值的实际定义，或者是对容器中另一个bean的引用。
> - 作为值的每个属性或构造函数参数都将从其指定的格式转换为该属性或构造函数参数的实际类型。默认情况下，*<u>Spring可以将以字符串格式提供的值转换为所有内置类型，例如int，long，String，boolean等</u>*。
>
> 在创建容器时，Spring容器会验证每个bean的配置。但是，在实际创建Bean之前，不会设置Bean属性本身。
>
> - ==创建容器时，将创建具有单例作用域并设置为预先实例化（默认）的Bean==。 范围在Bean范围中定义。否则，仅在请求时才创建Bean。
> - 创建和分配bean的依赖关系及其依赖关系时，创建bean可能会导致创建一个bean图。请注意，这些依赖项之间的解析不匹配可能会在后期出现，即在第一次创建受影响的bean时。

> **循环依赖**
>
> 如果主要使用构造函数注入，则可能会创建无法解决的循环依赖方案。 例如：A类通过构造函数注入需要B类的实例，而B类通过构造函数注入需要A类的实例。如果为将要插入的类A和B配置Bean，Spring IoC容器将在运行时检测到此循环引用，并引发BeanCurrentlyInCreationException。
>
> 解决方案：
>
> - 一种可能的解决方案是编辑某些类的源代码，这些类的源代码由设置者而不是构造函数来配置。
> - 或者，避免构造函数注入，而仅使用setter注入。尽管不建议这样做。与“没有循环依赖关系”不同，Bean A和Bean B之间的循环依赖关系迫使其中一个Bean在完全完全初始化之前被注入另一个Bean（经典的“鸡与蛋”场景）。

> Spring*<u>在容器加载时检测配置问题</u>*，例如对不存在的Bean的引用和循环依赖项。*<u>在实际创建bean时，Spring设置属性并尽可能晚地解决依赖关系</u>*。这可能会延迟某些配置问题的可见性，这就是为什么==默认情况下ApplicationContext实现会预先实例化单例bean的原因==。在实际需要这些bean之前先花一些时间和内存来创建它们，您会在创建ApplicationContext时发现配置问题，而不是稍后。仍然可以覆盖此默认行为，以便单例bean延迟初始化，而不是预先实例化。
>
> 如果不存在循环依赖关系，则在将一个或多个协作Bean注入到从属Bean中时，每个协作Bean在注入到从属Bean中之前都已完全配置。这意味着，==如果bean A依赖于bean B，则Spring IoC容器会在对bean A调用setter方法之前完全配置beanB==。换句话说，bean被实例化（如果不是预先实例化的singleton），设置其依赖项，并调用相关的生命周期方法（例如已配置的init方法或InitializingBean回调方法）

###### 3.1.4 依赖注入的示例

> 以下示例将基于XML的配置元数据用于基于setter的DI。 Spring XML配置文件的一小部分指定了一些bean定义：
>
> ```xml
> <bean id="exampleBean" class="com.chance.spring.beans.ExampleBean">
>     <!-- setter injection using the nested ref element -->
>     <property name="beanOne">
>         <ref bean="anotherExampleBean"/>
>     </property>
> 
>     <!-- setter injection using the neater ref attribute -->
>     <property name="beanTwo" ref="yetAnotherBean"/>
>     <property name="integerProperty" value="1"/>
> </bean>
> 
> <bean id="anotherExampleBean" class="com.chance.spring.beans.AnotherBean"/>
> <bean id="yetAnotherBean" class="com.chance.spring.beans.YetAnotherBean"/>
> ```
>
> ExampleBean类：
>
> ```java
> public class ExampleBean {
> 
>     private AnotherBean beanOne;
> 
>     private YetAnotherBean beanTwo;
> 
>     private int i;
> 
>     public void setBeanOne(AnotherBean beanOne) {
>         this.beanOne = beanOne;
>     }
> 
>     public void setBeanTwo(YetAnotherBean beanTwo) {
>         this.beanTwo = beanTwo;
>     }
> 
>     public void setIntegerProperty(int i) {
>         this.i = i;
>     }
> }
> ```
>
> 在前面的示例中，声明了setter以与XML文件中指定的属性匹配。

> 以下示例使用基于构造函数的DI：
>
> ```xml
> <bean id="exampleBean" class="examples.ExampleBean">
>     <!-- constructor injection using the nested ref element -->
>     <constructor-arg>
>         <ref bean="anotherExampleBean"/>
>     </constructor-arg>
> 
>     <!-- constructor injection using the neater ref attribute -->
>     <constructor-arg ref="yetAnotherBean"/>
> 
>     <constructor-arg type="int" value="1"/>
> </bean>
> 
> <bean id="anotherExampleBean" class="examples.AnotherBean"/>
> <bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
> ```
>
> ExampleBean类：
>
> ```java
> public class ExampleBean {
> 
>     private AnotherBean beanOne;
> 
>     private YetAnotherBean beanTwo;
> 
>     private int i;
> 
>     public ExampleBean(
>         AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
>         this.beanOne = anotherBean;
>         this.beanTwo = yetAnotherBean;
>         this.i = i;
>     }
> }
> ```
>
> bean定义中指定的构造函数参数用作ExampleBean构造函数的参数。

> 现在考虑该示例的一个变体，在该变体中，不是使用构造函数，而是告诉Spring调用静态工厂方法以返回对象的实例：
>
> ```xml
> <bean id="exampleBean" class="examples.ExampleBean" factory-method="createInstance">
>     <constructor-arg ref="anotherExampleBean"/>
>     <constructor-arg ref="yetAnotherBean"/>
>     <constructor-arg value="1"/>
> </bean>
> 
> <bean id="anotherExampleBean" class="examples.AnotherBean"/>
> <bean id="yetAnotherBean" class="examples.YetAnotherBean"/>
> ```
>
> 以下示例显示了相应的ExampleBean类：
>
> ```java
> public class ExampleBean {
> 
>     // a private constructor
>     private ExampleBean(...) {
>         ...
>     }
> 
>     // a static factory method; the arguments to this method can be
>     // considered the dependencies of the bean that is returned,
>     // regardless of how those arguments are actually used.
>     public static ExampleBean createInstance (
>         AnotherBean anotherBean, YetAnotherBean yetAnotherBean, int i) {
> 
>         ExampleBean eb = new ExampleBean (...);
>         // some other operations...
>         return eb;
>     }
> }
> ```
>
> 静态工厂方法的参数由`<constructor-arg />`元素提供，与实际使用构造函数的情况完全相同。*<u>factory方法返回的类的类型不必与包含静态工厂方法的类的类型相同</u>*。 实例（非静态）工厂方法可以以基本上相同的方式使用。

##### 3.2 依赖性和详细配置

###### 3.2.1 Straight Values（原始值或字符串等）

> `<property />元素`的value属性或构造函数指定为可读的字符串表示形式。Spring的转换服务用于将这些值从字符串转为属性或参数的实际类型。以下示例显示了设置的各种值：
>
> ```xml
> <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
>     <!-- results in a setDriverClassName(String) call -->
>     <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
>     <property name="url" value="jdbc:mysql://localhost:3306/mydb"/>
>     <property name="username" value="root"/>
>     <property name="password" value="misterkaoli"/>
> </bean>
> ```
>
> 以下示例使用p-namespace进行更简洁的XML配置：
>
> ```java
> <beans xmlns="http://www.springframework.org/schema/beans"
>     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>     xmlns:p="http://www.springframework.org/schema/p"
>     xsi:schemaLocation="http://www.springframework.org/schema/beans
>     https://www.springframework.org/schema/beans/spring-beans.xsd">
> 
>     <bean id="myDataSource" class="org.apache.commons.dbcp.BasicDataSource"
>         destroy-method="close"
>         p:driverClassName="com.mysql.jdbc.Driver"
>         p:url="jdbc:mysql://localhost:3306/mydb"
>         p:username="root"
>         p:password="misterkaoli"/>
> 
> </beans>
> ```

> 还可以配置java.util.Properties实例，如下所示：
>
> ```java
> <bean id="mappings" class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
> 
>     <!-- typed as a java.util.Properties -->
>     <property name="properties">
>         <value>
>             jdbc.driver.className=com.mysql.jdbc.Driver
>             jdbc.url=jdbc:mysql://localhost:3306/mydb
>         </value>
>     </property>
> </bean>
> ```
>
> Spring容器*<u>通过使用JavaBeans `PropertyEditor机制`将<value />元素内的文本转换为java.util.Properties实例</u>*。

> **idref元素**
>
> idref元素只是一种防错方法，可以将容器中另一个bean的id（字符串值-不是引用）传递给<constructor-arg />或<property />元素。 以下示例显示了如何使用它：
>
> ```xml
> <bean id="theTargetBean" class="..."/>
> 
> <bean id="theClientBean" class="...">
>     <property name="targetName">
>         <idref bean="theTargetBean"/>
>     </property>
> </bean>
> ```
>
> 前面的bean定义片段（在运行时）与下面的片段完全等效：
>
> ```xml
> <bean id="theTargetBean" class="..." />
> 
> <bean id="client" class="...">
>     <property name="targetName" value="theTargetBean"/>
> </bean>
> ```
>
> 第一种形式优于第二种形式，因为*<u>使用idref标记可使容器在部署时验证所引用的名为bean的实际存在</u>*。 在第二个变体中，不对传递给客户端bean的targetName属性的值执行验证。 拼写错误仅在实际实例化客户端bean时才发现（最有可能导致致命的结果）。 如果客户端Bean是原型Bean，则可能在部署容器很长时间之后才发现此错字和所产生的异常。

> 在4.0 Bean XSD中不再支持idref元素上的local属性，因为它不再提供常规Bean引用上的值。 升级到4.0模式时，将现有的idref本地引用更改为idref bean。<idref />元素带来价值的一个常见地方（至少在Spring 2.0之前的版本中）是*<u>在ProxyFactoryBean bean定义中的AOP拦截器的配置中。 在指定拦截器名称时使用<idref />元素可防止您拼写错误的拦截器ID。</u>*

###### 3.2.2 对其他Bean的引用（协作者）

> `ref元素`是`<constructor-arg />或<property />`定义元素内的最后一个元素。在这里，您将Bean的指定属性的值设置为对容器管理的另一个Bean（协作者）的引用。引用的bean是要设置其属性的bean的依赖关系，并且在设置属性之前根据需要对其进行初始化。（如果协作者是单例bean，则它可能已经由容器初始化了。）所有引用最终都是对另一个对象的引用。范围和验证取决于您是通过bean还是父属性指定另一个对象的ID或名称。 *<u>通过<ref />标记的bean属性指定目标bean是最通用的形式，并且允许创建对同一容器或父容器中任何bean的引用，而不管它是否在同一XML文件中</u>*。 bean属性的值可以与目标bean的id属性相同，也可以与目标bean的name属性中的值之一相同。下面的示例演示如何使用ref元素：
>
> ```xml
> <ref bean="someBean"/>
> ```
>
> *<u>通过parent属性指定目标Bean将创建对当前容器的父容器中的Bean的引用</u>*。父属性的值可以与目标Bean的id属性或目标Bean的名称属性中的值之一相同。目标Bean必须位于当前容器的父容器中。 主要在具有容器层次结构并且要使用与父bean名称相同的代理将现有bean包装在父容器中时，才应使用此bean参考变量。 以下显示了如何使用parent属性：
>
> ```xml
> <!-- 在父上下文中 -->
> <bean id="accountService" class="com.something.SimpleAccountService">
>     <!-- 根据需要在此处插入依赖项 -->
> </bean>
> ```
>
> ```xml
> <!-- 在子上下文中 -->
> <!-- bean名称与父bean相同 -->
> <bean id="accountService" 
>     class="org.springframework.aop.framework.ProxyFactoryBean">
>     <property name="target">
>         <ref parent="accountService"/> <!-- 注意我们如何引用父bean -->
>     </property>
>     <!-- 根据需要在此处插入其他配置和依赖项 -->
> </bean>
> ```
>
> ref元素的local属性在4.0 Bean XSD中不再受支持，因为它不再提供常规Bean引用上的值。升级到4.0模式时，*<u>将现有的ref本地引用更改为ref bean</u>*。

###### 3.2.3 内部Bean

> <property />或<constructor-arg />元素内的<bean />元素定义了一个内部bean，如以下示例所示：
>
> ```xml
> <bean id="outer" class="...">
>     <!-- 无需使用对目标bean的引用，只需简单地内联定义目标bean -->
>     <property name="target">
>         <bean class="com.example.Person"> <!-- 这是内联的bean -->
>             <property name="name" value="Fiona Apple"/>
>             <property name="age" value="25"/>
>         </bean>
>     </property>
> </bean>
> ```
>
> *<u>内部bean定义不需要定义的ID或名称</u>*。如果指定，则容器不使用该值作为标识符。容器在创建时也将忽略作用域标志，因为内部Bean始终是匿名的，并且始终与外部Bean一起创建。不可能独立地访问内部bean或将其注入到协作bean中而不是封装到封闭bean中。 作为一个极端的例子，可以从自定义范围接收破坏回调，例如对于单例bean中包含的请求范围内的bean。内部bean实例的创建与其包含的bean绑定在一起，但是销毁回调使它可以参与请求范围的生命周期。这不是常见的情况。内部bean通常只共享其包含bean的作用域。

###### 3.2.4 集合

> 使用`<list/>, <set/>, <map/>和<props/>元素`分别设置Java的集合类型（List、Set、Map和Properties），以下示例显示了如何使用它们：
>
> ```xml
> <bean id="moreComplexObject" class="example.ComplexObject">
>     <!-- java.util.Properties -->
>     <property name="adminEmails">
>         <props>
>             <prop key="administrator">administrator@example.org</prop>
>             <prop key="support">support@example.org</prop>
>             <prop key="development">development@example.org</prop>
>         </props>
>     </property>
>     <!-- java.util.List -->
>     <property name="someList">
>         <list>
>             <value>a list element followed by a reference</value>
>             <ref bean="myDataSource" />
>         </list>
>     </property>
>     <!-- java.util.Map -->
>     <property name="someMap">
>         <map>
>             <entry key="an entry" value="just some string"/>
>             <entry key ="a ref" value-ref="myDataSource"/>
>         </map>
>     </property>
>     <!-- java.util.Set -->
>     <property name="someSet">
>         <set>
>             <value>just some string</value>
>             <ref bean="myDataSource" />
>         </set>
>     </property>
> </bean>
> ```
>
> ```xml
> bean | ref | idref | list | set | map | props | value | null
> ```

> **集合合并**
>
> Spring容器支持合并集合。应用开发人员可以定义`父<list />，<map />，<set />或<props />元素`，并具有`子<list />，<map />，<set />或<props />元素`。从父集合继承并覆盖值。即子集合的值是合并父集合和子集合的元素的结果，*<u>子集合的元素会覆盖父集合中指定的值</u>*（父子bean机制）。下面的示例演示了集合合并：
>
> ```xml
> <beans>
>     <bean id="parent" abstract="true" class="example.ComplexObject">
>         <property name="adminEmails">
>             <props>
>                 <prop key="administrator">administrator@example.com</prop>
>                 <prop key="support">support@example.com</prop>
>             </props>
>         </property>
>     </bean>
>     <bean id="child" parent="parent">
>         <property name="adminEmails">
>             <!-- 在子集合中指定合并 -->
>             <props merge="true">
>                 <prop key="sales">sales@example.com</prop>
>                 <prop key="support">support@example.co.uk</prop>
>             </props>
>         </property>
>     </bean>
> <beans>
> ```
>
> 注意：在子bean定义的adminEmails属性的<props />元素上使用`merge="true"属性`。当子bean由容器解析并实例化后，生成的实例具有adminEmails Properties集合，其中包含将孩子的adminEmails集合与父对象的adminEmails集合合并的结果。 以下清单显示了结果：
>
> ```
> administrator=administrator@example.com
> sales=sales@example.com
> support=support@example.co.uk
> ```
>
> 此合并行为类似地适用于<list />，<map />和<set />集合类型。
>
> 在<list />元素的特定情况下，将维护与List集合类型关联的语义（即，值的有序集合的概念）。父级的值先于子级列表的所有值。对于Map，Set和Properties集合类型，不存在排序。因此，对于容器内部使用的关联Map，Set和Properties实现类型基础的集合类型，没有任何排序语义有效。

> **集合合并的限制**
>
> 不能合并不同的集合类型（例如Map和List）。如果这样做，Exception可能会被抛出。 必须在下面的继承的子定义中指定merge属性。在父集合定义上指定merge属性是多余的，不会导致所需的合并。

> **强类型集合**
>
> 随着Java 5中泛型类型的引入，可以使用强类型集合。也就是说，可以声明一个Collection类型，使其只能包含String元素。如果使用Spring将强类型的Collection依赖注入到Bean中，则可以利用Spring的类型转换支持，以便在将强类型的Collection实例的元素添加到Bean中之前，先将其转换为适当的类型。以下Java类和bean定义显示了如何执行此操作：
>
> ```java
> public class SomeClass {
> 
>     private Map<String, Float> accounts;
> 
>     public void setAccounts(Map<String, Float> accounts) {
>         this.accounts = accounts;
>     }
> }
> ```
>
> ```xml
> <beans>
>     <bean id="something" class="x.y.SomeClass">
>         <property name="accounts">
>             <map>
>                 <entry key="one" value="9.99"/>
>                 <entry key="two" value="2.75"/>
>                 <entry key="six" value="3.99"/>
>             </map>
>         </property>
>     </bean>
> </beans>
> ```
>
> 当准备注入某bean的accounts属性时，*<u>可以通过反射获得有关强类型Map <String，Float>的元素类型的泛型信息</u>*。因此，Spring的类型转换基础结构将各种值元素识别为Float类型，并将字符串值（9.99、2.75和3.99）转换为实际的Float类型。

###### 3.2.5 Null和Empty字符串值

> Spring将属性等的空参数视为空字符串。以下基于XML的配置元数据代码段将email属性设置为空的字符串值（“”）。
>
> ```xml
> <bean class="ExampleBean">
>     <property name="email" value=""/>
> </bean>
> ```
>
> 上述示例等效于以下代码：`exampleBean.setEmail("");`
>
> 以下示例显示了`<null />元素`处理空值：
>
> ```xml
> <bean class="ExampleBean">
>     <property name="email">
>         <null/>
>     </property>
> </bean>
> ```
>
> 上述示例等效于以下代码：`exampleBean.setEmail(null);`

###### 3.2.6 具有p-spacename的方式

> 使用`p-namespace`， Spring支持带有名称空间的可扩展配置格式，这些名称空间基于XML Schema定义。但是，p命名空间未在XSD文件中定义，仅存在于Spring的核心中。以下示例显示了两个XML代码段（第一个使用标准XML格式，第二个使用p-命名空间），它们可以解析为相同的结果：
>
> ```xml
> <beans xmlns="http://www.springframework.org/schema/beans"
>     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>     xmlns:p="http://www.springframework.org/schema/p"
>     xsi:schemaLocation="http://www.springframework.org/schema/beans
>         https://www.springframework.org/schema/beans/spring-beans.xsd">
> 
>     <bean name="classic" class="com.example.ExampleBean">
>         <property name="email" value="someone@somewhere.com"/>
>     </bean>
> 
>     <bean name="p-namespace" class="com.example.ExampleBean"
>         p:email="someone@somewhere.com"/>
> </beans>
> ```
>
> 该示例在bean定义中的p命名空间中显示了一个名为email的属性。这告诉Spring包含一个属性声明。下一个示例包括另外两个bean定义，它们都引用了另一个bean：
>
> ```xml
> <beans xmlns="http://www.springframework.org/schema/beans"
>     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>     xmlns:p="http://www.springframework.org/schema/p"
>     xsi:schemaLocation="http://www.springframework.org/schema/beans
>         https://www.springframework.org/schema/beans/spring-beans.xsd">
> 
>     <bean name="john-classic" class="com.example.Person">
>         <property name="name" value="John Doe"/>
>         <property name="spouse" ref="jane"/>
>     </bean>
> 
>     <bean name="john-modern"
>         class="com.example.Person"
>         p:name="John Doe"
>         p:spouse-ref="jane"/>
> 
>     <bean name="jane" class="com.example.Person">
>         <property name="name" value="Jane Doe"/>
>     </bean>
> </beans>
> ```
>
> - 第一个bean定义使用<property name =“ spouse” ref =“ jane” />创建从bean john到bean jane的引用。
> - 第二个bean定义使用p：spouse-ref =“ jane”作为属性完全一样的东西。在这种情况下，配偶是属性名称，而-ref部分表示这不是一个直接值，而是对另一个bean的引用。

###### 3.2.7 具有c-namespace的XML方式

> 与带有p-namespace的XML方式类似，Spring 3.1中引入的c-namespace允许使用内联属性来配置构造函数参数，而不是嵌套的builder-arg元素。 以下示例使用c-namespace执行与基于构造函数的依赖注入相同的操作：
>
> ```xml
> <beans xmlns="http://www.springframework.org/schema/beans"
>     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>     xmlns:c="http://www.springframework.org/schema/c"
>     xsi:schemaLocation="http://www.springframework.org/schema/beans
>         https://www.springframework.org/schema/beans/spring-beans.xsd">
> 
>     <bean id="beanTwo" class="x.y.ThingTwo"/>
>     <bean id="beanThree" class="x.y.ThingThree"/>
> 
>     <!-- traditional declaration with optional argument names -->
>     <bean id="beanOne" class="x.y.ThingOne">
>         <constructor-arg name="thingTwo" ref="beanTwo"/>
>         <constructor-arg name="thingThree" ref="beanThree"/>
>         <constructor-arg name="email" value="something@somewhere.com"/>
>     </bean>
> 
>     <!-- c-namespace declaration with argument names -->
>     <bean id="beanOne" class="x.y.ThingOne" c:thingTwo-ref="beanTwo"
>         c:thingThree-ref="beanThree" c:email="something@somewhere.com"/>
> 
> </beans>
> ```

###### 3.2.8 符合属性名称

> 设置bean属性时，可以使用复合属性名称或嵌套属性名称，只要路径中除最终属性名称以外的所有组件都不为空即可。考虑以下bean定义：
>
> ```xml
> <bean id="something" class="things.ThingOne">
>     <property name="fred.bob.sammy" value="123" />
> </bean>
> ```
>
> 某些bean具有fred属性，该属性具有bob属性，该属性具有sammy属性，并且最终的sammy属性被设置为123的值。构造bean之后，fred的of不能为null。否则，将引发NullPointerException。

##### 3.3 使用`depends-on`

> 如果一个bean是另一个bean的依赖项，则通常意味着将一个bean设置为另一个bean的属性。通常，可以使用基于XML的配置元数据中的<ref />元素来完成此操作。但是，有时bean之间的依赖性不太直接。一个示例是何时需要触发类中的静态初始值设定项，例如用于数据库驱动程序注册。依赖属性可以显式地强制初始化一个或多个使用该元素的bean之前的bean。下面的示例使用depends-on属性来表示对单个bean的依赖关系：
>
> ```xml
> <bean id="beanOne" class="ExampleBean" depends-on="manager"/>
> <bean id="manager" class="ManagerBean" />
> ```
>
> 要表达对多个bean的依赖关系，请提供一个bean名称列表作为depends-on属性的值（逗号，空格和分号是有效的定界符）：
>
> ```xml
> <bean id="beanOne" class="ExampleBean" depends-on="manager,accountDao">
>     <property name="manager" ref="manager" />
> </bean>
> 
> <bean id="manager" class="ManagerBean" />
> <bean id="accountDao" class="x.y.jdbc.JdbcAccountDao" />
> ```
>
> `Depends-on属性`既可以指定初始化时间依赖性，也可以仅在单例bean的情况下指定相应的销毁时间依赖性。与给定bean定义依赖关系的从属bean首先被销毁，然后再销毁给定bean本身。因此，依赖也可以控制关闭顺序。

##### 3.4 延迟初始化Bean

> 默认情况下，作为初始化过程的一部分，ApplicationContext实现会急于创建和配置所有单例bean。通常，这种预初始化是可取的，因为与数小时甚至数天后相比，会立即发现配置或周围环境中的错误。如果不希望使用此行为，则可以通过将bean定义标记为延迟初始化来防止单例bean的预实例化。==延迟初始化的bean告诉IoC容器在首次请求时（而不是在启动时）创建一个bean实例==。在XML中，此行为由`<bean />元素上的lazy-init属性`控制，如以下所示：
>
> ```xml
> <bean id="lazy" class="com.something.ExpensiveToCreateBean" lazy-init="true"/>
> <bean name="not.lazy" class="com.something.AnotherBean"/>
> ```
>
> - 当前面的配置被ApplicationContext占用时，在ApplicationContext启动时不会急于预先实例化懒惰的bean，而在not.lazy Bean中则会急切地预先实例化。
> - 但是，当延迟初始化的bean是未延迟初始化的单例bean的依赖项时，ApplicationContext会在启动时创建延迟初始化的bean，因为它必须满足单例的依赖关系。 延迟初始化的bean被注入未初始化的其他地方的单例bean中。
> - 还可以通过使用<beans />元素上的default-lazy-init属性在容器级别上控制延迟初始化，如以下示例所示：
>
> ```xml
> <beans default-lazy-init="true">
>     <!-- 不会预先实例化任何bean... -->
> </beans>
> ```

##### 3.5 自动装配协作器

> Spring容器可以自动装配协作bean之间的关系。可以通过检查ApplicationContext的内容，让Spring为您的bean自动解决协作者（其他bean）。自动装配具有以下优点：
>
> - ==自动装配可以大大减少指定属性或构造函数参数的需要==。（例如Bean模板，在这方面也很有价值。） 
> - ==自动装配可以更新配置==。例如，如果需要将依赖项添加到类中，则无需修改配置即可自动满足该依赖项。
>
> 使用基于XML的配置元数据时，可以使用<bean />元素的autowire属性为bean定义指定自动装配模式。自动装配功能具有四种模式。可以为每个bean指定自动装配，因此可以选择要自动装配的bean。下表描述了四种自动装配模式：
>
> | 装配模式      | 说明                                                         |
> | :------------ | :----------------------------------------------------------- |
> | `no`          | （默认）无自动装配。 Bean引用必须由`ref`元素定义。 对于大型部署，建议不要更改默认设置，因为明确指定协作者可以提供更好的控制和清晰度。在某种程度上，它记录了系统的结构。 |
> | `byName`      | 按属性名称自动装配。 ==Spring查找与需要自动装配的属性同名的bean==。例如，如果一个bean定义被设置为按名称自动装配，并且包含一个“ master”属性（也就是说，它具有一个“ setMaster（..）”方法），那么Spring将查找一个名为“ master”的bean定义并使用 它来设置属性。 |
> | `byType`      | 如果容器中恰好存在一个属性类型的bean，则使该属性自动装配。==如果存在多个，则将引发致命异常==，这表明您不得对该bean使用`byType`自动装配。如果没有匹配的bean，则什么都不会发生（未设置该属性）。 |
> | `constructor` | 与`byType`类似，但适用于构造函数参数。 如果容器中不存在构造函数参数类型的一个bean，则将引发致命错误。 |
>
> 使用byType或构造函数自动装配模式，可以装配数组和类型化的集合。在这种情况下，将提供容器中与预期类型匹配的所有自动装配候选，以满足相关性。如果期望的键类型为String，则可以自动装配强类型Map实例。自动关联的Map实例的值包括与期望类型匹配的所有bean实例，并且Map实例的键包含相应的bean名称。

###### 3.5.1 自动装配的局限性和缺点

> 当在项目中一致使用自动装配时，自动装配效果最佳。考虑自动装配的局限性和缺点：
>
> - 属性和构造器参数设置中的显式依赖关系始终会覆盖自动装配。不能自动装配属性，例如基元，字符串和类（以及此类简单属性的数组）。此限制是设计使然。 
> - 自动装配不如显式装配精确。尽管如前所述，Spring还是小心避免在可能产生意外结果的歧义情况下进行猜测。Spring管理的对象之间的关系不再明确记录。
> - 装配信息可能不适用于可能从Spring容器生成文档的工具。
> - 容器内的多个bean定义可能与要自动装配的setter方法或构造函数参数指定的类型匹配。对于arrays，collections或Map实例，这不一定是问题。但是，对于需要单个值的依赖项，不会任意解决此歧义。如果没有唯一的bean定义可用，则会引发异常。

> 在后一种情况下，您有几种选择：
>
> - 放弃自动布线，转而使用明确的布线。
> - 通过将其bean的autowire-candidate属性设置为false，避免自动装配bean定义，如下一节所述。 
> - 通过将其<bean />元素的primary属性设置为true，将单个bean定义指定为主要候选对象。 
> - 如基于注释的容器配置中所述，通过基于注释的配置实现更细粒度的控件。

###### 3.5.2 从自动装配中排除Bean

> 在每个bean的基础上，可以从自动装配中排除一个bean。使用Spring的XML格式，将<bean />元素的autowire-candidate属性设置为false。 容器使特定的bean定义对自动装配基础结构不可用（包括批注样式配置，例如@Autowired）。

> `autowire-candidate属性`设计为仅影响基于类型的自动装配。它不会影响按名称显示的显式引用，即使未将指定的bean标记为自动装配候选，该名称也可得到解析。因此，如果名称匹配，按名称自动装配仍会注入Bean。

> 还可以基于与Bean名称的模式匹配来限制自动装配候选。顶级<beans />元素在其default-autowire-candidates属性内接受一个或多个模式。例如，要将自动装配候选状态限制为名称以Repository结尾的任何bean，请提供* Repository值。要提供多种模式，请在以逗号分隔的列表中定义它们。Bean定义的autowire-candidate属性的显式值true或false始终优先。对于此类bean，模式匹配规则不适用。这些技术对于您不希望通过自动装配将其注入其他bean的bean非常有用。这并不意味着排除的bean本身不能使用自动装配进行配置。相反，bean本身不是自动装配其他bean的候选对象。

##### 3.6 方法注入

> 在大多数应用场景中，容器中的大多数bean是单例的。==当单例Bean需要与另一个单例Bean协作或非单例Bean需要与另一个非单例Bean协作时，通常可以通过将一个Bean定义为另一个Bean的属性来处理依赖性==。==当bean的生命周期不同时会出现问题==。
>
> 假设单例bean A需要使用非单例（原型）bean B，也许在A的每个方法调用上都使用。容器仅创建一次单例bean A，因此只有一次机会来设置属性。每次需要一个容器时，容器都无法为bean A提供一个新的bean B实例。
>
> 解决方案是放弃某些控制反转。可以通过实现`ApplicationContextAware接口`，并通过对容器进行getBean(“ B”)调用来使Bean A知道该容器，每次bean A需要它时都请求Bean B实例。以下示例显示了此方法：
>
> ```java
> public class CommandManager implements ApplicationContextAware {
> 
>     private ApplicationContext applicationContext;
> 
>     public Object process(Map commandState) {
>         // 获取合适Command的新实例
>         Command command = createCommand();
>         // 在（希望是全新的）Command实例上设置状态
>         command.setState(commandState);
>         return command.execute();
>     }
> 
>     protected Command createCommand() {
>         // 注意Spring API的依赖
>         return this.applicationContext.getBean("command", Command.class);
>     }
> 
>     public void setApplicationContext(
>             ApplicationContext applicationContext) throws BeansException {
>         this.applicationContext = applicationContext;
>     }
> }
> ```
>
> 前面的内容是不理想的，因为业务代码意识到并耦合到Spring框架。 方法注入是Spring IoC容器的一项高级功能，使您可以简洁地处理此情况。

###### 3.6.1 查找方法注入

> 查找方法注入是在容器管理的Bean上重写方法的方法并返回容器中另一个命名Bean的查找结果的能力。查找通常涉及原型bean，如上一节中所述。Spring框架通过使用CGLIB库中的字节码生成来动态生成覆盖该方法的子类，从而实现此方法注入。
>
> - 为了使此动态子类起作用，Spring Bean容器子类的类也不能是final，而要覆盖的方法也不能是final。
> - 对具有抽象方法的类进行单元测试需要您自己对该类进行子类化，并提供该抽象方法的存根实现。
> - 组件扫描也需要具体的方法，这需要具体的类。
> - 另一个关键限制是，查找方法不适用于工厂方法，尤其不适用于配置类中的@Bean方法，因为在这种情况下，容器不负责创建实例，因此无法创建运行时生成的 动态子类。

> 对于前面的代码片段中的CommandManager类，Spring容器动态地覆盖createCommand（）方法的实现。 如重做的示例所示，CommandManager类没有任何Spring依赖项：
>
> ```java
> public abstract class CommandManager {
> 
>     public Object process(Object commandState) {
>         // 获取适当的Command接口的新实例
>         Command command = createCommand();
>         // 设置Command实例（全新的）的状态属性
>         command.setState(commandState);
>         return command.execute();
>     }
> 
>     // 好的...但是此方法的实现在哪里？
>     protected abstract Command createCommand();
> }
> ```
>
> 在包含要注入的方法的客户端类（在本例中为CommandManager）中，要注入的方法需要以下形式的签名：
>
> ```java
> <public|protected> [abstract] <return-type> theMethodName(no-arguments);
> ```
>
> 如果该方法是抽象的，则动态生成的子类将实现该方法。否则，动态生成的子类将覆盖原始类中定义的具体方法。考虑以下示例：
>
> ```xml
> <!-- 部署为原型的有状态Bean（非单个） -->
> <bean id="myCommand" class="fiona.apple.AsyncCommand" scope="prototype">
>     <!-- 根据需要在此处注入依赖项 -->
> </bean>
> 
> <!-- commandProcessor使用statefulCommandHelper -->
> <bean id="commandManager" class="fiona.apple.CommandManager">
>     <lookup-method name="createCommand" bean="myCommand"/>
> </bean>
> ```
>
> 每当需要新的myCommand bean实例时，标识为commandManager的bean就会调用其自己的createCommand()方法。如果确实需要将myCommand bean部署为原型，则必须小心。如果是单例，则每次都返回myCommand bean的相同实例。另外，在基于注释的组件模型中，可以通过@Lookup注释声明一个查找方法，如以下示例所示：
>
> ```java
> public abstract class CommandManager {
> 
>     public Object process(Object commandState) {
>         Command command = createCommand();
>         command.setState(commandState);
>         return command.execute();
>     }
> 
>     @Lookup("myCommand")
>     protected abstract Command createCommand();
> }
> ```
>
> 或者，可以依赖于目标bean根据lookup方法的声明的返回类型来解析：
>
> ```java
> public abstract class CommandManager {
> 
>     public Object process(Object commandState) {
>         MyCommand command = createCommand();
>         command.setState(commandState);
>         return command.execute();
>     }
> 
>     @Lookup
>     protected abstract MyCommand createCommand();
> }
> ```
>
> 请注意，通常应使用具体的存根实现声明此类带注释的查找方法，以使它们与Spring的组件扫描规则（默认情况下抽象类会被忽略）兼容。此限制不适用于显式注册或显式导入的Bean类。

###### 3.6.2 任意方法替换

> 与查找方法注入相比，方法注入的一种不太有用的形式是能够用另一种方法实现替换托管bean中的任意方法。您可以放心地跳过本节的其余部分，直到您真正需要此功能为止。

#### 4. Bean作用域

> 创建bean definition时，将创建一个配方用于创建该bean definition所定义的类实例。 `bean definition`的想法很重要，*<u>这意味着与类一样，可以从一个配方中创建许多对象实例</u>*。
>
> 你不仅可以控制要插入到对象（从特定`bean definition`创建的）中的各种依赖项和配置值，还可以控制对象的作用域。
>
> Spring框架支持6种作用域，其中4种作用域只有在使用支持web的ApplicationContext时才可用（request、session、application、websocket）。还可以创建自定义范围。
>
> | Scope       | Description                                                  |
> | :---------- | :----------------------------------------------------------- |
> | singleton   | (默认) IOC容器仅创建一个Bean实例，IOC容器每次返回的是同一个Bean实例。 |
> | prototype   | 每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean()时，相当于执行newXxxBean()。 |
> | request     | 每次HTTP请求都会创建一个新的Bean，该作用域仅适用于web的Spring WebApplicationContext环境。 |
> | session     | 同一个HTTP Session共享一个Bean，不同Session使用不同的Bean。该作用域仅适用于web的Spring WebApplicationContext环境。 |
> | application | 限定一个Bean的作用域为`ServletContext`的生命周期。该作用域仅适用于web的Spring WebApplicationContext环境。 |
> | websocket   | 仅适用于web的Spring WebApplicationContext环境。              |

##### 4.1 singleton

> 当定义一个bean definition并且它的作用域是一个单例对象时，Spring IoC容器会创建该bean definition定义的对象的一个实例。这个单一实例存储在这样的单例bean的缓存中，对这个已命名bean的*<u>所有后续请求和引用都会返回缓存的对象</u>*。

##### 4.2 prototype

> 非单例原型作用域导致每次对特定bean发出请求时都创建一个新的bean实例。也就是说，该bean被注入到另一个bean中，或者您通过容器上的getBean()方法调用请求它。作为规则，应该==对所有有状态bean使用原型作用域==，==对无状态bean使用单例作用域==。
>
> (数据访问对象(DAO)通常不配置为原型，因为典型的DAO不持有任何会话状态。)
>
> ```xml
> <bean id="accountService" class="com.something.DefaultAccountService" scope="prototype"/>
> ```

> ==Spring不管理原型bean的完整生命周期==。*<u>容器实例化、配置和装配原型对象并将其交给客户端，而不进一步记录该原型实例</u>*。因此，尽管初始化生命周期回调方法在所有对象上都被调用，==但在原型的情况下，配置的销毁生命周期回调不会被调用==。客户端代码必须清理原型范围的对象并释放原型bean所持有的昂贵资源。
>
> 要让Spring容器释放原型作用域bean所持有的资源，请尝试使用自定义bean后处理程序，它持有对需要清理的bean的引用。

##### 4.3 request,session,application,and websocket

> 请求、会话、应用程序和websocket作用域只有在使用web感知的Spring ApplicationContext实现(如XmlWebApplicationContext)时才可用。如果您将这些作用域与常规的Spring IoC容器(如ClassPathXmlApplicationContext)一起使用，就会抛出一个IllegalStateException，该异常报告一个未知的bean作用域。

###### 4.3.1 初始化Web配置

> 为了支持在`request`, `session`, `application`和`websocket`级别上bean的作用域，在定义bean之前需要进行一些小的初始配置（对单例和原型来说不是必需的）。
>
> 如何完成这个初始设置取决于特定的Servlet环境。
>
> - 如果在**Spring Web MVC**中访问范围限定的bean（实际上是在由Spring `DispatcherServlet`处理的请求中访问），则不需要进行特殊设置。DispatcherServlet已经公开了所有相关状态。
> - 如果使用**Servlet 2.5 web容器**，并且请求是在Spring的DispatcherServlet之外处理的(例如，在使用JSF或Struts时)，需要注册`org.springframework.web.context.request.RequestContextListener` `ServletRequestListener`。
> - 对于**Servlet 3.0+**，这可以通过使用WebApplicationInitializer接口以编程方式完成。
> - 对于较旧的容器，添加以下声明到您的web应用程序的web.xml文件：
>
> ```xml
> <web-app>
>     ...
>     <listener>
>         <listener-class>
>             org.springframework.web.context.request.RequestContextListener
>         </listener-class>
>     </listener>
>     ...
> </web-app>
> ```
>
> 另外，如果监听器设置存在问题，可以考虑使用Spring的`RequestContextFilter`。过滤器映射取决于周围的web应用程序配置，因此必须适当地更改它。下面显示了web应用程序的过滤器配置部分：
>
> ```xml
> <web-app>
>     ...
>     <filter>
>         <filter-name>requestContextFilter</filter-name>
>         <filter-class>org.springframework.web.filter.RequestContextFilter</filter-class>
>     </filter>
>     <filter-mapping>
>         <filter-name>requestContextFilter</filter-name>
>         <url-pattern>/*</url-pattern>
>     </filter-mapping>
>     ...
> </web-app>
> ```
>
> `DispatcherServlet`、`RequestContextListener`和`RequestContextFilter`都执行完全相同的操作，即将HTTP请求对象绑定到为该请求提供服务的线程。这使得作用域为请求和会话的bean在调用链的更深处可用。

##### 4.4 自定义Bean作用域



#### 5. 自定义Bean的性质（bean的生命周期）

> Spring框架提供了许多接口，可用于自定义Bean的性质。如下：
>
> - `Lifecycle Callbacks`（生命周期回调）
> - `ApplicationContextAware` 和 `BeanNameAware`
> - 其他 `Aware` 接口

##### 5.1 Lifecycle Callbacks(生命周期回调)

> 为了与容器对bean生命周期的管理进行交互，可以
>
> - 实现Spring `InitializingBean`和`DisposableBean`接口。容器为前者调用`afterPropertiesSet()`，并为后者调用`destroy()`，以*<u>使Bean在初始化和销毁Bean时执行某些操作</u>*。
> - 还可以在定义bean元数据的时候指定init-method属性。
> - `@PostConstruct`和`@PreDestroy`注解通常被认为是在Spring应用程序中*<u>接收生命周期回调</u>*的最佳实践。

> Spring框架内使用`BeanPostProcessor`实现来*<u>处理它可以找到的任何回调接口并调用适当的方法</u>*。如果你需要自定义功能或其他生命周期行为，则你可以自己实现BeanPostProcessor。除了==初始化和销毁回调==，Spring管理的对象还可以实现Lifecycle接口，以便这些对象可以在容器自身的生命周期的驱动下参与启动和关闭过程。

###### 5.1.1 初始化回调

> 实现`org.springframework.beans.factory.InitializingBean`接口，允许容器在设置了所有必要的属性后，执行初始化工作。
>
> ```java
> /**
>  * 在设置了提供的所有bean属性后，由BeanFactory调用
>  * (以及满足BeanFactoryAware和applicationcontext taware)。
>  */
> void afterPropertiesSet() throws Exception;
> ```
>
> - 建议不要使用InitializingBean接口，因为它将代码耦合到Spring。
> - ==建议使用@PostConstruct批注或指定POJO初始化方法==。
> - 对于基于XML的配置元数据，可以使用init-method属性指定具有无返回无参数签名的方法的名称。
> - 通过Java配置，可以使用@Bean的initMethod属性。
>
> ```xml
> <!-- 方式一 -->
> <bean id="exampleInitBean" class="examples.ExampleBean" init-method="init"/>
> ```
>
> ```java
> // 方式二
> @PostConstruct
> public void init() {
>     System.out.println("do some initialization work");
> }
> ```

###### 5.1.2 销毁回调

> 实现`org.springframework.beans.factory.DisposableBean接口`让bean在包含它的容器被销毁时获得回调。DisposableBean接口指定了一个方法:
>
> ```java
> /**
>  * 由BeanFactory在销毁单例时调用。
>  */
> void destroy() throws Exception;
> ```
>
> - 建议不要使用DisposableBean回调接口，因为它不必要地将代码耦合到Spring。
> - 建议使用`@PreDestroy`注解或指定bean定义支持的通用方法。
> - 使用基于XML的配置元数据时，可以使用destroy-method属性。
> - 通过Java配置，可以使用@Bean的destroyMethod属性。
>

###### 5.1.3 默认的初始化和销毁方法

> - 顶级<beans />元素属性上的`default-init-method`属性：*<u>使Spring IoC容器将Bean类上称为init的方法识别为初始化方法回调。在创建和组装bean时，如果bean类具有这种方法，则会在适当的时间调用它</u>*。
>
> - 顶级<beans />元素上的`default-destroy-method`属性：配置destroy方法回调。如果现有的Bean类已经具有按惯例命名的回调方法，则可以通过使用<bean />的init-method和destroy-method属性指定方法名称来覆盖默认值。
>
> 假设你的初始化回调方法命名为init()，而destroy回调方法命名为destroy()。class如下示所示：
>
> ```java
> public class DefaultBlogService implements BlogService {
> 
>  private BlogDao blogDao;
> 
>  public void setBlogDao(BlogDao blogDao) {
>      this.blogDao = blogDao;
>  }
> 
>  // this is (unsurprisingly) the initialization callback method
>  public void init() {
>      if (this.blogDao == null) {
>          throw new IllegalStateException("The [blogDao] property must be set.");
>      }
>  }
> }
> ```
>
> 然后，可以在以下内容的Bean中使用该类：
>
> ```xml
> <beans default-init-method="init">
> 
>  <bean id="blogService" class="com.something.DefaultBlogService">
>      <property name="blogDao" ref="blogDao" />
>  </bean>
> 
> </beans>
> ```
>

> ==Spring容器保证在为bean提供所有依赖项后立即调用已配置的初始化回调==。因此对原始bean引用调用初始化回调，这意味着AOP拦截器等还没有应用到bean。
>
> 首先完全创建目标bean，然后应用带有拦截器链的AOP代理(例如)。如果目标bean和代理是单独定义的，你的代码甚至可以绕过代理与原始目标bean交互。因此，将拦截器应用于init方法将是不一致的，因为这样做将把目标bean的生命周期与其代理/拦截器耦合起来，并在代码直接与原始目标bean交互时留下奇怪的语义。

###### 5.1.4 组合合生命周期机制

> 从Spring 2.5开始，有三种方式来控制Bean生命周期机制：
>
> - `InitializingBean`和`DisposableBean`回调接口
> - `自定义init()和destroy()方法`
> - `@PostConstruct和@PreDestroy注解`。

> - 如果为bean配置了多个生命周期机制，并且每个机制都配置了一个不同的方法名，那么按照下面列出的*<u>顺序执行每个配置的方法</u>*。
> - 但是，如果为这些生命周期机制中的一个以上配置了相同的方法名，则该方法将*<u>执行一次</u>*，

> 为同一个bean配置的具有不同初始化方法的多种生命周期机制如下：
>
> 1. 使用`@PostConstruct`注释的方法
> 2. 由`InitializingBean`回调接口定义的`afterPropertiesSet()`
> 3. 自定义配置的`init()`方法 
>
> 销毁方法的调用顺序相同： 
>
> 1. 使用`@PreDestroy`注释的方法 
> 2. 由`DisposableBean`回调接口定义的`destroy()` 
> 3. 自定义配置的`destroy()`方法

###### 5.1.5 启动和停止回调

> `Lifecycle`接口定义了任何对象的基本方法，它有自己的生命周期需求（例如启动和停止一些后台进程）：
>
> ```java
> public interface Lifecycle {
> 
>     void start();
> 
>     void stop();
> 
>     boolean isRunning();
> }
> ```
>
> ==任何Spring管理的对象都可以实现Lifecycle接口==。然后，当ApplicationContext本身接收到启动和停止信号时，例如，对于运行时的停止/重启场景，它将把这些调用级联到在该上下文中定义的所有Lifecycle实现。它通过委派给一个LifecycleProcessor来做到这一点：
>
> ```java
> public interface LifecycleProcessor extends Lifecycle {
> 
>     void onRefresh();
> 
>     void onClose();
> }
> ```
>
> `LifecycleProcessor`本身是Lifecycle接口的扩展。它还添加了另外两种方法来响应正在刷新和关闭的上下文。

> ==启动和关闭调用的顺序可能很重要==。
>
> - 如果任何两个对象之间存在“依赖”关系，则依赖方将在依赖项之后启动，并在依赖项之前停止。
> - 但是，有时直接依赖项是未知的。可能只知道某种类型的对象应该先于另一种类型的对象开始。在这些情况下，SmartLifecycle接口定义了另一个选项，即在其父接口Phased上定义的getPhase()方法。以下清单显示了Phased接口的定义：
>
> ```java
> public interface Phased {
> 
>  int getPhase();
> }
> ```
>
> 以下显示了SmartLifecycle接口的定义：
>
> ```java
> public interface SmartLifecycle extends Lifecycle, Phased {
> 
>  boolean isAutoStartup();
> 
>  void stop(Runnable callback);
> }
> ```
>
> 启动时，相位最低的对象首先启动。停止时，遵循相反的顺序。因此，*<u>实现SmartLifecycle并且其getPhase（）方法返回Integer.MIN_VALUE的对象将是第一个启动且最后一个停止的对象</u>*。在频谱的另一端，Integer.MAX_VALUE的相位值将指示该对象应最后启动并首先停止（可能是因为它依赖于正在运行的其他进程）。在考虑相位值时，==对于任何没有实现SmartLifecycle的“正常”生命周期对象，默认相位都是0==。因此，任何负相位值都表示对象应该在这些标准组件之前启动(在它们之后停止)，反之亦然。
>
> 由SmartLifecycle定义的stop方法接受回调。任何实现都必须在该实现的关闭过程完成后调用该回调的run()方法。这在必要时启用异步关闭，因为LifecycleProcessor接口的默认实现DefaultLifecycleProcessor将等待到每个阶段中的对象组调用回调的超时值。每个阶段的默认超时时间为30秒。*<u>可以通过在上下文中定义一个名为“lifecycleProcessor”的bean来覆盖默认的lifecycle processor实例</u>*。如果只想修改超时，那么以下内容就足够了：
>
> ```xml
> <bean id="lifecycleProcessor" class="org.springframework.context.support.DefaultLifecycleProcessor">
>     <!-- 超时属性 单位ms -->
>     <property name="timeoutPerShutdownPhase" value="10000"/>
> </bean>
> ```
>
> 如上所述，LifecycleProcessor接口还定义了用于刷新和关闭上下文的回调方法。后者将简单地驱动关机进程，就像显式地调用stop()一样，但它将在上下文关闭时发生。另一方面，“refresh”回调启用了SmartLifecycle bean的另一个特性。当上下文被刷新时(在所有对象被实例化和初始化之后)，回调将被调用，此时默认的生命周期处理器将检查每个SmartLifecycle对象的isAutoStartup()方法返回的布尔值。如果“true”，则该对象将在该点启动，而不是等待显式调用上下文的start()方法(与上下文刷新不同，对于标准上下文实现，上下文启动不会自动发生)。“相位”值以及任何“依赖”关系将以与上面描述的相同的方式确定启动顺序。

###### 5.1.6 在非Web应用程序中优雅地关闭Spring IoC容器

> 如果在非web应用程序环境中使用Spring的IoC容器。例如，在富客户机桌面环境中，你可以向JVM注册一个关机钩子。这样做可以确保优雅地关闭并调用单例bean上的相关销毁方法，以便释放所有资源。当然，你仍然必须正确配置和实现这些销毁回调。
>
> 要注册一个关闭钩子，您可以调用registerShutdownHook()方法，该方法在`ConfigurableApplicationContext`接口上声明：
>
> ```java
> public final class Boot {
>     public static void main(final String[] args) throws Exception {
>         ConfigurableApplicationContext ctx = new ClassPathXmlApplicationContext("beans.xml");
> 
>         // 为上面的上下文添加一个关闭钩子…
>         ctx.registerShutdownHook();
> 
>         // 应用程序运行在这里...
>         // main方法退出，在应用程序关闭之前调用hook…
>     }
> }
> ```

##### 5.2 ApplicationContextAware和BeanAware

> 当ApplicationContext创建实现`org.springframework.context.ApplicationContextAware`接口的对象实例时，将为该实例提供对该ApplicationContext的引用。==一种用途是通过编程方式检索其他bean。但是，应尽量避免使用它，因为它会将代码耦合到Spring，并且不遵循控制反转样式（将协作者作为属性提供给bean）==。

> ApplicationContext的其他方法提供*<u>对文件资源的访问</u>*、<u>*发布应用程序事件*</u>以及<u>*访问MessageSource*</u>。

> 从Spring 2.5开始，**自动装配是获得对ApplicationContext的引用的另一种选择**。构造函数和byType自动装配模式可以分别为构造函数参数或setter方法参数提供ApplicationContext类型的依赖关系。
>
> 为了获得更大的灵活性，包括自动装配字段和多个参数方法的能力，可以使用新的基于注解的自动装配特性，那么ApplicationContext将被自动拖放到一个字段、构造函数参数或方法参数中，==如果该字段、构造函数或方法携带@Autowired注释，则该字段、构造函数或方法将期望得到ApplicationContext类型==。

> 当ApplicationContext创建一个实现`org.springframework.beans.factory.BeanNameAware`接口的类时，该类将获得对其关联对象定义中定义的名称的引用。BeanNameAware接口的定义：
>
> ```java
> public interface BeanNameAware {
> 
>     void setBeanName(String name) throws BeansException;
> }
> ```
>
> *<u>在填充普通bean属性之后，但在初始化回调(如InitializingBean、afterPropertiesSet或自定义初始化方法)之前调用回调</u>*。

##### 5.3 其他Aware接口

> Spring提供了多种Aware回调接口，允许bean向容器表明它们需要某种基础设施依赖关系。下表总结了最重要的Aware接口：
>
> | Name                             | Injected Dependency                                          | Explained in…                                          |
> | :------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------- |
> | `ApplicationContextAware`        | Declaring `ApplicationContext`.                              | `ApplicationContextAware` and `BeanNameAware`          |
> | `ApplicationEventPublisherAware` | Event publisher of the enclosing `ApplicationContext`.       | Additional Capabilities of the `ApplicationContext`    |
> | `BeanClassLoaderAware`           | Class loader used to load the bean classes.                  | Instantiating Beans                                    |
> | `BeanNameAware`                  | Name of the declaring bean.                                  | `ApplicationContextAware` and `BeanNameAware`          |
> | `LoadTimeWeaverAware`            | Defined weaver for processing class definition at load time. | Load-time Weaving with AspectJ in the Spring Framework |
> | `MessageSourceAware`             | Configured strategy for resolving messages (with support for parametrization and internationalization). | Additional Capabilities of the `ApplicationContext`    |
> | `NotificationPublisherAware`     | Spring JMX notification publisher.                           | Notifications                                          |
> | `ResourceLoaderAware`            | Configured loader for low-level access to resources.         | Resources                                              |
> | `ServletConfigAware`             | Current `ServletConfig` the container runs in. Valid only in a web-aware Spring`ApplicationContext`. | Spring MVC                                             |
> | `ServletContextAware`            | Current `ServletContext` the container runs in. Valid only in a web-aware Spring`ApplicationContext`. | Spring MVC                                             |
> 

> Tips：==这些接口的使用将您的代码绑定到Spring API，并且不遵循控制反转样式。因此，建议将它们用于需要对容器进行编程访问的基础设施bean==。

#### 6. Bean Definition继承

> Bean Definition 可以包含许多配置信息，包括：
>
> - *<u>构造函数参数</u>*
> - *<u>属性值</u>*
> - *<u>特定于容器的信息</u>*（例如初始化方法，静态工厂方法名称等）

> 子bean definition从父定义继承配置数据。子定义可以覆盖某些值或根据需要添加其他值。使用父bean和子bean定义可以节省很多输入。实际上，这是一种模板形式。

> 如果以编程方式使用ApplicationContext接口，==子bean定义由ChildBeanDefinition类表示==。大多数用户是在`ClassPathXmlApplicationContext`之类的类中声明性地配置Bean定义。当基于XML配置元数据时，可以使用`parent属性`（将父bean指定为该属性的值）来指示子bean definition。
>
> 以下示例使用抽象属性显式地将父bean定义标记为抽象。
>
> ```xml
> <bean id="inheritedTestBean" abstract="true"
>      class="org.springframework.beans.TestBean">
>     <property name="name" value="parent"/>
>     <property name="age" value="1"/>
> </bean>
> 
> <bean id="inheritsWithDifferentClass"
>      class="org.springframework.beans.DerivedTestBean"
>      parent="inheritedTestBean" init-method="initialize">  
>     <property name="name" value="override"/>
>     <!-- age属性值1将从父bean继承 -->
> </bean>
> ```
>
> - 如果没有指定bean类，则子bean definition使用父bean definition中的bean类，但也可以覆盖它。
> - 在后一种情况下，子bean类必须与父类兼容，也就是说，它必须接受父类的属性值。
> - *<u>子bean definition继承父bean</u>*的==作用域==、==构造函数参数值==、==属性值==和==方法==，并具有添加新值的选项。指定的任何范围、初始化方法、销毁方法和/或静态工厂方法设置都将覆盖相应的父设置。
> - 其余的设置总是取自子bean definition：==依赖==、==自动装配模式==、==依赖项检查==、==单例==、==惰性初始化==。

> 如果父bean定义没有指定类，则需要显式地将父bean定义标记为抽象，如下所示:
>
> ```xml
> <bean id="inheritedTestBeanWithoutClass" abstract="true">
>  <property name="name" value="parent"/>
>  <property name="age" value="1"/>
> </bean>
> 
> <bean id="inheritsWithClass" class="org.springframework.beans.DerivedTestBean"
>      parent="inheritedTestBeanWithoutClass" init-method="initialize">
>  <property name="name" value="override"/>
>  <!-- age属性值1将从父bean继承 -->
> </bean>
> ```
>
> - ==父bean不能单独实例化，因为它是不完整的，并且还被显式标记为抽象==。
> - 如果试图单独使用这样一个抽象的父bean，将其引用为另一个bean的ref属性，或者使用父bean id显式地调用getBean()，将返回一个错误。类似地，容器的内部preInstantiateSingletons()方法忽略定义为抽象的bean definition。

> ==默认情况下，ApplicationContext预实例化所有单例==。因此，如果你有一个你只打算使用作为模板的(父)bean定义，这个定义指定了一个类，确保设置抽象属性为true很重要(至少对单例bean)，否则应用程序上下文会(试图)pre-instantiate抽象的bean。

#### 7. 容器扩展点

> Spring的IoC部分被设计成可扩展的。应用程序开发者通常不需要继承各种各样的BeanFactory或者ApplicationContext的实现类（BeanFactory和ApplicationContext都是接口）。*<u>通过插入特殊集成接口的实现，可以无限扩展Spring IoC容器</u>*。
>
> **扩展点：**就是允许你在*<u>不修改Spring源码</u>*的情况下，通过实现一些Spring预留的接口来把你自己的代码融入到Spring IoC容器初始化的过程中。

##### 7.1 使用`BeanPostProcessor`来自定义Bean

> `BeanPostProcessor`接口定义了回调方法，可以实现这些回调方法以提供自己的（或覆盖容器的默认值）实例化逻辑，依赖关系解析逻辑等。如果您想*<u>在Spring容器完成实例化，配置和初始化bean之后</u>*实现一些自定义逻辑，则可以插入一个或多个自定义BeanPostProcessor实现。
>
> ```java
> public interface BeanPostProcessor {
> 
> 	/**bean初始化之前，要干点什么或者什么都不干，但是干不干都要把人家给返回回去，不能一进来就出不去了*/
> 	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
> 
> 	/**bean已经初始化完了，这时候你想干点什么或者什么都不干，和上面一样，都要给人家返回回去*/
> 	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;
> 
> }
> ```
>
> 可以配置多个BeanPostProcessor实例，并且可以通过设置`order属性`来控制这些BeanPostProcessor实例的运行顺序。仅当BeanPostProcessor实现==Ordered接口==时，才可以设置此属性。

> BeanPostProcessor实例在bean（或对象）实例上运行。
>
> - Spring IoC容器实例化一个bean实例，然后BeanPostProcessor实例完成其工作。
> - BeanPostProcessor实例是按容器划分作用域的。仅在使用容器层次结构时，这才有意义。==如果在一个容器中定义BeanPostProcessor，它将仅对该容器中的bean进行后处理==。一个容器中定义的Bean不会被另一个容器中定义的BeanPostProcessor进行后处理，即使这两个容器是同一层次结构的一部分。
> - 要更改实际的bean definition(即定义bean的蓝图)，需要使用`BeanFactoryPostProcessor`。
>
> BeanPostProcessor接口恰好由两个回调方法组成。当这样一个类注册为容器的后处理器，对于容器创建的每个bean实例，
>
> - 后处理器在容器初始化方法(如InitializingBean afterPropertiesSet()和任何声明的init()方法)之前以及任何bean初始化后回调。
> - 后处理器可以对bean实例执行任何操作，包括完全忽略回调。
> - bean后处理器通常检查回调接口，或者使用代理包装bean。
> - 为了提供代理包装逻辑，一些==Spring AOP基础设施类被实现为bean后处理器==。
>
> ApplicationContext自动检测在配置元数据中定义的实现BeanPostProcessor接口的任何bean。ApplicationContext将这些bean注册为后处理程序，以便稍后在创建bean时调用它们。Bean后处理器可以像任何其他Bean一样部署在容器中。
>
> 注意，当在配置类上使用@Bean工厂方法声明BeanPostProcessor时，工厂方法的返回类型应该是实现类本身，或者至少是org.springframework.bean.factory.config.BeanPostProcessor接口，清楚地指示该bean的后处理器特性。否则，ApplicationContext将无法在完全创建它之前按类型自动检测它。由于为了保证上下文中其他bean的初始化，需要尽早实例化BeanPostProcessor，所以这种早期类型检测非常重要。

###### 7.1.1 示例

> 字面上的意思是bean的后置处理器，什么意思呢？
>
> 自定义BeanPostProcessor实现类定义如下：
>
> ```java
> public class InstantiationTracingBeanPostProcessor implements BeanPostProcessor {
> 
>     /**
>      * 只需按原样返回实例化的bean
>      *
>      * @param bean
>      * @param beanName
>      * @return
>      * @throws BeansException
>      */
>     @Override
>     public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
>         // 我们可以返回这里的任何对象引用…
>         return bean;
>     }
> 
>     @Override
>     public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
>         System.out.println("Bean '" + beanName + "' created : " + bean.toString());
>         return bean;
>     }
> }
> ```
>
> ```xml
> <?xml version="1.0" encoding="UTF-8"?>
> <beans xmlns="http://www.springframework.org/schema/beans"
>        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>        xmlns:context="http://www.springframework.org/schema/context"
>        xsi:schemaLocation="http://www.springframework.org/schema/beans
>        http://www.springframework.org/schema/beans/spring-beans.xsd
>        http://www.springframework.org/schema/context
>        http://www.springframework.org/schema/context/spring-context.xsd">
> 
>     <bean id="myBeanPostProcessor" class="com.chance.spring.beans.MyBeanPostProcessor"/>
>     
> </beans>
> ```

###### 7.1.2 示例

> 与自定义BeanPostProcessor实现一起使用回调接口或注解是扩展Spring IoC容器的常见方法。一个例子是Spring的`RequiredAnnotationBeanPostProcessor`——一个随Spring发行版一起发布的BeanPostProcessor实现，==它确保用(任意)注解标记的bean上的JavaBean属性实际上(配置为)依赖于注入一个值==。

##### 7.2 使用`BeanFactoryPostProcessor`自定义配置元数据

> ==BeanFactoryPostProcessor对Bean配置元数据进行操作==。（Spring IoC容器允许BeanFactoryPostProcessor读取配置元数据，并有可能在容器实例化除BeanFactoryPostProcessor实例以外的任何bean之前更改它）
>
> ==可以配置多个BeanFactoryPostProcessor实例==，并且可以通过设置order属性来控制这些BeanFactoryPostProcessor实例的运行顺序。
>
> ```java
> public interface BeanFactoryPostProcessor {
> 
> 	/**
> 	 * 在标准初始化之后修改应用程序上下文的内部bean工厂。所有bean定义都已加载，但还没有实例化bean。这允许覆盖或添加属性，甚至可以在快速初始化bean中。
> 	 */
> 	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
> 
> }
> ```

> 如果希望更改实际bean实例(即从配置元数据创建的对象)，则需要使用BeanPostProcessor(在前面通过使用BeanPostProcessor自定义bean中进行了描述)。虽然在BeanFactoryPostProcessor中使用bean实例在技术上是可行的(例如，通过使用BeanFactory.getBean())，但是这样做会导致过早的bean实例化，违反标准的容器生命周期。这可能会导致负面的副作用，比如绕过bean的后处理。
>
> 另外，BeanFactoryPostProcessor实例的作用域是针对每个容器。一个容器中的bean definition不会被另一个容器中的BeanFactoryPostProcessor实例进行后处理，即使这两个容器属于同一层次结构。

> 当bean工厂后处理器在ApplicationContext中声明时，它将自动运行，以便对定义容器的配置元数据应用更改。Spring包括许多预定义的bean工厂后处理器，如`PropertyOverrideConfigurer`和`PropertySourcesPlaceholderConfigurer`。您还可以使用自定义BeanFactoryPostProcessor—例如，用于注册自定义属性编辑器。
>
> ApplicationContext自动检测部署到其中实现BeanFactoryPostProcessor接口的任何bean。在适当的时候，它将这些bean用作bean工厂的后处理器。可以像部署任何其他bean一样部署这些后处理bean。

###### 7.2.1 示例：类名替换PropertySourcesPlaceholderConfigurer

> 使用PropertyPlaceholderConfigurer将属性值从bean定义外部化到使用标准Java属性格式的单独文件中。这样做使部署应用程序的人员能够定制特定于环境的属性，如数据库url和密码，而免去了修改主XML定义文件或容器文件的复杂性或风险。
>
> ###### 考虑以下基于xml的配置元数据片段，其中定义了具有占位符值的数据源。该示例显示了从外部属性文件配置的属性。在运行时，*<u>PropertyPlaceholderConfigurer应用于元数据，元数据将替换数据源的一些属性</u>*。要替换的值指定为表单==${property-name}==的占位符，该表单遵循Ant / log4j / JSP EL样式。
>
> ```xml
> <bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
>     <property name="locations" value="classpath:com/something/jdbc.properties"/>
> </bean>
> 
> <bean id="dataSource" destroy-method="close"
>         class="org.apache.commons.dbcp.BasicDataSource">
>     <property name="driverClassName" value="${jdbc.driverClassName}"/>
>     <property name="url" value="${jdbc.url}"/>
>     <property name="username" value="${jdbc.username}"/>
>     <property name="password" value="${jdbc.password}"/>
> </bean>
> ```
>
> 实际值来自另一个标准Java属性格式的文件：
>
> ```properties
> jdbc.driverClassName=org.hsqldb.jdbcDriver
> jdbc.url=jdbc:hsqldb:hsql://production:9002
> jdbc.username=sa
> jdbc.password=root
> ```
>
> 因此，字符串${jdbc.username}在运行时被替换为值“sa”。PropertyPlaceholderConfigurer检查bean定义的大多数属性和属性中的占位符。此外，还可以定制占位符前缀和后缀。
>
> 使用Spring 2.5中引入的上下文命名空间，可以使用专用的配置元素配置属性占位符。可以在location属性中以逗号分隔的列表的形式提供一个或多个位置。
>
> ```xml
> <context:property-placeholder location="classpath:com/foo/jdbc.properties"/>
> ```
>
> PropertyPlaceholderConfigurer不仅在指定的属性文件中查找属性。默认情况下，如果不能在指定的属性文件中找到属性，它还会检查Java系统属性。您可以使用以下三个受支持的整数值之一来设置配置程序的systemPropertiesMode属性，从而自定义此行为：
>
> - never(0)：从不检查系统属性
> - fallback(1)：如果在指定的属性文件中无法解析，则检查系统属性。这是默认值。
> - override(2)：在尝试指定的属性文件之前，先检查系统属性。这允许系统属性覆盖任何其他属性源。

###### 7.2.2 示例：PropertyOverrideConfigurer

> PropertyOverrideConfigurer是另一个bean工厂后处理器，原始定义可以有bean属性的默认值，也可以没有值。如果覆盖属性文件没有针对某个bean属性的条目，则使用默认上下文定义。
>
> 请注意，bean definition不知道被覆盖，因此从XML定义文件中不能立即看出正在使用覆盖配置器。在多个PropertyOverrideConfigurer实例为同一个bean属性定义不同值的情况下，由于覆盖机制，最后一个实例胜出。`
>
> 下面的清单显示了这种格式的一个示例：
>
> ```properties
> dataSource.driverClassName=com.mysql.jdbc.Driver
> dataSource.url=jdbc:mysql:mydb
> ```
>
> 这个示例文件可以与一个容器定义一起使用，该容器定义包含一个名为dataSource的bean，该bean具有驱动程序和url属性。 
>
> 还支持复合属性名，只要路径的每个组件(除了被覆盖的final属性)都是非空的(假定由构造函数初始化)。在下面的示例中，tom bean的fred属性的bob属性的sammy属性被设置为标量值123：
>
> ```properties
> foo.fred.bob.sammy=123
> ```
>
> 使用Spring 2.5中引入的上下文命名空间，可以使用专用的配置元素配置覆盖属性：
>
> ```xml
> <context:property-override location="classpath:override.properties"/>
> ```

##### 7.3 使用`FactoryBean`定制实例化逻辑

> 实现org.springframework.beans.factory.FactoryBean接口用于本身就是工厂的对象。
>
> FactoryBean接口是一个可插入Spring IoC容器实例化逻辑的点。如果有复杂的初始化代码，相对于(潜在的)冗长的XML，用Java更好地表达，那么您可以创建自己的FactoryBean，在该类中编写复杂的初始化，然后将定制的FactoryBean插入容器中。 FactoryBean接口提供了三种方法：
>
> - **Object getObject()**：返回此工厂创建的对象的实例。实例可以共享，具体取决于该工厂是否返回单例或原型。
> - **boolean isSingleton()**：如果此FactoryBean返回单例则返回true，否则返回false。
> - **Class getObjectType()**：返回由getObject()方法返回的对象类型；如果类型未知，则返回null。
>
> Spring框架中的许多地方都使用了FactoryBean概念和接口。Spring附带了50多个FactoryBean接口实现。

> *<u>当需要向容器要求一个实际的FactoryBean实例本身而不是由它产生的bean时，请在调用ApplicationContext的getBean()方法时在该bean的ID前面加上一个`＆`符号</u>*。因此，对于给定的myBean id为FactoryBean的FactoryBean，在容器上调用getBean(“ myBean”)将返回FactoryBean的乘积，而调用getBean(“＆myBean”)将返回FactoryBean实例本身。

#### 9. 类路径扫描和托管组件



#### 10. 使用JSR 330标准注解



#### 11. 基于Java的容器配置



#### 12. Environment 抽象



#### 13. 注册一个`LoadTimeWeaver`



#### 14. `ApplicationContext`的其他功能



#### 15. BeanFactory

> BeanFactory API为Spring的IoC功能提供了底层基础。它的特定约定主要用于与Spring的其他部分和相关的第三方框架的集成，它的`DefaultListableBeanFactory`实现是高级别的`GenericApplicationContext`容器中的一个关键委托。
>
> BeanFactory和相关接口(如BeanFactoryAware、InitializingBean、DisposableBean)是其他框架组件的重要集成点。由于不需要任何注解甚至反射，它们允许容器与其组件之间进行非常有效的交互。应用程序级bean可能使用相同的回调接口，但通常更倾向于通过注释或编程配置进行声明性依赖注入。
>
> 请注意，核心BeanFactory API级别及其DefaultListableBeanFactory实现没有对要使用的配置格式或任何组件注释进行假设。所有这些风格都通过扩展(比如XmlBeanDefinitionReader和AutowiredAnnotationBeanPostProcessor)实现，并在共享的BeanDefinition对象上进行操作，作为核心元数据表示。这就是使Spring的容器如此灵活和可扩展的本质所在。

##### 15.1 BeanFactory 或 ApplicationContext

> 解释BeanFactory和ApplicationContext容器级别之间的差异，以及引导的含义。
>
> 你应该使用ApplicationContext，除非您有很好的理由不这样做，使用GenericApplicationContext和它的子类AnnotationConfigApplicationContext作为自定义引导的通用实现。这些是用于所有常见目的的Spring核心容器的主要入口点:加载配置文件、触发类路径扫描、以编程方式注册bean定义和带注释的类，以及(从5.0开始)注册功能性bean定义。
>
> 因为ApplicationContext包含了BeanFactory的所有功能，所以一般建议它优于普通的BeanFactory，除非需要对bean处理进行完全控制的场景除外。在ApplicationContext(比如GenericApplicationContext实现)的几种检测到bean按照惯例(也就是说,由bean名称或bean类型——特别是,后处理器),而普通DefaultListableBeanFactory不可知论者是关于任何特殊bean。
>
> 对于许多扩展的容器特性，如注释处理和AOP代理，BeanPostProcessor扩展点是必不可少的。如果只使用普通的DefaultListableBeanFactory，默认情况下不会检测到这种后处理器并激活它。这种情况可能令人困惑，因为您的bean配置实际上没有任何错误。相反，在这样的场景中，容器需要通过附加的设置进行完全引导。
>
> 下表列出了BeanFactory和ApplicationContext接口和实现提供的特性。
>
> | Feature                                                 | `BeanFactory` | `ApplicationContext` |
> | :------------------------------------------------------ | :------------ | :------------------- |
> | Bean instantiation/wiring                               | Yes           | Yes                  |
> | Integrated lifecycle management                         | No            | Yes                  |
> | Automatic `BeanPostProcessor`registration               | No            | Yes                  |
> | Automatic `BeanFactoryPostProcessor`registration        | No            | Yes                  |
> | Convenient `MessageSource` access (for internalization) | No            | Yes                  |
> | Built-in `ApplicationEvent`publication mechanism        | No            | Yes                  |
>
> 要显式地用DefaultListableBeanFactory注册一个bean后处理器，你需要通过编程调用addBeanPostProcessor，如下面的例子所示：
>
> ```java
> DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
> // populate the factory with bean definitions
> 
> // now register any needed BeanPostProcessor instances
> factory.addBeanPostProcessor(new AutowiredAnnotationBeanPostProcessor());
> factory.addBeanPostProcessor(new MyBeanPostProcessor());
> 
> // now start using the factory
> ```
>
> 要将一个BeanFactoryPostProcessor应用到一个普通的DefaultListableBeanFactory中，你需要调用它的postProcessBeanFactory方法，如下面的例子所示：
>
> ```java
> DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
> XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
> reader.loadBeanDefinitions(new FileSystemResource("beans.xml"));
> 
> // bring in some property values from a Properties file
> PropertySourcesPlaceholderConfigurer cfg = new PropertySourcesPlaceholderConfigurer();
> cfg.setLocation(new FileSystemResource("jdbc.properties"));
> 
> // now actually do the replacement
> cfg.postProcessBeanFactory(factory);
> ```
>
> 在这两种情况下,明确登记步骤是不方便的,这就是为什么各种ApplicationContext变体都优于纯DefaultListableBeanFactory回弹应用程序,尤其是当依靠BeanFactoryPostProcessor和BeanPostProcessor实例扩展容器功能在一个典型的企业设置。
>
> 一个AnnotationConfigApplicationContext注册了所有公共注释后处理器，并可能通过配置注释(如@EnableTransactionManagement)在后台引入额外的处理器。在Spring的基于注解的配置模型的抽象层上，bean后处理器的概念仅仅成为内部容器的细节。

