#### 1. ArrayList 和 LinkedList 的区别

---

>- ArrayList数据结构是==数组==，所以查询快，增删慢，可以通过索引进行查找，而增删慢的原因是要<u>*不断创建新数组扩容*</u>；
>- LinkedList数据结构是==链表==，所以增删快，查找慢，因为链表之间首尾相连，<u>*查找时需要移动指针*</u>，而增删快的原因是不需要频繁创建新结构来扩容，只需要修改节点之间的引用关系即可。



#### 2. ArrayList源码

---

```java
/**
 * 初始长度为10，如果已知数据长度，最好一开始就设置好，这样可以避免扩容
 */
private static final int DEFAULT_CAPACITY = 10;

>add方法：
>
>```java
>public boolean add(E e) {
>  ensureCapacityInternal(size + 1);  // 增加modCount！！
>  elementData[size++] = e;
>  return true;
>}
>```
>
>在添加元素时，ArrayList会先判断是否需要扩容：
>
>```java
>private void ensureCapacityInternal(int minCapacity) {
>  ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
>}
>
>private void ensureExplicitCapacity(int minCapacity) {
>  modCount++;
>
>  if (minCapacity - elementData.length > 0)
>    grow(minCapacity);
>}
>```
>
>- 首先，modCound自增，记录当前集合变化的次数；
>- 然后判断当前增加一个元素后的集合长度-当前集合的长度是否大于0，如果是，则需要扩容，否则不需要。
>
>```java
>private void grow(int minCapacity) {
>  // overflow-conscious code
>  int oldCapacity = elementData.length;
>  int newCapacity = oldCapacity + (oldCapacity >> 1);
>  if (newCapacity - minCapacity < 0)
>    newCapacity = minCapacity;
>  if (newCapacity - MAX_ARRAY_SIZE > 0)
>    newCapacity = hugeCapacity(minCapacity);
>  // minCapacity is usually close to size, so this is a win:
>  elementData = Arrays.copyOf(elementData, newCapacity);
>}
>```
>
>扩容步骤：
>
>1. 计算当前集合的长度
>
>2. 计算新长度 = 当前集合长度 + 当前容器长度右移运算1位后的新集合长度，也就是当前集合长度的1.5倍
>
>3. 判断新长度 - 当前集合所需长度是否小于0，如果是则表示当前集合扩容1.5倍无法满足当前集合所需长度，所以将当前集合所需长度的值赋给新长度
>
>4. 判断新集合长度 - (Integer最大值 - 8) 是否大于0，如果是则进入hugeCapacity(minCapacity)方法
>
>   ```java
>   private static int hugeCapacity(int minCapacity) {
>     if (minCapacity < 0) // 小于0，是因为minCapacity的值大于Integer所允许的最大值，则抛出内存溢出异常
>       throw new OutOfMemoryError();
>     // 判断当前所需长度是否大于集合最大长度，如果是，则将Integer所允许的最大值赋值给集合的新长度，否则使用允许的最大集合长度
>     return (minCapacity > MAX_ARRAY_SIZE) ?
>       Integer.MAX_VALUE :
>     MAX_ARRAY_SIZE;
>   }
>   ```
>
>5. 将当前数组按照新长度完全复制一份给新数组
>
>   ```java
>   elementData = Arrays.copyOf(elementData, newCapacity);
>   ```

>ArrayList集合的最大长度为什么为Integer.MAX_VALUE - 8？ ==因为存储数组需要8字节，所以需要减8==。



#### 3. LinkedList源码

---

- 链表的集合，每一个节点都保存了当前节点的数据和上下连接的两个节点的实例。
- 没有初始大小，因为每一个都是首尾连接的Node

>add方法：
>
>```java
>public boolean add(E e) {
>  linkLast(e);
>  return true;
>}
>```
>
>```java
>void linkLast(E e) {
>  final Node<E> l = last;
>  final Node<E> newNode = new Node<>(l, e, null);
>  last = newNode;
>  if (l == null)
>    first = newNode;
>  else
>    l.next = newNode;
>  size++;
>  modCount++;
>}
>```
>
>1. 获取当前集合的末端节点；
>2. 创建一个新节点，参数由左至右依次：上一节点，当前节点数据，下一节点；
>3. 将新的节点设置为末端节点；
>4. 如果上一个末端节点为空，则证明这是当前集合的第一个节点，则将新节点设置为头部节点。否则证明当前集合之前已有数据，则将新节点与上一个末端节点连接起来；
>5. 当前集合长度+1；
>6. 记录集合变化树+1。

