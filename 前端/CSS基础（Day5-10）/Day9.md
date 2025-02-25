### 一、过渡与动画

---

#### 1.1 过渡（Transition）

实现元素在不同状态（如悬停、点击）间的平滑变化效果。

```css
transition: property duration timing-function delay;
/* 示例：所有属性变化在0.3秒内以缓入方式过渡 */
transition: all 0.3s ease-in;
```

核心属性：

- **property**：参与过渡的属性（如 `width`, `opacity`，或 `all`）
- **duration**：过渡时间（如 `1s`）
- **timing-function**：过渡的时间函数，定义过渡的速度曲线（如 `ease`, `linear`, `cubic-bezier()`）
- **delay**：延迟时间（如 `0.2s`）

>适用场景：
>
>- 按钮悬停放大：`transform: scale(1.1);`
>- 颜色渐变：背景色/文字色变化
>- 显示隐藏元素：结合 `opacity` 和 `visibility`

#### 1.2 动画（Animation）

通过关键帧实现复杂、多阶段动态效果。

1. 定义关键帧

   ```css
   @keyframes rotate {
     from { transform: rotate(0); }
     to { transform: rotate(360deg); }
   }
   /* 或百分比定义多阶段 */
   @keyframes move {
     0% { left: 0; }
     50% { left: 200px; opacity: 0.5; }
     100% { left: 0; }
   }
   ```

2. 调用动画

   ```css
   animation: name duration timing-function delay iteration-count direction fill-mode;
   /* 示例：无限次旋转，交替方向 */
   animation: rotate 2s linear infinite alternate;
   ```

   - **iteration-count**：执行次数（`infinite` 表示无限循环）
   - **direction**：播放方向（`alternate` 实现来回效果）
   - **fill-mode**：结束状态（`forwards` 保留最后一帧）

> 适用场景
>
> - 加载旋转图标
> - 复杂路径运动（如弹跳球）
> - 序列帧动画

#### 1.4 实战

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>过渡与动画</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        .box {
            display: flex;
            grid-template-rows: repeat(3, 1fr);
            justify-content: center;
            gap: 80px;
            margin: 100px ;
            justify-self: center;
            
        }

        /* 悬停放大按钮 */
        .hover-button {
            transition: transform 0.3s ease;
        }

        .hover-button:hover {
            transform: scale(1.1);
        }

        /* 加载旋转动画 */
        @keyframes spin {
            to {
                transform: rotate(360deg);
            }
        }

        .loading-spinner {
            animation: spin 60s linear infinite;
            color: #3498db;
        }


        /* 3D卡片翻转 */
        .flip-card {
            transition: transform 1.8s;
            transform-style: preserve-3d;
        }

        .flip-card:hover {
            transform: rotateY(180deg);
        }
    </style>
</head>

<body>
    <div class="box">
        <button class="hover-button">Hover Me</button>
        <div class="loading-spinner">loading-spinner</div>
        <div class="flip-card">Flip Card</div>
    </div>
</body>

</html>
```



### 二、变换效果

---

#### 2.1 旋转（rotate）

`rotate` 用于旋转元素，单位为角度（deg）。

```css
transform: rotate(45deg); /* 顺时针旋转 45 度 */
```

#### 2.2 平移（translate）

`translate` 用于平移元素，接受两个参数，分别表示水平方向和垂直方向的位移。

```css
transform: translate(50px, 100px); /* 水平平移 50px，垂直平移 100px */
```

如果只指定一个值，另一个方向的位移默认为 0。

```css
transform: translate(50px); /* 水平平移 50px，垂直平移 0px */
```

#### 2.3 缩放（scale）

`scale` 用于缩放元素，接受一个或两个参数。

- 一个参数：在 X 和 Y 轴上等比例缩放。

  ```css
  transform: scale(1.5); /* 放大 1.5 倍 */
  ```

- 两个参数：分别表示 X 和 Y 轴的缩放比例。

  ```css
  transform: scale(1.5, 2); /* 水平放大 1.5 倍，垂直放大 2 倍 */
  ```

#### 2.4 倾斜（skew）

`skew` 用于倾斜元素，接受一个或两个角度值，分别表示 X 和 Y 轴的倾斜角度。

```css
transform: skew(30deg, 20deg); /* X 轴倾斜 30 度，Y 轴倾斜 20 度 */
```

如果只指定一个角度，另一个方向的倾斜角度默认为 0。

```css
transform: skew(30deg); /* X 轴倾斜 30 度，Y 轴倾斜 0 度 */
```

#### 2.5 组合变换

多个变换可以组合使用，按从右到左的顺序依次应用。

```css
transform: rotate(45deg) translate(50px, 100px) scale(1.5);
```

在上述示例中，元素首先被旋转 45 度，然后平移 50px 和 100px，最后缩放 1.5 倍。

#### 2.6 变换原点（transform-origin）

`transform-origin` 用于设置变换的原点，默认值为元素的中心。

```css
transform-origin: top left; /* 以元素的左上角为原点 */
```

你可以使用百分比或长度值来精确控制原点的位置。

#### 2.7 3D变换

CSS还支持3D变换，通过设置`perspective` 属性来定义视距，从而创建深度效果。

```css
perspective: 500px; /* 设置视距为 500px */
```

然后，你可以使用 `rotateX`、`rotateY` 和 `translateZ` 等属性来进行 3D 变换。

```css
transform: rotateY(45deg) translateZ(100px);
```

需要注意的是，3D 变换的效果在不同浏览器和设备上的支持程度可能有所差异，使用时需进行兼容性测试。

#### 2.8 卡片翻转

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>卡片翻转效果</title>
    <style>
        .card-container {
            perspective: 1000px;
            /* 设置视距 */
            width: 200px;
            height: 300px;
            margin: 50px auto;
        }

        .card {
            width: 100%;
            height: 100%;
            transform-style: preserve-3d;
            /* 保持 3D 变换 */
            transition: transform 1.6s;
        }

        .card:hover {
            transform: rotateY(180deg);
            /* 鼠标悬停时旋转 180 度 */
        }

        .card .front,
        .card .back {
            align-content: center;
            position: absolute;
            width: 100%;
            height: 100%;
            backface-visibility: hidden;
            /* 隐藏背面 */
        }

        h2 {
            text-align: center;
        }

        .card .front {
            background: lightblue;
        }

        .card .back {
            background: lightcoral;
            transform: rotateY(180deg);
            /* 背面旋转 180 度 */
        }
    </style>
</head>

<body>
    <div class="card-container">
        <div class="card">
            <div class="front">
                <h2>正面</h2>
            </div>
            <div class="back">
                <h2>背面</h2>
            </div>
        </div>
    </div>
</body>

</html>
```



### 三、制作加载动画

---

#### 3.1 旋转圆环加载动画（经典样式）

旋转圆环是最常见的加载动画之一，通常由一个圆形元素组成，通过旋转来标识加载过程。

```html
<div class="loader"></div>

<style>
.loader {
  width: 50px;
  height: 50px;
  border: 5px solid #f3f3f3;
  border-top: 5px solid #3498db; /* 修改颜色可改变主题色 */
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>

```

> `.loader`元素通过`@keyframes`定义了一个旋转动画，使其持续旋转，表示加载过程。

#### 3.2 点状加载动画（适合轻量级场景）

点点跳动的加载动画通过多个小圆点的缩放效果，模拟加载过程。

```html
<div class="dot-loader">
  <div class="dot"></div>
  <div class="dot"></div>
  <div class="dot"></div>
</div>

<style>
.dot-loader {
  display: flex;
  gap: 8px;
}

.dot {
  width: 12px;
  height: 12px;
  background: #2ecc71; /* 修改颜色 */
  border-radius: 50%;
  animation: bounce 1.4s infinite ease-in-out;
}

.dot:nth-child(2) { animation-delay: 0.2s; }
.dot:nth-child(3) { animation-delay: 0.4s; }

@keyframes bounce {
  0%, 80%, 100% { transform: translateY(0); }
  40% { transform: translateY(-20px); }
}
</style>
```

>三个小圆点通过不同的动画延迟，依次进行缩放，模拟跳动效果。

#### 3.3 进度条加载动画（带百分比）

线条加载动画通过水平线条的宽度变化，表示加载进度。

```html
<div class="progress-loader">
  <div class="progress-bar"></div>
</div>

<style>
.progress-loader {
  width: 200px;
  height: 4px;
  background: #eee;
  border-radius: 2px;
  overflow: hidden;
}

.progress-bar {
  width: 0;
  height: 100%;
  background: #e74c3c;
  animation: progress 2s ease-in-out infinite;
}

@keyframes progress {
  0% { width: 0; }
  50% { width: 70%; }
  100% { width: 100%; }
}
</style>
```

#### 3.4 现代感旋转边框（适合科技风格）

































































