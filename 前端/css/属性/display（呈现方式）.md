### 一、display

---

#### 1.1 定义

设置元素是否被视为[块级或行级盒子](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_flow_layout)以及用于子元素的布局，例如[流式布局](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_flow_layout)、[网格布局](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_grid_layout)或[弹性布局](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_flexible_box_layout)。

形式上，**`display`** 属性设置元素的内部和外部的*显示类型*。外部类型设置元素参与流式布局；内部类型设置子元素的布局。一些 `display` 值在它们自己的单独规范中完整定义；例如，在 CSS 弹性盒模型的规范中，定义了声明 `display: flex` 时会发生的细节。

#### 1.2 使用

```css
/* 预组合值 */
display: block;
display: inline;
display: inline-block;
display: flex;
display: inline-flex;
display: grid;
display: inline-grid;
display: flow-root;

/* 生成盒子 */
display: none;
display: contents;

/* 多关键字语法 */
display: block flex;
display: block flow;
display: block flow-root;
display: block grid;
display: inline flex;
display: inline flow;
display: inline flow-root;
display: inline grid;

/* 其他值 */
display: table;
display: table-row; /* 所有的 table 元素 都有等效的 CSS display 值 */
display: list-item;

/* 全局值 */
display: inherit;
display: initial;
display: revert;
display: revert-layer;
display: unset;
```

#### 1.3 属性值

| 值                 | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| none               | 此元素不会被显示。                                           |
| block              | 此元素将显示为块级元素，此元素前后会带有换行符。             |
| inline             | 默认。此元素会被显示为内联元素，元素前后没有换行符。         |
| inline-block       | 行内块元素。（CSS2.1 新增的值）                              |
| list-item          | 此元素会作为列表显示。                                       |
| run-in             | 此元素会根据上下文作为块级元素或内联元素显示。               |
| compact            | CSS 中有值 compact，不过由于缺乏广泛支持，已经从 CSS2.1 中删除。 |
| marker             | CSS 中有值 marker，不过由于缺乏广泛支持，已经从 CSS2.1 中删除。 |
| table              | 此元素会作为块级表格来显示（类似 <table>），表格前后带有换行符。 |
| inline-table       | 此元素会作为内联表格来显示（类似 <table>），表格前后没有换行符。 |
| table-row-group    | 此元素会作为一个或多个行的分组来显示（类似 <tbody>）。       |
| table-header-group | 此元素会作为一个或多个行的分组来显示（类似 <thead>）。       |
| table-footer-group | 此元素会作为一个或多个行的分组来显示（类似 <tfoot>）。       |
| flow-root          | 生成一个块级元素盒，其会建立一个新的块级格式化上下文，定义格式化上下文的根元素。 |
| table-row          | 此元素会作为一个表格行显示（类似 <tr>）。                    |
| table-column-group | 此元素会作为一个或多个列的分组来显示（类似 <colgroup>）。    |
| table-column       | 此元素会作为一个单元格列显示（类似 <col>）                   |
| table-cell         | 此元素会作为一个表格单元格显示（类似 <td> 和 <th>）          |
| table-caption      | 此元素会作为一个表格标题显示（类似 <caption>）               |
| inherit            | 规定应该从父元素继承 display 属性的值。                      |