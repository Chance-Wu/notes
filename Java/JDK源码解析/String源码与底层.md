### 一、基础

---

#### 1.1 String的修饰符与实现类

String类的由**final修饰**的，并且实现了Serializable，Comparable，CharSequence等接口。

```java
public final class String implements Serializable, Comparable<String>, CharSequence {

}
```

final修饰String类表示：

1. String不能被继承，并且String类中的成员方法都默认是final方法。
2. String类一旦被创建，就无法改变，对String对象的任何操作都不会影响到原对象，任何的**change操作都会产生新的String对象**。

#### 1.2 String类的成员变量

```java
/** 用于字符存储 */
private final char value[];

/** 缓存字符串的hash代码 */
private int hash; // 默认为 0

/** 使用JDK 1.0.2中的serialVersionUID实现互操作性 */
private static final long serialVersionUID = -6849794470754667710L;

/**
 * 类String在序列化流协议中是特殊的。
 * 根据对象序列化规范6.2节，将字符串实例写入ObjectOutputStream。
 */
private static final ObjectStreamField[] serialPersistentFields =
  new ObjectStreamField[0];
```

1. String是通过char数组来保存字符串的。

2. hash值将用于String类hashCode()方法的计算。

3. serialVersionUID属性作为String类的序列化ID。

4. transient是用于指定哪个字段不被默认序列化，对于不需要序列化的属性直接用transient修饰即可。而**serialPersistentFields用于指定哪些字段需要被默认序列化**，具体用法如下：

   ```java
   private static final ObjectStreamField[] serialPersistentFields = {
     new ObjectStreamField("name", String.class),
     new ObjectStreamField("age", Integer.Type)
   }
   ```

>注意：如果同时定义了serialPersistentFields与transient，transient会被忽略。

#### 1.3 创建String对象

1. 直接使用“”，即直接使用“字面量”赋值。

   ```java
   String name = "chance";
   ```

2. 使用连接符“+”来赋值。

   ```java
   String name = "chanc" + "e";
   ```

3. 使用关键字new来创建对象。

   ```java
   String name = new String("chance");
   ```

4. 除了上面几种创建方式外，还可以使用以下方法创建String对象

   - 使用clone()方法
   - 使用反射
   - 使用反序列化

#### 1.4 String被设计为不可变性的原因

- 主要是为了“**效率**” 和 “**安全性**” 的缘故。若 String允许被继承，由于它的高度被使用率，可能会降低程序的性能，所以String被定义成final。
- 由于字符串常量池的存在，为了更有效的管理和优化字符串常量池里的对象，将String设计为不可变性。
- 安全性考虑。因为使用字符串的场景非常多，设计成不可变可以有效的防止字符串被有意或者无意的篡改。
- **作为HashMap、HashTable等hash型数据key的必要。因为不可变的设计，jvm底层很容易在缓存String对象的时候缓存其hashcode，这样在执行效率上会大大提升**。



### 二、深入String

---

#### 2.1 先了解Java内存区域

JAVA的运行时数据区包括以下几个区域：

1. 方法区（Method Area）
2. Java堆区（Heap）
3. 本地方法栈（Native Method Stack）
4. 虚拟机栈（VM Stack）
5. 程序计数器（Program Conter Register）

具体内容可参考[总结Java内存区域和常量池](https://blog.csdn.net/CoderBruis/article/details/85240273)

对于String类来说，存在一个字符串常量池，对于字符串常量池，在HotSpot VM里实现的string pool功能的是一个StringTable类，它是一个**哈希表**，里面存的是驻留字符串(也就是我们常说的用双引号括起来的)的**引用**（而不是驻留字符串实例本身），也就是说在堆中的某些字符串实例被这个StringTable引用之后就等同被赋予了”驻留字符串”的身份。这个StringTable在每个HotSpotVM的实例只有一份，**被所有的类共享**。

总结：

1. 字符串常量池在每个VM中只有一份，存放的是字符串常量的引用值。
2. 字符串常量池——string pool，也叫做string literal pool。
3. 字符串池里的内容是在类加载完成，经过验证，准备阶段之后在堆中生成字符串对象实例，然后将该字符串对象实例的引用值存到string pool中。
4. string pool中存的是引用值而不是具体的实例对象，**具体的实例对象是在堆中开辟的一块空间存放的**。

#### 2.2 String与Java内存区域

如下看使用“”和new的方式创建的字符串在底层都发生了些什么。

```java
public class TestString {
  public static void main(String[] args) {
    String name = "bruis";
    String name2 = "bruis";
    String name3 = new String("bruis");
    //System.out.println("name == name2 : " + (name == name2));// true
    //System.out.println("name == name3 : " + (name == name3));// false
  }
}
```

因为语句`String name = "bruis";`已经将创建好的字符串对象存放在了常量池中，所以name引用指向常量池中的"bruis"对象，而name2就直接指向已经存在在常量池中的"bruis"对象，所以name和name2都指向了同一个对象。这就能理解为什么name == name2 为true了。

使用new 方式创建字符串。首先会在堆上创建一个对象，然后判断字符串常量池中是否存在字符串的常量，如果不存在则在字符串常量池上创建常量；如果有则不作任何操作。所以name是指向字符串常量池中的常量，而name3是指向堆中的对象，所以name == name3 为false。

进入TestString.class的目录下，对TestString类进行反编译，看看反编译之后的内容：

```bash
$ javap -c TestString.class
Compiled from "TestString.java"
public class org.springframework.core.env.TestString {
  public org.springframework.core.env.TestString();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #2                  // String bruis
       2: astore_1
       3: ldc           #2                  // String bruis
       5: astore_2
       6: new           #3                  // class java/lang/String
       9: dup
      10: ldc           #2                  // String bruis
      12: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
      15: astore_3
      16: return
}
```

从反编译的结果中可以看到，首先是进行无参构造方法的调用。

```
0: aload_0       // 表示对this进行操作，把this装载到操作数栈中
1: invokespecial #1        // 调用<init>

0: ldc   #2      //将常量池中的bruis值加载到虚拟机栈中
2: astore_1      //将0中的引用赋值给第一个局部变量，即String name="bruis"
3: ldc  #2       //将常量池中的bruis值加载到虚拟机栈中
5: astore_2      //将3中的引用赋值给第二个局部变量，即String name2= "bruis"
6: new           //调用new指令，创建一个新的String对象，并存入堆中。因为常量池中已经存在了"bruis"，所以新创建的对象指向常量池中的"bruis"
9: dup           //复制引用并并压入虚拟机栈中
10: ldc          //加载常量池中的"bruis"到虚拟机栈中
12: invokespecial //调用String类的构造方法
15: astore_3      //将引用赋值给第三个局部变量，即String name3=new String("bruis")
```

使用如下命令来查看常量池的内容：

```java
$ javap -verbose TestString
▒▒▒▒: ▒▒▒▒▒▒▒ļ▒TestString▒▒▒▒org.springframework.core.env.TestString
Classfile /D:/bruislearningcode/springframeworksources/spring-framework-master/spring-framework-master/out/test/classes/org/springframework/core/env/TestString.class
  Last modified 2019-7-3; size 600 bytes
  MD5 checksum 85315424cab60ed8f47955dfd577f6e0
  Compiled from "TestString.java"
public class org.springframework.core.env.TestString
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#24         // java/lang/Object."<init>":()V
   #2 = String             #25            // bruis
   #3 = Class              #26            // java/lang/String
   #4 = Methodref          #3.#27         // java/lang/String."<init>":(Ljava/lang/String;)V
   #5 = Class              #28            // org/springframework/core/env/TestString
   #6 = Class              #29            // java/lang/Object
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               LocalVariableTable
  #12 = Utf8               this
  #13 = Utf8               Lorg/springframework/core/env/TestString;
  #14 = Utf8               main
  #15 = Utf8               ([Ljava/lang/String;)V
  #16 = Utf8               args
  #17 = Utf8               [Ljava/lang/String;
  #18 = Utf8               name
  #19 = Utf8               Ljava/lang/String;
  #20 = Utf8               name2
  #21 = Utf8               name3
  #22 = Utf8               SourceFile
  #23 = Utf8               TestString.java
  #24 = NameAndType        #7:#8          // "<init>":()V
  #25 = Utf8               bruis
  #26 = Utf8               java/lang/String
  #27 = NameAndType        #7:#30         // "<init>":(Ljava/lang/String;)V
  #28 = Utf8               org/springframework/core/env/TestString
  #29 = Utf8               java/lang/Object
  #30 = Utf8               (Ljava/lang/String;)V
{
  public org.springframework.core.env.TestString();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 3: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lorg/springframework/core/env/TestString;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=3, locals=4, args_size=1
         0: ldc           #2                  // String bruis
         2: astore_1
         3: ldc           #2                  // String bruis
         5: astore_2
         6: new           #3                  // class java/lang/String
         9: dup
        10: ldc           #2                  // String bruis
        12: invokespecial #4                  // Method java/lang/String."<init>":(Ljava/lang/String;)V
        15: astore_3
        16: return
      LineNumberTable:
        line 5: 0
        line 6: 3
        line 7: 6
        line 10: 16
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      17     0  args   [Ljava/lang/String;
            3      14     1  name   Ljava/lang/String;
            6      11     2 name2   Ljava/lang/String;
           16       1     3 name3   Ljava/lang/String;
}
SourceFile: "TestString.java"
```

可以看到值"bruis"已经存放在了常量池中了

```
#2 = String             #25            // bruis
```

以及局部变量表LocalVariableTable中存储的局部变量：

```
LocalVariableTable:
Start  Length  Slot  Name   Signature
    0      17     0  args   [Ljava/lang/String;
    3      14     1  name   Ljava/lang/String;
    6      11     2 name2   Ljava/lang/String;
    16       1     3 name3   Ljava/lang/String;
```

> 这里有一个需要注意的地方，在java中使用"+"连接符时，一定要注意到"+"的连接符效率非常低下，因为**"+"连接符的原理就是通过StringBuilder.append()来实现的**。所以如：String name = "a" + "b";在底层是先new 出一个StringBuilder对象，然后再调用该对象的append()方法来实现的，调用过程等同于：
>
> ```java
> // String name = "a" + "b";
> String name = new StringBuilder().append("a").append("b").toString();
> ```

#### 2.3 String的intern方法

字符串常量池由String独自维护，当调用intern()方法时，如果字符串常量池中包含该字符串，则直接返回字符串常量池中的字符串。否则将此String对象添加到字符串常量池中，并返回对此String对象的引用。

```java
String a1 = new String("AA") + new String("BB");
System.out.println("a1 == a1.intern() " + (a1 == a1.intern()));//true

String test = "ABABCDCD";
String a2 = new String("ABAB") + new String("CDCD");
String a3 = "ABAB" + "CDCD";
System.out.println("a2 == a2.intern() " + (a2 == a2.intern()));//false
System.out.println("a2 == a3 " + (a2 == a3));//false
System.out.println("a3 == a2.intern() " + (a3 == a2.intern()));//true
```

重新理解使用new和字面量创建字符串的两种方式

1. 使用字面量的方式创建字符串要分两种情况：

   1. 如果字符串常量池中没有值，则直接创建字符串，并将值存入字符串常量池中；

      ```java
      String name = "bruis";
      ```

      对于字面量形式创建出来的字符串，JVM会在编译期时对其进行优化并**将字面量值存放在字符串常量池中**。**运行期在虚拟机栈栈帧中的局部变量表里创建一个name局部变量，然后指向字符串常量池中的值**，如图所示：

      ![1](img/1.png)

   2. 如果字符串常量池中存在字面量值，此时要看这个是真正的字符串还是引用。如果是字符串则将局部变量指向常量池中的值；否则指向引用指向的地方。比如常量池中的值是指向堆中的引用，则name变量为将指向堆中的引用，如图所示：

      ![68747470733a2f2f696d672d626c6f672e6373646e696d672e636e2f323031393037303631373435343733302e706e673f782d6f73732d70726f636573733d696d6167652f77617465726d61726b2c747970655f5a6d46755a33706f5a57356e6147567064476b2c736861646f775f31302c746578745f61](img/68747470733a2f2f696d672d626c6f672e6373646e696d672e636e2f323031393037303631373435343733302e706e673f782d6f73732d70726f636573733d696d6167652f77617465726d61726b2c747970655f5a6d46755a33706f5a57356e6147567064476b2c736861646f775f31302c746578745f61.png)

2. 使用new的方式创建字符串

   ```java
   String name = new String("bruis");
   ```

   首先在堆中new出一个对象，然后常量池中创建一个指向堆中“bruis”的引用。

   ```java
   /**
    * 首先对于new出的两个String()对象进行字符串连接操作，编译器无法进行优化，只有等到运行期期间，通过各自的new操作创建出对象之后，然后使用"+"连接符拼接字符串，再从字符串常量池中创建三个分别指向堆中"AA"、"BB"，而"AABB"是直接在池中创建的字面量值，这一点可以通过类的反编译来证明，这里就不具体展开了。
    */
   String a7 = new String("AA") + new String("BB");
   System.out.println("a7 == a7.intern() " + (a7 == a7.intern())); //true
   
   
   /**
    *  对于下面的实例，首先在编译期就是将"ABABCDCD"存入字符串常量池中，其对于"ABABCDCD"存入的是具体的字面量值，而不是引用。
    *  因为在编译器在编译期无法进行new 操作，所以就无法知道a8的地址，在运行期期间，使用a8.intern()可以返回字符串常量池的字面量。而a9
    *  在编译期经过编译器的优化，a9变量会指向字符串常量池中的"ABABCDCD"。所以a8 == a8.intern()为false；a8 == a9为false；a9 == a8.intern()为
    *  true。
    */
   String test = "ABABCDCD";
   String a8 = new String("ABAB") + new String("CDCD");
   String a9 = "ABAB" + "CDCD";
   System.out.println("a8 == a8.intern() " + (a8 == a8.intern())); //false
   System.out.println("a8 == a9 " + (a8 == a9)); //false
   System.out.println("a9 == a8.intern() " + (a9 == a8.intern())); //true
   ```

针对于编译器优化，总结以下两点：

1. 常量可以被认为运行时不可改变，所以编译时被以常量折叠方式优化。
2. 变量和动态生成的常量必须在运行时确定值，所以不能在编译期折叠优化



### 三、String常用方法

---

#### 3.1 equals方法

```java
public boolean equals(Object anObject) {
  if (this == anObject) {
    return true;
  }
  if (anObject instanceof String) {
    String anotherString = (String)anObject;
    int n = value.length;
    if (n == anotherString.value.length) {
      char v1[] = value;
      char v2[] = anotherString.value;
      int i = 0;
      while (n-- != 0) {
        if (v1[i] != v2[i])
          return false;
        i++;
      }
      return true;
    }
  }
  return false;
}
```

equals方法比较是“字符串对象的地址”，如果不相同比较字符串内容，时间也就是char数组的内容。

#### 3.2 hashcode方法

```java
public int hashCode() {
  int h = hash;
  if (h == 0 && value.length > 0) {
    char val[] = value;

    for (int i = 0; i < value.length; i++) {
      h = 31 * h + val[i];
    }
    hash = h;
  }
  return h;
}
```

在String类中，有个字段hash存储着String的哈希值，**如果字符串为空，则hash的值为0**。String类中的hashCode计算方法就是以31为权，每一位为字符的ASCII值进行运算，用自然溢出来等效取模，经过第一次的hashcode计算之后，属性hash就会赋哈希值。从源码的英文注释可以了解到哈希的计算公式：

```
s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]
```

#### 3.3 hashcode()和equals()

由上面的介绍，可以知道**String的equals()方法实际比较的是两个字符串的内容**，而String的hashCode()方法比较的是字符串的hash值，那么单纯的a.equals(b)为true，就可以断定a字符串等于b字符串了吗？或者单纯的a.hash == b.hash为true，就可以断定a字符串等于b字符串了吗？答案是否定的。 比如下面两个字符串：

```java
String a = "gdejicbegh";
String b = "hgebcijedg";
System.out.println("a.hashcode() == b.hashcode() " + (a.hashCode() == b.hashCode()));
System.out.println("a.equals(b) " + (a.equals(b)));
```

结果为： true false

这个回文字符串就是典型的hash值相同，但是字符串却不相同。对于算法这块领域，回文字符串和字符串匹配都是比较重要的一块，比如马拉车算法、KMP算法等，有兴趣的小伙伴可以在网上搜索相关的算法学习一下。

其实Java中任何一个对象都具备equals()和hashCode()这两个方法，因为他们是在Object类中定义的。

在Java中定义了关于hashCode()和equals()方法的规范，总结来说就是：

1. **如果两个对象equals()，则它们的hashcode一定相等**。
2. 如果两个对象不equals()，它们的hashcode可能相等。
3. 如果两个对象的hashcode相等，则它们不一定equals。
4. **如果两个对象的hashcode不相等，则它们一定不equals**。

#### 3.4 compareTo()

```java
public int compareTo(String anotherString) {
  int len1 = value.length;
  int len2 = anotherString.value.length;
  int lim = Math.min(len1, len2);
  char v1[] = value;
  char v2[] = anotherString.value;

  int k = 0;
  while (k < lim) {
    char c1 = v1[k];
    char c2 = v2[k];
    if (c1 != c2) {
      return c1 - c2;
    }
    k++;
  }
  return len1 - len2;
}
```

这方法是**先比较两个字符串内的字符串数组的ASCII值**，如果最小字符串都比较完了都还是相等的，则返回字符串长度的差值；否则在最小字符串比较完之前，字符不相等，则返回不相等字符的ASCII值差值。

#### 3.5 startWith(String prefix)

```java
public boolean startsWith(String prefix) {
  return startsWith(prefix, 0);
}

public boolean startsWith(String prefix, int toffset) {
  char ta[] = value;
  int to = toffset;
  char pa[] = prefix.value;
  int po = 0;
  int pc = prefix.value.length;
  // Note: toffset might be near -1>>>1.
  if ((toffset < 0) || (toffset > value.length - pc)) {
    return false;
  }
  while (--pc >= 0) {
    if (ta[to++] != pa[po++]) {
      return false;
    }
  }
  return true;
}
```

如果参数字符序列是该字符串字符序列的前缀，则返回true；否则返回false；

```java
String a = "abc";
String b = "abcd";
System.out.println(b.startsWith(a));
```

#### 3.6 indexOf(int ch)

```java
public int indexOf(int ch) {
  return indexOf(ch, 0);
}

public int indexOf(int ch, int fromIndex) {
  final int max = value.length;
  if (fromIndex < 0) {
    fromIndex = 0;
  } else if (fromIndex >= max) {
    // Note: fromIndex might be near -1>>>1.
    return -1;
  }

  if (ch < Character.MIN_SUPPLEMENTARY_CODE_POINT) {
    final char[] value = this.value;
    for (int i = fromIndex; i < max; i++) {
      if (value[i] == ch) {
        return i;
      }
    }
    return -1;
  } else {
    return indexOfSupplementary(ch, fromIndex);
  }
}
```

对于String的indexOf(int ch)方法，查看其源码可知其方法入参为ASCII码值，然后和目标字符串的ASCII值来进行比较的。其中常量Character.MIN_SUPPLEMENTARY_CODE_POINT表示的是0x010000——十六进制的010000，十进制的值为65536，这个值表示的是十六进制的最大值。

下面再看看indexOfSupplementary(ch, fromIndex)方法

```java
private int indexOfSupplementary(int ch, int fromIndex) {
  if (Character.isValidCodePoint(ch)) {
    final char[] value = this.value;
    final char hi = Character.highSurrogate(ch);
    final char lo = Character.lowSurrogate(ch);
    final int max = value.length - 1;
    for (int i = fromIndex; i < max; i++) {
      if (value[i] == hi && value[i + 1] == lo) {
        return i;
      }
    }
  }
  return -1;
}
```

java中特意对超过两个字节的字符进行了处理，例如emoji之类的字符。处理逻辑就在indexOfSupplementary(int ch, int fromIndex)方法里。

Character.class

```java
public static boolean isValidCodePoint(int codePoint) {
  // Optimized form of:
  //     codePoint >= MIN_CODE_POINT && codePoint <= MAX_CODE_POINT
  int plane = codePoint >>> 16;
  return plane < ((MAX_CODE_POINT + 1) >>> 16);
}

```

对于方法isValidCodePoint(int codePoint)方法，用于确定指定代码点是否是一个有效的Unicode代码点。代码

```java
int plane = codePoint >>> 16;
return plane < ((MAX_CODE_POINT + 1) >>> 16);
```

表达的就时判断codePoint是否在MIN_CODE_POINT和MAX_CODE_POINT值之间，如果是则返回true，否则返回false。

#### 3.7 split(String regex, int limit)

```java
public String[] split(String regex, int limit) {
  char ch = 0;
  if (((regex.value.length == 1 &&
        ".$|()[{^?*+\\".indexOf(ch = regex.charAt(0)) == -1) ||
       (regex.length() == 2 &&
        regex.charAt(0) == '\\' &&
        (((ch = regex.charAt(1))-'0')|('9'-ch)) < 0 &&
        ((ch-'a')|('z'-ch)) < 0 &&
        ((ch-'A')|('Z'-ch)) < 0)) &&
      (ch < Character.MIN_HIGH_SURROGATE ||
       ch > Character.MAX_LOW_SURROGATE))
  {
    int off = 0;
    int next = 0;
    // 如果limit > 0，则limited为true
    boolean limited = limit > 0;
    ArrayList<String> list = new ArrayList<>();
    while ((next = indexOf(ch, off)) != -1) {
      if (!limited || list.size() < limit - 1) {
        list.add(substring(off, next));
        off = next + 1;
      } else {    // last one
        // limit > 0，直接返回原字符串
        list.add(substring(off, value.length));
        off = value.length;
        break;
      }
    }
    // 如果没匹配到，则返回原字符串
    if (off == 0)
      return new String[]{this};

    // 添加剩余的字字符串
    if (!limited || list.size() < limit)
      list.add(substring(off, value.length));

    int resultSize = list.size();
    if (limit == 0) {
      while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
        resultSize--;
      }
    }
    String[] result = new String[resultSize];
    return list.subList(0, resultSize).toArray(result);
  }
  return Pattern.compile(regex).split(this, limit);
}
```

if判断中**第一个括号**先判断一个字符的情况，并且这个字符不是任何特殊的正则表达式。也就是下面的代码：

```java
(regex.value.length == 1 &&
             ".$|()[{^?*+\\".indexOf(ch = regex.charAt(0)) == -1)
```

如果要根据特殊字符来截取字符串，则需要使用`\\`来进行字符转义。

在if判断中，**第二个括号**判断有两个字符的情况，并且如果这两个字符是以`\`开头的，并且不是字母或者数字的时候。如下列代码所示：

```java
(regex.length() == 2 && regex.charAt(0) == '\\' && (((ch = regex.charAt(1))-'0')|('9'-ch)) < 0 && ((ch-'a')|('z'-ch)) < 0 && ((ch-'A')|('Z'-ch)) < 0)
```

判断完之后，在进行**第三个括号**判断，判断是否是两字节的unicode字符。如下列代码所示：

```java
(ch < Character.MIN_HIGH_SURROGATE ||
             ch > Character.MAX_LOW_SURROGATE)
```

对于下面这段复杂的代码，我们结合示例一句一句来分析。

```java
int off = 0;
int next = 0;
boolean limited = limit > 0;
ArrayList<String> list = new ArrayList<>();
while ((next = indexOf(ch, off)) != -1) {
  if (!limited || list.size() < limit - 1) {
    list.add(substring(off, next));
    off = next + 1;
  } else {    // last one
    //assert (list.size() == limit - 1);
    list.add(substring(off, value.length));
    off = value.length;
    break;
  }
}
// 如果没有匹配到的内容则直接返回
if (off == 0)
  return new String[]{this};

// 添加剩余的segment到list集合中
if (!limited || list.size() < limit)
  list.add(substring(off, value.length));

// Construct result
int resultSize = list.size();
if (limit == 0) {
  while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
    resultSize--;
  }
}
String[] result = new String[resultSize];
return list.subList(0, resultSize).toArray(result);
```

示例代码1：

```java
String splitStr1 = "what,is,,,,split";
String[] strs1 = splitStr1.split(",");
for (String s : strs1) {
  System.out.println(s);
}
System.out.println(strs1.length);
```

运行结果：

```
what
is

split
6
```

示例代码2：

```java
String splitStr1 = "what,is,,,,";
String[] strs1 = splitStr1.split(",");
for (String s : strs1) {
  System.out.println(s);
}
System.out.println(strs1.length);
```

运行结果：

```
what
is
2
```

示例代码3：

```java
String splitStr1 = "what,is,,,,";
String[] strs1 = splitStr1.split(",", -1);
for (String s : strs1) {
  System.out.println(s);
}
System.out.println(strs1.length);
```

运行结果：

```
what
is


6
```

对比了一下示例代码和结果之后，小伙伴们是不是很困惑呢？困惑就对了，下面就开始分析代码吧。在split(String regex, int limit)方法的if判断内部，定义了off和next变量，作为拆分整个字符串的两个指针，然后limit作为拆分整个string字符串的一个阈值。在split()方法内部的复杂逻辑判断中，都围绕着这三个变量来进行。

下面将示例代码1的字符串拆分成字符数组，如下(n代表next指针，o代表off指针)：

```
    w h a t , i s , , , ,  s  p  l  i  t
    0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
    n 
    o
```

由于regex为','，所以满足if括号里的判断。一开始next和off指针都在0位置，limit为0，在while里的判断逻辑指的是获取','索引位置，由上图拆分的字符数组可知，next会分别为4,7,8,9,10。由于limited = limit > 0，得知limited为false，则逻辑会走到

```java
if (!limited || list.size() < limit - 1) {
  list.add(substring(off, next));
  off = next + 1;
}
```

进入第一次while循环体，此时的字符数组以及索引关系如下：

```
    w h a t , i s , , , ,  s  p  l  i  t
    0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
            n 
    o
```

所以list集合里就会添加进字符串what。

第二次进入while循环时，此时的字符数组以及索引关系如下：

```
    w h a t , i s , , , ,  s  p  l  i  t
    0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
                  n 
              o
```

list集合里就会添加进字符串is

第三次进入while循环时，此时的字符数组以及索引关系如下：

```
    w h a t , i s , , , ,  s  p  l  i  t
    0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
                    n 
                  o
```

list集合里就会添加进空字符串""

第四次进入while循环时，此时的字符数组以及索引关系如下：

```
    w h a t , i s , , , ,  s  p  l  i  t
    0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15
                      n 
                    o
```

list集合里就会添加进空字符串""

当o指针指向位置10时，while((next = indexOf(ch, off)) != -1)结果为false，因为此时已经获取不到','了。

注意，此时list中包含的元素有：

```
[what,is, , , ,]
```

当程序走到时，

```java
if(!limited || list.size() < limit) {
  list.add(substring(off, value.length);
           }

           int resultSize = list.size();
           if (limit == 0) {
             while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
               resultSize--;
             }
           }
```

会将字符数组off（此时off为10）位置到value.length位置的字符串存进list集合里，也就是split元素,由于list集合最后一个元素为split，其大小不为0，所以就不会进行resultSize--。所以最终list集合里的元素就有6个元素，值为

```
[what,is, , , ,split]
```

这里相信小伙伴们都知道示例1和示例2的区别在那里了，是因为示例2最后索引位置的list为空字符串，所以list.get(resultSize-1).length()为0，则会调用下面的代码逻辑：

```java
while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
  resultSize--;
}
```

最终会将list中的空字符串给减少。所以示例2的最终结果为

```
[what,is]
```

对于入参limit，可以总结一下为：

1. limit > 0，split()方法最多把字符串拆分成limit个部分。
2. limit = 0，split()方法会拆分匹配到的最后一位regex。
3. limit < 0，split()方法会根据regex匹配到的最后一位，如果最后一位为regex，则多添加一位空字符串；如果不是则添加regex到字符串末尾的子字符串。

就以示例代码一为例，对于字符串"what,is,,,,"。

**对于limit > 0**，由于代码：

```java
boolean limited = limit > 0;  // limited为true
..
  ..
  // !limited为false；list.size()一开始为0，则如果limit > 2，则list.size() < limit - 1为true，则进行字符串截取subString方法调用; 
  // 所以如果limit = 1，则直接返回原字符串；如果limit > 1则将字符串拆分为limit个部分；
  if(!limited || list.size() < limit - 1) {  
    // 截取off到next字符串存入list中
    list.add(substring(off, next));
    off = next + 1;
  } else {
    list.add(substring(off, value.length));
    off = value.length;
    break;
  }
```

所以返回的原字符串：

```
what,is,,,,
1
```

**对于limit = 0**，由于代码：

```java
if (limit == 0) {
  while (resultSize > 0 && list.get(resultSize - 1).length() == 0) {
    resultSize--;
  }
}
```

会将空字符串从list集合中移除掉，所以返回的是：

```
what
is
2
```

**对于limit < 0**，由于代码：

```java
if (!limited || list.size() < limit)
  list.add(substring(off, value.length));
```

会在原来的集合内容上（[what,is,'','','']）再加一个空字符串，也就是[what,is,'','','','']。
