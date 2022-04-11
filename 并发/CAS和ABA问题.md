#### 1. 什么是CAS

>`Compare-And-Swap`，是一种用于在多线程环境下实现同步功能的机制。
>
>CAS 操作包含三个操作数——==内存位置==、==预期数值==和==新值==。CAS 的实现逻辑是将内存位置处的数值与预期数值想比较，若相等，则将内存位置处的值替换为新值。若不相等，则不做任何操作。
>
>==原语的执行必须是连续的，在执行过程中不允许被中断，也就是说CAS是一条CPU的原子指令，不会造成所谓的数据不一致性问题。==

#### 2. 关联知识

>CPU通过总线和内存进行数据传输的。多个核心通过同一条总线和内存以及其他硬件进行通信。
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gmmbi2g4pyj30fw08eq2z.jpg" style="zoom:100%">
>
>在上图中，CPU 通过两个蓝色箭头标注的总线与内存进行通信。大家考虑一个问题，CPU 的多个核心同时对同一片内存进行操作，若不加以控制，会导致什么样的错误?这里简单说明一下，假设核心1经32位带宽的总线向内存写入64位的数据，核心1要进行两次写入才能完成整个操作。若在核心1第一次写入32位的数据后，核心2从核心1写入的内存位置读取了64位数据。由于核心1还未完全将64位的数据全部写入内存中，核心2就开始从该内存位置读取数据，那么读取出来的数据必定是混乱的。
>
>不过对于这个问题，实际上不用担心。通过 Intel 开发人员手册，我们可以了解到自奔腾处理器开始，==Intel 处理器会保证以原子的方式读写按64位边界对齐的四字(quadword)==。
>
>根据上面的说明，我们可总结出，Intel 处理器可以保证单次访问内存对齐的指令以原子的方式执行。但如果是两次访存的指令呢？答案是无法保证。比如递增指令inc dword ptr [...]，等价于DEST = DEST + 1。该指令包含三个操作读-改-写，涉及两次访存。考虑这样一种情况，在内存指定位置处，存放了一个为1的数值。现在 CPU 两个核心同时执行该条指令。两个核心交替执行的流程如下：
>
>```
>核心1 从内存指定位置出读取数值1，并加载到寄存器中
>核心2 从内存指定位置出读取数值1，并加载到寄存器中
>核心1 将寄存器中值递减1
>核心2 将寄存器中值递减1
>核心1 将修改后的值写回内存
>核心2 将修改后的值写回内存
>```
>
>经过执行上述流程，内存中的最终值时2，而我们期待的是3，这就出问题了。要处理这个问题，就要==避免两个或多个核心同时操作同一片内存区域==。那么怎样避免呢？这就要引入本文的主角 - ==lock 前缀==。关于该指令的详细描述，可以参考 Intel 开发人员手册 Volume 2 Instruction Set Reference，Chapter 3 Instruction Set Reference A-L。我这里引用其中的一段，如下：
>
>```
>LOCK—Assert LOCK# Signal Prefix
>LOCK-设置LOCK＃信号前缀
>
>Causes the processor’s LOCK# signal to be asserted during execution of the accompanying instruction (turns the instruction into an atomic instruction). In a multiprocessor environment, the LOCK# signal ensures that the processor has exclusive use of any shared memory while the signal is asserted.
>使处理器的LOCK＃信号在执行伴随指令的过程中被声明（将指令转换为原子指令）。在多处理器环境中，LOCK＃信号可确保在断言该信号时处理器拥有对任何共享内存的独占使用。
>```
>
>在多处理器环境下，==LOCK# 信号可以确保处理器独占使用某些共享内存==。lock 可以被添加在下面的指令前：
>
>ADD, ADC, AND, BTC, BTR, BTS, CMPXCHG, CMPXCH8B, CMPXCHG16B, DEC, INC, NEG, NOT, OR, SBB, SUB, XOR, XADD, and XCHG.
>
>通过在 inc 指令前添加 lock 前缀，即可让该指令具备原子性。多个核心同时执行同一条 inc 指令时，会以串行的方式进行，也就避免了上面所说的那种情况。那么这里还有一个问题，==lock 前缀是怎样保证核心独占某片内存区域的呢？==
>
>在 Intel 处理器中，有两种方式保证处理器的某个核心独占某片内存区域。
>
>1. 第一种方式是通过==锁定总线==，让某个核心独占使用总线，但这样代价太大。总线被锁定后，其他核心就不能访问内存了，可能会导致其他核心短时内停止工作。
>2. 第二种方式是==锁定缓存==，若某处内存数据被缓存在处理器缓存中。处理器发出的 LOCK# 信号不会锁定总线，而是锁定缓存行对应的内存区域。其他处理器在这片内存区域锁定期间，无法对这片内存区域进行相关操作。相对于锁定总线，锁定缓存的代价明显比较小。

#### 3. 源码分析

>在Java中，并没有直接实现CAS，CAS相关的实现是通过C++内联汇编的形式实现的。Java代码需通过JNI才能调用。
>
>对 java.util.concurrent.atomic 包下的原子类 `AtomicInteger` 中的 `compareAndSet` 方法进行分析，相关分析如下：
>
>```java
>public class AtomicInteger extends Number implements java.io.Serializable {
>    private static final long serialVersionUID = 6214790243416807050L;
>
>    // 设置为使用Unsafe.compareAndSwapInt进行更新
>    private static final Unsafe unsafe = Unsafe.getUnsafe();
>    private static final long valueOffset; 
>
>    static {
>        try {
>            valueOffset = unsafe.objectFieldOffset
>                (AtomicInteger.class.getDeclaredField("value"));
>        } catch (Exception ex) { throw new Error(ex); }
>    }
>
>    private volatile int value;
>
>    public final boolean compareAndSet(int expect, int update) {
>        return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
>    }
>
>}
>```
>
>如上，AtomicInteger类内部维护了`private volatile int value`和`private static final Unsafe unsafe`两个比较重要的参数。还有变量`valueOffset`——表示该变量在内存地址中的==偏移地址==，因为<u>Unsafe就是根据内存偏移地址获取数据的</u>。变量value用==volatile==修饰，保证了多线程之间的内存可见性。
>
>Unsafe类：
>
>```java
>public final class Unsafe {
>
>    // native类型的方法
>    public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
>
>    public final int getAndAddInt(Object var1, long var2, int var4) {
>        int var5;
>        do {
>            var5 = this.getIntVolatile(var1, var2);
>        } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
>
>        return var5;
>    }
>}
>```
>
>AtomicInteger.compareAndSwapInt()调用了Unsafe.compareAndSwapInt()方法。Unsafe类的大部分方法都是native的，用来像C语言一样从底层操作内存。
>
>这个方法的var1和var2，就是根据==对象==和==偏移量==得到在==主内存的快照值var5==。然后==`compareAndSwapInt`方法通过var1和var2得到当前主内存的实际值==。如果这个实际值跟快照值相等，那么就更新主内存的值为var5+var4。如果不等，那么就一直循环，一直获取快照，一直比对，直到实际值和快照值相等为止。
>
>```
>比如有A、B两个线程：
>1）一开始都从主内存中拷贝了原值为3；
>2）A线程执行到var5=this.getIntVolatile，即var5=3。此时A线程挂起；
>3）B修改原值为4，B线程执行完毕，由于加了volatile，所以这个修改是立即可见的；
>4）A线程被唤醒，执行this.compareAndSwapInt()方法，发现这个时候主内存的值不等于快照值3，所以继续循环，重新从主内存获取。
>5）线程A重新获取value值，因为变量value被volatile修饰，所以其他线程对它的修改，线程A总是能够看到，线程A继续执行compareAndSwapInt进行比较替换，直至成功。
>```

##### AtomicReference原子引用

>可以用`AtomicReference`来包装这个POJO，==使其操作原子化==。
>
>```
>User user1 = new User("Jack",25);
>User user2 = new User("Lucy",21);
>AtomicReference<User> atomicReference = new AtomicReference<>();
>atomicReference.set(user1);
>System.out.println(atomicReference.compareAndSet(user1,user2)); //true 
>System.out.println(atomicReference.compareAndSet(user1,user2));//false 
>```
>
>本质是比较的是两个对象的地址是否相等。

#### 4. CAS的ABA问题

>因为CAS需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有发生变化，但是实际上却变化了。
>
>比如线程A操作值a，将值修改为b，这时线程B也拿到了a的值，将a改为b，再改为a，这时线程A比对值的时候，发送值还是和expect的一样，就会继续操作。

##### 导致ABA问题的原因

>Compare And Swap 过程中==只是简单进行了“值”的校验==，在有些情况下，“值”相同不会引入错误的业务逻辑（例如库存），有些情况下，“值”虽相同，却已经不是原来的数据。

##### AtomicStampedReference和ABA问题的解决

>CAS不能只比对值，一个数据一个版本，版本变化，即使值相同，也不应该修改成功。
>
>实践：==“版本号”的对比==，一个数据一个版本。
>
>使用`AtomicStampedReference`类可以解决ABA问题。这个类维护了一个“版本号”stamp，其实有点类似乐观锁的意思。==在进行CAS操作的时候，不仅要比较当前值，还要比较版本号==。只有两者都相等，才执行更新操作。
>
>```java
>public class AtomicStampedReference<V> {
>
>    private static class Pair<T> {
>        final T reference;
>        final int stamp;
>        private Pair(T reference, int stamp) {
>            this.reference = reference;
>            this.stamp = stamp;
>        }
>        static <T> Pair<T> of(T reference, int stamp) {
>            return new Pair<T>(reference, stamp);
>        }
>    }
>
>    private volatile Pair<V> pair;
>    public boolean compareAndSet(V   expectedReference,
>                                 V   newReference,
>                                 int expectedStamp,
>                                 int newStamp) {
>        Pair<V> current = pair;
>        return
>            expectedReference == current.reference &&
>            expectedStamp == current.stamp &&
>            ((newReference == current.reference &&
>              newStamp == current.stamp) ||
>             casPair(current, Pair.of(newReference, newStamp)));
>    }
>}
>```

#### 5. CAS总结

>CAS缺点：
>
>- 实际上是一种自旋锁，一直循环，开销比较大。
>- 只能保证一个变量的原子操作，多个变量依然要加锁。
>- 引出了ABA问题（AtomicStampedReference可解决）。
>
>使用场景：适合在一些并发量不高、线程竞争较少的情况，加锁太重。但是一旦线程冲突严重的情况下，循环时间太长，为给CPU带来很大的开销。

