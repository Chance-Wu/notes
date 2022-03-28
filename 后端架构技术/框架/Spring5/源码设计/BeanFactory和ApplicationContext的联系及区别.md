<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gpvzugk5ubj329m0aqwer.jpg" style="zoom:120%">

>Spring中的容器类可以分为两大类：
>
>- 一类是由==BeanFactory接口==定义的核心容器。
>
>  其基本实现类为==DefaultListableBeanFactory==。之所以称其为核心容器，是因为该类容器实现IoC的核心功能（<u>配置文件的加载解析，Bean依赖的注入以及生命周期的管理等</u>）。BeanFactory面向Spring框架本身，一般不会被用户直接使用。
>
>- 另一类则是由==ApplicationContext接口==定义的容器，称为应用上下文（应用容器）。
>
>  在BeanFactory提供的核心IoC功能之上作了扩展。通常ApplicationContext的实现类内部都持有一个BeanFactory的实例，IoC容器的核心功能会交由它去完成。而ApplicationContext本身，则专注于在应用层面对BeanFactory作扩展（<u>国际化的支持，支持框架级的事件监听机制以及增加了很多对应用环境的适配等</u>）。面向Spring框架的开发者。==ClassPathXmlApplicationContext==就是典型的Spring的应用容器。

