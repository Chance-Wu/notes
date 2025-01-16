### 一、@keyframes

---

#### 1.1 定义

`@keyframes` 是 CSS 中用于定义动画的关键帧规则。通过 @keyframes，可以创建一个动画序列，指定动画开始时、进行中和结束时的样式变化。每个关键帧使用百分比来表示动画的特定时刻，`0%` 表示动画的开始（也可以用 `from` 关键字），`100%` 表示动画的结束（也可以用 `to` 关键字）。你可以在这些关键帧之间定义任意数量的中间状态。

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
	position:relative;
	animation:mymove 5s infinite;
}

@keyframes mymove
{
	from {top:0px;}
	to {top:200px;}
}

</style>
</head>
<body>

<p><strong>注意:</strong>  @keyframes 规则 不兼容 IE 9 以及更早版本的浏览器.</p>

<div></div>

</body>
</html>
```

#### 1.3 语法

```css
@keyframes animationname {keyframes-selector {css-styles;}}
```

| 值                   | 说明                                                         |
| :------------------- | :----------------------------------------------------------- |
| *animationname*      | 必需的。定义animation的名称。                                |
| *keyframes-selector* | 必需的。动画持续时间的百分比。合法值：0-100% from (和0%相同) to (和100%相同)**注意：** 您可以用一个动画keyframes-selectors。 |
| *css-styles*         | 必需的。一个或多个合法的CSS样式属性                          |