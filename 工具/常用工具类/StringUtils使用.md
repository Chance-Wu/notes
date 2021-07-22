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

```java
public static String trimWhitespace(String str) {
  if (!hasLength(str)) {
    return str;
  }

  int beginIndex = 0;
  int endIndex = str.length() - 1;

  // 去除字符串头部空白
  while (beginIndex <= endIndex && Character.isWhitespace(str.charAt(beginIndex))) {
    beginIndex++;
  }

  // 去除字符串尾部空白
  while (endIndex > beginIndex && Character.isWhitespace(str.charAt(endIndex))) {
    endIndex--;
  }

  return str.substring(beginIndex, endIndex + 1);
}
```



#### 3. 去除字符串所有空白

---

```java
public static String trimAllWhitespace(String str) {
  if (!hasLength(str)) {
    return str;
  }

  int len = str.length();
  StringBuilder sb = new StringBuilder(str.length());
  //从第一个字符开始判断，是否为空，如果是则删除，如果不是则index加1
  //判断第二个，然后一直到sb.length=index，则跳出循环
  for (int i = 0; i < len; i++) {
    char c = str.charAt(i);
    if (!Character.isWhitespace(c)) {
      sb.append(c);
    }
  }
  return sb.toString();
}
```



#### 4. 忽略大小写，判断字符串是否以prefix开头

---

```java
public static boolean startsWithIgnoreCase(@Nullable String str, @Nullable String prefix) {
  return (str != null && prefix != null && str.length() >= prefix.length() &&
          str.regionMatches(true, 0, prefix, 0, prefix.length()));
}
```



#### 5. 忽略大小写，判断字符串是否以suffix结尾

---

```java
public static boolean endsWithIgnoreCase(@Nullable String str, @Nullable String suffix) {
  return (str != null && suffix != null && str.length() >= suffix.length() &&
          str.regionMatches(true, str.length() - suffix.length(), suffix, 0, suffix.length()));
}
```

