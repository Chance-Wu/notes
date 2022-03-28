Collection 是一个集合接口。 它提供了对集合对象进行基本操作的通用接口方法。Collection接口在Java 类库中有很多具体的实现。是list，set等的父接口。

Collections 是一个包装类。 它包含有各种有关集合操作的静态多态方法。此类不能实例化，就像一个工具类，服务于Java的Collection框架。



#### 1. Collection接口

---

- Collection接口是List、Set和Queue接口的父接口，该接口里定义的方法既可用于操作`Set`集合，也可用于操作`List`和`Queue`集合。
- JDK不提供此接口的任何直接实现，而是提供更具体的子接口(如：Set和List)实现。
- 在JDK5之前，Java集合会丢失容器中所有对象的数据类型，把所有对象都当成Object类型处理；从Java5增加了泛型以后，Java集合可以记住容器中对象的数据类型。

```java
Collection coll = new ArrayList();
```

- size();  返回集合中元素的个数

- add(Object obj);   向集合中添加一个元素
- addAll(Collection coll);   将形参coll中包含的所有元素添加到当前集合
- isEmpty();   判断集合是否为空
- clear();   清空集合元素
- contains(Object obj);   判断集合中是否包含指定的obj元素，包含则返回true。（判断的依据，根据元素所在的类的equals()方法进行判断。自定义类要重写）
- retainAll(Collection coll);   当前集合与coll的共有的元素返回给当前集合。
- remove(Object obj);   删除集合中的obj元素，若删除成功，则返回true。
- removeAll(Collection coll);   从当前集合中删除包含在coll中的元素。equals();   判断集合中的所有元素是否完全相同。
- hashCode();   
- toArray();   将集合转化为数组
- iterator();   返回一个iterator接口实现类的对象，进而实现集合的遍历。

 

#### 2. Collections工具类的使用

---

- Collections是一个操作Set、List和Map等集合的工具类。
- Collections中提供了一系列静态的方法集合元素进行排序、查询和修改等操作，还提供了对集合对象设置不可变、对集合对象实现同步控制等方法。

##### 2.1 排序操作

- reverse(List)——反转List中元素的顺序。
- shuffle(List)——对List集合元素进行随机排序。
- sort(List)——根据元素的自然顺序对指定List集合元素按升序排序。
- sort(List，Comparator)——根据指定的Comparator产生的顺序对List集合元素进行排序。
- swap(List，int，int)——将指定list集合中的i处元素和j处元素进行交换。

##### 2.2 查找、替换

- Object max(Collection)——根据元素的自然顺序，返回给定集合中的最大元素。
- Object max(Collection，Comparator)——根据Comparator指定的顺序，返回给定集合中的最大元素。
- Object min(Collection)
- Object min(Collection，Comparator)
- int frequency(Collection，Object)——返回指定集合中指定元素的出现次数。
- void copy(List dest,List src)——将src中的内容复制到dest中。
- boolean replaceAll(List list，Object oldVal，Object newVal)——使用新值替换List对象的所有旧值。

##### 2.3 同步控制

Collections类中提供了多个synchronizedXxx()方法，该方法可**将指定集合包装成线程同步的集合**，从而可以解决多线程并发访问集合时的线程安全问题。