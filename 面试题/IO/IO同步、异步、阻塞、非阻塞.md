[参考文章](https://blog.csdn.net/qq_28613081/article/details/106008016)



以下讨论背景是`Linux`环境下的`nework IO`



#### 1. 5种IO模型

---

>- 阻塞IO
>- 非阻塞IO
>- 多路复用IO
>- 信号驱动IO（不常用，忽略）
>- 异步IO



#### 2. IO发生时涉及的对象和步骤

---

>对于一个network IO（以read举例），涉及到两个系统对象：调用这个IO的进程（或线程）；系统内核（kernel）。当一个read操作发生时，经历两个阶段：
>
>- ==等待数据准备==
>- ==将数据从内核拷贝到进程中==
>
>IO模型的区别就在上面两个阶段的不同。



#### 3. 阻塞IO

---

==BIO的特点就是在IO执行的两个阶段都被阻塞了。==

在linux中，默认情况下所有的socket都是blocking，一个典型的读操作流程大概是这样：

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gqas1ism3mj30i20a5glt.jpg" style="zoom:60%">

- 当用户进行调用了recvfrom这个系统调用，kernel就开始了IO的第一个阶段：==准备数据==；
- 这个时候kernel需要等待足够的数据到来，而用户进程这边，整个进程会被阻塞；
- 数据准备好了之后，就会将数据从kernel中拷贝到用户内存，然后kernel返回结果，用户进程才解除阻塞状态，重新运行。



#### 4. 非阻塞IO

---

==用户进程需要不断地主动询问kernel数据好了没。==

linux下，可以通过设置socket使其变为non-blocking。当对一个non-blocking socket执行读操作时，流程是这个样子：

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gqas8xi868j30h00dxgm4.jpg" style="zoom: 50%;">

- 当用户进程发出read操作时，如果kernel中的数据还没有准备好，那么它并不会block用户进程，而是==立刻返回一个error==。
- 用户进程收到error时，它就知道数据还没有准备好，于是可以再次发送read操作。一旦kernel中的数据准备好了，并且又再次收到用户进程的system call，那么它马上就将数据拷贝到用户内存，然后返回。



#### 5. IO 多路复用

---

`select`/`epoll`的好处就在于==单个process就可以同时处理多个网络连接的IO==。

原理：select / epoll这个方法会不断地轮询所有socket，当某个socket有数据到达了，就通知用户进程。

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gqatxol1tlj30hb09r0sz.jpg" style="zoom:50%">

- 当用户进程调用了select，那么整个进程会被block；
- 同时，kernel会监视所有select负责的socket，当任何一个socket中的数据准备好了，select就会返回。这个时候用户进程再调用read操作，将数据从kernel拷贝到用户进程。



#### 6. 异步IO

---

linux下的asynchronous IO其实用得很少。

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gqaxsvctpdj30hs09kglt.jpg" style="zoom:50%">

- 用户进程发起read操作之后，立刻就可以开始去做其他的事。而另一方面，从kernel的角度，当它收到有个asynchronous read之后，首先它会立刻返回，所以不会对用户进程产生任何block。
- kernel会等待数据准备完成，将数据拷贝到用户内存。<u>一切都完成之后，kernel会给用户发送一个信号，告诉它read操作完成了</u>。

==由于Linux只支持文件AIO，不支持网络AIO。==



#### 7. BIO和NIO的区别

---

- 阻塞IO会一直阻塞注对应的进程知道操作完成；
- 非阻塞IO在kernel还在准备数据的情况下会立刻返回。



#### 8. 同步IO和异步IO

---

- 同步IO：阻塞IO、非阻塞IO、IO多路复用（IO操作的时候会将进程阻塞）

- 异步IO