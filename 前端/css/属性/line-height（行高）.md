### 一、line-height

---

#### 1.1 定义

用于设置多行元素的空间量，如多行文本的间距。对于块级元素，它指定元素行盒（line boxes）的最小高度。对于非替代的 inline 元素，它用于计算行盒（line box）的高度。

#### 1.2 使用

```css
line-height: normal;

line-height: 3.5;

line-height: 3em;

line-height: 34%;

line-height: inherit;
line-height: initial;
line-height: unset;
```

#### 1.3 属性值

| 值       | 描述                                                 |
| :------- | :--------------------------------------------------- |
| normal   | 默认。设置合理的行间距。                             |
| *number* | 设置数字，此数字会与当前的字体尺寸相乘来设置行间距。 |
| *length* | 设置固定的行间距。                                   |
| *%*      | 基于当前字体尺寸的百分比行间距。                     |
| inherit  | 规定应该从父元素继承 line-height 属性的值。          |