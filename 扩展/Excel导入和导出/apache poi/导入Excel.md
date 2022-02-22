#### 1. POI相关maven依赖

```xml
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi</artifactId>
    <version>3.17</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>3.13</version>
</dependency>
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml-schemas</artifactId>
    <version>3.13</version>
</dependency>
```

#### 2. 准备好Excel源文件

| 标题说明 |             |        |        |      |
| -------- | ----------- | ------ | ------ | ---- |
| 姓名     | 手机号      | 密码   | 部门   | 年级 |
| 测试1    | 13871195841 | admin1 | 开发部 | 2019 |
| 测试2    | 13871195842 | admin2 | 研发部 | 2020 |

#### 3. 获取文件流+解析Excel

```java
File file = ResourceUtils.getFile("classpath:test.xlsx");
FileInputStream is = new FileInputStream(file);
// 通过文件流创建Excel工作簿对象
Workbook workbook = WorkbookFactory.create(is);
// 得到第一个Excel工作表对象
Sheet sheet = workbook.getSheetAt(0);
// 得到Excel工作表总行数
int totalRow = sheet.getPhysicalNumberOfRows();

List<ImportVo> importVoList = new ArrayList<>();
// 从第三行开始解析
for (int i = 2; i < totalRow; i++) {
    // 得到Excel工作表指定的行
    Row row = sheet.getRow(i);
    ImportVo importVo = new ImportVo();
    importVo.setMemberName(row.getCell(0).getStringCellValue());
    importVo.setPhoneNumber(row.getCell(1).getStringCellValue());
    importVo.setPassword(row.getCell(2).getStringCellValue());
    importVo.setDepartment(row.getCell(3).getStringCellValue());
    importVo.setGrade(row.getCell(4).getStringCellValue());
    importVoList.add(importVo);
}
```

#### 4. Excel的后缀名

如果是`xls`，使用`HSSFWorkbook`；
 如果是`xlsx`，使用`XSSFWorkbook`，
 读取两种格式使用`Workbook`,不然会报异常`org.apache.poi.poifs.filesystem.OfficeXmlFileException: The supplied data appears to be in the Office 2007+ XML.`

##### 5. 相关方法说明

> 得到Excel工作簿对象
XSSFWorkbook book = new XSSFWorkbook();

> 得到Excel工作表对象
XSSFSheet sheet = book.getSheetAt(0);

> 总行数
sheet.getLastRowNum();

> 得到Excel工作表的行
sheet.getRow();

> 得到Excel工作表指定行的单元格
row.getCell();

#### 6. 常见问题

在取数据的时候很容易碰到这样的异常`Cannot get a text value from a numeric cell`无法将数值类型转化成`String`

解决：

1. 通过String.valueOf()转换
2. 将所有列中的内容都设置成String类型格式【`row.setCell(要设置的列数，从0开始).setCellType(CellType.STRING);`】