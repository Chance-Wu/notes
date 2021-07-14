遍历删除List中的元素有很多方法，运用不当的时候就会产生问题。

1. 通过增强的for循环删除符合条件的多个元素
2. 通过增强的for循环删除符合条件的一个元素；
3. 通过普通的for循环删除符合条件的多个元素；
4. 通过Iterator进行遍历删除符合条件的多个元素。



#### 1. fail-fast机制

---

==Java集合（Collection）中的一种错误机制==。当多个线程对同一个集合的内容进行操作时，就可能会产生fail-fast事件。

>例如：当某一个线程A通过iterator去遍历某集合的过程中，若该集合的内容被其他线程所改变了；那么线程A访问集合时，就会抛出`ConcurrentModificationException`异常，产生fail-fast事件。当方法检测到对象的并发修改，但不允许这种修改时就抛出该异常。同时需要注意的是，该异常不会始终指出对象已经由不同线程并发修改，如果单线程违反了规则，同样也有可能会抛出改异常。



当使用fail-fast iterator对Collection或Map进行迭代操作过程中尝试直接修改Collection/Map的内容时，即使是在单线程下运行，java.util.ConcurrentModificationException异常也将被抛出。

Iterator是工作在一个独立的线程中，并且拥有一个mutex锁。Iterator被创建之后会建立一个指向原来对象的单链索引表，当原来的对象数量发生变化时，这个索引表的内容不会同步改变，所以当索引指针往后移动的时候就找不到要迭代的对象，所以按照fail-fast原则Iterator会马上抛出java.util.ConcurrentModificationException异常。

==Iterator在工作的时候是不允许被迭代的对象被改变的。但可以使用Iterator本身的方法remove()来删除对象==，Iterator.remove()方法会在删除当前迭代对象的同时维护索引的一致性。