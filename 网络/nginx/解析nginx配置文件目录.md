### 一、配置文件目录结构

---

nginx的配置文件目录默认在/etc/nginx下，但是在编译安装的情况下，也可以指定其他的配置文件目录。以下是nginx默认配置文件目录结构：

```
/etc/nginx/
├── conf.d/
│   └── default.conf
├── fastcgi_params
├── mime.types
├── nginx.conf
├── scgi_params
├── uwsgi_params
└── win-utf
```

其中，nginx.conf是nginx配置文件的主文件，其他文件是一些额外的配置文件。conf.d目录则是存放用户自定义的配置文件。



### 二、nginx.conf主文件配置

---

nginx.conf是nginx配置文件的主文件，负责整个nginx的全局配置。以下是一个nginx.conf的模板：

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

events {
  worker_connections 1024;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;
  log_format main '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';

  access_log /var/log/nginx/access.log  main;

  sendfile on;

  keepalive_timeout 65;

  server {
    listen 80;
    server_name localhost;

    location / {
      root   /usr/share/nginx/html;
      index  index.html index.htm;
    }
  }
}
```

其中，user指定nginx的运行用户，worker_processes指定worker进程的数量，error_log指定nginx的错误日志文件，pid指定nginx的进程id文件。

events块中设置与事件相关的参数，如worker_connections等。

http块中设置http协议相关的参数，如log_format、access_log等。

server块中设置针对具体服务的参数，如listen、server_name、location等。



### 三、conf.d目录配置

---

在conf.d目录中，可以存放多个nginx配置文件。每个文件对应一个具体的服务配置。

#### 3.1 virtual host配置

virtual host是nginx中的一个重要配置技术，可以在一台物理服务器中，部署多个虚拟主机。以下是一个虚拟主机配置文件的示例：

```nginx
server {
  listen       80;
  server_name  example.com;

  location / {
    root   /var/www/html/example;
    index  index.html index.htm;
  }
}
```

其中，listen指定了监听的端口，server_name指定了该虚拟主机的域名，location指定了该虚拟主机的根目录和默认文件。

#### 3.2 静态文件缓存

nginx可以缓存静态文件，以提高访问速度。以下是缓存静态文件的配置示例：

```nginx
http {
  # ...
  proxy_cache_path    /var/cache/nginx levels=1:2 keys_zone=STATIC_CACHE:10m inactive=60m;
  proxy_cache_valid   200  60m;
  proxy_cache_valid   404  1m;
  # ...

  server {
    # ...
    location /images/ {
      proxy_cache           STATIC_CACHE;
      proxy_cache_key       "$request_uri";
      proxy_cache_valid     200 60m;
      proxy_cache_valid     404 1m;

      proxy_pass            http://backend_server;
    }
    # ...
  }
  # ...
}
```

其中，proxy_cache_path指定了缓存路径和大小，proxy_cache_valid指定了缓存的有效期。在server的location中，使用了proxy_cache指定了使用缓存，proxy_cache_key指定了缓存的key，proxy_pass指定了后端的服务器地址。

#### 3.3 gzip压缩

nginx支持gzip压缩，可以减小传输数据的大小，提高访问速度。以下是gzip压缩的配置示例：

```nginx
http {
  # ...
  gzip on;
  gzip_types text/plain text/css text/javascript application/json application/xml;

  server {
    # ...
    location / {
      # ...
      gzip_static on;
      # ...
    }
    # ...
  }
  # ...
}
```

其中，gzip on开启gzip压缩，gzip_types指定了需要被压缩的文件类型。在server的location中，使用了gzip_static on开启gzip静态文件压缩。

#### 3.4 负载均衡

nginx支持负载均衡，可以将请求分发到多个服务器上，实现高可用。以下是负载均衡的配置示例：

```nginx
http {
  # ...
  upstream backend_servers {
    server backend1.example.com:8080;
    server backend2.example.com:8080;
    server backend3.example.com:8080;
  }

  server {
    # ...
    location / {
      proxy_pass http://backend_servers;
    }
    # ...
  }
  # ...
}
```

其中，upstream块中指定了多台后端服务器的地址和端口。在server的location中，使用了proxy_pass将请求转发到后端服务器上。

#### 3.5 SSL配置

nginx支持SSL，可以加密传输数据，保护数据安全。以下是SSL的配置示例：

```nginx
http {
  # ...
  server {
    listen       443 ssl;
    server_name  example.com;
    ssl_certificate           /etc/nginx/ssl/example.crt;
    ssl_certificate_key       /etc/nginx/ssl/example.key;

    location / {
      # ...
    }
  }
}
```

其中，listen指定了监听的端口和协议，ssl_certificate和ssl_certificate_key指定了SSL证书和私钥文件的路径。



### 四、nginx配置文件重载

---

为了使nginx重新加载配置文件，可以使用nginx -s reload命令。