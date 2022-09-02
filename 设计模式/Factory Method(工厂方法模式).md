> 既然都有了构造函数，何必再折腾这么多事情呢？

### 一、构造函数的作用

---

最早出现的C，创建资源差不多要这么干：

```c
some_struct * p = (some_struct*)malloc(sizeof(some_struct));
init_some_struct(p);
do_something(p);
```

即先分配内存，再做类型转换，再初始化，然后使用。而在OOP的时代，创建一个对象是很频繁的事情。同时，**一个没有初始化的数据结构是无法使用的**。因此，构造函数被发明出来，将分配内存+初始化合并到了一起。如C++的语法是：

```c++
SomeClz *p = new SomeClz();
do_something(p); 
// or
p.do_something_else();
```

java也沿用了这个设计。但是，整个构造函数完成的工作从更高层的代码设计角度还是太过于初级。因此**复杂的创建逻辑还是需要写代码来控制**。所以还是需要：

```java
SomeClz * createSomeClz(...) {
  // 做一些逻辑
  SomeClz *p = new SomeClz(); // 或者复用已经有的对象
  // 再做一些额外的初始化
  return p;
} 
```

这就是Factory的雏形。



### 二、Factory要解决的问题

---

希望能够创建一个对象，但**创建过程比较复杂**，希望对外隐藏这些细节。

#### 2.1 例1

创建对象可能是一个pool里的，不是每次都凭空创建一个新的。而pool的大小等参数可以用另外的逻辑去控制。比如连接池对象，**线程池对象**就是个很好的例子。

#### 2.2 例2

对象代码的作者希望**隐藏对象真实的的类型**，而构造函数一定要真实的类名才能用。比如作者提供了

```java
abstract class Foo { 
 //...
}
```

而真实的实现类是：

```java
public class FooImplV1 extends Foo {
  // ...
}
```

但他不希望你知道FoolImplV1的存在（没准下次就改成V2了），只希望你知道Foo，所以必须提供某种类似于这样的方式供调用：

```java
Foo foo = FooCreator.create();
// do something with foo ...
```

#### 2.3 例3

对象创建时会**有很多参数来决定如何创建出这个对象**。比如你有一个数据写在文件里，可能是xml也可能是json。这个文件的数据可以变成一个对象，大概就可以搞成。

```java
Foo foo = FooCreator.fromFile("/path/to/the/data-file.ext");
```

再比如这个文件是描述一个可以显示在浏览器的UI的基础数据。而不同浏览器可以正确显示的需要的数据不太一样。这个“不一样”可以表达为：

```java
Foo foo = FooCreator.fromFile("/path/to/the/data-file.ext", BrowserType.CHROME);
```

这里第二个参数"BrowserType"是一个枚举，表示如何去生成指定要求的对象。所以这个fromFile内部可能是：

```java
public Foo fromFile(String path, BrowserType type) {
  byte[] bytes = Files.load(path);
  switch (type) {
    case CHROME: return new FooChromeImpl(bytes);
    case IE8: return new FooIE8V1Impl(bytes);
      // ...
  }
}
```

当然，实际场景可能会复杂得多，会有大量的配置参数。

```java
Foo foo = FooCreator.fromFile("....", param1, param2, param3, ...);
```

如果需要，可以帮params弄成一个Config对象。而如果这个Config对象也很复杂，也许还得给Config弄个Factory。如果Factory本身的创建也挺复杂呢？嗯，弄个Factory的Factory。

#### 2.4 例4

简化一些常规的创建过程。上面可以看到根据配置去创建一个对象也很复杂。但可能95%的情况我们就创建某个特定类型的对象。这时可以弄个函数直接省略那些配置过程。纯粹就是为了方便。

比如Java的线程池相关创建API就是这么干的，如下：

```java
Executors.newFixedThreadPool(10)；

public static ExecutorService newFixedThreadPool(int nThreads) {
  return new ThreadPoolExecutor(nThreads, nThreads,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>());
}
```

#### 2.5 例5

**创建一个对象有复杂的依赖关系**，比如Foo对象的创建依赖A，A又依赖B，B又依赖C……。于是创建过程是一组对象的的创建和注入。手写太麻烦了。所以要把创建过程本身做很好地维护。Spring IoC就是这么干的。

#### 2.6 例6

你知道怎么创建一个对象，但是**无法把控创建的时机**。你需要把“如何创建”的代码塞给“负责什么时候创建”的代码。后者在适当的时机，就回调创建的函数。

在支持用函数传参的语言，比如js，go等，直接塞创建函数就行了。对于`名词王国java`，就得搞个XXXXFactory的类再去传。Spring IoC 也利用了这个机制，可以了解下`FactoryBean`。

#### 2.7 例7

避免在构造函数中抛出异常。"构造函数里不要抛出异常"这条原则很多人都知道。不在这里展开讨论。但问题是，业务要求必须在这里抛一个异常怎么办？就像上面的`Foo`要求从文件读出来数据并创建对象。但如果文件不存在或者磁盘有问题读不出来都会抛异常。因此用`FooCreator.fromFile`这个工厂来搞定异常这件事。 

要点是，当你有任何复杂的的创建对象过程时，你都需要写一个某种createXXXX的函数帮你实现。再拓展一下范围，哪怕创建的不是对象，而是任何资源，也都得这么干。一句话：

不管你用什么语言，创建什么资源。当你开始为“创建”本身写代码的时候，就是在使用“工厂模式”了。

具体形式可以根据当时的场景去调整，不管你用的是静态函数，抽象类还是模版等，那都是细节。不同语言的支持也不太一样。比如Java这方面就略微土一些，函数不是一等公民限制了表达力。所以你会看到各种XXXXFactory，AbstractXXXXFactory的类。

kotlin提倡用静态工厂方法解决一部分问题，即给一个class的companion object做一个表示工厂的函数。在Effective Koltin第一条就是这个。

```kotlin
interface ImageReader {
  fun read(file: File): Bitmap

  companion object {
    // 提供静态工厂方法
    fun newImageReader(format: String) = when (format) {
      "jpg" -> JpegReader()
      "gif" -> GifReader()
      else -> throw IllegalStateException("Unknown format")
    }
  }
}

// 使用静态工厂方法
val reader = ImageReader.newImageReader("jpg")
Bitmap bitmap = reader.read(someFile)
```

面对go，一般用一个函数去创建一个初始化好的对象（或者叫struct？）。go的想法很简单：反正你总是要写一个函数，就写函数吧，不要搞出那么多幺蛾子概念。

```go
type SomeStruct struct {
  // ...
}

func NewSomeStruct() *SomeStruct {
  s := SomeStruct{...}
  // 做一些初始化
  return &s
}
```