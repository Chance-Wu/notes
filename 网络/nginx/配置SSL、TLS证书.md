### 一、获取SSL证书

---

- 你可以从证书颁发机构（CA）购买SSL证书，或者使用免费的Let's Encrypt证书。
- 如果只是测试环境，你也可以生成自签名证书。



### 二、安装依赖

---

确保你的服务器上安装了Nginx和OpenSSL。



### 三、步骤

---

1. 生成自签名证书（仅适用于测试环境）

   - 使用以下命令生成自签名证书

     ```shell
     openssl req -newkey rsa:2048 -nodes -keyout server.key -x509 -days 365 -out server.crt
     ```

     这将创建两个文件这将创建两个文件：`server.crt`（公钥证书）和`server.key`（私钥）。

2. 安装Let's Encrypt证书（推荐用于生产环境）

   - 使用Let's Encrypt可以通过ACME协议自动获取和更新证书。
   - 你可以使用Certbot工具来自动化这个过程。

3. 配置Nginx

   - 编辑nginx配置文件，通常位于 `/etc/nginx/conf.d/` 或 `/etc/nginx/sites-available/` 目录下。



### 四、配置注意事项

---

```nginx
server {
    listen 443 ssl; # HTTPS监听端口
    server_name example.com www.example.com; # 主机名

    # SSL证书路径
    ssl_certificate /path/to/server.crt;
    ssl_certificate_key /path/to/server.key;

    # SSL额外配置
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # 支持的TLS版本
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384'; # 支持的加密套件
    ssl_prefer_server_ciphers on; # 优先使用服务器的加密套件
    ssl_session_cache shared:SSL:1m; # SSL会话缓存
    ssl_session_timeout 10m; # SSL会话超时时间

    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubdomains" always;

    # Content Security Policy
    add_header Content-Security-Policy "default-src 'self'"; # 自定义CSP策略

    # CORS
    add_header Access-Control-Allow-Origin "*";

    location / {
        # 你的网站内容配置
        root /usr/share/nginx/html;
        index index.html index.htm;
    }
}
```

- **SSL证书路径** (`ssl_certificate` 和 `ssl_certificate_key`) 需要指向正确的文件位置。
- `server_name` 指定你的域名，确保它与证书中列出的域名匹配。
- `ssl_protocols` 和 `ssl_ciphers` 可以根据你的安全需求进行调整。
- HSTS (`Strict-Transport-Security`) 可以强制客户端通过HTTPS访问你的站点。
- CSP (`Content-Security-Policy`) 和 CORS (`Access-Control-Allow-Origin`) 可以增强安全性。

检查和重启nginx：

1. 检查nginx配置文件是否有语法错误

   ```shell
   nginx -t
   ```

2. 如果没有错误，重启nginx以使更改生效

   ```shell
   systemctl restart nginx
   ```