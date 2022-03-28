是所有Java类的根父类。



#### 1. ==

---

- 基本类型比较值：只要两个变量的值相等，即为true；
- 引用类型比较引用（是否指向同一个对象）；
- 用"=="进行比较时，符号两边的数据类型必须兼容（可自动转换的基本数据类型除外），否则编译出错。



#### 2. equals方法

---

- 所有类都继承了Object，也就获得了equals()方法。还可以重写。
- 只能比较引用类型，比较是否指向同一个对象。
- 特例：当用equals方法进行比较时，对类File、String、Date及包装类（Wrapper Class）来说，是比较类型及内容而不考虑引用的是否是同一个对象；原因：在这些类中重写了Object类的equals()方法。



#### 3. toString方法

---

- toString()方法在Object类中定义，其返回值是String类型，返回类名和它的引用地址。

- 在进行String与其它类型数据连接操作时，自动调用toString()方法。

  ```java
  Date now=new Date();
  System.out.println(“now=”+now);  
  // 相当于 System.out.println(“now=”+now.toString());
  ```

- 可以根据需要在用户自定义类型中重写toString()方法。如String类重写了toString()方法，返回字符串的值。

- 基本类型数据转换为String类型时，调用了对应包装类的toString()方法。

  ```java
  int a = 10;
  System.out.println(“a=” + a);
  ```

​	像String类、包装类、File类、Date类等，已经实现了toString()方法的重写。
