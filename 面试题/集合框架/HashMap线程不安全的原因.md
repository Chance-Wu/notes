#### 1.jdk 1.8 中的线程不安全

>并发执行put操作会出现==数据覆盖==的情况。
>
>```java
>final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
>               boolean evict) {
>  Node<K,V>[] tab; Node<K,V> p; int n, i;
>  if ((tab = table) == null || (n = tab.length) == 0)
>    n = (tab = resize()).length;
>  if ((p = tab[i = (n - 1) & hash]) == null) // 如果没有hash碰撞则直接插入元素
>    tab[i] = newNode(hash, key, value, null);
>  else {
>    Node<K,V> e; K k;
>    if (p.hash == hash &&
>        ((k = p.key) == key || (key != null && key.equals(k))))
>      e = p;
>    else if (p instanceof TreeNode)
>      e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
>    else {
>      for (int binCount = 0; ; ++binCount) {
>        if ((e = p.next) == null) {
>          p.next = newNode(hash, key, value, null);
>          if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
>            treeifyBin(tab, hash);
>          break;
>        }
>        if (e.hash == hash &&
>            ((k = e.key) == key || (key != null && key.equals(k))))
>          break;
>        p = e;
>      }
>    }
>    if (e != null) { // existing mapping for key
>      V oldValue = e.value;
>      if (!onlyIfAbsent || oldValue == null)
>        e.value = value;
>      afterNodeAccess(e);
>      return oldValue;
>    }
>  }
>  ++modCount;
>  if (++size > threshold)
>    resize();
>  afterNodeInsertion(evict);
>  return null;
>}
>```
>
>假设两个线程A、B都在进行put操作，并且==hash函数计算出的插入下标是相同的==：
>
>- 当线程A执行完第6行代码后由于时间片耗尽导致被挂起；
>- 线程B得到的时间片后在该下标出插入了元素，完成了正常的插入；
>- 然后线程A获得时间片，由于之前已经进行了hash碰撞的判断，所以此时不会再进行判断，而是直接插入，这就导致了线程B插入的数据被线程A覆盖了，从而线程不安全。
>
>除此之外，第38行++size，线程A、B同时进行put操作时，假设当前HashMap的size大小为10：
>
>- 当线程A执行到38行时，从主内存中获得size的值为10准备进行+1操作，但是由于时间片耗尽只好让出CPU；
>- 线程B拿到CPU还是从主内存中拿到size的值10进行+1操作，完成了put操作并将size=11写回主内存；
>- 然后线程A再次拿到CPU并继续执行(此时size的值仍为10)，当执行完put操作后，还是将size=11写回内存，此时，线程A、B都执行了一次put操作，但是size的值只增加了1，所有说还是由于数据覆盖又导致了线程不安全。

