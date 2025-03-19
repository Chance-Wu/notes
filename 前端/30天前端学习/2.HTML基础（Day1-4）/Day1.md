### 学习内容

- 了解Web工作原理
- HTML文档结构
- 常用标签：标题、段落、列表、图片、链接
- 实战：创建个人简历页面

### 一、Web工作原理

---

```mermaid
sequenceDiagram
    participant 用户
    participant 浏览器
    participant DNS
    participant 服务器
    用户->>浏览器: 输入URL
    浏览器->>DNS: 查询域名IP
    DNS-->>浏览器: 返回IP地址
    浏览器->>服务器: 发起HTTP请求
    服务器->>服务器: 处理请求
    服务器-->>浏览器: 返回HTML/CSS/JS
    浏览器->>浏览器: 解析渲染页面
```

**Web工作原理**是理解前端开发的基础，它描述了用户通过浏览器访问网站时，数据如何在全球网络中传输和呈现。以下是核心要点解析。

#### 1.1 基础架构模型（客户端-服务器模型）

1. **客户端（Client）**
   - 用户使用的设备（如浏览器、手机APP）
   - 发起请求（Request）：例如输入`https://www.google.com`
   - 负责渲染和展示内容
2. **服务器（Server）**
   - 存储网站文件和数据的高性能计算机
   - 处理请求并返回响应（Response）
   - 常见服务器类型：Web服务器、数据库服务器

#### 1.2 关键工作流程（以访问网页为例）

1. 输入URL
   用户在浏览器地址栏输入网址（如 `https://www.example.com`）

2. DNS解析

   - 浏览器向DNS服务器查询域名对应的IP地址
   - 过程：浏览器缓存 → 系统缓存 → 路由器缓存 → ISP DNS服务器 → 递归查询
   - 最终获取目标服务器IP（如 `93.184.216.34`）

3. 建立TCP连接

   - 浏览器通过IP地址与服务器建立**TCP连接**（传输控制协议），确保可靠传输。

   - **三次握手**：
     1. 客户端发送 `SYN（Synchronize，同步）` 包。
     2. 服务器返回 `SYN-ACK（Synchronize Acknowledge，同步确认）` 包。
     3. 客户端确认 `ACK（Acknowledge，确认）` 包，连接建立。

4. 发送HTTP请求

   - 浏览器通过TCP连接发送**HTTP请求**。

     ```http
     GET /index.html HTTP/1.1
     Host: www.example.com
     User-Agent: Mozilla/5.0
     ```

   - 请求包含：
     - **方法**（GET、POST等）。
     - **请求头**（浏览器类型、语言、Cookie等）。
     - **请求体**（POST请求的数据）。

5. 服务器处理请求

   - **Web服务器**（如Nginx、Apache）接收请求，根据路径定位资源。
   - **静态资源**（HTML、图片）直接返回。
   - **动态资源**（如PHP、Python脚本）交由后端程序处理，生成HTML。

   - 可能涉及数据库查询、API调用等操作。

6. 服务器返回HTTP响应，包含：

   ```http
   HTTP/1.1 200 OK
   Content-Type: text/html
   <!DOCTYPE html>
   <html>...</html>
   ```

   - **状态码**（如 `200 OK` 成功，`404 Not Found` 未找到）。

   - **响应头**（内容类型、缓存策略等）。

   - **响应体**（HTML、JSON等数据）。

7. 浏览器解析与渲染

   1. **解析HTML**：构建DOM（文档对象模型）树。
   2. **解析CSS**：生成CSSOM（CSS对象模型）树。
   3. **合并为渲染树**：结合DOM和CSSOM，确定页面布局。
   4. **绘制（Paint）**：将像素渲染到屏幕上。
   5. **执行JavaScript**：可能修改DOM/CSSOM，触发重新渲染（重排/重绘）。

8. 断开TCP连接

   - 数据传输完成后，通过**四次挥手**释放TCP连接：
     1. 客户端发送 `FIN（Finish，结束）` 包。
     2. 服务器确认 `ACK（Acknowledge，确认）`。
     3. 服务器发送 `FIN（Finish，结束）` 包。
     4. 客户端确认 `ACK（Acknowledge，确认）`，连接关闭。

#### 1.3 核心协议与技术

1. **HTTP/HTTPS协议**

   - **HTTP方法**：GET（获取）、POST（提交）、PUT（更新）、DELETE（删除）
   - **状态码**：
     - 2xx（成功）：200 OK
     - 3xx（重定向）：301 永久重定向
     - 4xx（客户端错误）：404 找不到页面
     - 5xx（服务器错误）：500 服务器内部错误
   - **HTTPS**：通过SSL/TLS加密传输（端口443）

2. **URL结构**

   ```
   https           :// www.example.com  :   443    /path/to/file                 ? name=value            #fragment
   └─协议─┘   └───域名───┘└端口┘└────路径────┘ └──查询参数──┘└锚点┘
   ```

3. **缓存机制**

   - 浏览器缓存（强缓存/协商缓存）
   - CDN（内容分发网络）
   - 缓存头：Cache-Control、ETag、Last-Modified

4. **HTML/CSS/JavaScript**：构建网页内容、样式和交互。

5. **CDN（内容分发网络）**：缓存静态资源，加速访问。

#### 1.4 关注重点

1. **网络请求优化**
   - 减少HTTP请求数量（雪碧图、文件合并）
   - 使用CDN加速静态资源
   - 压缩资源（Gzip/Brotli）
2. **安全机制**
   - CORS（跨域资源共享）
   - CSRF/XSS防御
   - HTTPS强制部署
3. **现代Web特性**
   - HTTP/2多路复用
   - WebSocket实时通信
   - Service Worker离线缓存

### 二、HTML文档结构

---

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <!-- 元数据区域 -->
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="description" content="网站描述">
    <title>页面标题</title>
    <link rel="stylesheet" href="styles.css">
    <script src="script.js" defer></script>
</head>
<body>
    <!-- 可见内容区域 -->
    <header>
        <h1>网站主标题</h1>
        <nav>
            <a href="#home">首页</a>
            <a href="#about">关于</a>
        </nav>
    </header>
    
    <main>
        <article>
            <h2>文章标题</h2>
            <p>段落内容...</p>
            <img src="image.jpg" alt="图片描述">
        </article>
    </main>

    <footer>
        <p>&copy; 2023 版权所有</p>
    </footer>
</body>
</html>
```

#### 2.1 文档类型声明

```html
<!DOCTYPE html>
```

- 必须位于文档最顶端
- 声明使用HTML5标准
- 避免浏览器进入怪异模式

#### 2.2 根元素

```html
<html lang="zh-CN">
```

- 包裹所有HTML内容
- `lang`属性声明主要语言（SEO优化关键）

#### 2.3 头部区域

| 元素                 | 作用说明                              | 必备性   |
| :------------------- | :------------------------------------ | :------- |
| `<meta charset>`     | 定义文档字符编码（推荐UTF-8）         | 必需     |
| `<title>`            | 浏览器标签页标题/SEO重要指标          | 必需     |
| `<meta viewport>`    | 移动端适配关键设置                    | 强烈推荐 |
| `<meta description>` | 搜索引擎结果展示描述                  | 推荐     |
| `<link>`             | 引入外部CSS文件                       | 按需     |
| `<script>`           | 引入JavaScript文件（建议加defer属性） | 按需     |

#### 2.4 主体区域

- **语义化标签**（HTML5新增）：

  ```html
  <header>   <!-- 页眉 -->
  <nav>      <!-- 导航 -->
  <main>     <!-- 主内容 -->
  <article>  <!-- 独立内容块 -->
  <section>  <!-- 主题分区 -->
  <aside>    <!-- 侧边栏 -->
  <footer>   <!-- 页脚 -->
  ```

- 内容呈现顺序建议：

  ```
  1.页眉导航
  2.主要内容区
  3.辅助内容区
  4.页脚信息
  ```

  #### 2.5 实践指南

  1. **编码规范**

     - 统一使用小写标签
     - 属性值使用双引号
     - 自闭合标签无需闭合（`<img>`、`<meta>`）

  2. **移动优先**

     ```html
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     ```

  3. SEO优化

     ```html
     <meta name="keywords" content="关键词1,关键词2">
     <meta property="og:title" content="社交分享标题">
     ```

  4. 性能优化

     ```html
     <!-- 预加载关键资源 -->
     <link rel="preload" href="critical.css" as="style">
     
     <!-- 异步加载非关键JS -->
     <script src="analytics.js" async></script>
     ```



### 三、常用标签

**手动转义字符**：你可以将HTML标签中的特殊字符转换成它们对应的HTML实体。例如：

- `<` 转换为 `&lt;`
- `>` 转换为 `&gt;`
- `&` 转换为 `&amp`

#### 3.1 标题标签`<h1>-<h6>`

```html
<h1>主标题</h1>  <!-- 一个页面建议只出现一次 -->
<h2>二级标题</h2>
<h3>三级标题</h3>
<!-- ...依此类推到h6 -->
```

>**特性**：
>
>- 浏览器会自动地在标题的前后添加空行。
>- 搜索引擎使用标题为您的网页的结构和内容编制索引。因为用户可以通过标题来快速浏览您的网页，所以用标题来呈现文档结构是很重要的。应该将 h1 用作主标题（最重要的），其后是 h2（次重要的），再其次是 h3，以此类推。

#### 3.2 段落标签`<p>`

```html
<p>这是一个段落文本，包含多个句子。HTML会合并连续空格，保留换行需要特殊处理。</p>
```

>**特性**：
>
>- 浏览器会自动地在段落的前后添加空行。（`</p>` 是块级元素）
>- 自动上下边距（默认1em）
>- 空白折叠现象：连续空格/换行显示为单个空格
>- 禁止嵌套块级元素（如div、h1等）
>
>**保留格式技巧**：
>
>```html
><pre> <!-- 预格式化文本 -->
>  保留空格
>  和换行
></pre>
><p>使用&nbsp;强制空格<br>换行标签</p>
>```

#### 3.3 列表

无序列表`<ul>`

```html
<ul>
  <li>列表项1</li>
  <li>列表项2</li>
</ul>
```

>**应用场景**：
>
>- 导航菜单
>- 功能列表
>- 商品特征展示

有序列表`<ol>`

```html
<ol type="A" start="3">
  <li>步骤一</li>
  <li>步骤二</li>
</ol>
```

>**属性控制**：
>
>- `type`：编号类型（1/a/A/i/I）
>- `start`：起始编号
>- `reversed`：倒序排列

#### 3.4 图片标签`<img>`

```html
<img 
  src="image.jpg" 
  alt="图片描述文本" 
  width="400" 
  height="300"
  loading="lazy"
  srcset="image-320w.jpg 320w,
          image-640w.jpg 640w"
  sizes="(max-width: 600px) 480px,
         800px">
```

>**关键属性**：
>
>| 属性           | 作用说明                        | 必需性   |
>| :------------- | :------------------------------ | :------- |
>| `src`          | 图片资源路径                    | 必需     |
>| `alt`          | 替代文本（SEO与无障碍访问关键） | 强烈推荐 |
>| `width/height` | 显示尺寸（建议CSS控制样式）     | 可选     |
>| `loading`      | 懒加载设置（lazy/eager）        | 可选     |
>| `srcset`       | 响应式图片源                    | 可选     |
>| `sizes`        | 媒体查询尺寸匹配                | 可选     |

#### 3.5 超链接标签`<a>`

```html
<a 
  href="https://example.com" 
  target="_blank" 
  rel="noopener noreferrer"
  download="filename.pdf"
>
  访问示例网站
</a>
```

>**属性详解**：
>
>- `href` 取值类型：
>
>  ```html
>  <a href="#section2">页面锚点</a>
>  <a href="tel:13800138000">拨打电话</a>
>  <a href="mailto:contact@example.com">发送邮件</a>
>  <a href="javascript:void(0)">伪链接（慎用）</a>
>  ```
>
>- `target` 行为控制：
>
>  - `_blank`：新标签页打开
>  - `_self`：当前页面（默认）
>  - 自定义命名窗口
>
>**安全实践**：
>
>```html
><!-- 防止钓鱼攻击 -->
><a href="external.com" rel="noopener">外部链接</a>
>```

#### 3.6 开发注意事项

1. **语义化优先**：
   - 避免用div代替语义标签
   - 正确嵌套标签（如p标签内不放div）
2. **可访问性**：
   - 为所有img添加有效alt描述
   - 使用ARIA标签增强交互元素
3. **SEO优化**：
   - 重要内容使用h标签包裹
   - 合理设置锚文本内容



### 四、创建个人简历页面

---

#### 4.1 语义化结构

- 使用header/main/footer划分页面大区块
- 每个模块使用section包裹
- 合理使用h1-h3标题层级

#### 4.2 样式设计原则

- 移动优先的响应式布局
- 柔和的阴影和圆角提升视觉效果
- 使用CSS Grid和Flexbox进行布局
- 建立统一的间距系统（margin/padding）

#### 4.3 最佳实践

```css
/* 盒模型统一设置 */
* { box-sizing: border-box; }

/* 响应式图片处理 */
img { max-width: 100%; height: auto; }

/* 打印优化 */
@media print {
    .skill-tag { background: none; border: 1px solid #000; }
}
```

#### 4.4 示例

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    </meta>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    </meta>
    <meta name="description" content="简历">
    </meta>
    <title>chance的个人简历</title>
    <link rel="stylesheet" href="../../css/resume_style.css">
    </link>
    <script src="../../js/script.js" async defer></script>
</head>

<body>
    <header class="header">
        <img src="../../img/image.jpg" alt="chance的头像" class="avatar">
        <h1>chance</h1>
        <p>🎓 本科 | 📮 chancewu@aliyun.com | 📱 13871195843</p>
        <p>📍 江阴 | 🌐 chance.com</p>
    </header>
    <main>
        <section class="section">
            <h2>个人简介</h2>
            <p>全栈开发工程师，6年开发经验，擅长Spring技术栈，熟悉Vue全栈开发，对用户体验和代码质量有极致追求。</p>
        </section>

        <section class="section">
            <h2>工作经验</h2>
            <div class="job-item">
                <h3>java后端开发工程师 - xxx科技有限公司</h3>
                <p class="job-date">2019.07 - 至今</p>
                <ul>
                    <li>主导开发msp</li>
                    <li>参与众多核心项目的开发与优化</li>
                    <li>实施代码规范</li>
                </ul>
            </div>
        </section>

        <section class="section">
            <h2>教育背景</h2>
            <h3>电子信息工程 学士</h3>
            <p>xxx大学 | 2014.09 - 2018.06</p>
        </section>

        <section class="section">
            <h2>技能</h2>
            <div class="skills">
                <span class="skill-tag">HTML5</span>
                <span class="skill-tag">CSS3</span>
                <span class="skill-tag">JavaScript (ES6+)</span>
                <span class="skill-tag">React</span>
                <span class="skill-tag">Node.js</span>
                <span class="skill-tag">Webpack</span>
                <span class="skill-tag">Git</span>
            </div>
        </section>
    </main>

    <footer class="section" style="text-align: center; margin-top: 30px;">
        <p>© 2025 chance - 更新于2025年2月</p>
        <p>
            <a href="https://github.com/Chance-Wu" target="_blank">GitHub</a>
        </p>
    </footer>
</body>

</html>
```

```css
/* 基础样式，盒模型统一设置 */
* {
    margin: 0;
    padding: 0;
    /* 设置盒模型为border-box，使宽度和高度包含内边距和边框。 */
    box-sizing: border-box;
}

/* 响应式图片处理 */
img {
    max-width: 18%;
    height: auto;
}

/* 打印优化 */
@media print {
    .skill-tag {
        background: none;
        border: 1px solid #000;
    }
}

/* 头部样式 */
.header {
    text-align: center;
    margin-bottom: 30px;
    padding: 20px;
    background-color: #fff;
    border-radius: 10px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

body {
    font-family: 'Segoe UI', Arial, sans-serif;
    line-height: 1.6;
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f5f5f5;
}

.avatar {
    border-radius: 50%;
    margin-bottom: 10px;
    border: 3px solid #3498db;
}

/* 主要内容区块 */
.section {
    background: white;
    padding: 25px;
    margin-bottom: 25px;
    border-radius: 8px;
    box-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
}

.section h2 {
    color: #3498db;
    border-bottom: 2px solid #3498db;
    padding-bottom: 10px;
    margin-bottom: 20px;
}

/* 工作经验条目 */
.job-item {
    margin-bottom: 25px;
}

.job-item h3 {
    color: #2c3e50;
}

.job-date {
    color: #7f8c8d;
    font-size: 0.9em;
}

/* 技能标签 */
.skills {
    display: grid;
    /* grid-template-columns: repeat(auto-fit, minmax(120px, max-content)); */
    /* grid-template-columns: 160px 160px 160px 160px; */
    /* grid-template-columns: 1fr 1fr 1fr 1fr; */
    /* grid-template-columns: repeat(4, 1fr); */
    grid-template-columns: repeat(auto-fill, minmax(130px, 1fr));
    gap: 16px 16px;
    /* column-gap: 8px; */
    /* row-gap: 16px; */
}

.skill-tag {
    background: #3498db;
    color: white;
    padding: 5px 15px;
    border-radius: 25px;
    text-align: center;
    font-size: 0.8em;
    transition: transform 0.5s ease;
}

.skill-tag:hover {
    /* transform: scale(1.2); */
    transform: translateY(-2px);
}

/* 响应式设计 */
@media screen and (max-width: 600px) {
    body {
        padding: 10px;
    }

    .section {
        padding: 15px;
    }
}

/* @media (hover:hover) {
    .section:hover {
        color: #ff0000;
        transition-duration: 0.3s;
    }
} */

@media (prefers-color-scheme: dark) {
    body {
        background: #2c3e50;
        color: #ecf0f1;
    }

    .header {
        background: #34495e;
    }

    .section {
        background: #34495e;
    }
}
```

#### 4.5 扩展建议

- 添加更多交互效果：

  ```css
  .skill-tag {
      transition: transform 0.3s ease;
  }
  .skill-tag:hover {
      transform: translateY(-2px);
  }

- 实现暗黑模式

  ```css
  @media (prefers-color-scheme: dark) {
      body {
          background: #2c3e50;
          color: #ecf0f1;
      }
  
      .header {
          background: #34495e;
      }
  
      .section {
          background: #34495e;
      }
  }

#### 4.6 常见问题处理

1. 布局错位调试：

   - 使用浏览器开发者工具检查元素盒模型
   - 临时添加边框辅助调试：`border: 1px solid red;`

2. 移动端适配：

   ```css
   /* 防止手机端字体过小 */
   @media (max-width: 480px) {
       html { font-size: 14px; }
   }
   ```

#### 4.7 后续优化

1. 添加PDF导出功能

   1. **引入库**：在 `<head>` 中添加了 html2pdf.js 的 CDN 链接，确保页面加载时能使用该库。

   2. **按钮**：在 header 部分添加了一个按钮，点击后调用 `downloadPDF()` 函数。

   3. **导出函数**：函数中利用 html2pdf.js 将整个页面（即 `document.body`）转换成 PDF 并下载。你可以根据需要调整导出的区域和参数。

   4. 若页面内容较长，可利用 html2pdf.js 提供的分页控制选项。你可以在配置中增加 `pagebreak` 选项，避免在不合适的位置自动分页。例如：

      ```js
      const opt = {
          margin:       0.5,
          filename:     'chance_resume.pdf',
          image:        { type: 'jpeg', quality: 0.98 },
          html2canvas:  { scale: 2, useCORS: true },
          jsPDF:        { unit: 'in', format: 'letter', orientation: 'portrait' },
          pagebreak: {
              mode: ['avoid-all', 'css', 'legacy']  // 尝试调整此选项，避免在中间拆分内容
          }
      };
      html2pdf().set(opt).from(element).save();

2. 集成在线表单提交

3. 连接GitHub API动态显示项目

4. 增加多语言支持

   1. 引入国际化库：在 `<head>` 部分通过 CDN 引入了 [i18next](https://www.i18next.com/) 及其浏览器语言检测插件，这两个库可以帮助你自动检测浏览器语言并提供国际化支持。
   2. 设置翻译资源：在 i18next 的初始化配置中，我们为中文（zh）和英文（en）分别定义了对应的翻译内容。每个需要国际化的元素在 HTML 中使用 `data-i18n` 属性，并设置一个对应的键值（例如 `"profileDesc"`）。
   3. 更新页面文本：初始化完成后调用 `updateContent()` 函数，该函数遍历所有带有 `data-i18n` 属性的元素，并使用 `i18next.t(key)` 方法更新文本内容。
   4. 语言切换功能：页面上提供了两个按钮，分别调用 `changeLanguage('zh')` 与 `changeLanguage('en')`，切换当前语言后重新调用 `updateContent()` 更新页面显示。
   5. 与 PDF 导出共存
      - PDF 导出部分依然调用 html2pdf.js 库。这里建议将导出的区域限定为主要内容（例如通过给 `<main>` 元素设置 id="resume"），这样可以避免导出语言切换按钮等干扰部分。

5. 使用CSS变量统一主题色

   1. 定义全局CSS变量。在你的主 CSS 文件（例如 `resume_style.css`）中，可以在 `:root` 选择器下定义全局变量。例如：

      ```css
      :root {
        /* 定义主题色变量 */
        --primary-color: #3498db;   /* 主题蓝色 */
        --secondary-color: #2ecc71; /* 辅助绿色 */
        --background-color: #f9f9f9;/* 背景色 */
        --text-color: #333;         /* 正文字体颜色 */
        --header-bg-color: #2980b9; /* 页头背景色 */
      }
      
      ```

   2. 在CSS中使用变量。定义好变量后，你可以在各个样式规则中使用这些变量。例如：

      ```css
      body {
        background-color: var(--background-color);
        color: var(--text-color);
        font-family: Arial, sans-serif;
        margin: 0;
        padding: 0;
      }
      
      .header {
        background-color: var(--header-bg-color);
        color: #fff;
        padding: 20px;
        text-align: center;
      }
      
      .header h1 {
        margin: 0;
      }
      
      .language-selector button,
      #download-pdf {
        background-color: var(--primary-color);
        border: none;
        color: #fff;
        padding: 8px 16px;
        margin: 5px;
        cursor: pointer;
        border-radius: 4px;
        transition: background-color 0.3s ease;
      }
      
      .language-selector button:hover,
      #download-pdf:hover {
        background-color: var(--secondary-color);
      }
      
      /* 示例：表单样式 */
      #contactForm {
        max-width: 600px;
        margin: 20px auto;
        padding: 20px;
        background-color: #fff;
        border: 1px solid var(--primary-color);
        border-radius: 8px;
      }
      
      #contactForm label {
        display: block;
        margin-bottom: 5px;
        font-weight: bold;
      }
      
      #contactForm input,
      #contactForm textarea {
        width: 100%;
        padding: 8px;
        margin-bottom: 10px;
        border: 1px solid var(--primary-color);
        border-radius: 4px;
      }
      
      ```

   3. 在HTML中引用CSS

6. 集成第三方组件库（如Font Awesome图标）