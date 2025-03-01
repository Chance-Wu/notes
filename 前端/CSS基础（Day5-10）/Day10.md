### 一、CSS预处理器（Sass基础）

---

CSS预处理器是一种工具，它允许开发者使用更高级的语法和功能来编写CSS代码，然后将其编译成浏览器能够识别的标准CSS。Sass（Syntactically Awesome Style Sheets）是其中一种流行的CSS预处理器。

#### 1.1 什么是Sass

Sass是一种CSS预处理器，它通过引入变量、嵌套、Mixins（混合）、继承等功能，使CSS的编写更加高效、可维护。它支持两种语法：

- **.sass**：基于缩进的语法，不使用大括号和分号。
- **.scss**：类似标准CSS的语法，使用大括号和分号（本文将以.scss为例）。

Sass文件需要编译成CSS才能在浏览器中使用。

#### 1.2 Sass基础

1. **全局安装Sass**

   ```sh
   npm install -g sass
   ```

2. **编译Sass**

   - 手动编译

     ```sh
     sass input.scss output.css
     ```

   - 自动监视并编译

     ```sh
     sass --watch input.scss:output.css
     ```

3. **变量**：可以用来存储颜色、字体大小等值，便于统一管理和修改。

   ```css
   $primary-color: #333;
   body {
     color: $primary-color;
   }
   ```

   编译后，$primary-color会被替换为#333。

4. **嵌套**：允许选择器嵌套，使代码结构更清晰，尤其适合处理复杂的HTML。

   ```css
   nav {
     ul {
       margin: 0;
       padding: 0;
       list-style: none;
     }
     li {
       display: inline-block;
     }
     a {
       display: block;
       padding: 6px 12px;
       text-decoration: none;
     }
   }
   ```

   编译后会生成标准的CSS层级选择器。

5. **导入**：通过@import指令，可以将多个Sass文件合并到一个文件中，便于模块化管理。

   ```css
   @import "variables";
   @import "header";
   ```

   注意：导入的文件名不需要加扩展名，Sass会自动识别。

6. **Mixins（混合）**：Mixins允许定义可重用的代码块，可以带参数：

   ```css
   @mixin border-radius($radius) {
     -webkit-border-radius: $radius;
     -moz-border-radius: $radius;
     border-radius: $radius;
   }
   .box {
     @include border-radius(10px);
   }
   ```

   编译后，`.box`会包含所有浏览器前缀的圆角样式。

7. **继承**：使用`@extend`，一个选择器可以继承另一个选择器的样式，避免代码重复：

   ```css
   .message {
     border: 1px solid #ccc;
     padding: 10px;
     color: #333;
   }
   .success {
     @extend .message;
     border-color: green;
   }
   ```

   编译后，`.success`会包含`.message`的所有样式，并覆盖`border-color`。

8. **运算**：Sass支持加减乘除等数学运算，方便动态计算样式值：

9. 







































































































































