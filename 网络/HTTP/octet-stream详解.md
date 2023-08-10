application/octet-stream

当浏览器在请求资源时，会通过http返回头中的content-type决定如何显示/处理将要加载的数据，如果这个类型浏览器能够支持阅览，浏览器就会直接展示该资源，比如png、jpeg、video等格式。

在某些下载文件的场景中，服务端可能会返回文件流，并在返回头中带上`Content-Type:application/octet-stream`，**告知浏览器这是一个字节流，浏览器处理字节流的默认方式就是下载**。

Application/octet-stream是应用程序文件的默认值。意思是未知的应用程序文件，浏览器一般不会自动执行或询问执行。浏览器会像对待，设置了HTTP头Content-Disposition值为attachment的文件一样来对待这类文件，即浏览器会触发下载行为。

浏览器并不认得这是什么类型，也不知道应该如何展示，只知道这是一种二进制文件，因此遇到content-type为application/octet-stream的文件时，浏览器会直接把它下载下来。这个类型一般会配合另一个响应头Content-Disposition，该响应头指示回复的内容该以何种形式展示，是以内联的形式（即网页或者网页的一部分），还是以附件的形式下载并保存到本地。



>在电脑领域里，一个**octet**是指八个比特（bit）为一组的单位，中文称作字节。
>
>在法国和罗马尼亚， *octet* 这个字通常是指一个字节（byte）的意思；当我们称一兆字节（megabyte，MB），在这些地区会称作 *megaoctet*。 *bit* 和 *byte* 在法语里是异义同音字。
>
>*Octet* 除了下面提到的唯一例外之外，都是指一个具有八个比特的实体。因此在电脑网络标准中，在*byte*容易引起混淆的地方都仅使用*Octet*。
>
>**octet**-**stream指任意类型的二进制流数据。**
>
>Octets 有两种不同的前缀，一种为 2 的多次方，另一种是国际单位制（SI，International System of Units）。2 的多次方的格式为国际电工委员会（International Electrotechnical Commission）在 1998 年所制定。
>
>**octet** 这个词是从拉丁文和希腊文的数字 octo 派生而来的，意指八。











































