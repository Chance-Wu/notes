### 一、异步编程（Promise/async-await）

---

#### 1.1 Promise基本概念

Promise是一个对象，用于表示一个异步操作的最终完成（或失败）及其结果值。它有三种状态：

- Pending（进行中）：初始状态，既没有完成也没有失败。
- Fulfilled（已成功）：操作成功完成，返回一个结果值。
- Rejected（已失败）：操作失败，返回一个错误原因。

一旦Promise的状态确定（成功或失败），它将保持这个状态不变，直到被消费（通过 `.then` 或 `.catch`）。

- **创建Promise**

  通过 `new Promise()` 创建一个 `Promise` 对象，并传入一个执行器函数（executor function）。器执行函数接收两个参数：`resolve` 和 `reject`，分别用于将 `Promise` 状态设置为成功或失败。

- **Promise的消费**

  使用.then()和.catch()方法来处理Promise的结果。`.then()`用于处理成功的结果。可以链式调用，支持多步异步操作。`.catch()`用于处理失败的情况。可以捕获reject的错误，或者`.then()`中的异常。`.finally()`无论成功还是失败都会执行的回调，通常用于清理工作。

```javascript
function fetchData(url) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            Math.random() > 0.2
                ? resolve({ data: "模拟数据" })
                : reject("请求失败");
        }, 1000);
    }).then(data => {
        console.log(data);
    }).catch(error => {
        console.log(error);
    }).finally(() => {
        console.log("无论成功还是失败都会执行");
    });
}
```

#### 1.2 Promise静态方法

- **Promise.all()**

  接收一个Promise数组，当所有Promise都成功时，返回一个包含所有结果的数组；如果有一个失败，则返回第一个失败的错误。

- **Promise.race()**

  接收一个Promise数组，返回第一个完成（成功或失败）的Promise的结果。

- **Promise.allSettled()**

  接收一个Promise数组，返回一个包含所有Promise状态和结果的数组，无论成功还是失败。

- **Promise.resolve()**

  将值或Promise转换为一个已经成功（fulfilled）的Promise。

- **Promise.reject()**

  创建一个已经失败（rejected）的 Promise。

#### 1.3 Promise使用场景

- API请求：如 `fetch` 或 `axios`，返回的是Promise。
- 异步任务：如文件读取、数据库操作等。
- 并发操作：使用 `Promise.all()` 或 `Promise.race()` 处理多个异步任务。

#### 1.4 async/await

- **async 关键字**：用于定义一个异步函数，该函数返回一个 Promise。
- **await 关键字**：用于等待一个 Promise 完成，只有在 `async` 函数内部才能使用。

```javascript
async function myAsyncFunction() {
  try {
    const result = await somePromise();
    console.log(result); // 处理成功的结果
  } catch (error) {
    console.error(error); // 处理错误
  }
}
```



### 二、Fetch API

---

#### 2.1 基本用法

fetch() 是 Fetch() API 的核心方法，用于发送网络请求。

```javascript
function fetchData() {
    fetch("https://api.examples.com", { mode: 'no-cors' })
        .then(response => {
            if (!response.ok) {
                throw new Error(`HTTP error! status: ${response.status}`);
            }
            return response.json(); // 解析 JSON 响应体
        })
        .then(data => {
            console.log(data); // 处理响应数据
        })
        .catch(error => {
            console.error("Error:", error.message); // 处理错误
        });
}
```

#### 2.2 请求方法

`fetch()` 默认发送 GET 请求，但可以通过配置请求选项来发送其他类型的请求（如 POST、PUT、DELETE 等）。

**POST请求：**

```javascript
async function postData() {
    try {
        const response = await fetch("https://api.examples.com/", {
            mode: 'no-cors',
            method: "POST", // 请求方法
            headers: {
                "Content-Type": "application/json", // 设置请求头
            },
            body: JSON.stringify({ name: "John", age: 30 }), // 请求体
        });

        if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
        const data = await response.json();
        console.log(data);
    } catch (error) {
        console.error("Error:", error.message);
    }
}
```

#### 2.3 处理响应

fetch() 返回一个 Response 对象，可以通过以下方法处理响应：

- response.json()：解析JSON格式的响应体。
- response.text()：解析文本格式的响应体。
- response.blob()：解析二级制数据（如图片、文件）。
- response.ok：检查HTTP状态码是否在 200-299 范围内。

#### 2.4 并发请求

如果需要同时发送多个请求，可以使用 `Promise.all` 。

```javascript
async function fetchMultipleData() {
    try {
        const [response1, response2] = await Promise.all([
            fetch("https://api.example.com/data1"),
            fetch("https://api.example.com/data2"),
        ]);

        if (!response1.ok || !response2.ok) {
            throw new Error("One or more requests failed");
        }

        const data1 = await response1.json();
        const data2 = await response2.json();

        console.log("Data 1:", data1);
        console.log("Data 2:", data2);
    } catch (error) {
        console.error("Error:", error.message);
    }
}
```

#### 2.5 高级用法

1. 设置超时

   ```javascript
   function fetchWithTimeout(url, timeout = 5000) {
     return Promise.race([
       fetch(url),
       new Promise((_, reject) =>
         setTimeout(() => reject(new Error("Request timed out")), timeout)
       ),
     ]);
   }
   
   // 使用
   fetchWithTimeout("https://api.example.com/data", 3000)
     .then(response => response.json())
     .then(data => console.log(data))
     .catch(error => console.error("Error:", error.message));
   ```

2. 取消请求

   ```javascript
   async function fetchDataWithAbort() {
     const controller = new AbortController();
     const signal = controller.signal;
   
     // 10 秒后取消请求
     setTimeout(() => controller.abort(), 10000);
   
     try {
       const response = await fetch("https://api.example.com/data", { signal });
       const data = await response.json();
       console.log(data);
     } catch (error) {
       if (error.name === "AbortError") {
         console.log("Request aborted");
       } else {
         console.error("Error:", error.message);
       }
     }
   }
   
   fetchDataWithAbort();
   ```



### 三、实战：获取GitHub用户信息

---

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>GitHub用户信息查询</title>

    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 800px;
            margin: 2rem auto;
            padding: 0 1rem;
        }

        .search-box {
            display: flex;
            gap: 1rem;
            margin-bottom: 2rem;
        }

        input {
            flex: 1;
            padding: 0.8rem;
            border: 1px solid #ddd;
            border-radius: 4px;
            font-size: 1rem;
        }

        button {
            padding: 0.8rem 2rem;
            background: #2ea44f;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            transition: background 0.3s;
        }

        button:hover {
            background: #22863a;
        }

        .user-card {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 2rem;
            display: none;
            margin-top: 2rem;
        }

        .avatar {
            width: 120px;
            height: 120px;
            border-radius: 50%;
            margin-bottom: 1rem;
        }

        .stats {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 1rem;
            margin: 1rem 0;
        }

        .stat-item {
            text-align: center;
            padding: 1rem;
            background: #f6f8fa;
            border-radius: 4px;
        }

        .error {
            color: #cb2431;
            padding: 1rem;
            border: 1px solid #f3c3c7;
            background: #f8d7da;
            border-radius: 4px;
            margin-top: 1rem;
            display: none;
        }

        .loading {
            display: none;
            text-align: center;
            margin: 2rem 0;
            color: #586069;
        }

        .repo-cards {
            margin-top: 2rem;
            display: flex;
            flex-wrap: wrap;
            gap: 1.5rem;
            justify-content: center;
        }

        .repo-cards>div {
            border: 1px solid #e0e0e0;
            border-radius: 8px;
            padding: 1.5rem;
            background: rgba(255, 255, 255, 0.1);
            /* 半透明背景 */
            backdrop-filter: blur(10px);
            /* 毛玻璃效果 */
            /* 使用深色渐变 */
            box-shadow: 0 8px 12px rgba(0, 0, 0, 0.1);
            /* 更强的阴影效果 */
            width: calc(40% - 3rem);
            transition: transform 0.3s, box-shadow 0.3s, background 0.3s;
            white-space: normal;
            overflow: visible;
            word-wrap: break-word;
            word-break: break-all;
            color: #333;
            position: relative; /* 添加相对定位 */
        }

        .repo-cards>div:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 15px rgba(0, 0, 0, 0.15);
            background: rgba(255, 255, 255, 0.2); /* 悬停时的半透明背景 */
        }

        .repo-cards>div h2 {
            margin-top: 0;
            font-size: 1.2rem;
            color: #333;
            font-weight: bold;
        }

        .repo-cards>div p {
            margin: 0.5rem 0;
            color: #666;
            /* 文字颜色改为浅灰色 */
            font-size: 0.9rem;
        }
    </style>
</head>

<body>
    <div class="container">
        <div class="search-box">
            <input type="text" id="username" placeholder="输入GitHub用户名" autocomplete="off">
            <button onclick="searchUser()">搜索</button>
        </div>

        <div class="loading" id="loading">加载中...</div>
        <div class="error" id="error"></div>

        <div class="user-card" id="userCard">
            <div class="user-header">
                <img class="avatar" id="avatar" alt="用户头像">
                <h2 id="name"></h2>
                <p id="bio"></p>
                <a id="profileLink" target="_blank">查看完整资料</a>
            </div>

            <div class="stats">
                <div class="stat-item">
                    <h3>仓库</h3>
                    <p id="repos"></p>
                </div>
                <div class="stat-item">
                    <h3>关注者</h3>
                    <p id="followers"></p>
                </div>
                <div class="stat-item">
                    <h3>正在关注</h3>
                    <p id="following"></p>
                </div>
            </div>
        </div>

        <!-- 仓库卡片容器 -->
        <div class="repo-cards" id="repoCards"></div>
    </div>

    <script src="./github-userinfo.js"></script>
</body>

</html>
```

```javascript
// DOM元素引用
const usernameInput = document.getElementById('username');
const userCard = document.getElementById('userCard');
const repoCardsContainer = document.getElementById('repoCards');
const errorEl = document.getElementById('error');
const loadingEl = document.getElementById('loading');

// 输入框回车支持
usernameInput.addEventListener('keypress', (e) => {
    if (e.key === 'Enter') {
        searchUser();
        getRepos();
    }
});

async function searchUser() {
    const username = usernameInput.value.trim();
    if (!username) return;

    try {
        showLoading(true);
        clearError();

        const response = await fetch(`https://api.github.com/users/${username}`);

        if (!response.ok) {
            throw new Error(
                response.status === 404
                    ? '用户不存在'
                    : '获取数据失败'
            );
        }

        const userData = await response.json();
        displayUser(userData);

    } catch (error) {
        showError(error.message);
    } finally {
        showLoading(false);
    }
}

function displayUser(data) {
    // 更新基本信息
    document.getElementById('avatar').src = data.avatar_url;
    document.getElementById('name').textContent = data.name || data.login;
    document.getElementById('bio').textContent = data.bio || '暂无简介';
    document.getElementById('profileLink').href = data.html_url;

    // 更新统计信息
    document.getElementById('repos').textContent = data.public_repos;
    document.getElementById('followers').textContent = data.followers;
    document.getElementById('following').textContent = data.following;

    // 显示卡片
    userCard.style.display = 'block';
}

function showLoading(show) {
    loadingEl.style.display = show ? 'block' : 'none';
}

function showError(message) {
    errorEl.textContent = message;
    errorEl.style.display = 'block';
    userCard.style.display = 'none';
}

function clearError() {
    errorEl.style.display = 'none';
    errorEl.textContent = '';
}

async function getRepos() {
    const username = usernameInput.value.trim();
    if (!username) return;
    try {
        // 调用仓库API
        const response = await fetch(`https://api.github.com/users/${username}/repos`);
        if (!response.ok) {
            throw new Error(
                response.status === 404
                    ? '用户不存在'
                    : '获取数据失败'
            );
        }
        const repoData = await response.json();
        displayRepos(repoData);
    } catch (error) {
        showError(error.message);
    } finally {
        showLoading(false);
    }
}

function displayRepos(data) {
    console.log(data);
    let id = 1;
    data.forEach(item => {
        // 创建一个卡片的 div 容器
        const card = document.createElement("div");
        card.className = "repo-card" + "-" + id; // 添加样式类

        // 创建标题元素
        const a = document.createElement("a");
        a.textContent = item.name;
        a.href = item.url;

        // 将标题和描述添加到卡片中
        card.appendChild(a);

        // 将卡片添加到容器中
        repoCardsContainer.appendChild(card);
        id++;
    });

    // 显示卡片
    // repoCards.style.display = 'block';
}
```
