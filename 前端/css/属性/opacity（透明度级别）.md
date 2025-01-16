### 一、属性定义及使用说明

---

### 1.1 定义

opacity 属性设置一个元素的透明度级别。

#### 1.2 使用

```html
<!DOCTYPE html>
<html>
<head>
<style> 
div
{
background-color:red;
opacity:0.5;
}
</style>
</head>
<body>

<div>This element's opacity is 0.5! Note that both the text and the background-color are affected by the opacity level!</div>

</body>
</html>
```



### 二、语法

---

| 值      | 描述                                               |
| :------ | :------------------------------------------------- |
| *value* | 指定不透明度。从0.0（完全透明）到1.0（完全不透明） |
| inherit | Opacity属性的值应该从父元素继承                    |