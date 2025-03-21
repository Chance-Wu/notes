### 一、属性定义及使用说明

---

#### 1.1 定义

Transform属性应用于元素的2D或3D转换。这个属性允许你将元素**旋转**，**缩放**，**移动**，**倾斜**等。

#### 1.2 使用

```html
<html><!DOCTYPE html>
<html>
<head>
<style> 
#div1
{
width:120px;
height:100px;
background-color:yellow;
border:1px solid black;
transform:rotate(7deg);
}
</style>

<script>
function rotate(value)
{
document.getElementById('div1').style.transform="rotate(" + value + "deg)";
document.getElementById('span1').innerHTML=value + "deg";
}
</script>

</head>
<body>

<p>Rotate the yellow div element:</p>

<div id="div1">HELLO</div>

Rotate: <br>
<input type="range" min="-360" max="360" value="7" onchange="rotate(this.value)" /><br>
transform: rotate(<span id="span1">7deg</span>);

</body>
</html>
```



### 二、语法

---

| 值                                                           | 描述                                    |
| :----------------------------------------------------------- | :-------------------------------------- |
| none                                                         | 定义不进行转换。                        |
| matrix(*n*,*n*,*n*,*n*,*n*,*n*)                              | 定义 2D 转换，使用六个值的矩阵。        |
| matrix3d(*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*,*n*) | 定义 3D 转换，使用 16 个值的 4x4 矩阵。 |
| translate(*x*,*y*)                                           | 定义 2D 转换。                          |
| translate3d(*x*,*y*,*z*)                                     | 定义 3D 转换。                          |
| **translateX(*x*)**                                          | 定义转换，只是用 X 轴的值。             |
| **translateY(*y*)**                                          | 定义转换，只是用 Y 轴的值。             |
| translateZ(*z*)                                              | 定义 3D 转换，只是用 Z 轴的值。         |
| scale(*x*[,*y*]?)                                            | 定义 2D 缩放转换。                      |
| scale3d(*x*,*y*,*z*)                                         | 定义 3D 缩放转换。                      |
| scaleX(*x*)                                                  | 通过设置 X 轴的值来定义缩放转换。       |
| scaleY(*y*)                                                  | 通过设置 Y 轴的值来定义缩放转换。       |
| scaleZ(*z*)                                                  | 通过设置 Z 轴的值来定义 3D 缩放转换。   |
| **rotate(*angle*)**                                          | 定义 2D 旋转，在参数中规定角度。        |
| rotate3d(*x*,*y*,*z*,*angle*)                                | 定义 3D 旋转。                          |
| rotateX(*angle*)                                             | 定义沿着 X 轴的 3D 旋转。               |
| rotateY(*angle*)                                             | 定义沿着 Y 轴的 3D 旋转。               |
| rotateZ(*angle*)                                             | 定义沿着 Z 轴的 3D 旋转。               |
| skew(*x-angle*,*y-angle*)                                    | 定义沿着 X 和 Y 轴的 2D 倾斜转换。      |
| skewX(*angle*)                                               | 定义沿着 X 轴的 2D 倾斜转换。           |
| skewY(*angle*)                                               | 定义沿着 Y 轴的 2D 倾斜转换。           |
| perspective(*n*)                                             | 为 3D 转换元素定义透视视图。            |