### 一、padding

---

#### 1.1 定义

一个元素的内边距区域指的是其内容与其边框之间的空间。

>**备注：** 内边距创建元素**内部**的额外空间。相反，[`margin`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/margin) 创建元素**外部**的额外空间。

#### 1.2 使用

```css
/* 应用于所有边 */
padding: 1em;

/* 上边下边 | 左边右边 */
padding: 5% 10%;

/* 上边 | 左边右边 | 下边 */
padding: 1em 2em 2em;

/* 上边 | 右边 | 下边 | 左边 */
padding: 5px 1em 0 2em;

/* 全局值 */
padding: inherit;
padding: initial;
padding: revert;
padding: revert-layer;
padding: unset;
```

#### 1.3 属性值

| 值       | 说明                                                     |
| :------- | :------------------------------------------------------- |
| *length* | 规定以具体单位计的填充值，比如像素、厘米等。默认值是 0px |
| *%*      | 规定基于父元素的宽度的百分比的填充                       |
| inherit  | 指定应该从父元素继承padding                              |