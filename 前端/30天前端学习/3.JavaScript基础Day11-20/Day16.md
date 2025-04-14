### 一、异步编程（Promise/async-await）

---

#### 1.1 Promise基本概念

Promise是一个对象，用于表示一个异步操作的最终完成（或失败）及其结果值。它有三种状态：

- **Pending（进行中）**：初始状态，既没有完成也没有失败。
- **Fulfilled（已成功）**：操作成功完成，返回一个结果值。
- **Rejected（已失败）**：操作失败，返回一个错误原因。

一旦Promise的状态确定（成功或失败），它将保持这个状态不变，直到被消费（通过 `.then` 或 `.catch`）。

- 创建Promise：

  通过 `new Promise()` 创建一个 `Promise` 对象，并传入一个执行器函数（executor function）。器执行函数接收两个参数：`resolve` 和 `reject`，分别用于将 `Promise` 状态设置为成功或失败。

- Promise的消费

  使用.then()和.catch()方法来处理Promise的结果。

- 

```javascript
const promise = new Promise((resolve, reject) => {
  // 异步操作
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve("操作成功"); // 成功时调用 resolve
    } else {
      reject("操作失败"); // 失败时调用 reject
    }
  }, 1000);
});
```























































































































