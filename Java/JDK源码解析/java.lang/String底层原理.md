#### 1. String的定义

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
}
```

- final类，不能被继承
- 实现了`java.io.Serializable接口`，可以实现序列化
- 实现了`Comparable<String>接口`，可用于比较大小
- 实现了`CharSequence接口`，表示一个有序字符的序列，因为==String本质是一个char类型数组==。

#### 2. 字段属性

```java
/** 用来存储char型字符数组，而且是final的，不可变的 */
private final char value[];

/** 缓存hash，默认0 */
private int hash;

/** 实现序列化的标识 */
private static final long serialVersionUID = -6849794470754667710L;
```

#### 3. 构造函数

可以构造空字符串对象,既""

可以根据String，StringBuilder，StringBuffer构造字符串对象

可以根据char数组，其子数组构造字符串对象

可以根据int数组，其子数组构造字符串对象

可以根据某个字符集编码对byte数组，其子数组解码并构造字符串对象

#### 4. 长度、是否为空

```java
public int length() {
    return value.length;
}

public boolean isEmpty() {
    return value.length == 0;
}
```

#### 5. charAt、codePointAt类型函数

```java
/**
 * 返回String对象的char数组index位置的元素
 */
public char charAt(int index) {
    if ((index < 0) || (index >= value.length)) {
        throw new StringIndexOutOfBoundsException(index);
    }
    return value[index];
}

/**
 * 返回String对象的char数组index位置的元素的ASSIC码(int类型)
 */
public int codePointAt(int index) {
    if ((index < 0) || (index >= value.length)) {
        throw new StringIndexOutOfBoundsException(index);
    }
    return Character.codePointAtImpl(value, index, value.length);
}

/**
 * 方法返回的是代码点个数，是实际上的字符个数,功能类似于length()
 * 对于正常的String来说，length方法和codePointCount没有区别，都是返回字符个数。
 * 但当String是Unicode类型时则有区别了。
 * 例如：String str = “/uD835/uDD6B” (即 'Z' ), length() = 2 ,codePointCount() = 1 
 */
public int codePointBefore(int index) {
    int i = index - 1;
    if ((i < 0) || (i >= value.length)) {
        throw new StringIndexOutOfBoundsException(index);
    }
    return Character.codePointBeforeImpl(value, index, 0);
}

/**
 * 相对Unicode字符集而言的，从index位置算起，偏移codePointOffset个位置，返回偏移后的位置是多少
 * 例如，index = 2 ,codePointOffset = 3 ，maybe返回 5 
 */
public int codePointCount(int beginIndex, int endIndex) {
    if (beginIndex < 0 || endIndex > value.length || beginIndex > endIndex) {
        throw new IndexOutOfBoundsException();
    }
    return Character.codePointCountImpl(value, beginIndex, endIndex - beginIndex);
}
```

#### 6. getChar、getBytes类型函数

```java
/** 返回String对象的char数组index位置的元素*/
public char charAt(int index) {
    if ((index < 0) || (index >= value.length)) {
        throw new StringIndexOutOfBoundsException(index);
    }
    return value[index];
}

//返回String对象的char数组index位置的元素的ASSIC码(int类型)
public int codePointAt(int index) {
    if ((index < 0) || (index >= value.length)) {
        throw new StringIndexOutOfBoundsException(index);
    }
    return Character.codePointAtImpl(value, index, value.length);
}

//返回index位置元素的前一个元素的ASSIC码(int型)
public int codePointBefore(int index) {
    int i = index - 1;
    if ((i < 0) || (i >= value.length)) {
        throw new StringIndexOutOfBoundsException(index);
    }
    return Character.codePointBeforeImpl(value, index, 0);
}

/**
 * 方法返回的是代码点个数，是实际上的字符个数,功能类似于length()
 * 对于正常的String来说，length方法和codePointCount没有区别，都是返回字符个数。
 * 但当String是Unicode类型时则有区别了。
 * 例如：String str = “/uD835/uDD6B” (即 'Z' ), length() = 2 ,codePointCount() = 1 
 */
public int codePointCount(int beginIndex, int endIndex) {
    if (beginIndex < 0 || endIndex > value.length || beginIndex > endIndex) {
        throw new IndexOutOfBoundsException();
    }
    return Character.codePointCountImpl(value, beginIndex, endIndex - beginIndex);
}

/**
 * 相对Unicode字符集而言的，从index位置算起，偏移codePointOffset个位置，返回偏移后的位置是多少
 * 例如，index = 2 ,codePointOffset = 3 ，maybe返回 5 
 */
public int offsetByCodePoints(int index, int codePointOffset) {
    if (index < 0 || index > value.length) {
        throw new IndexOutOfBoundsException();
    }
    return Character.offsetByCodePointsImpl(value, 0, value.length,
                                            index, codePointOffset);
}
```

#### 7. equals函数

equals()方法具有层次感和参考意义：

- 首先判断是否为同一个对象；
- 再判断是否为要比较的类型；
- 再判断两个对象的长度是否相等。

==首先从广的角度过滤不符合的对象，在符合条件的对象基础上再一个一个字符的比较。==

```java
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    // 首先判断是否是同一个对象
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        // 比较长度，长度相等则再逐个比较字符串
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}

// 忽略大小写比较是否内容相同,先判断地址,再判断长度,最后再执行regionMatchs方法,忽略大小写
public boolean equalsIgnoreCase(String anotherString) {
    return (this == anotherString) ? true
        : (anotherString != null)
    && (anotherString.value.length == value.length)
        && regionMatches(true, 0, anotherString, 0, value.length);
}
```

#### 8. compareTo类函数

```java
/**
 * 比较字符串大小
 * 返回的int类型，正数为大，负数为小，是基于字符的ASSIC码比较的
 * 例如: "abc".compareTo("bac") 结果是-1,因为a-b=-1
 *      "abc".compareTo("abcggff"); //-4,因为长度差4
 */
public int compareTo(String anotherString) {
    int len1 = value.length;
    int len2 = anotherString.value.length;
    int lim = Math.min(len1, len2);
    char v1[] = value;
    char v2[] = anotherString.value;

    int k = 0;
    while (k < lim) {
        char c1 = v1[k];
        char c2 = v2[k];
        if (c1 != c2) {
            return c1 - c2;
        }
        k++;
    }
    return len1 - len2;
}
```

#### 9. startWith、endWith类函数

```java
/**
 * 判断当前字符串对象是否以字符串prefix起头
 * 是返回true,否返回fasle
 */
public boolean startsWith(String prefix) {
    return startsWith(prefix, 0);
}

/**
 * 判断当前字符串对象是否以字符串prefix结尾
 * 是返回true,否返回fasle
 */
public boolean endsWith(String suffix) {
    //suffix是需要判断是否为尾部的字符串。
    //value.length - suffix.value.length是suffix在当前对象的起始位置
    return startsWith(suffix, value.length - suffix.value.length); 
}
```

#### 10. hashCode函数

- hashCode的重点是hash函数；
- String的哈希函数就是循环len次，每次循环体为31*每次循环获得的hash+第i次循环的字符。

```java
/**
 * 这是String字符串重写了Object类的hashCode方法。
 * 给由哈希表来实现的数据结构来使用，比如String对象要放入HashMap中。
 * 如果没有重写HashCode，或HaseCode质量很差则会导致严重的后果，既不靠谱的后果
 */
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;
        //重点，String的哈希函数, //遍历len次
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];  //每次都是31 * 每次循环获得的h +第i个字符的ASSIC码
        }
        hash = h;
    }
    return h;
}
```