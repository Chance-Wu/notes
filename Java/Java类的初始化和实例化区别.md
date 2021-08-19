#### 类的初始化和实例化

---

> **类的初始化：**
>
> 完成程序执行前的准备工作。在这个阶段，==静态的（变量，方法，代码块）会被执行==。同时在会==开辟一块存储空间来存放静态的数据==。==初始化只在类加载的时候执行一次==。

>**类的实例化：**
>
>创建一个对象的过程。这个过程中会==在堆中开辟内存==，将一些==非静态的方法，变量==存放在里面。在程序执行的过程中，可以创建多个对象，即多次实例化。每次实例化都会开辟一块新的内存。



|                    | 执行次数 | 触发机制                                                     | 是否执行构造方法 | 生命周期 | 执行内容                                                     |
| ------------------ | -------- | ------------------------------------------------------------ | ---------------- | -------- | ------------------------------------------------------------ |
| 初始化（类级别）   | 1        | Class.forName()，new，main方法的类，通过子类加载父类静态成员导致父类的初始化 | 否               | 第五步   | 为静态成员赋值，执行静态代码块                               |
| 实例化（实例对象） | N        | new，Class.newInstance()                                     | 是               | 第六步   | 在堆区分配内存空间，执行实例对象初始化，设置引用变量a指向刚分配的内存地址 |



#### Java类的生命周期

---

一个class文件从加载到卸载的全过程，类的完整生命周期包括7个部分：加载——验证——准备——解析——初始化——使用——卸载，如下图所示

其中，验证——准备——解析 称为连接阶段，除了解析外，其他阶段是顺序发生的，而解析可以与这些阶段交叉进行，因为Java支持动态绑定（晚期绑定），需要运行时才能确定具体类型；在使用阶段实例化对象。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gteahpz5kuj60i106qdfv02.jpg)



>**类加载过程**
>
>1. 加载：通过类名获取类的二进制字节流是通过类加载器来完成的。其加载过程使用“双亲委派模型”
>2. 验证：当一个类被加载之后，必须要验证一下这个类是否合法，比如这个类是不是符合字节码的格式、变量与方法是不是有重复、数据类型是不是有效、继承与实现是否合乎标准等等。总之，这个阶段的目的就是保证加载的类是能够被jvm所运行。
>3. 准备：为类变量（静态变量）在方法区分配内存，并设置零值。注意：这里是类变量，不是实例变量，实例变量是对象分配到堆内存时根据运行时动态生成的。
>4. 解析：把常量池中的符号引用解析为直接引用：根据符号引用所作的描述，在内存中找到符合描述的目标并把目标指针指针返回。
>5. 初始化：类的初始化过程是这样的：按照顺序自上而下运行类中的变量赋值语句和静态语句，如果有父类，则首先按照顺序运行父类中的变量赋值语句和静态语句在类的初始化阶段，只会初始化与类相关的静态赋值语句和静态语句，也就是有static关键字修饰的信息，而没有static修饰的赋值语句和执行语句在实例化对象的时候才会运行。执行<clinit>()方法（clinit是class initialize的简写）
>6. 实例化：在堆区分配内存空间，执行实例对象初始化，设置引用变量a指向刚分配的内存地址

>**运行时区内存分配**
>
>![](https://tva1.sinaimg.cn/large/008i3skNgy1gted623zbbj618s0hitbv02.jpg)



#### 具体流程

---

```java
public class A {
  private int m=2;
  private String str1="youyou";

  public final static String MESS="world";
  static String ms="world";

  public String getName(String input){
    String temp=input;
    return  temp;
  }

  public  static int getId(){return 0;}
}
```

Test类

```java
public class Test {
  public static void main(String[] args) {
    Class clazz=A.class;
    A a=new A();
    A a1=new A();
  }
  public void change(int i)
  {
    i=123;
  }
}
```

![](https://tva1.sinaimg.cn/large/008i3skNgy1gtev12pvbjj60o30b8t8y02.jpg)

![](https://tva1.sinaimg.cn/large/008i3skNgy1gtevp00rivj60ot0kdmy502.jpg)

引用对象：A中成员B b，b即为引用对象，

常量三种：8种基本数据类型byte boolean，char short，float int，double long的具体值，受final修改的变量，""双引号引起来的字符串。



#### String s = new String("abc")创建了几个对象

---





#### Java成员变量和局部变量在内存中的分配

---

- 成员变量就是方法外部，类的内部定义的变量；
- 局部变量就是方法或语句块内部定义的变量。

局部变量必须初始化。==形式参数是局部变量==，==局部变量中基础数据类型的引用和值都存储在栈中，对象引用存在栈中，对象存在堆中==。栈内存中的局部变量随着方法的消失而消失。 成员变量存储在堆中的对象里面，由垃圾回收器负责回收。 如以下代码：

```java
class BirthDate {
  private int day;
  private int month;
  private int year;

  public BirthDate(int d, int m, int y) {
    day = d;
    month = m;
    year = y;
  }
  // 省略get,set方法………
}

public class Test {
  public static void main(String args[]) {
    int date = 9;
    Test test = new Test();
    test.change(date);
    BirthDate d1 = new BirthDate(7, 7, 1970);
  }

  public void change(int i) {
    i = 1234;
  }
}
```

对于以上这段代码，date为局部变量，i，d，m，y都是形参为局部变量，day，month，year为成员变量。下面分析一下代码执行时候的变化：

1. main方法开始执行：int date = 9; date局部变量，基础类型，引用和值都存在栈中。

2. Test test = new Test();test为对象引用，存在栈中，对象(new Test())存在堆中。

3. test.change(date); i为局部变量，引用和值存在栈中。当方法change执行完成后，i就会从栈中消失。

4. BirthDate d1= new BirthDate(7,7,1970); d1为对象引用，存在栈中，对象(new BirthDate())存在堆中，其中d，m，y为局部变量存储在栈中，且它们的类型为基础类型，因此它们的数据也存储在栈中。day,month,year为成员变量，它们存储在堆中(new BirthDate()里面)。当BirthDate构造方法执行完之后，d,m,y将从栈中消失。

5. main方法执行完之后，date变量，test，d1引用将从栈中消失，new Test(), new BirthDate()将等待垃圾回收。

![](https://tva1.sinaimg.cn/large/008i3skNgy1gtew9he5dxj60j80aqt8s02.jpg)



#### Java类和对象的生命周期详解

---

https://blog.csdn.net/yanliguoyifang/article/details/80964237

