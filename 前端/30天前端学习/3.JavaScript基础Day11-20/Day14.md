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











































































































