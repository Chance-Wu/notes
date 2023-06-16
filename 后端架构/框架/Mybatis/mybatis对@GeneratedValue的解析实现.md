### 一、@GeneratedValue注解id生成策略

---

使用范围：方法和属性

```java
@Target({METHOD, FIELD})
@Retention(RUNTIME)
public @interface GeneratedValue {

  /**
   * 主键生成策略
   */
  GenerationType strategy() default AUTO;

  /**
   * 在{@link SequenceGenerator}或{@link TableGenerator}注解中指定的主键生成器的名称。
   * 默认为持久提供程序提供的id生成器
   */
  String generator() default "";
}

```

- **strategy** ：主键生成策略，有`TABLE`，`SEQUENCE`，`IDENTITY`，`AUTO`四种。
- **generator**：主键生成的来源，既由谁来执行生成主键，从哪里获取主键值。下面会讲到mybatis可以配置的值。



### 二、主键生成策略

---

#### 2.1 GenerationType.TABLE

使用一个特定的数据库表格来保存主键，持久化引擎通过关系数据库的一张特定的表格来生成主键，这种策略的好处就是不依赖于外部环境和数据库的具体实现，在不同数据库间可以很容易的进行移植，但由于其不能充分利用数据库的特性，所以不会优先使用。该策略一般与另外一个注解一起使用@TableGenerator，@TableGenerator注解指定了生成主键的表(可以在实体类上指定也可以在主键字段或属性上指定)，然后JPA将会根据注解内容自动生成一张表作为序列表(或使用现有的序列表)。如果不指定序列表，则会生成一张默认的序列表，表中的列名也是自动生成,数据库上会生成一张名为sequence的表(SEQ_NAME,SEQ_COUNT)。序列表一般只包含两个字段：第一个字段是该生成策略的名称，第二个字段是该关系表的最大序号，它会随着数据的插入逐渐累加。类似于

```java
@Id
@GeneratedValue(strategy = GenerationType.TABLE, generator = "roleSeq")
@TableGenerator(name = "roleSeq", allocationSize = 1, table = "seq_table", pkColumnName = "seq_id", valueColumnName = "seq_count")
private Integer id;
```

在以上例子中，roleSeq唯一的标识了该生成器，在@GeneratedValue注解中的generator属性可以根据此标识来声明主键生成器。

#### 2.2 GenerationType.SEQUENCE

在某些数据库中，不支持主键自增长，**比如Oracle，其提供了一种叫做"序列(sequence)"的机制生成主键**。该策略的不足之处正好与TABLE相反，由于只有部分数据库(Oracle，PostgreSQL，DB2)支持序列对象，所以该策略一般不应用于其他数据库。类似的，该策略一般与另外一个注解一起使用@SequenceGenerator，**@SequenceGenerator注解指定了生成主键的序列**。然后会根据注解内容创建一个序列(或使用一个现有的序列)。如果不指定序列，则会自动生成一个序列SEQ_GEN_SEQUENCE。

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "menuSeq")
@SequenceGenerator(name = "menuSeq", initialValue = 1, allocationSize = 1, sequenceName = "MENU_SEQUENCE")
private Integer id;
```

在以上例子中，menuSeq唯一的标识了该生成器，@SequenceGenerator可以理解为将数据库中存在的序列进行了一个映射，在@GeneratedValue注解中的generator属性可以根据此标识来声明主键生成器。

#### 2.3 GenerationType.IDENTITY

此种主键生成策略就是通常所说的主键自增长，数据库在插入数据时，会自动给主键赋值，比如MYSQL可以在创建表时声明"auto_increment" 来指定主键自增长。该策略在大部分数据库中都提供了支持(指定方法或关键字可能不同)，但还是有少数数据库不支持，所以可移植性略差。使用自增长主键生成策略是只需要声明strategy = GenerationType.IDENTITY即可。

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY,generator = "mysql")
private Integer id;
```

**注意：同一张表中自增列最多只能有一列。**

#### 2.4 GenerationType.AUTO

把主键生成策略交给持久化引擎(persistence engine)，持久化引擎会根据数据库在以上三种主键生成策略中选择其中一种。此种主键生成策略比较常用，由于JPA默认的生成策略就是GenerationType.AUTO，所以使用此种策略时。可以显式的指定@GeneratedValue(strategy = GenerationType.AUTO)也可以直接@GeneratedValue。

```java
@GeneratedValue(strategy = GenerationType.AUTO)
private Integer id;

// 或
@GeneratedValue
private Integer id;
```



### 三、tk-mybatis对@GeneratedValue的解析实现

---

```java
/**
 * 处理主键策略
 */
protected void processKeyGenerator(EntityTable entityTable, EntityField field, EntityColumn entityColumn) {
  //KeySql 优先级最高
  if (field.isAnnotationPresent(KeySql.class)) {
    processKeySql(entityTable, entityColumn, field.getAnnotation(KeySql.class));
  } else if (field.isAnnotationPresent(GeneratedValue.class)) {
    //执行 sql - selectKey
    processGeneratedValue(entityTable, entityColumn, field.getAnnotation(GeneratedValue.class));
  }
}

/**
 * 处理 GeneratedValue 注解
 */
protected void processGeneratedValue(EntityTable entityTable, EntityColumn entityColumn, GeneratedValue generatedValue) {
  if ("JDBC".equals(generatedValue.generator())) {
    entityColumn.setIdentity(true);
    entityColumn.setGenerator("JDBC");
    entityTable.setKeyProperties(entityColumn.getProperty());
    entityTable.setKeyColumns(entityColumn.getColumn());
  } else {
    //允许通过generator来设置获取id的sql,例如mysql=CALL IDENTITY(),hsqldb=SELECT SCOPE_IDENTITY()
    //允许通过拦截器参数设置公共的generator
    if (generatedValue.strategy() == GenerationType.IDENTITY) {
      //mysql的自动增长
      entityColumn.setIdentity(true);
      if (!"".equals(generatedValue.generator())) {
        String generator = null;
        IdentityDialect identityDialect = IdentityDialect.getDatabaseDialect(generatedValue.generator());
        if (identityDialect != null) {
          generator = identityDialect.getIdentityRetrievalStatement();
        } else {
          generator = generatedValue.generator();
        }
        entityColumn.setGenerator(generator);
      }
    } else {
      throw new MapperException(entityColumn.getProperty()
                                + " - 该字段@GeneratedValue配置只允许以下几种形式:" +
                                "\n1.useGeneratedKeys的@GeneratedValue(generator=\\\"JDBC\\\")  " +
                                "\n2.类似mysql数据库的@GeneratedValue(strategy=GenerationType.IDENTITY[,generator=\"Mysql\"])");
    }
  }
}
```

由上面的mybatis对@GeneratedValue注解的解析可知：

1. mybatis先解析注解@GeneratedValue中的generator；
2. 如果generator的值不是JDBC时，再判断strategy；
3. 如果类型不是GenerationType.IDENTITY则会抛出异常；
4. 因此strategy在使用mybatis框架时，值必须是GenerationType.IDENTITY；
5. 而generator的取值可以在下面的枚举值中看到，且不区分大小写。

```java
package tk.mybatis.mapper.code;

public enum IdentityDialect {
  DB2("VALUES IDENTITY_VAL_LOCAL()"),
  MYSQL("SELECT LAST_INSERT_ID()"),
  SQLSERVER("SELECT SCOPE_IDENTITY()"),
  CLOUDSCAPE("VALUES IDENTITY_VAL_LOCAL()"),
  DERBY("VALUES IDENTITY_VAL_LOCAL()"),
  HSQLDB("CALL IDENTITY()"),
  SYBASE("SELECT @@IDENTITY"),
  DB2_MF("SELECT IDENTITY_VAL_LOCAL() FROM SYSIBM.SYSDUMMY1"),
  INFORMIX("select dbinfo('sqlca.sqlerrd1') from systables where tabid=1");

  private String identityRetrievalStatement;

  private IdentityDialect(String identityRetrievalStatement) {
    this.identityRetrievalStatement = identityRetrievalStatement;
  }

  public static IdentityDialect getDatabaseDialect(String database) {
    IdentityDialect returnValue = null;
    if ("DB2".equalsIgnoreCase(database)) {
      returnValue = DB2;
    } else if ("MySQL".equalsIgnoreCase(database)) {
      returnValue = MYSQL;
    } else if ("SqlServer".equalsIgnoreCase(database)) {
      returnValue = SQLSERVER;
    } else if ("Cloudscape".equalsIgnoreCase(database)) {
      returnValue = CLOUDSCAPE;
    } else if ("Derby".equalsIgnoreCase(database)) {
      returnValue = DERBY;
    } else if ("HSQLDB".equalsIgnoreCase(database)) {
      returnValue = HSQLDB;
    } else if ("SYBASE".equalsIgnoreCase(database)) {
      returnValue = SYBASE;
    } else if ("DB2_MF".equalsIgnoreCase(database)) {
      returnValue = DB2_MF;
    } else if ("Informix".equalsIgnoreCase(database)) {
      returnValue = INFORMIX;
    }
    return returnValue;
  }

  public String getIdentityRetrievalStatement() {
    return identityRetrievalStatement;
  }
}
```

因此项目集成mybatis使用mysql数据库的主键配置为：

```java
@Id
@GeneratedValue(generator = "mysql", strategy = GenerationType.IDENTITY)
private Long id;
```

- strategy：必须为 GenerationType.IDENTITY；
- generator： mysql大写忽略。