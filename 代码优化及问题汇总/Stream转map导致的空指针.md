### 一、问题

---

Java中，`Collectors.toMap()` 方法用于将流（Stream）收集到一个 Map 中。然而，使用这个方法时可能会遇到空指针异常（NullPointerException），这通常是因为以下几种原因之一：

1. **键或值为null**： 如果流中的元素映射出的键或值为 null，那么 toMap 会抛出空指针异常。你可以通过提供合并函数（merge function）和指定如何处理键冲突来避免这个问题，但是 null 键仍然不被允许。
2. **未指定合并函数**： 当流中存在重复的键，并且没有提供用于解决键冲突的合并函数时，toMap 也会抛出空指针异常。你需要提供一个合并函数来定义当遇到相同键时应该怎么做。
3. **下游收集器的结果为null**： 如果你使用了下游收集器，并且它返回了 null，这也可能导致空指针异常。



### 二、解决方法

---

- 确保键和值不会是null。（如果不能保证，考虑过滤掉可能产生null的元素）
- 提供一个合并函数来处理可能得键冲突。例如，`(existingValue，newValue) -> existingValue` 将总是保留第一个值。
  - 使用Collectors.toMap的其他重载形式，它们允许指定一个 `BinaryOperator<U>` 来合并具有相同键的值，或者指定一个Supplier来创建Map实例。

```java
List<User> users = new ArrayList<>();
        User user1 = new User("WCY", "WCY");
        User user2 = new User("CHANCE", "CHANCE");
        User user3 = new User("CHANCE", "WCY");
        User user4 = new User("YANG", null);
        users.add(user1);
        users.add(user2);
        users.add(user3);
        users.add(user4);

        Map<String, String> collect = users.stream()
                .filter(ele -> ele.getUsername() != null && ele.getPassword() != null)
                .collect(Collectors.toMap(User::getUsername, User::getPassword, (k1, k2) -> k2));
        System.out.println(collect);

// 输出：
// {WCY=WCY, CHANCE=WCY}
```

