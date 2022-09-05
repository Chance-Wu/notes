##### 1. PageHelper 5.0.0及以上版本

> 使用的是这个类PageHelper，我们在项目中可以这样写：
>
> ```java
> PageInterceptor pageHelper = new PageInterceptor();
> properties.setProperty(“helperDialect”, “mysql”);
> ```

> ```java
> @Bean(name = "sqlSessionFactory")
> public SqlSessionFactory sqlSessionFactoryBean() {
>     SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
>     bean.setDataSource(dataSource);
>     bean.setTypeAliasesPackage("com.springboot.demo");
>     // 分页插件
>     PageInterceptor pageHelper = new PageInterceptor();
>     Properties properties = new Properties();
>     properties.setProperty("reasonable", "true");
>     properties.setProperty("supportMethodsArguments", "true");
>     properties.setProperty("returnPageInfo", "check");
>     properties.setProperty("params", "count=countSql");
>     properties.setProperty("helperDialect", "mysql");
>     pageHelper.setProperties(properties);
>     // 添加插件
>     bean.setPlugins(new Interceptor[]{pageHelper});
>     // 添加XML目录
>     ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
>     try {      bean.setMapperLocations(resolver.getResources("classpath:/mapper/*.xml"));
>          // 驼峰匹配          bean.getObject().getConfiguration().setMapUnderscoreToCamelCase(true);
>          return bean.getObject();
>         } catch (Exception e) {
>         e.printStackTrace();
>         throw new RuntimeException(e);
>     }
> }
> ```

##### 2. PageHelper5.0.5以下版本

> 使用的是这个类Pagehelper,我们在项目中可以这样写:
>
> ```java
> PageHelper pageHelper = new PageHelper();
> properties.setProperty(“dialect”, “mysql”);
> ```

> ```java
> public SqlSessionFactory sqlSessionFactoryBean() {
>     SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
>     bean.setDataSource(dataSource);
>     bean.setTypeAliasesPackage("com.springboot.demo");
>     // 分页插件
>     PageHelper pageHelper = new PageHelper();
>     Properties properties = new Properties();
>     properties.setProperty("reasonable", "true");
>     properties.setProperty("supportMethodsArguments", "true");
>     properties.setProperty("returnPageInfo", "check");
>     properties.setProperty("params", "count=countSql");
>     properties.setProperty("dialect", "mysql");
>     pageHelper.setProperties(properties);
>     // 添加插件
>     bean.setPlugins(new Interceptor[] { pageHelper });
>     // 添加XML目录
>     ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
>     try {
>         bean.setMapperLocations(resolver.getResources("classpath:/mapper/*.xml"));
>         // 驼峰匹配
>         bean.getObject().getConfiguration().setMapUnderscoreToCamelCase(true);
>         return bean.getObject();
>     } catch (Exception e) {
>         e.printStackTrace();
>         throw new RuntimeException(e);
>     }
> }
> ```

