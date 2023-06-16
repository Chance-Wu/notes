### 一、SQL映射文件的顶级元素

SQL 映射文件只有很少的几个顶级元素（按照应被定义的顺序列出）：

- cache——给定命名空间的缓存配置
- cache-ref——其他命名空间缓存配置的引用
- resultMap——描述如何从数据库结果集中来加载对象
- sql——可被其他语句引用的可重用语句块
- insert
- update
- delete
- select

#### 1.1 select

```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

这个语句名为 selectPerson，接受一个 int（或 Integer）类型的参数，并返回一个 **HashMap 类型的对象（键是列名，值是结果行中的对应值）**。

注意参数符号： `#{id}`

即告诉 MyBatis **创建一个预处理语句（PreparedStatement）参数**，在 JDBC 中，这样的一个参数在 SQL 中会由一个“?”来标识，并被传递到一个新的预处理语句中，如下：

```java
// 近似的 JDBC 代码，非 MyBatis 代码...
String selectPerson = "SELECT * FROM PERSON WHERE ID=?";
PreparedStatement ps = conn.prepareStatement(selectPerson);
ps.setInt(1,id);
```

select 元素允许配置很多属性来配置每条语句的行为细节。

| 属性            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| id              | 在命名空间中唯一的标识符，可以被用来引用这条语句             |
| `parameterType` | 将会传入这条语句的参数类的完全限定名或别名。**MyBatis 可以通过 TypeHandler 推断出具体传入语句的参数**，默认值为 unset |
| `resultType`    | 从这条语句中返回的期望类型的类的完全限定名或别名。**注意如果是集合情形，那应该是集合可以包含的类型，而不能是集合本身**。使用 resultType 或 resultMap，但不能同时使用。 |
| resultMap       | 外部 resultMap 的命名引用。结果集的映射是 MyBatis 最强大的特性，对其有一个很好的理解的话，许多复杂映射的情形都能迎刃而解。使用 resultMap 或 resultType，但不能同时使用。 |
| flushCache      | 将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：false。 |
| useCache        | 将其设置为 true，将会导致本条语句的结果被二级缓存，默认值：对 select 元素为 true。 |
| timeout         | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）。 |
| fetchSize       | 这是尝试影响驱动程序每次批量返回的结果行数和这个设置值相等。默认值为 unset（依赖驱动）。 |
| `statementType` | STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| resultSetType   | FORWARD_ONLY，SCROLL_SENSITIVE 或 SCROLL_INSENSITIVE 中的一个，默认值为 unset （依赖驱动）。 |
| databaseId      | 如果配置了 databaseIdProvider，MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。 |
| resultOrdered   | 这个设置仅针对嵌套结果 select 语句适用：如果为 true，就是假设包含了嵌套结果集或是分组了，这样的话当返回一个主结果行的时候，就不会发生有对前面结果集的引用的情况。这就使得在获取嵌套的结果集的时候不至于导致内存不够用。默认值：false。 |
| resultSets      | 这个设置仅对多结果集的情况适用，它将列出语句执行后返回的结果集并每个结果集给一个名称，名称是逗号分隔的。 |

#### 1.2 insert、update、delete

```xml
<insert
        id="insertAuthor"
        parameterType="domain.blog.Author"
        flushCache="true"
        statementType="PREPARED"
        keyProperty=""
        keyColumn=""
        useGeneratedKeys=""
        timeout="20">

  <update
          id="updateAuthor"
          parameterType="domain.blog.Author"
          flushCache="true"
          statementType="PREPARED"
          timeout="20">

    <delete
            id="deleteAuthor"
            parameterType="domain.blog.Author"
            flushCache="true"
            statementType="PREPARED"
            timeout="20">
```

| 属性             | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| id               | 命名空间中的唯一标识符，可被用来代表这条语句。               |
| parameterType    | 将要传入语句的参数的完全限定类名或别名。这个属性是可选的，因为 MyBatis 可以通过 TypeHandler 推断出具体传入语句的参数，默认值为 unset。 |
| flushCache       | 将其设置为 true，任何时候只要语句被调用，都会导致本地缓存和二级缓存都会被清空，默认值：true（对应插入、更新和删除语句）。 |
| timeout          | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为 unset（依赖驱动）。 |
| statementType    | STATEMENT，PREPARED 或 CALLABLE 的一个。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
| useGeneratedKeys | （仅对 insert 和 update 有用）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系数据库管理系统的自动递增字段），默认值：false。 |
| keyProperty      | （仅对 insert 和 update 有用）唯一标记一个属性，MyBatis 会通过 getGeneratedKeys 的返回值或者通过 insert 语句的 selectKey 子元素设置它的键值，默认：unset。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| keyColumn        | （仅对 insert 和 update 有用）通过生成的键值设置表中的列名，这个设置仅在某些数据库（像 PostgreSQL）是必须的，当主键列不是表中的第一列的时候需要设置。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| databaseId       | 如果配置了 databaseIdProvider，MyBatis 会加载所有的不带 databaseId 或匹配当前 databaseId 的语句；如果带或者不带的语句都有，则不带的会被忽略。 |

在插入语句里面有一些额外的属性和子元素用来处理主键的生成，而且有多种生成方式。

1. 如果你的数据库支持自动生成主键的字段（MySQL和SQL Server），可以设置`useGeneratedKeys="true"`。
2. 把keyProperty设置到目标属性上。
3. 如果你的数据库还支持多行插入，你也可以传入一个Authors数组或集合，并返回自动生成的主键。

```xml
<insert id="insertAuthor" useGeneratedKeys="true"
        keyProperty="id">
  insert into Author (username, password, email, bio) values
  <foreach item="item" collection="list" separator=",">
    (#{item.username}, #{item.password}, #{item.email}, #{item.bio})
  </foreach>
</insert>
```

selectKey 元素描述如下：

```xml
<selectKey
           keyProperty="id"
           resultType="int"
           order="BEFORE"
           statementType="PREPARED">
```

| 属性          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| keyProperty   | selectKey 语句结果应该被设置的目标属性。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| keyColumn     | 匹配属性的返回结果集中的列名称。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。 |
| resultType    | 结果的类型。MyBatis 通常可以推算出来，但是为了更加确定写上也不会有什么问题。MyBatis 允许任何简单类型用作主键的类型，包括字符串。如果希望作用于多个生成的列，则可以使用一个包含期望属性的 Object 或一个 Map。 |
| order         | 这可以被设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它会首先选择主键，设置 keyProperty 然后执行插入语句。如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 元素 - 这和像 Oracle 的数据库相似，在插入语句内部可能有嵌入索引调用。 |
| statementType | 与前面相同，MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 语句的映射类型，分别代表 PreparedStatement 和 CallableStatement 类型。 |

#### 1.3 sql 元素的属性

这个元素可以被用来定义可重用的 SQL 代码段，可以包含在其他语句中。它可以静态地(在加载阶段)参数化。不同的属性值可以在include实例中有所不同。

```xml
<sql id="userColumns"> ${alias}.id,${alias}.username,${alias}.password
</sql>
```

这个sql片段可以被包含在其他语句中：

```xml
<select id="selectUsers" resultType="map">
  select
  <include refid="userColumns">
    <property name="alias" value="t1"/>
  </include>,
  <include refid="userColumns">
    <property name="alias" value="t2"/>
  </include>
  from some_table t1
  cross join some_table t2
</select>
```



### 二、参数

---

参数可以指定一个特殊的数据类型。

`#{property,javaType=int,jdbcType=NUMERIC}`

- 像 MyBatis 的剩余部分一样，javaType 通常可以从参数对象中来去确定，前提是**只要对象不是一个 HashMap。那么 javaType 应该被确定来保证使用正确类型处理器**。
- **如果 null 被当作值来传递，对于所有可能为空的列，JDBCType 是需要的**。

为了以后定制类型处理方式，可以指定一个特殊的类型处理器类（或别名），比如：

```
#{age,javaType=int,jdbcType=NUMERIC,typeHandler=MyTypeHandler}
```

对于数值类型，还有一个小数保留位数的设置，来确定小数点后保留的位数。

```
#{height,javaType=double,jdbcType=NUMERIC,numericScale=2}
```

**mode 属性允许指定 IN，OUT 或 INOUT 参数**。如果参数为 OUT 或 INOUT，参数对象属性的真实值将会被改变，就像你在获取输出参数时所期望的那样。如果 mode 为 OUT（或 INOUT），而且 jdbcType 为 CURSOR(也就是 Oracle 的 REFCURSOR)，你必须指定一个 resultMap 来映射结果集到参数类型。要注意这里的 javaType 属性是可选的，如果左边的空白是 jdbcType 的 CURSOR 类型，它会自动地被设置为结果集。

```
#{department, mode=OUT, jdbcType=CURSOR, javaType=ResultSet, resultMap=departmentResultMap}
```

MyBatis 也支持很多高级的数据类型，比如结构体，但是当注册 out 参数时你必须告诉它语句类型名称。比如（再次提示，在实际中要像这样不能换行）：

```
#{middleInitial, mode=OUT, jdbcType=STRUCT, jdbcTypeName=MY_TYPE, resultMap=departmentResultMap}
```

尽管所有这些强大的选项很多时候你只简单指定属性名，其他的事情 MyBatis 会自己去推断，最多你需要为可能为空的列名指定 `jdbcType`。

```
#{firstName}
#{middleInitial,jdbcType=VARCHAR}
#{lastName}
```



### 三、字符串替换

---

- 默认情况下，使用 `#{}` 格式的语法会**导致Mybatis创建预处理语句属性并安全地设置值**。
- 有时候你只是想**直接在SQL语句中插入一个不改变的字符串**。比如，像ORDER BY，你可以这样来使用：`ORDER BY ${columnName}`

这里Mybatis不会修改或转义字符串。以这种方式接受从用户输出的内容并提供给语句中不变的字符串是不安全的，会导致潜在的SQL注入攻击，因此要么不允许用户输入这些字段，要么自行转义并检验。



### 四、resultMap元素

---

ResultMap 的设计就是很多复杂语句确实需要描述它们的关系。

#### 4.1 简单映射语句

没有明确的resultMap

```xml
<select id="selectUsers" resultType="map">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

这样一个语句简单作用于**所有列被自动映射到 HashMap 的键上**，这由 `resultType` 属性指定。但是 HashMap 不能很好描述一个领域模型。所以你的应用程序会使用 JavaBeans 或 POJOs(Plain Old Java Objects)来作为领域 模型。MyBatis 对两者都支持。

#### 4.2基于 JavaBean 的规范

```java
public class User {
  private int id;
  private String username;
  private String hashedPassword;

  public int getId() {
    return id;
  }
  public void setId(int id) {
    this.id = id;
  }
  public String getUsername() {
    return username;
  }
  public void setUsername(String username) {
    this.username = username;
  }
  public String getHashedPassword() {
    return hashedPassword;
  }
  public void setHashedPassword(String hashedPassword) {
    this.hashedPassword = hashedPassword;
  }
}
```

上面这个类有 3 个属性：id，username 和 hashedPassword。这些在 select 语句中会精确匹配到列名。这样的一个 JavaBean 可以被映射到结果集，就像映射到 HashMap 一样简单。

```xml
<select id="selectUsers" resultType="com.chance.model.User">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

使用类型别名可以不用输入类的全路径。

```xml
<!-- In mybatis-config.xml file -->
<typeAlias type="com.someapp.model.User" alias="User"/>

<!-- In SQL Mapping XML file -->
<select id="selectUsers" resultType="User">
  select id, username, hashedPassword
  from some_table
  where id = #{id}
</select>
```

在这样的情况下，mybatis会在幕后自动创建一个ReaultMap，基于属性名来映射列到JavaBean的属性上。

#### 4.3 列名没有精确匹配

可以在列名上使用select字句的别名来匹配标签。

```xml
<select id="selectUsers" resultType="User">
  select
  user_id             as "id",
  user_name           as "userName",
  hashed_password     as "hashedPassword"
  from some_table
  where id = #{id}
</select>
```

#### 4.4 解决列名不匹配的另外一种方式

```xml
<resultMap id="userResultMap" type="User">
  <id property="id" column="user_id" />
  <result property="username" column="username"/>
  <result property="password" column="password"/>
</resultMap>
```

引用它的语句使用resultMap属性就行了（注意：如下使用的resultMap属性）

```xml
<select id="selectUsers" resultMap="userResultMap">
  select user_id, user_name, hashed_password
  from some_table
  where id = #{id}
</select>
```

#### 4.5 高级结果映射

```xml
<select id="selectBlogDetails" resultMap="detailedBlogResultMap">
  select
  B.id as blog_id,
  B.title as blog_title,
  B.author_id as blog_author_id,
  A.id as author_id,
  A.username as author_username,
  A.password as author_password,
  A.email as author_email,
  A.bio as author_bio,
  A.favourite_section as author_favourite_section,
  P.id as post_id,
  P.blog_id as post_blog_id,
  P.author_id as post_author_id,
  P.created_on as post_created_on,
  P.section as post_section,
  P.subject as post_subject,
  P.draft as draft,
  P.body as post_body,
  C.id as comment_id,
  C.post_id as comment_post_id,
  C.name as comment_name,
  C.comment as comment_text,
  T.id as tag_id,
  T.name as tag_name
  from Blog B
  left outer join Author A on B.author_id = A.id
  left outer join Post P on B.id = P.blog_id
  left outer join Comment C on P.id = C.post_id
  left outer join Post_Tag PT on PT.post_id = P.id
  left outer join Tag T on PT.tag_id = T.id
  where B.id = #{id}
</select>
```

```xml
<resultMap id="detailedBlogResultMap" type="Blog">
  <constructor>
    <idArg column="blog_id" javaType="int"/>
  </constructor>
  <result property="title" column="blog_title"/>
  <association property="author" javaType="Author">
    <id property="id" column="author_id"/>
    <result property="username" column="author_username"/>
    <result property="password" column="author_password"/>
    <result property="email" column="author_email"/>
    <result property="bio" column="author_bio"/>
    <result property="favouriteSection" column="author_favourite_section"/>
  </association>
  <collection property="posts" ofType="Post">
    <id property="id" column="post_id"/>
    <result property="subject" column="post_subject"/>
    <association property="author" javaType="Author"/>
    <collection property="comments" ofType="Comment">
      <id property="id" column="comment_id"/>
    </collection>
    <collection property="tags" ofType="Tag" >
      <id property="id" column="tag_id"/>
    </collection>
    <discriminator javaType="int" column="draft">
      <case value="1" resultType="DraftPost"/>
    </discriminator>
  </collection>
</resultMap>
```

resultMap元素有很多子元素和一个值得讨论的结构。

- constructor：类在实例化时，用来注入结构到构造方法中
- idArg：ID参数；标记结构作为ID
- arg：注入到构造方法的一个普通结果
- id：ID结果；标记结构作为ID
- result：注入到字段或JavaBean属性的普通结果
- association：一个复杂的类型关联;许多结果将包成这种类型
- collection：复杂类型的集
- discriminator：使用结果值来决定使用哪个结果映射
- case：基于某些值的结果映射

| 属性        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| id          | 此名称空间中的唯一标识符，可用于引用此结果映射。             |
| type        | 一个完全特定的 Java 类名，或者一个类型别名(参见上表中的内置类型别名列表)。 |
| autoMapping | 如果存在，MyBatis 将启用或禁用这个 ResultMap 的自动操作。此属性覆盖全局`autoMappingBehavior`。默认值：未设置的。 |



### 五、支持的jdbcType

---

MyBatis 通过包含的 jdbcType 枚举型，支持下面的 JDBC 类型。

| BIT      | FLOAT   | CHAR        | TIMESTAMP     | OTHER   | UNDEFINED |
| ---------- | --------- | ------------- | --------------- | --------- | ----------- |
| TINYINT  | REAL    | VARCHAR     | BINARY        | BLOG    | NVARCHAR  |
| SMALLINT | DOUBLE  | LONGVARCHAR | VARBINARY     | CLOB    | NCHAR     |
| INTEGER  | NUMERIC | DATE        | LONGVARBINARY | BOOLEAN | NCLOB     |
| BIGINT   | DECIMAL | TIME        | NULL          | CURSOR  | ARRAY     |



### 六、自动映射

**当自动映射查询结果时，会获取 sql 返回的列名并在 java 类中查找相同名字的属性（忽略大小写）**。 这意味着如果发现了 _ID_ 列和 _id_ 属性，Mybatis会将_ID_ 的值赋给 id。

通常数据库列使用大写单词命名，单词间用下划线分隔；而java属性一般遵循驼峰命名法。 为了在这两种命名方式之间启用自动映射，需要**将 mapUnderscoreToCamelCase设置为 true**。

自动映射甚至在特定的 resultmap下也能工作。在这种情况下，对于每一个 result map，所有的 ResultSet 提供的列， 如果没有被手工映射，则将被自动映射。**自动映射处理完毕后手工映射才会被处理**。 在接下来的例子中， id 和 _userName_列将被自动映射， hashed_password 列将根据配置映射。

```xml
<select id="selectUsers" resultMap="userResultMap">
  select
  user_id             as "id",
  user_name           as "userName",
  hashed_password
  from some_table
  where id = #{id}
</select>
```

```xml
<resultMap id="userResultMap" type="User">
  <result property="password" column="hashed_password"/>
</resultMap>
```

有三个自动映射级别:

- NONE - 禁用自动映射，只有手动映射属性才会被设置。
- `PARTIAL` - 将自动映射结果，除了那些嵌套结果映射（连接）内的结果。
- FULL - 自动映射一切

**默认值是 PARTIAL**



### 七、缓存

---

默认情况下是没有开启缓存的，除了局部的 session 缓存，可以增强变现而且处理循环依赖也是必须的。要开启二级缓存,你需要在你的 SQL 映射文件中添加一行:

`<cache/>`

这个简单语句的效果如下:

- 映射语句文件中的所有 select 语句将会被缓存。
- 映射语句文件中的所有 insert,update 和 delete 语句会刷新缓存。
- 缓存会使用 Least Recently Used(LRU,最近最少使用的)算法来收回。
- 根据时间表(比如 no Flush Interval,没有刷新间隔), 缓存不会以任何时间顺序 来刷新。
- 缓存会存储列表集合或对象(无论查询方法返回什么)的 1024 个引用。
- 缓存会被视为是 read/write(可读/可写)的缓存,意味着对象检索不是共享的,而 且可以安全地被调用者修改,而不干扰其他调用者或线程所做的潜在修改。

所有的这些属性都可以通过缓存元素的属性来修改。比如：

```xml
<cache
      eviction="FIFO"
      flushInterval="60000"
      size="512"
      readOnly="true"/>
```

这个更高级的配置创建了一个 FIFO 缓存,并每隔 60 秒刷新,存数结果对象或列表的 512 个引用,而且返回的对象被认为是只读的,因此在不同线程中的调用者之间修改它们会 导致冲突。

可用的收回策略有:

- `LRU` – 最近最少使用的：移除最长时间不被使用的对象。
- FIFO – 先进先出：按对象进入缓存的顺序来移除它们。
- SOFT – 软引用：移除基于垃圾回收器状态和软引用规则的对象。
- WEAK – 弱引用：更积极地移除基于垃圾收集器状态和弱引用规则的对象。

默认的是LRU。

flushInterval (刷新间隔)可以被设置为任意的正整数,而且它们代表一个合理的毫秒 形式的时间段。默认情况是不设置,也就是没有刷新间隔,缓存仅仅调用语句时刷新。

size(引用数目)可以被设置为任意正整数,要记住你缓存的对象数目和你运行环境的 可用内存资源数目。默认值是 1024。

readOnly (只读)属性可以被设置为 true 或 false。只读的缓存会给所有调用者返回缓 存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。可读写的缓存 会返回缓存对象的拷贝(通过序列化) 。这会慢一些,但是安全,因此默认是 false。



### 八、使用自定义缓存

---

可以通过实现你自己的缓存或为其他第三方缓存方案 创建适配器来完全覆盖缓存行为。

```xml
<cache type="com.domain.something.MyCustomCache"/>
```

type 属性指定的类必须实现 org.mybatis.cache.Cache 接口。

```java
public interface Cache {
  String getId();
  int getSize();
  void putObject(Object key, Object value);
  Object getObject(Object key);
  boolean hasKey(Object key);
  Object removeObject(Object key);
  void clear();
}
```

要配置你的缓存, 简单和公有的 JavaBeans 属性来配置你的缓存实现, 而且是通过 cache 元素来传递属性, 比如, 下面代码会在你的缓存实现中调用一个称为 "setCacheFile(String file)" 的方法：

```xml
<cache type="com.domain.something.MyCustomCache">
  <property name="cacheFile" value="/tmp/my-custom-cache.tmp"/>
</cache>
```

你可以使用所有简单类型作为 JavaBeans 的属性,MyBatis 会进行转换。

记得缓存配置和缓存实例是绑定在 SQL 映射文件的命名空间是很重要的。因此，所有在相同命名空间的语句正如绑定的缓存一样。 语句可以修改和缓存交互的方式，或在语句的 语句的基础上使用两种简单的属性来完全排除它们。默认情况下，语句可以这样来配置：

```xml
<select ... flushCache="false" useCache="true"/>
<insert ... flushCache="true"/>
<update ... flushCache="true"/>
<delete ... flushCache="true"/>
```

因为那些是默认的，你明显不能明确地以这种方式来配置一条语句。相反，如果你想改变默认的行为，只能设置 flushCache 和 useCache 属性。比如，在一些情况下你也许想排除从缓存中查询特定语句结果，或者你也许想要一个查询语句来刷新缓存。相似地，你也许有一些更新语句依靠执行而不需要刷新缓存。



### 九、参照缓存

---

在命名空间中共享相同的缓存配置和实例。在这样的情况下你可以使用 cache-ref 元素来引用另外一个缓存。

```xml
<cache-ref namespace="com.someone.application.data.SomeMapper"/>
```
