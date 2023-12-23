`upstream` 配置块是 Nginx 配置中的一个重要组成部分，它用于**定义一组后端服务器，可以将请求负载均衡地分发给这些服务器**。



### 一、定义upstream

---

在http配置模块内创建一个 `upstream` 配置块，并为其指定一个名字。这个名字将在其他配置块中用于引用这个 upstream。

```nginx
http {
  upstream backend_servers {
    # 后端服务器配置
  }
}
```



### 二、添加后端服务器配置

---

在 upstream 配置块内部，可以使用 server 配置项添加后端服务器。每个 `server` 配置项指定一个 IP 地址和端口，以及其他可选参数，如权重、连接数限制等。

```nginx
upstream backend_servers {
  server backend1.example.com:8080;
  server backend2.example.com:8080 weight=2;
  server backend3.example.com:8080 max_fails=3 fail_timeout=30s;
}
```



### 三、负载均衡方法

---

Nginx 支持多种负载均衡方法。默认情况下，Nginx 使用轮询（round-robin）算法。可以通过在 `upstream` 配置块中添加 `least_conn`、`ip_hash` 或 `hash` 配置项来改变负载均衡方法。

- 轮询（round-robin） ：默认方法。请求按顺序分发给每个后端服务器。
- 最少连接（least_conn） ：将请求发送到当前连接数最少的服务器。
- IP 哈希（ip_hash） ：根据客户端 IP 地址进行哈希，将来自同一 IP 的请求发送到相同的后端服务器。这对于需要会话保持的应用场景非常有用。
- 哈希（hash） ：基于指定的 key（例如：URI、请求参数等）进行哈希，将相同 key 的请求发送到相同的后端服务器。

```nginx
upstream backend_servers {
  least_conn;
  # 或者
 	ip_hash;
  # 或者
  hash $request_uri consistent;

  # 后端服务器配置
}
```



### 四、使用upstream

---

在 `location` 配置块中，可以使用 `proxy_pass` 配置项将请求转发到指定的 upstream 。

```nginx
location /api {
  proxy_pass http://backend_servers;
}
```

通过使用 `upstream` 配置块，你可以轻松地在 Nginx 中实现负载均衡和反向代理。这对于提高应用性能、提高可用性和实现弹性扩展等场景非常有用。

`proxy_pass` 是 Nginx 配置中的一个重要指令，它用于将请求转发到其他服务器（例如后端服务器）。在 location 配置块中使用 proxy_pass 可以实现反向代理。下面是一个具体的例子，解释了 proxy_pass 的工作原理和它如何替换 URI。

假设你有一个 Nginx 服务器用作反向代理，它将接收到的请求转发给后端的应用服务器。Nginx 服务器的域名是 proxy.example.com，后端应用服务器的域名是 backend.example.com。

首先，在 Nginx 配置文件的 http 配置块中，创建一个名为 backend 的 upstream 配置块，添加后端服务器的地址：

```nginx
http {
  upstream backend {
    server backend.example.com:8080;
  }

  server {
    # server 配置 ...
  }
}
```

接下来，在server配置块的location配置中，使用proxy_pass指令将请求转发给刚才定义的backend：

```nginx
server {
  listen 80;
  server_name proxy.example.com;

  location /api {
    proxy_pass http://backend;
  }
}
```

现在，当客户端发起请求到 http://proxy.example.com/api/some_endpoint 时，Nginx 会将该请求转发给后端服务器 http://backend.example.com:8080/api/some_endpoint。

注意，proxy_pass 会替换 location 中的路径。在这个例子中，location 匹配了 /api，而 proxy_pass 指向了 http://backend。所以，当请求被转发时，/api 路径保持不变。

但是，如果你想在转发请求时删除或替换某个路径，可以在 proxy_pass 的 URL 中添加一个具体的路径。例如，假设想将请求转发到后端服务器的 /backend 路径，而不是 /api 路径。那么，你可以这样配置 proxy_pass：

```nginx
location /api {
  proxy_pass http://backend/backend;
}
```

现在，当客户端发起请求到 http://proxy.example.com/api/some_endpoint 时，Nginx 会将该请求转发给后端服务器 http://backend.example.com:8080/backend/some_endpoint。可以看到，**原始请求中的 /api 被替换为了 /backend**。

通过 proxy_pass，Nginx 可以轻松地实现反向代理，将客户端的请求转发到后端服务器，同时**可以根据需要替换请求的路径**。

>**try_files**
>
>try_files 是 Nginx 配置中的一个重要指令，它用于按顺序检查指定的文件和路径，如果找到其中一个文件或路径存在，则立即停止搜索并处理请求。try_files 通常用于处理静态文件请求，实现 URL 重写和改进应用程序的路由机制。下面是一个具体的例子，解释了 try_files 的工作原理以及它如何帮助我们替换 URI。
>
>假设你有一个 Web 应用，其中包含静态资源（如 CSS、JS、图像等）和动态内容。Nginx 服务器的域名是 example.com。
>
>首先，在 Nginx 配置文件的 server 配置块中，设置根目录（即 Web 应用的根目录）：
>
>```nginx
>server {
>  listen 80;
>  server_name example.com;
>  root /path/to/your/webapp;
>}
>```
>
>接下来，为了处理静态资源和动态内容，可以在 server 配置块中添加一个 location 配置块，并使用 `try_files` 指令：
>
>```nginx
>location / {
>  try_files $uri $uri/ /index.php?$args;
>}
>```
>
>在这个例子中，try_files 会按顺序尝试以下三个选项：
>
>1. `$uri`：Nginx 会检查请求的 URI 是否对应于静态文件（如 example.com/css/main.css）。如果找到这个文件，Nginx 将返回该文件并停止进一步处理请求。
>2. `$uri/`：如果请求的 URI 对应于一个目录（如 example.com/blog），Nginx 会尝试在该目录中寻找一个默认的索引文件（例如 index.html 或 index.php），并返回找到的文件。
>3. `/index.php?$args`：如果前两个选项都没有找到对应的文件或目录，Nginx 会将请求转发到 /index.php，并将所有请求参数传递给该文件。这通常用于处理动态内容，例如 PHP 应用程序。
>
>`try_files` 指令允许 Nginx 按照指定的顺序搜索文件和路径，从而实现更灵活的请求处理。通过 `try_files`，Nginx 可以轻松地根据请求的 URI 选择适当的静态资源或动态内容进行处理。