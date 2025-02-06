### 一、border-radius

---

#### 1.1 定义

设置元素的外边框圆角。当使用一个半径时确定一个圆形，当使用两个半径时确定一个椭圆。这个（椭）圆与边框的交集形成圆角效果。

#### 1.2 使用

```css
border-radius: 1em/5em;

/* 等价于： */

border-top-left-radius: 1em 5em;
border-top-right-radius: 1em 5em;
border-bottom-right-radius: 1em 5em;
border-bottom-left-radius: 1em 5em;
```

```css
border-radius: 4px 3px 6px / 2px 4px;

/* 等价于： */

border-top-left-radius: 4px 2px;
border-top-right-radius: 3px 4px;
border-bottom-right-radius: 6px 2px;
border-bottom-left-radius: 3px 4px;
```

#### 1.3 属性值

| 值       | 描述                  |
| :------- | :-------------------- |
| *length* | 定义弯道的形状。      |
| *%*      | 使用%定义角落的形状。 |

