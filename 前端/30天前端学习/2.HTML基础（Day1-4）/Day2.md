### 学习内容

- 表格与表单元素
- 语义化标签（header/nav/article等）
- HTML5新特性
- 实战：制作课程报名表



### 一、表格与表单元素

---

#### 1.1 表格系统

1. 基础表单格结构

   ```html
   <table>
       <!-- 表格标题，简述表格内容 -->
       <caption>
           表格系统
       </caption>
   
       <!-- 表头行，定义表格的列 -->
       <thead>
           <tr>
               <th scope="col">标签</th>
               <th scope="col">描述</th>
               <th scope="col"></th>
           </tr>
       </thead>
   
       <!-- 表体内容 -->
       <tbody>
           <tr>
               <th scope="row">&lt;table&gt;</th>
               <td>定义表格</td>
           </tr>
           <tr>
               <th scope="row">&lt;th&gt;</th>
               <td>定义表格的表头</td>
           </tr>
           <tr>
               <th scope="row">&lt;tr&gt;</th>
               <td>定义表格的行</td>
           </tr>
           <tr>
               <th scope="row">&lt;td&gt;</th>
               <td>定义表格单元</td>
           </tr>
           <tr>
               <th scope="row">&lt;caption&gt;</th>
               <td>定义表格标题</td>
           </tr>
           <tr>
               <th scope="row">&lt;colgroup&gt;</th>
               <td>定义表格列的组</td>
           </tr>
           <tr>
               <th scope="row">&lt;col&gt;</th>
               <td colspan="2">定义用于表格列的属性</td>
           </tr>
           <tr>
               <th scope="row">&lt;thead&gt;</th>
               <td>定义表格的页眉</td>
           </tr>
           <tr>
               <th scope="row">&lt;tbody&gt;</th>
               <td>定义表格的主体</td>
           </tr>
           <tr>
               <th scope="row">&lt;tfoot&gt;</th>
               <td>定义表格的页脚</td>
           </tr>
       </tbody>
   
       <!-- 表尾总结行 -->
       <tfoot>
           <tr>
               <th scope="row" colspan="2">页脚</th>
           </tr>
       </tfoot>
   
   </table>
   ```

   - 核心标签解析：

     - `<table>`：表格容器

     - `<tr>`：表格行（table row）

     - `<td>`：标准单元格（table data）

     - `<th>`：表头单元格（加粗居中 table header）

     - `<caption>`：表格标题（HTML5语义化）

     - `<thead>`/`<tbody>`/`<tfoot>`：分组结构

   - 属性控制：

     - `colspan`：横向合并单元格
     - `rowspan`：纵向合并单元格
     - `border`：边框显示（建议用CSS替代）

   - 高级表格特性

     ```html
     <table>
       <colgroup>
         <col span="1" style="background-color: #f0f0f0">
         <col style="width: 100px">
       </colgroup>
       <!-- 表格内容 -->
     </table>
     ```

   - 增强功能：

     - `<colgroup>`：列样式分组

     - `<col>`：定义列属性

     - `scope`属性（用于屏幕阅读器）

       ```html
       <th scope="col">学科</th>
       <th scope="row">张三</th>
       ```

#### 1.2 表单系统

1. 基础表单结构

   ```html
   <form action="/submit" method="POST" enctype="multipart/form-data">
       <fieldset>
           <legend>注册信息</legend>
   
           <div class="form-group">
               <label for="username">用户名：</label>
               <input type="text" id="username" name="username" placeholder="4-16位字母数字" required
                   pattern="[A-Za-z0-9]{4,16}">
           </div>
   
           <div class="form-group">
               <label for="password">密码：</label>
               <input type="password" id="password" name="password" required>
           </div>
       </fieldset>
   </form>
   ```

2. 核心组件

   - `<form>`：表单容器
     - `action`：提交地址
     - `method`：HTTP方法（GET/POST）
     - `enctype`：编码类型（文件上传需用multipart/form-data）
   - `<fieldset>`：表单分组
   - `<legend>`：分组标题
   - `<label>`：标签关联
   - `<input>`：输入控件

3. 其他表单元素

   ```html
   <!-- 下拉选择 -->
   <select name="city" id="city">
     <option value="">请选择城市</option>
     <optgroup label="一线城市">
       <option value="beijing">北京</option>
       <option value="shanghai">上海</option>
     </optgroup>
   </select>
   
   <!-- 多行文本 -->
   <textarea name="bio" rows="4" cols="50" maxlength="200"></textarea>
   
   <!-- 数据列表 -->
   <input list="browsers" name="browser">
   <datalist id="browsers">
     <option value="Chrome">
     <option value="Firefox">
   </datalist>
   
   <!-- 进度条 -->
   <progress value="70" max="100"></progress>
   ```

#### 1.3 实战：课程报名表

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    </meta>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    </meta>
    <meta name="description" content="课程报名表">
    </meta>
    <title>课程报名表</title>
    <link rel="stylesheet" href="../../css/course_registration_form.css">
</head>

<body>
    <form class="form-container">
        <h2>课程报名表</h2>

        <div class="form-group">
            <label for="name">姓名：</label>
            <input type="text" id="name" name="name" required>
        </div>

        <div class="form-group">
            <label>性别：</label>
            <label><input type="radio" name="gender" value="male" required> 男</label>
            <label><input type="radio" name="gender" value="female"> 女</label>
        </div>

        <div class="form-group">
            <label for="course">选择课程：</label>
            <select id="course" name="course" required>
                <option value="">请选择</option>
                <option value="frontend">前端开发</option>
                <option value="backend">后端开发</option>
            </select>
        </div>

        <div class="form-group">
            <label>学习方式：</label>
            <label><input type="checkbox" name="method" value="online"> 在线学习</label>
            <label><input type="checkbox" name="method" value="offline"> 线下学习</label>
        </div>

        <div class="form-group">
            <label for="email">邮箱：</label>
            <input type="email" id="email" name="email" required>
        </div>

        <div class="form-group">
            <label for="phone">手机：</label>
            <input type="tel" id="phone" name="phone" pattern="[0-9]{11}">
        </div>

        <div class="form-group">
            <label for="message">留言：</label>
            <textarea id="message" rows="4"></textarea>
        </div>

        <input type="submit" value="立即报名">
    </form>
</body>

</html>
```

```css
.form-container {
    max-width: 600px;
    margin: 20px auto;
    padding: 20px;
    border: 1px solid #ddd;
}

.form-group {
    margin-bottom: 15px;
}

label {
    display: inline-block;
    width: 120px;
}

input[type="submit"] {
    background-color: #4CAF50;
    color: white;
    padding: 10px 20px;
    border: none;
    cursor: pointer;
}
```

#### 1.4 开发注意事项

1. 嵌套错误

   ```html
   <!-- 错误示例 -->
   <table>
     <tr>
       <td>
         <table> <!-- 表格内直接嵌套表格 -->
           ...
         </table>
       </td>
     </tr>
   </table>

2. 语义缺失

   ```html
   <!-- 应使用thead/tbody替代 -->
   <table>
     <tr style="background: gray">
       <td>标题1</td>
     </tr>
   </table>

#### 1.5 表单最佳实践

1. 可访问性增强

   ```html
   <!-- 使用aria标签 -->
   <div role="alert" id="errorMsg"></div>
   
   <!-- 错误提示关联 -->
   <input aria-describedby="emailError">
   <span id="emailError" class="error"></span>
   ```

2. 安全防护

   ```html
   <!-- CSRF防护 -->
   <input type="hidden" name="csrf_token" value="...">
   
   <!-- 文件类型限制 -->
   <input type="file" accept=".pdf,.docx">



### 二、语义化标签

---

#### 2.1 语义化核心价值

1. 机器可读性
   - 提升搜索引擎理解（**SEO优化**）
   - 增强屏幕阅读器解析（无障碍访问）
2. 开发维护性
   - 代码结构清晰易读
   - **减少class命名负担**
3. 未来兼容性
   - 适配新兴设备（智能眼镜、车载系统等）

#### 2.2 主要语义标签详解

1. 页面级容器

   | 标签       | 用途说明               | 示例场景                   |
   | :--------- | :--------------------- | :------------------------- |
   | `<header>` | 页面/区块的头部内容    | 网站LOGO、导航栏、搜索框   |
   | `<footer>` | 页面/区块的底部内容    | 版权信息、联系方式、备案号 |
   | `<nav>`    | 导航链接集合           | 主导航、侧边栏目录、分页器 |
   | `<main>`   | 页面主要内容区（唯一） | 文章主体、产品列表         |

2. 内容区块

   | 标签        | 用途说明                     | 示例场景                                |
   | :---------- | :--------------------------- | :-------------------------------------- |
   | `<article>` | 独立完整的内容块             | 博客文章、新闻条目、论坛帖子            |
   | `<section>` | 主题性内容分组               | 章节、标签页内容、产品特性              |
   | `<aside>`   | 与主体内容相关但不直接的部分 | 侧边栏、广告、引用内容                  |
   | `<figure>`  | 独立流内容（配合figcaption） | 图片、图表、代码示例                    |
   | `<time>`    | 时间/日期信息                | 发布日期 `<time datetime="2023-07-20">` |

3. 语义增强

   | 标签         | 用途说明     | 示例场景             |
   | :----------- | :----------- | :------------------- |
   | `<mark>`     | 突出显示文本 | 搜索关键词高亮       |
   | `<progress>` | 任务进度指示 | 文件上传进度条       |
   | `<meter>`    | 标量值测量   | 磁盘使用量、投票比例 |

#### 2.3 应用对比示例

**传统div布局**：

```html
<div class="page">
  <div class="top-area">
    <div class="logo"></div>
    <div class="menu"></div>
  </div>
  
  <div class="content">
    <div class="post">
      <div class="text-block"></div>
      <div class="image-box"></div>
    </div>
  </div>
</div>
```

**语义化重构**：

```html
<body>
  <header>
    <img src="logo.png" alt="网站标志">
    <nav>
      <a href="/">首页</a>
      <a href="/about">关于我们</a>
    </nav>
  </header>

  <main>
    <article>
      <h1>人工智能发展简史</h1>
      <section>
        <h2>早期发展阶段</h2>
        <figure>
          <img src="ai-history.jpg" alt="人工智能发展时间线">
          <figcaption>图1：人工智能发展历程</figcaption>
        </figure>
      </section>
    </article>

    <aside>
      <h3>相关阅读</h3>
      <ul>
        <li><a href="#">机器学习基础</a></li>
      </ul>
    </aside>
  </main>

  <footer>
    <p>&copy; 2023 科技前沿网</p>
    <address>联系我们: contact@tech.com</address>
  </footer>
</body>
```

#### 2.4 使用规范与误区

1. 嵌套规则

   - `<header>`/`<footer>`可出现在多个区块

   - `<article>`可包含多个`<section>`

   - 禁止错误嵌套：`<nav>`内不应放`<aside>`

2. 常见错误

   ```html
   <!-- 错误1：滥用section -->
   <section> <!-- 缺少标题 -->
     <p>普通段落内容...</p>
   </section>
   
   <!-- 错误2：误用article -->
   <article> <!-- 非独立内容 -->
     <p>产品列表的单个商品</p>
   </article>
   
   <!-- 错误3：忽略层级 -->
   <main>
     <header>...</header> <!-- 应放在main外部 -->
   </main>
   ```

3. 最佳实践

   1. 标题层级管理

      ```html
      <article>
        <h1>主标题</h1> <!-- 允许在article内重置标题层级 -->
        <section>
          <h2>子标题</h2>
        </section>
      </article>
      ```

   2. WAI-ARIA增强

      ```html
      <nav aria-label="主导航">
      <article role="article">
      ```

   3. 微数据标记

      ```html
      <article itemscope itemtype="https://schema.org/Article">
        <h1 itemprop="headline">标题</h1>
        <time itemprop="datePublished" datetime="2023-07-20"></time>
      </article>
      ```

#### 2.5 语义验证工具

- Chrome：Elements面板 > Accessibility树
- W3C Nu Checker：https://validator.w3.org/nu/
- Lighthouse无障碍审计

#### 2.6 实战：重构新闻网站

原始代码：

```html
<div class="news-page">
  <div class="top-bar">...</div>
  
  <div class="content">
    <div class="post">
      <div class="title">...</div>
      <div class="body">...</div>
    </div>
    
    <div class="side-info">...</div>
  </div>
</div>
```

语义话重构：

```html
<body>
  <header>
    <nav aria-label="快速导航">
      <a href="#breaking-news">突发新闻</a>
      <a href="#politics">时政要闻</a>
    </nav>
  </header>

  <main>
    <article itemscope itemtype="https://schema.org/NewsArticle">
      <h1 itemprop="headline">量子计算机突破性进展</h1>
      <div itemprop="author" itemscope itemtype="https://schema.org/Person">
        作者: <span itemprop="name">王研究员</span>
      </div>
      
      <section>
        <h2>技术细节</h2>
        <figure itemprop="image" itemscope itemtype="https://schema.org/ImageObject">
          <img src="quantum-chip.jpg" alt="新型量子芯片">
          <figcaption itemprop="description">图：最新研发的量子处理单元</figcaption>
        </figure>
      </section>
    </article>

    <aside aria-label="相关报道">
      <h3>延伸阅读</h3>
      <ul>
        <li><a href="#" itemprop="relatedLink">量子计算原理科普</a></li>
      </ul>
    </aside>
  </main>
</body>
```



### 三、HTML5新特性

---

#### 3.1 语义化标签增强

```html
<!-- 文档结构标签 -->
<header>导航/标题</header>
<nav>菜单导航</nav>
<main>主要内容</main>
<section>内容分区</section>
<article>独立内容</article>
<aside>侧边内容</aside>
<footer>页脚信息</footer>

<!-- 文本级语义标签 -->
<mark>高亮文本</mark>
<time datetime="2023-07-20">日期</time>
<progress value="75" max="100"></progress>
<meter min="0" max="100" value="60"></meter>
```

优势：

- 提升SEO优化效果
- 增强无障碍访问能力
- 代码可读性提升40%+

#### 3.2 多媒体原生支持

1. 音视频嵌入：

   ```html
   <video controls width="600" poster="preview.jpg">
     <source src="movie.mp4" type="video/mp4">
     <track label="中文字幕" kind="subtitles" src="subs.vtt" srclang="zh">
     您的浏览器不支持视频播放
   </video>
   
   <audio controls loop>
     <source src="audio.ogg" type="audio/ogg">
   </audio>
   ```

   关键属性：

   - `controls`：显示控制条
   - `autoplay`：自动播放（受浏览器限制）
   - `preload`：预加载策略
   - `muted`：静音模式

#### 3.3 图形处理能力

1. Canvas绘图

   ```html
   <canvas id="myCanvas" width="400" height="200"></canvas>
   <script>
     const ctx = document.getElementById('myCanvas').getContext('2d');
     ctx.fillStyle = 'red';
     ctx.fillRect(10, 10, 100, 50);
   </script>
   ```

2. SVG矢量图

   ```html
   <svg width="100" height="100">
     <circle cx="50" cy="50" r="40" stroke="green" fill="yellow"/>
   </svg>
   ```

| 特性     | Canvas          | SVG             |
| :------- | :-------------- | :-------------- |
| 图像类型 | 位图            | 矢量图          |
| 更新性能 | 适合高频更新    | 适合静态展示    |
| 事件支持 | 无              | 支持元素级事件  |
| 适用场景 | 数据可视化/游戏 | 图标/可缩放图形 |

#### 3.4 增强表单功能

1. 新输入类型

   ```html
   <input type="email" required placeholder="输入邮箱">
   <input type="date" min="2020-01-01">
   <input type="color" value="#ff0000">
   <input type="range" min="0" max="100" step="5">
   ```

2. 表单验证API

   ```javascript
   const emailInput = document.getElementById('email');
   emailInput.addEventListener('invalid', () => {
     if(emailInput.validity.typeMismatch) {
       emailInput.setCustomValidity('请输入有效的邮箱地址');
     }
   });
   ```

   - `pattern`：正则验证
   - `required`：必填项
   - `minlength/maxlength`：长度限制
   - `novalidate`：禁用浏览器验证

#### 3.5 本地存储方案

1. Web Storage

   ```javascript
   // 存储数据
   localStorage.setItem('user', JSON.stringify({name: 'John'}));
   
   // 读取数据
   const user = JSON.parse(localStorage.getItem('user'));
   
   console.log(user);
   
   // 删除数据
   localStorage.removeItem('user');
   ```

2. IndexedDB

   ```html
   <!DOCTYPE html>
   <html lang="zh-CN">
   
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <meta name="description" content="本地存储方案">
       <title>本地存储方案</title>
   </head>
   
   <body>
   
       <h1>IndexedDB 示例页面</h1>
   
       <button onclick="addData('张三', 'zhangsan@example.com')">添加数据</button>
       <button onclick="getAllData()">获取所有数据</button>
       <ul id="results"></ul>
   
       <script>
           // // web storage
           // // 存储数据
           // localStorage.setItem('user', JSON.stringify({ name: 'John' }));
   
           // // 读取数据
           // const user = JSON.parse(localStorage.getItem('user'));
   
           // console.log(user);
           // // 删除数据
           // localStorage.removeItem('user');
   
           // indexeddb
           let db;
           const request = window.indexedDB.open("MyTestDatabase", 1);
   
           // 如果需要更新数据库结构，增加版本号并在此处理 onupgradeneeded 事件
           request.onupgradeneeded = function (event) {
               db = event.target.result;
               if (!db.objectStoreNames.contains('customers')) {
                   // 创建一个对象存储空间
                   let objectStore = db.createObjectStore("customers", { keyPath: "id", autoIncrement: true });
                   // 定义索引
                   objectStore.createIndex("name", "name", { unique: false });
                   objectStore.createIndex("email", "email", { unique: true });
               }
           };
   
           request.onsuccess = function (event) {
               db = event.target.result;
               console.log("数据库打开成功");
           };
   
           request.onerror = function (event) {
               console.error("数据库打开失败");
           };
   
           // 添加数据到数据库
           function addData(name, email) {
               let transaction = db.transaction(["customers"], "readwrite");
               let objectStore = transaction.objectStore("customers");
               let request = objectStore.add({ name: name, email: email });
   
               request.onsuccess = function (event) {
                   console.log("数据添加成功", event);
               };
   
               transaction.oncomplete = function (event) {
                   console.log("事务完成");
               };
   
               transaction.onerror = function (event) {
                   console.error("事务失败");
               };
           }
   
           // 获取所有数据
           function getAllData() {
               let transaction = db.transaction(["customers"], "readonly");
               let objectStore = transaction.objectStore("customers");
               let request = objectStore.getAll();
   
               request.onsuccess = function (event) {
                   const resultsList = document.getElementById('results');
                   resultsList.innerHTML = ''; // 清空列表
                   request.result.forEach(function (item) {
                       let li = document.createElement('li');
                       li.textContent = `ID: ${item.id}, Name: ${item.name}, Email: ${item.email}`;
                       resultsList.appendChild(li);
                   });
               };
           }
       </script>
   </body>
   
   </html>
   ```

> 存储方案对比
>
> | 类型           | 容量  | 数据类型   | 同步性 |
> | :------------- | :---- | :--------- | :----- |
> | Cookie         | 4KB   | 字符串     | 同步   |
> | localStorage   | 5MB   | 字符串     | 同步   |
> | sessionStorage | 5MB   | 字符串     | 同步   |
> | IndexedDB      | 50MB+ | 结构化数据 | 异步   |

#### 3.6 设备API集成

1. 地理位置

   ```javascript
   navigator.geolocation.getCurrentPosition(
     (position) => {
       console.log(`纬度: ${position.coords.latitude}`);
       console.log(`经度: ${position.coords.longitude}`);
     },
     (error) => console.error(error)
   );
   ```

2. 陀螺仪/加速计

   ```javascript
   window.addEventListener('deviceorientation', (event) => {
     console.log(`Alpha: ${event.alpha}`); // Z轴旋转
     console.log(`Beta: ${event.beta}`);   // X轴旋转
     console.log(`Gamma: ${event.gamma}`); // Y轴旋转
   });
   ```

#### 3.7 通信协议增强

1. WebSocket

   是一种在客户端与[服务器](https://developer.mozilla.org/zh-CN/docs/Glossary/Server)之间保持 [TCP](https://developer.mozilla.org/zh-CN/docs/Glossary/TCP) 长连接的[协议](https://developer.mozilla.org/zh-CN/docs/Glossary/Protocol)，这样它们就可以随时进行信息交换。

   虽然任何客户端或服务器上的应用都可以使用 WebSocket，但原则上还是指[浏览器](https://developer.mozilla.org/zh-CN/docs/Glossary/Browser)与服务器之间使用。通过 WebSocket，服务器无需客户端预先请求就可以直接向客户端发送数据，从而能动态地更新数据内容。

   ```javascript
   const socket = new WebSocket('wss://example.com/ws');
   socket.onmessage = (event) => {
     console.log('收到消息:', event.data);
   };
   socket.send('Hello Server!');

2. Server-Sent Events（SSE）

   ```javascript
   const eventSource = new EventSource('/updates');
   eventSource.onmessage = (e) => {
     console.log('实时更新:', e.data);
   };
   ```

#### 3.8 性能优化API

1. Web Workers

   ```javascript
   // main.js
   const worker = new Worker('worker.js');
   worker.postMessage({data: 1000});
   worker.onmessage = (e) => console.log(e.data);
   
   // worker.js
   self.onmessage = (e) => {
     const result = heavyCalculation(e.data);
     self.postMessage(result);
   };
   ```

2. requestAnimationFrame

   ```javascript
   function animate() {
     // 更新动画帧
     requestAnimationFrame(animate);
   }
   animate();
   ```

#### 3.9 安全增强机制

1. Content Security Policy (CSP)

   HTTP 响应标头 **`Content-Security-Policy`** 允许站点管理者控制用户代理能够为指定的页面加载哪些资源。除了少数例外情况，设置的政策主要涉及指定源服务器和脚本端点。这将帮助防止[跨站脚本攻击](https://developer.mozilla.org/zh-CN/docs/Glossary/Cross-site_scripting)。

   ```html
   <meta http-equiv="Content-Security-Policy" 
         content="default-src 'self'; script-src 'unsafe-inline'">
   ```

2. CORS控制

   ```javascript
   // 通过 fetch 发送跨域请求
   // mode: 'cors' 表示使用 CORS 协议进行跨域请求
   // credentials: 'include' 表示发送请求时包含用户凭证（cookies）
   fetch('https://example.com', {
       mode: 'cors',
       credentials: 'include'
   });
   ```

#### 3.10 移动端适配方案

1. Viewport元标签

   ```html
   <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
   ```

   设置了视口的属性，确保网页在移动设备上能够正确显示。

   - width=device-width：设置视口宽度等于设备宽度。
   - initial-scale=1.0：初始缩放比例为1。
   - maximum-scale=1.0：最大缩放比例为1。
   - user-scalable=no：禁止用户手动缩放。

2. 触摸事件

   ```javascript
   element.addEventListener('touchstart', (e) => {
     const touch = e.touches[0];
     console.log(`触点X: ${touch.clientX}`);
   });
   ```

#### 3.11实践建议

1. 优先使用语义化标签构建页面骨架

2. 利用FormData对象处理复杂表单提交

3. 渐进增强策略：检测功能支持情况

   ```javascript
   if('geolocation' in navigator) {
     // 使用定位功能
   } else {
     // 降级方案
   }
   ```

兼容性处理：

- 使用Modernizr检测特性支持
- 通过Polyfill实现兼容（如html5shiv）

性能考量：

- 避免过度使用Web Storage
- Canvas渲染注意内存释放
- Web Workers处理CPU密集型任务
