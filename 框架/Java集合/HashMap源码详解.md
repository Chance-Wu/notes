参考文章：[深入理解HashMap底层原理剖析(JDK1.8)](https://my.oschina.net/u/2307589/blog/1800587)

<img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gmhdhfe4o4j30u80ip0tc.jpg" style="zoom:60%">

探讨HashMap前先了解两种数据结构：数组、链表

#### 1. 数组

>数组具有<u>遍历快</u>，<u>增删慢</u>的特点。
>
>- 数组在堆中是==一块连续的存储空间==，遍历时数组的首地址是知道的。（元素地址=首地址+元素字节数*index）`O(1)`
>- 增删慢是因为，当在中间插入或删除元素时，会造成该元素后面所有元素地址的改变。`O(n)`

#### 2. 链表

>链表具有<u>增删块</u>，<u>遍历慢</u>的特点。
>
>- 链表中各元素的==内存空间是不连续的==，一个节点包含节点数据、前驱节点、后继节点的引用，所以在在插入删除时，只需修改该位置的前驱节点和后继节点的引用即可。`O(1)`
>- 遍历时，get(n)元素，需要从第一个开始，依次拿到后面元素的地址，进行遍历，直到遍历到第n个元素，所以效率低。`O(n)`

#### 3. HashMap

>- HashMap==根据key的hashCode值存储数据==，大多数情况可以直接定位到它的值，因而具有很快的速度，但遍历顺序确实不确定的。
>- HashMap最多==只允许一条记录的键为null==，允许多条记录的值为null。
>- 非线程安全，任一时刻可以有多个线程同时写HashMap，可能会导致数据的不一致，可使用`Collections.synchronizedMap(map)`使map具有线程安全的能力，或者使用`ConcurrentHashMap`。

##### 3.1 存储结构

>数组+链表+红黑树
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gmhdi3rkxtj30qx0kcmxm.jpg" style="zoom:60%">

##### 3.2 Node<K,V> 类

>用来实现数组及链表的数据结构。
>
>```java
>static class Node<K,V> implements Map.Entry<K,V> {
>    final int hash; // 保存节点的 hash 值
>    final K key; // 保存节点的 key 值
>    V value; // 保存节点的 value 值
>    Node<K,V> next; // 指向链表结构下的当前节点的 next 节点，红黑树 TreeNode节点中也有用到。
>
>    Node(int hash, K key, V value, Node<K,V> next) {
>        this.hash = hash;
>        this.key = key;
>        this.value = value;
>        this.next = next;
>    }
>
>    public final K getKey()        { return key; }
>    public final V getValue()      { return value; }
>    public final String toString() { return key + "=" + value; }
>
>    public final int hashCode() {
>        return Objects.hashCode(key) ^ Objects.hashCode(value);
>    }
>
>    public final V setValue(V newValue) {
>        V oldValue = value;
>        value = newValue;
>        return oldValue;
>    }
>
>    public final boolean equals(Object o) {
>        if (o == this)
>            return true;
>        if (o instanceof Map.Entry) {
>            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
>            if (Objects.equals(key, e.getKey()) &&
>                Objects.equals(value, e.getValue()))
>                return true;
>        }
>        return false;
>    }
>}
>```

##### 3.3 TreeNode<K, V> 类

>继承自 LinkedHashMap.Entry<K, V>，用来实现红黑树相关的存储结构：
>
>```java
>/**
>  * 
>  */
>static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
>    TreeNode<K,V> parent;  // 红黑树链接
>    TreeNode<K,V> left;
>    TreeNode<K,V> right;
>    TreeNode<K,V> prev;    // 删除后需要取消链接
>    boolean red; // 存储当前节点的颜色
>    
>    TreeNode(int hash, K key, V val, Node<K,V> next) {
>        super(hash, key, val, next);
>    }
>}
>```

##### 3.4 HashMap 各常量、成员变量作用

>```java
>// 创建 HashMap 时未指定初始容量情况下的默认容量16 1*2*2*2*2   
>static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 
>
>// HashMap 的最大容量 1*2^30
>static final int MAXIMUM_CAPACITY = 1 << 30;
>
>// HashMap 默认的装载因子,当 HashMap 中元素数量超过 容量*装载因子 时，进行　resize()　操作
>static final float DEFAULT_LOAD_FACTOR = 0.75f;
>
>// 用来确定何时将解决 hash 冲突的链表转变为红黑树
>static final int TREEIFY_THRESHOLD = 8;
>
>// 用来确定何时将解决 hash 冲突的红黑树转变为链表
>static final int UNTREEIFY_THRESHOLD = 6;
>
>/* 当需要将解决 hash 冲突的链表转变为红黑树时，需要判断下此时数组容量，若是由于数组容量太小（小于　MIN_TREEIFY_CAPACITY　）导致的 hash 冲突太多，则不进行链表转变为红黑树操作，转为利用　resize() 函数对　hashMap 扩容　*/
>static final int MIN_TREEIFY_CAPACITY = 64;
>
>//保存Node<K,V>节点的数组
>transient Node<K,V>[] table;
>
>//由　hashMap 中 Node<K,V>　节点构成的 set
>transient Set<Map.Entry<K,V>> entrySet;
>
>//记录 hashMap 当前存储的元素的数量
>transient int size;
>
>//记录　hashMap 发生结构性变化的次数（注意　value 的覆盖不属于结构性变化）
>transient int modCount;
>
>//threshold的值应等于 table.length * loadFactor, size 超过这个值时进行　resize()扩容
>int threshold;
>
>//记录 hashMap 装载因子
>final float loadFactor;
>```

##### 3.5 hash算法

>```java
>static final int hash(Object key) {
>    int h;
>    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
>}
>```
>
>- 如果key为null，hash==0
>- 否则
>  - 计算==key的hashCode值==，得到h1
>  - 然后对h1==右移16位==，得到h2
>  - 最后将h1和h2进行==异或运算==，得到最终的hash值

>举例：
>
>1. 先获得key的hashCode值，h1
>
>   `1111 1111 1111 1111 1010 1010 1010 1010`
>
>2. 对hashCode值右移16位，h2
>
>   `0000 0000 0000 0000 1111 1111 1111 1111`
>
>3. 最后将这两个值进行异或运算
>
>   1111 111 1111 1111 0101 0101 0101 0101`
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gmifwz8r6jj30z00ggag1.jpg" style="zoom:50%">

>结论：
>
>1. 最终结果就是将h1的高16位和低16位进行了异或(^)运算。
>2. 达到的效果就是混合原始哈希码的高位和地位，以此来加大低位的随机性。
>3. 顺带着，低16位可以同时拥有高16位的信息。

##### 3.6 put(K key, V value)方法

>```java
>public V put(K key, V value) {
>    return putVal(hash(key), key, value, false, true);
>}
>
>final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
>               boolean evict) {
>    // 获取数组的长度
>    Node<K,V>[] tab; Node<K,V> p; int n, i;
>    if ((tab = table) == null || (n = tab.length) == 0)
>        n = (tab = resize()).length;
>    
>    // 计算元素在数组中的索引位置
>    if ((p = tab[i = (n - 1) & hash]) == null)
>        tab[i] = newNode(hash, key, value, null);
>    else {
>        
>        // 省略部分代码
>    }
>    ++modCount;
>    if (++size > threshold)
>        resize();
>    afterNodeInsertion(evict);
>    return null;
>}
>```
>
>计算元素在数组中的索引位置：
>
>`i = (n - 1) & hash`	==位运算的效率最高==
>
>如果让我们来做，通常会想到用下面==取模的方式计算索引==。
>
>`i = hash % n`
>
>其实，上面两种算法得到的结果是一样的。【前提是HashMap的数组长度要取2的整次幂，这也说明了为什么HashMap的初始容量不能随便给】。

##### 3.7 总结

>1. 当HashMap的数组长度是2的整次幂时，`(n-1) & hash` 与 `hash % n` 计算的结果是等价的。后者是更容易想到的办法，前者是更有效率的办法。
>2. HashMap的hash算法，其实就是把hashCode值的高16为与低16位进行了异或运算，目的就是==增大随机性==。



