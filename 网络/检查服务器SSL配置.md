1. 检查SSL/TSL版本的支持情况

   ```shell
   openssl s_client -connect yourserver.com:443 -tls1_2
   ```

   如果连接成功，那么你的服务器就支持指定的TLS版本。你可以通过改变`-tls1_2`参数为`-ssl2`, `-ssl3`, `-tls1`, `-tls1_1`等来测试其他版本的支持情况。

2. 检查支持的加密套件

   ```shell
   openssl ciphers -v 'ALL' | grep -i "yourservername"
   ```

   或者直接连接到服务器并查看支持的套件：

   ```shell
   echo | openssl s_client -showcerts -connect yourserver.com:443 -cipher
   ```