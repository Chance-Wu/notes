### 一、grid-template-columns

---

#### 1.1 定义

基于[网格列](https://developer.mozilla.org/zh-CN/docs/Glossary/Grid_Column)的维度，去定义网格线的名称和网格轨道的尺寸大小。

#### 1.2 使用

```css
grid-template-columns: <track-size> ... | <line-name> <track-size> <line-name> ... | repeat() | auto-fill | auto-fit | minmax();
```

- `<track-size>`：可以是长度值（如 `px`, `rem`, `%`）、比例单位（如 `fr`）或其他关键字（如 `auto`）。
- `<line-name>`：可选，为行或列定义自定义名称。
- 函数如 `repeat()`, `auto-fill`, `auto-fit`, 和 `minmax()` 提供了更复杂的布局定义方式。

定义一个包含三列的网格布局，每列宽度分别为100px,200px和300px

```css
.grid-container {
  display: grid;
  grid-template-columns: 100px 200px 300px;
}
```

使用分数单位 fr 来按比例分配可用空间：

```css
.grid-container {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr; /* 第二列是其他两列的两倍宽 */
}
```

自动重复列，利用 `repeat()` 函数简化重复模式：

```css
.grid-container {
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* 等同于 1fr 1fr 1fr */
}
```

自适应布局，使用 `auto-fill` 或 `auto-fit` 关键字与 `minmax()` 函数一起创建自适应布局：

```css
.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr)); /* 根据容器宽度自动填充最小150px最大1fr的列 */
}
```

- `auto-fill`：即使在空余空间不足以容纳另一个列轨道时也会尽可能多地创建列轨道。
- `auto-fit`：功能类似 `auto-fill`，但在没有足够内容填充所有列时会折叠那些多余的轨道。

#### 1.3 属性值

| 描述          |                                                              |
| :------------ | ------------------------------------------------------------ |
| none          | 默认值，不指定列的大小。                                     |
| *auto*        | 列的大小由容器的大小和列中网格元素内容的大小决定。           |
| *max-content* | 每列的大小设置为该列中最大网格元素的大小。                   |
| *min-content* | 每列的大小设置为该列中最小网格元素的大小。                   |
| *length*      | 长度值，可以是 px 为单位的数值或百分比 %。 0 是默认值。 了解更多[长度单位](https://www.runoob.com/cssref/css-units.html)。 |
| initial       | 将此属性设置为默认值。 *[阅读关于 initial](https://www.runoob.com/cssref/css-initial.html)* |
| inherit       | 从父元素继承该属性。 *[阅读关于 inherit](https://www.runoob.com/cssref/css-inherit.html)* |