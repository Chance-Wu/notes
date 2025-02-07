### 一、gap

---

#### 1.1 定义

设置行与列之间的间隙。

#### 1.2 使用

```css
/* 一个 <length> 值 */
gap: 20px;
gap: 1em;
gap: 3vmin;
gap: 0.5cm;

/* 一个 <percentage> 值 */
gap: 16%;
gap: 100%;

/* 两个 <length> 值 */
gap: 20px 10px;
gap: 1em 0.5em;
gap: 3vmin 2vmax;
gap: 0.5cm 2mm;

/* 一个或两个 <percentage> 值 */
gap: 16% 100%;
gap: 21px 82%;

/* calc() 值 */
gap: calc(10% + 20px);
gap: calc(20px + 10%) calc(10% - 5px);

/* 全局值 */
gap: inherit;
gap: initial;
gap: revert;
gap: revert-layer;
gap: unset;
```

#### 1.3 属性值

| 值         | 描述                           |
| :--------- | :----------------------------- |
| *row-gap*  | 设置网格布局中行之间的间隙大小 |
| column-gap | 设置网格布局中列之间的间隙大小 |



### 二、row-gap

---

列间隙



### 三、column-gap

---

行间隙