1、Java
--
* Apollo服务端：1.8+
* Apollo客户端：1.7+

由于Quick start会在本地同时启动服务端和客户端，所以需要在本地安装Java 1.8+

2、MySQL
--
版本要求：5.6.5+

Apollo的表结构对timestamp使用了多个default声明，所以需要5.6.5以上版本。

3、下载Quick Start安装包或手动打包Quick Start安装包
--
在本地解压apollo-quick-start.zip

修改后重新打包的步骤：

>1. 修改apollo-configservice, apollo-adminservice和apollo-portal的pom.xml，注释掉spring-boot-maven-plugin和maven-assembly-plugin
>2. 在根目录下执行mvn clean package -pl apollo-assembly -am -DskipTests=true
>3. 复制apollo-assembly/target下的jar包，rename为apollo-all-in-one.jar

3、创建数据库
--
Apollo服务端共需要两个数据库：
导入`apolloportaldb.sql`和`apolloconfigdb.sql`两个数据库脚本。

4、配置数据库连接信息
--
Apollo服务端需要知道如何连接到你前面创建的数据库，编辑demo.sh，修改==ApolloPortalDB==和==ApolloConfigDB==相关的数据库连接串信息。

```properties
#apollo config db info
apollo_config_db_url=jdbc:mysql://localhost:3306/ApolloConfigDB?characterEncoding=utf8
apollo_config_db_username=用户名
apollo_config_db_password=密码（如果没有密码，留空即可）

# apollo portal db info
apollo_portal_db_url=jdbc:mysql://localhost:3306/ApolloPortalDB?characterEncoding=utf8
apollo_portal_db_username=用户名
apollo_portal_db_password=密码（如果没有密码，留空即可）
```

5、启动Apollo配置中心
--
确保端口未被占用：Quick Start脚本会在本地启动3个服务，分别使用8070, 8080, 8090端口，请确保这3个端口当前没有被使用。

```text
lsof -i:8080
```

执行启动脚本：`./demo.sh start`

> 如遇异常，可以查看service和portal目录下的log文件排查问题。

打开 http://localhost:8070

Quick Start集成了Spring Security简单认证。

用户名`apollo`，密码`admin`

6、查看样例配置
--

点击SampleApp进入配置界面，可以看到当前有一个配置timeout=100
> 如果提示系统出错，请重试或联系系统负责人，请稍后几秒钟重试一下，因为通过Eureka注册的服务有一个刷新的延时。

7、运行客户端程序
--
准备了一个简单的Demo客户端来演示从Apollo配置中心获取配置。

程序很简单，就是用户输入一个key的名字，程序会输出这个key对应的值。

如果没找到这个key，则输出undefined。

同时，客户端还会监听配置变化事件，一旦有变化就会输出变化的配置信息。
运行`./demo.sh client`启动Demo客户端，输入timeout，看到如下信息：
```text
> timeout
Loading key : timeout with value: 100
```

8、修改配置并发布
--
* 在配置界面点击timeout这一项的修改按钮
* 在弹出框中把值改成200并提交
* 点击发布按钮，并填写发布信息

9、客户端查看修改后的值
--

如果客户端一直在运行的话，在配置发布后就会监听到配置变化，并输出修改的配置信息：
```text
> timeout
Loading key : timeout with value: 100
> Changes for namespace application
Change - key: timeout, oldValue: 100, newValue: 200, changeType: MODIFIED
```

再次输入timeout查看对应的值，会看到如下信息：
```text
> timeout
Loading key : timeout with value: 200
```

