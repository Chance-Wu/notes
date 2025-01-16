### 一、transition

---

#### 1.1 定义

transition 属性允许你在元素状态改变时控制过渡效果。它可以让你在元素从一种样式变换到另一种样式时产生平滑的过渡效果，比如从一种颜色渐变到另一种颜色，或者从隐藏到显示。

这个属性包含四个值：

- transition-property：指定要过渡的 CSS 属性的名称。例如，`color`、`background-color` 等。
- transition-duration：指定过渡效果持续的时间，以秒或毫秒为单位。
- transition-timing-function：指定过渡效果的速度曲线。它可以是 `linear`（线性）、`ease`（渐入渐出）、`ease-in`（渐入）、`ease-out`（渐出）、`ease-in-out`（先渐入后渐出）等等。
- transition-delay：指定过渡效果开始之前的延迟时间，以秒或毫秒为单位。

#### 1.2 使用

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<style>
div
{
	width:100px;
	height:100px;
	background:red;
	transition:width 2s;
}

div:hover
{
	width:300px;
}
</style>
</head>
<body>
<p><b>注意：</b>该实例无法在 Internet Explorer 9 及更早 IE 版本上工作。</p>

<div></div>

<p>鼠标移动到 div 元素上，查看过渡效果。</p>

</body>
</html>
```

#### 1.3 语法

| 值                         | 描述                                       |
| :------------------------- | :----------------------------------------- |
| transition-property        | 指定CSS属性的name，transition效果          |
| transition-duration        | transition效果需要指定多少秒或毫秒才能完成 |
| transition-timing-function | 指定transition效果的转速曲线               |
| transition-delay           | 定义transition效果开始的时候               |