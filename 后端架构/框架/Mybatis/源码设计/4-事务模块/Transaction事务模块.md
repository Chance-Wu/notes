### 一、介绍

---

事务模块位于`org.apache.ibatis.transaction`包，其中的类均是事务相关的类：

```
org.apache.ibatis.transaction
		.jdbc
				JdbcTransaction.java
				JdbcTransactionFactory.java
		.managed
				ManagedTransaction.java
				ManagedTransactionFactory.java
		Transaction.java
		TransactionException.java
		TransactionFactory.java
```

事务模块采用了==工厂模式==。



### 二、事务接口

---

在Mybatis中有两种事务管理器类型（`JDBC` | `MANAGED`）

#### 2.1 Transaction

Transaction接口**对具体的事务类型进行抽象**，便于扩展。

事务接口，定义了如下方法：

- commit()-事务提交
- rollback()-事务回滚
- close()-关闭数据库连接
- getConnection()-获取数据库连接

#### 2.2 TransactionFactory

TransactionFactory接口是**对事务工厂的抽象**，便于具体类型的事务工厂的扩展。

事务工厂接口，定义了如下方法：

- setProperties(Properties props)-设置属性
- newTransaction(Connection conn)-创建事务实例
- newTransaction(DataSource dataSource, TransactionIsolationLevel level, boolean autoCommit)-创建事务实例



### 三、Mybatis事务类型

---

org.apache.ibatis.session.Configuration类。

```java
typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);
typeAliasRegistry.registerAlias("MANAGED", ManagedTransactionFactory.class);
```

#### 3.1 JDBC、MANAGED 的区别

- JDBC类型是直接使用JDK提供的JDBC来管理事务的各个环节：**提交、回滚、关闭等操作**。
- MANAGED类型则是什么都不做。那有什么意义呢？

单独使用Mybatis来构建项目时，我们要在Configuration配置文件中进行Environment配置，在其中要设置事务类型为JDBC类型的事务模型，因为在这个模型中定义了事务的各个方面，使用它可以完成事务的各项操作。而MANAGED类型的事务模型其实就是一个托管模型，也就是说它自身并不实现任何事务功能，而是托管出去由其他框架来实现，这个事务的具体实现就交由Spring之类的框架来实现，而且在使用SSM整合框架后已经不再需要单独配置环境信息了（包括事务配置与数据源配置），因为**在mybatis-spring.jar中拥有覆盖mybatis里面这部分逻辑的代码，实际情况是即使显式设置了相关配置，系统也会视而不见**。

**托管的意义显而易见，正是为整合而设。**

所以有关JDBC事务模块的内容明显不再是MyBatis功能中的重点，也许只有在单独使用MyBatis的少量系统中才会使用到。

#### 3.2 JDBC事务模型

JDBC事务的实现是对JDK中提供的JDBC事务模块的再封装，以适用于MyBatis环境。

MyBatis中的JDBC事务模块包括两个部分，分别为JDBC事务工厂和JDBC事务，整个事务模块采用的是抽象工厂模式，那么对应于每一项具体的事务处理模块必然拥有自己的事务工厂，事务模块实例通过事务工厂来创建（事务工厂将具体的事务实例的创建封装起来）。

首先来看JDBC事务工厂，`JdbcTransactionFactory`继承自`TransactionFactory接口`，实现了其中的两个新建事务实例的方法（参数不同）。

```java
public class JdbcTransactionFactory implements TransactionFactory {

  @Override
  public Transaction newTransaction(Connection conn) {
    return new JdbcTransaction(conn);
  }

  @Override
  public Transaction newTransaction(DataSource ds, TransactionIsolationLevel level, boolean autoCommit) {
    return new JdbcTransaction(ds, level, autoCommit);
  }
}
```

针对JDBC事务模型，在事务工厂的设置属性方法中没有任何执行代码，也就说明**JDBC事务模块并不支持设置属性的功能**，即使你在配置文件中设置的一些信息，也不会有任何作用。

那么`setProperties(Properties props)`方法有什么用呢？这个设置用于覆盖默认设置，只是JDBC事务模块并不支持而已，但并不代表别的事务模型不支持，同时这个方法也可用于功能扩展。

然后我们来看看JDBC事务类JdbcTransaction：

```java
protected Connection connection;
protected DataSource dataSource;
protected TransactionIsolationLevel level;
protected boolean autoCommit;
```

这四个参数分别对应事务工厂中的两个生产方法中的总共四个参数，对应的在事务类中定义了两个构造器，构造实例的同时进行赋值：

```java
public JdbcTransaction(DataSource ds, TransactionIsolationLevel desiredLevel, boolean desiredAutoCommit) {
  dataSource = ds;
  level = desiredLevel;
  autoCommit = desiredAutoCommit;
}

public JdbcTransaction(Connection connection) {
  this.connection = connection;
}
```

其次在该类中实现了Transaction接口，

```java
// 三个功能性方法：commit、rollback、close
// 其中都使用了connection来进行具体操作，因此这些方法使用的前提就是先获取connection数据库连接
@Override
public void commit() throws SQLException {
  if (connection != null && !connection.getAutoCommit()) {
    if (log.isDebugEnabled()) {
      log.debug("Committing JDBC Connection [" + connection + "]");
    }
    connection.commit();
  }
}

@Override
public void rollback() throws SQLException {
  if (connection != null && !connection.getAutoCommit()) {
    if (log.isDebugEnabled()) {
      log.debug("Rolling back JDBC Connection [" + connection + "]");
    }
    connection.rollback();
  }
}

@Override
public void close() throws SQLException {
  if (connection != null) {
    resetAutoCommit();
    if (log.isDebugEnabled()) {
      log.debug("Closing JDBC Connection [" + connection + "]");
    }
    connection.close();
  }
}
```

```java
// 一个获取数据库连接的方法
@Override
public Connection getConnection() throws SQLException {
  if (connection == null) {
    openConnection();
  }
  return connection;
}

protected void openConnection() throws SQLException {
  if (log.isDebugEnabled()) {
    log.debug("Opening JDBC Connection");
  }
  // 可见connection是从数据源dataSource中获取的
  connection = dataSource.getConnection();
  if (level != null) {
    connection.setTransactionIsolation(level.getLevel());
  }
  // 为connection中的自动提交赋值（true or false）
  setDesiredAutoCommit(autoCommit);
}
```

创建事务实例所提供的三个参数就是为connection服务的，其中

- DataSource是用来获取Connection实例的；
- 而TransactionIsolationLevel（事务级别）和boolean（自动提交）是用来填充connection的。

通过三个参数我们获得了一个圆满的Connection实例，然后我们就可以使用这个实例来进行事务操作：提交、回滚、关闭。



### 四、关于自动提交

---

在JdbcTransaction中可以看到关闭操作之前调用了一个方法resetAutoCommit()：

```java
@Override
public void close() throws SQLException {
  if (connection != null) {
    resetAutoCommit();
    if (log.isDebugEnabled()) {
      log.debug("Closing JDBC Connection [" + connection + "]");
    }
    connection.close();
  }
}

protected void resetAutoCommit() {
  try {
    if (!connection.getAutoCommit()) {
      if (log.isDebugEnabled()) {
        log.debug("Resetting autocommit to true on JDBC Connection [" + connection + "]");
      }
      connection.setAutoCommit(true);
    }
  } catch (SQLException e) {
    if (log.isDebugEnabled()) {
      log.debug("Error resetting autocommit to true "
                + "before closing the connection.  Cause: " + e);
    }
  }
}
```

如果设置自动提交为真，那么数据库将会将每一个SQL语句当做一个事务来执行，为了将多条SQL当做一个事务进行提交，必须将自动提交设置为false，然后进行手动提交。

这个方法中通过对connection实例中的自动提交设置（真或假）进行判断，如果为false，表明不执行自动提交，则复位，重新将其设置为true。这个操作执行在connection关闭之前。可以看做是**连接关闭之前的复位操作**。



### 五、问题

---

在JdbcTransaction中提供的两个构造器中以Connection为参数的构造器的作用是什么呢？

```java
public JdbcTransaction(Connection connection) {
    this.connection = connection;
}
```

我们需要自动组装一个完整的Connection，以其为参数来生产一个事务实例。这用在什么场景中呢？
