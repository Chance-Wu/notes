### 一、499状态

---

nginx 源码中对499状态码的定义如下：

```
/*
 * HTTP does not define the code for the case when a client closed
 * the connection while we are processing its request so we introduce
 * own code to log such situation when a client has closed the connection
 * before we even try to send the HTTP header to it
 */
 
#define NGX_HTTP_CLIENT_CLOSED_REQUEST 499
```

- 499 状态码不是 HTTP 的标准代码。

- 499 状态码是 Nginx 自己定义，用来记录服务端向客户端发送 HTTP 请求头之前，客户端已经关闭连接的一种情况。
- 最常见的场景就是 timeout 设置不合理，**Nginx 把请求转发上游服务器，上游服务器慢吞吞的处理，客户端等不及了主动断开链接**，Nginx 就负责记录了 499。



### 二、什么情况Nginx记录499错误日志

---

我们使用curl模拟请求以下

```bash
for i in $(seq 1 10); do curl -m 2 "http://api.example.test"; done
```

curl -m 2 "http://api.example.test"  2秒没有响应则断开

```
tail -f /var/log/nginx/apiexample.access.log
 
172.19.0.1 - - [15/Nov/2019:06:32:19 +0000] "GET / HTTP/1.1" 499 0 "-" "curl/7.67.0"
172.19.0.1 - - [15/Nov/2019:06:32:22 +0000] "GET / HTTP/1.1" 499 0 "-" "curl/7.67.0"
172.19.0.1 - - [15/Nov/2019:06:32:24 +0000] "GET / HTTP/1.1" 499 0 "-" "curl/7.67.0"
172.19.0.1 - - [15/Nov/2019:06:32:26 +0000] "GET / HTTP/1.1" 499 0 "-" "curl/7.67.0"
172.19.0.1 - - [15/Nov/2019:06:32:28 +0000] "GET / HTTP/1.1" 499 0 "-" "curl/7.67.0"
172.19.0.1 - - [15/Nov/2019:06:32:30 +0000] "GET / HTTP/1.1" 499 0 "-" "curl/7.67.0"
172.19.0.1 - - [15/Nov/2019:06:32:32 +0000] "GET / HTTP/1.1" 499 0 "-" "curl/7.67.0"
172.19.0.1 - - [15/Nov/2019:06:32:34 +0000] "GET / HTTP/1.1" 499 0 "-" "curl/7.67.0"
172.19.0.1 - - [15/Nov/2019:06:32:36 +0000] "GET / HTTP/1.1" 499 0 "-" "curl/7.67.0"
172.19.0.1 - - [15/Nov/2019:06:32:38 +0000] "GET / HTTP/1.1" 499 0 "-" "curl/7.67.0"
```

记录 499 的情形：

- 如上所示，数据传输的最大允许时间超时的话，Curl 断开了请求，而 Web 服务器如 Nginx 还在处理的话，则 Nginx 会记录 499。
- 如果 Nginx 作为反向代理时，Nginx 将请求分发至对应的处理服务器时，有两对超时参数的设置：`proxy_send_timeout` 和 `proxy_read_timeout`、`fastcgi_send_timeout` 和`fastcgi_read_timeout`。两对参数默认的超时时间都是 60s。在 Nginx 出现 499 的情况下，可以结合请求断开前的耗时和这两对设定的时间进行对比，看一下是不是在 proxy_pass 或者 fastcgi_pass 处理时，设置的超时时间短了。
- 如果两次提交 POST 过快就会出现 499 的情况，Nginx 认为是不安全的连接，主动拒绝了客户端的连接。
- 相关负载均衡配置等。

>**proxy_send_timeout（默认值 60s）**
>
>这个指定设置了发送请求给upstream服务器的超时时间。超时设置不是为了整个发送期间，而是在两次write操作期间。如果超时后，upstream没有收到 新的数据，Nginx会关闭连接。
>
>**proxy_read_timeout（默认值 60s）**
>
>该指令设置与代理服务器的读超时时间。它决定了Nginx会等待多长时间来获得请求的响应。这个时间不是获得整个response的时间，而是两次reading 操作的时间。（？？什么是两次reading操作的时间）

>**fastcgi_send_timeout（默认60s）**
>
>指定nginx向后端传送请求超时时间(指已完成两次握手后向fastcgi传送请求超时时间)
>
>**fastcgi_read_timeout（默认60s）**
>
>指定nginx接受后端fastcgi响应请求超时时间 (指已完成两次握手后nginx接受fastcgi响应请求超时时间)



### 三、如何有效防止Nginx记录499错误

---

HTTP 请求在指定的时间内没能拿到响应而关闭了连接，就会发生 Nginx 记录 499 错误的情况。这个涉及到两个重要的问题：**时间问题** 和 **性能问题**（性能问题太过宽泛就不提及了），所以解决这个问题也就从这两方面入手。

当然还有配置 proxy_ignore_client_abort 参数为 on 来解决的（让代理服务端不要主动关闭客户端的连接）。但是这样也有一定的风险，会拖垮服务器。发生这个错误，如果服务器 CPU 和 Memory 不算太高，一般是数据库和程序的问题，数据库处理较慢或者程序线程较低。结合情况调整，比如读写分离或者程序线程数调高。























































