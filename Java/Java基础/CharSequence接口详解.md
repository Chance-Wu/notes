#### 1. CharSequence接口

---

描述字符串结构的接口。



只要有字符串就可以为CharSequence实例化，定义了如下方法：

- charAt(int index)：获取指定索引的字符；
- length()：获取字符串长度；
- subSequence(int start, int end)：截取部分字符串



#### 2. 常用实现类

---

- String
- StringBuffer（线程安全）
- StringBuilder

