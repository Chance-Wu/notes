> Java8之前处理日期、日历和时间的不足之处：==将java.util.Date设定为可变类型（没用final修饰类）==，以及==SimpleDateFormat的非线程安全==使其应用非常受限。

> 新API的好处之一就是，明确了日期时间概念，例如：瞬时（Instant）、长短（Duration）、日期（LocalDate）、时间（LocalTime）、时区（ZoneId）和周期（Period）。
>
> 同时继承了Joda库按人类语言和计算机各自解析的时间处理方式。不同于老版本，新API基于ISO标准日历系统，==java.time包下的所有类都是不可变类型而且线程安全的==。



#### 1. java.time包下的类

---

- Instant：瞬时实例
- **LocalDate**：==本地日期==，不包含具体时间。例如：2014-01-14
- LocalTime：本地时间，不包含日期。
- **LocalDateTime**：==组合了日期和时间==，但不包含时差和时区信息。
- **ZoneDateTime**：最完整的日期时间，包含时区和相对UTC或格林威治的时差。



#### 2. 当前日期

---

当天的日期，不含时间。

```java
LocalDate now = LocalDate.now();
// 2021-07-13
```



#### 3. 年月日信息

---

LocalDate提供了获取年、月、日的快捷方法，其实例还包含很多其他的日期属性。

```java
LocalDate today = LocalDate.now(); // 2021-07-13
int year = today.getYear(); // 2021
int month = today.getMonthValue(); // 5
int day = today.getDayOfMonth(); // 13
```



#### 4. 处理特定日期

---

`LocalDate.of(int year, int month, int dayOfMonth)`创建任意日期，该方法需要传入年、月、日做参数，返回对应的LocalDate实例。

```java
LocalDate dateOfBirth = LocalDate.of(1994,09,19);
// 1994-09-19
```



#### 5. 判断两个日期是否相等

---

LocalDate重载了equals方法。比较日期如果是字符型的，需要先解析成日期对象再做判断。

```java
LocalDate today1 = LocalDate.now();
LocalDate date1 = LocalDate.of(2020, 05, 26);

if (date1.equals(today1)) {

}
```



#### 6. 检查像生日这种周期性事件

---

`MonthDay`类组合了==月份和日==，去掉了年，可以用它判断每年都会发生的事件。类似的还有`YearMonth`类。这些类都是不可变的且线程安全的值类型。

```java
LocalDate today = LocalDate.now();
LocalDate dateOfBirth = LocalDate.of(2020, 9, 19);
MonthDay birthDay = MonthDay.of(dateOfBirth.getMonth(), dateOfBirth.getDayOfMonth());
//当前的月份日期
MonthDay currentMonthDay = MonthDay.from(today);
if (currentMonthDay.equals(birthDay)) {
  System.out.println("Many Many happy returns of the day !!");
} else {
  System.out.println("Sorry, today is not your birthday");
}
```



#### 7. 获取当前时间

---

`LocalTime`类==只有时间==。可以调用now()方法来获取当前时间。默认格式`hh:mm:ss:nnn`。

```java
LocalTime time = LocalTime.now();
// 16:24:49:638
```



#### 8. 在现在的时间上增加小时

---

`plusHours(long hours)`方法返回一个全新的LocalTime实例，由于其不可变性，返回后一定要用变量赋值。

```java
LocalTime now = LocalTime.now();
LocalTime newTime = now.plusHours(2); //加2小时
```



#### 9. 计算一个星期之后的日期

---

LocalDate日期不包含时间信息，它的plus(long amountToAdd, TemporalUnit unit)方法用来增加天、周、月，ChronoUnit类声明了这些时间单位。

```java
LocalDate today = LocalDate.now();
LocalDate nextWeek = today.plus(1, ChronoUnit.WEEKS);
```



#### 10. 计算一年或一年后的日期

---

`minus(long amountToSubtract, TemporalUnit unit)`方法计算一年前的日期。

```java
LocalDate now = LocalDate.now();
LocalDate previousYear = now.minus(1, ChronoUnit.YEARS);
```



#### 11. 使用Java8的Clock时钟类

---

`Clock`时钟类用于获取当时的时间戳，或当前时区下的日期时间信息。以前用到`System.currentTimeInMillis()`的地方都可以用Clock替换。

```java
// 根据系统时间返回当前时间并设置为UTC
Clock clock = Clock.systemUTC();
System.out.println("Clock : " + clock);

// 根据系统时钟区域返回时间
Clock defaultClock = Clock.systemDefaultZone();
System.out.println("DefaultClock : " + defaultClock);
```



#### 12. 判断日期是早于还是晚于另一个日期

---

LocalDate类有两类方法`isBefore(ChronoLocalDate other)`和`isAfter(ChronoLocalDate other)`用于比较日期。调用isBefore()方法时，如果给定日期小于当前日期则返回true。

```java
LocalDate today = LocalDate.now();
LocalDate tomorrow = LocalDate.of(2020, 05, 28);
if (tomorrow.isAfter(today)) {
  System.out.println("Tomorrow comes after today");
}

LocalDate yesterday = today.minus(1, ChronoUnit.DAYS);
if (yesterday.isBefore(today)) {
  System.out.println("Yesterday is day before today");
}
```



#### 13. 处理时区

---

Java 8不仅分离了日期和时间，也把时区分离出来了。

- `ZoneId`==处理特定时区==；
- `ZoneDateTime`==表示某时区下的时间==。



#### 14. 体现固定日期

---

YearMonth实例的`lengthOfMonth()`方法可以返回当月的天数，在判断2月有28天还是29天时非常有用。

```java
YearMonth currentYearMonth = YearMonth.now();
System.out.printf("Days in month year %s: %d%n", currentYearMonth, currentYearMonth.lengthOfMonth());

YearMonth creditCardExpiry = YearMonth.of(2028, Month.FEBRUARY);
System.out.printf("Your credit card expires on %s %n", creditCardExpiry);
```



#### 15. 检查闰年

---

LocalDate类有个`isLeapYear()`方法判断该实例是否是一个闰年。

```java
LocalDate today = LocalDate.now();
if (today.isLeapYear()) {
  
}
```



#### 16. 计算两个日期之间的天数和月数

---

用`Period`类来做计算。

当前月份，距离到7月份，中间相隔几个月？

```java
LocalDate now1 = LocalDate.now();
LocalDate java8Release = LocalDate.of(2021, Month.JULY, 14);
Period periodToNextJavaRelease = Period.between(now1, java8Release);
System.out.println("Months left between today and Java 8 release : " + periodToNextJavaRelease.getMonths());
```



#### 17. 包含时差信息的日期和时间

---

ZoneOffset类用来表示时区，举例来说印度与GMT或UTC标准时区相差+05:30，可以通过ZoneOffset.of()静态方法来获取对应的时区。一旦得到了时差就可以通过传入LocalDateTime和ZoneOffset来创建一个OffSetDateTime对象。



#### 18. 获取当前时间戳

---

`Instant`类有一个静态工厂方法`now()`会返回当前的时间戳。

```java
Instant timestamp = Instant.now();
```



#### 19. 使用预定义的格式化工具去解析格式化日期

---

`DateTimeFormatter`

```java
String dateStr = "20200529";
LocalDate formatted = LocalDate.parse(dateStr, DateTimeFormatter.BASIC_ISO_DATE);
// 2021-07-14

LocalDateTime now = LocalDateTime.now();
DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("YYYY-mm-dd hh:mm:ss");
String format = dateTimeFormatter.format(now);
// 2021-07-14 13:25:12
```



#### 20. 总结

---

1）提供了javax.time.ZoneId 获取时区。

2）提供了LocalDate和LocalTime类。

3）Java 8 的所有日期和时间API都是不可变类并且线程安全。

4）主包是 java.time，包含了表示日期、时间、时间间隔的一些类。里面有两个子包java.time.format用于格式化， java.time.temporal用于更底层的操作。

5）时区代表了地球上某个区域内普遍使用的标准时间。每个时区都有一个代号，格式通常由区域/城市构成（Asia/Tokyo），在加上与格林威治或 UTC的时差。例如：东京的时差是+09:00。