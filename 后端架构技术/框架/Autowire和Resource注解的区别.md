@Resource和@Autowired都是做bean的注入时使用，其实@Resource并不是Spring的注解，它的包是`javax.annotation.Resource`，需要导入，但是Spring支持该注解的注入。

| 对比项   | @Autowired               | @Resource                            |
| -------- | ------------------------ | ------------------------------------ |
| 注解来源 | Spring注解               | JDK注解（JSR-250标准注解，属于J2EE） |
| 装配方式 | 优先按类型               | 优先按名称                           |
| 属性     | required                 | name、type                           |
| 作用范围 | 字段、setter方法、构造器 | 字段、setter方法                     |



#### 1. 共同点

---

两者都可以写在字段和setter方法上。两者如果都写在字段上，那么就不需要再写setter方法。



#### 2. 不同点

---

>@Autowired为Spring提供的注解，需要导入包`org.springframework.beans.factory.annotation.Autowired;`只按照byType注入。
>
>```java
>public class TestServiceImpl {
>  // 下面两种@Autowired只要使用一种即可
>  @Autowired
>  private UserDao userDao; // 用于字段上
>
>  @Autowired
>  public void setUserDao(UserDao userDao) {
>    // 用于属性的方法上
>    this.userDao = userDao;
>  }
>}
>```
>
>==@Autowired注解是按照类型（byType）装配依赖对象，默认情况下它要求依赖对象必须存在==，如果允许null值，可以设置它的required属性为false。==如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用==。如下：
>
>```java
>public class TestServiceImpl {
>  @Autowired
>  @Qualifier("userDao")
>  private UserDao userDao;
>}
>```
>
>经常可以在IDEA中看到关于@Autowired注解的黄牌警告，如下：
>
>```java
>@Autowire
>private JdbcTemplate jdbcTemplate;
>```
>
>提示的警告信息
>
>```
>Field injection is not recommended Inspection info: Spring Team recommends: "Always use constructor based dependency injection in your beans. Always use assertions for mandatory dependencies".
>```
>
>大致翻译一下：
>
>属性字段注入的方式不推荐，检查到的问题是：Spring团队建议:"始终在bean中使用基于构造函数的依赖项注入，始终对强制性依赖项使用断言"。

>@Resource 是JDK1.6支持的注解，由J2EE提供，需要导入包`javax.annotation.Resource`。默认按照名称进行装配，名称可以通过name属性进行指定。也提供按照byType 注入。
>
>@Resource有两个重要的属性：name 和 type，而Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。
>
>```java
>@Resource(name = "userDaoImpl2",type = UserDaoImpl.class)
>private UserDao userDao;
>```
>
>如果没有指定name属性，当注解写在字段上时，默认取字段名，按照名称查找。
>
>当注解标注在属性的setter方法上，即默认取属性名作为bean名称寻找依赖对象。
>
>当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。
>
>```java
>public class TestServiceImpl {
>  // 下面两种@Resource只要使用一种即可
>  @Resource(name="userDao")
>  private UserDao userDao; // 用于字段上
>
>  @Resource(name="userDao")
>  public void setUserDao(UserDao userDao) { // 用于属性的setter方法上
>    this.userDao = userDao;
>  }
>}
>```
>
>注：最好是将@Resource放在setter方法上，因为这样更符合面向对象的思想，通过set、get去操作属性，而不是直接去操作属性。



#### 3. @Resource装配顺序

---

①如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常。

②如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常。

③如果指定了type，则从上下文中找到类似匹配的唯一bean进行装配，找不到或是找到多个，都会抛出异常。

④如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配。

@Resource的作用相当于@Autowired，只不过@Autowired按照byType自动注入。