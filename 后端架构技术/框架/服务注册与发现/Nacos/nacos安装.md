Nacos支持三种部署模式：

- 单机模式：用于测试和单机使用
- 集群模式：用于生产环境，确保高可用
- 多集群模式：用于多数据中心场景



#### 1. 单机部署

---

0.7版本之后支持mysql数据源能力，具体的操作步骤：

1. 安装数据库，版本要求：5.6.5+

2. 初始化mysql数据库`nacos_config`，数据库初始化文件：nacos-mysql.sql

3. 修改`conf/application.properties`文件，添加mysql数据源的url、用户名和密码。

   ```properties
   spring.datasource.platform=mysql
   
   db.num=1
   db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
   db.user.0=root
   db.password.0=539976
   ```

4. 以单机模式启动nacos：`sh startup.sh -m standalone`

访问[localhost:8848/nacos]()，用户名/密码：[nacos/nacos]()

5. 关闭nacos：`sh shutdown.sh`



#### 2. 集群模式下运行Nacos

---

因此开源的时候推荐用户把所有服务列表放到一个vip下面，然后挂到一个域名下面

[http://ip1](http://ip1/):port/openAPI 直连ip模式，机器挂则需要修改ip才可以使用。

[http://SLB](http://slb/):port/openAPI 挂载SLB模式(内网SLB，不可暴露到公网，以免带来安全风险)，直连SLB即可，下面挂server真实ip，可读性不好。

[http://nacos.com](http://nacos.com/):port/openAPI 域名 + SLB模式(内网SLB，不可暴露到公网，以免带来安全风险)，可读性好，而且换ip方便，推荐模式。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gts9rhqdwdj60dj08b0sr02.jpg)

>请确保是在环境中安装使用:
>
>1. 64 bit OS Linux/Unix/Mac，推荐使用Linux系统。
>2. 64 bit JDK 1.8+；
>3. Maven 3.2.x+；
>4. 3个或3个以上Nacos节点才能构成集群。



> 下载编译后压缩包方式
>
> [zip包](https://github.com/alibaba/nacos/releases/download/1.3.0/nacos-server-1.3.0.zip)
>
> ```shell
> unzip nacos-server-1.3.0.zip
> ```
>
> [tar.gz包](https://github.com/alibaba/nacos/releases/download/1.3.0/nacos-server-1.3.0.tar.gz)
>
> ```shell
> tar -xvf nacos-server-1.3.0.tar.gz
> ```



> 集群配置文件（在nacos的解压目录nacos/的conf目录下，有配置文件cluster.conf，配置3个或3个以上节点）：
>
> ```
> # ip:port
> 200.8.9.16:8848
> 200.8.9.17:8848
> 200.8.9.18:8848
> ```



>确定数据源
>
>- 使用内置数据源（无需配置）
>  - `sh startup.sh -p embedded`
>- 使用mysql（生产建议至少主备模式），初始化mysql数据库，修改application.properties文件
>  - `sh startup.sh`



>服务注册：`curl -X PUT 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=nacos.naming.serviceName&ip=20.18.7.10&port=8080'`
>
>服务发现：`curl -X GET 'http://127.0.0.1:8848/nacos/v1/ns/instance/list?serviceName=nacos.naming.serviceName'`
>
>发布配置：`curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test&content=helloWorld"`
>
>获取配置：`curl -X GET "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test"`



#### 3. 多集群模式

---

