#### 1. 迭代器模式

---

**迭代器模式：就是提供一种方法对一个容器对象中的各个元素进行访问，而又不暴露该对象容器的内部细节。**

Java集合框架的集合类，有时候称之为容器。容器的种类有很多种，比如ArrayList、LinkedList、HashSet...，每种容器都有自己的特点，ArrayList底层维护的是一个数组；LinkedList是链表结构的；HashSet依赖的是哈希表，每种容器都有自己特有的数据结构。

因为容器的内部结构不同，很多时候可能不知道该怎样去遍历一个容器中的元素。所以为了使对容器内元素的操作更为简单，==Java引入了迭代器模式。把访问逻辑从不同类型的集合类中抽取出来，从而避免向外部暴露集合的内部结构==。

java.util.Iterator接口，提供了迭代的基本规则。

```
public interface Iterator<E> {
    /**
     * 判断是否存在下一个对象元素
     */
    boolean hasNext();

    /**
     * 获取下一个元素
     */
    E next();

    /**
     * 移除元素
     */
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }

    /**
     * 
     */
    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```



#### 2. Iterable接口

---

Java中还提供了一个Iterable接口，Iterable接口实现后的功能是“返回”一个迭代器，我们常用的实现了该接口的子接口有:Collection<E>、List<E>、Set<E>等。该接口的==iterator()方法返回一个标准的Iterator实现==。==实现Iterable接口允许对象成为Foreach语句的目标。就可以通过foreach语句来遍历你的底层序列==。

>在使用Iterator的时候禁止对所遍历的容器进行改变其大小结构的操作。
>
>在ArrayList中modCount是当前集合的版本号，每次修改(增、删)集合都会加1；expectedModCount是当前迭代器的版本号，在迭代器实例化时初始化为modCount。我们看到在checkForComodification()方法中就是在验证modCount的值和expectedModCount的值是否相等，所以当你在调用了ArrayList.add()或者ArrayList.remove()时，只更新了modCount的状态，而迭代器中的expectedModCount未同步，因此才会导致再次调用Iterator.next()方法时抛出异常。

>为什么使用Iterator.remove()就没有问题呢？通过源码的第32行发现，在Iterator的remove()中同步了expectedModCount的值，所以当你下次再调用next()的时候，检查不会抛出异常。



#### 3. for循环与迭代器的对比

---

- ArrayList对随机访问比较快，而for循环中使用的get()方法，采用的即是随机访问的方法，因此在ArrayList里for循环快。
- LinkedList则是顺序访问比较快，Iterator中next()方法采用的是顺序访问方法，因此在LinkedList里使用Iterator较快。











































