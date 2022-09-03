### 一、Mybatis是什么？

---

ORM框架（Object Relational Mapping）

把`字段`映射为`对象的属性`。

```
user表
id int key auto_increment,
name varchar(10) not null,
age int not null
```

```java
class User {
    private Integer id;
    private String name;
    private Integer age;
}
```



### 二、为什么这两者要建立关联？

---

#### 2.1 映射之前

1. 导入JDBC驱动包
2. 通过DriverManager注册驱动
3. ==创建连接==
4. ==创建Statement==
5. ==CRUD==
6. ==操作结果集==
7. 关闭连接

```java
Connection connection = null;
Statement statement = null;
try {
  // 创建连接
  connection = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/test",
    "root",
    "539976");

  // 创建Statement
  statement = connection.createStatement();

  // CRUD
  ResultSet resultSet = statement.executeQuery("select * from user");

  //
  while (resultSet.next()) {
    String username = resultSet.getString("username");
    System.out.println(username);
  }
} catch (SQLException e) {
  e.printStackTrace();
} finally {
  // 关闭连接
  try {
    if (null != statement) {
      statement.close();
    }
  } catch (SQLException e) {
    logger.info("异常：{}", e.getMessage());
  } finally {
    try {
      if (null != connection) {
        connection.close();
      }
    } catch (SQLException e) {
      logger.info("异常：{}", e.getMessage());
    }
  }
}
```

>上述标注的难点：
>
>- 手动打开关闭数据库连接
>- sql写在业务代码里
>- 需要手动处理结果集

#### 2.2 映射之后

1. 引入jdbc驱动包
2. 引入mybatis包

```java
DataSource dataSource = BlogDataSourceFactory.getBlogDataSource();
TransactionFactory transactionFactory = new JdbcTransactionFactory();
Environment environment = new Environment("development", transactionFactory, dataSource);
Configuration configuration = new Configuration(environment);
configuration.addMapper(BlogMapper.class);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);

try (SqlSession session = sqlSessionFactory.openSession()) {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
}
```

>整合Spring后会更方便。
>
>优点：
>
>- **把sql语句从java代码中抽取出来**，方便维护，并且修改sql代码时，不需要修改java代码；
>- 不用手动设置参数和对结果集的处理；（ResultSet直接转为java对象）
>- 减少代码里，提高开发效率。