当MySQL实例启动时，会将自己的进程ID写入一个pid文件。该文件可由参数 `pid_file` 控制，默认位于数据库目录下，文件名为主机名.pid。

```sql
mysql> show variables like 'pid_file';
+---------------+----------------------------------------+
| Variable_name | Value                                  |
+---------------+----------------------------------------+
| pid_file      | /usr/local/mysql/data/mysqld.local.pid |
+---------------+----------------------------------------+
1 row in set (0.01 sec)
```