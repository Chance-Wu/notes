### 一、定义

---

观察者模式又叫发布-订阅，定义对象间一种一对多的依赖关系，使得每当一个对象改变状态，则所有依赖于它的对象都会得到通知并自动更新。

这种设计模式的思想应用比较广泛，比如在Redis的主从服务器中，当从服务器完成了各种步骤进入与主服务器命令传播的步骤之后，每次主服务器接收到一次外部命令写入，都会把该命令广播给所有的从服务器，背后的实现本质都是观察者的设计思路。

这种**一对多的关系**就是观察者模式，其中观察者是各个从服务器，被观察者是主服务器。



### 二、模型

---

![观察者模式的结构图](img/7ee58a37b8879ead9a792b8fa4620282.gif)

#### 2.1 抽象主题（Subject）角色

指被观察的对象，又称为主题，定义了一个观察者集合，一个观察目标可以接受任意数量的观察者来观察，它提供一系列方法来增加和删除观察者对象。

#### 2.2 具体主题（Concrete Subject）角色

也叫具体目标类，它实现抽象目标中的通知方法，当具体主题的内部状态发生改变时，通知所有注册过的观察者对象。

#### 2.3 抽象观察者（Observer）角色

是一个抽象类或接口，它包含了一个更新自己的抽象方法，当接到具体主题的更改通知时被调用。

#### 2.4 具体观察者（Concrete Observer）角色

实现抽象观察者中定义的抽象方法，以便在得到目标的更改通知时更新自身的状态。



### 三、实现

---

#### 3.1 创建被观察者

被观察者至少有三个方法：添加监听者、删除监听者和通知监听者

```java
public interface Subject {

  void registerObserver(Observer observer);
  void removeObserver(Observer observer);
  void notifyAllObserver();

  abstract void doSomething();
}
```

#### 3.2 创建具体被观察者

```java
public class ConcreteSubject implements Subject {

  /**
   * 观察者数组
   */
  private List<Observer> observers = new ArrayList<>();

  @Override
  public void registerObserver(Observer observer) {
    observers.add(observer);
  }

  @Override
  public void removeObserver(Observer observer) {
    observers.remove(observer);
  }

  @Override
  public void notifyAllObserver() {
    for (Observer observer : observers) {
      observer.update();
    }
  }

  @Override
  public void doSomething() {
    System.out.println("具体目标发生改变");
    this.notifyAllObserver();
  }
}
```

#### 3.3 创建观察者

观察者至少有一个更新方法

```java
public interface Observer {

  void update();
}
```

#### 3.4 创建具体观察者

具体观察者1

```java
public class ConcreteObserver1 implements Observer {
  @Override
  public void update() {
    System.out.println("观察者1收到消息，进行处理");
  }
}
```

具体观察者2

```java
public class ConcreteObserver2 implements Observer {
  @Override
  public void update() {
    System.out.println("观察者2收到消息，进行处理");
  }
}
```



### 四、优缺点

---

#### 4.1 优点

1. 主题与观察者之间松耦合

   观察者模式在被观察者和观察者之间建立一个抽象的耦合。

2. 支撑广播通信

   被观察者会向所有的登记过的观察者发出通知，一条消息可以通知给多个人。

#### 4.2 缺点

1. 目标与观察者之间的依赖关系并没有完全解除，而且有可能出现循环引用。
2. 如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。
3. 当观察者对象很多时，通知的发布会花费很多时间，影响程序的效率。
4. 如果采用顺序通知，当某个观察者卡住了，其他的观察者将无法接收到通知。



### 五、应用场景

---

1. 对一个对象状态的更新，需要其他对象同步更新，而且其他对象的数量动态可变。
2. 系统存在事件多级触发时。
3. 对象仅需要将自己的更新通知给其他对象而不需要知道其他对象的细节。

#### 5.1 spring的发布订阅

基于同步的观察者模式：简单来说就是将所有的监听者注册到一个列表里面，然后当发布事件时，通过循环里面调用每个监听者的onEvent方法，每个监听者实现的在onEvent方法里面判断传入的event是够属于当前需要的event，属于就处理该事件，反之不处理。

spring的`ApplicationEventMulticaster`就是观察者顶层接口。

```java
public interface ApplicationEventMulticaster {

  void addApplicationListener(ApplicationListener<?> listener);

  void addApplicationListenerBean(String listenerBeanName);

  void removeApplicationListener(ApplicationListener<?> listener);

  void removeApplicationListenerBean(String listenerBeanName);

  void removeAllListeners();

  void multicastEvent(ApplicationEvent event);

  void multicastEvent(ApplicationEvent event, @Nullable ResolvableType eventType);

}
```

`ApplicationListener` 就是监听者顶层接口

```java
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {

  void onApplicationEvent(E event);

}
```

在`refresh`方法里面调用的`registerListeners();`方法就是将所有的监听者实现类注册到观察者的注册表中

```java
protected void registerListeners() {
  // Register statically specified listeners first.
  for (ApplicationListener<?> listener : getApplicationListeners()) {
    getApplicationEventMulticaster().addApplicationListener(listener);
  }

  // Do not initialize FactoryBeans here: We need to leave all regular beans
  // uninitialized to let post-processors apply to them!
  String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
  for (String listenerBeanName : listenerBeanNames) {
    getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
  }

  // Publish early application events now that we finally have a multicaster...
  Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
  this.earlyApplicationEvents = null;
  if (!CollectionUtils.isEmpty(earlyEventsToProcess)) {
    for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
      getApplicationEventMulticaster().multicastEvent(earlyEvent);
    }
  }
}
```

`ApplicationEventMulticaster`的`multicastEvent`方法就是上面讲的通知方法，这里就是循环监听者注册表，调用每个监听者的onApplicationEvent方法（这里的`invokeListener`方法里面最终会调用到`listener.onApplicationEvent(event);`）

```java
@Override
public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
  ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
  Executor executor = getTaskExecutor();
  for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
    if (executor != null) {
      executor.execute(() -> invokeListener(listener, event));
    }
    else {
      invokeListener(listener, event);
    }
  }
}
```

随便看一个`onApplicationEvent`方法的实现

```java
@Override
public void onApplicationEvent(ApplicationEvent event) {
  if (failFast && event instanceof ContextRefreshedEvent) {
    this.sqlSessionFactory.getConfiguration().getMappedStatementNames();
  }
}
```

#### 5.2 nacos

nacos中有很多地方都使用到了观察者模式，如client端和server端建立连接，发布连接事件，相关监听者做相应的处理，断开连接也是一样。

在server端接收到client端的注册请求后，会发布一个注册事件的通知。



在nacos-server启动的时候也是会开启一个线程做死循环，循环的去queue里面take数据，如果没有的话就会阻塞。所以死循环只有在queue里面一直有数据的时候才会一直循环，当queue里面没有数据的时候就会阻塞在`queue.take();`方法处。



我们看看`receiveEvent(event);`方法里面做了什么，这里就体现了框架里面设计的精妙：在上面我们自己的设计中，这里应该是需要循环调用所有的listener的`onApplicationEvent`方法，但是当注册表中listener太多的时候就会出现（有些event可能会有多个listener需要处理）循环调用太慢的问题，这里使用多线程的处理方式，让这些调用并行处理，大大的提高了框架的事件处理效率。

#### 5.3 业务使用场景

可以说观察者模式能解决的，消息队列也可以解决，并且可以做的更好。主要根据实际情况取舍。

**考虑使用消息队列的情况**

当公司的服务器资源充足，并且用户量大，相关业务逻辑调用频繁，消息要求高可靠性，以及消息要求发布订阅更灵活，就可以考虑使用消息队列。

**考虑使用观察者模式的情况**

当服务器资源不充足，或者调用比较少，或者希望使用轻量的通知机制，对于消息可靠性要求不高，可以考虑在项目代码里面使用观察者模式。 当然使用观察者模式比较麻烦的一点就是要自己写一定量的代码，而且功能还不如消息队列的强大，并且不能保证消息的可靠性，当观察者获取消息在自己的处理逻辑里面产生异常时，可能还需要自己先写好发生异常后的降级代码（当然如果对可靠性要求不高的业务场景就不需要）。

**为什么框架使用观察者模式而不使用消息队列（个人理解）**：

1. 消息队列太重；

2. 本身就是开源框架（本身代表原创），不适合再引入另一个很重的消息队列。如果引入会增加用户的使用和部署成本以及难度，对于自身的推广也不利。