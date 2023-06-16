```java
// 例：将集合：list 分割成每个集合25条数据的集合。
List<String> list = new ArrayList<>(100);
for (int i = 0; i < 100; i++) {
  list.add(String.valueOf(i));
}
List<List<String>> partition = Lists.partition(list , 25);
for (List<String> data : partition) {

  //执行插入
  System.out.println(data.size());
}
```

1. 会创建一个RandomAccessPartition对象，RandomAccessPartition继承自partition，partition又继承自`AbstractList<List>`

   ```java
   public static <T> List<List<T>> partition(List<T> list, int size) {
     checkNotNull(list);
     checkArgument(size > 0);
     return (list instanceof RandomAccess)
       ? new RandomAccessPartition<T>(list, size)
       : new Partition<T>(list, size);
   }
   ```

2. 获取大小的时候会调用partition中重写的size方法

   ```java
   public int size() {
     return IntMath.divide(list.size(), size, RoundingMode.CEILING);
   }
   ```

3. 示例代码中foreach循环获取元素会调用重写的get方法，这里面实现了集合的拆分。

   ```java
   public List<T> get(int index) {
     checkElementIndex(index, size());
     int start = index * size;
     int end = Math.min(start + size, list.size());
     return list.subList(start, end);
   }
   ```

4. get方法会调用AbstractLists的subList方法

   ```java
   public List<E> subList(int fromIndex, int toIndex) {
     return (this instanceof RandomAccess ?
             new RandomAccessSubList<>(this, fromIndex, toIndex) :
             new SubList<>(this, fromIndex, toIndex));
   }
   ```



### 注意

如果对也就是对子集合的操作会反映到原集合， 对原集合的操作也会影响子集合。

对于这种问题，有两种解决方案：

1. 使用Lists.newArrayList(list)重新建一个list；
2. 在接口设计时，参数尽量使用Array，而不是list。