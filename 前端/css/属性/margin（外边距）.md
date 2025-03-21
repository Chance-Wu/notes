### 一、margin

#### 1.1 定义

margin简写属性在一个声明中设置所有**外边距**属性。该属性可以有1到4个值。（负值是允许的）

#### 1.2 使用

```
margin:10px 5px 15px 20px;
上边距是 10px
右边距是 5px
下边距是 15px
左边距是 20px

margin:10px 5px 15px;
上边距是 10px
右边距和左边距是 5px
下边距是 15px

margin:10px 5px;
上边距和下边距是 10px
右边距和左边距是 5px

margin:10px;
所有四个边距都是 10px
```

#### 1.3 属性值

| 值       | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| auto     | 浏览器计算外边距。                                           |
| *length* | 规定以具体单位计的外边距值，比如像素、厘米等。默认值是 0px。 |
| *%*      | 规定基于父元素的宽度的百分比的外边距。                       |
| inherit  | 规定应该从父元素继承外边距。                                 |



### 二、margin-top

---

#### 2.1 定义

margin-top 属性设置元素的**上部边距**。（负值是允许的）

#### 2.2 使用

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<style>
p.ex1 {margin-top:2cm;}
</style>
</head>

<body>

<p>一个没有指定边距大小的段落。</p>
<p class="ex1">一个2厘米上边距的段落。</p>
<p>一个没有指定边距大小的段落。</p>

</body>
</html>
```

#### 2.3 属性值

| 值       | 描述                                   |
| :------- | :------------------------------------- |
| auto     | 浏览器设置的上外边距。                 |
| *length* | 定义固定的上外边距。默认值是 0。       |
| *%*      | 定义基于父对象总宽度的百分比上外边距。 |
| inherit  | 规定应该从父元素继承上外边距。         |



### 三、margin-bottom

元素下边距



### 四、margin-left

元素左边距



### 五、margin-right

元素右边距

