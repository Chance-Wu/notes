[参考文章](https://www.jianshu.com/p/8d9a6ae40303)

参考tomcat的Catalina类



#### 1. digester简介

解析XML的工具类库。

`digester`底层是基于==SAX+事件驱动+栈==的方式来搭建实现的。那么在digester中，这三种元素分别起什么作用呢？

1. SAX：用于解析XML
2. 事件驱动：在SAX解析的过程中加入事件来支持我们的对象映射。
3. 栈：当解析xml元素的开始和结束的时候，需要通过xml元素映射的类对象的入栈和出栈来完成事件的调用。

通过一些实实在在的场景和例子，我们发现一个元素的作用无非是在其解析前后加入一些扩展逻辑！例如：

- 开始解析某个节点的时候，是否需要创建一个类
- 开始解析某个节点的时候，是否需要入栈操作
- 结束解析某个节点的时候，是否需要执行某个方法
- 结束解析某个节点的时候，是否需要出栈操作

#### 2. 使用

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-digester3</artifactId>
    <version>3.2</version>
</dependency>
```

加入需要解析的如下xml文件：

```xml
<?xml version='1.0' encoding='utf-8'?>
<School name="Jen">
    <Grade name="1">
        <Class name="1" number="31"/>
        <Class name="2" number="32"/>
    </Grade>
    <Grade name="2">
        <Class name="1" number="41"/>
        <Class name="2" number="42"/>
        <Class name="3" number="37"/>
    </Grade>
</School>
```

同时假设下面的约定成立：

- 一个学校有名字属性，下面有多个年级
- 每个年级有名字属性，下面有多个班
- 每个班有名字和学生人数两个属性

根据上面的规则，创建关联的3个类，School、Grade、Class。





























