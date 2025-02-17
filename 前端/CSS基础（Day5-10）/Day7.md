- Flex布局基础
- 定位机制
- 实战：导航栏+卡片布局

### 一、Flex布局基础

---

#### 1.1 Flex布局模型

- Flex容器：设置 `display：felx` 的元素
- Flex项目：容器的直接子元素
- 主轴（Main Axis）：项目的排列方式
- 交叉轴（Cross Axis）：与主轴垂直的方向

![Flex轴示意图](./img/00-basic-terminology.svg)

#### 1.2 容器属性详解

```css
.container {
  /* 定义Flex容器 */
  display: flex; 
  
  /* 主轴方向 */
  flex-direction: row | row-reverse | column | column-reverse;
  
  /* 主轴对齐 */
  justify-content: flex-start | flex-end | center | space-between | space-around | space-evenly;
  
  /* 交叉轴对齐 */
  align-items: stretch | flex-start | flex-end | center | baseline;
  
  /* 多行排列 */
  flex-wrap: nowrap | wrap | wrap-reverse;
  
  /* 多行对齐 */
  align-content: stretch | flex-start | flex-end | center | space-between | space-around;
  
  /* 复合属性 */
  flex-flow: column wrap; /* flex-direction + flex-wrap */
}
```

| 属性              | 典型用例         |
| :---------------- | :--------------- |
| `flex-direction`  | 移动端竖屏布局   |
| `justify-content` | 导航栏项目分布   |
| `align-items`     | 垂直居中内容     |
| `flex-wrap`       | **响应式图片墙** |

#### 1.3 项目属性详解

```css
.item {
  /* 排列顺序 */
  order: 5; /* 默认0，数值越小越靠前 */
  
  /* 放大比例 */
  flex-grow: 2; /* 默认0 */
  
  /* 缩小比例 */
  flex-shrink: 1; /* 默认1 */
  
  /* 初始尺寸 */
  flex-basis: 200px | auto; /* 默认auto */
  
  /* 简写属性 */
  flex: 1 1 200px; /* flex-grow | flex-shrink | flex-basis */
  
  /* 单个项目对齐 */
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

> **伸缩比例计算**：
>
> - 剩余空间分配：`flex-grow`值比例分配
> - 空间不足压缩：`flex-shrink`值比例收缩

#### 1.4 典型布局案例

1. 经典导航栏

   ```css
   .nav {
     display: flex;
     justify-content: space-between;
     align-items: center;
     padding: 1rem;
   }
   
   .logo { margin-right: auto; }
   
   .menu-item {
     padding: 0 1rem;
     flex: 0 1 auto;
   }
   ```

2. 等高卡片布局

   ```css
   .card-container {
     display: flex;
     gap: 20px; /* 项目间距 */
   }
   
   .card {
     flex: 1; /* 等分容器 */
     min-width: 300px; /* 响应式断点 */
   }
   ```

3. 圣杯布局

   ```css
   .layout {
     display: flex;
     flex-direction: column;
     min-height: 100vh;
   }
   
   .main-content {
     flex: 1; /* 占据剩余空间 */
     display: flex;
   }
   
   .sidebar {
     flex: 0 0 250px;
     order: -1;
   }
   ```

#### 1.5 响应式技巧

```css
/* 移动端优先 */
.product-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.product-card {
  flex: 1 1 300px; 
  /* 基础300px，可伸缩，可换行 */
}

@media (min-width: 768px) {
  .product-card {
    flex-basis: calc(50% - 1rem);
  }
}

@media (min-width: 1200px) {
  .product-card {
    flex-basis: calc(33.33% - 1rem);
  }
}
```

#### 1.6 常见错误排查

1. 项目不换行

   ```css
   .container {
     flex-wrap: wrap; /* 需要设置允许换行 */
     width: 100%; /* 确保容器有明确宽度 */
   }
   ```

2. 间距不一致

   ```css
   /* 使用gap属性替代margin */
   .container {
     gap: 20px; /* 现代浏览器支持 */
   }

3. 内容溢出

   ```css
   .item {
     flex-shrink: 1; /* 允许缩小 */
     min-width: 200px; /* 设置最小限制 */
   }
   ```

#### 1.7 浏览器支持与优化

1. 兼容性处理

   ```css
   .container {
     display: -webkit-flex; /* 旧版Safari */
     display: flex;
   }
   ```

2. 性能优化

   - 避免过度嵌套Flex容器
   - 对大量项目使用`will-change: transform`
   - 优先使用`gap`替代margin控制间距



### 二、定位机制

---

#### 2.1 定位体系概述

**定位类型**：

1. **静态定位**：`position: static`（默认）
2. **相对定位**：`position: relative`
3. **绝对定位**：`position: absolute`
4. **固定定位**：`position: fixed`
5. **粘性定位**：`position: sticky`

**坐标系参考**：

- **视口**：浏览器可视区域（fixed定位）
- **最近定位祖先**（absolute定位）
- **文档流原位置**（relative定位）

#### 2.2 相对定位

```css
.box {
  position: relative;
  top: 20px;
  left: 30px;
  z-index: 1;
}
```

- 相对于元素原始位置偏移
- 保留原文档流空间
- 常用作绝对定位的参照容器

**应用场景**：

- 微调元素位置
- 创建层叠效果
- 建立定位上下文

#### 2.3 绝对定位

```css
.modal {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

- 脱离文档流
- 相对于最近非static定位祖先定位
- 需配合`top/right/bottom/left`使用

**应用场景**：

- 模态对话框
- 下拉菜单
- 自定义提示框

#### 2.4 固定定位

```css
.header {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  z-index: 100;
}
```

- 相对于浏览器视口定位
- 滚动时不移动
- 需要处理内容遮挡（预留padding）

**应用场景**：

- 固定导航栏
- 悬浮按钮
- 广告横幅

#### 2.5 粘性定位

```css
.table-header {
  position: sticky;
  top: 20px;
  background: white;
}
```

- 混合定位（relative + fixed）
- 需要指定阈值（如top值）
- 父容器不能有`overflow:hidden`

**典型用例**：

- 固定表格头
- 分页导航停留
- 侧边栏跟随

#### 2.6 层叠上下文

控制属性：

```css
.element {
  z-index: 10; /* 仅在同级层叠上下文中比较 */
}
```

**创建层叠上下文的条件**：

1. 根元素（HTML）
2. position非static且z-index非auto
3. opacity < 1
4. transform/filter属性
5. flex/grid容器的子项且z-index非auto

#### 2.7 实战：电商商品卡片

```html
<article class="product-card">
  <span class="badge">新品</span>
  <img class="product-image" src="product.jpg" alt="商品图">
  <div class="info">
    <h3 class="title">无线蓝牙耳机</h3>
    <p class="price">￥299</p>
    <button class="cart-btn">加入购物车</button>
  </div>
</article>
```

```css
.product-card {
  position: relative; /* 建立定位上下文 */
  width: 300px;
  border-radius: 8px;
  overflow: hidden;
}

.badge {
  position: absolute;
  top: 10px;
  right: -30px;
  background: #e74c3c;
  color: white;
  padding: 5px 30px;
  transform: rotate(45deg);
  z-index: 2;
}

.product-image {
  position: relative;
  z-index: 1;
  transition: transform 0.3s;
}

.product-card:hover .product-image {
  transform: scale(1.05);
}

.cart-btn {
  position: absolute;
  bottom: -40px;
  left: 50%;
  transform: translateX(-50%);
  opacity: 0;
  transition: all 0.3s;
}

.product-card:hover .cart-btn {
  bottom: 20px;
  opacity: 1;
}
```

#### 2.8 定位常见问题解决方案

1. 绝对定位元素溢出父容器

   ```css
   .parent {
     position: relative;
     overflow: visible; /* 覆盖默认hidden */
   }
   ```

2. 移动端fixe定位抖动

   ```css
   .fixed-element {
     position: fixed;
     -webkit-overflow-scrolling: touch; /* iOS优化 */
     backface-visibility: hidden;
   }
   ```

3. 粘性定位失效排查

   ```css
   .sticky-element {
     position: sticky;
     top: 0;
     /* 确保父容器：
        1. 高度足够
        2. 没有overflow:hidden
        3. 不是flex/grid容器（部分浏览器限制） */
   }
   ```

#### 2.9 响应式定位技巧

1. 媒体查询调整定位

   ```css
   .sidebar {
     position: static;
   }
   
   @media (min-width: 1024px) {
     .sidebar {
       position: sticky;
       top: 20px;
     }
   }
   ```

2. 动态定位切换

   ```css
   function checkPosition() {
     const element = document.querySelector('.banner');
     if (window.innerWidth < 768) {
       element.style.position = 'static';
     } else {
       element.style.position = 'fixed';
     }
   }
   ```

3. 安全区域适配

   ```css
   .fixed-bottom {
     bottom: max(20px, env(safe-area-inset-bottom));
   }



### 三、实战：导航栏+卡片布局

---

```html
<!-- 导航栏 -->
<nav class="navbar">
    <ul>
        <li><a href="#">首页</a></li>
        <li><a href="#">关于</a></li>
        <li><a href="#">服务</a></li>
        <li><a href="#">博客</a></li>
        <li><a href="#">联系我们</a></li>
    </ul>
</nav>

<!-- 卡片布局 -->
<div class="cards-container">
    <div class="card">
        <img src="https://via.placeholder.com/400x200" alt="示例图片1">
        <div class="card-content">
            <h3>卡片标题 1</h3>
            <p>这是一段示例描述，简单介绍此卡片所表达的内容和信息。</p>
        </div>
    </div>
    <div class="card">
        <img src="https://via.placeholder.com/400x200" alt="示例图片2">
        <div class="card-content">
            <h3>卡片标题 2</h3>
            <p>这是一段示例描述，简单介绍此卡片所表达的内容和信息。</p>
        </div>
    </div>
    <div class="card">
        <img src="https://via.placeholder.com/400x200" alt="示例图片3">
        <div class="card-content">
            <h3>卡片标题 3</h3>
            <p>这是一段示例描述，简单介绍此卡片所表达的内容和信息。</p>
        </div>
    </div>
    <div class="card">
        <img src="https://via.placeholder.com/400x200" alt="示例图片4">
        <div class="card-content">
            <h3>卡片标题 4</h3>
            <p>这是一段示例描述，简单介绍此卡片所表达的内容和信息。</p>
        </div>
    </div>
    <!-- 根据需要，可继续添加更多卡片 -->
</div>
```

```css
/* 重置部分默认样式 */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: "Arial", sans-serif;
    background-color: #f4f4f4;
    color: #333;
    line-height: 1.6;
}

/* 导航栏样式 */
.navbar {
    background-color: #333;
    padding: 15px;
}

.navbar ul {
    list-style: none;
    display: flex;
    justify-content: center;
}

.navbar li {
    margin: 0 20px;
}

.navbar a {
    color: #fff;
    text-decoration: none;
    font-size: 1.1rem;
    transition: color 0.3s;
}

.navbar a:hover {
    color: #ddd;
}

/* 卡片布局容器 */
.cards-container {
    max-width: 1200px;
    margin: 30px auto;
    padding: 0 15px;
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 20px;
}

/* 卡片样式 */
.card {
    background-color: #fff;
    border-radius: 8px;
    overflow: hidden;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
    transition: transform 0.3s, box-shadow 0.3s;
}

.card:hover {
    transform: translateY(-5px);
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
}

.card img {
    width: 100%;
    display: block;
}

.card-content {
    padding: 15px;
}

.card-content h3 {
    font-size: 1.3rem;
    margin-bottom: 10px;
}

.card-content p {
    font-size: 1rem;
    line-height: 1.5;
}
```

1. **导航栏**

   - 使用 `.navbar` 类设置背景色为深灰色，并使用 Flexbox 居中排列导航链接。

   - 每个链接都设置了悬停效果，颜色从白色变为浅灰色。

2. **卡片布局**

   - `.cards-container` 使用 CSS Grid 布局，`grid-template-columns` 使用 `repeat(auto-fit, minmax(280px, 1fr))` 实现响应式排列。

   - 每个 `.card` 设置背景色为白色、圆角和盒子阴影，同时添加了悬停动画效果，使卡片在鼠标悬停时稍微上浮。

3. **卡片内容**
   - 每个卡片内包含一张图片和一段文字。图片宽度设为 100% 保证自适应，卡片内的文本内容通过 `.card-content` 类设置内边距，并分开标题与描述。