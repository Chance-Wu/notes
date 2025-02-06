### 一、font-family

---

#### 1.1 定义

- 指定一个元素的字体。
- 可以把多个字体名称作为一个"回退"系统来保存。如果浏览器不支持第一个字体，则会尝试下一个。

#### 1.2 使用

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<style>
p.serif{font-family:"Times New Roman",Times,serif;}
p.sansserif{font-family:Arial,Helvetica,sans-serif;}
</style>
</head>

<body>
<h1>CSS font-family</h1>
<p class="serif">这一段的字体是 Times New Roman </p>
<p class="sansserif">这一段的字体是 Arial.</p>

</body>
</html>
```

#### 1.3 属性值

| 值                            | 描述                                                         |
| :---------------------------- | :----------------------------------------------------------- |
| family-name<br>generic-family | 用于某个元素的字体族名称或/及类族名称的一个优先表。默认值：取决于浏览器。 |
| inherit                       | 规定应该从父元素继承字体系列。                               |

使用某种特定的字体系列（Geneva）完全取决于用户机器上该字体系列是否可用；这个属性没有指示任何字体下载。因此，强烈推荐使用一个通用字体系列名作为后路。

**注意：**

- 每个值用逗号分开。
- 如果字体名称包含空格，它必须加上引号。在HTML中使用"style"属性时，必须使用单引号。