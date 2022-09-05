#### jar包

---

```xml
<dependency>
  <groupId>org.mybatis.spring.boot</groupId>
  <artifactId>mybatis-spring-boot-starter</artifactId>
  <version>2.1.3</version>
</dependency>
```



#### 添加配置

---

```yaml
# 配置数据源
spring:
  datasource:
    driverClassName: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test
    password: sa
    username: sa
```



#### 添加配置类

---

```java
/**
 * 配置自动扫描器，用于扫描自定义的mapper接口，
 * Mybatis会针对这些接口生成代理来调用对应的XML中的SQL
 */
@Configuration
@MapperScan("com.example.springbootdemo.mapper")
public class MyBatisConfig {
}
```



#### 定义实体类型

---

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@EqualsAndHashCode
@Builder
@ApiModel("书籍模型")
public class Book {
  @ApiModelProperty(value = "书籍ID", notes = "书籍ID",example = "1")
  private Integer bookId;
  @ApiModelProperty(value = "书籍页数", notes = "书籍页数",example = "100")
  private Integer pageNum;
  @ApiModelProperty(value = "书籍名称", notes = "书籍名称",example = "Java编程思想")
  private String bookName;
  @ApiModelProperty(value = "书籍类型", notes = "书籍类型",hidden = false)
  private BookType BookType;
  @ApiModelProperty(value = "书籍简介")
  private String bookDesc;
  @ApiModelProperty(value = "书籍价格")
  private Double bookPrice;
  @ApiModelProperty(value = "创建时间",hidden = true)
  private LocalDateTime createTime;
  @ApiModelProperty(value = "修改时间",hidden = true)
  private LocalDateTime modifyTime;
}

public enum BookType {
  TECHNOLOGY,//技术
  LITERARY,//文学
  HISTORY//历史
    ;
}
```



#### 定义mapper接口

---

```java
// 位置为配置文件配置的包路径下
public interface BookRepository {
  int addBook(Book book);
  int updateBook(Book book);
  int deleteBook(int id);
  Book getBook(int id);
  List<Book> getBooks(Book book);
}
```



#### 定义Mapper映射文件

---

```xml
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.springbootdemo.mapper.BookRepository">
  <insert id="addBook" parameterType="Book">
    INSERT INTO BOOK(
    <if test="pageNum != null">
      PAGE_NUM,
    </if>
    <if test="bookType != null">
      BOOK_TYPE,
    </if>
    <if test="bookName != null">
      BOOK_NAME,
    </if>
    <if test="bookDesc != null">
      BOOK_DESC,
    </if>
    <if test="bookPrice != null">
      BOOK_PRICE,
    </if>
    CREATE_TIME,
    MODIFY_TIME)
    VALUES (
    <if test="pageNum != null">
      #{pageNum},
    </if>
    <if test="bookType != null">
      #{bookType},
    </if>
    <if test="bookName != null">
      #{bookName},
    </if>
    <if test="bookDesc != null">
      #{bookDesc},
    </if>
    <if test="bookPrice != null">
      #{bookPrice},
    </if>
    sysdate,sysdate)
  </insert>
  <update id="updateBook" parameterType="Book">
    UPDATE BOOK SET
    <if test="pageNum != null">
      PAGE_NUM = #{pageNum},
    </if>
    <if test="bookType != null">
      BOOK_TYPE = #{bookType},
    </if>
    <if test="bookDesc != null">
      BOOK_DESC = #{bookDesc},
    </if>
    <if test="bookPrice != null">
      BOOK_PRICE = #{bookPrice},
    </if>
    <if test="bookName != null">
      BOOK_NAME = #{bookName},
    </if>
    MODIFY_TIME=sysdate
    WHERE 1=1
    <if test="bookId != null">
      and BOOK_ID = #{bookId}
    </if>
  </update>
  <delete id="deleteBook" parameterType="int">
    delete from BOOK where BOOK_id=#{bookId}
  </delete>
  <select id="getBook" parameterType="int" resultMap="bookResultMap">
    select * from BOOK where BOOK_ID=#{bookId}
  </select>
  <select id="getBooks" resultMap="bookResultMap">
    select * from BOOK WHERE 1=1
    <if test="bookId != null">
      and BOOK_ID = #{bookId}
    </if>
    <if test="pageNum != null">
      and PAGE_NUM = #{pageNum}
    </if>
    <if test="bookType != null">
      and BOOK_TYPE = #{bookType}
    </if>
    <if test="bookDesc != null">
      and BOOK_DESC = #{bookDesc}
    </if>
    <if test="bookPrice != null">
      and BOOK_PRICE = #{bookPrice}
    </if>
    <if test="bookName != null">
      and BOOK_NAME = #{bookName}
    </if>
  </select>
  <resultMap id="bookResultMap" type="Book">
    <id column="BOOK_ID" property="bookId"/>
    <result column="PAGE_NUM" property="pageNum"/>
    <result column="BOOK_NAME" property="bookName"/>
    <result column="BOOK_TYPE" property="bookType"/>
    <result column="BOOK_DESC" property="bookDesc"/>
    <result column="BOOK_PRICE" property="bookPrice"/>
    <result column="CREATE_TIME" property="createTime"/>
    <result column="MODIFY_TIME" property="modifyTime"/>
  </resultMap>
</mapper>
```



#### 添加必要配置

---

```yaml
mybatis:
  mapper-locations: classpath*:/mapper/*.xml #配置xml配置的位置
  type-aliases-package: com.example.springbootdemo.entity #配置实体类型别名
```

这两个也和之前的扫描器注解一样，都是自动配置时未知的，需要手动配置。



#### 定义Service和Controller

---



#### 拦截器分页

---

数据量大时，RowBounds就力不从心了，需要使用分页拦截器实现分页。

PageHelper

```xml
<dependency>
  <groupId>com.github.pagehelper</groupId>
  <artifactId>pagehelper</artifactId>
  <version>1.2.10</version>
</dependency>
```

```java
@Configuration
public class PageHelperConfig {
  @Bean
  public PageHelper pageHelper(){
    PageHelper pageHelper = new PageHelper();
    Properties properties = new Properties();
    properties.setProperty("offsetAsPageNum","true");
    properties.setProperty("rowBoundsWithCount","true");
    properties.setProperty("reasonable","true");
    properties.setProperty("dialect","mysql");    //配置mysql数据库的方言
    pageHelper.setProperties(properties);
    return pageHelper;
  }
}
```

```java
@Service
@Log4j2
public class BookService {

  @Autowired
  private BookRepository bookRepository;

  // 省略多余内容

  public ResponseEntity<PageInfo<Book>> getBooksByPageHelper(int pageId, int pageSize) {
    // 使用PageHelper提供的PageInfo来承载分页信息
    PageHelper.startPage(pageId, pageSize);
    List<Book> books = bookRepository.getBooks(Book.builder().build());
    int totalNum  = bookRepository.count(Book.builder().build());
    PageInfo<Book> page = new PageInfo<>();
    page.setPageNum(pageId);
    page.setPageSize(pageSize);
    page.setSize(totalNum);
    page.setList(books);
    return ResponseEntity.ok(page);
  }
}
```