Spring Resource Loader 提供了统一的 getResource 方法，供我们通过资源路径获取外部资源。



### 一、资源接口代表一个资源

---

Resource 是 Spring 中用于表示外部资源的通用接口。Spring 为 Resource 接口提供了以下 6 种实现：

1. UrlResource
2. ClassPathResource
3. FileSystemResource
4. ServletContextResource
5. InputStreamResource
6. ByteArrayResource



### 二、资源加载器

---

用于加载资源，例如类路径或文件资源路径。有如下两种方法：

```java
//Expose the ClassLoader used by this ResourceLoader.
ClassLoader getClassLoader()

//Return a Resource handle for the specified resource location.
Resource getResource(String location)
```

getResource 方法会根据资源路径决定要实例化哪个 Resource 实现。要获取 ResourceLoader 的引用，请实现 `ResourceLoaderAware` 接口。

```java
Resource dataResource = resourceLoader.getResource("file:c:/temp/filesystemdata.txt");
```



### 三、使用ApplicationContext加载资源

---

在 Spring 中，所有应用程序上下文都实现了 ResourceLoader 接口。因此，所有应用程序上下文都可以用于获取资源实例。

要获取 ApplicationContext 的引用，请实现 ApplicationContextAware 接口。

```java
Resource banner = ctx.getResource("file:c:/temp/filesystemdata.txt");
```



### 四、使用ResourceLoaderAware加载资源

---

为了演示下面的各种示例，我在不同位置放置了一个名称匹配的文件，我将展示如何加载它们中的每一个。

CustomResourceLoader.java 编写如下，将加载的资源文件的内容打印到控制台。

```java
/**
 * <p> 自定义资源加载组件 </p>
 *
 * @author chance
 * @date 2023/8/9 14:52
 * @since 1.0
 */
@Component
public class CustomResourceLoader implements ResourceLoaderAware {

  private static final Logger logger = LoggerFactory.getLogger(CustomResourceLoader.class);

  private ResourceLoader resourceLoader;

  @Override
  public void setResourceLoader(ResourceLoader resourceLoader) {
    this.resourceLoader = resourceLoader;
  }

  /**
   * 获取资源文件流
   *
   * @param path
   * @return
   * @throws IOException
   */
  public InputStream getInputStream(String path) throws IOException {
    Resource banner = resourceLoader.getResource(path);
    return banner.getInputStream();
  }

  public void showResourceData(String path) throws IOException {
    Resource banner = resourceLoader.getResource(path);
    InputStream in = banner.getInputStream();
    BufferedReader reader = new BufferedReader(new InputStreamReader(in));
    while (true) {
      String line = reader.readLine();
      if (line == null) {
        break;
      }
      logger.info(line);
    }
    reader.close();
  }
}
```

测试：

```java
ConfigurableApplicationContext applicationContext = SpringApplication.run(SpringbootMybatisApplication.class, args);

CustomResourceLoader customResourceLoader = (CustomResourceLoader) applicationContext.getBean("customResourceLoader");
try {
  customResourceLoader.showResourceData("https://www.clc.plus/fqa/100067.html");
} catch (IOException e) {
  System.out.println(e);
}
```

因为是通过spring的资源加载器来访问资源的，所以自定义的资源加载器必须实现 `ApplicationContextAware` 接口或者 `ResourceLoaderAware` 接口。



### 五、加载外部资源

---

#### 5.1 从应用程序根文件夹加载资源

使用如下模板：

```java
Resource banner = resourceLoader.getResource("file:data.txt");
```



#### 5.2 从类路径加载资源

使用如下模板：

```java
Resource banner = resourceLoader.getResource("classpath:classpathdata.txt");
```



#### 5.3 从文件系统加载资源

使用如下模板：

```java
Resource banner = resourceLoader.getResource("file:c:/temp/filesystemdata.txt");
```



#### 5.4 从URL加载资源

使用如下模板：

```java
Resource banner = resourceLoader.getResource("//clc.plus/readme.txt");
```