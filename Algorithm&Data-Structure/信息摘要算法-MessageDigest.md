#### 1. MessageDigest类

---

该类为应用程序==提供信息摘要算法==的功能，如MD5或SHA算法。信息摘要是安全的==单向哈希函数==，它接收任意大小的数据，输出固定长度的哈希值。简单点说就是用于生成散列码。

- 通过`getInstance()`系列静态函数来进行实例化和初始化。
- 该对象通过使用`update`方法处理数据。任何时候都可以调用`reset`方法重置摘要。
- 一旦所需更新的数据都已经被更新了，应该调用`digest`方法之一完成哈希计算并返回结果。

>对于给定数量的更新数据，digest方法只能被调用一次，digest被调用后，MessageDigest对象被重新设置成其初始状态。



#### 2. 使用

---

1. ==创建MessageDigest对象==：（调用程序可选择指定提供者名称，以保证所要求的算法是由已命名提供者实现的，调用getInstance返回已初始化过的MessageDigest对象）

   ```java
   // 不区分算法的大小写
   MessageDigest.getInstance("SHA");
   MessageDigest.getInstance("MD5");
   ```

2. 向已初始化的MessageDigest对象提供要计算的数据。（这将通过一次或多次调用以下某个update方法来完成）

   ```java
   MessageDigest md5 = MessageDigest.getInstance("MD5");
   // 传入需要计算的字符串
   md5.update(x.getBytes("UTF8"));
   ```

3. 调用一下某个digest方法来计算摘要（即生成散列码）

   ```java
   public byte[] digest();
   public byte[] digest(byte[] input);
   public int digest(byte[] buf, int offset, int len);
   ```

   前两个方法返回计算出的摘要。后一个方法把计算出的摘要储存在所提供的 buf 缓冲区中，起点是 offset。len 是 buf 中分配给该摘要的字节数。该方法返回实际存储在 buf 中的字节数。



具体编码步骤如下：

```java
public static void main(String[] args) throws Exception {

  String x = "abc";
  byte[] bytes = messageDigest(x);
  String hexString = convertToHexString(bytes);
  System.out.println(hexString);
}

public static byte[] messageDigest(String x) throws Exception {

  // 生成MessageDigest对象
  MessageDigest md5 = MessageDigest.getInstance("MD5");
  // 传入需要计算的字符串
  md5.update(x.getBytes("UTF8"));
  // 计算信息摘要
  byte[] digest = md5.digest();
  return digest;
}

/**
 * 处理计算结果 转换成十六进制字符串
 *
 * @param data
 * @return
 */
public static String convertToHexString(byte data[]) {
  StringBuffer strBuffer = new StringBuffer();
  for (int i = 0; i < data.length; i++) {
    strBuffer.append(Integer.toHexString(0xff & data[i]));
  }
  return strBuffer.toString();
}
```