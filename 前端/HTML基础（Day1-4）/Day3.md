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

1. 沙盒机制
2. 























































































