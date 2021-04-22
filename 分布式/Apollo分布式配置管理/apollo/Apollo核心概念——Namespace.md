#### 1. 什么是Namespace

`Namespace`：配置项的集合，类似于一个配置文件的概念。

#### 2. 什么是“application”的Namespace

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gogw7mxsy7j31gq070glr.jpg" style="zoom:80%">

==Apollo在创建项目的时候，都会默认创建一个“application”的Namespace==。“application”是给应用自身使用的，SpringBoot项目都有一个默认配置文件application.yml。就等同于”application“的Namespace。

>客户端获取“application”Namespace的代码：
>
>```java
>Config config = ConfigService.getConfig();
>```
>
>客户端获取非“application”Namespace的代码：
>
>```java
>Config config = ConfigService.getConfig(namespaceName);
>```

#### 3. Namespace的格式有哪些









































