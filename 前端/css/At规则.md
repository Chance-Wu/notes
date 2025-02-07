**At 规则**是一个 CSS 语句，用来指示 CSS 如何运行。以 at 符号开头，'`@`'，后跟一个标识符，并包括直到下一个分号的所有内容，'`;`'，或下一个 CSS 块，以先到者为准。

### 一、语法

---

#### 1.1 规则

```css
/* 一般结构 */
@identifier (RULE);

/* 示例：通知浏览器使用 UTF-8 字符集 */
@charset "utf-8";
```

下面是一些 at 规则，由它们的标示符指定，每种规则都有不同的语法：

- `@charset`——定义样式表使用的字符集。
- `@import`——告诉 CSS 引擎引入一个外部样式表。
- `@namespace`——告诉 CSS 引擎必须考虑 XML 命名空间。

#### 1.2 嵌套

```css
@identifier (RULE) {
}
```

嵌套 at 规则，是嵌套语句的子集，不仅可以作为样式表里的一个语句，也可以用在条件规则组里：

- `@media`——如果满足媒介查询的条件则条件规则组里的规则生效。
- `@supports`——如果满足给定条件则条件规则组里的规则生效。实验性
- `@document`——如果文档样式表满足给定条件则条件规则组里的规则生效。（推迟至 CSS Level 4 规范）
- `@page`——描述打印文档时布局的变化。
- `@font-face`——描述将下载的外部的字体。实验性
- `@keyframes`——描述 CSS 动画的中间步骤。实验性
- `@counter-style`——定义不属于预定义样式集的特定计数器样式。（在候选推荐阶段，仅在 Gecko 中实现）
- `@font-feature-values`（加上 `@swash`、`@ornaments`、`@annotation`、`@stylistic`、`@styleset` 和 `@character-variant`）——在 [`font-variant-alternates`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/font-variant-alternates) 中定义通用名称，以便在 OpenType 中以不同方式激活功能。（在候选推荐阶段，仅在 Gecko 中实现）
- `@property` 实验性——描述自定义属性和变量。（目前处于工作草案阶段）
- `@layer`——声明一个级联层，并在有多个级联层时定义优先顺序。



### 二、条件规则组

---

就像属性值那样，每条 at 规则都有不同的语法。不过一些 @规则可以归为一个特殊的分类：**条件规则组**。这些语句使用相同的语法。它们都可以包括 嵌套语句——规则集或者是嵌套 at 规则。它们都表达：它们所指的条件 (类型不同) 总等效于 **true** 或者 **false**，如果为 **true** 那么它们之中的语句生效。

条件规则组由 [CSS Conditionals Level 3](https://drafts.csswg.org/css-conditional-3/) 定义：

- `@media`
- `@supports`
- `@document`（推迟至 CSS Level 4 规范）

既然条件规则组可以嵌套语句，那么嵌套层级不定。



### 三、使用CSS嵌套来嵌套@layer

---

[级联层](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@layer)可以嵌套以[创建嵌套层](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@layer#嵌套层)。它们用 `.`（点）连接。这也可以使用 [CSS 嵌套](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_nesting/Nesting_at-rules#嵌套级联层（layer）)来实现。



### 四、索引

---

- [`@charset`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@charset)
- [`@color-profile`](https://developer.mozilla.org/en-US/docs/Web/CSS/@color-profile)
- [`@container`](https://developer.mozilla.org/en-US/docs/Web/CSS/@container)
- [`@counter-style`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@counter-style)
- [`@font-face`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@font-face)
- [`@font-feature-values`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@font-feature-values)
- [`@font-palette-values`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-palette-values)
- [`@import`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@import)
- [`@keyframes`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@keyframes)
- [`@layer`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@layer)
- [`@media`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@media)
- [`@namespace`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@namespace)
- [`@page`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@page)
- [`@property`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@property) 实验性
- [`@supports`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@supports)



### 五、@media

---

基于一个或多个媒体查询的结果来应用样式表的一部分。使用它，你可以指定一个媒体查询和一个 CSS 块，当且仅当该媒体查询与正在使用其内容的设备匹配时，该 CSS 块才能应用于该文档。

>**备注：** 在 JavaScript 中，可以使用 [`CSSMediaRule`](https://developer.mozilla.org/zh-CN/docs/Web/API/CSSMediaRule) CSS 对象模型接口访问使用 `@media` 创建的规则。

#### 5.1 示例

```css
@media (max-width: 600px) {
    body {
        padding: 10px;
    }

    .section {
        padding: 15px;
    }
}
```

定义当视口（浏览器窗口）宽度小于或等于 600 像素时应用的样式规则。

#### 5.2 语法

`@media` at 规则可置于你代码的顶层或嵌套至其他任何的 [at 条件规则组](https://developer.mozilla.org/zh-CN/docs/Web/CSS/At-rule#条件规则组)中。

```css
/* 在你的代码的顶层 */
@media screen and (min-width: 900px) {
  article {
    padding: 1rem 3rem;
  }
}

/* 嵌套至其他的 at 条件规则中 */
@supports (display: flex) {
  @media screen and (min-width: 900px) {
    article {
      display: flex;
    }
  }
}
```

#### 5.3 描述

媒体类型（media type）描述设备的一般类别。除非使用 `not` 或 `only` 逻辑运算符，否则媒体类型是可选的，并且会（隐式地）应用 `all` 类型。

- `all`

  适用于所有设备。

- `print`

  适用于在打印预览模式下在屏幕上查看的分页材料和文档。（有关特定于这些格式的格式问题的信息，请参阅[分页媒体](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_paged_media)。）

- `screen`

  主要用于屏幕。

#### 5.4 媒体特性

媒体特性（media feature）描述了[用户代理](https://developer.mozilla.org/zh-CN/docs/Glossary/User_agent)、输出设备或环境的具体特征。媒体特性表达式是完全可选的，其用于测试这些特征是否存在以及它们的值。每个媒体特性表达式都必须用括号括起来。

- `any-hover`：是否有任何可用的输入机制允许用户（将鼠标等）悬停在元素上？于媒体查询第 4 版中被添加。
- `any-pointer`：可用的输入机制中是否有任何指针设备，如果有，它的精度如何？于媒体查询第 4 版中被添加。
- `aspect-ratio`：视口（viewport）的宽高比。
- `color`：输出设备每个颜色分量的比特值，如果设备不支持输出彩色，则该值为 0。
- `color-gamut`：用户代理和输出设备大致程度上支持的色域。于媒体查询第 4 版中被添加。
- `color-index`：输出设备的颜色查询表（color lookup table）中的条目数量，如果设备不使用颜色查询表，则该值为 0。
- `display-mode`：应用程序的显示模式，显示模式由 web 应用的清单（manifest）中的 [`display`](https://developer.mozilla.org/zh-CN/docs/Web/Manifest#display) 成员所指定。定义于 [Web App Manifest 规范](https://w3c.github.io/manifest/#the-display-mode-media-feature)。
- `dynamic-range`：用户代理和输出设备支持的亮度、对比度和色彩深度的组合。于媒体查询第 5 版中被添加。
- `forced-colors`：检测用户代理是否限制调色板。于媒体查询第 5 版中被添加。
- `grid`：输出设备使用网格屏幕还是点阵屏幕？
- `height`：视口的高度。
- `hover`：主输入机制是否允许用户在元素上悬停。于媒体查询第 4 版中被添加。
- `inverted-colors`：用户代理或者底层操作系统是否反转了颜色。于媒体查询第 5 版中被添加。
- `monochrome`：输出设备单色帧缓冲区中每个像素的位深度。如果设备并非单色屏幕，则该值为 0。
- `orientation`：视口的旋转方向。
- `overflow-block`：输出设备如何处理沿块轴溢出视口的内容。于媒体查询第 4 版中被添加。
- `overflow-inline`：沿行轴溢出视口的内容是否可以滚动。于媒体查询第 4 版中被添加。
- `pointer`：主输入机制是一个指针设备吗？如果是，它的精度如何？于媒体查询第 4 版中被添加。
- `prefers-color-scheme`：检测用户倾向于选择亮色还是暗色的配色方案。于媒体查询第 5 版中被添加。
- `prefers-contrast`：检测用户是否有向系统要求提高或降低相近颜色之间的对比度。于媒体查询第 5 版中被添加。
- `prefers-reduced-motion`：用户是否希望页面上出现更少的动态效果。于媒体查询第 5 版中被添加。
- `resolution`：输出设备的像素密度（分辨率）。
- `scripting`：检测脚本（例如 JavaScript）是否可用。于媒体查询第 5 版中被添加。
- `update`实验性：输出设备修改渲染内容的频率。于媒体查询第 4 版中被添加。
- `video-dynamic-range`：用户代理的视频平面（video plane）和输出设备支持的亮度、对比度及色彩深度的组合。于媒体查询第 5 版中被添加。
- `width`：视口（包括纵向滚动条）的宽度。

#### 5.5 运算逻辑符

逻辑运算符（logical operator）`not`、`and`、`only` 和 `or` 可用于联合构造复杂的媒体查询，你还可以通过用逗号分隔多个媒体查询，将它们组合为一个规则。

- `and`：用于将多个媒体查询规则组合成单条媒体查询，当每个查询规则都为真时则该条媒体查询为 `true`，它还用于将媒体特性与媒体类型结合在一起。
- `not`：用于否定媒体查询，如果不满足这个条件则返回 `true`，否则返回 `false`。如果出现在以逗号分隔的查询列表中，它将仅否定应用了该查询的特定查询。如果使用 `not` 运算符，则*还必须*指定媒体类型。**备注：** 在第 3 版中，`not` 关键字不能用于否定单个媒体特性表达式，而只能用于否定整个媒体查询。
- `only`：仅在整个查询匹配时才应用样式。这对于防止较老的浏览器应用所选样式很有用。当不使用 `only` 时，较老的浏览器会将 `screen and (max-width: 500px)` 简单地解释为 `screen`，忽略查询的其余部分，并将其样式应用于所有屏幕。如果使用 `only` 运算符，则*还必须*指定媒体类型。
- `,`（逗号）：逗号用于将多个媒体查询合并为一个规则。逗号分隔列表中的每个查询都与其他查询分开处理。因此，如果列表中的任何查询为 `true`，则整个媒体查询语句返回 `true`。换句话说，列表的行为类似于逻辑或（`or`）运算符。
- `or`：等价于 `,` 运算符。于媒体查询第 4 版中被添加。