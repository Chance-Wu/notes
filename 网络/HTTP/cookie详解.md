### Cookie是什么

---

>cookie的中文翻译是曲奇，小甜饼的意思。cookie其实就是一些数据信息，类型为“小型文本文件”，存储于电脑上的文本文件中。



### Cookie有什么用

---

>例如，当打开一个网站时，如果这个网站我们曾经登陆过，那么当我们再次打开网站时，发现就不需要再次登录了，而是直接进入首页。
>
>其实就是浏览器保存了我们的cookie，里面记录了一些信息，当然，这些cookie是服务器创建后返回给浏览器的。浏览器只进行了保存。下面展示bilibili网站保存的cookie。



### Cookie的表示

---

一般情况下，cookie是以键值对进行表示的（key-value），例如name=chance，这个就表示cookie的名字是name，cookie携带的值是chance。



### Cookie的组成

---

cookie中常用属性的解释。

- Name：cookie的名字
- Value：cookie的值
- Path：定义了Web站点上可以访问该Cookie的目录
- Expires：这个值表示cookie的过期时间，cookie在这个值之前都有效。
- Size：表示cookie的大小



### Cookie的HTTP传输

---

#### HTTP请求

在发送HTTP请求时，发现浏览器将我们的cookie都进行了携带（**浏览器只会携带在当前请求的url中包含了该cookie中path值的cookie**），并且是以key：value的形式进行表示的。多个cookie用；进行隔开。

#### HTTP响应

在HTTP响应中， cookie的表示形式是，Set-Cookie：cookie的名字，cookie的值。如果有多个cookie，那么在HTTP响应中就使用多个Set-Cookie进行表示。



### Cookie的生命周期

---

cookie有2种存储方式，一种是会话性，一种是持久性。

会话性：如果cookie为会话性，那么cookie仅会保存在客户端的内存中，当我们关闭客户端时cookie也就失效了。

持久性：如果cookie为持久性，那么cookie会保存在用户的硬盘中，直至生存期结束或者用户主动将其销毁。

>cookie是可以进行设置的，可以人为设置cookie的有效时间，什么时候创建，什么时候销毁。



### Cookie使用的常见方法

---

java中Cookie对象的方法进行讲解

`new Cookie(String name, String value)`：创建一个Cookie对象，必须传入cookie的名字和cookie的值。

getValue()：得到cookie保存的值。

getName()：获取cookie的名字。

setMaxAge(int expiry)：设置cookie的有效期，默认为-1。这个如果设置负数，表示客服端关闭，cookie就会删除。0表示马上删除。正数表示有效时间，单位是秒。

setPath(String uri)：设置cookie的作用域。



### Cookie安全性

---

通常 cookie 信息都是使用HTTP连接传递数据，这种传递方式很容易被查看，在控制台下运行document.cookie,一目了然；所以 cookie 存储的信息容易被窃取。假如 cookie 中所传递的内容比较重要，那么就要求使用加密的数据传输。所以 cookie 的这个属性的名称是“secure”，默认的值为空。如果一个 cookie 的属性为secure，那么它与服务器之间就通过HTTPS或者其它安全协议传递数据。语法如下： 

> document.cookie = “username=Darren;secure” 

把cookie设置为secure，只保证 cookie 与服务器之间的数据传输过程加密，而保存在本地的 cookie文件并不加密。如果想让本地cookie也加密，得自己加密数据。

注： 就算设置了secure 属性也并不代表他人不能看到你机器本地保存的 cookie 信息，所以说到底，别把重要信息放cookie就对了。