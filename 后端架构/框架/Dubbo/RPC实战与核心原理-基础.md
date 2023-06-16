### 一、核心原理：RPC的通信流程

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
2. 拷贝数据，就是系统内核在获取到数据后，将数据拷贝到用户进程的空间中；

![在这里插入图片描述](%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-RPC%E7%9A%84%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B.assets/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA546L6IOW5rO9,size_20,color_FFFFFF,t_70,g_se,x_16-20220706094556401.png)

应用进程的每一次`写操作`，都会把数据写到用户空间的缓冲区中，再由 CPU 将数据拷贝到系统内核的缓冲区中，之后再由 DMA 将这份数据拷贝到网卡中，最后由网卡发送出去。一次写操作数据要`拷贝两次`才能通过网卡发送出去，而用户进程的读操作则是将整个流程反过来，数据同样会拷贝两次才能让应用程序读取到数据。

应用进程的一次完整的读写操作，都需要在用户空间与内核空间中来回拷贝，并且每一次拷贝，都需要 CPU 进行一次上下文切换（由用户进程切换到系统内核，或由系统内核切换到用户进程），这样很浪费 CPU 和性能。

零拷贝（Zero-copy） — 可以减少进程间的数据拷贝，提高数据传输的效率。零拷贝就是取消用户空间与内核空间之间的数据拷贝操作，应用进程每一次的读写操作，都可以通过一种方式，==让应用进程向用户空间写入或者读取数据，就如同直接向内核空间写入或者读取数据一样==，再通过 DMA 将内核中的数据拷贝到网卡，或将网卡中的数据 copy 到内核。

用户空间与内核空间都将数据写到一个地方，就不需要拷贝 --> 虚拟内存。

零拷贝有两种解决方式：

- `mmap+write` 方式
  通过虚拟内存来解决
- `sendfile` 方式

#### Netty中的零拷贝

- 操作系统层面上的零拷贝
  主要目标是避免用户空间与内核空间之间的数据拷贝操作，可以提升 CPU 的利用率。
- Netty 的零拷贝
  站在了用户空间上，也就是 JVM 上，它的零拷贝主要是偏向于数据操作的优化上。

#### Netty这么做的意义

在传输过程中，RPC 并不会把请求参数的所有二进制数据整体一下子发送到对端机器上，中间可能会拆分成好几个数据包，也可能会合并其他请求的数据包，所以**消息都需要有边界**。那么一端的机器收到消息之后，就需要对数据包进行处理，根据边界对数据包进行分割和合并，最终获得一条完整的消息。

收到消息后，==对数据包的分割和合并，是在用户空间完成的==，不是在内核空间完成的

因为对数据包的处理工作都是由应用程序来处理的，这里可能会存在数据的拷贝操作，当然不是在用户空间与内核空间之间的拷贝，是用户空间内部内存中的拷贝处理操作。Netty 的零拷贝就是为了解决这个问题，在==用户空间对数据操作进行优化==。

那么 Netty 是怎么对数据操作进行优化的呢？

- Netty 提供了 CompositeByteBuf 类，它可以将多个 ByteBuf 合并为一个逻辑上的 ByteBuf，避免了各个 ByteBuf 之间的拷贝。
- ByteBuf 支持 slice 操作，因此可以将 ByteBuf 分解为多个共享同一个存储区域的 ByteBuf，避免了内存的拷贝。
- 通过 wrap 操作，可以将 byte[] 数组、ByteBuf、ByteBuffer 等包装成一个 Netty ByteBuf 对象, 进而避免拷贝操作。

Netty 框架中很多内部的 ChannelHandler 实现类，都是**通过 CompositeByteBuf、slice、wrap 操作来处理 TCP 传输中的拆包与粘包问题的**。

那么 Netty 有没有解决用户空间与内核空间之间的数据拷贝问题的方法呢？

Netty 的 ByteBuffer 可以采用 Direct Buffers，==使用堆外直接内存进行 Socket 的读写操作==，最终的效果与刚才讲解的虚拟内存所实现的效果是一样的。

Netty 还提供 FileRegion 中包装 NIO 的 FileChannel.transferTo() 方法实现了零拷贝，这与 Linux 中的 sendfile 方式在原理上也是一样的。

#### 总结

RPC 框架在网络通信的处理上，更倾向选择 IO 多路复用的方式

零拷贝带来的好处就是避免没必要的 CPU 拷贝，让 CPU 解脱出来去做其他的事，同时也减少了 CPU 在用户空间与内核空间之间的上下文切换，从而提升了网络通信效率与应用程序的整体性能

Netty 的零拷贝与操作系统的零拷贝是有些区别的，==Netty 的零拷贝偏向于用户空间中对数据操作的优化，这对处理 TCP 传输中的拆包粘包问题有着重要的意义，对应用程序处理请求数据与返回数据也有重要的意义==。

合理使用 ByteBuf 子类，做到完全零拷贝，提升 RPC 框架的整体性能。

#### 思考

IO多路复用分为select，poll和epoll，文中描述的应该是select的过程，nigix，redis等使用的是epoll

目前很多的主流的需要通信的中间件都差不多都实现了零拷贝，如Kfaka，RocketMQ等。
kafka的零拷贝是通过java.nio.channels.FileChannel中的transferTo方法来实现的，transferTo方法底层是基于操作系统的sendfile这个system call来实现的。



### 五、动态代理：面向接口编程，屏蔽PRC处理流程

---

不需要改动原有代码的前提下，还能实现非业务逻辑跟业务逻辑的解耦。

核心：采用`动态代理技术`，通过对字节码进行增强，在方法调用的时候进行拦截，以便于在方法调用前后，增加需要的额外处理逻辑。

#### 远程调用

RPC 会自动给接口生成一个代理类，当在项目中注入接口的时候，运行过程中实际绑定的是这个`接口生成的代理类`。
这样在接口方法被调用的时候，它实际上是被生成代理类拦截到了，这样就可以在生成的代理类里面，加入远程调用逻辑。

![在这里插入图片描述](%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-RPC%E7%9A%84%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B.assets/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA546L6IOW5rO9,size_20,color_FFFFFF,t_70,g_se,x_16-20220706101951021.png)

帮用户屏蔽远程调用的细节，实现像调用本地一样地调用远程的体验。

#### 实现原理

```java
/** 要代理的接口 */
interface Hello {
  String say();
}

/** 真实调用对象 */
class RealHello {
  public String invoke() {
    return "i'm proxy";
  }
}

/** JDK代理类生成 */
class JDKProxy implements InvocationHandler {
  private Object target;

  JDKProxy(Object target) {
    this.target = target;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] paramValues) {
    return ((RealHello) target).invoke();
  }
}

/** 测试例子 */
public class TestProxy {

  public static void main(String[] args) {
    // 构建代理器
    JDKProxy proxy = new JDKProxy(new RealHello());
    ClassLoader classLoader = ClassLoaderUtils.getCurrentClassLoader();
    // 把生成的代理类保存到文件
    System.setProperty("jdk.proxy.ProxyGenerator.saveGeneratedFiles", "true");
    // 生成代理类
    Hello test = (Hello) Proxy.newProxyInstance(classLoader, new Class[] {Hello.class}, proxy);
    // 方法调用
    System.out.println(test.say());
  }
}
```

给 Hello 接口生成一个动态代理类，并调用接口 say() 方法，真实返回的值是来自 RealHello 里面的 invoke() 方法返回值。

> Proxy.newProxyInstance

生成代理类的流程：

![在这里插入图片描述](%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-RPC%E7%9A%84%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B.assets/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA546L6IOW5rO9,size_20,color_FFFFFF,t_70,g_se,x_16-20220706150811618.png)

在生成字节码的那个地方，也就是 `ProxyGenerator.generateProxyClass()` 方法里面。通过代码可以看到，里面是用参数 saveGeneratedFiles 来控制是否把生成的字节码保存到本地磁盘。同时为了更直观地了解代理的本质，需要把参数 saveGeneratedFiles 设置成 true，但这个参数的值是由 key 为“jdk.proxy.ProxyGenerator.saveGeneratedFiles”的 Property 来控制的，动态生成的类会保存在工程根目录下的 jdk/proxy1 目录里面。

反编译 $Proxy0.class 文件：

```java
package jdk.proxy1;

import com.chance.reflection.proxy.Subject;
import java.lang.invoke.MethodHandles;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class C$Proxy0 extends Proxy implements Subject {
  private static final Method m0;
  private static final Method m1;
  private static final Method m2;
  private static final Method m3;
  private static final Method m4;

  public C$Proxy0(InvocationHandler invocationHandler) {
    super(invocationHandler);
  }

  public final int hashCode() {
    ?? intValue;
    try {
      try {
        intValue = ((Integer) ((Proxy) this).h.invoke(this, m0, null)).intValue();
        return intValue;
      } catch (Error | RuntimeException unused) {
        throw intValue;
      }
    } catch (Throwable th) {
      throw new UndeclaredThrowableException(th);
    }
  }

  public final boolean equals(Object obj) {
    ?? booleanValue;
    try {
      try {
        booleanValue = ((Boolean) ((Proxy) this).h.invoke(this, m1, new Object[]{obj})).booleanValue();
        return booleanValue;
      } catch (Throwable th) {
        throw new UndeclaredThrowableException(th);
      }
    } catch (Error | RuntimeException unused) {
      throw booleanValue;
    }
  }

  public final String toString() {
    ?? r0;
    try {
      try {
        r0 = (String) ((Proxy) this).h.invoke(this, m2, null);
        return r0;
      } catch (Error | RuntimeException unused) {
        throw r0;
      }
    } catch (Throwable th) {
      throw new UndeclaredThrowableException(th);
    }
  }

  public final void rent() {
    ?? invoke;
    try {
      try {
        invoke = ((Proxy) this).h.invoke(this, m3, null);
      } catch (Error | RuntimeException unused) {
        throw invoke;
      }
    } catch (Throwable th) {
      throw new UndeclaredThrowableException(th);
    }
  }

  public final void hello(String str) {
    ?? invoke;
    try {
      try {
        invoke = ((Proxy) this).h.invoke(this, m4, new Object[]{str});
      } catch (Error | RuntimeException unused) {
        throw invoke;
      }
    } catch (Throwable th) {
      throw new UndeclaredThrowableException(th);
    }
  }

  static {
    try {
      m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
      m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
      m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
      m3 = Class.forName("com.chance.reflection.proxy.Subject").getMethod("rent", new Class[0]);
      m4 = Class.forName("com.chance.reflection.proxy.Subject").getMethod("hello", Class.forName("java.lang.String"));
    } catch (ClassNotFoundException e) {
      throw new NoClassDefFoundError(e.getMessage());
    } catch (NoSuchMethodException e2) {
      throw new NoSuchMethodError(e2.getMessage());
    }
  }

  private static MethodHandles.Lookup proxyClassLookup(MethodHandles.Lookup lookup) throws IllegalAccessException {
    if (lookup.lookupClass() != Proxy.class || !lookup.hasFullPrivilegeAccess()) {
      throw new IllegalAccessException(lookup.toString());
    }
    return MethodHandles.lookup();
  }
}
```

可以看到 $Proxy0 类里面有一个跟 Subject 一样签名的 rent() 方法， 其中 this.h 绑定的是刚才传入的 DynamicProxy 对象， 所以当调用 Subject.rent的时候，其实它是被转发到了 DynamicProxy.invoke()。

#### 实现方法

Java 领域完成代理功能的:

- **JDK 默认的 InvocationHandler**
  单纯从代理功能上来看，JDK 默认的代理功能是有一定的局限性的，它要求==被代理的类只能是接口==。原因是**因为生成的代理类会继承 Proxy 类，但 Java 是不支持多重继承的**。
  这个限制在 RPC 应用场景里面还是挺要紧的，因为对于服务调用方来说，在使用 RPC 的时候本来就是面向接口来编程的。使用 JDK 默认的代理功能，最大的问题就是性能问题。它生成后的代理类是**使用反射来完成方法调用的**，而这种方式相对直接用编码调用来说，性能会降低，但 JDK8 及以上版本对反射调用的性能有很大的提升。

- **Javassist**

  相对 JDK 自带的代理功能，Javassist 的定位是能够操纵底层字节码，所以使用起来并不简单，要生成动态代理类恐怕是有点复杂了。但好的方面是，通过 Javassist 生成字节码，不需要通过反射完成方法调用，所以性能肯定是更胜一筹的。在使用中，要注意一个问题，通过 Javassist 生成一个代理类后，**此 CtClass 对象会被冻结起来，不允许再修改；否则，再次生成时会报错**。

- **Byte Buddy**
  Byte Buddy 则属于后起之秀，在很多优秀的项目中，像 **Spring、Jackson 都用到了 Byte Buddy 来完成底层代理**。相比 Javassist，Byte Buddy 提供了更容易操作的 API，编写的代码可读性更高。更重要的是，生成的代理类执行速度比 Javassist 更快。

区别

- 通过什么方式生成的代理类
- 在生成的代理类里面是怎么完成的方法调用

#### 动态代理框架选型

- 因为==代理类是在运行中生成的==，那么代理框架生成代理类的速度、生成代理类的字节码大小等等，都会影响到其性能——生成的字节码越小，运行所占资源就越小。
- 生成的代理类，是用于接口方法请求拦截的，所以==每次调用接口方法的时候，都会执行生成的代理类==，这时生成的代理类的执行效率就需要很高效。
- 从使用角度出发，肯定希望选择一个使用起来很方便的代理类框架，比如可以考虑：API 设计是否好理解、社区活跃度、还有就是依赖复杂度等等。

#### 思考

如果没有动态代理完成方法调用拦截，用户该怎么完成RPC调用。

>官方:
>参考下 gRPC 框架。
>gRPC 框架中就没有使用动态代理，它是**通过代码生成的方式生成 Service 存根**，这个 Service 存根起到的作用和 RPC 框架中的动态代理是一样的。
>gRPC 框架用代码生成的 Service 存根来代替动态代理主要是为了实现多语言的客户端，因为有些语言是不支持动态代理的，比如 C++、go 等，但缺点也是显而易见的。
>如果使用过 gRPC 这种代码生成 Service 存根的方式与动态代理相比还是很麻烦的，并不如动态代理的方式使用起来方便、透明。

### 六、PRC实战：剖析gRPC源码，动手实现一个完整的RPC

---

gRPC 有很多特点，比如跨语言，通信协议是基于标准的 HTTP/2 设计的，序列化支持 PB（Protocol Buffer）和 JSON，整个调用示例如下图所示：

gRPC调用示例图

![在这里插入图片描述](%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-RPC%E7%9A%84%E9%80%9A%E4%BF%A1%E6%B5%81%E7%A8%8B.assets/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA546L6IOW5rO9,size_20,color_FFFFFF,t_70,g_se,x_16-20220707005834775.png)

#### 通过Protocal Buffer定义接口

```protobuf
// 指定版本
syntax = "proto3";
//定义包名
package com.chance.basis.serialization.protoc;

option java_multiple_files = true;
//生成的包名
option java_package = "io.grpc.hello";
//生成的java名，需要注意一点的是proto文件和生成的Java文件名称不能一致!
option java_outer_classname = "HelloProto";
option objc_class_prefix = "HLW";

service HelloService{
  rpc Say(HelloRequest) returns (HelloReply) {}
}

//定义数据结构
message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

`protoc --java_out=/Users/chenyang/code-space/java-basis/src/main/proto/protoc Hello.proto`
