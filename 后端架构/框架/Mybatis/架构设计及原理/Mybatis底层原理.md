### 一、反射机制，sql解析替换与JDK Proxy

---

#### 1.1 Mapper接口的工作原理是什么？

接口是不能实例化的。Proxy类提供了静态方法创建动态代理类和实例。

解析sql字符串的核心：

```java
interface UserMapper {
  @Select("SELECT * FROM blog WHERE id = #{id}")
  List<User> selectUserList(Integer id, String name);
}
```

```java
/**
 * 反射机制，sql解析替换，JDK动态代理来实现Mapper接口的方法拦截
 */
public class MybatisTest {

  public static void main(String[] args) {
    UserMapper userMapper = (UserMapper) Proxy.newProxyInstance(MybatisTest.class.getClassLoader(), new Class<?>[]{UserMapper.class}, new InvocationHandler() {
      @Override
      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        Select annotation = method.getAnnotation(Select.class);

        Map<String, Object> nameArgMap = buildMethodArgNameMap(method, args);

        if (null != annotation) {
          String[] value = annotation.value();
          String sql = value[0];
          sql = parseSQL(sql, nameArgMap);
          System.out.println(sql);

          // 获取返回类型去处理结果集
          System.out.println(method.getReturnType());
          System.out.println(method.getGenericReturnType());
        }
        return null;
      }
    });
    userMapper.selectUserList(1, "test");
  }

  /**
   * 解析SQL，进行参数替换
   *
   * @param sql        原sql
   * @param nameArgMap 方法参数map
   */
  public static String parseSQL(String sql, Map<String, Object> nameArgMap) {
    StringBuilder stringBuilder = new StringBuilder();
    int length = sql.length();
    for (int i = 0; i < length; i++) {
      char c = sql.charAt(i);
      if (c == '#') {
        int nextIndex = i + 1;
        char nextChar = sql.charAt(nextIndex);
        if (nextChar != '{') {
          throw new RuntimeException(String.format("这里应该为#{\nsql:%s\nindex:%d}", stringBuilder.toString(), nextIndex));
        }
        // 如果当前字符为'{'，获取到参数名
        StringBuilder argSB = new StringBuilder();
        i = parseSQLArg(argSB, sql, nextIndex);
        String argName = argSB.toString();
        Object argValue = nameArgMap.get(argName);
        stringBuilder.append(argValue.toString());
        continue;
      }
      stringBuilder.append(c);
    }
    return stringBuilder.toString();
  }

  /**
   * 解析sql中的参数
   *
   * @param argSB
   * @param sql
   * @param nextIndex
   */
  private static int parseSQLArg(StringBuilder argSB, String sql, int nextIndex) {
    nextIndex++;
    for (; nextIndex < sql.length(); nextIndex++) {
      char c = sql.charAt(nextIndex);
      if (c != '}') {
        argSB.append(c);
        continue;
      }
      if (c == '}') {
        return nextIndex;
      }
    }
    throw new RuntimeException(String.format("缺少右括号\nindex:%d}", nextIndex));
  }

  /**
   * 构建方法参数 Map
   *
   * @param method
   * @param args
   */
  public static Map<String, Object> buildMethodArgNameMap(Method method, Object[] args) {
    HashMap<String, Object> nameArgMap = Maps.newHashMap();
    Parameter[] parameters = method.getParameters();

    final int[] index = {0};
    Arrays.asList(parameters).forEach(parameter -> {
      String name = parameter.getName();
      nameArgMap.put(name, args[index[0]]);
      index[0]++;
    });
    return nameArgMap;
  }
}
```



### 二、架构设计

---

- 首先需要个**配置类**（配置类里读Mapper接口）
- **Mapper注册中心**（配置类注册到Mapper注册中心）
- **执行器**（拿到mapper之后怎么执行，怎么其解析SQL，怎么封装）
- **StatementHandler**（底层与数据库的交互）
- **结果集Handler**（需要接口向上向下转型，**字段映射处理器**作为基础中间件）
- Spring整合一块



### 三、mybatis源码结构分析

---

| mybatis包                              | 描述                                                         |
| -------------------------------------- | ------------------------------------------------------------ |
| org.apache.ibatis.<br/>	annotations | 包含所有mapper接口中用到的注解（@Param、@Update、@Select、@Delete） |
| `binding`                              | 生成mapper接口的动态代理并进行管理（MapperProxyFactory、MapperProxy、MapperMethod、MapperRegistry） |
| builder                                | 包含Configuration对象所有构建器，主要包括XML、注解2种方式配置解析 |
| cache                                  | 缓存功能实现、包含各种缓存装饰器                             |
| cursor                                 | 实现游标的方式查询数据、游标非常适合处理百万级别的数据查询，通常情况下不适合一次性加载到内存中 |
| `datasource`                           | 数据源（包括jndi数据源、连接池功能）                         |
| exceptions                             | 框架异常（常见异常，TooManyResultsException）                |
| `executor`                             | 核心包，包括SQL语句执行器（主键生成、执行参数解析、执行结果集解析、SQL执行器、缓存执行器） |
| io                                     | 资源文件读取                                                 |
| javassist                              | java字节码编辑库                                             |
| jdbc                                   | JDBC一些操作、SqlRunner SQL执行、ScriptRunner脚本执行（可以执行建库语句） |
| lang                                   | 只有两个注解@UseJava7@UseJAva8，标识哪些可以使用JDK7或8的API |
| logging                                | 日志功能，实现多种日志框架的对接（.jdbc 代理所有功能JDBC 操作，实现了在debug模式下能够输出SQL） |
| `mapping`                              | 配置文件与实体对象的映射功能，Mapper映射、参数映射、结果映射等 |
| ognl                                   |                                                              |
| `parsing`                              | 解析工具包（GenericTokenParser解析#{}${}这种占位符、XPathParser解析XML、PropertyParser） |
| plugin                                 | 拦截功能实现，使用代理模式实现拦截                           |
| reflection                             | 反射器功能，实现元数据编程（通过把Java对象转换成元数据对象MetaObject，然后就可以对元数据对象进行赋值操作，数据库查询结果到Java对象映射就是通过元对象实现） |
| scripting                              | 动态SQL语言实现，借助OGNL表达式，可以扩展自己的语言实现功能  |
| `session`                              | 核心包，主要实现SqlSession功能（SqlSession包含了Mybatis工作的所有Java接口，通过这些接口可以执行SQL命令（insert/delete/update/select）；获取Mapper；管理事务） |
| `transaction`                          | 事务功能实现，包装了数据库连接，处理数据库连接生命周期包括：连接创建，预编译，提交\回滚和关闭 |
| type                                   | 类型处理器，包括所有数据库类型对应Java类型的处理器，如果要实现自己类型处理器就需要实现包下的基础接口 |



### 四、将应用与数据库的交互抽象成一个SqlSession（有状态的会话）

---
