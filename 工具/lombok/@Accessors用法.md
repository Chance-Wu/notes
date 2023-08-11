### 一、@Accessors(chain = true)

---

开启链式编程  设置chain=true时，setter方法返回的是this（也就是对象自己），代替了默认的返回值void，直接链式操作对象。

示例：

```java
@Data
@Accessors(chain = true) //开启链式编程
public class User implements Serializable {
  private String id;
  private String name;
  private int age;
}
```

```java
public static void main(String[] args) {
  User user = new User();
  user.setId("123").setAge(17).setName("小明");
  System.out.println(user);
}
```



### 二、@Accessors(fluent=true)

---

省略给对象赋值和取值时候得set、get前缀。

示例：

```java
@Data
@Accessors(fluent = true)   //不用带set和get前缀
public class User implements Serializable {
  private String id;
  private String name;
  private int age;
}
```

```java
public static void main(String[] args) {
  User user = new User();
  user.id("124").age(19).name("小丽");
  System.out.println(user);
}
```