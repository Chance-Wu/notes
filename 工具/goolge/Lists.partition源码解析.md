```java
// 例：将集合：list 分割成每个集合25条数据的集合。
List<String> list = new ArrayList<>(100);
for (int i = 0; i < 100; i++) {
  list.add(String.valueOf(i));
}
List<List<String>> partition = Lists.partition(list , 25);
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





























