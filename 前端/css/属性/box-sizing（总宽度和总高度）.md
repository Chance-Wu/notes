### 一、box-sizing

---

#### 1.1 定义

定义如何计算一个元素的**总宽度**和**总高度**，主要设置是否需要加上内边距(padding)和边框等。

例如，假如您需要并排放置两个带边框的框，可通过将 box-sizing 设置为 "border-box"。这样就可以让浏览器呈现出带有指定宽度和高度的框，并把边框和内边距放入框中。

默认情况下，元素的宽度(width) 和高度(height)计算方式如下：

```
width(宽度) + padding(内边距) + border(边框) = 元素实际宽度
height(高度) + padding(内边距) + border(边框) = 元素实际高度
```

#### 1.2 使用

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title>
<style> 
#example1 {
  box-sizing: content-box;  
  width: 300px;
  height: 100px;
  padding: 30px;  
  border: 10px solid blue;
}

#example2 {
  box-sizing: border-box;
  width: 300px;
  height: 100px;
  padding: 30px;  
  border: 10px solid blue;
}
</style>
</head>
<body>

<h1>box-sizing 属性</h1>
<p>定义如何计算一个元素的总宽度和总高度，是否包含内边距和边框。</p>

<h2>box-sizing: content-box (默认):</h2>
<p>高度和宽度只应用于元素的内容:</p>
<div id="example1">这个 div 的宽度为 300px。但完整宽度为 300px + 20px (左边框和右边框) + 60px (左边距和右边距) = 380px!</div>

<h2>box-sizing: border-box:</h2>
<p>高度和宽度应用于元素的所有部分: 内容、内边距和边框:</p>
<div id="example2">不管如何这里的完整宽度为300px!</div>

</body>
</html>
```

#### 1.3 属性值

| 值          | 说明                                                         |
| :---------- | :----------------------------------------------------------- |
| content-box | 默认值。如果你设置一个元素的宽为 100px，那么这个元素的内容区会有 100px 宽，并且任何边框和内边距的宽度都会被增加到最后绘制出来的元素宽度中。 |
| border-box  | 告诉浏览器：你想要设置的边框和内边距的值是包含在 width 内的。也就是说，如果你将一个元素的 width 设为 100px，那么这 100px 会包含它的 border 和 padding，内容区的实际宽度是 width 减 去(border + padding) 的值。大多数情况下，这使得我们更容易地设定一个元素的宽高。 **注：**border-box 不包含 margin。 |
| inherit     | 指定 box-sizing 属性的值，应该从父元素继承                   |