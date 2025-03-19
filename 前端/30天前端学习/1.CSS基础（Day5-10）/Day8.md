- Grid布局入门
- 媒体查询基础
- 实战：响应式相册

### 一、Grid布局入门

---

#### 1.1 grid基础概念

CSS Grid 布局是一种强大的二维布局系统，可以让你同时控制行和列。它非常适合构建复杂的网页布局，比如响应式页面、仪表板和照片画廊等。与 Flexbox 相比，Grid 更专注于整体页面结构的布局，而 Flexbox 则更适合单一轴向的对齐。

1. **网格容器**：`display: grid`
2. **网格项目**：容器的直接子元素
3. **网格轨道**：行/列（由`grid-template-rows/columns`定义）
4. **网格线**：轨道之间的分界线（可用数字或命名）
5. **网格区域**：由四条网格线围成的矩形空间

#### 1.2 容器属性详解

```css
.container {
  display: grid;
  
  /* 定义列轨道 */
  grid-template-columns: 200px 1fr minmax(300px, 1fr);
  
  /* 定义行轨道 */
  grid-template-rows: auto repeat(3, 100px);
  
  /* 显式网格间距 */
  gap: 20px 30px; /* row-gap column-gap */
  
  /* 隐式轨道尺寸 */
  grid-auto-rows: minmax(100px, auto);
  
  /* 排列方向 */
  grid-auto-flow: row dense; /* 填充空白 */
  
  /* 区域模板 */
  grid-template-areas:
    "header header"
    "sidebar main"
    "footer footer";
}
```

| 属性                    | 功能       | 常用值示例                            |
| :---------------------- | :--------- | :------------------------------------ |
| `grid-template-columns` | 定义列宽   | `repeat(4, 1fr)` `minmax(200px, 1fr)` |
| `grid-template-rows`    | 定义行高   | `auto` `200px`                        |
| `gap`                   | 网格间距   | `20px` `1rem`                         |
| `grid-auto-flow`        | 排列方向   | `row` `column` `dense`                |
| `grid-template-areas`   | 可视化布局 | 命名区域模板                          |

#### 1.3 项目属性详解

```css
.item {
  /* 位置控制 */
  grid-column: 1 / 3;       /* 起始线 / 结束线 */
  grid-row: span 2;         /* 跨越行数 */
  
  /* 区域定位 */
  grid-area: header;        /* 命名区域 */
  
  /* 对齐方式 */
  justify-self: stretch;    /* 水平对齐 */
  align-self: center;       /* 垂直对齐 */
}
```

>定位语法：
>
>- **数字定位**：`grid-column: 1 / 3`
>- **跨度定位**：`grid-row: 2 / span 3`
>- **命名定位**：`grid-column: col-start / col-end`

#### 1.4 响应式布局技巧

1. 自适应列数：

   ```css
   .card-grid {
     grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
   }
   ```

2. 媒体查询适配

   ```css
   .layout {
     grid-template-columns: 1fr;
   }
   
   @media (min-width: 768px) {
     .layout {
       grid-template-columns: 250px 1fr;
     }
   }
   ```

3. 动态间距

   ```css
   .container {
     gap: clamp(10px, 2vw, 20px);
   }

#### 1.5 实战：新闻门户布局

```html
<div class="new-portal">
    <header class="header">网站标题</header>
    <nav class="sidebar">导航菜单</nav>
    <main class="content">
        <div class="news-list">
            <article class="featured">头条新闻</article>
            <section class="news-list">新闻列表</section>
        </div>
    </main>
    <aside class="ad">广告位</aside>
    <footer class="footer">版权信息</footer>
</div>
```

```css
/* 定义新闻门户的整体布局 */
.new-portal {
    display: grid;
    min-height: 100vh;
    grid-template-columns: 250px 1fr 200px;
    grid-template-rows: auto 1fr auto;
    grid-template-areas:
        "header header header"
        "sidebar content ad"
        "footer footer footer";
    gap: 20px;

    background-color: #f4f4f4;
}

/* 设置头部样式 */
.header {
    grid-area: header;

    background-color: #333;
    color: white;
    padding: 20px;
    text-align: center;
    font-size: 24px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

/* 设置侧边栏样式 */
.sidebar {
    grid-area: sidebar;

    background-color: #e9ecef;
    padding: 20px;
    border-right: 1px solid #ddd;
}

/* 设置主要内容区域样式 */

.content {
    grid-area: content;
    display: grid;
    grid-template-rows: 300px auto;
    gap: 15px;

    padding: 20px;
    background-color: white;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

/* 设置广告区域样式 */
.ad {
    grid-area: ad;

    background-color: #e9ecef;
    padding: 20px;
    border-left: 1px solid #ddd;
}

/* 设置底部样式 */
.footer {
    grid-area: footer;

    background-color: #333;
    color: white;
    text-align: center;
    padding: 10px;
    box-shadow: 0 -2px 4px rgba(0, 0, 0, 0.1);
}

/* 响应式布局设置，当屏幕宽度小于1024px时应用以下样式 */
@media (max-width: 1024px) {
    .new-portal {
        grid-template-columns: 1fr;
        grid-template-areas:
            "header"
            "sidebar"
            "content"
            "ad"
            "footer";
    }

    /* 调整头部和底部样式以适应较小的屏幕 */
    .header,
    .footer {
        padding: 15px;
        font-size: 20px;
    }

    /* 调整侧边栏、内容和广告区域的样式以适应较小的屏幕 */
    .sidebar,
    .content,
    .ad {
        padding: 15px;
    }
}
```

#### 1.6 常见问题解决方案

1. 问题一：内容溢出网格单元

   ```css
   .item {
     min-width: 0; /* 允许缩小 */
     overflow: hidden;
   }
   ```

2. 问题二：隐式轨道尺寸不一致

   ```css
   .container {
     grid-auto-rows: 100px; /* 统一隐式行高 */
   }
   ```

3. 问题三：间距适配问题

   ```css
   .container {
     gap: 20px;
     margin: 0 -10px; /* 抵消边缘间距 */
   }
   ```

#### 1.7 Grid与Flex对比选择

| 特性         | Grid     | Flex     |
| :----------- | :------- | :------- |
| **维度**     | 二维布局 | 一维布局 |
| **对齐**     | 双向对齐 | 主轴对齐 |
| **空白处理** | 精准控制 | 自动分配 |
| **适用场景** | 复杂布局 | 线性排列 |

#### 1.8 日历视图组件

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>日历组件</title>
    <style>
        .calendar {
            --primary-color: #2196F3;
            --text-color: #333;
            --border-color: #eee;
            font-family: Arial, sans-serif;
            max-width: 600px;
            margin: 20px auto;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            border-radius: 8px;
            overflow: hidden;
        }

        /* 日历头部 */
        .calendar-header {
            background: var(--primary-color);
            color: white;
            padding: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .nav-button {
            background: none;
            border: none;
            color: white;
            font-size: 1.2rem;
            cursor: pointer;
            padding: 5px 15px;
            border-radius: 4px;
            transition: background 0.3s;
        }

        .nav-button:hover {
            background: rgba(255,255,255,0.1);
        }

        .month-year {
            font-size: 1.5rem;
        }

        /* 日历主体 */
        .calendar-body {
            padding: 20px;
            background: white;
        }

        .days-grid {
            display: grid;
            grid-template-columns: repeat(7, 1fr);
            gap: 2px;
        }

        .day-header {
            text-align: center;
            padding: 10px;
            font-weight: bold;
            color: var(--primary-color);
            border-bottom: 2px solid var(--border-color);
        }

        .day-cell {
            aspect-ratio: 1;
            padding: 10px;
            display: flex;
            flex-direction: column;
            align-items: center;
            cursor: pointer;
            background: white;
            border: 1px solid var(--border-color);
            transition: all 0.2s;
        }

        .day-cell:hover {
            background: #f5f5f5;
            transform: scale(1.05);
            z-index: 1;
        }

        .other-month {
            color: #999;
            background: #fafafa;
        }

        .today {
            background: #e3f2fd;
            font-weight: bold;
        }

        .event-marker {
            width: 6px;
            height: 6px;
            background: var(--primary-color);
            border-radius: 50%;
            margin-top: 4px;
        }
    </style>
</head>
<body>
    <div class="calendar">
        <div class="calendar-header">
            <button class="nav-button" id="prev-month">&lt;</button>
            <div class="month-year" id="current-month"></div>
            <button class="nav-button" id="next-month">&gt;</button>
        </div>
        <div class="calendar-body">
            <div class="days-grid" id="days-header"></div>
            <div class="days-grid" id="calendar-grid"></div>
        </div>
    </div>

    <script>
        class Calendar {
            constructor(options = {}) {
                this.container = document.querySelector(options.container || '.calendar');
                this.currentDate = options.initialDate || new Date();
                this.events = options.events || [];
                this.init();
            }

            init() {
                this.renderHeaders();
                this.renderCalendar();
                this.addEventListeners();
            }

            renderHeaders() {
                // 渲染星期标题
                const days = ['日', '一', '二', '三', '四', '五', '六'];
                document.getElementById('days-header').innerHTML = days
                    .map(day => `<div class="day-header">${day}</div>`)
                    .join('');

                // 更新月份显示
                document.getElementById('current-month').textContent = 
                    `${this.currentDate.getFullYear()}年 ${this.currentDate.getMonth() + 1}月`;
            }

            renderCalendar() {
                const grid = document.getElementById('calendar-grid');
                grid.innerHTML = '';
                
                const firstDay = new Date(
                    this.currentDate.getFullYear(),
                    this.currentDate.getMonth(),
                    1
                );
                
                const startDay = firstDay.getDay(); // 当月第一天星期几
                const lastDay = new Date(
                    this.currentDate.getFullYear(),
                    this.currentDate.getMonth() + 1,
                    0
                ).getDate(); // 当月总天数

                // 生成日期格子
                const cells = [];
                const today = new Date();

                // 填充上月天数
                for (let i = 0; i < startDay; i++) {
                    const date = new Date(firstDay);
                    date.setDate(date.getDate() - (startDay - i));
                    cells.push(this.createDayCell(date, true));
                }

                // 填充当月天数
                for (let i = 1; i <= lastDay; i++) {
                    const date = new Date(
                        this.currentDate.getFullYear(),
                        this.currentDate.getMonth(),
                        i
                    );
                    cells.push(this.createDayCell(date)));
                }

                // 填充下月天数（补满42格）
                let nextMonthDays = (42 - cells.length) % 7;
                if (nextMonthDays < 0) nextMonthDays += 7;
                for (let i = 1; i <= nextMonthDays; i++) {
                    const date = new Date(
                        this.currentDate.getFullYear(),
                        this.currentDate.getMonth() + 1,
                        i
                    );
                    cells.push(this.createDayCell(date, true));
                }

                grid.innerHTML = cells.join('');
            }

            createDayCell(date, isOtherMonth = false) {
                const isToday = this.isSameDate(date, new Date());
                const hasEvent = this.events.some(event => 
                    this.isSameDate(date, event.date));

                return `
                    <div class="day-cell 
                        ${isOtherMonth ? 'other-month' : ''} 
                        ${isToday ? 'today' : ''}"
                        data-date="${date.toISOString()}">
                        ${date.getDate()}
                        ${hasEvent ? '<div class="event-marker"></div>' : ''}
                    </div>
                `;
            }

            addEventListeners() {
                document.getElementById('prev-month').addEventListener('click', () => {
                    this.currentDate.setMonth(this.currentDate.getMonth() - 1);
                    this.update();
                });

                document.getElementById('next-month').addEventListener('click', () => {
                    this.currentDate.setMonth(this.currentDate.getMonth() + 1);
                    this.update();
                });

                document.getElementById('calendar-grid').addEventListener('click', (e) => {
                    if (e.target.closest('.day-cell')) {
                        const date = new Date(e.target.closest('.day-cell').dataset.date);
                        this.handleDateClick(date);
                    }
                });
            }

            update() {
                this.renderHeaders();
                this.renderCalendar();
            }

            // 实用方法
            isSameDate(date1, date2) {
                return date1.toDateString() === date2.toDateString();
            }

            // 可扩展的方法
            handleDateClick(date) {
                console.log('选中日期:', date.toLocaleDateString());
                // 在这里添加自定义点击逻辑
            }
        }

        // 初始化日历（示例事件数据）
        const calendar = new Calendar({
            events: [
                { date: new Date(2023, 6, 15), title: '项目会议' },
                { date: new Date(2023, 6, 20), title: '产品发布' }
            ]
        });
    </script>
</body>
</html>
```

1. **核心功能**：
   - 月视图展示（支持跨月显示）
   - 日期点击事件处理
   - 月份导航（上一月/下一月）
   - 今日高亮显示
   - 事件标记显示
2. **样式设计**：
   - 响应式网格布局（CSS Grid）
   - 平滑的过渡动画
   - 现代扁平化设计风格
   - 支持CSS自定义属性
   - 移动端友好
3. **扩展性**：
   - 支持自定义事件数据
   - 可配置的初始化参数
   - 易于扩展的类方法
   - 日期点击回调函数s



### 二、媒体查询基础

---

#### 2.1 核心语法

```css
@media 媒体类型 and (媒体特征规则) {
  /* 符合条件时生效的CSS规则 */
}
```

- **媒体类型**（可选）：
  `screen`（屏幕设备，最常用）、`print`（打印模式）、`all`（默认值，所有设备）。
- **媒体特征规则**：
  常用如 `min-width`、`max-width`（视口宽度）、`orientation: portrait`（竖屏）等。

#### 2.2 典型应用场景

1. 响应式布局

   ```css
   /* 视口≤750px时标题变红 */
   @media screen and (max-width: 750px) {
     h1 { color: red; }
   }
   ```

2. 多断点控制

   ```css
   /* 视口在700px~1200px之间时应用圆角 */
   @media (min-width: 700px) and (max-width: 1200px) {
     .box { border-radius: 50%; }
   }
   ```

3. 设备特性检测

   ```css
   /* 触屏设备禁用悬停效果 */
   @media (hover: none) {
     button { background: blue; }
   }
   ```

#### 2.3 实用技巧

- **移动优先原则**：先写小屏样式，再通过`min-width`逐步增强大屏样式3

- **单位选择**：推荐使用`vw`/`vh`（视口比例单位）或`rem`（根元素相对单位）4

- **外链样式表**：根据条件加载不同CSS文件

  ```html
  <link rel="stylesheet" media="(max-width:600px)" href="mobile.css">
  ```

#### 2.4 注意事项

- 媒体查询条件中`min-width`和`max-width`均包含等于的情况
- 多个条件用`and`连接，或逻辑用逗号分隔（如`@media (max-width:600px), print`）
- 现代浏览器已广泛支持媒体查询，但需注意旧版本兼容性13

通过合理使用媒体查询，可实现从手机到4K屏幕的全设备适配，是响应式设计的核心工具。建议结合Flexbox/Grid布局体系使用效果更佳。



### 三、实战：响应式相册

---

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>响应式相册</title>
    <style>
        * {
            box-sizing: border-box;
        }

        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #f0f0f0;
        }

        .album-cover {
            text-align: center;
            background-color: #ffffff;
            padding: 40px;
            border-radius: 15px;
            box-shadow: 0 12px 24px rgba(0, 0, 0, 0.1);
            transition: transform .2s ease-in-out;
            border: 2px solid #e0e0e0;
        }

        .album-cover:hover {
            transform: scale(1.05);
        }

        .album-cover h2 {
            font-size: 28px;
            color: #333;
            margin-bottom: 20px;
        }

        .album-cover button {
            background-color: #ff7e5f;
            color: white;
            border: none;
            padding: 12px 24px;
            font-size: 16px;
            border-radius: 5px;
            cursor: pointer;
            transition: background-color 0.3s ease;
        }

        .album-cover button:hover {
            background-color: #feb47b;
        }

        .gallery-container {
            position: relative;
            width: 100%;
            max-width: 600px;
            margin: auto;
        }

        .gallery img {
            width: 100%;
            height: auto;
            display: none;
            /* 默认隐藏所有图片 */
            border-radius: 10px;
            object-fit: cover;
            transition: opacity 0.5s ease-in-out;
            border: 5px solid #fff;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        }

        .gallery img.active {
            /* 当前显示的图片 */
            display: block;
            opacity: 1;
        }

        .gallery img:not(.active) {
            opacity: 0;
        }

        .prev,
        .next {
            cursor: pointer;
            position: absolute;
            top: 50%;
            width: 60px;
            /* 增加按钮宽度 */
            height: 60px;
            /* 增加按钮高度 */
            padding: 16px;
            /* 增加内边距 */
            margin-top: -30px;
            /* 调整垂直居中 */
            color: white;
            font-weight: bold;
            font-size: 24px;
            /* 增加字体大小 */
            transition: background-color 0.6s ease, transform 0.3s ease;
            border-radius: 50%;
            user-select: none;
            background: linear-gradient(135deg, #ff7e5f, #feb47b);
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
        }

        .next {
            right: 0;
        }

        .prev {
            left: 0;
        }

        .prev:hover,
        .next:hover {
            background: linear-gradient(135deg, #feb47b, #ff7e5f);
            transform: scale(1.1);
        }

        .gallery-wrapper {
            position: relative;
            width: 100%;
            max-width: 600px;
            margin: auto;
        }
    </style>
</head>

<body>
    <div class="album-cover" id="albumCover">
        <h2>相册</h2>
        <button onclick="toggleAlbum()">点击进入相册</button>
    </div>

    <div class="gallery-container" id="galleryContainer" style="display:none;">
        <div class="gallery-wrapper">
            <div class="gallery">
                <!-- 图片将通过JavaScript动态加载 -->
            </div>
            <button class="prev" onclick="changeImage(-1)">&#10094;</button>
            <button class="next" onclick="changeImage(1)">&#10095;</button>
        </div>
    </div>


    <script>
        let currentIndex = 0;
        let images = [];

        document.addEventListener('DOMContentLoaded', function () {
            loadImages();
        });

        function toggleAlbum() {
            const cover = document.getElementById('albumCover');
            const gallery = document.getElementById('galleryContainer');

            // 切换相册和封面的显示状态
            if (gallery.style.display === "none") {
                gallery.style.display = "block";
                cover.style.display = "none";
            } else {
                gallery.style.display = "none";
                cover.style.display = "block";
            }
        }

        async function loadImages() {
            const imgFolder = '../../img/baby/'; // 图片所在文件夹
            const gallery = document.querySelector('.gallery');

            if (!gallery) {
                console.error('Gallery element not found');
                return;
            }

            try {
                const response = await fetch(imgFolder);
                if (!response.ok) throw new Error('Network response was not ok');
                const data = await response.text();

                // 使用正则表达式匹配所有的图片文件名
                const imgUrls = data.match(/[\w-]+\.(jpg|jpeg|png|gif)/gi) || [];

                if (imgUrls.length === 0) {
                    console.warn('No images found in the gallery.');
                    return;
                }

                images = imgUrls.map(src => {
                    const imgElement = document.createElement('img');
                    imgElement.src = imgFolder + src;
                    imgElement.alt = 'Gallery Image';
                    gallery.appendChild(imgElement);

                    imgElement.onerror = function () {
                        console.error(`Failed to load image: ${imgFolder + src}`);
                    };

                    return imgElement;
                });

                if (images.length > 0) {
                    images[currentIndex].classList.add('active');
                }
            } catch (error) {
                console.error('There has been a problem with your fetch operation:', error);
            }
        }

        function changeImage(step) {
            if (images.length === 0) return;

            images[currentIndex].classList.remove('active');
            currentIndex += step;
            if (currentIndex >= images.length) currentIndex = 0;
            if (currentIndex < 0) currentIndex = images.length - 1;

            images[currentIndex].classList.add('active');
        }
    </script>
</body>

</html>
```

