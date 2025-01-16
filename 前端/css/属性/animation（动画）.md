### 一、animation

---

#### 1.1 定义

动画属性

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
	from {left:0px;}
	to {left:200px;}
}

</style>
</head>
<body>

<p><strong>注意: </strong> Internet Explorer 9 及更早IE版本不支持 animation 属性。</p>

<div></div>

</body>
</html>
```

#### 1.3 语法

| 值                        |                                                              |
| :------------------------ | :----------------------------------------------------------- |
| animation-name            | 指定要绑定到选择器的关键帧的名称                             |
| animation-duration        | 动画指定需要多少秒或毫秒完成                                 |
| animation-timing-function | 设置动画将如何完成一个周期                                   |
| animation-delay           | 设置动画在启动前的延迟间隔。                                 |
| animation-iteration-count | 定义动画的播放次数。                                         |
| animation-direction       | 指定是否应该轮流反向播放动画。                               |
| animation-fill-mode       | 规定当动画不播放时（当动画完成时，或当动画有一个延迟未开始播放时），要应用到元素的样式。 |
| animation-play-state      | 指定动画是否正在运行或已暂停。                               |
| initial                   | 设置属性为其默认值。 [阅读关于 *initial*的介绍。](https://www.runoob.com/cssref/css-initial.html) |
| inherit                   | 从父元素继承属性。 [阅读关于 *initinherital*的介绍。](https://www.runoob.com/cssref/css-inherit.html) |