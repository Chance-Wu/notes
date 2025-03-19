- 常用属性（color/font/background）
- 文本样式与排版
- 实战：新闻文章排版

### 一、常用属性

---

#### 1.1 color文本颜色

1. 基础颜色设置

   ```css
   .text {
     color: red;                  /* 颜色名称 */
     color: #ff0000;              /* 十六进制 */
     color: rgb(255, 0, 0);       /* RGB */
     color: rgba(255, 0, 0, 0.5); /* 带透明度的RGB */
     color: hsl(0, 100%, 50%);    /* 色相-饱和度-明度 */
   }
   ```

2. 高级技巧

   - 透明度控制

     ```css
     .overlay {
       color: #ff000080; /* 十六进制带透明度 */
       background-color: hsl(120, 100%, 50%, 0.3);
     }
     ```

   - 继承与动态颜色

     ```css
     .icon {
       border: 2px solid currentColor; /* 继承文字颜色 */
       fill: currentColor;            /* 适用于SVG */
     }
     ```

#### 1.2 font字体属性

1. 复合属性（推荐顺序）

   ```css
   .title {
     font: italic small-caps bold 24px/1.2 "Helvetica Neue", sans-serif;
   }
   /* 等效于 */
   .title {
     font-style: italic;
     font-variant: small-caps;
     font-weight: bold;
     font-size: 24px;
     line-height: 1.2;
     font-family: "Helvetica Neue", sans-serif;
   }
   ```

2. 分项属性详解

   | 属性           | 值示例                               | 说明                             |
   | :------------- | :----------------------------------- | :------------------------------- |
   | `font-family`  | `'Microsoft YaHei', sans-serif`      | 字体栈（优先使用第一个可用字体） |
   | `font-size`    | `1.2rem`, `clamp(1rem, 2vw, 1.5rem)` | 推荐使用相对单位                 |
   | `font-weight`  | `300` (细体), `600` (半粗)           | 数字范围100-900                  |
   | `font-style`   | `italic`, `oblique 15deg`            | 斜体样式                         |
   | `font-variant` | `small-caps`, `tabular-nums`         | 小型大写字母/等宽数字            |

3. 字体加载优化

   ```css
   @font-face {
     font-family: 'CustomFont';
     src: url('font.woff2') format('woff2'),
          url('font.woff') format('woff');
     font-display: swap; /* 显示回退字体直到自定义字体加载 */
   }

#### 1.3 backgroud背景属性

1. 复合属性（推荐顺序）

   ```css
   .hero {
     background: 
       linear-gradient(rgba(0,0,0,0.5), 
       url("banner.jpg") no-repeat center / cover,
       #f0f0f0;
   }
   /* 等效于 */
   .hero {
     background-color: #f0f0f0;
     background-image: linear-gradient(rgba(0,0,0,0.5)), url("banner.jpg");
     background-repeat: no-repeat;
     background-position: center;
     background-size: cover;
   }
   ```

2. 分项属性详解

   | 属性                    | 值示例                                   | 说明         |
   | :---------------------- | :--------------------------------------- | :----------- |
   | `background-color`      | `transparent` (透明)                     | 纯色背景     |
   | `background-image`      | `url("img.png")`, `linear-gradient(...)` | 支持多背景图 |
   | `background-repeat`     | `repeat-x`, `space` (等间距平铺)         | 平铺方式     |
   | `background-position`   | `right 10px bottom 20%`                  | 精确控制位置 |
   | `background-size`       | `contain` (完整显示), `100px auto`       | 尺寸控制     |
   | `background-attachment` | `fixed` (固定背景)                       | 滚动效果     |
   | `background-clip`       | `text` (文字裁剪背景)                    | 高级效果     |

3. 渐变背景示例

   ```css
   .button {
     background: linear-gradient(
       135deg,
       #ff6b6b 0%,
       #ff8e8e 25%,
       #ffaaaa 50%,
       #ff8e8e 75%,
       #ff6b6b 100%
     );
   }
   ```

4. 多背景图应用

   ```css
   .card {
     background: 
       url("corner-deco.png") left top no-repeat,
       url("corner-deco.png") right bottom no-repeat,
       linear-gradient(to bottom right, #fff, #f0f0f0);
   }
   ```

#### 1.4 实战：卡片组件

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="卡片组件">
    <title>卡片组件</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #f8f9fa;
        }

        .card {
            width: 300px;
            border-radius: 10px;
            overflow: hidden;
            box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
            transition: transform 0.3s ease, box-shadow 0.3s ease;
            background-color: #ffffff;
        }

        .card:hover {
            transform: translateY(-10px);
            box-shadow: 0 6px 15px rgba(0, 0, 0, 0.2);
        }

        .card-image {
            width: 100%;
            height: 150px;
            object-fit: cover;
        }

        .card-body {
            padding: 1.5rem;
        }

        .card-title {
            font-size: 1.5rem;
            margin-bottom: 0.5rem;
            color: #2c3e50;
        }

        .card-content {
            font-size: 1rem;
            line-height: 1.6;
            color: #7f8c8d;
            margin-bottom: 1rem;
        }

        .card-button {
            display: inline-block;
            padding: 0.5rem 1rem;
            background-color: #3498db;
            color: #ffffff;
            text-decoration: none;
            border-radius: 5px;
            transition: background-color 0.3s ease;
        }

        .card-button:hover {
            background-color: #2980b9;
        }
    </style>
</head>

<body>
    <div class="card">
        <img src="../../img/texture.png" alt="卡片图片" class="card-image">
        <div class="card-body">
            <h2 class="card-title">卡片标题</h2>
            <p class="card-content">这是一段简单的卡片内容，用于展示卡片的基本功能。</p>
            <a href="#" class="card-button">了解更多</a>
        </div>
    </div>
</body>

</html>
```

#### 1.5 常见问题与解决方案

1. 字体加载闪烁

   ```css
   body {
     font-family: system-ui; /* 系统默认字体作为回退 */
   }
   .loaded-font {
     font-family: 'CustomFont', system-ui;
   }
   ```

   ```javascript
   document.fonts.load('1em CustomFont').then(() => {
     document.body.classList.add('loaded-font');
   });
   ```

2. 背景渐变兼容

   ```css
   .background {
     background: #ff6b6b; /* 回退色 */
     background: linear-gradient(to right, #ff6b6b, #ff8e8e);
   }

3. 文字抗锯齿优化

   ```css
   .text {
     -webkit-font-smoothing: antialiased; /* Mac */
     -moz-osx-font-smoothing: grayscale;
   }
   ```



### 二、文本样式与排版

---

#### 2.1 基础文本样式

- **字体与字号**

  - `font-family`：指定文本所使用的字体。常见的做法是设置一组字体（字体栈），如：

    ```css
    body {
      font-family: "Helvetica", "Arial", sans-serif;
    }
    ```

    这样，如果用户设备上没有安装首选字体，浏览器会依次使用后备字体。

  - `font-size`：用于设置文字大小，可以使用绝对单位（如 px）或相对单位（如 em、rem）。例如：

    ```css
    p {
      font-size: 16px;
    }
    h1 {
      font-size: 2rem; /* 基于根元素的字体大小 */
    }
    ```

- **行高（Line Height）**

  - `line-height`：调整文本行与行之间的垂直间距，既能提高可读性，又能让文本看起来更美观。常见的写法是：

    ```css
    p {
      line-height: 1.5;
    }
    ```

#### 2.2 文本排版与布局

- **对齐方式**

  - `text-align`：控制文本在容器内的水平对齐方式，常见值包括 `left`（默认）、`center`、`right` 和 `justify`（两端对齐）。

    ```css
    .center-align {
      text-align: center;
    }
    ```

- **首行缩进**

  - `text-indent`：设置段落首行的缩进，通常用 em 或 px 单位：

    ```css
    p {
      text-indent: 2em;
    }
    ```

    这在中文排版中常用来模拟传统的段落空格效果。

- **字间距与单词间距**

  - `letter-spacing`：调整字符之间的间距，适用于英文或者需要微调字体时使用：

    ```css
    h1 {
      letter-spacing: 2px;
    }
    ```

  - `word-spacing`：控制单词之间的额外空隙：

    ```css
    h1 {
      word-spacing: 5px;
    }
    ```

#### 2.3 文本装饰与特殊效果

- **文本装饰**

  - `text-decoration`：用于添加下划线、上划线或删除线。例如，为链接去除默认下划线或添加删除线：

    ```css
    a {
      text-decoration: none;
    }
    .oldPrice {
      text-decoration: line-through;
    }
    ```

- **文本阴影**

  - `text-shadow`：为文本添加阴影效果，可以指定水平偏移、垂直偏移、模糊半径和颜色：

    ```css
    h1 {
      text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
    }
    ```

    多个阴影效果可以通过逗号分隔添加。

#### 2.4 字体样式与粗细

- **字体样式**

  - `font-style`：常用值为 `normal`、`italic` 和 `oblique`，用于设置文本是否为斜体。

    ```css
    
    em {
      font-style: italic;
    }
    ```

- **字体粗细**

  - `font-weight`：控制文字的粗细，既可以用关键字（如 normal、bold），也可以用数字（100~900）来设置。

    ```css
    strong {
      font-weight: bold;
    }
    ```

#### 2.5 字体属性的简写

`font`：你可以使用 font 属性将多个字体相关属性合并为一行声明，其基本语法为：

```css
p {
  font: normal 16px/1.6 "Verdana", sans-serif;
}
```

其中，`16px` 是字体大小，`1.6` 是行高。



### 三、实战：新闻文章排版

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>新闻文章排版示例</title>
  <style>
    /* 全局基础设置 */
    body {
      font-family: "Helvetica", "Arial", sans-serif;
      background-color: #f4f4f4;
      margin: 0;
      padding: 20px;
      color: #333;
      line-height: 1.6;
    }
    /* 新闻文章容器 */
    .news-container {
      max-width: 800px;
      margin: 0 auto;
      background-color: #fff;
      padding: 30px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }
    /* 文章头部：标题和元信息 */
    .news-header {
      text-align: center;
      margin-bottom: 20px;
    }
    .news-header h1 {
      font-size: 2.4rem;
      margin: 0;
      text-transform: capitalize;
    }
    .news-meta {
      font-size: 0.9rem;
      color: #777;
      text-align: center;
      margin-bottom: 30px;
    }
    .news-meta span {
      margin: 0 10px;
    }
    /* 正文内容样式 */
    .news-content p {
      margin-bottom: 1.2rem;
      text-indent: 2em;
    }
    .news-content img {
      width: 100%;
      display: block;
      margin: 20px auto;
    }
    .news-content h2 {
      font-size: 1.8rem;
      margin-top: 1.8rem;
      margin-bottom: 1rem;
      border-bottom: 2px solid #eee;
      padding-bottom: 5px;
    }
  </style>
</head>
<body>
  <div class="news-container">
    <!-- 文章头部 -->
    <div class="news-header">
      <h1>最新科技新闻：人工智能革新未来</h1>
    </div>
    <div class="news-meta">
      <span>2025年2月14日</span> | <span>来源：新华网</span>
    </div>
    <!-- 文章正文 -->
    <div class="news-content">
      <p>随着人工智能技术的不断发展，越来越多的企业开始将其应用于各行各业。专家指出，未来几年内，人工智能将深刻改变我们的生活方式，并为经济增长带来新的动力。</p>
      <img src="https://via.placeholder.com/800x400" alt="人工智能新闻">
      <p>近日，在全球人工智能大会上，多家知名科技公司展示了最新的人工智能成果。从自动驾驶汽车到智能家居，各种应用场景层出不穷，让人们对未来充满期待。</p>
      <h2>人工智能在医疗领域的应用</h2>
      <p>在医疗领域，人工智能技术正被用于辅助诊断和治疗。通过大数据和机器学习，医生能够更快地分析患者的病情，从而制定出更为有效的治疗方案。</p>
      <p>专家表示，随着技术的成熟，未来人工智能将在医疗健康管理方面发挥更大作用，为患者提供更精准的服务。</p>
    </div>
  </div>
</body>
</html>
```

1. **全局设置**
   - 使用 `body` 设置了全局的字体、背景颜色、内边距以及行高，确保页面整体风格统一。
2. **容器设计**
   - `.news-container` 设定了最大宽度、居中显示、内边距和盒子阴影，使文章部分更加突出，阅读体验更佳。
3. **头部排版**
   - 文章标题采用了大字号、居中对齐和首字母大写（通过 `text-transform`）。
   - 元信息（发布日期和来源）采用较小的字体，并设置颜色为浅灰色。
4. **正文样式**
   - 每个段落设置了 `text-indent`（首行缩进）以模拟传统排版习惯。
   - 图片自适应宽度，并居中显示。
   - 小标题 `<h2>` 通过边框和内边距与正文区分开，起到分段提示作用。

