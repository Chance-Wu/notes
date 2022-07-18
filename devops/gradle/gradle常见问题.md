#### 1. Tomcat找不到web模块的问题

问题描述：Gradle作为编译器，导致了开启服务器时，Tomcat找不到web模块，就会导致开启服务器时报错`XXXX.war not found for the web module `

>原因：
>
>Gradle编译器不会给你自动创建exploded目录，但是Tomcat找的时候是去那个目录找的，所以改成使用IDEA编译就行了。
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gmqkd2s1gbj31460juaax.jpg" style="zoom:60%">







