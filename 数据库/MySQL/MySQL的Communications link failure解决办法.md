https://www.oschina.net/question/1390315_2193452?sort=time



报错信息：

```
0:Communications link failure
           The last packet successfully received from the server was 87,537,289 milliseconds ago. The last packet sent successfully to the server was 0 milliseconds ago.
```

从服务器成功接收的最后一个数据包是87537289（24.3小时）毫秒前。最后一个成功发送到服务器的数据包是0毫秒前。

说明这个连接已经24个小时没有从服务器上接收数据了，而mysql上定义，==如果一个连接的空闲时间超过数据库的配置参数wait_timeout的值，MySQL将自动断开该连接，而连接池却认为该连接还是有效的(因为并未校验连接的有效性)==，当应用申请使用该连接时，就会导致上面的报错。而我们的数据库的配置参数wait_timeout的值，执行sql: show variables like  'wait_timeout' ， 值是86400 ，正好是24个小时。



==项目连接池配置中未配置testWhileIdle，则默认为false; 只有设置为true，连接池才会运行清理无效连接的线程；==



#### 1. 查看和连接mysql事件有关的系统变量

```shell
mysql -hlocalhost -u用户名 -p密码

show variables like '%timeout%';
```

```
+-----------------------------+----------+
| Variable_name               | Value    |
+-----------------------------+----------+
| connect_timeout             | 10       |
| delayed_insert_timeout      | 300      |
| have_statement_timeout      | YES      |
| innodb_flush_log_at_timeout | 1        |
| innodb_lock_wait_timeout    | 50       |
| innodb_rollback_on_timeout  | OFF      |
| interactive_timeout         | 28800    |
| lock_wait_timeout           | 31536000 |
| net_read_timeout            | 30       |
| net_write_timeout           | 60       |
| rpl_stop_slave_timeout      | 31536000 |
| slave_net_timeout           | 60       |
| wait_timeout                | 28800    |
+-----------------------------+----------+
```

>**（1）interactive_timeout**
>服务器关闭交互式连接前等待活动的秒数。
>
>交互式客户端定义为mysql_real_connect()中使用CLIENT_INTERACTIVE选项的客户端。参数默认值：28800秒（8小时）
>
>**（2）wait_timeout**
>服务器关闭非交互连接之前等待活动的秒数。
>
>在线程启动时，根据全局wait_timeout值或全局interactive_timeout值初始化会话wait_timeout值，取决于客户端类型(由mysql_real_connect()的连接选项CLIENT_INTERACTIVE定义)。
>参数默认值：28800秒（8小时）
>
>==wait_timeout==：超时控制的变量，为28800s，就是8小时之后会断开，需要重新连接，必须==调整系统变量来控制==。

==Linux环境中：1-31536000s==

配置文件`/etc/my.cnf`