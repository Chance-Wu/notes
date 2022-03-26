#### 1. 字符串的不可变性

---

>一旦一个string对象在内存(堆)中被创建出来，他就无法被修改。特别要注意的是，==String类的所有方法都没有改变字符串本身的值，都是返回了一个新的对象==。
>
>如果需要一个可修改的字符串，应该使用StringBuffer 或者 StringBuilder。否则会有大量时间浪费在垃圾回收上，因为每次试图修改都有新的string对象被创建出来。

##### 1.1 定义一个字符串

```java
String s = "abcd";
```

<img src="https://tva1.sinaimg.cn/large/e6c9d24egy1h0krykigkwj20ic0b4t8p.jpg" style="zoom:67%;" />

s中保存了string对象的引用。下面的箭头可以理解为“存储他的引用”。

##### 1.2 使用变量来赋值

```java
String s2 = s;
```

<img src="https://tva1.sinaimg.cn/large/e6c9d24egy1h0ks2smpf0j20ic0b4dfw.jpg" style="zoom:67%;" />

s2保存了相同的引用值，因为他们代表同一个对象。

##### 1.3 字符串连接

```java
s = s.concat("ef");
```

<img src="https://tva1.sinaimg.cn/large/e6c9d24egy1h0ks6mop0cj20i207rgln.jpg" style="zoom:67%;" />

s中保存的是一个重新创建出来的string对象的引用。



#### 2. JDK 6和JDK 7中substring的原理及区别

---

##### 2.1 substring()的作用

`substring(int beginIndex, int endIndex)`方法截取字符串并返回其[beginIndex,endIndex-1]范围内的内容。

```java
String x = "abcdef";
x = x.substring(1,3);
System.out.println(x);
```

输出：

```
bc
```

##### 2.2 调用substring()时发生了什么

因为x是不可变的，当使用`x.substring(1,3)`对x赋值的时候，它会指向一个全新的字符串。

>【JDK6】
>
>String是通过字符数组实现的。在jdk 6 中，String类包含三个成员变量：`char value[]`， `int offset`，`int count`。他们分别用来存储真正的字符数组，数组的第一个位置索引以及字符串中包含的字符个数。
>
>当调用substring方法的时候，会创建一个新的string对象，但是这个string的值仍然指向堆中的同一个字符数组。这两个对象中只有count和offset 的值是不同的。
>
><img src="https://tva1.sinaimg.cn/large/e6c9d24egy1h0kx6y5vqvj20i20atjrh.jpg" style="zoom:80%;" />
>
>```java
>//JDK 6
>String(int offset, int count, char value[]) {
>  this.value = value;
>  this.offset = offset;
>  this.count = count;
>}
>
>public String substring(int beginIndex, int endIndex) {
>  //check boundary
>  return  new String(offset + beginIndex, endIndex - beginIndex, value);
>}
>```
>
>问题：如果你有一个很长很长的字符串，但是当你使用substring进行切割的时候你只需要很短的一段。这可能导致性能问题，因为你需要的只是一小段字符序列，但是你却引用了整个字符串（因为这个非常长的字符数组一直在被引用，所以无法被回收，就可能导致内存泄露）。在JDK 6中，一般用以下方式来解决该问题，原理其实就是生成一个新的字符串并引用他。
>
>解决办法：`x = x.substring(x, y) + ""`

>内存泄露：在计算机科学中，内存泄漏指由于疏忽或错误造成程序未能释放已经不再使用的内存。 内存泄漏并非指内存在物理上的消失，而是应用程序分配某段内存后，由于设计错误，导致在释放该段内存之前就失去了对该段内存的控制，从而造成了内存的浪费。

>【JDK7】
>
>在jdk 7 中，substring方法会在堆内存中创建一个新的数组。
>
>![](https://tva1.sinaimg.cn/large/e6c9d24egy1h0kxem4a14j20i20atwel.jpg)
>
>```java
>//JDK 7
>public String(char value[], int offset, int count) {
>  //check boundary
>  this.value = Arrays.copyOfRange(value, offset, offset + count);
>}
>
>public String substring(int beginIndex, int endIndex) {
>  //check boundary
>  int subLen = endIndex - beginIndex;
>  return new String(value, beginIndex, subLen);
>}
>```
>
>以上是JDK 7中的subString方法，其使用`new String`创建了一个新字符串，避免对老字符串的引用。从而解决了内存泄露问题。



#### 3. replaceFirst、replaceAll、replace的区别

---

##### 3.1 replaceAll() 替换符合正则的所有文字

```java
//文字替换（全部） 
Pattern pattern = Pattern.compile("正则表达式"); 
Matcher matcher = pattern.matcher("正则表达式 Hello World,正则表达式 Hello World"); 
//替换所有符合正则的数据 
System.out.println(matcher.replaceAll("Java")); 
```

##### 3.2 replaceFirst() 替换第一个符合正则的数据

```java
//文字替换（首次出现字符） 
Pattern pattern = Pattern.compile("正则表达式"); 
Matcher matcher = pattern.matcher("正则表达式 Hello World,正则表达式 Hello World"); 
//替换第一个符合正则的数据 
System.out.println(matcher.replaceFirst("Java")); 
```

##### 3.3 replaceAll()替换所有html标签

```java
//去除html标记 
Pattern pattern = Pattern.compile("<.+?>", Pattern.DOTALL); 
Matcher matcher = pattern.matcher("<a href=\"index.html\">主页</a>"); 
String string = matcher.replaceAll(""); 
System.out.println(string); 
```

##### 3.4 replaceAll() 替换指定文字

```java
//替换指定{}中文字 
String str = "Java目前的发展史是由{0}年-{1}年";
String[][] object = {
  new String[] {
    "\\{0\\}",
    "1995"
  },
  new String[] {
    "\\{1\\}",
    "2007"
  }
};
System.out.println(replace(str, object));
public static String replace(final String sourceString, Object[] object) {
  String temp = sourceString;
  for (int i = 0; i < object.length; i++) {
    String[] result = (String[]) object[i];
    Pattern pattern = Pattern.compile(result[0]);
    Matcher matcher = pattern.matcher(temp);
    temp = matcher.replaceAll(result[1]);
  }
  return temp;
}
```

##### 3.5 replace()替换字符串

```java
System.out.println("abac".replace("a", "\a")); //\ab\ac
```



#### 4. String对"+"的重载

---

1. String s = "a" + "b"，编译器会进行常量折叠(因为两个都是编译期常量，编译期可知)，即变成 String s = "ab"
1. 对于能够进行优化的(`String s = "a" + 变量`等)用 StringBuilder 的 append() 方法替代，最后调用 toString() 方法 (底层就是一个 new String())



#### 5. 字符串拼接

---

**String是Java中一个不可变的类**，所以他一旦被实例化就无法被修改。

> 不可变类的实例一旦创建，其成员变量的值就不能被修改。这样设计有很多好处，比如可以缓存hashcode、使用更加便利以及更加安全等。

但是，既然字符串是不可变的，那么字符串拼接又是怎么回事呢？

其实，所有的所谓字符串拼接，都是重新生成了一个新的字符串。

##### 5.1 使用+拼接字符串

**Java是不支持运算符重载的**。这其实只是Java提供的一个**语法糖**。

```java
String wechat = "Hollis";
String introduce = "每日更新Java相关技术文章";
String hollis = wechat + "," + introduce;
```

反编译后：

```java
String wechat = "Hollis";
String introduce = "\u6BCF\u65E5\u66F4\u65B0Java\u76F8\u5173\u6280\u672F\u6587\u7AE0";//每日更新Java相关技术文章
String hollis = (new StringBuilder()).append(wechat).append(",").append(introduce).toString();
```

原来字符串常量在拼接过程中，是将String转成了StringBuilder后，使用其append方法进行处理的。

那么也就是说，Java中的`+`对字符串的拼接，其实现原理是使用`StringBuilder.append`。

>运算符重载：在计算机程序设计中，运算符重载（operator overloading）是多态的一种。运算符重载，就是对已有的运算符重新进行定义，赋予其另一种功能，以适应不同的数据类型。
>
>语法糖：语法糖（Syntactic sugar），也译为糖衣语法，是由英国计算机科学家彼得·兰丁发明的一个术语，指计算机语言中添加的某种语法，这种语法对语言的功能没有影响，但是更方便程序员使用。语法糖让程序更加简洁，有更高的可读性。

##### 5.2 concat

```java
String wechat = "Hollis";
String introduce = "每日更新Java相关技术文章";
String hollis = wechat.concat(",").concat(introduce);
```

源码：

```java
public String concat(String str) {
  int otherLen = str.length();
  if (otherLen == 0) {
    return this;
  }
  int len = value.length;
  char buf[] = Arrays.copyOf(value, len + otherLen);
  str.getChars(buf, len);
  return new String(buf, true);
}
```

这段代码首先创建了一个字符数组，长度是已有字符串和待拼接字符串的长度之和，再把两个字符串的值复制到新的字符数组中，并使用这个字符数组创建一个新的String对象并返回。

通过源码我们也可以看到，经过concat方法，其实是new了一个新的String。

##### 5.3 StringBuffer

Java中还提供了可以用来定义**字符串变量**的StringBuffer类，它的对象是可以扩充和修改的。

使用StringBuffer可以方便的对字符串进行拼接。如：

```java
StringBuffer wechat = new StringBuffer("Hollis");
String introduce = "每日更新Java相关技术文章";
StringBuffer hollis = wechat.append(",").append(introduce);
```

##### 5.4 StringBuilder

除了StringBuffer以外，还有一个类StringBuilder也可以使用，其用法和StringBuffer类似。如：

```java
StringBuilder wechat = new StringBuilder("Hollis");
String introduce = "每日更新Java相关技术文章";
StringBuilder hollis = wechat.append(",").append(introduce);
```

和String类类似，StringBuilder类也封装了一个字符数组，定义如下：

```java
char[] value;
```

与String不同的是，它并不是`final`的，所以他是可以修改的。另外，与String不同，字符数组中不一定所有位置都已经被使用，它有一个实例变量，表示数组中已经使用的字符个数，定义如下：

```java
int count;
```

其append源码如下：

```java
public StringBuilder append(String str) {
  super.append(str);
  return this;
}
```

该类继承了AbstractStringBuilder类，看下其append方法：

```java
public AbstractStringBuilder append(String str) {
  if (str == null)
    return appendNull();
  int len = str.length();
  ensureCapacityInternal(count + len);
  str.getChars(0, len, value, count);
  count += len;
  return this;
}
```

append会直接拷贝字符到内部的字符数组中，如果字符数组长度不够，会进行扩展。

StringBuffer和StringBuilder类似，最大的区别就是StringBuffer是线程安全的，看一下StringBuffer的append方法。

```java
public synchronized StringBuffer append(String str) {
  toStringCache = null;
  super.append(str);
  return this;
}
```

该方法使用`synchronized`进行声明，说明是一个线程安全的方法。而StringBuilder则不是线程安全的。

##### 5.5 StringUtils.join

除了JDK中内置的字符串拼接方法，还可以使用一些开源类库中提供的字符串拼接方法名，如apache.commons中提供的StringUtils类，其中的join方法可以拼接字符串。

```java
String wechat = "Hollis";
String introduce = "每日更新Java相关技术文章";
System.out.println(StringUtils.join(wechat, ",", introduce));
```

这里简单说一下，StringUtils中提供的join方法，最主要的功能是：将数组或集合以某拼接符拼接到一起形成新的字符串，如：

```java
String []list  ={"Hollis","每日更新Java相关技术文章"};
String result= StringUtils.join(list,",");
System.out.println(result);
//结果：Hollis,每日更新Java相关技术文章
```

并且，Java8中的String类中也提供了一个静态的join方法，用法和StringUtils.join类似。

##### 5.6 效率对比

```java
long t1 = System.currentTimeMillis();
//这里是初始字符串定义
for (int i = 0; i < 50000; i++) {
  //这里是字符串拼接代码
}
long t2 = System.currentTimeMillis();
System.out.println("cost:" + (t2 - t1));
```

我们使用形如以上形式的代码，分别测试下五种字符串拼接代码的运行时间。得到结果如下：

```
+ cost:5119
StringBuilder cost:3
StringBuffer cost:4
concat cost:3623
StringUtils.join cost:25726
```

从结果可以看出，用时从短到长的对比是：

StringBuilder<StringBuffer<concat<+<StringUtils.join

StringBuffer在StringBuilder的基础上，做了同步处理，所以在耗时上会相对多一些。

StringUtils.join也是使用了StringBuilder，并且其中还是有很多其他操作，所以耗时较长，这个也容易理解。其实==StringUtils.join更擅长处理字符串数组或者列表的拼接==。

那么问题来了，前面我们分析过，其实使用+拼接字符串的实现原理也是使用的StringBuilder，那为什么结果相差这么多，高达1000多倍呢？

我们再把以下代码反编译下：

```text
long t1 = System.currentTimeMillis();
String str = "hollis";
for (int i = 0; i < 50000; i++) {
    String s = String.valueOf(i);
    str += s;
}
long t2 = System.currentTimeMillis();
System.out.println("+ cost:" + (t2 - t1));
```

反编译后代码如下：

```text
long t1 = System.currentTimeMillis();
String str = "hollis";
for(int i = 0; i < 50000; i++)
{
    String s = String.valueOf(i);
    str = (new StringBuilder()).append(str).append(s).toString();
}

long t2 = System.currentTimeMillis();
System.out.println((new StringBuilder()).append("+ cost:").append(t2 - t1).toString());
```

我们可以看到，反编译后的代码，在for循环中，每次都是new了一个StringBuilder，然后再把String转成StringBuilder，再进行append。频繁的新建对象当然要耗费很多时间了，不仅仅会耗费时间，频繁的创建对象，还会造成内存资源的浪费。

所以，阿里巴巴Java开发手册建议：循环体内，字符串的连接方式，使用 StringBuilder 的 append 方法进行扩展。而不要使用+。

##### 5.7 小结

由于字符串拼接过程中会创建新的对象，所以如果要在一个循环体中进行字符串拼接，就要考虑内存问题和效率问题。

因此，经过对比，我们发现，直接使用StringBuilder的方式是效率最高的。因为StringBuilder天生就是设计来定义可变字符串和字符串的变化操作的。

但是，还要强调的是：

1. 如果不是在循环体中进行字符串拼接的话，直接使用+就好了。
2. 如果在并发场景中进行字符串拼接的话，要使用StringBuffer来代替StringBuilder。



#### 6. String.valueOf和Integer.toString的区别

---

有三种方式将一个int类型的变量变成一个String类型，那么他们有什么区别？

```java
int i = 5;
String i1 = "" + i;
String i2 = String.valueOf(i);
String i3 = Integer.toString(i);
```

第三行和第四行没有任何区别，因为String.valueOf(i)也是调用Integer.toString(i)来实现的。

第二行代码其实是String i1 = (new StringBuilder()).append(i).toString();，首先创建一个StringBuilder对象，然后再调用append方法，再调用toString方法。



#### 7. switch对String的支持

---

Java 7中，switch的参数可以是String类型。到目前为止switch支持这样几种数据类型：`byte` `short` `int` `char` `String` 。switch对整型的支持是怎么实现的呢？对字符型是怎么实现的呢？String类型呢？有一点Java开发经验的人这个时候都会猜测switch对String的支持是使用equals()方法和hashcode()方法。那么到底是不是这两个方法呢？接下来我们就看一下，switch到底是如何实现的。

##### 7.1 switch对整型支持的实现

```java
public class switchDemoInt {
  public static void main(String[] args) {
    int a = 5;
    switch (a) {
      case 1:
        System.out.println(1);
        break;
      case 5:
        System.out.println(5);
        break;
      default:
        break;
    }
  }
}
//output 5
```

反编译后的代码如下：

```java
public class switchDemoInt
{
  public switchDemoInt()
  {
  }
  public static void main(String args[])
  {
    int a = 5;
    switch(a)
    {
      case 1: // '\001'
        System.out.println(1);
        break;

      case 5: // '\005'
        System.out.println(5);
        break;
    }
  }
}
```

> **switch对int的判断是直接比较整数的值。**

##### 7.2 对字符型支持的实现

```java
public class switchDemoInt {
  public static void main(String[] args) {
    char a = 'b';
    switch (a) {
      case 'a':
        System.out.println('a');
        break;
      case 'b':
        System.out.println('b');
        break;
      default:
        break;
    }
  }
}
```

编译后的代码如下：

```java
public class switchDemoChar
{
  public switchDemoChar()
  {
  }
  public static void main(String args[])
  {
    char a = 'b';
    switch(a)
    {
      case 97: // 'a'
        System.out.println('a');
        break;
      case 98: // 'b'
        System.out.println('b');
        break;
    }
  }
}
```

> ==对char类型进行比较的时候，实际上比较的是ascii码，编译器会把char型变量转换成对应的int型变量。==

##### 7.3 对字符串支持的实现

```java
public class switchDemoString {
  public static void main(String[] args) {
    String str = "world";
    switch (str) {
      case "hello":
        System.out.println("hello");
        break;
      case "world":
        System.out.println("world");
        break;
      default:
        break;
    }
  }
}
```

对代码进行反编译：

```java
public class switchDemoString
{
  public switchDemoString()
  {
  }
  public static void main(String args[])
  {
    String str = "world";
    String s;
    switch((s = str).hashCode())
    {
      default:
        break;
      case 99162322:
        if(s.equals("hello"))
          System.out.println("hello");
        break;
      case 113318802:
        if(s.equals("world"))
          System.out.println("world");
        break;
    }
  }
}
```

字符串的switch是通过`equals()`和`hashCode()`方法来实现的。**switch中只能使用整型**，比如`byte`。`short`，`char`(ackii码是整型)以及`int`。还好`hashCode()`方法返回的是`int`，而不是`long`。通过这个很容易记住`hashCode`返回的是`int`这个事实。仔细看下可以发现，进行`switch`的实际是哈希值，然后通过使用equals方法比较进行安全检查，这个检查是必要的，因为哈希可能会发生碰撞。因此它的性能是不如使用枚举进行switch或者使用纯整数常量，但这也不是很差。因为Java编译器只增加了一个`equals`方法，如果你比较的是字符串字面量的话会非常快，比如”abc” ==”abc”。如果你把`hashCode()`方法的调用也考虑进来了，那么还会再多一次的调用开销，因为字符串一旦创建了，它就会把哈希值缓存起来。因此如果这个`switch`语句是用在一个循环里的，比如逐项处理某个值，或者游戏引擎循环地渲染屏幕，这里`hashCode()`方法的调用开销其实不会很大。



#### 8. 字符串池

---

String作为一个Java类，可以通过以下两种方式创建一个字符串：

```java
String str = "Hollis";
String str = new String("Hollis");
```

第一种是我们比较常用的做法，这种形式叫做"字面量"。

==在JVM中，为了减少相同的字符串的重复创建，为了达到节省内存的目的。会单独开辟一块内存，用于保存字符串常量，这个内存区域被叫做字符串常量池。==

当代码中出现双引号形式（字面量）创建字符串对象时，JVM 会先对这个字符串进行检查，如果字符串常量池中存在相同内容的字符串对象的引用，则将这个引用返回；否则，创建新的字符串对象，然后将这个引用放入字符串常量池，并返回该引用。这种机制，就是**字符串驻留或池化**。

- JDK1.7之前，运行时常量池（字符串常量池也在里边）是存放在方法区，此时方法区的实现是永久带。
- JDK1.7字符串常量池被单独从方法区移到堆中，运行时常量池剩下的还在永久带（方法区）
- JDK1.8，永久带更名为元空间（方法区的新的实现），但字符串常量池池还在堆中，运行时常量池在元空间（方法区）



#### 9. String长度限制

---

String类中有很多重载的构造函数，其中有几个是支持用户传入length来执行长度的：

```java
public String(byte bytes[], int offset, int length)
```

可以看到，这里面的参数length是使用int类型定义的，那么也就是说，String定义的时候，最大支持的长度就是int的最大范围值。

根据Integer类的定义，`java.lang.Integer#MAX_VALUE`的最大值是2^31 - 1;

那么，我们是不是就可以认为String能支持的最大长度就是这个值了呢？

其实并不是，这个值只是在运行期，我们构造String的时候可以支持的一个最大长度，而实际上，在编译期，定义字符串的时候也是有长度限制的。

如下代码：

```java
String s = "11111...1111";//其中有10万个字符"1"
```

当使用如上形式定义一个字符串，执行javac编译时，是会抛出如下异常：

```
错误: 常量字符串过长
```

那么，明明String的构造函数指定的长度是可以支持2147483647(2^31 - 1)的，为什么像以上形式定义的时候无法编译呢？

其实，形如`String s = "xxx";`定义String的时候，xxx被我们称之为字面量，这种字面量在编译之后会以常量的形式进入到Class常量池。

那么问题就来了，因为==要进入常量池，就要遵守常量池的有关规定==。

##### 9.1 编译期限制

javac是将Java文件编译成class文件的一个命令，那么在Class文件生成过程中，就需要遵守一定的格式。

根据《Java虚拟机规范》中第4.4章节常量池的定义，`CONSTANT_String_info` 用于表示 java.lang.String 类型的常量对象，格式如下：

```java
CONSTANT_String_info {
  u1 tag;
  u2 string_index;
}
```

其中，string_index 项的值必须是对常量池的有效索引，常量池在该索引处的项必须是 CONSTANT_Utf8_info 结构，表示一组 Unicode 码点序列，这组 Unicode 码点序列最终会被初始化为一个 String 对象。

CONSTANT_Utf8_info 结构用于表示字符串常量的值：

```java
CONSTANT_Utf8_info {
  u1 tag;
  u2 length;
  u1 bytes[length];
}
```

其中，length则指明了 bytes[]数组的长度，其类型为u2，

翻阅《规范》，我们可以获悉。u2表示两个字节的无符号数，那么1个字节有8位，2个字节就有16位。16位无符号数可表示的最大值位==2^16 - 1 = 65535==。

也就是说，Class文件中常量池的格式规定了，其字符串常量的长度不能超过65535。

那么，我们尝试使用以下方式定义字符串：

```java
String s = "11111...1111";//其中有65535个字符"1"
```

尝试使用javac编译，同样会得到"错误: 常量字符串过长"，那么原因是什么呢？

其实，这个原因在javac的代码中是可以找到的，在Gen类中有如下代码：

```java
private void checkStringConstant(DiagnosticPosition var1, Object var2) {
  if (this.nerrs == 0 && var2 != null && var2 instanceof String && ((String)var2).length() >= 65535) {
    this.log.error(var1, "limit.string", new Object[0]);
    ++this.nerrs;
  }
}
```

代码中可以看出，当参数类型为String，并且长度大于等于65535的时候，就会导致编译失败。

这个地方大家可以尝试着debug一下javac的编译过程，也可以发现这个地方会报错。

如果我们尝试以65534个字符定义字符串，则会发现可以正常编译。

其实，关于这个值，在《Java虚拟机规范》也有过说明：

>if the Java Virtual Machine code for a method is exactly 65535 bytes long and ends with an instruction that is 1 byte long, then that instruction cannot be protected by an exception handler. A compiler writer can work around this bug by limiting the maximum size of the generated Java Virtual Machine code for any method, instance initialization method, or static initializer (the size of any code array) to 65534 bytes

##### 9.2 运行期限制

上面提到的这种String长度的限制是编译期的限制，也就是使用String s= “”;这种字面值方式定义的时候才会有的限制。

String在运行期也是有限制，就是 Integer.MAX_VALUE ，这个值约等于4G，在运行期，如果String的长度超过这个范围，就可能会抛出异常。(在jdk 1.9之前）

int 是一个 32 位变量类型，取正数部分来算的话，他们最长可以有

```
2^31-1 =2147483647 个 16-bit Unicodecharacter

2147483647 * 16 = 34359738352 位
34359738352 / 8 = 4294967294 (Byte)
4294967294 / 1024 = 4194303.998046875 (KB)
4194303.998046875 / 1024 = 4095.9999980926513671875 (MB)
4095.9999980926513671875 / 1024 = 3.99999999813735485076904296875 (GB)
```

有近 4G 的容量。

很多人会有疑惑，编译的时候最大长度都要求小于65535了，运行期怎么会出现大于65535的情况呢。这其实很常见，如以下代码：

```java
String s = "";
for (int i = 0; i <100000 ; i++) {
  s+="i";
}
```

得到的字符串长度就有10万，另外我之前在实际应用中遇到过这个问题。

之前一次系统对接，需要传输高清图片，约定的传输方式是对方将图片转成BASE64编码，我们接收到之后再转成图片。

在将BASE64编码后的内容赋值给字符串的时候就抛了异常。