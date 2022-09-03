我们沿用惯例，仍然称他们`checked exception（CE）`和`runtime exception（RE）`。

对应的，try..catch..对于CE和RE其实也是完全不同的概念（虽然在JVM层的实现上使用了相同的机制）。从type theory角度讲，Java其实重载了try..catch..这个概念（即用于实现两种完全不同的语法）。

我们先看一下union type是什么。

**Union type是用来表示一个对象要么是A这个type的，要么是B这个type的**。比如定义个一个Union type `File | IOError`，表示这个对象要么是一个File，要么是一个IOError。 

我们该怎么知道它到底是什么呢？这就需要用到pattern matching。我们可以借用switch的语法这样理解pattern matching：

```java
File|IOError result = FileUtil.open("somefile.txt");
switch (result) {
  case File file:
    //....
  case IOError err:
    //....
}
```

Java虽然不支持union type，但它的**CE其实就是union type的一种特殊实现**。比如我们完全可以这样定义：

```java
public class FileUtil {
  // 想象中的union type的写法
  public File|IOError open(String filename);
  // Java的写法
  public File open(String filename) throws IOError;
}
```

而对于CE的try..catch..其实就是模式匹配的一种语法糖。

之所以我们强调CE是union type，是因为我们想要有力地阐述下面这个事实：

**union type是type signature的一部分，描述着函数的行为**，即CE本身就是**正常执行流**的一部分，是这个函数**预期**的行为。打开文件就是会**正常地**失败。查找HashMap里的key就是会**正常地**找不到——这些都是人们预料之内会发生的事，是自然而然的事情，也是library designer和user之间的一个约定（interface）。

如果不提供union type（或者不提供CE），人们会用更差的方法错误处理，导致更加严重的问题。比如使用tuple(File, IOError)当作函数返回值。tuple的含义就是这个函数可以同时给你一个File和IOError，也可以既不给你File也不给你IOError。或者更严重地，使用null表示失败，从而导致各种NPE。

而Java CE的真正问题可能是：try..catch..的写法太繁琐了，导致人们没有办法好好地利用这个功能，能不用CE就不用CE，甚至把本该用CE表示的东西用其他方法来写。而且try..catch..也很难写得很type-safe，因为它是语句不是expression。也许写成以下这样的expression style，人们对CE的想法会好一点：

```java
// 想象中的CE（用raises与RE的throws相区分）
public class HashMap {
  public Item get(String key) raises NotFound;
  public void put(String key, Item value);
}

Item item = map.get("mykey") onraise NotFound {
  map.put("mykey", initialItem);
  cont initialItem;     // 赋值操作 item <- initialItem
};
```



>- 对于基础设施团队，需要使用Checked Exception，并且定义良好的异常继承体系，并且认真处理所有的异常。有限度的使用RuntimeException。
>
>- 对于业务团队，基于RuntimeException定义了一个BizException来描述各种业务问题，这个BizExcpetion包含了错误码和错误描述信息；同时定义InteralServerError来包装各种系统错误，比如网络超时等。在输出http response时，二者的输出不同，然后定义不同的监控和报警机制。
>
>- 针对特定的异常，由产品和技术共同商讨介入来设计
>
>- - 哪些可以完全不管（比如一个不关键的数据拿不到），软处理；
>  - 哪些要前端用户知晓和处理（比如登录时用户名或密码错误）；
>  - 哪些由程序尽量自己处理（比如关注某个产品超时，后端要尝试重试几次）