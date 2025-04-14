### 一、DOM操作

---

#### 1.1 DOM核心概念

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>DOM操作</title>
    <style>
        #logo {
            width: 100px;
            height: 100px;
            background-color: blue;
        }
    </style>
</head>

<body>
    <button id="submit">按钮</button>

    <ul>
        <li class="list-item">苹果</li>
        <li class="list-item">香蕉</li>
        <li class="list-item">橙子</li>
        <li class="list-item">删除我</li>
    </ul>

    <form name="login">
        <input type="text" id="username">用户名</input>
        <input type="password" id="password">密码</input>
    </form>

    <img id="logo" alt="Logo"/>
    <div id="reference"><p>reference</p></div>
    <script src="./dom.js"></script>
</body>

</html>
```

```javascript
// 选择元素
// 单元素
const btn = document.getElementById('submit');
console.log("document.getElementById()获取单元素：", btn);
// 节点集合
const items = document.querySelectorAll('.list-item');
console.log("document.querySelectorAll()获取节点集合：");
items.forEach(item => {
    console.log(item);
});
// 表单访问
const form = document.forms['login'];

// 元素属性操作
const logo = document.querySelector('#logo');
logo.setAttribute('src', 'new-logo.jpg');       // 设置属性
const altText = logo.getAttribute('alt');       // 获取属性
logo.classList.add('active');                   // 添加类名
console.log("元素属性：", logo);

// 创建与插入节点
const newDiv = document.createElement('div');
newDiv.textContent = '动态内容';
document.body.appendChild(newDiv);              // 末尾插入
const reference = document.querySelector('#reference');
document.body.insertBefore(newDiv, reference);  // 指定位置插入
```



### 二、事件处理机制

---

#### 2.1 事件监听标准方式

```html
<button id="btn">Click event listener</button>
```

```javascript
const button = document.querySelector('#btn');
button.addEventListener('click', handleClick);

function handleClick(event) {
    console.log(event.target);      // 触发元素
    event.preventDefault();         // 阻止默认行为
    event.stopPropagation();        // 阻止冒泡
}
```

>**`event.preventDefault()` 的作用**：
>
>当用户触发某些特定事件时（如点击链接、提交表单、按下键盘键等），浏览器会执行与该事件关联的默认操作。例如，点击 `<a>` 链接标签时，浏览器会跳转到指定的 URL；提交 `<form>` 表单时，浏览器会刷新页面并将数据发送到服务器。该方法是阻止这些默认行为的发生。
>
>**使用场景**：
>
>- 阻止链接跳转：当点击 `<a>` 标签时，如果需要自定义逻辑而不希望页面跳转，可以调用 `preventDefault()`。
>- 阻止表单提交：在表单验证中，如果检测到输入无效，可以通过 `preventDefault()` 阻止表单提交。
>- 阻止滚动行为：对于某些按键事件（如空格键或方向键），可以阻止页面滚动。

>**`event.stopPropagation()`作用**：
>在 DOM 事件中，事件会按照一定的顺序传播，通常分为两个阶段：
>
>- 捕获阶段：事件从文档根节点向下传播到目标元素。
>- 冒泡阶段：事件从目标元素向上传播到文档根节点。
>
>`event.stopPropagation()` 的作用是阻止事件继续传播到其他祖先或后代元素，从而避免触发其他绑定在这些元素上的事件监听器。
>
>**使用场景**：
>
>- 阻止点击子元素时触发父元素的点击事件。
>- 在复杂的事件委托场景中，避免不必要的事件传播。
>- 当某个事件已经被处理完毕后，不需要再触发其他层级的事件监听器。

#### 2.2 事件委托

```html
<ul class="list">
    <button class="add-btn">Add item</button>
</ul>
```

```javascript
// 缓存 DOM 引用
const list = document.querySelector('.list');
// 事件委托：删除列表项
list.addEventListener('click', e => {
    if (e.target.matches('.delete-btn')) {
        deleteItem(e);
    }
});
// 事件委托：新增列表项
list.addEventListener('click', e => {
    if (e.target.matches('.add-btn')) {
        addItem();
    }
});

function deleteItem(event) {
    // 从当前元素开始向上查找，直到找到一个匹配指定选择器的祖先元素。
    const item = event.target.closest('li');
    if (item) {
        console.log('Deleting item :', item);
        // 移除列表项
        item.remove();
    }
}

let id = 1;
// 新增列表项
function addItem() {
    const list = document.querySelector('.list');
    const li = document.createElement('li');
    li.dataset.id = id++;
    li.textContent = `Item ${id - 1}`;

    const deleteBtn = document.createElement('button');
    deleteBtn.className = 'delete-btn';
    deleteBtn.textContent = 'Delete';

    li.appendChild(deleteBtn);
    console.log('Adding item:', li);
    list.appendChild(li);
}
```

>`e.target.matches(selector)` 会检查事件目标元素（`e.target`）是否与给定的 CSS 选择器（`selector`）匹配。如果匹配，则返回 `true`，否则返回 `false`。

#### 2.3 常见事件类型——输入事件

```html
<input id="input" type="text" />
```

```javascript
const input = document.querySelector('#input');
// 使用防抖技术优化频繁触发的 input 事件
const debounce = (func, delay) => {
    let timer;
    return function(...args) {
        clearTimeout(timer);
        timer = setTimeout(() => func.apply(this, args), delay);
    };
};

// 封装事件处理逻辑
const handleInput = (e) => {
    if (!e || !e.target) {
        console.error('Invalid event object');
        return;
    }
    const value = e.target.value;
    console.log('User input:', value);
};

// 绑定事件监听器，并应用防抖
input.addEventListener('input', debounce(handleInput, 300));
```



### 三、实战案例：动态图片画廊

---

```html
<!DOCTYPE html>
<html>
<head>
    <title>图片画廊</title>
    <style>
        .gallery {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 1rem;
            padding: 1rem;
        }
        .thumbnail {
            cursor: pointer;
            transition: transform 0.3s;
        }
        .thumbnail:hover {
            transform: scale(1.05);
        }
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.8);
            justify-content: center;
            align-items: center;
        }
        .modal-img {
            max-width: 90%;
            max-height: 90%;
        }
    </style>
</head>
<body>
    <div class="gallery" id="gallery"></div>
    <div class="modal" id="modal">
        <img class="modal-img" id="modalImg">
    </div>

    <script>
        // 图片数据
        const images = [
            { url: 'image1.jpg', alt: '风景1' },
            { url: 'image2.jpg', alt: '风景2' },
            { url: 'image3.jpg', alt: '风景3' }
        ];

        // 初始化画廊
        function initGallery() {
            const gallery = document.getElementById('gallery');
            gallery.innerHTML = images.map(img => `
                <img src="${img.url}" 
                     alt="${img.alt}" 
                     class="thumbnail"
                     data-src="${img.url}">
            `).join('');
        }

        // 模态框控制
        const modal = document.getElementById('modal');
        const modalImg = document.getElementById('modalImg');

        document.getElementById('gallery').addEventListener('click', e => {
            if (e.target.classList.contains('thumbnail')) {
                modal.style.display = 'flex';
                modalImg.src = e.target.dataset.src;
            }
        });

        modal.addEventListener('click', e => {
            if (e.target === modal) {
                modal.style.display = 'none';
            }
        });

        // 键盘事件
        document.addEventListener('keydown', e => {
            if (e.key === 'Escape') {
                modal.style.display = 'none';
            }
        });

        // 初始化
        initGallery();
    </script>
</body>
</html>
```



### 四、性能优化技巧

---

#### 4.1 事件委托优化

```javascript
// 避免为每个元素单独绑定事件
document.querySelector('.list').addEventListener('click', e => {
    if(e.target.closest('.item')) {
        handleItemClick(e.target);
    }
});
```

#### 4.2 防抖与节流 

```javascript
// 使用Lodash库
// 防抖
window.addEventListener('resize', _.debounce(handleResize, 200));
// 节流
document.addEventListener('scroll', _.throttle(handleScroll, 100));
```

>1. **节流（Throttle）**
>
>   节流是指在一定时间内只允许函数执行一次。如果事件触发的频率高于设定的时间间隔，函数将不会每次都执行，而是在间隔时间结束后执行一次。
>
>   应用场景：
>
>   1. 滚动事件（`scroll`）
>   2. 窗口大小改变事件（`resize`）
>   3. 鼠标移动事件（`mousemove`）
>   4. 连续的键盘输入事件（`keydown`）
>
>   实现原理
>
>   1. **时间戳法**：记录上次执行的时间戳，如果当前时间与上次执行时间的差大于设定的间隔，则执行函数。
>   2. **定时器法**：设置一个定时器，在间隔时间内不执行函数，直到定时器结束。
>
>2. **防抖（Debounce）**
>
>   防抖是指在事件停止触发后，才执行函数。如果事件在设定的时间间隔内再次被触发，则重新计时。
>
>   应用场景：
>
>   - 搜索框的输入事件（`input`）
>   - 窗口大小改变事件（`resize`）
>   - 连续的点击事件
>
>   实现原理：
>
>   防抖通过设置一个定时器来实现。每当事件被触发时，清除之前的定时器并重新设置一个新的定时器。只有当事件停止触发一段时间后，定时器才会执行函数。

#### 4.3 区别与选择

- **节流**：在高频事件中，确保函数在一定时间内执行一次，适合需要持续响应但不需要立即响应的场景。
- **防抖**：确保函数在事件停止触发后才执行，适合需要在事件完全结束后才执行的场景。



### 五、扩展功能实现

---

#### 5.1 图片懒加载优化

```html
<img data-src="image.jpg" class="lazyload">

<script>
const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
        if(entry.isIntersecting) {
            const img = entry.target;
            img.src = img.dataset.src;
            observer.unobserve(img);
        }
    });
});

document.querySelectorAll('.lazyload').forEach(img => {
    observer.observe(img);
});
</script>
```



### 六、调试与错误处理

---

1. DOM断点：

   - Chrome开发者工具 > Elements > 右键元素 > Break on > Subtree modifications

2. 事件监听检查：

   ```javascript
   // 查看元素绑定的事件
   getEventListeners(document.getElementById('btn'));

3. 错误边界处理：

   ```javascript
   try {
       document.querySelector('.missing-element').addEventListener(...);
   } catch (error) {
       console.error('元素不存在:', error);
       // 降级处理或创建备用元素
   }



### 七、要点

---

1. 掌握DOM节点的增删改查操作
2. 理解事件传播机制（捕获/冒泡）
3. 熟练使用事件委托优化性能
4. 实践现代浏览器API（Intersection Observer等）
5. 培养组件化开发思维
6. 提升交互设计能力
