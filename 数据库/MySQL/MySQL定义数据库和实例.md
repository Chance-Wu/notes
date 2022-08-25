- **数据库**：物理操作系统文件或其他形式文件类型的集合。在MySQL数据库中，数据文件可以是frm、MYD、MYI、ibd结尾的文件。
- **实例**：MySQL数据库由**后台线程**以及**一个共享内存区**组成。共享内存可以被运行的后台线程所共享。数据库实例才是真正用于操作数据库文件的。

实例与数据库的关系通常是一一对应的，即一个实例对应一个数据库。但是，在集群情况下可能存在一个数据库被多个数据实例使用的情况。

MySQL被设计为一个**单进程多线程**架构的数据库，即**MySQL数据库实例在系统上的表现就是一个进程**。这点与SQL Server比较类似，但与Oracle多进程的架构有所不同。

Linux系统通过如下命令启动MySQL数据库实例，并通过ps观察启动后的进程情况：

```shell
./mysqld_safe &

ps -ef | grep mysqld
  501 17685 85495   0 11:31下午 ttys001    0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn --exclude-dir=.idea --exclude-dir=.tox mysqld
    0 88015     1   0 一03下午 ttys001    0:00.02 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/usr/local/mysql/data --pid-file=/usr/local/mysql/data/chances-MacBook-Pro.local.pid
   74 88099 88015   0 一03下午 ttys001    3:15.56 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=chances-MacBook-Pro.local.err --pid-file=/usr/local/mysql/data/chances-MacBook-Pro.local.pid
```

进程号88099的进程就是MySQL实例。

启动实例时，MySQL数据库会读取配置文件，根据配置文件的参数来启动数据库实例。如下命令查看当MySQL数据库实例启动时，会在哪些位置查找配置文件。

```shell
mysql --help | grep my.cnf
                      order of preference, my.cnf, $MYSQL_TCP_PORT,
/etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf
```

可以看到，MySQL数据库是按 

`/etc/my.cnf` 

-> /etc/mysql/my.cnf

 -> /usr/local/mysql/etc/my.cnf 

-> ~/.my.cnf

的顺序读取配置文件的。如果几个配置文件都有同一个参数，那么会以最后一个为准。

Linux系统下默认的`datadir`为 `/usr/local/mysql/data`，不过该路径只是一个链接，具体如下：

```sql
system ls-lh /usr/local/mysql/data
```

通常MySQL数据库的权限为 `mysql:mysql`