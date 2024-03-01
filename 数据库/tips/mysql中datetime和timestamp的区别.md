### 一、时间戳

---

时间戳就是相对于格林威治时间1970年01月01日00时00分00秒起至现在过去的总秒数，这个总秒数在世界上的任何地方都是相同，也就是说**时间戳在世界上的任何地方都是相同的，时间戳没有时区的概念**。



### 二、时区

---

**时间和时间戳想比较，时间强调的是现在时，现在是何年何月何日，何时何分何妙，而时间戳强调的是过去时，过去了多少秒。**

通常我们将每天太阳在最高点的时候定义为12点，而由于地球的自转，导致每个地方太阳出现在最高点的时间又不尽相同，为了统一时间（太阳最高点为12点）于是就出现了时区的概念。

我们中国处于东+8区，美国处于西-5区

最后总结一个表达式：`实践戳+时区=当前时间`



### 三、mysql的时区

---

#### 3.1 查看时区

```bash
show variables like '%time_zone%';

+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | CST    |
| time_zone        | SYSTEM |
+------------------+--------+
```

system_time_zone是指**系统默认的时区**，time_zone指**当前时区**。

#### 3.2 修改时区

命令行修改：

```bash
set global time_zone = '+8:00'; ##修改mysql全局时区为北京时间，即我们所在的东8区
set time_zone = '+8:00'; ##修改当前会话时区
flush privileges; #立即生效
```

配置文件my.conf修改：

```bash
##在[mysqld]区域中加上
default-time_zone = '+8:00'
## 重启mysql
systemctl  restart mysql
```



#### 3.3 连接mysql客户端的时区

mysql有时区，连接mysql的客户端同样也有时区的问题，以java为例：

```java
// 加载JDBC驱动程序
Class.forName("com.mysql.jdbc.Driver");
            
// 建立与MySQL服务器的连接
            String url = "jdbc:mysql://localhost:3306/mydatabase";
            String username = "root";
            String password = "password";
            Connection connection = DriverManager.getConnection(url, username, password);
            
// 创建Statement对象来执行查询语句或更新操作
            Statement statement = connection.createStatement();
// 设置时区为UTC+8（北京时间）
            statement.executeUpdate("SET time_zone='Asia/Shanghai'");
```



### 四、操作系统时区

---

操作系统也是有时区的，输入`date -R`查看操作系统时区。看到后面是+0800，也就是代表北京时间。

```bash
date -R
Sat, 17 Feb 2024 09:45:57 +0800
```

操作系统时区影响的是依附操作系统获取时间的操作，例如在shell脚本中获取时间。

```bash
#!/bin/bash

current_time=$(date +"%Y-%m-%d %H:%M:%S")
echo "当前时间是：$current_time"
```

还有最常用的`crontab定时任务`，crontab中的时分秒肯定是依附于操作系统时区的。



### 五、datetime和timestamp的区别

---

#### 5.1 占用空间

| 类型      | 占据字节 | 表现形式            |
| --------- | -------- | ------------------- |
| datetime  | 8字节    | yyyy-mm-dd hh:mm:ss |
| timestamp | 4字节    | yyyy-mm-dd hh:mm:ss |

#### 5.2 表示范围

| 类型      | 表现范围                                                 |
| --------- | -------------------------------------------------------- |
| datetime  | 1000-01-01 00:00:00.000000 - 9999-12-31 23:59:59.999999  |
| timestamp | 1970-01-01 00:00:01.000000 to 2038-01-19 03:14:07.999999 |

#### 5.3 存储形式

- datetime类型的字段值写入的是什么，存储的就是什么，**与时区没有关系**。
- timestamp类型的字段值本质存储的是时间戳。比如写入的是2020-03-10 08:09:30，且mysql时区为+8,所以存储的是**UNIX_TIMESTAMP(2020-03-10 00:09:30)的值，与时区有关。**

#### 5.4 表现形式

- datetime和timestamp类型的字段，表现的都是yyyy-mm-dd hh:mm:ss，**即用户从数据库获取到的数据都是yyyy-mm-dd hh:mm:ss类型的，不会因为timestamp本质存储的是时间戳，而用户获取的就是时间戳，他俩的表现形式是一样的**。
- timestamp类型的表现形式过程：FROM_UNIXTIME(value)+时区=时间，最终把时间展示出来。

#### 5.5 写入形式

- datetime和timestamp的写入形式都是一样的，都必须是`时间格式的`，**不能因为timestamp本质存储的是时间戳，然后我们就可以直接写入时间戳**。

- timestamp类型的写入形式过程：UNIX_TIMESTAMP(时间-时区)=时间戳，然后把该时间戳存起来。**timestamp是不能直接写入时间戳的**，实验如下，created_time是timestamp格式的。

  ```sql
  INSERT INTO users(name,created_time) values('ff',1584417730);
  1054 - Unknown column 'created_time' in 'field list'
  
  mysql> INSERT INTO users(name,created_at) values('ff','2020-03-10');
  Query OK, 1 row affected (0.01 sec)
  ```

#### 5.6 时区

| 类型      | 时区影响 |
| --------- | -------- |
| datetime  | 没有影响 |
| timestamp | 有影响   |

通过上面的分析，我们知道了mysql的时区大概作用于以下俩点

> 1. **timestamp类型的数据，mysql实际存储的是`时间戳`，而表现形式却是`时间`，时间戳转化为时间肯定需要时区的转化。`时间戳+时区=时间。`**
>
> 2. **mysql中的函数FROM_UNIXTIME()(把时间戳转化为时间)和UNIX_TIMESTAMP()(把时间转化为时间戳)也会根据mysql时区进行的转换**

```sql
SELECT FROM_UNIXTIME(1708143330);

SELECT UNIX_TIMESTAMP('2024-02-17 12:15:30');
```



### 六、mysql存储时间时选择datetime,timestamp,int还是varchar类型

---

![img](img/webp)

首先varchar类型是肯定不建议的，有以下俩点原因：

- 字符串占用的空间更大；
- 字符串存储的日期比较效率比较低（逐个字符进行比对），无法用日期相关的 API 进行计算和比较。

int和timestamp类型相比较发现，占用字节和范围表示是相同：

- timestamp与时区有关系，如果需要根据时区变化，选择timestamp即可，如果不需要选择int类型即可。

timestamp和datetime相比较，主要以下俩个原因：

- DateTime 类型没有时区信息
- DateTime 类型所表示的范围更大



### 七、线上迁库事故

---

数据库是使用timestamp类型来存储时间的，当前的数据库的时区为+8:00。

有一次我们需要迁库，迁移的那个库没太注意时区，迁移过去之后发现所有用户的每天获取的积分都不对了，获取积分的时间都少了8个小时。

最后反应过来可能是新的数据库的时区与老数据库不一致，查看后发现新数据库的时区果然不是+8:00，时区修改完之后就和之前的时间对上了。

所以存储时间的类型是很重要的，选择时需要考虑清楚。



### 八、总结

---

如果数据库中有timestamp类型的字段，**mysql数据库不管是迁库，还是集群，都一定要保证时区的相同。如果mysql集群中的数据库时区不一致，timestamp的字段将会造成数据不一致的情况发生。**`在迁移库或者搭建集群时一定检查时区，保证时区的相同`。

中国时区默认是+8,所以不管是单节点mysql，还是mysql集群，我们第一件事就是应该将当前时区`time_zone`设置为`+8:00`。