> 要进行垃圾回收，如何判断一个对象是否可以被回收？
>
> - ==引用计数法==（很难解决对象直接的循环引用问题）
> - ==枚举根节点做可达性分析==
>
> *<u>所有可达性算法都会有起点——GC Root</u>*。即需要通过GC Root找出所有活的对象，那么剩下所有的没有标记的对象就是需要回收的对象。

#### 1. GC Root的特点

> 当前时刻存活的对象！

#### 2. 哪些对象可以作为GC Root的对象

> - 虚拟机栈中==局部变量==（也叫局部变量表）中引用的对象。
> - 方法区中类的==静态变量==、==常量==引用的对象。
> - 本地方法栈中JNI（==Native方法==）引用的对象。

```java
public class GCRootDemo {
    private byte[] byteArray = new byte[100 * 1024 * 1024];
 
    private static GCRootDemo gc2; // 方法区中类的静态变量
    private static final GCRootDemo gc3 = new GCRootDemo(); // 方法区中类的常量
 
    public static void m1(){
        GCRootDemo gc1 = new GCRootDemo(); // 虚拟机栈中的局部变量
        System.gc();
        System.out.println("第一次GC完成");
    }
    public static void main(String[] args) {
        m1();
    }
}
```

gc1：是虚拟机栈中的局部变量；

gc2：是方法区中类的静态变量；

gc3：是方法区中的常量；

都可以作为GC Roots 的对象。