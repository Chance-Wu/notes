1. **减少关键 CSS 的体积**

   - 将首屏关键 CSS 内联到 `<head>` 中（Critical CSS）。

   - 异步加载非关键 CSS：

     ```html
     <link rel="preload" href="styles.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
     ```

2. **避免使用 `@import`**

   - `@import` 会阻塞 CSS 解析，且无法并行下载。

3. **使用媒体查询分离样式**

   ```html
   <link rel="stylesheet" href="print.css" media="print">
   ```

   - 浏览器不会为不匹配当前设备的媒体类型阻塞渲染。

4. **避免过多复杂选择器**

   - 如 `div * .header > ul li a`，虽然不影响解析速度，但影响**样式匹配性能**。

5. **尽早加载 CSS**

   - 将 `<link rel="stylesheet">` 放在 `<head>` 中，让浏览器尽早开始下载和解析。