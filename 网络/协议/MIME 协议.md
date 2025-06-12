**媒体类型**（也通常称为**多用途互联网邮件扩展**或 **MIME** 类型）是一种标准，用来表示文档、文件或一组数据的性质和格式。

[互联网号码分配局（IANA）](https://www.iana.org/)负责跟踪所有官方 MIME 类型，你可以在[媒体类型](https://www.iana.org/assignments/media-types/media-types.xhtml)页面中找到最新的完整列表。

>**警告：** 浏览器通常使用 MIME 类型而不是文件扩展名来决定如何处理 URL，因此 Web 服务器在 [`Content-Type`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Headers/Content-Type) 响应标头中添加正确的 MIME 类型非常重要。如果配置不正确，浏览器可能会曲解文件内容，网站将无法正常工作，并且下载的文件也可能被错误处理。



### 一、MIME 类型的结构

---

MIME 类型通常仅包含两个部分：**类型（type）**和**子类型（subtype）**，中间由斜杠 `/` 分割，中间没有空白字符：

```
type/subtype
```

**类型**代表数据类型所属的大致分类，例如 `video` 或 `text`。

**子类型**标识了 MIME 类型所代表的指定类型的确切数据类型。以 `text` 类型为例，它的子类型包括：`plain`（纯文本）、`html`（[HTML](https://developer.mozilla.org/zh-CN/docs/Glossary/HTML) 源代码）、`calender`（iCalendar/`.ics` 文件）。

有一个可选的**参数**，能够提供额外的信息：

```
type/subtype;parameter=value
```

例如，对于主类型为 `text` 的任何 MIME 类型，可以添加可选的 `charset` 参数，以指定数据中的字符所使用的字符集。如果没有指定 `charset`，默认值为 ASCII（`US-ASCII`），除非被[用户代理的](https://developer.mozilla.org/zh-CN/docs/Glossary/User_agent)设置覆盖。要指定 UTF-8 文本文件，则使用 MIME 类型 `text/plain;charset=UTF-8`。

**MIME 类型对大小写不敏感，但是传统写法都是小写**。参数值可以是大小写敏感的。



### 二、独立类型

---

IANA 目前注册的独立类型如下：

- application
- audio
- example
- font
- image
- model
- text
- video



### 三、多部分类型

---

**多部分**类型指的是一类可分成不同部分的文件，其各部分通常是不同的 MIME 类型；也可用于——尤其在电子邮件中——表示属于同一事务的多个独立文件。它们代表一个**复合文档**。

HTTP 不会特殊处理多部分文档：信息会被传输到浏览器（如果浏览器不知道如何显示文档，很可能会显示一个“另存为”窗口）。除了几个例外，在 [HTML 表单](https://developer.mozilla.org/zh-CN/docs/Learn_web_development/Extensions/Forms)的 [`POST`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Methods/POST) 方法中使用的 `multipart/form-data`，以及用来发送部分文档，与 [`206`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Status/206) `Partial Content` 一同使用的 `multipart/byteranges`。







































































































