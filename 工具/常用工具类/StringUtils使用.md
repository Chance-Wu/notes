StringUtils类主要是处理关于字符串的功能方法。



#### 1. 判断字符串是否为空，如果为null或者""则返回true

---

```java
@Deprecated
public static boolean isEmpty(@Nullable Object str) {
  return (str == null || "".equals(str));
}
```

>注意：自 5.3 开始弃用，支持 
>
>- hasLength(String)
>- hasText(String)
>- ObjectUtils#isEmpty(Object)



#### 2. 去除字符串前后空白

---





























