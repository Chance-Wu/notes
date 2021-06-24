> 场景：工具类中需要注入配置文件的配置项到一个static修饰的变量中。



#### 1. 配置文件中添加配置项

---

```yaml
elasticSearch-config:
	url: http://172.18.208.141:9200
```



#### 2. 通过@Value()将配置项注入到static修饰的属性中

---

> 注意：==工具类上一定要加上@Component注解将该类添加到上下文中。==

常规的取值方式：

```java
@Component
public class EsUtils {

  @Value("${elasticSearch-config.url}")
  private static String elasticSearchUrl;
}
```

>方式一：通过set()的方式注入
>
>```java
>@Component
>public class EsUtils {
>
>  private static String elasticSearchUrl;
>  
>  @Value("${elasticSearch-config.url}")
>  public void setElasticSearchUrl(String url) {
>    EsUtils.elasticSearchUrl = url;
>  }
>}

>方式二：通过中间变量
>
>```java
>@Component
>public class EsUtils {
>
>  private static String value; // 中间变量
>  
>  @Value("${elasticSearch-config.url}")
>  private String elasticSearchUrl;
>  
>  @PostConstruct
>  public void init() {
>    value = elasticSearchUrl;
>  }
>}
>```

