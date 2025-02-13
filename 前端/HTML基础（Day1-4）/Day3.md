- 多媒体元素（video/audio）
- iframe嵌套
- 元数据标签
- 实战：创建视频展示页



### 一、多媒体元素video/audio

---

#### 1.1 视频元素`<video>`

1. 基础视频嵌入

   ```html
   <video 
     src="nature.mp4" 
     controls 
     width="800" 
     height="450"
     poster="preview.jpg"
     preload="metadata"
     playsinline
     muted
     loop>
     您的浏览器不支持视频播放
   </video>
   ```

   | 属性          | 作用           | 推荐值                 |
   | :------------ | :------------- | :--------------------- |
   | `controls`    | 显示默认控件   | 始终添加               |
   | `preload`     | 预加载策略     | `metadata`（平衡性能） |
   | `poster`      | 封面图URL      | 建议尺寸16:9           |
   | `playsinline` | 移动端内联播放 | 必加                   |
   | `muted`       | 初始静音       | 配合自动播放使用       |
   | `loop`        | 循环播放       | 适用于背景视频         |

2. 多源格式兼容

   ```html
   <video controls>
     <!-- 格式优先级：MP4 > WebM > Ogg -->
     <source src="video.mp4" type="video/mp4; codecs=avc1.42E01E,mp4a.40.2">
     <source src="video.webm" type="video/webm; codecs=vp9,opus">
     <source src="video.ogv" type="video/ogg; codecs=theora,vorbis">
   </video>
   ```

3. 编码建议

   - **MP4**：H.264视频 + AAC音频（最广泛兼容）
   - **WebM**：VP9视频 + Opus音频（高质量压缩）
   - 分辨率适配：准备720P/1080P/4K多版本源

#### 1.2 音频元素`<audio>`

1. 基础音频播放

   ```html
   <audio 
     controls 
     src="background.mp3"
     crossorigin="anonymous"
     preload="none"
     volume="0.5">
     您的浏览器不支持音频播放
   </audio>
   ```

2. 高级控制实现：

   ```javascript
   const audio = document.querySelector('audio');
   
   // 自定义播放器
   document.getElementById('playBtn').onclick = () => audio.play();
   document.getElementById('time').oninput = (e) => {
     audio.currentTime = e.target.value * audio.duration;
   };

3. 可视话音频

   ```html
   <canvas id="visualizer" width="600" height="100"></canvas>
   <script>
     const ctx = new AudioContext();
     const analyser = ctx.createAnalyser();
     const source = ctx.createMediaElementSource(audio);
     source.connect(analyser);
     analyser.connect(ctx.destination);
     
     // 绘制频谱
     function draw() {
       requestAnimationFrame(draw);
       const data = new Uint8Array(analyser.frequencyBinCount);
       analyser.getByteFrequencyData(data);
       // 在canvas绘制波形...
     }
     draw();
   </script>
   ```

#### 1.3 字母与多语言支持

1. WebVTT字幕格式

   ```html
   <video>
     <track 
       label="中文" 
       kind="subtitles" 
       srclang="zh" 
       src="subs.vtt" 
       default>
   </video>
   ```

   字幕文件示例：

   ```
   WEBVTT
   
   00:00:01.000 --> 00:00:04.000
   <v 解说员>欢迎来到自然世界纪录片
   
   00:00:05.500 --> 00:00:08.200
   <v 旁白>这里展示的是非洲大草原的黎明

2. 多语言切换

   ```javascript
   function switchSubtitle(lang) {
     const tracks = video.textTracks;
     for (let track of tracks) {
       track.mode = track.language === lang ? 'showing' : 'hidden';
     }
   }

#### 1.4 高级媒体功能

1. 画中画模式

   ```javascript
   const pipBtn = document.createElement('button');
   pipBtn.textContent = '画中画';
   pipBtn.onclick = () => video.requestPictureInPicture();
   video.parentNode.appendChild(pipBtn);

2. 媒体流捕获

   ```javascript
   // 获取摄像头输入
   navigator.mediaDevices.getUserMedia({ video: true })
     .then(stream => {
       const liveVideo = document.createElement('video');
       liveVideo.srcObject = stream;
       document.body.appendChild(liveVideo);
     });

#### 1.5 性能优化策略

1. 懒加载视频

   ```html
   <video 
     data-src="video.mp4" 
     loading="lazy" 
     onmouseover="this.src = this.dataset.src">
   </video>

2. 自适应流媒体

   ```javascript
   <!-- 使用HLS协议 -->
   <script src="https://cdn.jsdelivr.net/npm/hls.js@latest"></script>
   <script>
     if(Hls.isSupported()) {
       const hls = new Hls();
       hls.loadSource('https://example.com/video.m3u8');
       hls.attachMedia(video);
     }
   </script>

#### 1.6 安全与权限

1. 自动播放策略

   ```javascript
   // 用户交互后启用声音
   document.addEventListener('click', () => {
     video.muted = false;
     video.play();
   }, { once: true });
   ```

2. CORS配置

   ```html
   <!-- 服务端需设置 -->
   Access-Control-Allow-Origin: *
   Access-Control-Expose-Headers: Content-Length
   ```



### 二、iframe嵌套

---

#### 2.1 基础嵌套语法与属性

1. 基本嵌套示例

   ```html
   <iframe 
     src="https://example.com" 
     width="800" 
     height="600" 
     title="示例网站"
     loading="lazy"
     sandbox="allow-scripts allow-same-origin"
     allow="fullscreen"
   ></iframe>

2. 核心属性解析

   | 属性           | 作用           | 推荐值                        |
   | :------------- | :------------- | :---------------------------- |
   | `src`          | 资源地址       | URL或相对路径                 |
   | `width/height` | 尺寸控制       | 数值或百分比                  |
   | `title`        | 无障碍访问必需 | 描述性文本                    |
   | `loading`      | 加载策略       | `lazy`（延迟加载）            |
   | `sandbox`      | 安全限制       | 按需启用权限                  |
   | `allow`        | 功能授权       | `fullscreen`, `geolocation`等 |

#### 2.2 安全防护措施

1. 沙盒机制（Sandboxing）

   ```html
   <!-- 最低权限配置 -->
   <iframe 
     sandbox="allow-scripts allow-forms" 
     src="form.html">
   </iframe>
   
   <!-- 完全锁定 -->
   <iframe sandbox src="..."></iframe>
   ```

   **沙盒权限列表**：

   - `allow-forms`：允许表单提交
   - `allow-popups`：允许弹出窗口
   - `allow-same-origin`：保持同源策略
   - `allow-scripts`：允许执行脚本
   - `allow-top-navigation`：允许修改父页面URL

2. 安全头设置

   ```http
   # 服务端阻止被嵌套
   X-Frame-Options: DENY
   Content-Security-Policy: frame-ancestors 'none';
   ```

#### 2.3 性能优化方案

1. 懒加载技术

   ```html
   <iframe 
     data-src="heavy-content.html" 
     loading="lazy"
     onvisible="this.src=this.dataset.src">
   </iframe>

2. 异步加载实践

   ```javascript
   // 动态插入iframe
   window.addEventListener('load', () => {
     const iframe = document.createElement('iframe');
     iframe.src = 'widget.html';
     document.getElementById('container').appendChild(iframe);
   });
   ```

3. 资源限制

   ```html
   <iframe 
     src="https://maps.example.com" 
     importance="low"
     referrerpolicy="no-referrer-when-downgrade">
   </iframe>

#### 2.4 跨域通信机制

父子页面通信

父页面：

```javascript
const iframe = document.querySelector('iframe');

// 发送消息
iframe.contentWindow.postMessage(
  { type: 'UPDATE_COLOR', color: '#ff0000' }, 
  'https://child.example.com'
);

// 接收消息
window.addEventListener('message', (e) => {
  if(e.origin !== 'https://child.example.com') return;
  console.log('收到消息:', e.data);
});
```

子页面：

```javascript
// 接收消息
window.addEventListener('message', (e) => {
  if(e.origin !== 'https://parent.example.com') return;
  document.body.style.backgroundColor = e.data.color;
});

// 发送消息
parent.postMessage({ status: 'READY' }, 'https://parent.example.com');
```

#### 2.5 应用场景

1. 第三发给服务集成

   ```html
   <!-- 谷歌地图 -->
   <iframe
     src="https://maps.google.com/maps?q=Beijing&output=embed"
     allowfullscreen
     style="border:0">
   </iframe>
   
   <!-- 在线文档 -->
   <iframe 
     src="https://docs.google.com/document/..."
     sandbox="allow-scripts allow-same-origin">
   </iframe>
   ```

2. 微前端架构

   ```html
   <!-- 主应用 -->
   <div id="app1-container">
     <iframe src="https://app1.example.com"></iframe>
   </div>
   
   <div id="app2-container">
     <iframe src="https://app2.example.com"></iframe>
   </div>
   ```

#### 2.6 调试与问题排查

1. 控制台访问

   ```javascript
   // 访问子页面控制台（同源时）
   const iframeConsole = iframe.contentWindow.console;
   iframeConsole.log('来自父页面的日志');
   
   // 跨域错误捕获
   iframe.onerror = (e) => {
     console.error('iframe加载失败:', e);
   };

2. 性能检测

   ```javascript
   const perfObserver = new PerformanceObserver((list) => {
     list.getEntries().forEach(entry => {
       if(entry.name === iframe.src) {
         console.log('iframe加载耗时:', entry.duration);
       }
     });
   });
   perfObserver.observe({ type: 'resource' });

#### 2.7 实战：安全仪表盘

```html
<!DOCTYPE html>
<html>
<head>
    <style>
        .dashboard {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            padding: 20px;
        }
        .widget {
            border: 1px solid #ddd;
            border-radius: 8px;
            overflow: hidden;
        }
    </style>
</head>
<body>
    <div class="dashboard">
        <div class="widget">
            <iframe 
                src="https://monitor.example.com/cpu" 
                title="CPU监控"
                sandbox="allow-scripts allow-same-origin"
                loading="lazy"
            ></iframe>
        </div>
        
        <div class="widget">
            <iframe 
                src="https://monitor.example.com/network" 
                title="网络流量"
                sandbox="allow-scripts allow-same-origin"
                loading="lazy"
            ></iframe>
        </div>
    </div>

    <script>
        // 统一接收监控数据
        window.addEventListener('message', (e) => {
            if(e.origin !== 'https://monitor.example.com') return;
            
            switch(e.data.type) {
                case 'CPU_ALERT':
                    showAlert(`CPU过载: ${e.data.value}%`);
                    break;
                case 'NETWORK_ANOMALY':
                    showAlert(`网络异常: ${e.data.msg}`);
                    break;
            }
        });
    </script>
</body>
</html>
```

#### 2.8 禁止使用的场景

1. **敏感信息展示**
   ❌ 避免嵌入银行/支付页面
2. **核心功能依赖**
   ❌ 不要用iframe实现登录表单
3. **SEO关键内容**
   ❌ 重要文本内容不应放在iframe中

#### 2.9 最佳实践清单

1. 始终添加`title`属性
2. 非必要资源使用`loading="lazy"`
3. 严格设置`sandbox`权限
4. 监控iframe资源加载性能
5. 定期审查第三方内容安全性
6. 使用`X-Frame-Options`保护自有页面

掌握iframe的深度应用后，您将能安全高效地集成第三方服务、构建微前端架构，并实现复杂的页面模块化功能。



### 三、元数据标签

---

#### 3.1 核心元数据标签

1. `<title>`标题标签

   ```html
   <title>前端开发教程 | 慕课网</title>
   ```

   作用：

   - 浏览器标签页显示内容
   - 搜索引擎结果首要展示信息
   - 影响SEO权重（关键词布局）

   最佳实践：

   - 长度控制在50-60字符
   - 包含品牌词和核心关键词
   - 避免堆砌关键词

2. `<meta>`元信息标签

   **字符编码声明**

   ```html
   <meta charset="UTF-8">
   ```

   强制要求：

   - 必须放在文档最前面
   - 推荐统一使用UTF-8编码

   **视口控制**

   ```html
   <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=5.0, user-scalable=yes">
   ```

   | 参数            | 作用     | 推荐值         |
   | :-------------- | :------- | :------------- |
   | `width`         | 视口宽度 | `device-width` |
   | `initial-scale` | 初始缩放 | `1.0`          |
   | `maximum-scale` | 最大缩放 | ≤`5.0`         |
   | `user-scalable` | 允许缩放 | `yes/no`       |

3. SE0优化标签

   ```html
   <meta name="description" content="慕课网提供专业前端开发教程，涵盖HTML5/CSS3/JavaScript等核心技术">
   <meta name="keywords" content="前端开发,HTML教程,CSS学习,JavaScript入门">
   <meta name="author" content="慕课网教学团队">
   ```

4. 社交媒体优化

   ```html
   <!-- Open Graph协议 -->
   <meta property="og:title" content="免费前端开发课程">
   <meta property="og:image" content="https://example.com/cover.jpg">
   
   <!-- Twitter Cards -->
   <meta name="twitter:card" content="summary_large_image">

#### 3.2 高级元数据应用

1. 搜索引擎指令

   ```html
   <meta name="robots" content="index, follow, max-image-preview:large">
   <meta name="googlebot" content="notranslate">
   ```

   **常见指了值**：

   | 指令        | 作用       |
   | :---------- | :--------- |
   | `noindex`   | 禁止收录   |
   | `nofollow`  | 不追踪链接 |
   | `noarchive` | 不缓存快照 |

2. 安全策略控制

   ```html
   <!-- 内容安全策略 -->
   <meta http-equiv="Content-Security-Policy" content="default-src 'self'">
   
   <!-- 强制HTTPS -->
   <meta http-equiv="Strict-Transport-Security" content="max-age=31536000">
   
   <!-- 防止点击劫持 -->
   <meta http-equiv="X-Frame-Options" content="DENY">

#### 3.3 资源关联标签

1. 样式表关联

   ```html
   <link rel="stylesheet" href="main.css" media="print">
   ```

   媒体查询控制：

   ```html
   <link rel="stylesheet" href="mobile.css" media="(max-width: 600px)">

2. 网站图标定义

   ```html
   <!-- 现代多尺寸方案 -->
   <link rel="icon" href="/favicon.ico" sizes="any">
   <link rel="icon" href="/icon.svg" type="image/svg+xml">
   <link rel="apple-touch-icon" href="/apple-touch-icon.png">
   ```

3. 预加载优化

   ```html
   <link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>
   <link rel="preconnect" href="https://cdn.example.com">

#### 3.4 HTML5新增元数据

1. 移动端适配增强

   ```html
   <!-- 苹果全屏模式 -->
   <meta name="apple-mobile-web-app-capable" content="yes">
   
   <!-- 安卓状态栏颜色 -->
   <meta name="theme-color" content="#2196f3">
   ```

2. PWA应用配置

   ```html
   <!-- 应用清单 -->
   <link rel="manifest" href="/manifest.webmanifest">
   
   <!-- 应用安装提示 -->
   <meta name="apple-itunes-app" content="app-id=123456">
   ```

#### 3.5 典型配置示例

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <!-- 基础元数据 -->
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>前端开发入门教程 - 慕课网</title>
    <meta name="description" content="零基础学习HTML5/CSS3/JavaScript，30天掌握前端开发核心技术">
    
    <!-- 社交媒体优化 -->
    <meta property="og:type" content="website">
    <meta property="og:url" content="https://imooc.com/frontend">
    <meta property="og:image" content="https://imooc.com/og-image.jpg">

    <!-- 安全策略 -->
    <meta http-equiv="Content-Security-Policy" content="default-src 'self'">
    
    <!-- 资源预加载 -->
    <link rel="preload" href="main.css" as="style">
    <link rel="preconnect" href="https://fonts.gstatic.com">
    
    <!-- 样式表 -->
    <link rel="stylesheet" href="main.css">
    
    <!-- 网站图标 -->
    <link rel="icon" href="/favicon.ico">
    <link rel="apple-touch-icon" href="/apple-touch-icon.png">
    
    <!-- PWA配置 -->
    <link rel="manifest" href="/manifest.webmanifest">
    <meta name="theme-color" content="#ffffff">
</head>
<body>
    <!-- 页面内容 -->
</body>
</html>
```

#### 3.6 最佳实践清单

1. 始终声明`charset`为UTF-8
2. 优先使用`og:image`尺寸1200x630像素
3. 关键CSS使用`preload`预加载
4. 定期审查`Content-Security-Policy`配置
5. 为PWA应用配置`manifest`文件
6. 使用`theme-color`统一品牌色调