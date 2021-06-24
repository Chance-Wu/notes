> char类型，==2字节==，16位，==字面量用单引号括起来==，例如'A'是编码值为65所对应的字符常量。`'A'`与`"A"`不同，后者是包含一个字符A的字符串。



想要了解char类型，就必须了解Unicode编码机制。20世纪80年代开始启动设计Unicode的工作时，人们认为2个字节的代码宽度足以对世界上各种语言的所有字符进行编码，并有足够空间留给未来的扩展。在设计Java时决定采用16位的Unicode字符集。后来Unicode字符超过了65536（2^16）个，主要是增加了大量的汉语、日语和汉语中的表意文字。==Unicode标准已经扩展到包含多达1112064个字符，那些超出原来的16位限制的字符被称作增补字符==。那么Java是怎样做扩展的呢？

> **Unicode基本字符用一个char表示，Unicode增补字符用一对char型表示。**



#### 1. Unicode

---

>编码字符集是一个字符集，它为每个字符分配一个唯一数字。Unicode标准的核心是一个编码字符集，字母"A"的编码为 0041~（16）~。Unicode标准始终使用==十六进制==数字，而且在书写时在前面加上前缀“U+”，所以"A"的编码书写为“U+0041”。
>
>**基本字符**表示从`U+0000`到`U+FFFF`范围之间的字符集，也被称为基本多语言面。（BMP）
>
>**增补字符**表示从U+10000 到 U+10FFFF 范围之间的字符集。



#### 2. 字符编码方案

---

>字符编码方案是从一个或多个编码字符集到一个或多个固定宽度单元序列的映射。最常用的代码单元是字节，但是16位或32位整数也可用于内部处理。UTF-32、UTF-18、UTF-8是Unicode标准的编码字符集的字符编码方案。
>
>![](https://tva1.sinaimg.cn/large/008i3skNgy1grrylnkrchj32420mitap.jpg)

##### 2.1 UTF-8

UTF-8 使用一至四个字节的序列对编码 Unicode 代码点进行编码。

U+0000 至 U+007F 使用一个字节编码

U+0080 至 U+07FF 使用两个字节

U+0800 至 U+FFFF 使用三个字节

U+10000 至 U+10FFFF 使用四个字节

UTF-8 设计原理为：字节值 0x00 至 0x7F 始终表示代码点 U+0000 至 U+007F（Basic Latin 字符子集，它对应 ASCII 字符集）。这些字节值永远不会表示其他代码点，这一特性使 UTF-8 可以很方便地在软件中将特殊的含义赋予某些 ASCII 字符。

![](https://tva1.sinaimg.cn/large/008i3skNgy1grrylnkrchj32420mitap.jpg)

示例：

>拿欧元符号（ € )来举例
>1.它的 Unicode 代码点为 U+20AC
>2.根据上面的表格，欧元符号将被编码为 3 字节
>3.20AC 的二进制表示为 0010 0000 1010 1100 （补足前两个 0 是因为上述表格要求占据 16 位）
>4.将二进制表示按顺序填充到上述表格的 xxxx 位置
>5.得到欧元符号的 UTF-8 编码 1110 0010 1000 0010 1010 1100 ，表示为 16 进制为 E2 82 AC。

##### 2.2 UTF-16

UTF-16 使用一个或两个未分配的 16 位代码单元的序列对 Unicode 代码点进行编码。

值 U+0000 至 U+FFFF 编码为一个相同值的 16 位单元。

增补字符编码为两个代码单元，第一个单元来自于**高代理范围**（U+D800 至 U+DBFF），第二个单元来自于**低代理范围**（U+DC00 至 U+DFFF）。

值 U+D800 至 U+DFFF 保留用于 UTF-16，这些值没有分配给字符作为代码点。这意味着，对于一个字符串中的每个单独的代码单元，软件可以识别是否该代码单元表示某个单单元字符，或者是否该代码单元是某个双单元字符的第一个或第二单元。

![](https://tva1.sinaimg.cn/large/008i3skNgy1grrzvklh57j314006tgmt.jpg)

##### 2.3 UTF-32

UTF-32 将每一个 Unicode 代码点表示为相同值的 32 位整数。很明显，它是内部处理最方便的表达方式，但是，如果作为一般字符串表达方式，则要消耗更多的内存。



#### 3. 示例

---

使用基本类型int在底层API中表示代码点；所有形式的char序列均解释为UTF-16序列。

符号 𐐷 的 UTF-16 编码为 `D801 DC37` ，这时的代码点数量为 1 ，而代码单元长度为 2 ，因此最后输出的就是 2 了。

```java
public class Demo{
  public static void main(String... args) {
    char[] chs = Character.toChars(0x10437);
    System.out.printf("U+10437 高代理字符: %04x%n", (int)chs[0]);
    System.out.printf("U+10437 低代理字符: %04x%n", (int)chs[1]);        
    String str = new String(chs);
    System.out.println("代码单元长度: " + str.length());
    System.out.println("代码点数量: " + str.codePointCount(0, str.length()));
  }
}
```

输出：

```
U+10437 高代理字符: d801
U+10437 低代理字符: dc37
代码单元长度: 2
代码点数量: 1
```

代码点是指可用于编码字符集的数字。

实用API：

- char charAt( int index )：返回给定位置的代码单元。
- int codePointAt( int index )：返回从给定位置开始的码点。
- int offsetByCodePoints( int startIndex, int cpCount)：返回从 startIndex 代码点开始，位移 cpCount 后的码点索引。
- IntStream codePoints( )：将这个字符串的码点作为一个流返回。调用 toArray 将它们放在一个数组中。



#### 4. char初始化方式

---

```java
// 字符，可以是汉字，因为是Unicode编码
char c = 'c';

// 可以用整数赋值，0~65535
char c = 十进制数，八进制数，十六进制数等;

// 用字符的编码值来初始化
char c = '\u数字';
```



#### 5. char运算

---

char在ASCII等字符编码表中有对应的数值，所以可以运算。char + char，char + int——类型均提升为int，附值char变量后，输出字符编码表中对应的字符。

```java
char m = 'a';
char m = 'a' + 'b'; // char类型相加，提升为int
```

