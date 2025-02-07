### 一、At规则

---

**At 规则**是一个 CSS 语句，用来指示 CSS 如何运行。以 at 符号开头，'`@`'，后跟一个标识符，并包括直到下一个分号的所有内容，'`;`'，或下一个 CSS 块，以先到者为准。

#### 1.1 语法

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

- [`@media`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@media)——如果满足媒介查询的条件则条件规则组里的规则生效。
- [`@supports`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@supports)——如果满足给定条件则条件规则组里的规则生效。实验性
- [`@document`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@document)——如果文档样式表满足给定条件则条件规则组里的规则生效。*（推迟至 CSS Level 4 规范）*
- [`@page`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@page)——描述打印文档时布局的变化。
- [`@font-face`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@font-face)——描述将下载的外部的字体。实验性
- [`@keyframes`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@keyframes)——描述 CSS 动画的中间步骤。实验性
- [`@counter-style`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@counter-style)——定义不属于预定义样式集的特定计数器样式。*（在候选推荐阶段，仅在 Gecko 中实现）*
- [`@font-feature-values`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@font-feature-values)（加上 `@swash`、`@ornaments`、`@annotation`、`@stylistic`、`@styleset` 和 `@character-variant`）——在 [`font-variant-alternates`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/font-variant-alternates) 中定义通用名称，以便在 OpenType 中以不同方式激活功能。*（在候选推荐阶段，仅在 Gecko 中实现）*
- [`@property`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@property) 实验性——描述自定义属性和变量。*（目前处于工作草案阶段）*
- [`@layer`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@layer)——声明一个级联层，并在有多个级联层时定义优先顺序。

















































































































