> 结果映射为map不显示null值字段。

```properties
#mybatis返回类型为map返回字段值为null的字段
mybatis.configuration.call-setters-on-nulls=true
```



> 单数据源转多数据源后 yml配置文件里的mybatis配置会失效。

```java
@Configuration
@MapperScan(basePackages = {"扫包路径"}, sqlSessionTemplateRef = "mysqlSqlSessionTemplate")
public class MySqlDBConfig {

  @Bean(name = "mysqlDataSource")
  public DataSource dataSourceCar(@Qualifier("mysqlProperty") DruidXADataSource druidXADataSource) {
    AtomikosDataSourceBean xaDataSource = new AtomikosDataSourceBean();
    xaDataSource.setXaDataSource(druidXADataSource);
    xaDataSource.setUniqueResourceName("mysqlDataSource");
    return xaDataSource;
  }

  @Bean(name = "mysqlSqlSessionFactory")
  public SqlSessionFactory sqlSessionFactory(@Qualifier("mysqlDataSource") DataSource dataSource)
    throws Exception {
    SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
    bean.setDataSource(dataSource);
    bean.setMapperLocations(new PathMatchingResourcePatternResolver().getResources("classpath:mapper/dws/*Mapper.xml"));//扫描指定目录的xml
    //此处创建一个Configuration 注意包不要引错了
    org.apache.ibatis.session.Configuration configuration=new org.apache.ibatis.session.Configuration();
    //配置日志实现
    configuration.setLogImpl(StdOutImpl.class);
    //此处可以添加其他mybatis配置 例如转驼峰命名
    configuration.setMapUnderscoreToCamelCase(true);
    //bena工厂装载上面配置的Configuration 
    bean.setConfiguration(configuration);
    return bean.getObject();
  }

  @Bean(name = "mysqlSqlSessionTemplate")
  public SqlSessionTemplate sqlSessionTemplate(
    @Qualifier("mysqlSqlSessionFactory") SqlSessionFactory sqlSessionFactory) throws Exception {
    return new SqlSessionTemplate(sqlSessionFactory);
  }

}
```

