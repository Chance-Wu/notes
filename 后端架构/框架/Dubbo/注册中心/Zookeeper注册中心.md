Zookeeper是一个==树型的目录服务==，==支持变更推送==，适合作为Dubbo服务的注册中心，工业强度较高，可用于生产环境，并推荐使用。

<img src="https://dubbo.apache.org/imgs/user/zookeeper.jpg" style="zoom:100%">

流程说明：

- 服务提供者启动时：向/dubbo/com.foo.BarService/providers目录下写入自己的URL地址。
- 服务消费者启动时：订阅 `/dubbo/com.foo.BarService/providers` 目录下的提供者 URL 地址。并向 `/dubbo/com.foo.BarService/consumers` 目录下写入自己的 URL 地址。
- 监控中心启动时: 订阅 `/dubbo/com.foo.BarService` 目录下的所有提供者和消费者 URL 地址。

支持以下功能：

- 当提供者出现断电等异常停机时，注册中心能自动删除提供者信息
- 当注册中心重启时，能自动恢复注册数据，以及订阅请求
- 当会话过期时，能自动恢复注册数据，以及订阅请求
- 当设置 `<dubbo:registry check="false" />` 时，记录失败注册和订阅请求，后台定时重试
- 可通过 `<dubbo:registry username="admin" password="1234" />` 设置 zookeeper 登录信息
- 可通过 `<dubbo:registry group="dubbo" />` 设置 zookeeper 的根节点，不配置将使用默认的根节点。
- 支持 `*` 号通配符 `<dubbo:reference group="*" version="*" />`，可订阅服务的所有分组和所有版本的提供者

#### 1. 使用

在provider和consumer中增加zookeeper客户端jar包依赖：

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.3.3</version>
</dependency>
```

Dubbo支持zkclient和`curator`两种Zookeeper客户端实现（2.7.x版本已经移除了zkclient的实现。，如果要使用zkclient客户端，需要自行拓展）

#### 2. 使用curator客户端

curator是一个Zookeeper客户端实现。

配置：

```xml
<dubbo:registry ... client="curator" />
```

或：

```sh
dubbo.registry.client=curator
```

或：

```sh
zookeeper://10.20.153.10:2181?client=curator
```

需依赖：

```xml
<dependency>
    <groupId>com.netflix.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>1.1.10</version>
</dependency>
```

##### 2.1 Zookeeper单机配置

```yaml
dubbo:
  registry:
    address: zookeeper://10.20.153.10:2181
```

或：

```xml
dubbo:
  registry:
    protocol: zookeeper
    address: //10.20.153.10:2181
```

##### 2.2 Zookeeper集群配置

```yaml
dubbo:
  registry:
    address: zookeeper://10.20.153.10:2181?backup=10.20.153.11:2181,10.20.153.12:2181
```

或：

```yaml
dubbo:
  registry:
    protocol: zookeeper
    address: 10.20.153.10:2181?backup=10.20.153.11:2181,10.20.153.12:2181
```

同一 Zookeeper，分成多组注册中心:

```xml
<dubbo:registry id="chinaRegistry" protocol="zookeeper" address="10.20.153.10:2181" group="china" />
<dubbo:registry id="intlRegistry" protocol="zookeeper" address="10.20.153.10:2181" group="intl" />
```

##### 2.3 Zookeeper安装

Dubbo 未对 Zookeeper 服务器端做任何侵入修改，只需安装原生的 Zookeeper 服务器即可，所有注册中心逻辑适配都在调用 Zookeeper 客户端时完成。

安装：

```sh
wget http://archive.apache.org/dist/zookeeper/zookeeper-3.3.3/zookeeper-3.3.3.tar.gz
tar zxvf zookeeper-3.3.3.tar.gz
cd zookeeper-3.3.3
cp conf/zoo_sample.cfg conf/zoo.cfg
```

配置：

```sh
vi conf/zoo.cfg
```

如果不需要集群，`zoo.cfg` 的内容如下：

```fallback
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/dubbo/zookeeper-3.3.3/data
clientPort=2181
```

如果需要集群，`zoo.cfg` 的内容如下：

```fallback
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/home/dubbo/zookeeper-3.3.3/data
clientPort=2181
server.1=10.20.153.10:2555:3555
server.2=10.20.153.11:2555:3555
```

并在 data 目录下放置 myid 文件：

```sh
mkdir data
vi myid
```

myid 指明自己的 id，对应上面 `zoo.cfg` 中 `server.` 后的数字，第一台的内容为 1，第二台的内容为 2，内容如下：

```fallback
1
```

启动：

```sh
./bin/zkServer.sh start
```

停止：

```sh
./bin/zkServer.sh stop
```

命令行：

```sh
telnet 127.0.0.1 2181
dump
```

或者：

```shell
echo dump | nc 127.0.0.1 2181
```

用法：

```xml
dubbo.registry.address=zookeeper://10.20.153.10:2181?backup=10.20.153.11:2181
```

或者：

```xml
<dubbo:registry protocol="zookeeper" address="10.20.153.10:2181,10.20.153.11:2181" />
```

#### 3. 可靠性声明

阿里内部并没有采用 Zookeeper 做为注册中心，而是使用自己实现的基于数据库的注册中心。此 Zookeeper 桥接实现只为开源版本提供，其可靠性依赖于 Zookeeper 本身的可靠性。

#### 4. 兼容性声明

因 `2.0.8` 最初设计的 zookeeper 存储结构不能扩充不同类型的数据，`2.0.9` 版本做了调整，所以不兼容，需全部改用 `2.0.9` 版本才行，以后的版本会保持兼容 `2.0.9`。`2.2.0` 版本改为基于 zkclient 实现，需增加 zkclient 的依赖包，`2.3.0` 版本增加了基于 curator 的实现，作为可选实现策略。