### 一、异步RPC：压榨单机吞吐量

---

影响CPU 的利用率和服务的吞吐量。处理 RPC 请求比较耗时，RPC 本身处理请求的效率是毫秒级的，大部分都是业务耗时服务的业务逻辑，在执行较为耗时的业务逻辑的基础上，又同步调用了好几个其它的服务。例如，业务逻辑中有访问数据库执行慢 SQL 的操作，CPU 大部分时间都在等待资源。

#### 调用端如何异步

提升吞吐量 -> 异步

异步策略：

- 返回==Future对象==的Future方式（最简单），发起一次异步请求并且从请求上下文中拿到一个Future，之后就可以==调用Future的get方法获取结果==；
- 入参为Callback对象的回调方式。

RPC框架的Future方式：

对于调用端来说，向服务端发送请求消息与接收服务端发送过来的响应消息，这两个处理过程完全独立。

对于 RPC 框架，无论是同步调用还是异步调用，==调用端的内部实现都是异步的==。

调用端发送的每条消息都一个唯一的消息标识，实际上调用端向服务端发送请求消息之前会先创建一个 Future，并会存储这个==消息标识与这个 Future 的映射==，动态代理所获得的返回值最终就是从这个 Future 中获取的；

当收到服务端响应的消息时，调用端会根据响应消息的唯一标识，通过之前存储的映射找到对应的 Future，将结果注入给那个 Future，再进行一系列的处理逻辑，最后动态代理从 Future 中获得到正确的返回值。

- 同步调用，RPC 框架在调用端的处理逻辑中主动执行了这个 Future 的 get 方法，让动态代理等待返回值；
- 异步调用则是 RPC 框架没有主动执行这个 Future 的 get 方法，用户可以从请求上下文中得到这个 Future，自己决定什么时候执行这个 Future 的 get 方法。

![在这里插入图片描述](RPC%E5%AE%9E%E6%88%98%E4%B8%8E%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-%E9%AB%98%E7%BA%A7.assets/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA546L6IOW5rO9,size_20,color_FFFFFF,t_70,g_se,x_16.png)

#### 如何做到RPC调用全异步

Future 方式异步可以说是<调用端>异步。

RPC 服务端接收到请求的二进制消息之后会根据协议进行拆包解包，之后将完整的消息进行解码并反序列化，获得到入参参数之后再通过反射执行业务逻辑。

这些操作不在一个线程执行。

对二进制消息数据包拆解包的处理是一定要在处理网络 IO 的线程中，如果网络通信框架使用的是 Netty 框架，那么对二进制包的处理是在 IO 线程中，而解码与反序列化的过程也往往在 IO 线程中处理，那服务端的业务逻辑应该交给专门的业务线程池处理，以防止由于业务逻辑处理得过慢而影响到网络 IO 的处理。

==大多数情况下线程数配置到 200 还不够用就说明业务逻辑该优化。当访问量逐渐变大时，业务线程池很容易就被打满了，吞吐量很不理想，并且这时 CPU 的利用率也很低。==

服务端业务处理逻辑异步, 怎么做?

服务端执行完业务逻辑之后，要对返回值进行序列化并且编码，将消息响应给调用端，但如果是异步处理，业务逻辑触发异步之后方法就执行完了，来不及将真正的结果进行序列化并编码之后响应给调用端。

需要 RPC 框架提供一种回调方式，让业务逻辑可以异步处理，处理完之后调用 RPC 框架的回调接口，将最终的结果通过回调的方式响应给调用端。可以==让 RPC 框架支持 CompletableFuture，实现 RPC 调用在调用端与服务端之间完全异步==。

CompletableFuture 是 Java8 原生支持的。假如 RPC 框架能够支持 CompletableFuture，现在发布一个 RPC 服务，服务接口定义的返回值是 CompletableFuture 对象，整个调用过程会分为这样几步：

- 服务调用方发起 RPC 调用，直接拿到返回值 CompletableFuture 对象，之后就不需要任何额外的与 RPC 框架相关的操作了（通过请求上下文获取 Future 的操作），直接就可以进行异步处理；
- 在服务端的业务逻辑中创建一个返回值 CompletableFuture 对象，之后服务端真正的业务逻辑完全可以在一个线程池中异步处理，业务逻辑完成之后再调用这个 CompletableFuture 对象的 complete 方法，完成异步通知；
- 调用端在收到服务端发送过来的响应之后，RPC 框架再自动地调用调用端拿到的那个返回值 CompletableFuture 对象的 complete 方法，这样一次异步调用就完成了。

通过对CompletableFuture的支持，RPC框架可以真正地做到在调用端与服务端之前完全异步，同时提升了调用端与服务端的两端的单机吞吐量，并且CompletableFuture是Java8原生支持，业务逻辑中没有任何代码入侵性。

#### 总结

RPC 里面提升单机资源的利用率 — “异步化”。

- 调用方利用异步化机制实现并行调用多个服务，以缩短整个调用时间；
- 而服务提供方则可以利用异步化把业务逻辑放到自定义线程池里面去执行，以提升单机的 OPS。

影响到 RPC 调用的吞吐量的主要原因就是服务端的业务逻辑比较耗时，并且 CPU 大部分时间都在等待而没有去计算，导致 CPU 利用率不够，而提升单机吞吐量的最好办法就是使用异步 RPC。

RPC 框架的异步策略

- 调用端异步
  通过 Future 方式实现异步，调用端发起一次异步请求并且==从请求上下文中拿到一个 Future==，之后通过 Future 的 get 方法获取结果，如果业务逻辑中同时调用多个其它的服务，则可以通过 Future 的方式减少业务逻辑的耗时，提升吞吐量。
- 服务端异步
  服务端异步则需要一种回调方式，让==业务逻辑可以异步处理，之后调用 RPC 框架提供的回调接口==，将最终结果异步通知给调用端。

可以通过对 CompletableFuture 的支持，实现 RPC 调用在调用端与服务端之间的完全异步，同时提升两端的单机吞吐量。

RPC 框架其它的异步策略：

1. 集成 RxJava
2. gRPC 的 StreamObserver 入参对象

但 CompletableFuture 是 Java8 原生提供的，无代码入侵性，并且在使用上更加方便。如果是 Java 开发，让 RPC 框架支持 CompletableFuture 可以说是最佳的异步解决方案。

#### 思考

对于RPC调用远程方法提升吞吐量这个问题，是否还有其他的解决方案？还有哪些RPC框架的异步策略？

RPC这里远程方法调用方式，大致可以分成四种方式：

- **sync默认方式**，但是这只是【方法】内部同步，实际上RPC框架内部还是异步处理。
- **future方式**，RPC 消费者得到 future，自行决定何时获取返回结果
- **callback方式**，RPC调用端不需要同步处理响应结果，可以直接返回。最后返回结果将会在回调线程异步处理。
- **oneway方式**，调用端发送请求之后不需要接受响应。

其中 Dubbo 2.7 之后的版本，使用 CompletableFuture 提升异步的处理的能力，支持以上四种方式。



### 二、安全体系：如何建立可靠的安全体系

---

安全问题：SQL 注入、XSS 攻击 or 更广义的网络安全、信息安全。

RPC 是解决应用间互相通信的框架，而应用之间的远程调用过程一般不会暴露在公网，相对于公网环境，局域网的隔离性更好，也就相对更安全，所以在 RPC 里面我们很少考虑像**数据包篡改**、**请求伪造**等恶意行为。

RPC应用流程

- 服务提供方：一个接口的对外发布
  1. 定义好一个接口，并把这个接口的 Jar 包发布到私服上去
  2. 然后在项目中去实现这个接口
  3. 最后通过 RPC 提供的 API 把这个接口和其对应的实现类完成对外暴露如果是 Spring 应用的话直接定义成一个 Bean
- 服务调用方：
  1. 拿到刚才上传到私服上的 Jar 的坐标，就可以把发布到私服的 Jar 引入到项目中来。
  2. 然后借助 RPC 提供的动态代理功能，直接就可以在项目完成 RPC 调用了。

#### 问题描述

私服上所有的 Jar 坐标所有人都可以看到，只要拿到了 Jar 的坐标，就可以将其引入到项目中完成 RPC 调用。调用方**不经报备调用服务**， 服务提供方承担的调用量会变大, 可能会造成隐患。

#### 调用方之间的安全保证

给每个调用方设定一个唯一的身份：

- 登记过的调用方才能继续放行
- 没有登记过的调用方一律拒绝

授权平台：

- 调用方在授权平台上申请自己应用里面要调用的接口
- 服务提供方则可以在授权平台上进行审批，只有审批后调用方才能调用

只是解决了调用数据收集的问题，并没有完成真正的授权认证功能 – 缺少"检票"环节。

集中式认证："检票"环节在授权平台上。

![在这里插入图片描述](RPC%E5%AE%9E%E6%88%98%E4%B8%8E%E6%A0%B8%E5%BF%83%E5%8E%9F%E7%90%86-%E9%AB%98%E7%BA%A7.assets/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA546L6IOW5rO9,size_20,color_FFFFFF,t_70,g_se,x_16-20220801143329930.png)

好处：实现功能, 且整个认证过程对 RPC 使用者透明。
瓶颈：授权平台承担了公司内所有 RPC 请求的次数总和。

认证的逻辑放到业务请求过程中。可以减少授权平台的压力，但本质并没有发生变化 – 还是集中式的授权平台。

调用方能不能调用相关接口，由服务提供方决定。"检票"过程放到服务提供方
在调用方启动初始化接口的时候，带上授权平台上颁发的身份去服务提供方认证下，当认证通过后就认为这个接口可以调用。

但是服务提供方验票的时候对照的数据来自哪儿?

如果请求授权平台，又会造成瓶颈。

加密算法，不可逆加密算法，其中一种具体实现HMAC

服务提供方应用里面放一个用于 HMAC 签名的私钥，在授权平台上用这个私钥为申请调用的调用方应用进行签名，这个签名生成的串就变成了调用方唯一的身份。

服务提供方在收到调用方的授权请求之后，只要需要验证下这个签名跟调用方应用信息是否对应得上就行了，这样集中式授权的瓶颈也就不存在了。