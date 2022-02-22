- 在项目中，会遇到异常处理，对于**运行时异常**，需要我们自己==判断处理==。对于**受检异常**，需要我们==主动处理==。
- 主动处理中，繁琐的try{}catch嵌套在代码里，看着很不舒服。
- 这里不讨论性能，就代码来讲，来看看如何将它隐藏起来。原理是不变的，变的是写法。

#### 概念

---

##### 行为参数化

java8提出的，函数式编程的一种思想，通过把==代码包装为参数传递行为==，即把==代码逻辑包装为一个参数==，传到方法里。

##### Lambda表达式

Lambda表达式理解为简洁的表示==可传递匿名函数==的一种方式，它没有名称，但它有函数体，参数列表，返回类型。可以抛出一个异常类型。包装代码逻辑为参数即使用Lambda表达式。

##### 函数式接口

本质上只有一个==抽象方法==的==普通接口==，可以被隐式的转换为Lambda表达式，需要用注解定义（`@FunctionalInterface`）

>注：默认方法和静态方法虽然不属于抽象方法，但是可以在函数式接口中定义。

```java
@FunctionalInterfacepublic
interface ObjectMethodFunctionalInterface {
  void count(int i);
  String toString(); //same to Object.toString
  int hashCode(); //same to Object.hashCode
  boolean equals(Object obj); //same to Object.equals
}
```

如果函数式接口中额外定义多个抽象方法，那么这些抽象方法签名必须和 Object 的 public 方法一样，接口最终有确定的类实现， 而类的最终父类是 Object。 因此函数式接口可以定义 Object 的 public 方法。

>行为参数化是指导思想，Lambda表达式是表达方式，函数式接口是实现手法。

#### 如何隐藏

---

```java
 Class<?> clazz = Class.forName("类名");
```

以上是一个受检异常，需要抛出一个ClassNotFoundException。

正常写法：

```java
try {
  Class<?> clazzOld = Class.forName("类名");
} catch (ClassNotFoundException e) {
  e.printStackTrace();
}
```

隐藏写法：

```java
Class<?> clazzNew =classFind( o -> Class.forName(o),"类名");
```

把`Class<?> clazz = Class.forName("类名");`当做一种行为去处理，接受一个String，得到一个Class，所以定义一个函数式接口，描述这种行为，这种行为本身是需要处理受检异常的。

```java
@FunctionalInterface
public interface ClassFindInterface {
  Class<?> classNametoClass(String className)throws ClassNotFoundException;
}
```

这里，因为我们的行为需要抛出异常。所以在接口里也抛出异常。

然后，需要定义一个方法，将我们的行为作为参数传进去，同时，捕获以下我们的异常，

```java
public Class classFind(ClassFindInterface classFindInterface,String className){
  Class<?> clazz =null;
  try {
    clazz = classFindInterface.classNametoClass(className);
  } catch (ClassNotFoundException e) {
    logger4j.error(">>>>>>>>"+e.getMessage()+"<<<<<<<<");
    e.printStackTrace();
  }
  return clazz;
}
```

然后可以调用classFind方法，

```java
Class<?> clazzNew =classFind( o -> Class.forName(o),"类名");
```

#### Demo

---

文本转换为字符串：

将文本文件转换为字符串，需要使用高级流包装低级流，然后做缓存读出来。这里不可避免会遇到异常处理，流的关闭等操作，下面将这些代码都隐藏起来。专心写读的逻辑即可。

```java
FileInputStream fileInputStream = new FileInputStream(file));
InputStreamReader inputStreamReader = new InputStreamReader(fileInputStream));
BufferedReader bufferedReader = new BufferedReader(inputStreamReader));
String str = bufferedReader.readLine();
```

字节流-》字符流-》字符缓存流，即将字节流转换为字符流之后在用高级流包装。

思路：避免在逻辑里出现太多的IO流关闭，和异常捕获，专心处理逻辑即可，结合以下两种技术：

- try(){}【自动关闭流，1.7支持】
- lambda特性实现【行为参数化，1.8支持】

描述一个行为，`BufferedReader -> String`

```java
/**
 * @Description: 函数接口，描述BufferedReader -> String的转化方式
 * @Author: chance
 * @Date: 2022/2/10 2:05 下午
 * @Version 1.0
 */
@FunctionalInterface
public interface InputStreamProcess {

  /**
   * BufferedReader -> String
   * @param bufferedReader
   * @return
   * @throws IOException
   */
  String process(BufferedReader bufferedReader) throws IOException;
}
```

将一个行为，嵌入到定式里，任何BufferedReader -> String的Lambda表达式都可以作为参数传入。只要符合process方法的签名即可。

```java
public static String fileToBufferedReader(InputStreamProcess inputStreamProcess, File file) {
  String result = null;
  try (FileInputStream fileInputStream = new FileInputStream(file)) {
    try (InputStreamReader inputStreamReader = new InputStreamReader(fileInputStream)) {
      try (BufferedReader bufferedReader = new BufferedReader(inputStreamReader)) {
        result = inputStreamProcess.process(bufferedReader);
      }
    }
  } catch (IOException e) {
    e.printStackTrace();
  } finally {
    return result;
  }
}
```

使用这个定义好的行为

```java
File file = new File("/Users/chance/IdeaProjects/java-basis/src/main/java/com/chance/java8/functionalinterfaces/test.txt");
String result = fileToBufferedReader(bufferedReader -> {
  String str = null;
  StringBuilder stringBuilder = new StringBuilder();
  while ((str = bufferedReader.readLine()) != null) {
    stringBuilder.append(str);
  }
  return stringBuilder.toString();
}, file);
```