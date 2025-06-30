### 解析过程

---

1. 识别 MIME 类型：首先，通过 Content-Type头部确定数据的 MIME 类型，如text/html、image/jpeg等。
2. 处理编码：根据 Content-Transfer-Encoding 头部，对数据进行解码，如 Base64 解码或 Quoted-Printable 解码。
3. 解析结构：对于 multipart 类型的消息，根据边界字符串解析各个部分，每个不同的 MIME 类型和内容。
4. 显示内容：根据 MIME 类型，使用相应的应用程序或组件显示内容，如使用图像查看器显示图片，使用文本编辑器显示文本等。







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

HTTP 不会特殊处理多部分文档：信息会被传输到浏览器（如果浏览器不知道如何显示文档，很可能会显示一个“另存为”窗口）。除了几个例外，在 [HTML 表单](https://developer.mozilla.org/zh-CN/docs/Learn_web_development/Extensions/Forms)的 [`POST`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Methods/POST) 方法中使用的 `multipart/form-data`，以及用来发送部分文档，[206](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Status/206) `Partial Content` 一同使用的 `multipart/byteranges`。

有两种多部分类型：

#### 3.1 message

封装其他信息的信息。例如，这可以用来表示将转发信息作为其数据一部分的电子邮件，或将超大信息分快发送，就像发送多条信息一样。例如，message/rfc822（用于转发或回复信息的引用）和 message/partial（允许将大段信息自动拆分成小段，由收件人重新组装）是两个常见的例子。

#### 3.2 multipart

由多个组件组成的数据，这些组件可能各自具有不同的 MIME 类型。例如，multipart/form-data（用于使用 FormData API 生成的数据）和 multipart/byteranges。



### 四、对 Web 开发者至关重要的 MIME 类型

---

#### 4.1 application/octet-stream

这是二进制文件的默认值。由于这意味着未知的二进制文件，浏览器一般不会自动执行或询问执行。浏览器将这些文件视为 `Content-Disposition` 标头被设置为 `attachment` 一样，并弹出“另存为”对话框。

>在常规的 HTTP 应答中，`Content-Disposition` 响应标头指示回复的内容该以何种形式展示，是以内联的形式（即网页或者页面的一部分），还是以附件的形式下载并保存到本地。
>
>在 `multipart/form-data` 类型的应答消息体中，`Content-Disposition` 通用标头可以被用在 multipart 消息体的子部分中，用来给出其对应字段的相关信息。各个子部分由在 [Content-Type](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Headers/Content-Type) 中定义的*边界*（boundary）分隔。用在消息体自身则无实际意义。
>
>`Content-Disposition` 标头最初是在 MIME 标准中定义的，HTTP 表单及 [`POST`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Methods/POST) 请求只用到了其所有参数的一个子集。只有 `form-data` 以及可选的 `name` 和 `filename` 三个参数可以应用在 HTTP 上下文中。
>
>```http
>Content-Disposition: inline
>Content-Disposition: attachment
>Content-Disposition: attachment; filename="filename.jpg"
>```

#### 4.2 text/plain

这是文本文件的默认值。即使它其实意味着未知的文本文件，但浏览器认为是可以直接展示的。

#### 4.3 text/css

在网页中要被解析为 CSS 的任何 CSS 文件**必须**指定 MIME 为 `text/css`。通常，如果服务器不识别 CSS 文件的 `.css` 后缀，则可能将它们以 MIME 为 `text/plain` 或 `application/octet-stream` 来发送给浏览器：在这种情况下，大多数浏览器不将其识别为 CSS 文件而直接忽略。

#### 4.4 text/html

所有的 HTML 内容都应该使用这种类型。XHTML 的其他 MIME 类型（如 `application/xml+html`）现在基本不再使用。

#### 4.5 text/javascript

根据 [IANA 媒体类型注册表](https://www.iana.org/assignments/media-types/media-types.xhtml#text)、[RFC 9239](https://www.rfc-editor.org/rfc/rfc9239.html) 和 [HTML 规范](https://html.spec.whatwg.org/multipage/scripting.html#scriptingLanguages:text/javascript)，JavaScript 内容应始终使用 MIME 类型 `text/javascript` 提供。其他 MIME 类型对 JavaScript 无效，使用除 `text/javascript` 以外的任何 MIME 类型都可能导致脚本无法加载或运行。

你可能会发现某些 JavaScript 内容在 MIME 类型中错误地使用了 `charset` 参数，以指定脚本内容的字符集。对于 JavaScript 内容来说，`charset` 参数无效，在大多数情况下会导致脚本加载失败。

#### 4.6 图片类型

MIME 类型为 image 的文件包含图像数据。子类型指定数据所代表的具体图像文件格式。

以下是常用的图像类型，可在网页中安全使用：

- [image/apng](https://developer.mozilla.org/zh-CN/docs/Web/Media/Guides/Formats/Image_types#apng_animated_portable_network_graphics)：动画便携式网络图形（APNG）
- [image/avif](https://developer.mozilla.org/zh-CN/docs/Web/Media/Guides/Formats/Image_types#avif_图像)：AV1 图像文件格式（AVIF）
- [image/gif](https://developer.mozilla.org/zh-CN/docs/Web/Media/Guides/Formats/Image_types#gif_graphics_interchange_format)：图形交换格式（GIF）
- [image/jpeg](https://developer.mozilla.org/zh-CN/docs/Web/Media/Guides/Formats/Image_types#jpeg_joint_photographic_experts_group_image)：联合图像专家小组图片（JPEG）

- [image/png](https://developer.mozilla.org/zh-CN/docs/Web/Media/Guides/Formats/Image_types#png_portable_network_graphics)：便携式网络图形（PNG）
- [image/svg+xml](https://developer.mozilla.org/zh-CN/docs/Web/Media/Guides/Formats/Image_types#svg_scalable_vector_graphics)：可缩放矢量图形（SVG）

- [image/webp](https://developer.mozilla.org/zh-CN/docs/Web/Media/Guides/Formats/Image_types#webp_图像)：Web 图像格式（WEBP）

#### 4.7 音频与视频类型

与图像的情况一样，HTML 并不强制要求 web 浏览器支持 [``](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Reference/Elements/audio) 和 [``](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Reference/Elements/video) 元素的任何特定文件和编解码器类型，因此在选择媒体使用的文件类型和编解码器时，必须考虑目标受众以及他们可能使用的浏览器（和这些浏览器的版本）范围。

我们的[媒体容器格式指南](https://developer.mozilla.org/zh-CN/docs/Web/Media/Guides/Formats/Containers)提供了 web 浏览器通常支持的文件类型列表，包括其特殊用途、缺点、兼容性信息以及其他详细信息。

[音频编解码器](https://developer.mozilla.org/en-US/docs/Web/Media/Guides/Formats/Audio_codecs)和[视频编解码器](https://developer.mozilla.org/zh-CN/docs/Web/Media/Guides/Formats/Video_codecs)指南列出了 web 浏览器通常支持的各种编解码器，并提供了兼容性细节和技术信息，如它们支持多少音频通道、使用哪种压缩方式以及它们的比特率等。在此基础上，[WebRTC 使用的编解码器](https://developer.mozilla.org/en-US/docs/Web/Media/Guides/Formats/WebRTC_codecs)指南专门介绍了主要 web 浏览器支持的编解码器，因此你可以选择最适合你所希望支持的浏览器范围的编解码器。

音频和视频文件的 MIME 类型，通常指的是其容器格式（或者说文件类型）。添加可选的 [codec 参数](https://developer.mozilla.org/en-US/docs/Web/Media/Guides/Formats/codecs_parameter)到 MIME 类型中，能进一步指出要使用的编解码器和编码媒体时曾用到的选项，如编解码器配置文件、级别或其他此类信息。

#### 4.8 multipart/form-data

可用于 [HTML 表单](https://developer.mozilla.org/zh-CN/docs/Learn_web_development/Extensions/Forms)从浏览器发送信息给服务器。

作为多部分文档格式，它由边界线（一个由双横滑线 `--` 开始的字符串）划分出的不同部分组成。每一部分有自己的实体，以及自己的 HTTP 请求头，[Content-Disposition](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Headers/Content-Disposition) 和 [Content-Type](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Reference/Headers/Content-Type) 用于文件上传字段。

```http
Content-Type: multipart/form-data; boundary=aBoundaryString
(other headers associated with the multipart document as a whole)

--aBoundaryString
Content-Disposition: form-data; name="myFile"; filename="img.jpg"
Content-Type: image/jpeg

(data)
--aBoundaryString
Content-Disposition: form-data; name="myField"

(data)
--aBoundaryString
(more subparts)
--aBoundaryString--
```

如下所示`<form>`：

```html
<form
  action="http://localhost:8000/"
  method="post"
  enctype="multipart/form-data">
  <label>名字：<input name="myTextField" value="Test" /></label>
  <label><input type="checkbox" name="myCheckBox" /> 勾选</label>
  <label>
    上传文件：<input type="file" name="myFile" value="test.txt" />
  </label>
  <button>发送文件</button>
</form>
```

会发送这样的请求：

```http
POST / HTTP/1.1
Host: localhost:8000
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.9; rv:50.0) Gecko/20100101 Firefox/50.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=---------------------------8721656041911415653955004498
Content-Length: 465

-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myTextField"

Test
-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myCheckBox"

on
-----------------------------8721656041911415653955004498
Content-Disposition: form-data; name="myFile"; filename="test.txt"
Content-Type: text/plain

Simple file.
-----------------------------8721656041911415653955004498--
```



### 五、设置正确的 MIME 类型的重要性

---

很多 web 服务器使用默认的 application/octet-stream 来发送未知类型。出于一些安全原因，对于这些资源浏览器不允许设置一些自定义默认操作，强制用户存储到本地以使用。

常见的导致服务器配置错误的文件类型如下所示：

- RAR 压缩文件。在这种情况，理想状态是，设置真实的编码文件类型；但这通常不可能，因为 .RAR 文件可能包含多种不同类型的资源。这种情况，将所发送文件的 MIME 类型配置为 `application/x-rar-compressed`。
- 音频或视频文件。只有正确设置了 MIME 类型的文件才能被 [``](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Reference/Elements/video) 或[``](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Reference/Elements/audio) 元素识别和播放。请确保指定了正确的[音频和视频的媒体类型](https://developer.mozilla.org/zh-CN/docs/Web/Media/Guides/Formats)。
- 专有文件类型。避免使用 `application/octet-stream`，对于这种一般的 MIME 类型浏览器不允许定义默认行为（比如“在 Word 中打开”）。像 `application/vnd.mspowerpoint` 这样的类型可以让用户选择自动在幻灯片软件中打开这样的文件。



### 六、MIME 嗅探

---

在缺失 MIME 类型或客户端认为文件设置了错误的 MIME 类型时，浏览器可能会通过查看资源来进行 MIME 嗅探。

每一个浏览器在不同的情况下会执行不同的操作。（例如，safari 会在发送的 MIME 类型不合适时查看文件的扩展名。）由于某些 MIME 类型可能代表可执行内容，会存在一些安全问题。服务器可以通过发送 X-Content-Type-Options 标头来阻止 MIME 嗅探。



### 七、其他传送文件类型的方法

---

MIME 类型不是传达文档类型信息的唯一方式：

- 有时会使用名称后缀，特别是在 Microsoft Windows 系统上。并非所有的操作系统都认为这些后缀是有意义的（特别是 Linux 和 Max OS），并且像外部 MIME 类型一样，不能保证它们是正确的。
- 魔数（magic number）。不同类型的文件的语法通过查看结构来允许文件类型推断。例如，每个 GIF 文件以 47 49 46 38 39 十六进制值（GIF89）开头，每个 PNG 文件以 89 50 4E 47（.PNG）开头。并非所有类型的文件都有魔数，所以这也不是 100% 可靠的方式。