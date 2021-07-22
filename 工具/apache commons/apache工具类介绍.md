#### 1. org.apache.commons.io.IOUtils

---

Apache Commons IO库包含实用程序类、流实现、文件过滤器、文件比较器、端转换类等等。

```xml
<dependency>
  <groupId>commons-io</groupId>
  <artifactId>commons-io</artifactId>
  <version>2.11.0</version>
</dependency>
```

closeQuietly：关闭一个IO流、socket、或者selector且不抛出异常，通常放在finally块

toString：转换IO流、 Uri、 byte[]为String

copy：IO流数据复制，从输入流写到输出流中，最大支持2GB

toByteArray：从输入流、URI获取byte[]

write：把字节. 字符等写入输出流

toInputStream：把字符转换为输入流

readLines：从输入流中读取多行数据，返回List<String>

copyLarge：同copy，支持2GB以上数据的复制

lineIterator：从输入流返回一个迭代器，根据参数要求读取的数据量，全部读取，如果数据不够，则失败



#### 2. org.apache.commons.io.FileUtils

---

```xml
<dependency>
  <groupId>commons-io</groupId>
  <artifactId>commons-io</artifactId>
  <version>2.11.0</version>
</dependency>
```

deleteDirectory：删除文件夹

readFileToString：以字符形式读取文件内容

deleteQueitly：删除文件或文件夹且不会抛出异常

copyFile：复制文件

writeStringToFile：把字符写到目标文件，如果文件不存在，则创建

forceMkdir：强制创建文件夹，如果该文件夹父级目录不存在，则创建父级

write：把字符写到指定文件中

listFiles：列举某个目录下的文件(根据过滤器)

copyDirectory：复制文件夹

forceDelete：强制删除文件



#### 3. org.apache.commons.lang.StringUtils

---

Commons Lang，一个用于Java中的类的Java实用程序类的包。

```xml
<dependency>
  <groupId>commons-lang</groupId>
  <artifactId>commons-lang</artifactId>
  <version>2.6</version>
</dependency>
```

isBlank：字符串是否为空 (trim后判断)

isEmpty：字符串是否为空 (不trim并判断)

equals：字符串是否相等

join：合并数组为单一字符串，可传分隔符

split：分割字符串

EMPTY：返回空字符串

trimToNull：trim后为空字符串则转换为null

replace：替换字符串



#### 4. org.apache.http.util.EntityUtils

---

Apache HttpComponents客户端。

```xml
<dependency>
  <groupId>org.apache.httpcomponents</groupId>
  <artifactId>httpclient</artifactId>
  <version>4.5.13</version>
</dependency>
```

toString：把Entity转换为字符串

consume：确保Entity中的内容全部被消费。可以看到源码里又一次消费了Entity的内容，假如用户没有消费，那调用Entity时候将会把它消费掉

toByteArray：把Entity转换为字节流

consumeQuietly：和consume一样，但不抛异常

getContentCharset：获取内容的编码



#### 5. org.apache.commons.lang3.StringUtils

---

Apache Commons Lang，一个用于Java中的类的Java实用程序类的包。

```xml
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-lang3</artifactId>
  <version>3.12.0</version>
</dependency>
```

isBlank：字符串是否为空 (trim后判断)

isEmpty：字符串是否为空 (不trim并判断)

equals：字符串是否相等

join：合并数组为单一字符串，可传分隔符

split：分割字符串

EMPTY：返回空字符串

replace：替换字符串

capitalize：首字符大写



#### 6. org.apache.commons.io.FilenameUtils

---

```xml
<dependency>
  <groupId>commons-io</groupId>
  <artifactId>commons-io</artifactId>
  <version>2.11.0</version>
</dependency>
```

getExtension：返回文件后缀名

getBaseName：返回文件名，不包含后缀名

getName：返回文件全名

concat：按命令行风格组合文件路径(详见方法注释)

removeExtension：删除后缀名

normalize：使路径正常化

wildcardMatch：匹配通配符

seperatorToUnix：路径分隔符改成unix系统格式的，即/

getFullPath：获取文件路径，不包括文件名

isExtension：检查文件后缀名是不是传入参数(List<String>)中的一个



#### 7. org.apache.commons.lang.ArrayUtils

---

```xml
<dependency>
  <groupId>commons-lang</groupId>
  <artifactId>commons-lang</artifactId>
  <version>2.6</version>
</dependency>
```

contains：是否包含某字符串

addAll：添加整个数组

clone：克隆一个数组

isEmpty：是否空数组

add：向数组添加元素

subarray：截取数组

indexOf：查找某个元素的下标

isEquals：比较数组是否相等

toObject：基础类型数据数组转换为对应的Object数组



#### 8. org.apache.commons.lang.StringEscapeUtils

---

参考十五：org.apache.commons.lang3.StringEscapeUtils



#### 9. org.apache.http.client.utils.URLEncodedUtils

---

```xml
<dependency>
  <groupId>org.apache.httpcomponents</groupId>
  <artifactId>httpclient</artifactId>
  <version>4.5.13</version>
</dependency>
```

format：格式化参数，返回一个HTTP POST或者HTTP PUT可用application/x-www-form-urlencoded字符串

parse：把String或者URI等转换为List<NameValuePair>



#### 10. org.apache.commons.codec.digest.DigestUtils

---

Apache Commons Codec包包含各种格式的简单编码器和解码器，如Base64和十六进制。除了这些广泛使用的编码器和解码器之外，codec包还维护一个语音编码实用程序集合。

```xml
<dependency>
  <groupId>commons-codec</groupId>
  <artifactId>commons-codec</artifactId>
  <version>1.15</version>
</dependency>
```

md5Hex：MD5加密，返回32位字符串

sha1Hex：SHA-1加密

sha256Hex：SHA-256加密

sha512Hex：SHA-512加密

md5：MD5加密，返回16位字符串



#### 11. org.apache.commons.collections.CollectionUtils

---

扩展和增强Java集合框架的类型。

```xml
<dependency>
  <groupId>commons-collections</groupId>
  <artifactId>commons-collections</artifactId>
  <version>3.2.2</version>
</dependency>
```

isEmpty：是否为空

select：根据条件筛选集合元素

transform：根据指定方法处理集合元素，类似List的map()

filter：过滤元素，雷瑟List的filter()

find：基本和select一样

collect：和transform 差不多一样，但是返回新数组

forAllDo：调用每个元素的指定方法

isEqualCollection：判断两个集合是否一致



#### 12. org.apache.commons.lang3.ArrayUtils

---

```xml
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-lang3</artifactId>
  <version>3.12.0</version>
</dependency>
```

contains：是否包含某个字符串

addAll：添加整个数组

clone：克隆一个数组

isEmpty：是否空数组

add：向数组添加元素

subarray：截取数组

indexOf：查找某个元素的下标

isEquals：比较数组是否相等

toObject：基础类型数据数组转换为对应的Object数组



#### 14. org.apache.commons.beanutils.PropertyUtils

---

Apache Commons BeanUtils提供了一个关于反射和自省的易用但灵活的包装器。

```xml
<dependency>
  <groupId>commons-beanutils</groupId>
  <artifactId>commons-beanutils</artifactId>
  <version>1.9.4</version>
</dependency>
```

getProperty：获取对象属性值

setProperty：设置对象属性值

getPropertyDiscriptor：获取属性描述器

isReadable：检查属性是否可访问

copyProperties：复制属性值，从一个对象到另一个对象

getPropertyDiscriptors：获取所有属性描述器

isWriteable：检查属性是否可写

getPropertyType：获取对象属性类型



#### 15. org.apache.commons.lang3.StringEscapeUtils

---

```xml
<dependency>
  <groupId>org.apache.commons</groupId>
  <artifactId>commons-lang3</artifactId>
  <version>3.12.0</version>
</dependency>
```

unescapeHtml4：转义html

escapeHtml4：反转义html

escapeXml：转义xml

unescapeXml：反转义xml

escapeJava：转义unicode编码

escapeEcmaScript：转义EcmaScript字符

unescapeJava：反转义unicode编码

escapeJson：转义json字符

escapeXml10：转义Xml10

这个现在已经废弃了，建议使用commons-text包里面的方法。



#### 16. org.apache.commons.beanutils.BeanUtils

---

```xml
<dependency>
  <groupId>commons-beanutils</groupId>
  <artifactId>commons-beanutils</artifactId>
  <version>1.9.4</version>
</dependency>
```

copyPeoperties：复制属性值，从一个对象到另一个对象

getProperty：获取对象属性值

setProperty：设置对象属性值

populate：根据Map给属性复制

copyPeoperty：复制单个值，从一个对象到另一个对象

cloneBean：克隆bean实例

