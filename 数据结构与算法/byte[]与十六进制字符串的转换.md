二进制数据（包括内存地址）在计算机中一般以十六进制的方式表示。



MD5信息摘要算法返回的结果是一个==128bit的二进制数据==，128bit是16个byte，而一个byte转成16进制正好是2位（==16进制使用4个bit==，一个byte有8个bit），所以MD5算法返回的128bit转成16进制正好是==32位==。



MD5信息摘要之后，返回一个有16个字节的字节数组，需要把这个字节数组转成16进制格式的数据。



#### 1. 字节数组转十六进制字符串

---

>算法思想：
>
>- 把一个字节分为==高4位==和==低4位==；
>- 先取高四位的bit，得到这4bit对应的数字（0~15），就可以知道对应的16进制数是多少了；按同样的方式取到低4位对应的16进制数，把这些16进制数放到一个数组里，最后串成一个字符串。

<img src="https://tva1.sinaimg.cn/large/008i3skNgy1gsm00g1iltj30s20fudhd.jpg" style="zoom:80%;" />

```java
private static final char HEXCHARARR[] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f'};

public static String byteArrToHex(byte[] btArr) {
  char[] strArr = new char[btArr.length * 2];
  int i = 0;
  for (byte bt : btArr) {
    // 取每一个字节的高四位对应的16进制数
    strArr[i++] = HEXCHARARR[bt >>> 4 & 0xf];
    // 取每一个字节的低四位对应的16进制数
    strArr[i++] = HEXCHARARR[bt & 0xf];
  }
  return new String(strArr);
}
```



#### 2. 十六进制字符串转字节数组

---

>算法思想：
>
>- 把16进制字符串分为一个char数组；
>- 循环取其中的两个char，这两个char的值一定都是在【0123456789abcdef】之间；
>- 先找到第一个char对应的下标位置，比如说char的值为a，那么下标位置就是 10，把数字10转成byte类型，取低4位；
>- 然后用同样的方式找到第二个char对应的下标数字，取到该数字的低4位bit；
>- 将刚才取到的两个低4位bit拼在一起，第一个char对应的4位bit为字节的高4位，第二个char对应的4位bit为字节的低4位，这样组成一个完整的8bit字节；
>- 循环完char数组，就得到了16进制字符串对应的字节数组。

```java
private static final String HEXSTR = "0123456789abcdef";

public static byte[] hexToByteArr(String hexStr) {
  // 转换成char数组
  char[] charArr = hexStr.toCharArray();
  byte[] btArr = new byte[charArr.length / 2];
  int index = 0;
  for (int i = 0; i < charArr.length; i++) {
    int highBit = HEXSTR.indexOf(charArr[i]);
    int lowBit = HEXSTR.indexOf(charArr[++i]);
    btArr[index] = (byte) (highBit << 4 | lowBit);
    index++;
  }
  return btArr;
}
```

