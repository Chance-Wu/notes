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



















