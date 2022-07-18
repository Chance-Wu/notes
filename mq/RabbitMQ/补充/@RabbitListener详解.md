https://blog.csdn.net/u013905744/article/details/86736536

>为什么一个普通的方法加上@RabbitListener注解就能接收消息了呢？
>
>先总结来说，有一个BeanPostProcessor来处理这个注解，把注解相关的内容取出来，封装成一个RabbitListenerEndPoint。然后给每个Endpoint创建一个MessageListenerContainer，在这个container中注册一个MessageListener，在这个MessageListener中创建了一个HandlerAdapter，这个adapter与rabbitmq broker建立一个connection，接收rabbitmq broker push过来的message，放到一个blocking queue中。至此完成消息的接收。
>
>接下来是消息的处理。上文的adapter把我们用@RabbitListener注解的普通方法通过反射的方式还原出来，从blocking queue中poll出一个一个的message，进行处理。



从@RabbitListener最上面的注释，得知@RabbitListener的处理器是**RabbitListenerAnnotationBeanPostProcessor**。



#### 1. RabbitListenerAnnotationBeanPostProcessor

---

该处理器implements BeanPostProcessor，实现了postProcessAfterInitialization方法，当bean初始化完成后，执行这个方法。

```java
@Override
public Object postProcessAfterInitialization(final Object bean, final String beanName) throws BeansException {
  Class<?> targetClass = AopUtils.getTargetClass(bean);
  // 去执行buildMetadata方法，找出所有加了@RabbitListener注解的方法
  final TypeMetadata metadata = this.typeCache.computeIfAbsent(targetClass, this::buildMetadata);
  for (ListenerMethod lm : metadata.listenerMethods) {
    for (RabbitListener rabbitListener : lm.annotations) {
      processAmqpListener(rabbitListener, lm.method, bean, beanName);
    }
  }
  if (metadata.handlerMethods.length > 0) {
    processMultiMethodListeners(metadata.classAnnotations, metadata.handlerMethods, bean, beanName);
  }
  return bean;
}
```

之后执行processAmqpListener：

```java
protected void processAmqpListener(RabbitListener rabbitListener, Method method, Object bean, String beanName) {
  Method methodToUse = checkProxy(method, bean);
  MethodRabbitListenerEndpoint endpoint = new MethodRabbitListenerEndpoint();
  endpoint.setMethod(methodToUse);
  processListener(endpoint, rabbitListener, bean, methodToUse, beanName);
}
```

> 还可以通过==在类上加@RabbitListener注解，然后在方法上加@RabbitHandler注解==，如果采用这种方式会processMultiMethodListeners()方法来处理这些方法。

之后调用processListener方法，==读取@RabbitListener注解中的值，设置到endpoint中去==。获取配置的RabbitListenerContainerFactory bean，然后调用RabbitListenerEndpointRegistrar类的registerEndpoint()方法。



>RabbitListenerContainerFactory可以是默认的，也可以自定义。可以是这样在代码里面以@Bean来指定的：
>
>```java
>@Bean("autoAckContainerFactory")
>public RabbitListenerContainerFactory<SimpleMessageListenerContainer> autoAckContanierFactory(ConnectionFactory connectionFactory) {
>  SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
>  factory.setConnectionFactory(connectionFactory);
>  factory.setMessageConverter(jsonMessageConverter());
>  factory.setAcknowledgeMode(AcknowledgeMode.AUTO);
>  factory.setConcurrentConsumers(5);
>  factory.setMaxConcurrentConsumers(10);
>  return factory;
>}
>```
>
>使用时指定containerFactory
>
>```java
>@RabbitListener(
>  queues = "rabbitmq.simple.queue",
>  containerFactory = "autoAckContainerFactory"
>)
>public void onSimpleQueueMessage(Message message, Channel channel) {
>  String msg = new String(message.getBody());
>  log.info(">>>>>>>>>接收到消息：{}", msg);
>}
>```



#### 2. RabbitListenerEndpointRegistrar

---

调用RabbitListenerEndpointRegistrar类的registerEndpoint()方法。

```java
public void registerEndpoint(RabbitListenerEndpoint endpoint,
                             @Nullable RabbitListenerContainerFactory<?> factory) {
  Assert.notNull(endpoint, "Endpoint must be set");
  Assert.hasText(endpoint.getId(), "Endpoint id must be set");
  Assert.state(!this.startImmediately || this.endpointRegistry != null, "No registry available");
  // factory可能为空，在实际创建容器之前推迟解析
  AmqpListenerEndpointDescriptor descriptor = new AmqpListenerEndpointDescriptor(endpoint, factory);
  synchronized (this.endpointDescriptors) {
    if (this.startImmediately) { // 立即注册并开始
      this.endpointRegistry.registerListenerContainer(descriptor.endpoint, // NOSONAR 永远不会为空
                                                      resolveContainerFactory(descriptor), true);
    }
    else {
      this.endpointDescriptors.add(descriptor);
    }
  }
}
```

这里根据startImmediately看是否需要立刻注册endpoint，或者先将其添加到一个List，稍后统一注册。

对于统一注册的实现，RabbitListenerAnnotationBeanPostProcessor类除了实现BeanPostProcessor以外，还实现了SmartInitializingSingleton接口，所以当RabbitListenerAnnotationBeanPostProcessor这个bean实例化完成之后会调用它的afterSingletonsInstantiated()方法。

```java
@Override
public void afterSingletonsInstantiated() {
  this.registrar.setBeanFactory(this.beanFactory);

  if (this.beanFactory instanceof ListableBeanFactory) {
    Map<String, RabbitListenerConfigurer> instances =
      ((ListableBeanFactory) this.beanFactory).getBeansOfType(RabbitListenerConfigurer.class);
    for (RabbitListenerConfigurer configurer : instances.values()) {
      configurer.configureRabbitListeners(this.registrar);
    }
  }

  if (this.registrar.getEndpointRegistry() == null) {
    if (this.endpointRegistry == null) {
      Assert.state(this.beanFactory != null,
                   "BeanFactory must be set to find endpoint registry by bean name");
      this.endpointRegistry = this.beanFactory.getBean(
        RabbitListenerConfigUtils.RABBIT_LISTENER_ENDPOINT_REGISTRY_BEAN_NAME,
        RabbitListenerEndpointRegistry.class);
    }
    this.registrar.setEndpointRegistry(this.endpointRegistry);
  }

  if (this.defaultContainerFactoryBeanName != null) {
    this.registrar.setContainerFactoryBeanName(this.defaultContainerFactoryBeanName);
  }

  // Set the custom handler method factory once resolved by the configurer
  MessageHandlerMethodFactory handlerMethodFactory = this.registrar.getMessageHandlerMethodFactory();
  if (handlerMethodFactory != null) {
    this.messageHandlerMethodFactory.setMessageHandlerMethodFactory(handlerMethodFactory);
  }

  // Actually register all listeners
  this.registrar.afterPropertiesSet();

  // clear the cache - prototype beans will be re-cached.
  this.typeCache.clear();
}
```

因为之前已经将所有的endpoint添加到了RabbitListenerEndpointRegistrar类中的一个List中了，所以这里调用RabbitListenerEndpointRegistrar类的afterPropertiesSet()方法进行统一注册：

```java
@Override
public void afterPropertiesSet() {
  registerAllEndpoints();
}

protected void registerAllEndpoints() {
  Assert.state(this.endpointRegistry != null, "No registry available");
  synchronized (this.endpointDescriptors) {
    for (AmqpListenerEndpointDescriptor descriptor : this.endpointDescriptors) {
      this.endpointRegistry.registerListenerContainer(// NOSONAR never null
        descriptor.endpoint, resolveContainerFactory(descriptor));
    }
    this.startImmediately = true;  // trigger immediate startup
  }
}
```

循环，一个一个注册。



#### 3. RabbitListenerEndpointRegistry

---

注册过程：

```java
/**
	 * 为给定的 RabbitListenerEndpoint 创建一个消息侦听器容器。
	 * 这将创建必要的基础设施来保证该endpoint的配置。
	 */
public void registerListenerContainer(RabbitListenerEndpoint endpoint, RabbitListenerContainerFactory<?> factory) {
  registerListenerContainer(endpoint, factory, false);
}

public void registerListenerContainer(RabbitListenerEndpoint endpoint, RabbitListenerContainerFactory<?> factory,
                                      boolean startImmediately) {
  Assert.notNull(endpoint, "Endpoint must not be null");
  Assert.notNull(factory, "Factory must not be null");

  String id = endpoint.getId();
  Assert.hasText(id, "Endpoint id must not be empty");
  synchronized (this.listenerContainers) {
    Assert.state(!this.listenerContainers.containsKey(id),
                 "Another endpoint is already registered with id '" + id + "'");
    MessageListenerContainer container = createListenerContainer(endpoint, factory);
    this.listenerContainers.put(id, container);
    if (StringUtils.hasText(endpoint.getGroup()) && this.applicationContext != null) {
      List<MessageListenerContainer> containerGroup;
      if (this.applicationContext.containsBean(endpoint.getGroup())) {
        containerGroup = this.applicationContext.getBean(endpoint.getGroup(), List.class);
      }
      else {
        containerGroup = new ArrayList<MessageListenerContainer>();
        this.applicationContext.getBeanFactory().registerSingleton(endpoint.getGroup(), containerGroup);
      }
      containerGroup.add(container);
    }
    if (this.contextRefreshed) {
      container.lazyLoad();
    }
    if (startImmediately) {
      startIfNecessary(container);
    }
  }
}
```

> 可见，注册endpoint，实际上就是RabbitListenerContainerFactory将每一个endpoint都创建成MessageListenerContainer（具体创建过程，由RabbitListenerContainerFactory类自己去完成），然后根据startImmediately参数判断是否调用startIfNecessary()方法立即启动MessageListenerContainer。
>
> 实际接收消息是由这个MessageListenerContainer来做的，而MessageListenerContainer接口中有一个接口方法来设置MessageListener。
>
> ![](https://tva1.sinaimg.cn/large/008i3skNgy1gu9b8qwalsj61bs072jto02.jpg)



#### 4. AbstractMessageListenerContainer

---

```java
public void setMessageListener(MessageListener messageListener) {
  this.messageListener = messageListener;
}
```

```java
@FunctionalInterface
public interface MessageListener {

  /**
	  * 传递单个消息
	  */
  void onMessage(Message message);

  /**
	  * 由容器调用以通知侦听器其ack模式
	  */
  default void containerAckMode(AcknowledgeMode mode) {
    // NOSONAR - empty
  }

  /**
	  * 发送批量消息
	  */
  default void onMessageBatch(List<Message> messages) {
    throw new UnsupportedOperationException("This listener does not support message batches");
  }
}
```

这样接收并处理消息的所有工作就完成了。

如果不立即启动MessageListenerContainer，RabbitListenerEndpointRegistry也实现了SmartLifecycle接口，所以在spring context refresh的最后一步会去调用start()方法：

```java
@Override
public void start() {
  for (MessageListenerContainer listenerContainer : getListenerContainers()) {
    startIfNecessary(listenerContainer);
  }
}
```

```java
/**
	 * 启动特定的MessageListenerContainer，如果它需要在启动时被开启，或是启动后显式调用
	 */
private void startIfNecessary(MessageListenerContainer listenerContainer) {
  if (this.contextRefreshed || listenerContainer.isAutoStartup()) {
    listenerContainer.start();
  }
}
```

可以看到这里统一启动了所以的MessageListenerContainer。

所谓启动MessageListenerContainer其实就是调用MessageListenerContainer的start()方法。这也是SmartLifecycle的一个接口方法，它的实现必须保证调用了这个start()方法之后MessageListenerContainer将能够接受到消息。



#### 5. @RabbitListener实现流程总结

---

- @RabbitListener注解的方法所在的类首先是一个bean，因此，实现BeanPostProcessor接口对每一个初始化完成的bean进行处理。
- 遍历bean中由用户自定义的所有的方法，找出其中添加了@RabbitListener注解的方法（也可以是@RabbitHandler注解）
- 读取上面找出的所有方法上@RabbitListener注解中的值，并为每一个方法创建一个RabbitListenerEndpoint，保存在RabbitListenerEndpointRegistrar类中。
- 在所有的bean都初始化完成，即所有@RabbitListener注解的方法都创建了endpoint之后，==由我们配置的RabbitListenerContainerFactory将每个endpoint创建MessageListenerContainer==。
- 最后启动上面创建的MessageListenerContanier。
- 至此，全部完成，MessageListenerContainer启动后将能够接受到消息，再将消息交给它的MessageListener处理消息。

1. RabbitListenerContainerFactory只是个接口，它不会自己创建MessageListenerContainer，所以需要一个RabbitListenerContainerFactory实现类，它必须能创建MessageListenerContainer
2. MessageListenerContainer也只是一个接口，它不会自己接收消息，所以需要一个MessageListenerContainer实现类，它必须做到在启动后能够接收消息，同时它必须能设置MessageListener，用以处理消息。
3. MessageListener（或ChannelAwareMessageListener）也只是一个接口，所以还需要一个MessageListener实现类，它必须能调用我们加了@RabbitListener注解的方法，这样才实现了消息的处理

![](https://tva1.sinaimg.cn/large/008i3skNgy1gub9oh2yx2j614205k76c02.jpg)

通过SimpleRabbitListenerContainerFactory创建MessageListenerContainer。



#### 6. MessageListenerContainer的创建

---

之前的关键代码 RabbitListenerEndpointRegistry的registerListenerContainer方法MessageListenerContainer listenerContainer = factory.createListenerContainer(endpoint);

createListenerContainer跟踪进去，会进入到AbstractRabbitListenerContainerFactory的createListenerContainer方法中，其关键代码：

```java
endpoint.setupListenerContainer(instance);

initializeContainer(instance, endpoint);
```

initializeContainer这个方法里面，进入到SimpleRabbitListenerContainerFactory类中，会做一些它独有的属性设置。

```java
@Override
protected void initializeContainer(SimpleMessageListenerContainer instance, RabbitListenerEndpoint endpoint) {
  super.initializeContainer(instance, endpoint);

  JavaUtils javaUtils = JavaUtils.INSTANCE
    .acceptIfNotNull(this.batchSize, instance::setBatchSize);
  String concurrency = null;
  if (endpoint != null) {
    concurrency = endpoint.getConcurrency();
    javaUtils.acceptIfNotNull(concurrency, instance::setConcurrency);
  }
  javaUtils
    .acceptIfCondition(concurrency == null && this.concurrentConsumers != null, this.concurrentConsumers,
                       instance::setConcurrentConsumers)
    .acceptIfCondition((concurrency == null || !(concurrency.contains("-")))
                       && this.maxConcurrentConsumers != null,
                       this.maxConcurrentConsumers, instance::setMaxConcurrentConsumers)
    .acceptIfNotNull(this.startConsumerMinInterval, instance::setStartConsumerMinInterval)
    .acceptIfNotNull(this.stopConsumerMinInterval, instance::setStopConsumerMinInterval)
    .acceptIfNotNull(this.consecutiveActiveTrigger, instance::setConsecutiveActiveTrigger)
    .acceptIfNotNull(this.consecutiveIdleTrigger, instance::setConsecutiveIdleTrigger)
    .acceptIfNotNull(this.receiveTimeout, instance::setReceiveTimeout);
  if (Boolean.TRUE.equals(this.consumerBatchEnabled)) {
    instance.setConsumerBatchEnabled(true);
    instance.setDeBatchingEnabled(true);
  }
}

```

setupListenerContainer方法执行结束，MessageListener就设置到MessageListenerContainer里面去了，可以跟踪这个方法。

```java
@Override
public void setupListenerContainer(MessageListenerContainer listenerContainer) {
  AbstractMessageListenerContainer container = (AbstractMessageListenerContainer) listenerContainer;

  boolean queuesEmpty = getQueues().isEmpty();
  boolean queueNamesEmpty = getQueueNames().isEmpty();
  if (!queuesEmpty && !queueNamesEmpty) {
    throw new IllegalStateException("Queues or queue names must be provided but not both for " + this);
  }
  if (queuesEmpty) {
    Collection<String> names = getQueueNames();
    container.setQueueNames(names.toArray(new String[names.size()]));
  }
  else {
    Collection<Queue> instances = getQueues();
    container.setQueues(instances.toArray(new Queue[instances.size()]));
  }

  container.setExclusive(isExclusive());
  if (getPriority() != null) {
    Map<String, Object> args = new HashMap<String, Object>();
    args.put("x-priority", getPriority());
    container.setConsumerArguments(args);
  }

  if (getAdmin() != null) {
    container.setAmqpAdmin(getAdmin());
  }
  setupMessageListener(listenerContainer);
}
```

一直到AbstractRabbitListenerEndpoint类的下面这个方法：

```java
private void setupMessageListener(MessageListenerContainer container) {
  MessageListener messageListener = createMessageListener(container);
  Assert.state(messageListener != null, () -> "Endpoint [" + this + "] must provide a non null message listener");
  container.setupMessageListener(messageListener);
}
```

可以看到在这个方法里创建了MessageListener，并将其设置到MessageListenerContainer里面去。createMessageListener()方法有两个实现，实际调用的是MethodRabbitListenerEndpoint类里面的实现：

```java
@Override
protected MessagingMessageListenerAdapter createMessageListener(MessageListenerContainer container) {
  Assert.state(this.messageHandlerMethodFactory != null,
               "Could not create message listener - MessageHandlerMethodFactory not set");
  MessagingMessageListenerAdapter messageListener = createMessageListenerInstance();
  messageListener.setHandlerAdapter(configureListenerAdapter(messageListener));
  String replyToAddress = getDefaultReplyToAddress();
  if (replyToAddress != null) {
    messageListener.setResponseAddress(replyToAddress);
  }
  MessageConverter messageConverter = getMessageConverter();
  if (messageConverter != null) {
    messageListener.setMessageConverter(messageConverter);
  }
  if (getBeanResolver() != null) {
    messageListener.setBeanResolver(getBeanResolver());
  }
  return messageListener;
}
```

看到setHandlerMethod(configureListenerAdapter(messageListener))这一行，这里创建并设置了一个HandlerAdapter，这个HandlerAdapter能够调用我们加了@RabbitListener注解的方法。

```java
protected HandlerAdapter configureListenerAdapter(MessagingMessageListenerAdapter messageListener) {
  InvocableHandlerMethod invocableHandlerMethod =
    this.messageHandlerMethodFactory.createInvocableHandlerMethod(getBean(), getMethod());
  return new HandlerAdapter(invocableHandlerMethod);
}
```





























