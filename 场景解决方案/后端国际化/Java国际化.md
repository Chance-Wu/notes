### 一、简介

---

“国际化信息”也称为“本地化信息”，一般需要两个条件才可以确定一个特定类型的本地化信息，它们分别是“**语言类型**”和“**国家/地区的类型**”。如中文本地化信息既有中国大陆地区的中文，又有中国台湾、中国香港地区的中文，还有新加坡地区的中文。Java通过java.util.Locale类表示一个本地化对象，它允许通过语言参数和国家/地区参数创建一个确定的本地化对象。

- 语言参数使用ISO标准语言代码表示，这些代码是由ISO-639标准定义的，每一种语言由两个小写字母表示。在许多网站上都可以找到这些代码的完整列表，下面的网址是提供了标准语言代码的信息： http://www.loc.gov/standards/iso639-2/php/English_list.php。
- 国家/地区参数也由标准的ISO国家/地区代码表示，这些代码是由ISO-3166标准定义的，每个国家/地区由两个大写字母表示。用户可以从以下网址查看ISO-3166的标准代码： http://www.iso.ch/iso/en/prods-services/iso3166ma/02iso-3166-code-lists/list-en1.html。

| 语言 | 代码 | 国家/地区代码示例 | 代号 |
| ---- | ---- | ----------------- | ---- |
| 中文 | zh   | 中国大陆          | CN   |
| 英语 | en   | 中国台湾          | TW   |
| 法语 | fr   | 中国香港          | HK   |
| 德语 | de   | 英国              | EN   |
| 日语 | ja   | 美国              | US   |
| 韩语 | ko   | 加拿大            | CA   |



### 二、Locale

---

java.util.Locale是表示语言和国家/地区信息的本地化类，它是创建国际化应用的基础。下面给出几个创建本地化对象的示例：

```java
// 带有语言和国家/地区信息的本地化对象
Locale locale1 = new Locale("zh", "CN");
Locale locale2 = Locale.CHINA;

// 只有语言信息的本地化对象
Locale locale3 = new Locale("zh");
Locale locale4 = Locale.CHINESE;

// 获取本地系统默认的本地化对象
Locale locale5 = Locale.getDefault();
```

用户既可以同时指定语言和国家/地区参数定义一个本地化对象；也可以仅通过语言参数定义一个泛本地化对象；Locale类中通过静态常量定义了一些常用的本地化对象；此外，用户还可以获取系统默认的本地化对象。



### 三、本地化工具类

---

JDK的java.util包中提供了几个支持本地化的格式化操作工具类：NumberFormat、DateFormat、MessageFormat。

```java
Locale locale = new Locale("zh", "CN");  
NumberFormat currFmt = NumberFormat.getCurrencyInstance(locale);  
double amt = 123456.78;  
System.out.println(currFmt.format(amt));
```

上面的实例通过NumberFormat按本地化的方式对货币金额进行格式化操作，运行实例，输出以下信息：

```
￥123,456.78
```

```java
Locale locale = new Locale("en", "US");  
Date date = new Date();  
DateFormat df = DateFormat.getDateInstance(DateFormat.MEDIUM, locale);  
System.out.println(df.format(date));
```

通过`DateFormat#getDateInstance(int style,Locale locale)`方法按本地化的方式对日期进行格式化操作。该方法第一个入参为时间样式，第二个入参为本地化对象。运行以上代码，输出以下信息：

```
Jan 8, 2007
```

MessageFormat在NumberFormat和DateFormat的基础上提供了强大的占位符字符串的格式化功能，它支持时间、货币、数字以及对象属性的格式化操作。下面的实例演示了一些常见的格式化功能：

```java
//①信息格式化串  
String pattern1 = "{0}，你好！你于 {1} 在工商银行存入 {2} 元。";
String pattern2 = "At {1,time,short} On {1,date,long}，{0} paid {2,number, currency}.";

//②用于动态替换占位符的参数  
Object[] params = {"John", new GregorianCalendar().getTime(), 1.0E3};

//③使用默认本地化对象格式化信息  
String msg1 = MessageFormat.format(pattern1, params);

//④使用指定的本地化对象格式化信息  
MessageFormat mf = new MessageFormat(pattern2, Locale.US);
String msg2 = mf.format(params);
System.out.println(msg1);
System.out.println(msg2);
```

pattern1是简单形式的格式化信息串，通过{n}占位符指定动态参数的替换位置索引，{0}表示第一个参数，{1}表示第二个参数，以此类推。

pattern2格式化信息串比较复杂一些，除参数位置索引外，还指定了参数的类型和样式。从pattern2中可以看出格式化信息串的语法是很灵活的，一个参数甚至可以出现在两个地方：如 {1,time,short}表示从第二个入参中获取时间部分的值，显示为短样式时间；而{1,date,long}表示从第二个入参中获取日期部分的值，显示为长样式时间。关于MessageFormat更详细的使用方法，请参见JDK的Javadoc。

在②处，定义了用于替换格式化占位符的动态参数，这里，我们使用到了JDK5.0自动装包的语法，否则必须采用封装类表示基本类型的参数值。

在③处，通过MessageFormat的format()方法格式化信息串。它使用了系统默认的本地化对象，由于我们是中文平台，因此默认为Locale.CHINA。而在④处，我们显式指定MessageFormat的本地化对象。

运行上面的代码，输出以下信息：

```
John，你好！你于 14-7-7 下午11:29 在工商银行存入 1,000 元。
At 11:29 PM On July 7, 2014，John paid $1,000.00.
```

如果应用系统中某些信息需要支持国际化功能，则必须为希望支持的不同本地化类型分别提供对应的资源文件，并以规范的方式进行命名。国际化资源文件的命名规范规定资源名称采用以下的方式进行命名：

`<资源名>_<语言代码>_<国家/地区代码>.properties`
