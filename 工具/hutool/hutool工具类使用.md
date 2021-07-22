```xml
<dependency>
  <groupId>cn.hutool</groupId>
  <artifactId>hutool-all</artifactId>
  <version>5.7.5</version>
</dependency>
```



#### 1. 编码工具

---

- HexUtil
-  EscapeUtil
- HashUtil
- URLUtil
- Base32、Base64、Base32Codec、Base64Decoder、Base64Encoder
- UnicodeUtil



#### 2. JSONUtil工具类

---

```java
/**
 * 创建JSONObject
 * 
 * @return JSONObject
 */
public static JSONObject createObj() {
  return new JSONObject();
}
```

```java
**
  * 转换对象为JSON<br>
  * 支持的对象：<br>
  * String: 转换为相应的对象<br>
    * Array Collection：转换为JSONArray<br>
    * Bean对象：转为JSONObject
    * 
    * @param obj 对象
    * @return JSON
    */
    public static JSON parse(Object obj) {
    if (null == obj) {
      return null;
    }

    JSON json = null;
    if (obj instanceof JSON) {
      json = (JSON) obj;
    } else if (obj instanceof String) {
      String jsonStr = ((String) obj).trim();
      if (jsonStr.startsWith("[")) {
        json = parseArray(jsonStr);
      } else {
        json = parseObj(jsonStr);
      }
    } else if (obj instanceof Collection || obj.getClass().isArray()) {// 列表
      json = new JSONArray(obj);
    } else {// 对象
      json = new JSONObject(obj);
    }

    return json;
  }
```



#### 3. 类和对象

---



#### 4. 系统工具

---





#### 5. 文件相关

---





#### 6. 需要第三方的

---

