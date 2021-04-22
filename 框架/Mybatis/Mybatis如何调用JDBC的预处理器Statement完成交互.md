#### 1. JDBC执行过程

>获得连接——》		Connection
>
>预编译SQL——》	PreparedStatement
>
>执行SQL——》		ResultSet
>
>读取结果					Java Bean

##### 1.1 预编译的三种执行器

><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gm2670k7urj30sa0esmxl.jpg" style="zoom:60%">
>
>- `Statement`存在sql注入问题，发送一条一条静态sql语句(包含参数)，传输体量比较大。
>- `PreparedStatement`可以防止sql注入问题，发送一条sql语句包含若干组参数，传输体量小。
>- `CallableStatement`支持调用存储过程，提供了对输出和输入/输出参数(INOUT)的支持。

##### 1.2 Statement和PreparedStatement

>（1）Statement
>
>	- 执行静态sql存在sql注入问题，不需要设置参数
>	- 相同的sql查询参数不同时会组装成不同参数的多条静态sql进行多次发送，从而多次编译多次执行。
>
>（2）PreparedStatement
>
>	- 执行动态sql可以防止sql注入问题，需要设置预编译参数。
>	- 相同的sql查询参数不同时会组装成一条动态sql，多组不同参数进行发送，从而一次编译多次执行。

#### 2. mybatis执行过程

>1. SqlSession（门面模式）：基本API——增删改查，辅助API——提交、关闭会话
>2. Executor：基本API——增、改缓存维护，辅助API——提交、关闭执行器、批处理刷新
>3. StatementHandler：封装jdbc，参数处理，结果处理
>
>==SqlSession采用门面模式——方便调用==

##### 2.1 mybatis 的 Executor 体系

><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gm280exvygj30tq0ecwf8.jpg" style="zoom:70%">
>
>`CachingExecutor`类中是MyBatis关于二级缓存相关的逻辑。
>
>`BaseExecutor`抽象类中是MyBatis关于一级缓存相关的逻辑。
>
>==SimpleExecutor、ReuseExecutor、BatchExecutor是继承自BaseExecutor具体的执行器类,具体执行数据库的增删改查==。

>**（1）SimpleExecutor简单执行器**
>
>简单执行器每次都会创建一个新的预处理器（PreparedStatement）
>
>首先查看`ConnectionLogger`，该类是mybatis调用jdbc三种预处理器Statement进行交互的一个类（后面会介绍mybatis是如何调用jdbc框架的）。
>
>```java
>public Object invoke(Object proxy, Method method, Object[] params)
>    throws Throwable {
>    try {
>        if (Object.class.equals(method.getDeclaringClass())) {
>            return method.invoke(this, params);
>        }
>        if ("prepareStatement".equals(method.getName()) || "prepareCall".equals(method.getName())) {
>            if (isDebugEnabled()) {
>                debug(" Preparing: " + removeExtraWhitespace((String) params[0]), true);
>            }
>            PreparedStatement stmt = (PreparedStatement) method.invoke(connection, params);
>            stmt = PreparedStatementLogger.newInstance(stmt, statementLog, queryStack);
>            return stmt;
>        } else if ("createStatement".equals(method.getName())) {
>            Statement stmt = (Statement) method.invoke(connection, params);
>            stmt = StatementLogger.newInstance(stmt, statementLog, queryStack);
>            return stmt;
>        } else {
>            return method.invoke(connection, params);
>        }
>    } catch (Throwable t) {
>        throw ExceptionUtil.unwrapThrowable(t);
>    }
>}
>```

>**（2）ReuseExecutor可重用执行器**
>
>相同的sql只进行一次预处理，执行结果多次使用。
>
>由以下源码可知：ResueExecutor通过属性statementMap缓存Statement来实现可重用性。
>
>```java
>public class ReuseExecutor extends BaseExecutor {
>  
>  ...
>
>  // 判断如果Statement存在了直接从缓存中获取，若不存在创建并添加到缓存中
>  private Statement prepareStatement(StatementHandler handler, Log statementLog) throws SQLException {
>    Statement stmt;
>    BoundSql boundSql = handler.getBoundSql();
>    String sql = boundSql.getSql();
>    if (hasStatementFor(sql)) {
>      stmt = getStatement(sql);
>      applyTransactionTimeout(stmt);
>    } else {
>      Connection connection = getConnection(statementLog);
>      stmt = handler.prepare(connection, transaction.getTimeout());
>      putStatement(sql, stmt);
>    }
>    handler.parameterize(stmt);
>    return stmt;
>  }
>
>  // 判断缓存是否存在且有效
>  private boolean hasStatementFor(String sql) {
>    try {// 缓存存在且缓存未失效（会话未关闭）
>      Statement statement = statementMap.get(sql);
>      return statement != null && !statement.getConnection().isClosed();
>    } catch (SQLException e) {
>      return false;
>    }
>  }
>
>  private Statement getStatement(String s) {
>    return statementMap.get(s);
>  }
>
>  private void putStatement(String sql, Statement stmt) {
>    statementMap.put(sql, stmt);
>  }
>
>}
>```

>**（3）BatchExecutor批处理执行器**
>
>BatchExecutor批处理执行器顾名思义支持批量处理。
>
>由以下源码可知，BatchExecutor通过属性statementList存储Statement和属性batchResultList存储返回的结果集BatchResult(此时还没有真正填充结果集，只是new了一个对象)，从而实现批量处理的功能。
>
>```java
>public class BatchExecutor extends BaseExecutor {
>
>  public static final int BATCH_UPDATE_RETURN_VALUE = Integer.MIN_VALUE + 1002;
>
>  private final List<Statement> statementList = new ArrayList<>();
>  private final List<BatchResult> batchResultList = new ArrayList<>();
>  private String currentSql;
>  private MappedStatement currentStatement;
>
>  public BatchExecutor(Configuration configuration, Transaction transaction) {
>    super(configuration, transaction);
>  }
>
>  @Override
>  public int doUpdate(MappedStatement ms, Object parameterObject) throws SQLException {
>    final Configuration configuration = ms.getConfiguration();
>    final StatementHandler handler = configuration.newStatementHandler(this, ms, parameterObject, RowBounds.DEFAULT, null, null);
>    final BoundSql boundSql = handler.getBoundSql();
>    final String sql = boundSql.getSql();
>    final Statement stmt;
>    if (sql.equals(currentSql) && ms.equals(currentStatement)) {
>      int last = statementList.size() - 1;
>      stmt = statementList.get(last);
>      applyTransactionTimeout(stmt);
>      handler.parameterize(stmt);// fix Issues 322
>      BatchResult batchResult = batchResultList.get(last);
>      batchResult.addParameterObject(parameterObject);
>    } else {
>      Connection connection = getConnection(ms.getStatementLog());
>      stmt = handler.prepare(connection, transaction.getTimeout());
>      handler.parameterize(stmt);    // fix Issues 322
>      currentSql = sql;
>      currentStatement = ms;
>      statementList.add(stmt);
>      batchResultList.add(new BatchResult(ms, sql, parameterObject));
>    }
>    handler.batch(stmt);
>    return BATCH_UPDATE_RETURN_VALUE;
>  }
>  
>  ...
>
>}
>```

#### 3. mybatis究竟是如何调用jdbc的

>首先了解StatementHandler体系：
>
><img src="https://tva1.sinaimg.cn/large/0081Kckwgy1gm2f5q0l19j319e0oswfu.jpg" style="zoom:50%">
>
>RoutingStatementHandler类的源码：
>
>由以下源码可知该类是通过MappedStatement的属性statementType来判断mybatis执行时使用的是哪个Handler。
>
>```java
>public class RoutingStatementHandler implements StatementHandler {
>
>  private final StatementHandler delegate;
>
>  public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {
>
>    switch (ms.getStatementType()) {
>      case STATEMENT:
>        delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
>        break;
>      case PREPARED:
>        delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
>        break;
>      case CALLABLE:
>        delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
>        break;
>      default:
>        throw new ExecutorException("Unknown statement type: " + ms.getStatementType());
>    }
>
>  }
>  
>  ...
>  
>}
>```
>
>假设Mybatis使用的SimpleStatementHandler执行器，MappedStatement的statementType属性值为PREPARED，以此为例进行讲解mybatis具体调用JDBC预处理器Statement的流程。

>首先Mybatis使用的SimpleStatement配置的statementType属性值PREPARED，由`RountingStatementHandler`路由分配得到PreparedStatementHandler，而后执行PreparedStatementHandler的`instantiateStatement()`方法具体代码如下：
>
>```java
>protected Statement instantiateStatement(Connection connection) throws SQLException {
>  String sql = boundSql.getSql();
>  if (mappedStatement.getKeyGenerator() instanceof Jdbc3KeyGenerator) {
>    String[] keyColumnNames = mappedStatement.getKeyColumns();
>    if (keyColumnNames == null) {
>      return connection.prepareStatement(sql, PreparedStatement.RETURN_GENERATED_KEYS);
>    } else {
>      return connection.prepareStatement(sql, keyColumnNames);
>    }
>  } else if (mappedStatement.getResultSetType() == ResultSetType.DEFAULT) {
>    return connection.prepareStatement(sql);
>  } else {
>    return connection.prepareStatement(sql, mappedStatement.getResultSetType().getValue(), ResultSet.CONCUR_READ_ONLY);
>  }
>}
>```
>
>进一步走入ConnectionLogger代理类的invoke()方法，会根据上一步的方法名创建对应的PreparedStatement：
>
>```java
>public Object invoke(Object proxy, Method method, Object[] params)
>  throws Throwable {
>  try {
>    if (Object.class.equals(method.getDeclaringClass())) {
>      return method.invoke(this, params);
>    }
>    if ("prepareStatement".equals(method.getName()) || "prepareCall".equals(method.getName())) {
>      if (isDebugEnabled()) {
>        debug(" Preparing: " + removeExtraWhitespace((String) params[0]), true);
>      }
>      PreparedStatement stmt = (PreparedStatement) method.invoke(connection, params);
>      stmt = PreparedStatementLogger.newInstance(stmt, statementLog, queryStack);
>      return stmt;
>    } else if ("createStatement".equals(method.getName())) {
>      Statement stmt = (Statement) method.invoke(connection, params);
>      stmt = StatementLogger.newInstance(stmt, statementLog, queryStack);
>      return stmt;
>    } else {
>      return method.invoke(connection, params);
>    }
>  } catch (Throwable t) {
>    throw ExceptionUtil.unwrapThrowable(t);
>  }
>}
>```
>
>此处ConnectionLogger类是一个代理类，其主要功能是==对instantiateStatement()方法中调用的prepareStatement、prepareCall、createStatement方法进行判断分别创建jdbc对应的Statement的代理日志类，完成mybatis与jdbc的交互==。
>
>以上各个Statement的代理日志类其核心功能还是Statement对象，只是通过代理模式增强添加了日志打印；由此，可以借鉴系统与系统或接口与接口之间的交互在某种程度上可以使用代码模式进行交互以方便增强添加日志打印监控功能。



