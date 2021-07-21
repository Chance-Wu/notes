java bean拷贝操作的一个工具类。



org.springframework.beans.BeanUtils、org.apache.commons.beanutils.BeanUtils



a，b为对象



`BeanUtils.copyProperties(a,b);`

1. spring下的BeanUtils是a拷贝到b
2. apache下的是b拷贝到a



#### 1. Spring中的BeanUtils

>spring中实现copyProperties的方法很简单，就是对两个对象中相同名字的属性进行简单get/set，==仅检查属性的可访问性==。
>
>```java
>public static void copyProperties(Object source, Object target) throws BeansException {
>  copyProperties(source, target, null, (String[]) null);
>}
>
>private static void copyProperties(Object source, Object target, @Nullable Class<?> editable,
>                                   @Nullable String... ignoreProperties) throws BeansException {
>
>  Assert.notNull(source, "Source must not be null");
>  Assert.notNull(target, "Target must not be null");
>
>  Class<?> actualEditable = target.getClass();
>  if (editable != null) {
>    if (!editable.isInstance(target)) {
>      throw new IllegalArgumentException("Target class [" + target.getClass().getName() +
>                                         "] not assignable to Editable class [" + editable.getName() + "]");
>    }
>    actualEditable = editable;
>  }
>  PropertyDescriptor[] targetPds = getPropertyDescriptors(actualEditable);
>  List<String> ignoreList = (ignoreProperties != null ? Arrays.asList(ignoreProperties) : null);
>
>  for (PropertyDescriptor targetPd : targetPds) {
>    Method writeMethod = targetPd.getWriteMethod();
>    if (writeMethod != null && (ignoreList == null || !ignoreList.contains(targetPd.getName()))) {
>      PropertyDescriptor sourcePd = getPropertyDescriptor(source.getClass(), targetPd.getName());
>      if (sourcePd != null) {
>        Method readMethod = sourcePd.getReadMethod();
>        if (readMethod != null) {
>          ResolvableType sourceResolvableType = ResolvableType.forMethodReturnType(readMethod);
>          ResolvableType targetResolvableType = ResolvableType.forMethodParameter(writeMethod, 0);
>          if (targetResolvableType.isAssignableFrom(sourceResolvableType)) {
>            try {
>              if (!Modifier.isPublic(readMethod.getDeclaringClass().getModifiers())) {
>                readMethod.setAccessible(true);
>              }
>              Object value = readMethod.invoke(source);
>              if (!Modifier.isPublic(writeMethod.getDeclaringClass().getModifiers())) {
>                writeMethod.setAccessible(true);
>              }
>              writeMethod.invoke(target, value);
>            }
>            catch (Throwable ex) {
>              throw new FatalBeanException(
>                "Could not copy property '" + targetPd.getName() + "' from source to target", ex);
>            }
>          }
>        }
>      }
>    }
>  }
>}
>```



#### 2. apache下的BeanUtils

---

apache的BeanUtils里加了很多检验，包括==类型的转换==，甚至还会==检验对象所属的类的可访问性==。

不支持java.util.Date的转换。除了支持基本类型以及基本类型的数组之外，还支持java.sql.Date，java.sql.TimeStamp，java.io.File，javaio.URL这些类的对象，其余一概不支持。==不建议使用==。

```java
public static void copyProperties(final Object dest, final Object orig)
  throws IllegalAccessException, InvocationTargetException {

  BeanUtilsBean.getInstance().copyProperties(dest, orig);
}

public void copyProperties(final Object dest, final Object orig)
  throws IllegalAccessException, InvocationTargetException {

  // 验证指定的bean是否存在
  if (dest == null) {
    throw new IllegalArgumentException
      ("No destination bean specified");
  }
  if (orig == null) {
    throw new IllegalArgumentException("No origin bean specified");
  }
  if (log.isDebugEnabled()) {
    log.debug("BeanUtils.copyProperties(" + dest + ", " +
              orig + ")");
  }

  // 复制属性，根据需要进行转换
  if (orig instanceof DynaBean) {
    final DynaProperty[] origDescriptors =
      ((DynaBean) orig).getDynaClass().getDynaProperties();
    for (DynaProperty origDescriptor : origDescriptors) {
      final String name = origDescriptor.getName();
      // 需要检查 WrapDynaBean 的 isReadable()
      if (getPropertyUtils().isReadable(orig, name) &&
          getPropertyUtils().isWriteable(dest, name)) {
        final Object value = ((DynaBean) orig).get(name);
        copyProperty(dest, name, value);
      }
    }
  } else if (orig instanceof Map) {
    @SuppressWarnings("unchecked")
    final
      // Map属性始终为 <String, Object> 类型
      Map<String, Object> propMap = (Map<String, Object>) orig;
    for (final Map.Entry<String, Object> entry : propMap.entrySet()) {
      final String name = entry.getKey();
      if (getPropertyUtils().isWriteable(dest, name)) {
        copyProperty(dest, name, entry.getValue());
      }
    }
  } else /* if (orig is a standard JavaBean) */ {
    final PropertyDescriptor[] origDescriptors =
      getPropertyUtils().getPropertyDescriptors(orig);
    for (PropertyDescriptor origDescriptor : origDescriptors) {
      final String name = origDescriptor.getName();
      if ("class".equals(name)) {
        continue; // 尝试设置对象的类没有意义
      }
      if (getPropertyUtils().isReadable(orig, name) &&
          getPropertyUtils().isWriteable(dest, name)) {
        try {
          final Object value =
            getPropertyUtils().getSimpleProperty(orig, name);
          copyProperty(dest, name, value);
        } catch (final NoSuchMethodException e) {
          // 不应该发生
        }
      }
    }
  }

}
```

