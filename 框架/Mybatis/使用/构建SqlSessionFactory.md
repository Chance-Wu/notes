#### 1. 获取SqlSessionFactory

##### 1.1 从XML中构建 SqlSessionFactory

>每个基于 MyBatis 的应用都是==以一个 SqlSessionFactory 的实例为核心==。
>
>SqlSessionFactory 的实例可以通过 `SqlSessionFactoryBuilder` 获得。而 SqlSessionFactoryBuilder 则可以从 XML 配置文件或一个预先配置的 Configuration 实例来构建出 SqlSessionFactory 实例。
>
>建议使用类路径下的资源文件进行配置。 但也可以使用任意的输入流（InputStream）实例，比如用文件路径字符串或 `file:// URL` 构造的输入流。MyBatis 包含一个名叫 Resources 的工具类，它包含一些实用方法，使得从类路径或其它位置加载资源文件更加容易。
>
>```java
>public static SqlSessionFactory getSqlSessionFactoryFromXml() {
>    String resource = "mybatis-config.xml";
>    InputStream inputStream;
>    SqlSessionFactory sqlSessionFactory = null;
>    try {
>        inputStream = Resources.getResourceAsStream(resource);
>        sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
>    } catch (IOException e) {
>        e.printStackTrace();
>    }
>    return sqlSessionFactory;
>}
>```
>
>XML 配置文件中包含了对 MyBatis 系统的核心设置：
>
>- 获取数据库连接实例的数据源（DataSource）
>- 以及决定事务作用域和控制方式的事务管理器（TransactionManager）。
>
>```xml
><?xml version="1.0" encoding="UTF-8" ?>
><!DOCTYPE configuration
>        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
>        "http://mybatis.org/dtd/mybatis-3-config.dtd">
><configuration>
>
>    <environments default="development">
>        <environment id="development">
>            <transactionManager type="JDBC"></transactionManager>
>            <dataSource type="POOLED">
>                <property name="driver" value="${driver}"/>
>                <property name="url" value="${url}"/>
>                <property name="username" value="${username}"/>
>                <property name="password" value="${password}"/>
>            </dataSource>
>        </environment>
>    </environments>
>
>    <!-- 一组映射器 -->
>    <mappers>
>        <mapper resource="com/chance/spring/dao/BlogMapper.xml"/>
>    </mappers>
></configuration>
>```

##### 1.2 不使用XML构建 SqlSessionFactory

>```java
>public static SqlSessionFactory getSqlSessionFactory() {
>    // 1.构建数据库连接池
>    PooledDataSource dataSource = new PooledDataSource();
>    dataSource.setDriver("org.mysql.jdbc.Driver");
>    dataSource.setUsername("root");
>    dataSource.setPassword("539976");
>    dataSource.setUrl("jdbc:mysql://localhost:3306?test");
>    dataSource.setDefaultAutoCommit(false);
>
>    // 2.构建数据库事务方式
>    JdbcTransactionFactory transactionFactory = new JdbcTransactionFactory();
>
>    // 3.创建数据库运行环境
>    Environment environment = new Environment("development", transactionFactory, dataSource);
>
>    Configuration configuration = new Configuration(environment);
>    configuration.getTypeAliasRegistry().registerAlias("blog", Blog.class);
>    configuration.addMapper(BlogMapper.class);
>
>    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(configuration);
>
>    return sqlSessionFactory;
>}
>```

#### 2. 从SqlSessionFactory中获取SqlSession

>既然有了SqlSessionFactory，我们可以从中获得SqlSession的实例。
>
>SqlSession提供了在数据库执行SQL命令所需的所有方法。可以通过SqlSesssion实例来直接执行已映射的SQL语句。
>
>```java
>/**
> * 获取 SqlSession 实例
> */
>public static SqlSession getSqlSession() {
>    SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
>    SqlSession sqlSession = sqlSessionFactory.openSession();
>    return sqlSession;
>}
>```

>```java
>// 方式一
>try (SqlSession sqlSession = getSqlSessionFactory().openSession()) {
>    List<Blog> blog = sqlSession.selectList("com.chance.spring.dao.BlogMapper.selectBlog");
>}catch (Exception e) {
>    e.printStackTrace();
>}
>```

>```java
>// 获取SqlSession实例来执行已映射的SQL语句
>try(SqlSession sqlSession = getSqlSessionFactory().openSession()) {
>    BlogMapper blogMapper= sqlSession.getMapper(BlogMapper.class);
>    List<Blog> blog = blogMapper.selectBlog();
>} catch (Exception e) {
>    e.printStackTrace();
>}
>```

#### 3. 探究已映射的SQL语句

>在上面提到的例子中，一个语句既可以==通过 XML 定义==，也可以==通过注解定义==。

>先看看 XML 定义语句的方式。
>
>```xml
><?xml version="1.0" encoding="UTF-8" ?>
><!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
><mapper namespace="com.chance.spring.dao.BlogMapper">
>    <select id="selectBlog" resultType="com.chance.spring.dao.entity.Blog">
>        select id,text from blog
>    </select>
></mapper>
>```
>
>一个XLM映射文件中，可以定义无数个映射语句。
>
>它在命名空间"com.chance.spring.dao.BlogMapper"中定义了一个名为"selectBlog"映射语句，这样你就可以用全限定名"com.chance.spring.dao.BlogMapper.selectBlog"来调用映射语句了。
>
>```java
>List<Blog> blog = sqlSession.selectList("com.chance.spring.dao.BlogMapper.selectBlog");
>```
>
>这种方式和用全限定名调用Java对象的方法类似。这样，==该命名就可以直接映射到在命名空间中同名的映射器类，并将已映射的select语句匹配到对应名称、参数和返回类型的方法==。因此就可以在对应的映射器接口调用方法，如下：
>
>```java
>BlogMapper blogMapper = sqlSession.getMapper(BlogMapper.class);
>List<Blog> blog = blogMapper.selectBlog();
>```

>第二种方法的优势：
>
>- 不依赖于字符串字面值，更安全；
>- 如果IDE有代码补全功能，那么代码补全可以帮助快速选择到映射好的SQL语句。

##### 3.1 命名空间的作用

>1. 利用更长的全限定名来将不同的语句隔离开来
>2. 实现接口绑定

##### 3.2 命名解析

>为了减少输入量，Mybatis对所有具有名称的配置元素（包括语句、结果映射、缓存等）使用了如下的命名解析规则。
>
>- 全限定性名——如"com.chance.spring.dao.BlogMapper.selectBlog"将被直接用于查找及使用。
>- 短名称——如"selectBlog"，如果全局唯一也可以作为一个单独引用。如果不唯一，有两个或两个以上的相同名称，那么使用时就会产生“短名称不唯一”的错误。

##### 3.3 Java注释来映射SQL语句

>对于像 BlogMapper 这样的映射器类来说，还有另一种方法来完成语句映射。 它们映射的语句可以不用 XML 来配置，而使用 Java 注解来配置。
>
>```java
>public interface BlogMapper {
>
>    @Select("SELECT id,text FROM blog WHERE id = #{id}")
>    Blog selectOneBlog(int id);
>}
>```
>
>- 使用注解来映射简单语句会使代码显得更加简洁；
>- 如果需要做一些很复杂的操作，最好用 XML 来映射语句。

#### 4. 作用域Scope和生命周期

>依赖注入框架可以创建线程安全的、基于事务的 SqlSession 和映射器，并将它们直接注入到你的 bean 中，因此可以直接忽略它们的生命周期。

#### 5. SqlSessionFactoryBuilder

>这个类可以被实例化、使用和丢弃，一旦创建了SqlSessionFactory，就不再需要它了。
>
>因此`SqlSessionFactoryBuilder`实例的最佳作用域是方法作用域——==局部方法变量==。可以重用 SqlSessionFactoryBuilder 来创建多个 SqlSessionFactory 实例，但最好还是不要一直保留着它，以保证所有的 XML 解析资源可以被释放给更重要的事情。

#### 6. SqlSessionFactory

>SqlSessionFactory一旦被创建就应该==在应用的运行期间一直存在==。
>
>因此 SqlSessionFactory 的最佳作用域是应用作用域。 有很多方法可以做到，最简单的就是使用单例模式或者静态单例模式。

#### 7. SqlSession

>- ==每个线程都应该有它自己的 SqlSession 实例==。
>- SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。
>- 绝对不能将 SqlSession 实例的引用放在一个类的静态域，甚至一个类的实例变量也不行。 
>- 也绝不能将 SqlSession 实例的引用放在任何类型的托管作用域中，比如 [Servlet 框架](https://www.w3cschool.cn/servlet/)中的 [HttpSession](https://www.w3cschool.cn/servlet/servlet-session-tracking.html)。
>
>如果你现在正在使用一种 [Web 框架](https://www.w3cschool.cn/webservices/)，考虑将 SqlSession 放在一个和 [HTTP](https://www.w3cschool.cn/http/) 请求相似的作用域中。换句话说，每次收到 [HTTP 请求](https://www.w3cschool.cn/http/yerxcfmt.html)，就可以打开一个 SqlSession，返回一个响应后，就关闭它。这个关闭操作很重要，为了确保每次都能执行关闭操作，你应该把这个关闭操作放到 [finally ](https://www.w3cschool.cn/java/exception-finally.html)块中。 下面的示例就是一个确保 SqlSession 关闭的标准模式：
>
>```java
>try (SqlSession session = sqlSessionFactory.openSession()) {
>    // 你的应用逻辑代码
>}
>```

#### 8. 映射器实例

>- 映射器是一些绑定映射语句的接口。
>- 映射器接口的实例是从 SqlSession 中获得的。
>- 虽然从技术层面上来讲，任何映射器实例的最大作用域与请求它们的 SqlSession 相同。但方法作用域才是映射器实例的最合适的作用域。
>- 映射器实例应该在调用它们的方法中被获取，使用完毕之后即可丢弃。映射器实例并不需要被显式地关闭。==最好将映射器放在方法作用域内==。就像下面的例子一样：
>
>```java
>try (SqlSession sqlSession = getSqlSessionFactory().openSession()) {
>    BlogMapper blogMapper = sqlSession.getMapper(BlogMapper.class);
>
>    // 应用逻辑代码
>}
>```



