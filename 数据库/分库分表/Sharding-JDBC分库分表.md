### 1. 概述

#### 1.1 分库分表是什么

小明是一家初创电商平台的开发人员，他负责卖家模块的功能开发，其中涉及店铺、商品的相关业务，涉及如下数据库：

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjfwsr5ug4j30su0kk403.jpg" style="zoom:50%">

> 通过以下SQL能够获取到商品相关的店铺信息、地理区域信息：
>
> ```sql
> SELECT g.*,r.region_name,s.shop_name,s.reputation
> FROM good g
> LEFT JOIN region r ON g.location = r.region_code
> LEFT JOIN shop s ON g.shop_id = s.id;
> ```
>
> 随着公司业务快速发展，数据库中的数据量猛增，访问性能也变慢了。关系型数据库本身比较容易成为系统瓶颈，单机存储容量、连接数、处理能力都有限。==当单表的数据量达到1000W或100G以后==，由于查询维度较多，即使添加从库、优化索引，做很多操作时性能仍下降严重。

> - 方案1：通过提升服务器硬件能力来提高提高数据处理能力，比如增加存储容量、CPU等。这种方案成本很高，并且如果瓶颈在MySQL本身那么提高硬件性能也是有限的。
> - 方案2：把数据分散在不同的数据库中，使得单一数据库的数据量变小来缓解单一数据库的性能问题，从而达到提升数据库性能的目的，如下图：将电商数据库拆分为若干独立的数据库，并且对于大表也拆分为若干小表，通过这种数据库拆分的方法来解决数据库的性能问题。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjfz59terjj30zw0degm8.jpg" style="zoom:50%">

> 分库分表就是为了**解决由于数据量过大而导致数据库性能降低的问题**，==将原来独立的数据库拆分成若干数据库组成，将数据大表拆分成若干数据表组成==，使得单一数据库、单一数据表的数据量变小，从而达到提升数据库性能的目的。

#### 1.2 分库分表的方式

> - 分库：
>   - 垂直分库
>   - 垂直分表
>
> - 分表：
>   - 水平分库
>   - 水平分表

##### 1.2.1 垂直分表

> **垂直分表定义：将一个表按照字段分成多表，每个表存储其中一部分字段。**

> 1. 为了避免IO争抢并减少锁表的几率，查看详情的用户与商品信息浏览互补影响。
> 2. 充分发挥热门数据的操作效率，商品信息的操作的高效率不会被商品描述的低效率所拖累。

> 一般来说，某业务实体中的各个数据项的访问频次是不一样的，部分数据项可能是占用存储空间比较大的BLOB或 是TEXT。例如上例中的商品描述。所以，当表数据量很大时，可以将表按字段切开，将热门字段、冷门字段分开放 置在不同库中，这些库可以放在不同的存储设备上，避免IO争抢。垂直切分带来的性能提升主要集中在热门数据的 操作效率上，而且磁盘争用情况减少。
>
> 通常我们按以下原则进行垂直拆分:
>
> 1. 把不常用的字段单独放在一张表;
> 2. 把text，blob等大字段拆分出来放在附表中; 
> 3. 经常组合查询的列放在一张表中;
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjg4kv9smtj30ww0gktav.jpg" style="zoom:50%">

##### 1.2.2 垂直分库

> **定义：垂直分库是指按照业务将表进行分类，分布到不同的数据库上面，每个库可以放在不同的服务器上，它的核心理念是专库专用。**

> - 解决业务层面的耦合，业务清晰；
> - 能对不同业务的数据进行分级管理、维护、监控、扩展等；
> - 高并发场景下，垂直分库一定程度的提升IO、数据库连接数、降低单机硬件资源的瓶颈；
>
> 垂直分库通过将表按业务分类，然后分布在不同数据库，并且可以将这些数据库部署在不同服务器上，从而达到多个服务器共同分摊压力的效果，但是依然没有解决单表数据量过大的问题。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjg4lor2hpj31as0g4jth.jpg">

> 库内垂直分表只解决了单一表数据量过大的问题，但没有将表分布到不同的服务器上，因此每个 表还是竞争同一个物理机的CPU、内存、网络IO、磁盘。

##### 1.2.3 水平分库

> **定义：水平分库是把同一个表的数据按一定规则拆到不同的数据库中，每个库可以放在不同的服务器上。**

> - 解决了单库大数据，高并发的性能瓶颈。
> - 提高了系统的稳定性及可用性。
>
> 当一个应用难以再细粒度的垂直切分，或切分后数据量行数巨大，存在单库读写、存储性能瓶颈，这时候就需要进行水平分库了，经过水平切分的优化，往往能解决单库存储量及性能瓶颈。但由于同一个表被分配在不同的数据 库，需要额外进行数据操作的路由工作，因此大大提升了系统复杂度。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjg4muy2taj31ao0km0ut.jpg">
>
> 要操作某条数据，先分析这条数据所属的店铺ID。如果店铺ID为双数，将此操作映射至 RRODUCT_DB1(商品库1)；如果店铺ID为单数，将操作映射至RRODUCT_DB2(商品库2)。此操作要访问数据库名称的表达式为**RRODUCT_DB[店铺ID%2 + 1]**。

##### 1.2.4 水平分表

> **定义：水平分表是在同一个数据库内，把同一个表的数据按一定规则拆到多个表中。** 

> - 优化单一表数据量过大而产生的性能问题；
> - 避免IO争抢并减少锁表的几率；
>
> 库内的水平分表，解决了单一表数据量过大的问题，分出来的小表中只包含一部分数据，从而使得单个表的数据量 变小，提高检索性能。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjg4v42vy6j31aw0hwjti.jpg">
>
> 与水平分库的思路类似，不过这次操作的目标是表，商品信息及商品描述被分成了两套表。如果商品ID为双数，将此操作映射至商品信息1表；如果商品ID为单数，将操作映射至商品信息2表。此操作要访问表名称的表达式为**商品信息[商品ID%2 + 1]**。

##### 1.2.5 小结

> **垂直分表**：可以把一个宽表的字段按访问频次、是否是大字段的原则拆分为多个表，这样既能使业务清晰，还能提 升部分性能。拆分后，尽量从业务角度避免联查，否则性能方面将得不偿失。
>
> **垂直分库**：可以把多个表按业务耦合松紧归类，分别存放在不同的库，这些库可以分布在不同服务器，从而使访问 压力被多服务器负载，大大提升性能，同时能提高整体架构的业务清晰度，不同的业务库可根据自身情况定制优化 方案。但是它需要解决跨库带来的所有复杂问题。
>
> **水平分库**：可以把一个表的数据(按数据行)分到多个不同的库，每个库只有这个表的部分数据，这些库可以分布在 不同服务器，从而使访问压力被多服务器负载，大大提升性能。它不仅需要解决跨库带来的所有复杂问题，还要解 决数据路由的问题(数据路由问题后边介绍)。
>
> **水平分表**：可以把一个表的数据(按数据行)分到多个同一个数据库的多张表中，每个表只有这个表的部分数据，这 样做能小幅提升性能，它仅仅作为水平分库的一个补充优化。

> 一般来说，在系统设计阶段就应该根据业务耦合松紧来确定垂直分库，垂直分表方案，在数据量及访问压力不是特别大的情况，首先考虑缓存、读写分离、索引技术等方案。若数据量极大，且持续增长，再考虑水平分库水平分表方案。

#### 1.3 分库分表带来的问题

##### 1.3.1 事务一致性问题

> 由于分库分表把数据分布在不同库甚至不同服务器，不可避免会带来分布式事务问题。

##### 1.3.2 跨节点关联查询

> 在没有分库前，我们检索商品时可以通过以下SQL对店铺信息进行关联查询：
>
> ```sql
> SELECT p.*,r.[地理区域名称],s.[店铺名称],s.[信誉] FROM [商品信息] p
> LEFT JOIN [地理区域] r ON p.[产地] = r.[地理区域编码] LEFT JOIN [店铺信息] s ON p.id = s.[所属店铺] WHERE...ORDER BY...LIMIT...
> ```
>
> 但垂直分库后[商品信息]和[店铺信息]不在一个数据库，甚至不在一台服务器，无法进行关联查询。 *<u>可将原关联查询分为两次查询，第一次查询的结果集中找出关联数据id，然后根据id发起第二次请求得到关联数据，最后将获得到的数据进行拼装</u>*。

##### 1.3.3 跨节点分页、排序函数

> 跨节点多库进行查询时，limit分页、order by排序等问题，就变得比较复杂了。需要先在不同的分片节点中将数据进行排序并返回，然后将不同分片返回的结果集进行汇总和再次排序。
>
> 如，进行水平分库后的商品库，按ID倒序排序分页，取第一页：
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjgj0z9b1hj30o00rugnb.jpg" style="zoom:50%">

##### 1.3.4 主键避重

> 在分库分表环境中，由于表中数据同时存在不同数据库中，主键值平时使用的自增长将无用武之地，某个分区数据库生成的ID无法保证全局唯一。因此需要单独设计全局主键，以避免跨库主键重复问题。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjgjm1csgjj30x60h8jrz.jpg" style="zoom:50%">

##### 1.3.5 公共表

> 实际的应用场景中，参数表、数据字典表等都是数据量较小，变动少，而且属于高频联合查询的依赖表。例子中的地理区域表也属于此类型。
>
> 可以将这类表在每个数据库都保存一份，所有对公共表的更新操作都同时发送到所有分库执行。
>
> 由于分库分表之后，数据被分散在不同的数据库、服务器。因此，对数据的操作也就无法通过常规方式完成，并且 它还带来了一系列的问题。好在，这些问题不是所有都需要我们在应用层面上解决，市面上有很多中间件可供我们选择，其中Sharding-JDBC使用流行度较高。

#### 1.4 Sharding-JDBC介绍

> Sharding-JDBC，它定位为轻量级Java框架，在Java的JDBC层提供的额外服务。它使用客户端直连数据库，以jar包形式提供服务，无需额外部署和依赖，可理解为增强版的JDBC驱动，完全兼容JDBC和各种ORM框架。
>
> Sharding-JDBC的核心功能为==数据分片==和==读写分离==，通过Sharding-JDBC，应用可以透明的使用jdbc访问已经分库分表、读写分离的多个数据源，而不用关心数据源的数量以及数据如何分布。
>
> - 适用于任何基于Java的ORM框架，如: Hibernate, Mybatis, Spring JDBC Template或直接使用JDBC。
> - 基于任何第三方的数据库连接池，如:DBCP, C3P0, BoneCP, Druid, HikariCP等。
> - 支持任意实现JDBC规范的数据库。目前支持MySQL，Oracle，SQLServer和PostgreSQL。

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjgkec08c9j30qi0p4jt1.jpg" style="zoom:40%">

> 上图展示了Sharding-JDBC的工作方式，使用前需要人工对数据库进行分库分表，在应用程序中引入Shardig-JDBC的依赖，**由于Sharding-Jdbc是对 Jdbc驱动的增强，使用Sharding-Jdbc就像使用Jdbc驱动一样，在应用程序中是无需指定具体要操作的分库和分表 的**。

### 2. Sharding-JDBC快速入门

#### 2.1 需求说明

> 使用Sharding-JDBC完成对订单表的水平分表。
>
> 人工创建两张表，t_order_1和t_order_2，这两张表是订单表拆分后的表，通过Sharding-Jdbc向订单表插入数据， 按照一定的分片规则，主键为偶数的进入t_order_1，另一部分数据进入t_order_2，通过Sharding-Jdbc 查询数 据，根据 SQL语句的内容从t_order_1或t_order_2查询数据。

#### 2.2 环境搭建

##### 2.2.1 创建数据库

> - 创建订单库order_db；
> - 在order_db中人工创建两张表t_order_1、t_order_2
>
> ```sql
> CREATE DATABASE `order_db` CHARACTER SET 'utf8' COLLATE 'utf8_general_ci';
> 
> DROP TABLE IF EXISTS `t_order_1`; CREATE TABLE `t_order_1` (
> `order_id` bigint(20) NOT NULL COMMENT '订单id',
> `price` decimal(10, 2) NOT NULL COMMENT '订单价格',
> `user_id` bigint(20) NOT NULL COMMENT '下单用户id',
> `status` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '订单状态', PRIMARY KEY (`order_id`) USING BTREE
> ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
> 
> DROP TABLE IF EXISTS `t_order_2`; CREATE TABLE `t_order_2` (
> `order_id` bigint(20) NOT NULL COMMENT '订单id',
> `price` decimal(10, 2) NOT NULL COMMENT '订单价格',
> `user_id` bigint(20) NOT NULL COMMENT '下单用户id',
> `status` varchar(50) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL COMMENT '订单状态', PRIMARY KEY (`order_id`) USING BTREE
> ) ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
> ```

##### 2.2.2 引入maven依赖

```xml
<dependency>
    <groupId>org.apache.shardingsphere</groupId>
    <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
    <version>4.0.1</version>
</dependency>
```

#### 2.3 编写程序

##### 2.3.1 分片规则配置

> 分片规则配置是sharding-jdbc进行分库分表操作的重要依据，配置内容包括：数据源、主键生成策略、分片策略等。

> 1. 首先==定义数据源==m1，并对m1进行实际的参数配置。
> 2. 指定t_order表的==数据分布情况==，它分布在m1.t_order_1，m1.t_order_2。
> 3. 指定t_order表的==主键生成策略==为SNOWFLAKE（分布式自增算法，保证id全局唯一）。
> 4. 指定t_order==分片策略==，**order_id为偶数的数据落在t_order_1，为奇数的落在t_order_2**，分表策略的表达式为**t_order_$->{order_id % 2 + 1}**

> 在配置文件中配置：
>
> ```yaml
> spring:
>   # sharding数据源配置
>   shardingsphere:
>     datasource:
>       names: m1
>       m1:
>         type: com.alibaba.druid.pool.DruidDataSource
>         driver-class-name: com.mysql.jdbc.Driver
>         url: jdbc:mysql://localhost:3306/order_db?autoReconnect=true&useUnicode=true&characterEncoding=utf8&characterSetResults=utf8&useSSL=false
>         username: root
>         password: 539976
>     sharding:
>       tables:
>         t_order:
>           actual-data-nodes: m1.t_order_$->{1..2}
>           key-generator:
>             column: order_id
>             type: SNOWFLAKE
>           table-strategy:
>             inline:
>               algorithm-expression: t_order_$->{order_id % 2 + 1}
>               sharding-column: order_id
>     props:
>       sql:
>         show: true
> ```
>
> 1. 首先定义数据源m1。
> 2. 指定t_order表的==数据分布情况==，它分布在m1.t_order_1，m1.t_order_2，**m1.t_order_$->{1..2}**
> 3. 指定t_order表的==主键生成策略==为SNOWFLAKE。（分布式自增算法，保证id全局唯一）
> 4. 定义t_order的==分片策略==，order_id为偶数的数据落在t_order_1，为奇数的落在t_order_2，分表策略的表达式为**t_order_$->{ order_id % 2 +1}**

##### 2.3.2 数据操作

```java
public interface OrderMapper {
    /**
    * 新增订单
    * @param price 订单价格 * @param userId 用户id * @param status 订单状态 * @return
    */
    @Insert("insert into t_order(price,user_id,status) value(#{price},#{userId},#{status})")
    int insertOrder(@Param("price") BigDecimal price, @Param("userId")Long userId, @Param("status")String status);
    /**
    * 根据id列表查询多个订单
    * @param orderIds 订单id列表 * @return
    */
    @Select({"<script>" + "select " +
    "*"+
    " from t_order t" +
    " where t.order_id in " +
    "<foreach collection='orderIds' item='id' open='(' separator=',' close=')'>" + " #{id} " +
    "</foreach>"+
    "</script>"})
    List<Map> selectOrderbyIds(@Param("orderIds")List<Long> orderIds); 
}
```

##### 2.3.3 测试

```java
public class OrderMapperTest {

    @Resource
    private OrderMapper orderMapper;

    @Test
    public void testInsertOrder() {
        for (int i = 0; i < 10; i++) {
            orderMapper.insertOrder(new BigDecimal((i + 1) * 5), 1L, "WAIT PAY");
        }
    }

    @Test
    public void testSelectOrderByIds() {
        List<Long> ids = new ArrayList<>();
        ids.add(373771636085620736L);
        ids.add(373771635804602369L);
        List<Map> maps = orderMapper.selectOrderbyIds(ids);
        System.out.println(maps);
    }
}
```

> 执行testInsertOrder：
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjhyrgw62nj324s0fy0uj.jpg">
>
> 通过日志可以发现，order_id为奇数被插入t_order_2，为偶数被插入t_order_1。

> 执行testSelectOrderByIds：<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjhywn1faaj31fu03e3yj.jpg">
>
> 根据传入order_id的奇偶不同，sharding-jdbc分别去不同的表检索数据，达到预期目标。

#### 2.4 流程分析

> 通过日志分析，Shardig-JDBC在拿到用户要执行的sql之后做了什么事：
>
> - 解析sql，获取片键值，在本例中是order_id
> - Sharding-JDBC通过规则配置t_order_$->{order_id % 2 +1}，知道了当order_id为偶数时，应该往t_order_1表插数据，为奇数时，则往t_order_2插数据。
> - 于是Sharding-JDBC根据order_id的值改写sql语句，改写后的SQL语句时真实要执行的SQL语句。
> - 执行改写后的真实语句
> - 将所有真正执行sql的结果进行汇总合并，返回。

#### 2.5 四种集成方式

##### 2.5.1 Spring Boot Yaml配置

##### 2.5.2 Java配置

> 由于采用了配置类所以需要屏蔽原来application.properties文件中spring.shardingsphere开头的配置信息。 还需要在SpringBoot启动类中屏蔽使用spring.shardingsphere配置项的类。

```java
@Configuration
public class ShardingJdbcConfig {

    /**
     * 定义数据源
     */
    public Map<String, DataSource> createDataSourceMap() {
        DruidDataSource dataSource1 = new DruidDataSource();
        dataSource1.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource1.setUrl("jdbc:mysql://localhost:3306/order_db?useUnicode=true");
        dataSource1.setUsername("root");
        dataSource1.setPassword("root");
        Map<String, DataSource> result = new HashMap<>();
        result.put("m1", dataSource1);
        return result;
    }

    /**
     * 定义主键生成策略
     */
    private static KeyGeneratorConfiguration getKeyGeneratorConfiguration() {
        KeyGeneratorConfiguration result = new KeyGeneratorConfiguration("SNOWFLAKE", "order_id");
        return result;
    }

    /**
     * 定义t_order表的分片策略
     */
    public TableRuleConfiguration getOrderTableRuleConfiguration() {
        TableRuleConfiguration result = new TableRuleConfiguration("t_order", "m1.t_order_$‐> {1..2}");
        result.setTableShardingStrategyConfig(new InlineShardingStrategyConfiguration("order_id", "t_order_$‐>{order_id % 2 + 1}"));
        result.setKeyGeneratorConfig(getKeyGeneratorConfiguration());
        return result;
    }

    /**
     * 定义sharding‐Jdbc数据源
     */
    @Bean
    public DataSource getShardingDataSource() throws SQLException {
        ShardingRuleConfiguration shardingRuleConfig = new ShardingRuleConfiguration();
        shardingRuleConfig.getTableRuleConfigs().add(getOrderTableRuleConfiguration());
        //spring.shardingsphere.props.sql.show = true
        Properties properties = new Properties();
        properties.put("sql.show", "true");
        return ShardingDataSourceFactory.createDataSource(createDataSourceMap(), shardingRuleConfig, properties);
    }
}
```

##### 2.5.3 Spring Boot properties配置

##### 2.5.4 Spring命名空间配置（不推荐）

### 3. 执行原理

#### 3.1 基本概念

> **逻辑表**
>
> 水平拆分的数据表的总称。例：订单数据表根据主键尾数拆分为9张表，分别是t_order_0、t_order_1、t_order_2、t_order_3、t_order_4、t_order_5、t_order_6、t_order_7、t_order_8，它们的逻辑表明为t_order。

> **真实表**
>
> 在分片的数据库中真实存在的物理表。t_order_0~8

> **数据节点**
>
> ==数据分片的最小物理单元==。由数据名称和数据表组成，例：ds_0.t_order_0

> **绑定表**
>
> 指==分片规则一致的主表和子表==。例如: <u>*torder表和torder_item表*</u>，均按照 order_id 分片,绑定表之间的分区键完全相同，则此两张表互为绑定表关系。绑定表之间的多表关联查询不会出现笛卡尔积关联，关联查询效率将大大提升。
>
> 举例说明，如果SQL为：
>
> ```sql
> SELECT i.* FROM t_order o JOIN t_order_item i ON o.order_id=i.order_id WHERE o.order_id in (10, 11);
> ```
>
> 在不配置绑定表关系时，假设分片键order_id将数值10路由至第0片，将数值11路由至第1片，那么路由后的SQL应该为4条，它们呈现为笛卡尔积：
>
> ```sql
> SELECT i.* FROM t_order_0 o JOIN t_order_item_0 i ON o.order_id=i.order_id WHERE o.order_id in (10, 11);
> SELECT i.* FROM t_order_0 o JOIN t_order_item_1 i ON o.order_id=i.order_id WHERE o.order_id in (10, 11);
> SELECT i.* FROM t_order_1 o JOIN t_order_item_0 i ON o.order_id=i.order_id WHERE o.order_id in (10, 11);
> SELECT i.* FROM t_order_1 o JOIN t_order_item_1 i ON o.order_id=i.order_id WHERE o.order_id in (10, 11);
> ```
>
> 在配置绑定表关系后，路由的SQL应该为2条：
>
> ```sql
> SELECT i.* FROM t_order_0 o JOIN t_order_item_0 i ON o.order_id=i.order_id WHERE o.order_id in
> (10, 11);
> SELECT i.* FROM t_order_1 o JOIN t_order_item_1 i ON o.order_id=i.order_id WHERE o.order_id in
> (10, 11);
> ```

> **广播表**
>
> 指所有的分片数据源中都存在的表，表结构和表中的数据在每个数据库中均完全一致。适用于数据量不大且需要与海量数据的表进行关联查询的场景，例如:字典表。

> **分片键**
>
> ==用于分片的数据库字段==，是将数据库(表)水平拆分的关键字段。例：将订单表中的订单主键的尾数取模分片，则订单主键为分片字段。 SQL中如果无分片字段，将执行全路由，性能较差。 除了对单分片字段的支持，Sharding- Jdbc也支持根据多个字段进行分片。

> **分片算法**
>
> 通过过分片算法将数据分片，支持通过 = 、 BETWEEN 和 IN 分片。分片算法需要应用方开发者自行实现，可实现的灵活度非常高。包括：精确分片算法、范围分片算法、复合分片算法等。
>
> 例如：`where order_id = ?`将采用精确分片算法，`where order_id in (?,?,?)`将采用精确分片算法，`where order_id BETWEEN ? and ?`将采用范围分片算法，复合分片算法用于分片键有多个复杂情况。

> **分片策略**
>
> ==包含分片键和分片算法==，由于分片算法的独立性，将其独立抽离。真正可用于分片操作的是分片键 + 分片算法，也就是分片策略。内置的分片策略大致可分为==尾数取模==、==哈希==、==范围==、==标签==、==时间==等。常用的使用行表达式配置分片策略，它采用Groovy表达式表示，如：`t_user_$->{u_id % 8}`表示==t_user表根据u_id模8，而分成8张表，表名称为 t_user_0 到 t_user_7==。

> **自增主键生成策略**
>
> 通过在客户端生成自增主键替换以数据库原生自增主键的方式，做到分布式主键无重复。

#### 3.2 SQL解析

> 当Sharding-JDBC接收到一条SQL语句时，会陆续执行
>
> SQL解析=>查询优化=>SQL路由=>SQL改写=>SQL执行=>结果归并
>
> SQL解析过程分为**词法解析**和**语法解析**。
>
> - 词法解析器用于将SQL拆解为不可再分的原子符号，称为**Token**。并根据不同数据库方言所提供的字典，将其归类为*<u>关键字</u>*，*<u>表达式</u>*，*<u>字面量</u>*和*<u>操作符</u>*。
> - 再使用语法解析器将SQL转换为抽象语法树。

> 例如：
>
> ```sql
> SELECT id, name FROM t_user WHERE status = 'ACTIVE' AND age > 18
> ```
>
> 解析为抽象语法树如下：
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gji65lc5n7j311i0mqgn3.jpg" style="zoom:50%">
>
> 为了便于理解，抽象语法树中的关键字的Token用绿色表示，变量的Token用红色表示，晦涩表示需要进一步拆分。
>
> 最后，<u>*通过对抽象语法树的遍历去提炼分片所需的**上下文***</u>，并标记有可能需要**SQL改写**的位置。供分片使用的解析上下文包含：
>
> - 查询选择项（**Select Items**）
> - 表信息（**Table**）
> - 分片条件（**Sharding Condition**）
> - 自增主键信息（**Auto increment Primary Key**）
> - 排序信息（**Order By**）
> - 分组信息（**Group By**）
> - 以及分页信息
>
> SQLStatement: 
> SelectStatement(
> 	super=DQLStatement(
> 		super=AbstractSQLStatement(
> 			type=DQL, 
> 			tables=Tables(tables=[==Table==(name=t_order, alias=Optional.of(t))]),
> 			routeConditions=Conditions(
> 				orCondition=OrCondition(
> 					andConditions=[
> 						AndCondition(
> 							conditions=[
> 								Condition(
> 									column=Column(name=order_id, tableName=t_order), 
> 									operator=IN, 
> 									compareOperator=null, 
> 									positionValueMap={}, 
> 									positionIndexMap={0=0, 1=1}
> 								)
> 							]
> 						)
> 					]
> 				)
> 			), 
> 			encryptConditions=Conditions(orCondition=OrCondition(andConditions=[])), 
> 			sqlTokens=[TableToken(tableName=t_order, quoteCharacter=NONE, schemaNameLength=0)], 
> 			parametersIndex=2, 
> 			logicSQL=select order_id,price,user_id,status from t_order t where t.order_id in(  ? , ? )
> 		)
> 	), 
> 	containStar=false, 
> 	firstSelectItemStartIndex=7, 
> 	selectListStopIndex=35, 
> 	groupByLastIndex=0, 
> 	==itemsa===[
> 	    CommonSelectItem(expression=order_id, alias=Optional.absent()), 
> 	    CommonSelectItem(expression=price, alias=Optional.absent()), 
> 	    CommonSelectItem(expression=user_id, alias=Optional.absent()), 
> 	    CommonSelectItem(expression=status, alias=Optional.absent())
> 	], 
> 	==groupByItems===[], 
> 	==orderByItems===[], 
> 	limit=null, 
> 	subqueryStatement=null, 
> 	subqueryStatements=[], 
> 	subqueryConditions=[]
> )

#### 3.3 SQL路由

> **SQL路由**
>
> ==把针对逻辑表的数据操作映射到对数据结点操作的过程==。
>
> 根据解析上下文匹配数据库和表的分片策略，并生成路由路径。
>
> 对于携带分片键的SQL，
>
> 1. 根据分片键操作符不同可以划分：
>
> - **单片路由（分片键的操作符是=）**
> - **多片路由（分片键的操作符是IN）**
> - **范围路由（分片键的操作符是BETWEEN）**
>
> ==不携带分片键的SQL则采用广播路由==。
>
> 2. 根据分片键进行路由的场景可分为：
>    - **直接路由**
>    - **标准路**
>    - **笛卡尔路由**

##### 3.3.1 标准路由（Sharding-Jdbc最为推荐使用的分片方式）

> 适用范围：不包含关联查询或仅包含绑定表之间关联查询的SQL。 
>
> - 当分片运算符是=====时，路由结果将落入单库(表)；
> - 当分片运算符是==BETWEEN==或==IN==时，则路由结果不一定落入唯一的库(表)，因此一条逻辑SQL最终可能被拆分为多条用于执行的真实SQL。
>
> 举例说明，如果按照order_id的奇数和偶数进行数据分片，一个单表查询的SQL如下：
>
> ```sql
> SELECT * FROM t_order WHERE order_id IN (1,2);
> ```
>
> 那么路由的结果应为：
>
> ```sql
> SELECT * FROM t_order_0 WHERE order_id IN (1,2);
> SELECT * FROM t_order_1 WHERE order_id IN (1,2);
> ```

> 绑定表的关联查询与单表查询复杂度和性能相当。举例说明，如果一个包含绑定表的关联查询的SQL如下：
>
> ```sql
> SELECT * FROM t_order o JOIN t_order_item i ON o.order_id=i.order_id WHERE order_id IN (1,2);
> ```
>
> 那么路由结果应该为：
>
> ```sql
> SELECT * FROM t_order_1 o JOIN t_order_item_1 i ON o.order_id=i.order_id WHERE order_id IN (1,2);
> SELECT * FROM t_order_2 o JOIN t_order_item_2 i ON o.order_id=i.order_id WHERE order_id IN (1,2);
> ```
>
> SQL拆分的数目与单表时一致的。

#### 3.3.2 笛卡尔路由（最复杂的情况）

> 无法根据==绑定表==的关系定位分片规则，因此非绑定表之间的关联查询需要拆解为笛卡尔积组合执行。如果上个示例中的SQL并为配置绑定表关系，那么路由的结果应为：
>
> ```sql
> SELECT * FROM t_order_1 o JOIN t_order_item_1 i ON o.order_id=i.order_id WHERE order_id IN (1, 2);
> SELECT * FROM t_order_1 o JOIN t_order_item_2 i ON o.order_id=i.order_id WHERE order_id IN (1, 2);
> SELECT * FROM t_order_2 o JOIN t_order_item_1 i ON o.order_id=i.order_id WHERE order_id IN (1, 2);
> SELECT * FROM t_order_2 o JOIN t_order_item_2 i ON o.order_id=i.order_id WHERE order_id IN (1, 2);
> ```
>
> 笛卡尔路由查询性能较低，谨慎使用。

#### 3.3.3 全库表路由

> ==对于不携带分片键的SQL，则采取广播路由的方式。==
>
> 根据SQL类型又可以划分为5种类型：
>
> - 全库表路由
> - 全库路由
> - 全实例路由
> - 单播路由
> - 阻断路由
>
> 其中全库表路由用于处理对数据库中与其逻辑表相关的所有真实表的操作， 主要包括不带分片键的DQL(数据查询)和DML(数据操纵)，以及DDL(数据定义)等。例如：
>
> ```sql
> SELECT * FROM t_order WHERE good_prority IN (1, 10);
> ```
>
> 则==会遍历所有数据库中的所有表，逐一匹配逻辑表和真实表名，能够匹配的上则执行==。路由后成为：
>
> ```sql
> SELECT * FROM t_order_1 WHERE good_prority IN (1, 10);
> SELECT * FROM t_order_2 WHERE good_prority IN (1, 10);
> ```
> 

#### 3.4 SQL改写

> 工程师面向逻辑表书写的SQL，并不能够直接在真实的数据库中执行，SQL改写用于将逻辑SQL改写为真实数据库中可以正确执行的SQL。
>
> 如一个简单的例子，若逻辑SQL为：
>
> ```sql
> SELECT order_id FROM t_order WHERE order_id=1;
> ```
>
> 假设该SQL设置分片键order_id，并且order_id=1的情况，将路由至分片表2。那么改写之后的SQL为：
>
> ```sql
> SELECT order_id FROM t_order_1 WHERE order_id=1;
> ```
>
> 再比如，Sharding-JDBC需要在结果归并时获取相应数据，但该数据并未能通过查询的SQL返回。 这种情况主要是针对GROUP BY和ORDER BY。结果归并时，需要根据 GROUP BY 和 ORDER BY 的字段项进行分组和排序，但如果原始SQL的选择项中若并未包含分组项或排序项，则需要对原始SQL进行改写。 先看一下原始SQL中带有结果归并所需信息的场景：
>
> ```sql
> SELECT order_id, user_id FROM t_order ORDER BY user_id;
> ```
>
> 由于使用user_id进行排序，在结果归并中需要能够获取到user_id的数据，而上面的SQL是能够获取到user_id数据的，因此无需补列。 
>
> 如果选择项中不包含结果归并时所需的列，则需要进行补列，如以下SQL：
>
> ```sql
>  SELECT order_id FROM t_order ORDER BY user_id;
> ```
>
> 由于原始SQL中并不包含结果归并时所需的列，则需要进行补列，如以下SQL：
>
> ```sql
> SELECT order_id, user_id AS ORDER_BY_DERIVED_0 FROM t_order ORDER BY user_id;
> ```

#### 3.5 SQL执行

> Sharding-JDBC采用一套自动化的执行引擎，负责将路由和改写完成之后的真实SQL安全且高效发送到底层数据源执行。它不是简单地将SQL通过JDBC直接发送至数据源执行；也并非直接将执行请求放入线程池去并发执行。它更关注平衡数据源连接创建以及内存占用所产生的消耗，以及最大限度地合理利用并发等问题。执行引擎的目标是自动化的平衡资源控制与执行效率，他能在以下两种模式自适应切换：
>
> **（1）内存限制模式**
>
> 使用前提是，*<u>Sharding-JDBC对一次操作所耗费的数据库连接数量不做限制</u>*。 如果实际执行的SQL需要对 某数据库实例中的200张表做操作，则对每张表创建一个新的数据库连接，并通过多线程的方式并发处理，以达成 执行效率最大化。
>
> **（2）连接限制模式**
>
> 使用前提是，*<u>Sharding-JDBC严格控制对一次操作所耗费的数据库连接数量</u>*。如果实际执行的SQL需要对某数据库实例中的200张表做操作，那么只会创建唯一的数据库连接，并对其200张表串行处理。如果一次操作中 的分片散落在不同的数据库，仍然采用多线程处理对不同库的操作，但每个库的每次操作仍然只创建一个唯一的数据库连接。
>
> 内存限制模式适用于OLAP操作，可以通过放宽对数据库连接的限制提升系统吞吐量; 连接限制模式适用于OLTP操作，OLTP通常带有分片键，会路由到单一的分片，因此严格控制数据库连接，以保证在线系统数据库资源能够被 更多的应用所使用，是明智的选择。

#### 3.6 结果归并

> 将从各个数据节点获取的多数据结果集，组合成为一个结果集并正确的返回至请求客户端，称为结果归并。
>
> Sharding-JDBC支持的结果归并从功能上可分为**遍历**、**排序**、**分组**、**分页**和**聚合**5种类型，它们是组合而非互斥的关系。
>
> 归并引擎的整体结构划分如下图：
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjiskgckmzj31620n2415.jpg" style="zoom:50%">
>
> 结果归并从结构划分可分为流式归并、内存归并和装饰归并。流式归并和内存归并时互斥的，装饰者归并可以在流式归并和内存归并之上做进一步的处理。
>
> ==（1）内存归并==很容易理解，它是将所有分片结果集的数据都遍历big存储在内存中，再通过统一的分组、排序以及聚合等计算之后，再将其封装成为逐条访问的数据结果集返回。
>
> ==（2）流式归并==是指每一次从数据库结果集中获取到的数据，都能够通过游标逐条获取的方式返回正确的单条数据，它与数据库原生的返回结果集的方式最为契合。

> 下边举例说明排序归并的过程，如下图是一个通过分数进行排序的示例图，它采用流式归并方式。 图中展示了3张表返回的数据结果集，每个数据结果集已经根据分数排序完毕，但是3个数据结果集之间是无序的。 将3个数据结果集的当前游标指向的数据值进行排序，并放入优先级队列，t_score_0的第一个数据值最大，t_score_2的第一个数据值次之，t_score_1的第一个数据值最小，因此优先级队列根据t_score_0，t_score_2和t_score_1的方式排序队列。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjistrwnw1j315i0l8t9s.jpg" style="zoom:50%">
>
> 下图则展现了进行next调用的时候，排序归并是如何进行的。 通过图中我们可以看到，当进行第一次next调用 时，排在队列首位的t_score_0将会被弹出队列，并且将当前游标指向的数据值(也就是100)返回至查询客户端， 并且将游标下移一位之后，重新放入优先级队列。 而优先级队列也会根据t_score_0的当前数据结果集指向游标的数据值(这里是90)进行排序，根据当前数值，t_score_0排列在队列的最后一位。 之前队列中排名第二的 t_score_2的数据结果集则自动排在了队列首位。
>
> 在进行第二次next时，只需要将目前排列在队列首位的t_score_2弹出队列，并且将其数据结果集游标指向的值返回至客户端，并下移游标，继续加入队列排队，以此类推。 当一个结果集中已经没有数据了，则无需再次加入队列。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjiswgu1xej311a0qg408.jpg" style="zoom:50%">
>
> 可以看到，对于每个数据结果集中的数据有序，而多数据结果集整体无序的情况下，Sharding-JDBC无需将所有的 数据都加载至内存即可排序。 它使用的是流式归并的方式，每次next仅获取唯一正确的一条数据，极大的节省了 内存的消耗。

> ==（3）装饰者归并==是对所有的结果集归并进行统一的功能增强，比如归并时需要聚合SUM前，在进行聚合计算前，都会通过内存归并或流式归并查询出结果集。因此，聚合归并是在之前介绍的归并类型之上追加的归并能力，即装饰者模式。

#### 3.7 总结

> - **基础概念**：逻辑表，真实表，数据节点，绑定表，广播表，分片键，分片算法，分片策略，主键生成策略。
> - **核心功能**：数据分片，读写分离。
> - **执行流程**:：SQL解析 => 查询优化 => SQL路由 => SQL改写 => SQL执行 => 结果归并

### 4. 水平分表

> 水平分表是在同一个数据库内，*<u>把同一个表的数据按一定规则拆到多个表中</u>*。

##### 5. 水平分库

> 水平分库是*<u>把同一个表的数据按一定规则拆到不同的数据库中，每个库可以放在不同的服务器上。</u>*
>
> 接下来继续对快速入门中的例子进行完善：

> （1）将原有order_db库拆分为order_db_1、order_db_2
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjitc5etn5j30e80ecmxh.jpg" style="zoom:50%">
>
> （2）分片规则修改
>
> 由于数据库拆分了两个，这里需要配置两个数据源。
>
> 分库需要配置分库的策略，通过分库策略实现数据操作针对分库的数据进行操作。
>
> ```yaml
> spring:
>   profiles:
>     active: dev
>   application:
>     name: demo
>   main:
>     allow-bean-definition-overriding: true
>   # sharding数据源配置
>   shardingsphere:
>     datasource:
>       names: m1,m2
>       m1:
>         type: com.alibaba.druid.pool.DruidDataSource
>         driver-class-name: com.mysql.jdbc.Driver
>         url: jdbc:mysql://localhost:3306/order_db_1?autoReconnect=true&useUnicode=true&characterEncoding=utf8&characterSetResults=utf8&useSSL=false
>         username: root
>         password: 539976
>       m2:
>         type: com.alibaba.druid.pool.DruidDataSource
>         driver-class-name: com.mysql.jdbc.Driver
>         url: jdbc:mysql://localhost:3306/order_db_2?autoReconnect=true&useUnicode=true&characterEncoding=utf8&characterSetResults=utf8&useSSL=false
>         username: root
>         password: 539976
>     sharding:
>       tables:
>         t_order:
>           actual-data-nodes: m$->{1..2}.t_order_$->{1..2}
>           key-generator:
>             column: order_id
>             type: SNOWFLAKE
>           # 分库策略
>           database-strategy:
>             inline:
>               sharding-column: user_id
>               algorithm-expression: m$->{user_id % 2 + 1}
>           # 分表策略
>           table-strategy:
>             inline:
>               sharding-column: order_id
>               algorithm-expression: t_order_$->{order_id % 2 + 1}
>     props:
>       sql:
>         show: true
> ```
>
> 分库策略定义方式如下：
>
> ```properties
> # 分库策略，如何将一个逻辑表映射到多个数据源
> spring.shardingsphere.sharding.tables.<逻辑表名>.database-strategy.<分片策略>.<分片策略属性名>=分片策略属性值
> ```

> 支持一下几种分片策略：（分库、分表，策略基本一样）
>
> - **standard：标准分片策略**，对应StandardShardingStrategy。提供对SQL语句中的=, IN和BETWEEN AND的分片操作支持。StandardShardingStrategy只支持单分片键，提供PreciseShardingAlgorithm和RangeShardingAlgorithm两个分片算法。*<u>PreciseShardingAlgorithm是必选的</u>*，用于处理=和IN的分片。 RangeShardingAlgorithm是可选的，用于处理BETWEEN AND分片，如果不配置 RangeShardingAlgorithm，SQL中的BETWEEN AND将按照全库路由处理。
> - **complex：符合分片策略**，对应ComplexShardingStrategy。复合分片策略。提供对SQL语句中的=, IN和 BETWEEN AND的分片操作支持。ComplexShardingStrategy支持多分片键，由于多分片键之间的关系复杂，因此并未进行过多的封装，而是直接将分片键值组合以及分片操作符透传至分片算法，完全由应用开发 者实现，提供最大的灵活度。
> - **inline：行表达式分片策略**，对应InlineShardingStrategy。使用Groovy的表达式，提供对SQL语句中的=和 IN的分片操作支持，只支持单分片键。对于简单的分片算法，可以通过简单的配置使用，从而避免繁琐的Java 代码开发，如: t_user_$->{u_id % 8} 表示t_user表根据u_id模8，而分成8张表，表名称为 t_user_0 到t_user_7。
> - **hint：Hint分片策略**，对应HintShardingStrategy。通过Hint而非SQL解析的方式分片的策略。对于分片字段 非SQL决定，而由其他外置条件决定的场景，可使用SQL Hint灵活的注入分片字段。例:内部系统，按照员工 登录主键分库，而数据库中并无此字段。SQL Hint支持通过Java API和SQL注释(待实现)两种方式使用。
> - **none:不分片策略**，对应NoneShardingStrategy。不分片的策略。

> （3）插入测试
>
> ```java
> @Test
> public void testInsertOrder() {
>     for (int i = 0; i < 10; i++) {
>         orderMapper.insertOrder(new BigDecimal((i + 1) * 5), 1L, "WAIT_PAY");
>     }
>     for (int i = 0; i < 10; i++) {
>         orderMapper.insertOrder(new BigDecimal((i + 1) * 10), 2L, "WAIT_PAY");
>     }
> }
> ```
>
> 可以看出，根据user_id的奇偶不同，数据分别落在不同数据源。
>
> （4）查询测试
>
> ```java
> @Test
> public void testSelectOrderByIds() {
>     List<Long> ids = new ArrayList<>();
>     ids.add(521259158473801728L);
>     ids.add(521259158356361216L);
>     List<Map> maps = orderMapper.selectOrderByIds(ids);
>     System.out.println(maps);
> }
> ```
>
> sharding-jdbc将sql路由到m1和m2：
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjiufvzveaj31fw068aa5.jpg" style="zoom:50%">
>
> 问题分析：由于查询语句中并没有使用分片键user_id，所以sharding-jdbc将广播路由到每个数据结点。
>
> 下边我们在sql中添加分片键进行查询。
>
> ```xml
> <select id="selectOrderByUserAndIds" resultType="java.util.Map">
>     select order_id,price,user_id,status from t_order t where t.order_id in
>     <foreach collection='orderIds' item='id' open='(' separator=',' close=')'>#{id}</foreach>
>     and t.user_id = #{userId}
> </select>
> ```
>
> 查询条件user_id为1，根据分片策略m$->{user_id % 2 + 1}计算得出m2，此sharding-jdbc将sql路由到m2。

### 6.垂直分库

> *<u>指按照业务将表进行分类，分布到不同的数据库上面</u>*，每个库可以放在不同的服务器上，它的核心理念是专库专用。

> （1）创建数据库user_db
>
> ```yaml
> spring:
> 	shardingsphere:
> 		datasource:
>             m0:
>             	type: com.alibaba.druid.pool.DruidDataSource
>             	driver-class-name: com.mysql.jdbc.Driver
>             	url: jdbc:mysql://localhost:3306/user_db?autoReconnect=true&useUnicode=true&characterEncoding=utf8&characterSetResults=utf8&useSSL=false
>             	username: root
>             	password: 539976
> ```
>
> 
>
> （2）新增m0数据源，对应user_db；t_user分表策略，固定分配至m0的t_user真实表
>
> ```yaml
> spring:
> 	shardingsphere:
> 		sharding:
>             t_user:
>                 actual-data-nodes: m$->{0}.t_user
>                 table-strategy:
>                     inline:
>                         sharding-column: user_id
>                         algorithm-expression: t_user
> ```

### 7. 公共表

> 公共表属于系统中数据量较小，变动少，而且属于高频联合查询的依赖表。参数表、数据字典表等属于此类型。可以将这类表在每个数据库都保存一份，所有更新操作都同时发送到所有分库执行。接下来看一下如何使用 Sharding-JDBC实现公共表。
>
> （1）创建数据库
>
> 分别在user_db、order_db_1、order_db_2中创建t_dict表。
>
> （2）在Sharding-JDBC规则中修改。
>
> ```yaml
> # 指定t_dict为公共表
> spring:
> 	shardingsphere:
> 		sharding:
> 			broadcast‐tables: t_dict
> ```
>
> （3）字典关联查询测试
>
> ```xml
> <select id="selectUserInfoByIds" resultType="java.util.Map">
>     select * from t_user t ,t_dict b
>     where t.user_type = b.code and t.user_id in
>     <foreach collection='userIds' item='id' open='(' separator=',' close=')'>#{id}</foreach>
> </select>
> ```

### 8. 读写分离

#### 8.1 理解读写分离

> 面对日益增加的系统访问量，数据库的吞吐量面临着巨大瓶颈。对于同一时刻有大量并发读操作和较少写操作类型的应用系统来说，将数据库拆分为主库和从库，主库负责处理事务性的增删改操作，从库负责处理查询操作，能够有效的避免由数据更新导致的行锁，使得整个系统的查询性能得到极大的改善。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjj1be5mzdj30a208mglr.jpg" style="zoom:60%">
>
> 通过一主多从的配置方式，可以将查询请求均匀的分散到多个数据副本，能够进一步的提升系统的处理能力。 使用多主多从的方式，不但能够提升系统的吞吐量，还能够提升系统的可用性，可以达到在任何一个数据库宕机，甚至 磁盘物理损坏的情况下仍然不影响系统的正常运行。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjj1r72jtzj308i07mglr.jpg" style="zoom:60%">
>
> 读写分离的数据节点中的数据内容是一致的，而水平分片的每个数据节点的数据内容却并不相同。将水平分片和读 写分离联合使用，能够更加有效的提升系统的性能。
>
> *<u>Sharding-JDBC读写分离则是根据SQL语义的分析，将读操作和写操作分别路由至主库与从库</u>*。它提供透明化读写分离，让使用方尽量像使用一个数据库一样使用主从数据库集群。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjj1t6th3aj30ea0h6js1.jpg" style="zoom:50%">
>
> Sharding-JDBC提供**一主多从**的读写分离配置，可独立使用，也可配合分库分表使用，同一线程且同一数据库连接内，如有写入操作，以后的读操作均从主库读取，用于保证数据一致性。==Sharding-JDBC不提供主从数据库的数据同步功能，需要采用其他机制支持==。
>
> <img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjj1vdly6bj316o0mw0uv.jpg" style="zoom:60%">
>
> 



架构图

<img src="https://tva1.sinaimg.cn/large/007S8ZIlgy1gjflelhyggj30go0bzgm7.jpg" style="zoom:80%">




