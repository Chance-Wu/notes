### 一、非法字符问题

---

```
Illegal character in query at index 158: https://hope.demogic.com/open-api/saveOrUpdateStoreGoods.json?token=7417fd2eee04405fbb4277a8e79853b6&factoryCode=ADC&entMicroSignal=gh_efb70add9010&goodsInfo={"brandCode":"KA","brandName":"奥德臣","proName":"成长椅（经典系列）, 22灰色, F","proNo":"100125","proPrice":1890.0,"categoriesTree":[{"parentCatCode":"-1","catName":"G-女长袖梭织衣","catCode":"GA2NSA1"}],"attributes":[{"propName":"年份","propCode":"YEAR","valName":"2014","valCode":"2014"},{"propName":"季节","propCode":"SEASON","valName":"春季","valCode":"2014"},{"propName":"单位","propCode":"PACKET","valName":"件","valCode":"one"}],"goodsSku":[{"skuCode":"7040351001250","skuAttributes":[{"attrName":"颜色","attrCode":"COLOR","valName":"22灰色"},{"attrName":"尺码","attrCode":"SIZE","valName":"F"}],"skuPrice":1890.0},{"skuCode":"7040351001250","skuAttributes":[{"attrName":"颜色","attrCode":"COLOR","valName":"22灰色"},{"attrName":"尺码","attrCode":"SIZE","valName":"F"}],"skuPrice":1890.0}]}
```

原因：http请求时，没有对json参数进行转码

解决办法：进行转码

```java
// 对json字符串jsonStr进行转码
String encode = URLEncoder.encode(jsonStr, "UTF-8");
```



### 二、为何要进行URL编码

---

参考：http://www.javashuo.com/article/p-kvvdmftq-bt.html

Http协议中参数的传输是"key=value"这种键值对形式的，若是要传多个参数就须要用“&”符号对键值对进行分割。如"?name1=value1&name2=value2"，这样在服务端在收到这种字符串的时候，会用“&”分割出每个参数，而后再用“=”来分割出参数值。

针对“name1=value1&name2=value2”来讲一下客户端到服务端的概念上解析过程：

上述字符串在计算机中用**ASCII码**表示为：
 6E616D6531 3D 76616C756531 26 6E616D6532 3D 76616C756532

```
6E616D6531：name1 
3D：= 
76616C756531：value1 
26：&
6E616D6532：name2 
3D：= 
76616C756532：value2
```

服务端在接收到该数据后就能够遍历该字节流，首先一个字节一个字节的吃，当吃到3D这字节后，服务端就知道前面吃得字节表示一个key，再想后吃，若是遇到26，说明从刚才吃的3D到26字节之间的是上一个key的value，以此类推就能够解析出客户端传过来的参数。

现有这样一个问题，若是有个参数值中就包含=或&这种特殊字符的时候该怎么办。好比说“name1=value1”，其中value1的值是“va&lu=e1”字符串，那么实际在传输过程当中就会变成这样“name1=va&lu=e1”。咱们的本意是就只有一个键值对，可是服务端会解析成两个键值对。

> 如何解决上述问题带来的歧义呢？解决的办法就是对参数进行URL编码
>
> **URL编码只是简单的在特殊字符的各个字节前加上%**，例如，咱们对上述会产生奇异的字符进行URL编码后结果：“name1=va%26lu%3D”，这样**服务端会把紧跟在“%”后的字节当成普通的字节**，就是不会把它当成各个参数或键值对的分隔符。

又如，Url的编码格式采用的是ASCII码，而不是Unicode，这也就是说你不能在Url中包含任何非ASCII字符，例如中文。不然若是客户端浏览器和服务端浏览器支持的字符集不一样的状况下，中文可能会形成安全问题。Url编码的原则就是使用安全的字符（没有特殊用途或者特殊意义的可打印字符）去表示那些不安全的字符。

预备知识：URI是统一资源标识的意思，一般咱们所说的URL只是URI的一种。典型URL的格式以下所示。下面提到的URL编码，实际上应该指的是URI编码。

```
foo://example.com:8042/over/there?name=ferret#nose
   \_/ \______________/ \________/\_________/ \__/
    |         |              |         |        |
  scheme     authority     path      query   fragment
```

#### 2.1 哪些字符须要编码

RFC3986文档规定，Url中只容许包含英文字母（a-zA-Z）、数字（0-9）、-_.~4个特殊字符以及全部保留字符。RFC3986文档对Url的编解码问题作出了详细的建议，指出了哪些字符须要被编码才不会引发Url语义的转变，以及对为何这些字符须要编码作出了相应的解释。

US-ASCII字符集中没有对应的可打印字符：Url中只容许使用可打印字符。US-ASCII码中的10-7F字节全都表示控制字符，这些字符都不能直接出如今Url中。同时，对于80-FF字节（ISO-8859-1），因为已经超出了US-ACII定义的字节范围，所以也不能够放在Url中。

保留字符：Url能够划分红若干个组件，协议、主机、路径等。有一些字符（:/?#[]@）是用做分隔不一样组件的。例如：冒号用于分隔协议和主机，/用于分隔主机和路径，?用于分隔路径和查询参数，等等。还有一些字符（!$&'()*+,;=）用于在每一个组件中起到分隔做用的，如=用于表示查询参数中的键值对，&符号用于分隔查询多个键值对。当组件中的普通数据包含这些特殊字符时，须要对其进行编码。

RFC3986中指定了如下字符为保留字符：! * ' ( ) ; : @ & = + $ , / ? # [ ]

不安全字符：还有一些字符，当他们直接放在Url中的时候，可能会引发解析程序的歧义。这些字符被视为不安全字符，缘由有不少。

- 空格：Url在传输的过程，或者用户在排版的过程，或者文本处理程序在处理Url的过程，都有可能引入可有可无的空格，或者将那些有意义的空格给去掉。
- 引号以及<>：引号和尖括号一般用于在普通文本中起到分隔Url的做用
- \#：一般用于表示书签或者锚点
- %：百分号自己用做对不安全字符进行编码时使用的特殊字符，所以自己须要编码
- {}|\^[]`~：某一些网关或者传输代理会篡改这些字符

须要注意的是，对于Url中的合法字符，编码和不编码是等价的，可是对于上面提到的这些字符，若是不通过编码，那么它们有可能会形成Url语义的不一样。所以对于Url而言，只有普通英文字符和数字，特殊字符$-_.+!*'()还有保留字符，才能出如今未经编码的Url之中。其余字符均须要通过编码以后才能出如今Url中。

可是因为历史缘由，目前尚存在一些不标准的编码实现。例如对于`~`符号，虽然RFC3986文档规定，对于波浪符号`~`，不须要进行Url编码，可是仍是有不少老的网关或者传输代理会进行编码。

#### 2.2 如何对Url中的非法字符进行编码

Url编码一般也被称为百分号编码（Url Encoding，also known as percent-encoding），是由于它的编码方式很是简单，使用%百分号加上两位的字符——0123456789ABCDEF——表明一个字节的十六进制形式。Url编码默认使用的字符集是US-ASCII。例如a在US-ASCII码中对应的字节是0x61，那么Url编码以后获得的就是%61，咱们在地址栏上输入http://g.cn/search?q=%61%62%63，实际上就等同于在google上搜索abc了。又如@符号在ASCII字符集中对应的字节为0x40，通过Url编码以后获得的是%40。

对于非ASCII字符，须要使用ASCII字符集的超集进行编码获得相应的字节，而后对每一个字节执行百分号编码。对于Unicode字符，RFC文档建议使用utf-8对其进行编码获得相应的字节，而后对每一个字节执行百分号编码。如"中文"使用UTF-8字符集获得的字节为0xE4 0xB8 0xAD 0xE6 0x96 0x87，通过Url编码以后获得"%E4%B8%AD%E6%96%87"。

若是某个字节对应着ASCII字符集中的某个非保留字符，则此字节无需使用百分号表示。例如"Url编码"，使用UTF-8编码获得的字节是0x55 0x72 0x6C 0xE7 0xBC 0x96 0xE7 0xA0 0x81，因为前三个字节对应着ASCII中的非保留字符"Url"，所以这三个字节能够用非保留字符"Url"表示。最终的Url编码能够简化成"Url%E7%BC%96%E7%A0%81" ，固然，若是你用"%55%72%6C%E7%BC%96%E7%A0%81"也是能够的。

下面列出了这三个函数的安全字符（即函数不会对这些字符进行编码）

- escape（69个）：*/@+-._0-9a-zA-Z
- encodeURI（82个）：!#$&'()*+,/:;=?@-._~0-9a-zA-Z
- encodeURIComponent（71个）：!'()*-._~0-9a-zA-Z

兼容性不一样：escape函数是从Javascript 1.0的时候就存在了，其余两个函数是在Javascript 1.5才引入的。可是因为Javascript 1.5已经很是普及了，因此实际上使用encodeURI和encodeURIComponent并不会有什么兼容性问题。

对Unicode字符的编码方式不一样：这三个函数对于ASCII字符的编码方式相同，均是使用百分号+两位十六进制字符来表示。可是对于Unicode字符，escape的编码方式是%uxxxx，其中的xxxx是用来表示unicode字符的4位十六进制字符。这种方式已经被W3C废弃了。可是在ECMA-262标准中仍然保留着escape的这种编码语法。encodeURI和encodeURIComponent则使用UTF-8对非ASCII字符进行编码，而后再进行百分号编码。这是RFC推荐的。所以建议尽量的使用这两个函数替代escape进行编码。

适用场合不一样：encodeURI被用做对一个完整的URI进行编码，而encodeURIComponent被用做对URI的一个组件进行编码。从上面提到的安全字符范围表格来看，咱们会发现，encodeURIComponent编码的字符范围要比encodeURI的大。咱们上面提到过，保留字符通常是用来分隔URI组件（一个URI能够被切割成多个组件，参考预备知识一节）或者子组件（如URI中查询参数的分隔符），如：号用于分隔scheme和主机，?号用于分隔主机和路径。因为encodeURI操纵的对象是一个完整的的URI，这些字符在URI中原本就有特殊用途，所以这些保留字符不会被encodeURI编码，不然意义就变了。

组件内部有本身的数据表示格式，可是这些数据内部不能包含有分隔组件的保留字符，不然就会致使整个URI中组件的分隔混乱。所以对于单个组件使用encodeURIComponent，须要编码的字符就更多了。

#### 2.3 表单提交

当Html的表单被提交时，每一个表单域都会被Url编码以后才在被发送。因为历史的缘由，表单使用的Url编码实现并不符合最新的标准。例如对于空格使用的编码并非%20，而是+号，若是表单使用的是Post方法提交的，咱们能够在HTTP头中看到有一个Content-Type的header，值为application/x-www-form-urlencoded。大部分应用程序均能处理这种非标准实现的Url编码，可是在客户端Javascript中，并无一个函数可以将+号解码成空格，只能本身写转换函数。还有，对于非ASCII字符，使用的编码字符集取决于当前文档使用的字符集。例如咱们在Html头部加上

```html
<meta http-equiv="Content-Type" content="text/html; charset=gb2312" />
```

这样浏览器就会使用gb2312去渲染此文档（注意，当HTML文档中没有设置此meta标签，则浏览器会根据当前用户喜爱去自动选择字符集，用户也能够强制当前网站使用某个指定的字符集）。当提交表单时，Url编码使用的字符集就是gb2312。

以前在使用Aptana（为何专指aptana下面会提到）遇到一个很迷惑的问题，就是在使用encodeURI的时候，发现它编码获得的结果和我想的很不同。下面是个人示例代码：

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" 
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=gb2312" />
  </head>
  <body>
    <script type="text/javascript">
      document.write(encodeURI("中文"));
    </script>
  </body>
</html>
```

运行结果输出%E6%B6%93%EE%85%9F%E6%9E%83。显然这并非使用UTF-8字符集进行Url编码获得的结果（在Google上搜索"中文"，Url中显示的是%E4%B8%AD%E6%96%87）。

因此我当时就很质疑，难道encodeURI还跟页面编码有关，可是我发现，正常状况下，若是你使用gb2312进行Url编码也不会获得这个结果的才是。后来终于被我发现，原来是页面文件存储使用的字符集和Meta标签中指定的字符集不一致致使的问题。Aptana的编辑器默认状况下使用UTF-8字符集。也就是说这个文件实际存储的时候使用的是UTF-8字符集。可是因为Meta标签中指定了gb2312，这个时候，浏览器就会按照gb2312去解析这个文档，那么天然在"中文"这个字符串这里就会出错，由于"中文"字符串用UTF-8编码事后获得的字节是0xE4 0xB8 0xAD 0xE6 0x96 0x87，这6个字节又被浏览器拿gb2312去解码，那么就会获得另外三个汉字"涓枃"（GBK中一个汉字占两个字节），这三个汉字在传入encodeURI函数以后获得的结果就是%E6%B6%93%EE%85%9F%E6%9E%83。所以，encodeURI使用的仍是UTF-8，并不会受到页面字符集的影响。

对于包含中文的Url的处理问题，不一样浏览器有不一样的表现。例如对于IE，若是你勾选了高级设置"老是以UTF-8发送Url"，那么Url中的路径部分的中文会使用UTF-8进行Url编码以后发送给服务端，而查询参数中的中文部分使用系统默认字符集进行Url编码。为了保证最大互操做性，建议全部放到Url中的组件所有显式指定某个字符集进行Url编码，而不依赖于浏览器的默认实现。

另外，不少HTTP监视工具或者浏览器地址栏等在显示Url的时候会自动将Url进行一次解码（使用UTF-8字符集），这就是为何当你在Firefox中访问Google搜索中文的时候，地址栏显示的Url包含中文的缘故。但实际上发送给服务端的原始Url仍是通过编码的。你能够在地址栏上使用Javascript访问location.href就能够看出来了。在研究Url编解码的时候千万别被这些假象给迷惑了。





























