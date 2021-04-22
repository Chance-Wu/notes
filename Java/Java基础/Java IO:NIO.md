**同步和异步**的概念描述的是*<u>用户线程与内核的交互方式</u>*：

- 同步是指用户线程发起IO请求后需要等待或者轮询内核IO操作完成后才能继续执行；
- 异步是指用户线程发起IO请求后仍继续执行，当内核IO操作完成后会通知用户线程，或者调用用户线程注册的回调函数。

**阻塞和非阻塞**的概念描述的是*<u>用户线程调用内核IO操作的方式</u>*：

- 阻塞是指IO操作需要彻底完成后才返回到用户空间；
- 非阻塞是指IO操作被调用后立即返回给用户一个状态值，无需等到IO操作彻底完成。

#### 1. 阻塞IO模型

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjoksrvc12j31060jbgm6.jpg" style="zoom:50%">

> 一种传统的IO模型，即在读写数据过程中会发生阻塞现象。
>
> - 当用户线程发出IO请求之后，内核会去查看数据是否就绪，==如果没有就绪就会等待数据就绪，而用户线程就会处于阻塞状态，用户线程交出CPU==。
> - 当数据就绪之后，内核会将数据拷贝到用户线程，并返回结果给用户线程，用户线程才解除block状态。

> 典型的阻塞IO模型的例子为：
>
> ```java
> data = socket.read();
> ```
>
> 如果数据没有就绪，就会一直阻塞在read方法。

#### 2. 非阻塞IO模型

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjolragpqgj30yw0jkjs7.jpg" style="zoom:50%">

> - 当用户线程发起一个read操作后，并不需要等待，而是马上就得到了一个结果。
> - 如果结果是一个error时，它就知道数据还没有准备好，于是它可以再次发送read操作。
> - 一旦内核中的数据准备好了，并且又再次收到了用户线程的请求，那么它马上就将数据拷贝到了用户线程，然后返回。
>
> 在非阻塞IO模型中，==用户线程需要不断地询问内核数据是否就绪，也就说非阻塞IO不会交出CPU，而会一直占CPU==。

> 典型的非阻塞IO模型一般如下：
>
> ```java
> while(true) {
>  data = socket.read();
>  if (data != error) {
>      // 处理数据
>      break;
>  }
> }
> ```
>
> 对于非阻塞IO有一个非常严重的问题，在while循环中需要不断地去询问内核数据是否就绪，这样会导致**CPU占用率非常高**，

#### 3. 多路复用IO模型/异步阻塞IO（使用得比较多Java NIO）

> 建立在内核提供的多路分离函数select基础之上的，使用==select函数==可以避免同步非阻塞IO模型中轮询等待的问题。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjolwwg7zgj311i0mcmy4.jpg" style="zoom:50%">

> 用户首先将需要进行IO操作的socket添加到select中，然后阻塞等待select系统调用返回。当数据到达时，socket被激活，select函数返回。用户线程正式发起read请求，读取数据并继续执行。
>
> 从流程上来看，==使用select函数进行IO请求和同步阻塞模型没有太大的区别==，甚至还多了添加监视socket，以及调用select函数的额外操作，效率更差。但是，使用select以后最大的优势是用户可以在一个线程内同时处理多个socket的IO请求。用户可以注册多个socket，然后不断地调用select读取被激活的socket，即可达到在同一==个线程内同时处理多个IO请求的目的==。而在同步阻塞模型中，必须通过多线程的方式才能达到这个目的
>
> 用户线程使用select函数的伪代码描述为：
>
> ```java
> {
>  select(socket);
>  while(1) {
>      sockets = select();
>      for(socket in sockets) {
>          if(can_read(socket)) {
>              read(socket, buffer);
>              process(buffer);
>          }
>      }
>  }
> }
> ```
>
> 其中while循环前将socket添加到select监视中，然后在while内一直调用select获取被激活的socket，一旦socket可读，便调用read函数将socket中的数据读取出来。
>
> 然而，使用select函数的优点不仅限于此。虽然上述方式允许单线程内处理多个IO请求，但是每个IO请求的过程还是阻塞的（在select函数上阻塞），平均时间甚至比同步阻塞IO模型还要长。如果用户线程只注册自己感兴趣的socket或者IO请求，然后去做自己的事情，等到数据到来时再进行处理，则可以提高CPU的利用率。

> IO多路复用模型使用了Reactor设计模式实现了这一机制：
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjqfblvhcvj31500deta6.jpg" style="zoom:60%">
>
> - EventHandler抽象类表示IO事件处理器，它拥有IO文件句柄Handle（通过get_handle获取），以及对Handle的操作handle_event（读/写等）。
> - 继承于EventHandler的子类可以对事件处理器的行为进行定制。
> - Reactor类用于管理EventHandler（注册、删除等），并使用handle_events实现事件循环，不断调用同步事件多路分离器（一般是内核）的多路分离函数select，只要某个文件句柄被激活（可读/写等），select就返回（阻塞），handle_events就会调用与文件句柄关联的事件处理器的handle_event进行相关操作。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjqwlwh3usj31g00o8gn5.jpg" style="zoom:60%">
>
> 如图所示，
>
> - 通过Reactor的方式，可以将用户线程轮询IO操作状态的工作统一交给handle_events时间循环进行处理。
> - 用户线程注册事件处理器之后可以继续执行做其他的工作（异步），而Reactor线程负责调用内核的select函数检查socket状态。
> - 当有socket被激活时，则同住相应的用户线程（或执行用户线程的回调函数），执行handle_event进行数据读取、处理的工作。
>
> 注意：阻塞是指select函数执行时线程被阻塞。一般在使用IO多路复用模型时，socket都是设置为NONBLOCK的，不过这并不会产生影响，因为用户发起IO请求时，数据已经到达了，用户线程一定不会被阻塞。
>
> ```java
> void UserEventHandler::handle_event() {
>     if(can_read(socket)) {
>         read(socket, buffer);
>         process(buffer);
>     }
> }
> 
> {
>     Reactor.register(new UserEventHandler(socket));
> }
> ```
>
> 用户需要重写EventHandler的handle_event函数进行读取数据、处理数据的工作，用户线程只需要将自己的EventHandler注册到Reactor即可。Reactor中的handle_events事件循环的伪代码大致如下。
>
> ```java
> Reactor::handle_events() {
>     while(1) { // 事件循环不断地调用select获取被激活的socket
>         sockets = select();
>         for(socket in sockets) { // 根据获取socket对应的EventHandler，执行器handle_event函数即可
>          get_event_handler(socket).handle_event();
>         }
>     }
> }
> ```

> 多路复用IO为何比非阻塞IO模型的效率高。
>
> - 在非阻塞IO中，不断地轮询socket状态是通过==用户线程==去进行的；
> - 在多路复用IO中，轮询每个socket状态是==内核==在进行的，这个效率要比用户线程高的多。

> 多路复用IO模型是通过轮询的方式来检测是否有事件到达，并且对到达的事件逐一进行响应。因此对于多路复用IO模型来说，==一旦事件响应体很大，那么就会导致后续的事件迟迟得不到处理，并且会影响新的事件轮询==。

#### 4. 信号驱动IO模型

> 在信号驱动 IO 模型中，当用户线程发起一个 IO 请求操作，会给对应的 socket 注册一个信号函数，然后用户线程会继续执行，当内核数据就绪时会发送一个信号给用户线程，用户线程接收到信号之后，便在信号函数中调用 IO 读写操作来进行实际的 IO 请求操作。

#### 5. 异步IO模型（最理想的 IO 模型）

> 在IO多路复用模型中，事件循环将文件句柄的状态事件通知给用户线程，由用户线程自行读取数据、处理数据。而在异步IO模型中，当用户线程收到通知时，数据已经被内核读取完毕，并放在了用户线程指定的缓冲区内，内核在IO完成后通知用户线程直接使用即可。

> 异步IO模型使用了Proactor设计模式实现了这一机制。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjqze89p03j30yi0bhq3u.jpg" style="zoom:60%">
>
> Proactor模式中，用户线程将AsynchronousOperation（读/写等）、Proactor以及操作完成时的CompletionHandler注册到AsynchronousOperationProcessor。AsynchronousOperationProcessor使用Facade模式提供了一组异步操作API（读/写等）供用户使用，当用户线程调用异步API后，便继续执行自己的任务。AsynchronousOperationProcessor 会开启独立的内核线程执行异步操作，实现真正的异步。当异步IO操作完成时，AsynchronousOperationProcessor将用户线程与AsynchronousOperation一起注册的Proactor和CompletionHandler取出，然后将CompletionHandler与IO操作的结果数据一起转发给Proactor，Proactor负责回调每一个异步操作的事件完成处理函数handle_event。虽然Proactor模式中每个异步操作都可以绑定一个Proactor对象，但是一般在操作系统中，Proactor被实现为Singleton模式，以便于集中化分发操作完成事件。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjqzg1kloaj31fw0mkdh3.jpg" style="zoom:60%">
>
> 异步模型中：
>
> - 用户线程直接使用内核提供的异步IO API发起read请求，且发起后立即返回，继续执行用户线程代码。
> - 不过此时用户线程已经将调用的AsynchronousOperation和CompletionHandler注册到内核，然后操作系统开启独立的内核线程去处理IO操作。
> - 当read请求的数据到达时，由内核负责读取socket中的数据，并写入用户指定的缓冲区中。
> - 最后内核将read的数据和用户线程注册的CompletionHandler分发给内部Proactor，Proactor将IO完成的信息通知给用户线程（一般通过调用用户线程祖册的完成事件处理函数），完成异步IO。
>
> ```java
> void UserCompletionHandler::handle_event(buffer) {
>  process(buffer);
> }
> 
> {
>  aio_read(socket, new UserCompletionHandler);
> }
> ```
>
> 用户需要重写CompletionHandler的handle_event函数进行处理数据的工作，参数buffer表示Proactor已经准备好的数据，用户线程直接调用内核提供的异步IO API，并将重写的CompletionHandler注册即可。

> 注意：==异步IO是需要操作系统的底层支持==，在Java 7中，提供了Asynchronous IO。

> 相比于IO多路复用模型，异步IO并不十分常用，不少高性能并发服务程序使用==IO多路复用模型+多线程任务处理==的架构基本可以满足需求。况且目前操作系统对异步IO的支持并非特别完善，更多的是采用IO多路复用模型模拟异步IO的方式（IO事件触发时不直接通知用户线程，而是将数据读写完毕后放到用户指定的缓冲区中）。

#### 6. Java IO包

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjqzqyxyyzj30u012rgts.jpg" style="zoom:60%">

>1. 基于`字节`操作的 I/O 接口：InputStream、OutputStream
>2. 基于`字符`操作的 I/O 接口：Writer、Reader
>3. 基于`磁盘`操作的 I/O 接口：File
>4. 基于`网络`操作的 I/O 接口：Socket
>
>使用的==装饰者模式==。
>
>```java
>InputStream in=new BufferedInputStream(new FileInputStream("filename"));
>```

#### 7. Java NIO

> NIO主要有三大核心部分：
>
> - ==Channel==（通道）
> - ==Buffer==（缓冲区）
> - ==Selector==（选择器）
>
> 传统IO基于字节流和字符流进行操作，而NIO基于`Channel`和`Buffer（缓冲区）`进行操作，==数据总是从通道读取到缓冲区中，或者从缓冲区写入通道中==。
>
> Selector用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjrfi8dgsuj30ms0iq0v5.jpg" style="zoom:60%">
>
> ==IO是面向流的，NIO是面向缓冲区的。==

##### 7.1 Channel

> Channel和IO中的Stream是差不多一个等级的。只不过Stream是单向的，譬如：InputStream，OutputStream，而==Channel是双向的==，既可以用来进行读操作，又可以用来进行写操作。

> NIO中Channel的主要实现有：
>
> 1. FileChannel（文件IO）
> 2. DatagramChannel（UDP）
> 3. SocketChannel（TCP，Server和Client）
> 4. ServerSocketChannel

##### 7.2 Buffer

> ==缓冲区，实际上是一个容器，是一个连续数组==。Channel提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由Buffer。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjsaq46z1fj310k08ojt7.jpg" style="zoom:60%">
>
> 上面的图描写了从一个客户端向服务端发送数据，然后服务端接收数据的过程。客户端发送数据时，必须==先将数据存入Buffer==，==然后将Buffer中的内容写入Channel==。服务端这边接收数据必须通过Channel将数据读入到Buffer中，然后再从Buffer中取出数据来处理。
>
> 在NIO中，Buffer是一个顶层父类，它是一个抽象类，常用的Buffer的子类有：
>
> ByteBuffer、IntBuffer、 CharBuffer、 LongBuffer、 DoubleBuffer、FloatBuffer、
>
> ShortBuffer

##### 7.3 Selector

> Selector 类是 NIO 的核心类，==Selector 能够检测多个注册的通道上是否有事件发生，如果有事件发生，便获取事件然后针对每个事件进行相应的响应处理==。这样一来，只是用一个单线程就可以管理多个通道，也就是管理多个连接。这样使得只有在连接真正有读写事件发生时，才会调用函数来进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程，并且避免了多线程之间的上下文切换导致的开销。

##### 7.4 NIO的非阻塞

>  NIO 的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。 线程通常将非阻塞 IO 的空闲时间用于在其它通道上执行 IO 操作，所以==一个单独的线程现在可以管理多个输入和输出通道（channel）==。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjrhqrvje8j30qc17m42l.jpg" style="zoom:60%">