连接MySQL操作是一个连接进程和MySQL数据库实例进行通信。从程序设计的角度来说。本质上是**进程通信**。（管道、命名管道、命名字、TCP/IP套接字、UNIX域套接字）



### 一、TCP/IP

---

客户端与MySQL实例两台机器通过一个TCP/IP网络连接。

```shell
mysql -h192.168.0.101 -uroot -p
```

在通过TCP/IP连接到MySQL实例时，MySQL数据库会先检查一张权限视图user，用来判断发起请求的客户端IP是否允许连接到MySQL实例。



### 二、命名管道和共享内存

---

如果两个需要进程通信的进程在**同一台服务器**上，那么可以使用命名管道，在MySQL数据库中须在配置文件中启用 `--enable-named-pipe`选项。4.1版本后，还提供了共享内存的连接方式，这是通过在配置文件中添加 `-shared-memory` 实现的。如果想使用共享内存的方式，在连接时，MySQL客户端还必须使用 `--protocol=memory` 选项。



### 三、UNIX域套接字

---

在Linux和UNIX环境下，还可以使用UNIX域套接字。UNIX域套接字其实不是一个网络协议，所以只能在MySQL客户端和数据库实例在一台服务器上的情况下使用。用户可以在配置文件中指定套接字文件的路径，如 `--socket=/tmp/mysql.sock`。当数据库实例启动后，用户可以通过下列命令来进行UNIX域套接字文件的查找：

```shell
mysql> show variables like 'socket';
+---------------+-----------------+
| Variable_name | Value           |
+---------------+-----------------+
| socket        | /tmp/mysql.sock |
+---------------+-----------------+
1 row in set (0.01 sec)
```

在知道了UNIX域套接字文件的路径后，就可以使用该方式进行连接了，如下所示：

```shell
mysql -uroot -S /tmp/mysql.sock
```