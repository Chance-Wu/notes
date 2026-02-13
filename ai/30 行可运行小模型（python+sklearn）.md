训练一个模型来判断：订单是否高风险。

- 特征：4个数字
- 模型：逻辑回归
- 数据：手写几条就够



### 一、准备

---

- python 3.8+

- 安装一个库：

  ```bash
  pip install scikit-learn
  ```



### 二、完整代码

---

```python
from sklearn.linear_model import LogisticRegression

# 1. 训练数据（特征）
# [注册天数, 24h下单数, 优惠金额, 是否IP异常]
X = [
    [1, 5, 80, 1],
    [2, 4, 60, 1],
    [300, 1, 10, 0],
    [200, 2, 20, 0],
    [5, 6, 90, 1],
    [180, 1, 5, 0]
]

# 2. 标签：是否高风险
y = [1, 1, 0, 0, 1, 0]

# 3. 训练模型
model = LogisticRegression()
model.fit(X, y)

# 4. 预测一个新订单
new_order = [[3, 5, 70, 1]]
score = model.predict_proba(new_order)[0][1]

print(f"高风险概率: {score:.2f}")
```

#### 2.1 没写规则

没有说：

> 注册 < 3 天 就危险

而是：

>“这是历史数据，你自己学”



















































































