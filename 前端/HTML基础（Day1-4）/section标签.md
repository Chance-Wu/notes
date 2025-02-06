### 一、section

---

#### 1.1 定义

- 语义化标签。
- 用于定义文档中的一个部分或章节。
- 有助于提高网页内容的结构清晰度和可读性，同时对搜索引擎优化（SEO）和无障碍访问也很有帮助。

#### 1.2 使用

- **组织内容**：`<section>` 标签用来将文档分割成不同的主题部分。例如，新闻文章的不同段落、常见问题解答的不同问题等都可以用 `<section>` 来划分。
- **增强语义**：使用 `<section>` 标签代替无语义的 `<div>` 标签可以增加代码的可读性和含义，使浏览器、开发者和其他软件更容易理解页面的结构。

```html
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
</head>
<body>

<section>
  <h1>WWF</h1>
  <p>The World Wide Fund for Nature (WWF) is an international organization working on issues regarding the conservation, research and restoration of the environment, formerly named the World Wildlife Fund. WWF was founded in 1961.</p>
</section>

<section>
  <h1>WWF's Panda symbol</h1>
  <p>The Panda has become the symbol of WWF. The well-known panda logo of WWF originated from a panda named Chi Chi that was transferred from the Beijing Zoo to the London Zoo in the same year of the establishment of WWF.</p>
</section>

</body>
</html>
```