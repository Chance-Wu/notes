curl 是常用的命令行工具，**用来请求 Web 服务器**。它的名字就是客户端（client）的 URL 工具的意思。



### 一、基础

---

[curl](https://curl.haxx.se/)是一种命令行工具，作用是发出网络请求，然后得到和提取数据，显示在"标准输出"（stdout）上面。它支持多种协议。

#### 1.1 查看网页源码

直接在curl命令后加上网址，就可以看到网页源码。

```shell
curl www.sina.com
```

如果要把这个网页保存下来，可以使用 `-o` 参数，这就相当于使用wget命令。

```shell
curl -o [文件名] www.sina.com
```

#### 1.2 自动跳转

有的网址是自动跳转的。使用 `-L` 参数，curl就会跳转到新的网址。

```shell
键入上面的命令，结果就自动跳转为www.sina.com.cn。
```

#### 1.3 显示头信息

`-i` 参数可以**显示http response的头信息**，连同网页代码一起。

```shell
curl -i www.sina.com
```

`-I` 参数只显示http response的头信息。

```shell
curl -I www.sina.com

HTTP/1.1 200 OK
Server: nginx
Date: Thu, 29 Dec 2022 02:25:30 GMT
Content-Type: text/html
Content-Length: 24000
Connection: keep-alive
Vary: Accept-Encoding
ETag: W/"607e94a3-c2c"V=5965C31
X-Powered-By: shci_v1.13
Expires: Thu, 29 Dec 2022 02:26:40 GMT
Cache-Control: max-age=120
X-Via-SSL: ssl.63.sinag1.shx.lb.sinanode.com
Edge-Copy-Time: 1672280686982
Via: http/1.1 cmcc.guangzhou.union.7777 (ApacheTrafficServer/6.2.1 [cRs f ]), http/1.1 cmcc.nanjing.union.235 (ApacheTrafficServer/6.2.1 [cRs f ])
X-Via-Edge: 167228073004742a59c964a0000027fe0a8fe
Age: 44
X-Cache: HIT.235
X-Via-CDN: f=edge,s=cmcc.nanjing.union.68.nb.sinaedge.com,c=2409:8921:2290:70fc:a5df:7855:969c:2d9d;f=Edge,s=cmcc.nanjing.union.235,c=221.181.204.68
```

#### 1.4 显示通信过程

---

`-v` 参数可以显示一次http通信的整个过程，包括端口连接和http request头信息。

```shell
curl -v www.sina.com
```

`--trace` 参数可以查看更详细的过程。

```shell
curl --trace output.txt www.sina.com
或者
curl --trace-ascii output.txt www.sina.com
```

#### 1.5 发送表单信息

发送表单信息有GET和POST两种方法。GET方法相对简单，只要把数据附在网址后面就行。

```shell
curl example.com/form.cgi?data=xxx
```

POST方法必须把数据和网址分开，用到 `--data` 参数。

```shell
url -X POST --data "data=xxx" example.com/form.cgi
```

如果你的数据没有经过表单编码，还可以让curl为你编码，参数是`--data-urlencode`。

```shell
curl -X POST--data-urlencode "date=April 1" example.com/form.cgi
```

#### 1.6 HTTP动词

**curl默认的HTTP动词是GET**，使用 `-X` 参数可以支持其他动词。

```shell
curl -X POST www.example.com
```

```shell
curl -X DELETE www.example.com
```

#### 1.7 文件上传

假定文件上传的表单是下面这样：

```html
<form method="POST" enctype='multipart/form-data' action="upload.cgi">
  <input type=file name=upload>
  <input type=submit name=press value="OK">
</form>
```

可以使用`curl --form`上传文件：

```shell
curl --form upload=@localfilename --form press=OK [URL]
```

#### 1.8 Referer字段

有时你需要在http request头信息中，使用`--referer`参数提供一个referer字段，表示你是从哪里跳转过来的。

```shell
curl --referer http://www.example.com http://www.example.com
```

#### 1.9 User Agent字段

这个字段是用来表示客户端的设备信息。服务器有时会根据这个字段，针对不同设备，返回不同格式的网页，比如手机版和桌面版。

Mac的User Agent是

```http
user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/108.0.0.0 Safari/537.36
```

可以使用`--user-agent`参数模拟：

```shell
curl --user-agent "[User Agent]" [URL]
```

#### 1.10 cookie

使用`--cookie`参数，可以发送cookie。

```shell
curl --cookie "name=xxx" www.example.com
```

至于具体的cookie的值，可以从http response头信息的`Set-Cookie`字段中得到。

`-c cookie-file`可以保存服务器返回的cookie到文件，`-b cookie-file`可以使用这个文件作为cookie信息，进行后续的请求。

```shell
curl -c cookies http://example.com
curl -b cookies http://example.com
```

#### 1.11 增加头信息

有时需要在http request之中，自行增加一个头信息。`--header`参数就可以起到这个作用。

```shell
curl --header "Content-Type:application/json" http://example.com
```

#### 1.12 HTTP认证

有些网域需要HTTP认证，这时curl需要用到`--user`参数。

```shell
curl --user name:password example.com
```



### 二、curl常用参数

---

不带有任何参数时，curl 就是发出 GET 请求。

```bash
curl https://www.example.com
```

#### 2.1 -A

`-A`参数指定客户端的用户代理标头，即User-Agent。curl 的默认用户代理字符串是curl/[version]。

```bash
curl -A 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/76.0.3809.100 Safari/537.36' https://google.com
```

上面命令将User-Agent改成 Chrome 浏览器。

```bash
curl -A '' https://google.com
```

上面命令会移除`User-Agent`标头。

也可以通过`-H`参数直接指定标头，更改`User-Agent`。

```bash
curl -H 'User-Agent: php/1.0' https://google.com
```

#### 2.2 -b

`-b`参数用来向服务器发送 Cookie。

```bash
curl -b 'foo=bar' https://google.com
```

上面命令会生成一个标头`Cookie: foo=bar`，向服务器发送一个名为`foo`、值为`bar`的 Cookie。

```bash
curl -b 'foo1=bar;foo2=bar2' https://google.com
```

上面命令发送两个 Cookie。

```bash
curl -b cookies.txt https://www.google.com
```

上面命令读取本地文件`cookies.txt`，里面是服务器设置的 Cookie（参见`-c`参数），将其发送到服务器。

#### 2.3 -d

`-d`参数用于发送 POST 请求的数据体。

```bash
curl -d'login=emma＆password=123'-X POST https://google.com/login
# 或者
curl -d 'login=emma' -d 'password=123' -X POST  https://google.com/login
```

- 使用`-d`参数以后，HTTP 请求会自动加上标头Content-Type : application/x-www-form-urlencoded。并且会自动将请求转为 POST 方法，因此可以省略`-X POST`。
- `-d`参数可以读取本地文本文件的数据，向服务器发送。

#### 2.4 --date-urlencode

`--date-urlencode`参数等同于`-d`，发送 POST 请求的数据体，区别在于会自动将发送的数据进行 URL 编码。

```bash
curl --data-urlencode 'comment=hello world' https://google.com/login
```

上面代码中，发送的数据hello world之间有一个空格，需要进行 URL 编码。

#### 2.5 -e

`-e`参数用来设置 HTTP 的标头`Referer`，表示请求的来源。

```bash
curl -e 'https://google.com?q=example' https://www.example.com
```

上面命令将`Referer`标头设为`https://google.com?q=example`。

`-H`参数可以通过直接添加标头`Referer`，达到同样效果。

```bash
curl -H 'Referer: https://google.com?q=example' https://www.example.com
```

#### 2.6 -F

`-F`参数用来向服务器上传二进制文件。

```bash
curl -F 'file=@photo.png' https://google.com/profile
```

上面命令会给 HTTP 请求加上标头`Content-Type: multipart/form-data`，然后将文件`photo.png`作为`file`字段上传。

`-F`参数可以指定 MIME 类型。

```bash
curl -F 'file=@photo.png;type=image/png' https://google.com/profile
```

上面命令指定 MIME 类型为`image/png`，否则 curl 会把 MIME 类型设为`application/octet-stream`。

`-F`参数也可以指定文件名。

```bash
curl -F 'file=@photo.png;filename=me.png' https://google.com/profile
```

上面命令中，原始文件名为`photo.png`，但是服务器接收到的文件名为`me.png`。

#### 2.7 -G

`-G`参数用来构造 URL 的查询字符串。

```bash
curl -G -d 'q=kitties' -d 'count=20' https://google.com/search
```

上面命令会发出一个 GET 请求，实际请求的 URL 为`https://google.com/search?q=kitties&count=20`。如果省略`--G`，会发出一个 POST 请求。

如果数据需要 URL 编码，可以结合`--data--urlencode`参数。

```bash
curl -G --data-urlencode 'comment=hello world' https://www.example.com
```

#### 2.8 -H

`-H`参数添加 HTTP 请求的标头。

```bash
curl -H 'Accept-Language: en-US' https://google.com
```

上面命令添加 HTTP 标头`Accept-Language: en-US`。

```bash
curl -H 'Accept-Language: en-US' -H 'Secret-Message: xyzzy' https://google.com
```

上面命令添加两个 HTTP 标头。

```bash
curl -d '{"login": "emma", "pass": "123"}' -H 'Content-Type: application/json' https://google.com/login
```

上面命令添加 HTTP 请求的标头是`Content-Type: application/json`，然后用`-d`参数发送 JSON 数据。

#### 2.9 -i

`-i`参数打印出服务器回应的 HTTP 标头。

```bash
curl -i https://www.example.com
```

上面命令收到服务器回应后，先输出服务器回应的标头，然后空一行，再输出网页的源码。

#### 2.10 -I

`-I`参数向服务器发出 HEAD 请求，然会将服务器返回的 HTTP 标头打印出来。

 ```bash
 curl -I https://www.example.com
 ```

上面命令输出服务器对 HEAD 请求的回应。

`--head`参数等同于`-I`。

 ```bash
 curl --head https://www.example.com
 ```

#### 2.11 -k

`-k`参数指定跳过 SSL 检测。

 ```bash
 $ curl -k https://www.example.com
 ```

上面命令不会检查服务器的 SSL 证书是否正确。

#### 2.12 -L

`-L`参数会让 HTTP 请求跟随服务器的重定向。curl 默认不跟随重定向。

```bash
curl -L -d 'tweet=hi' https://api.twitter.com/tweet
```

#### 2.13 --limit-rate

`--limit-rate`用来限制 HTTP 请求和回应的带宽，模拟慢网速的环境。

 ```bash
 curl --limit-rate 200k https://google.com
 ```

上面命令将带宽限制在每秒 200K 字节。

#### 2.14 -o

`-o`参数将服务器的回应保存成文件，等同于`wget`命令。

 ```bash
 curl -o example.html https://www.example.com
 ```

上面命令将`www.example.com`保存成`example.html`。

#### 2.15 -O

`-O`参数将服务器回应保存成文件，并将 URL 的最后部分当作文件名。

```bash
curl -O https://www.example.com/foo/bar.html
```

上面命令将服务器回应保存成文件，文件名为`bar.html`。

#### 2.16 -s

`-s`参数将不输出错误和进度信息。

 ```bash
 curl -s https://www.example.com
 ```

上面命令一旦发生错误，不会显示错误信息。不发生错误的话，会正常显示运行结果。

如果想让 curl 不产生任何输出，可以使用下面的命令。

 ```bash
 curl -s -o /dev/null https://google.com
 ```

#### 2.17 -S

`-S`参数指定只输出错误信息，通常与`-s`一起使用。

```bash
curl -s -o /dev/null https://google.com
```

上面命令没有任何输出，除非发生错误。

#### 2.18 -u

`-u`参数用来设置服务器认证的用户名和密码。

```bash
curl -u 'bob:12345' https://google.com/login
```

上面命令设置用户名为`bob`，密码为`12345`，然后将其转为 HTTP 标头`Authorization: Basic Ym9iOjEyMzQ1`。

curl 能够识别 URL 里面的用户名和密码。

```bash
curl https://bob:12345@google.com/login
```

上面命令能够识别 URL 里面的用户名和密码，将其转为上个例子里面的 HTTP 标头。

```bash
curl -u 'bob' https://google.com/login
```

上面命令只设置了用户名，执行后，curl 会提示用户输入密码。

#### 2.19 -v

`-v`参数输出通信的整个过程，用于调试。

```bash
curl -v https://www.example.com
```

`--trace`参数也可以用于调试，还会输出原始的二进制数据。

```bash
curl --trace - https://www.example.com
```

#### 2.20 -x

`-x`参数指定 HTTP 请求的代理。

```bash
curl -x socks5://james:cats@myproxy.com:8080 https://www.example.com
```

上面命令指定 HTTP 请求通过`myproxy.com:8080`的 socks5 代理发出。

如果没有指定代理协议，默认为 HTTP。

```bash
curl -x james:cats@myproxy.com:8080 https://www.example.com
```

上面命令中，请求的代理使用 HTTP 协议。

#### 2.21 -X

`-X`参数指定 HTTP 请求的方法。

```bash
curl -X POST https://www.example.com
```

上面命令对`https://www.example.com`发出 POST 请求。
