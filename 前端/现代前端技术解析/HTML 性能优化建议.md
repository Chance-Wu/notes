1. **将 `<script>` 标签放在 `</body>` 前**
   避免阻塞页面解析，提升首屏渲染速度。

2. **使用 `async` 或 `defer` 加载非关键脚本**

   ```html
   <script src="app.js" defer></script>
   ```

3. **尽早输出 HTML 内容**
   服务端应尽快返回 `<html><head>...`，让浏览器尽早开始解析。

4. **避免 `document.write()`**
   它会强制重新解析文档，性能差且不安全。

5. **预加载关键资源**
   使用 `<link rel="preload">` 提前加载字体、CSS 等关键资源。