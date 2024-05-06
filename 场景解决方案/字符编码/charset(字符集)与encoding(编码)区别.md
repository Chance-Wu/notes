|   |charset|encoding|
|---|---|---|
|   |character set的简写，即字符集|charset encoding的简写，即字符集编码（编码）| 
|举例|Unicode、ASCII、GBK|UTF-8、UTF-16、UTF-32，ASCII，GBK|
 
* encoding依赖于charset
* 一个字符集可以有对个编码实现，就像一个接口可以有多个实现类一样

> 具体例子及规范用法：

一个自于 html 文件，用的是 charset：
`<meta http-equiv="content-type" content="text/html;charset=utf-8">`

另一个来自于 xml 文件，用的是 encoding：
`<?xml version="1.0" encoding="UTF-8"?>`

哪一种用法更规范呢？显然是后者，它更加准确地区分了字符集与编码的概念。

> 注意：“charset=utf-8”容易让人误解为存在一种叫“UTF-8”的字符集，但实际上，无论是 UTF-8，UTF-16 还是 UTF-32 都是对同一种字符集的不同编码实现而已。

> 为什么要严格区分字符集与编码这两个概念？

在早期，字符集与编码是一对一的，但随着时间的发展，出现了一对多的情形。

