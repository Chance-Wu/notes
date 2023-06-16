#### 1. 简介

##### 1.1 Spring是什么

> 详细描述：
>
> - 轻量级：非侵入性。基于Spring开发的应用中的对象可以不依赖于Spring的API。
> - ==控制反转(inversieon of control)：解耦合。一个对象依赖的其他对象会通过被动的方式传递进来，而不是这个对象自己创建或者查找依赖对象==。
> - ==面向切面(aspect oriented programming)：把业务逻辑和系统服务分开。==
> - ==容器：Spring包含并且管理应用对象的的配置和生命周期。==
> - 框架：实现了使用简单的组件配置组合成一个复杂的应用。在Spring中应用对象被声明式地组合，典型的是在一个XML文件里。Spring也提供了很多基础功能（事务管理、持久化框架集成等）。

##### 1.2 Spring模块

<img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gku8940co7j31220tadi1.jpg" style="zoom:40%">

#### 2. HelloWorld

> 非spring代码：
>
> ```java
> public class Main {
> 
>     public static void main(String[] args) {
>         // 1.创建HelloWorld的一个对象
>         HelloWorld helloWorld = new HelloWorld();
>         // 2.为name属性复制
>         helloWorld.setName("chance");
>         // 调用hello方法
>         helloWorld.hello();
>     }
> }
> ```
>
> 前两步可以交给Spring管理。

> 1. 所需基础jar包：
>
> `commons-logging-1.2.jar`	spring必须依赖的日志包
>
> `spring-beans-4.3.18.RELEASE.jar`
>
> `spring-context-4.3.18.RELEASE.jar`
>
> `spring-core-4.3.18.RELEASE.jar`
>
> `spring-expression-4.3.18.RELEASE.jar`

> 2. 创建一个Spring的配置文件 `applicationContext.xml`（一个典型的Spring项目需要创建一个或多个Bean配置文件，这些配置文件用于在Spring IOC容器里配置Bean。Bean的配置文件可以放在classpath下，也可以放在其他目录下。）
>
> ```xml
> <?xml version="1.0" encoding="UTF-8"?>
> <beans xmlns="http://www.springframework.org/schema/beans"
>        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
>        xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
> 
>     <!--配置bean-->
>     <bean id="helloWorld" class="com.chance.spring.beans.HelloWorld">
>         <property name="name" value="Spring"></property>
>     </bean>
> 
> </beans>
> ```

> 3. 代码
>
> ```java
> public class Main {
> 
>     public static void main(String[] args) {
>         // 1.创建 Spring 的 IOC 容器对象
>         ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
>         // 2.从 IOC 容器中获取 Bean 实例
>         HelloWorld helloWorld = (HelloWorld) ctx.getBean("helloWorld");
>         
>         // 调用hello方法
>         helloWorld.hello();
>     }
> }
> ```

> 测试创建容器，Spring做了哪些工作：
>
> ```
> 十一月 19, 2020 10:12:35 上午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
> 信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@7aec35a: startup date [Thu Nov 19 10:12:35 CST 2020]; root of context hierarchy
> 十一月 19, 2020 10:12:35 上午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
> 信息: Loading XML bean definitions from class path resource [applicationContext.xml]
> --------->HelloWorld无参构造器
> --------->setName：Spring
> ```

#### 3. IOC & DI 概述

> **IOC**：其思想就是*<u>反转资源获取的方向</u>*。传统的资源查找方式要求组件向容器内发起请求查找资源。作为回应，容器适时的返回资源，而应用了IOC之后，则是==容器主动地将资源推送给它所管理的组件，组件所要做的仅是一种合适的方式来接受资源==。这种行为也被称为查找的被动形式。
>
> **DI**：IOC的另一种表述方式，即==组件以一些预先定义好的方式（如setter方法）接受来自如容器的资源注入==。相对于IOC而言，这种表述更直接。

##### 3.1 IOC前生

需求：生成HTML或PDF格式的不同类型的报表。

> 1. 分离接口与实现（耦合度最高）
>
> <img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gkuaetuz2fj30z00a8q5v.jpg" style="zoom:60%">

> 2. 采用工厂设计模式
>
> <img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gkuag6v7kbj312c0ein1w.jpg" style="zoom:60%">

> 3. 采用反转控制
>
> <img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gkuakpwgrjj318a0gk7ae.jpg" style="zoom:60%">

#### 4. 配置Bean

##### 4.1 IOC容器里配置Bean

> 在xml文件中通过bean节点来配置bean：
>
> ```xml
> <bean id="helloWorld" class="com.chance.spring.beans.HelloWorld">
>     <property name="name" value="Spring"></property>
> </bean>
> ```
>
> class：bean的全类名，通过反射的方式在IOC容器中创建Bean，所以要求Bean中必须有无参数的构造器。
>
> id：标识容器中的bean，id唯一。若id没有指定，Spring自动将全限定类名作为Bean的名字。（如：com.chance.spring.beans.HelloWorld）


