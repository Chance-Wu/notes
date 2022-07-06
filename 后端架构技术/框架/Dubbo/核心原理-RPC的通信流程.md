### 一、RPC的通信流程

---

> Remote Procedure Call，远程过程调用
>
> - **屏蔽远程调用跟本地调用的区别**，感觉就像调用项目内的方法；
> - **隐藏底层网络通信的复杂性**，更专注于业务逻辑。

PRC常用于业务系统之间的数据交互，需要保证其可靠性。==一般默认采用tcp协议，grpc采用http2==。

#### RPC流程

![RPC流程图](%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-RPC%E7%9A%84%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B.assets/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA546L6IOW5rO9,size_20,color_FFFFFF,t_70,g_se,x_16.png)

AOP技术采用**动态代理**，通过字节码增强对方法进行拦截增强，以便于增加需要的额外处理逻辑。

用在rpc场景：

由服务提供者给出业务接口声明，在调用方的程序里面，RPC框架根据调用的服务接口提前生成==动态代理实现类==，并通过依赖注入等技术注入到声明了该接口的相关业务逻辑里面。

该代理实现类会==拦截所有的方法调用==，在提供的方法处理逻辑里面完成一整套的远程调用，并把远程调用结果返回给调用方，这样调用方在调用远程方法的时候就获得了像调用本地接口一样的体验。

#### PRC在架构中的位置

PRC框架能够解决系统拆分后的通信问题，并且能像调用本地一样去调用远程方法。



### 二、协议：可扩展并且向后兼容的协议

---

浏览器收到命令后会封装一个请求，并把请求发送到DNS解析出来的IP上，通过抓包工具可以抓到请求的数据包。

![image-20220704213051585](%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-RPC%E7%9A%84%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B.assets/image-20220704213051585.png)

#### 协议的作用

**只有二进制才能在网络中传输**，所以RPC请求在发送到网络中之前，需要把方法调用的请求参数转成二进制；转成二进制后，写入本地 Socket 中，然后被网卡发送到网络设备中。

在传输过程中，RPC并不会把请求参数的所有二进制数据整体一下子发送到对端机器上，中间可能会拆分成好几个数据包，也可能会合并其他请求的数据包（合并的前提是同一个TCP连接上的数据），对于服务提供方应用来说，会从TCP通道里面收到很多的二进制数据，服务提供方通过”断句“标示请求数据的结束位置。

#### 为什么不用HTTP？

RPC更多的是负责应用间的通信，所以性能要求相对更高。

- HTTP协议的数据包大小相对请求数据本身要大很多，有需要加入很多无用的内容，比如换行符、回车符等；
- HTTP协议属于==无状态协议==，客户端无法对请求和响应进行关联，每次请求都需要重新建立连接，响应完成后再关闭连接。

#### 如何设计一个私有RPC协议呢

PC每次发送请求发的大小都是不固定的，所以协议必须能让接收方正确地读出==不定长的内容==。

先固定一个长度（比如4个字节）用来保存整个请求数据大小。

收到数据的时候，先读取固定长度的位置里面的值，值的大小就代表协议体的长度，接着再根据值的大小来读取协议体的数据，整个协议可以设计成这样：

![](%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-RPC%E7%9A%84%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B.assets/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA546L6IOW5rO9,size_20,color_FFFFFF,t_70,g_se,x_16-20220704231618678.png)

- `协议头`是由一堆固定的长度参数组成。协议长度、序列化方式，还会放一些像协议标示、消息ID、消息类型这样的参数。
- `协议体`是根据请求接口和参数构造的，长度属于可变的，一般只放请求接口方法、请求的业务参数值和一些扩展属性。

![](%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-RPC%E7%9A%84%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B.assets/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA546L6IOW5rO9,size_20,color_FFFFFF,t_70,g_se,x_16-20220704235841798.png)

#### 可扩展的协议

保持能平滑地升级改造前后的协议。

让==协议头支持可扩展==，扩展后协议头的长度就不能定长了。

那要实现读取不定长的协议头里面的内容，在这之前肯定需要一个固定的地方读取长度，所以需要一个固定的写入协议头的长度。整体协议就变成了三部分内容：==固定部分、协议头内容、协议体内容==，前两部分还是可以统称为“协议头”，具体协议如下：

![](%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-RPC%E7%9A%84%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B.assets/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA546L6IOW5rO9,size_20,color_FFFFFF,t_70,g_se,x_16-20220705001116435.png)

#### 思考

RPC不直接使用HTTP协议的一个原因是**无法实现请求跟响应关联**，每次请求都需要重新建立连接，响应完成后再关闭连接，所以要设计私有协议。

> 在PRC里面怎么实现请求跟响应关联的呢？
>
> 官方解答:
> 为什么要把请求与响应关联?
> 这是因为在 RPC 调用过程中，调用端会向服务端发送请求消息，之后它还会收到服务端发送回来的响应消息，但这两个操作并不是同步进行的。
> 在高并发的情况下，调用端可能会在某一时刻向服务端连续发送很多条消息之后，才会陆续收到服务端发送回来的各个响应消息，这时调用端需要一种手段来区分这些响应消息分别对应的是之前的哪条请求消息，所以说 RPC 在发送消息时要请求跟响应关联。
>
> 只要调用端在收到响应消息之后，从响应消息中读取到一个标识，告诉调用端，这是哪条请求消息的响应消息就可以了。私有协议都会有消息 ID，这个消息 ID 的作用就是起到请求跟响应关联的作用。**调用端为每一个消息生成一个唯一的消息 ID，它收到服务端发送回来的响应消息如果是同一消息 ID，那么调用端就可以认为，这条响应消息是之前那条请求消息的响应消息**。

以Dubbo为例，消费者发送请求时，使用AtomicLong自增，产生一个消息ID。由于Dubbo底层IO操作是异步的，Dubbo发送请求之后，需要==阻塞等待消息者返回信息==。消费者会将消息ID保存到Map结构中。

为了保证请求响应可以一一对应，这就需要提供者返回的响应信息带上请求者消息ID。通过响应的消息ID，通过上面提到Map存储数据，就能找到对应的请求。（参考Dubbo 2.6 `DefaultFuture` 源码）



### 三、序列化：对象在网络中传输

---

网络传输的数据必须是二进制数据，但调用方请求的出入参数都是对象。对象是不能直接在网络中传输的，所以需要提前把它转成可传输的二进制，并且要求转换算法是可逆的，这个过程一般叫做“`序列化`”。

![对象在网络中传输](%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-RPC%E7%9A%84%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B.assets/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA546L6IOW5rO9,size_20,color_FFFFFF,t_70,g_se,x_16-20220705155633640.png)

#### RPC通信流程图

<img src="%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-RPC%E7%9A%84%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B.assets/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA546L6IOW5rO9,size_20,color_FFFFFF,t_70,g_se,x_16-20220705160019112.png" alt="RPC通信流程图"  />

#### JDK原生序列化

序列化具体的实现是由 `ObjectOutputStream` 完成的，而反序列化的具体实现是由 `ObjectInputStream` 完成的。

```java
FlyPig flyPig = new FlyPig();
flyPig.setColor("black");
flyPig.setName("naruto");
flyPig.setCar("0000");
// ObjectOutputStream 对象输出流，将 flyPig 对象存储到E盘的 flyPig.txt 文件中，完成对 flyPig 对象的序列化操作
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(new File("/Users/chenyang/Desktop/test/flyPig.txt")));
oos.writeObject(flyPig);
oos.close();
```

![](%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-RPC%E7%9A%84%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B.assets/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA546L6IOW5rO9,size_20,color_FFFFFF,t_70,g_se,x_16-20220705163441441.png)

序列化过程就是**在读取对象数据的时候，不断加入一些特殊分隔符，这些特殊分隔符用于在反序列化过程中截断用**。

- 头部数据用来声明序列化协议、序列化版本，用于高低版本向后兼容
- 对象数据主要包括类名、签名、属性名、属性类型及属性值，当然还有开头结尾等数据，除了属性值属于真正的对象值，其他都是为了反序列化用的元数据
- ==存在对象引用、继承的情况下，就是递归遍历“写对象”逻辑==

实际上任何一种序列化框架，核心思想就是设计一种序列化协议，将对象的类型、属性类型、属性值一一按照固定的格式写到二进制字节流中来完成序列化，再按照固定的格式一一读出对象的类型、属性类型、属性值，通过这些信息重新创建出一个新的对象，来完成反序列化。

#### JSON

json序列化的两个问题：

1. JSON 进行序列化的**额外空间开销比较大**，对于大数据量服务这意味着需要巨大的内存和磁盘开销；
2. **JSON 没有类型**，但像 Java 这种强类型语言，需要通过反射统一解决，所以性能不会太好。

所以如果 RPC 框架选用 JSON 序列化，服务提供者与服务调用者之间传输的数据量要相对较小，否则将严重影响性能。

#### Hessian

Hessian是动态类型、二进制、紧凑的，并且可跨语言移植的一种序列化框架。比JDK、JSON更加紧凑，性能上要比JDK、JSON序列化高效很多，而且生成的字节数也更小。所以Hessian更加适合作为RPC框架远程通信的序列化协议。

```java
FlyPig flyPig = new FlyPig();
flyPig.setColor("black");
flyPig.setName("naruto");
flyPig.setCar("0000");

// 把flyPig对象序列化为byte数组
ByteArrayOutputStream bos = new ByteArrayOutputStream();
Hessian2Output output = new Hessian2Output(bos);
output.writeObject(flyPig);
output.flushBuffer();
byte[] data = bos.toByteArray();
bos.close();

// 把刚才序列化出来的byte数组转化为student对象
ByteArrayInputStream bis = new ByteArrayInputStream(data);
Hessian2Input input = new Hessian2Input(bis);
FlyPig deFlyPig = (FlyPig) input.readObject();
input.close();

System.out.println(deFlyPig);
```

Hessian本身也有问题，官方版本对Java里面一些常见对象的类型不支持，比如：

- Linked系列，LinkedHashMap、LinkedHashSet等，但是可以通过扩展CollectionDeserializer类修复；
- Local类，可以通过扩展 ContextSerializerFactory 类修复。
- Byte/Short 反序列化的时候变成 Integer。

#### Protobuf

Protobuf 是 Google 公司内部的混合语言数据标准，是一种轻便、高效的结构化数据存储格式，可以用于结构化数据序列化，支持 Java、Python、C++、Go 等语言。

Protobuf 使用的时候需要定义 IDL（Interface description language），然后使用不同语言的 IDL 编译器，生成序列化工具类，它的优点是：

- 序列化后体积相比 JSON、Hessian 小很多；
- IDL 能清晰地描述语义，所以足以帮助并保证应用程序之间的类型不会丢失，无需类似 XML 解析器；
- 序列化反序列化速度很快，不需要通过反射获取类型；
- 消息格式升级和兼容性不错，可以做到向后兼容。

Protobuf 对于具有反射和动态能力的语言来说，这样用起来很费劲，这一点就不如 Hessian，
比如用 Java 的话，这个预编译过程不是必须的，可以考虑使用 Protostuff。

Protostuff 不需要依赖 IDL 文件，可以直接对 Java 领域对象进行反 / 序列化操作，在效率上跟 Protobuf 差不多，生成的二进制格式和 Protobuf 是完全相同的，可以说是一个 Java 版本的 Protobuf 序列化框架。一些不支持的情况：

- 不支持 null；(旧版本)
- ProtoStuff 不支持单纯的 Map、List 集合对象，需要包在对象里面。

#### RPC 框架中如何选择序列化

需要考虑：
性能 / 效率 / 空间开销 / 序列化协议的通用性和兼容性 / 序列化协议的安全性

![在这里插入图片描述](%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-RPC%E7%9A%84%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B.assets/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA546L6IOW5rO9,size_20,color_FFFFFF,t_70,g_se,x_16-20220705233406310.png)

首选的还是 Hessian 与 Protobuf，因为他们在性能、时间开销、空间开销、通用性、兼容性和安全性上，都满足了要求。

-  Hessian 在==使用上更加方便，在对象的兼容性上更好==；
- Protobuf 则更加高效，==通用性上更有优势==。

#### RPC框架在使用时要注意哪些问题

1. 对象构造得过于复杂，对象嵌套对象

   对象要尽量简单，没有太多的依赖关系，属性不要太多，尽量高内聚。

2. 对象过于庞大

   入参对象与返回值对象体积不要太大，更不要传太大的集合。

3. 使用序列化框架不支持的类作为入参类

   尽量使用简单的、常用的、开发语言原生的对象，尤其是集合类。比如 Hessian 框架，天然是不支持 LinkHashMap、LinkedHashSet 等，而且大多数情况下最好不要使用第三方集合类，如 Guava 中的集合类，很多开源的序列化框架都是优先支持编程语言原生的对象。

4. 对象有复杂的继承关系

   对象不要有复杂的继承关系，最好不要有父子类的情况。



### 四、网络通信：RPC框架更倾向于哪种IO模型

---

RPC是解决`进程间通信`的一种方式。

服务调用者通过网络IO发送一条请求消息，服务提供者接收并解析，处理完相关的业务逻辑之后，再发送一条响应消息给服务调用者，服务调用者接收并解析响应消息，处理完相关的响应逻辑，一次RPC调用便结束了。

#### 常见的网络IO模型

- 同步阻塞IO（BIO）
- 同步非阻塞IO（NIO）
- IO多路复用（IO multiplexing）
- 异步非阻塞IO（AIO）

#### 为什么说阻塞IO和IO多路复用最为常用

在网络 IO 的应用需要考虑

- 系统内核的支持
  大多数系统内核都会支持阻塞 IO、非阻塞 IO 和 IO 多路复用，但像信号驱动 IO、异步 IO，只有高版本的 Linux 系统内核才会支持
- 编程语言的支持
  无论 C++ 还是 Java，在高性能的网络编程框架的编写上，大多数都是基于 Reactor 模式，其中最为典型的便是 Java 的 Netty 框架，而 Reactor 模式是基于 IO 多路复用的。当然，在非高并发场景下，同步阻塞 IO 是最为常见的

这两种 IO 模型，已经可以满足绝大多数网络 IO 的应用场景。

#### RPC框架在网络通信上倾向选择哪种网络IO模型

- ==IO 多路复用更适合高并发的场景==，可以用较少的进程（线程）处理较多的 socket 的 IO 请求，但使用难度比较高。当然高级的编程语言支持得还是比较好的，比如 Java 语言有很多的开源框架对 Java 原生 API 做了封装，如 Netty 框架，使用非常简便。
- 阻塞 IO 与 IO 多路复用相比，阻塞 IO 每处理一个 socket 的 IO 请求都会阻塞进程（线程），但使用难度较低。==在并发量较低、业务逻辑只需要同步进行 IO 操作的场景下，阻塞 IO 已经满足了需求==，并且不需要发起 select 调用，开销上还要比 IO 多路复用低。

>PRC调用在大多数情况下，是一个`高并发调用`的场景，考虑到系统内核的支持、编程语言的支持已经IO模型本身的特点，在PRC框架的实现中，在网络通信的处理上，会选择多路复用的方式。
>
>开发语言的网络通信框架的选型上，最优的选择是基于 Reactor 模式实现的框架，如 Java 语言，首选的框架便是 Netty 框架（Java 还有很多其他 NIO 框架，但目前 Netty 应用得最为广泛），并且在 Linux 环境下，也要`开启 epoll 来提升系统性能`（Windows 环境下是无法开启 epoll 的，因为系统内核不支持）

#### 零拷贝（Zero-copy）

系统内核处理IO操作分为两个阶段：

1. 等待数据，就是系统内核在等待网卡接收到数据后，把数据写到内核中；
2. 拷贝数据，就是系统内核在获取到数据后，

















































