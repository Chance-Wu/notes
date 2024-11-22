### 案例

---

添加一个学生，前端把生日传给后端，后端使用Datel类型接收到后，然后调用其它服务进行保存入库。

与其它服务交互时，使用的是JSON格式，出现日期少一天。

```java
@Data
@AllArgsConstructor
public class Student {

  @JsonFormat(pattern = "yyyy-MM-dd")
  private Date birthday;

}
```

```java
@Test
public void test1() throws Exception {
  String date = "1990-06-01";
  Date birthday = DateUtils.parseDateStrictly(date, "yyyy-MM-dd");
  System.out.println("birthday:" + birthday);
  
  Student student = new Student(birthday);
  ObjectMapper mapper = new ObjectMapper();
  // 对象转JSON串
  System.out.println("student：" + mapper.writeValueAsString(student));
}
```

控制台输出结果：

birthday:Fri Jun 01 00:00:00 CDT 1990
student：{"birthday":"1990-05-31"}



### 原因

---

两个关键点：

1. birthday输出带有CDT；
2. 日期是1990-06-01

CDT：夏令时标志（一般在天亮早的夏季人为将时间调快一小时）

1986年至1991年，中华人民共和国在全国范围实行了六年夏令时。



### 解决方案

---

统一使用字符串交互，在开发中仅在Entity（与数据库表对应的）中出现Date，其它场景统一使用String。

