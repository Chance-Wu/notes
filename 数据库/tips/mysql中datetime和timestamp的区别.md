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































