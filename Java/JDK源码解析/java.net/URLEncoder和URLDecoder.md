>**URLDecoder**和**URLEncoder**它的作用主要是==用于普通字符串和application/x-www-form-rulencoded MIME字符串之间的转换==。
>
>- URLDecoder类包含一个`decode(String s,String charcter)`静态方法，它可以将看上去乱码的特殊字符串转换成普通字符串。
>- URLEncoder类包含一个`encode(String s,String charcter)`静态方法,它可以将普通字符串转换成application/x-www-form-urlencoded MIME字符串。
>
>```java
>//将application/x-www-form-urlencoded字符串转换成普通字符串
>String str = URLDecoder.decode("a1.+%E7%BD%91%E7%BB%9C", "UTF-8");
>//将普通字符串转换成application/x-www-form-urlencoded字符串
>String urlStr = URLEncoder.encode("a1. 网络", "UTF-8");

#### 1. URLEncoder

---

对String编码时，使用以下规则：

- 字母数字字符 “a” 到 “z”、“A” 到 “Z” 和 “0” 到 “9” 保持不变。
- 特殊字符 “.”、"-"、"*" 和 "_"保持不变。
- 空格字符 " " 转换为一个加号 “+”。
- 所有其他字符都是不安全的，因此首先使用一些编码机制将它们转换为一个或多个字节。然后每个字节用一个包含 3 个字符的字符串 "%xy"表示，其中 xy 为该字节的两位十六进制表示形式。推荐的编码机制是 UTF-8。

>在URI的最初设计时，希望能够通过书面转录，因此URI的构成字符必须是可写的ASCII字符。在这些可书写的字符里，由于一些字符在不同操作系统的编码有不同的解析，被包含在“不安全字符”之中，要格外注意。
>
>需要被转化的字符：
>
>1. ASCII的控制字符
>
>   这些字符是不可打印的。
>
>2. 一些非ASCII字符
>
>   这些字符是非法的字符范围。
>
>3. 一些保留字符
>
>   最常见的就是“&”，这个如果出现在url中了，那你认为是url中的一个字符呢，还是特殊的参数分割用的呢？
>
>4. 一些不安全的字符
>
>   例如：空格。为了防止引起歧义，需要被转化为“+”。
>
>URLEncoder只是为了url中一些非ASCII字符，可以正确无误的被传输。

#### 2. 总结

---

当URL地址中仅包含普通非中文字符串和application/x-www-form-urlencoded MIME字符串无须转换，而包含中文字符串的普通字符串则需要转换，换句话说，也就是说URL地址中有"中文字符串"传递时，才会考虑用到上面提到的两个类，这样就可以将传递过来的中文接受后，再还原成原来的中文字符串。如不转换，则通过URL传递过来的中文字符中会变成乱码，无法还原了。