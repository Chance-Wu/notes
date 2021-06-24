AMQP中的消息路由过程和JMS存在一些差别，AMQP中增加了`Exchange`和`Binding`的角色。生产者把消息发布到Exchange上，消息最终达到队列并被消费者接收，而Binding决定交换器的消息应该发送到哪个队列。



### 1. Exchange类型

---

>Exchange分发消息时根据类型的不同分发策略有区别，主要有四种类型：
>
>- direct
>- fanout
>- topic
>- headers（和direct交换器完全一致，但性能差很多，几乎不用）



#### 1.1 direct——直连交换机

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1grc380mjopj30xc0hc751.jpg" style="zoom: 50%;" />

>消息中的`routing key`如果和binding中的bingding key一致，交换器就将消息发到对应的队列中。
>
>如果一个队列绑定到交换机要求的路由键为“MEMEBER_ADD_ROUTING_KEY”，则只转发routing key标记为“MEMEBER_ADD_ROUTING_KEY”的消息。它是完全匹配，==单播的模式==。
>
>适用场景：有优先级的任务，根据任务的优先级把消息发送到对应的队列，这样可以指派更多的资源去处理高优先级的队列。



#### 1.2 fanout——扇形交换机

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1grc3clen0uj30xc0h0q3c.jpg" style="zoom: 50%;" />

>==广播消息==，扇形交换机会把能接收到的消息全部发送给绑定在自己身上的队列。扇形交换机处理消息的速度是所有交换机类型里最快的。



#### 1.3 topic——主题交换机

<img src="https://upload-images.jianshu.io/upload_images/1479657-48e5409a26f0c75b.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/571/format/webp" alt="img"  />

>发送到主题交换机的routing key需要有一定的规则，交换机和队列的bingding key需要采用 `*.#.*......` 的格式，每个部分用 `.` 分开。
>
>- `*` 表示一个单词
>- `#` 表示任意数量单词
>
>假设一条消息的routing key为`fast.rabbit.white`，那么带有这样routing key的几个队列都会接收这条消息：
>
>1. fast..
>2. ..white
>3. fast.#
>
>当一个队列的绑定建为`#`的时候，这个队列将会无视消息的路由键，接收所有的消息。



#### 1.4 headers——首部交换机

>忽略routing key的一种路由方式。路由器和交换机路由的规则是通过`Headers信息`来交换的，这个有点像HTTP的Headers。将一个交换机声明成首部交换机，绑定一个队列的时候，定义一个Hash的数据结构，消息发送的时候，会携带一组hash数据结构的信息，当Hash的内容匹配上的时候，消息就会被写入队列。
>
>绑定交换机和**队列**的时候，Hash结构中要求携带一个键“`x-match`”，这个键的`Value`可以是`any`或者`all`，这代表消息携带的`Hash`是需要全部匹配(all)，还是仅匹配一个键(any)就可以了。相比直连交换机，首部交换机的优势是匹配的规则不被限定为字符串(string)。