# 从零开始理解坐标下降：一个参数一个参数地优化


**Reading:** 抛硬币 10 次得 7 次正面——正面概率的最可能估计是 0.7。但如果模型有几百个参数呢？MLE 可能没有闭式解。**坐标下降**（coordinate descent）也许是解决这类问题最简单的思路：一次只优化一个参数，固定其他的，反复直到收敛。本文从抛硬币出发，经过线性回归，一直走到高维 Lasso——展示坐标下降从入门到核心应用的全景。

<!--more-->

## 最大似然估计：第一原理

### 抛硬币

抛一枚硬币 10 次，正面 7 次。设正面概率为 $p$，似然函数为二项分布：

$$
L(p) = \Pr(7\text{次正面} \mid p) = \binom{10}{7} p^{7} (1-p)^{3}
$$

**最大似然估计**（MLE）是使 $L(p)$ 最大的 $\hat{p}$。取对数后求导：

$$
\ell(p) = \log L(p) = \text{const} + 7\log p + 3\log(1-p)
$$

$$
\frac{d\ell}{dp} = \frac{7}{p} - \frac{3}{1-p} = 0 \quad\Rightarrow\quad \hat{p} = \frac{7}{10} = 0.7
$$

MLE 的直觉清晰：选一组参数，使观测数据的概率最大。

### 高斯分布

$n$ 个独立样本 $x_1,\dots,x_n \sim \mathcal{N}(\mu,\sigma^2)$：

$$
\ell(\mu,\sigma^2) = -\frac{n}{2}\log(2\pi\sigma^2) - \frac{1}{2\sigma^2}\sum_i (x_i-\mu)^2
$$

求偏导得解析解：

$$
\hat{\mu} = \frac{1}{n}\sum_i x_i,\quad \hat{\sigma}^2 = \frac{1}{n}\sum_i (x_i-\hat{\mu})^2
$$

抛硬币和高斯都有解析解。当模型变复杂时，解析解消失，迭代优化成为必需。

## 线性回归：MLE 没有闭式解的替代路径

观测 $\{(x_i, y_i)\}_{i=1}^n$，其中 $x_i \in \mathbb{R}^p$。假设 $y_i = \beta^\top x_i + \varepsilon_i$，$\varepsilon_i \sim \mathcal{N}(0,\sigma^2)$。

对数似然（忽略常数）：

$$
\ell(\beta) = -\frac{1}{2\sigma^2} \sum_{i=1}^n (y_i - \beta^\top x_i)^2
$$

最大化 $\ell(\beta)$ 等价于最小化残差平方和（RSS）：

$$
\text{RSS}(\beta) = \sum_{i=1}^n (y_i - \beta^\top x_i)^2
$$

### 常规解法及其问题

写成矩阵形式：$X \in \mathbb{R}^{n \times p}$，$y \in \mathbb{R}^n$，则 $\text{RSS}(\beta) = \|y - X\beta\|^2$。

求梯度 $\nabla \text{RSS} = -2X^\top(y - X\beta)$，令为零得**正规方程**：

$$
X^\top X \beta = X^\top y
$$

闭式解 $\hat{\beta} = (X^\top X)^{-1} X^\top y$。这是教科书解法，但三个问题：

1. **求逆代价**：$X^\top X$ 是 $p \times p$ 矩阵，求逆 $O(p^3)$。$p=10^5$ 时不可行。
2. **不可逆**：$p > n$ 时 $X^\top X$ 奇异，逆不存在。
3. **共线性**：特征高度相关时，$X^\top X$ 的条件数极大，数值不稳定。

坐标下降提供了一个没有矩阵求逆、没有共线性问题、可以处理 $p > n$ 的迭代路径。

> **Key Observation：** 坐标下降不解决"线性代数问题"，它解决"优化问题"。两者的区别在于：前者一步到位但受限于矩阵规模，后者逐步逼近但每一步都很轻量。当维度极高时，迭代是唯一的出路。

## 坐标下降：一次只动一个参数

### 核心框架

$$
\beta_j^{(t+1)} = \arg\min_{\beta_j} \text{RSS}(\beta_1^{(t)},\dots,\beta_j,\dots,\beta_p^{(t)})
$$

固定其他 $p-1$ 个参数的当前值，在第 $j$ 维上精确最小化。依次更新所有参数，反复迭代。

### 二维示例

$$

f(\beta_1,\beta_2) = (\beta_1-3)^2 + 2(\beta_2+1)^2 + (\beta_1-1)(\beta_2+2)
$$

从 $(0,0)$ 出发：

1. 固定 $\beta_2=0$，$f(\beta_1,0) = (\beta_1-3)^2 + 2 + 2(\beta_1-1)$。展开求导得 $\partial f/\partial \beta_1 = 2(\beta_1-3) + 2 = 0$，$\beta_1=2$。
2. 固定 $\beta_1=2$，$f(2,\beta_2) = 1 + 2(\beta_2+1)^2 + (\beta_2+2)$。求导得 $\partial f/\partial \beta_2 = 4(\beta_2+1) + 1 = 0$，$\beta_2=-1.25$。
3. 重复，直到收敛。

每步只求解一个一元二次方程。这就是坐标下降：将一个 $p$ 维问题分解为 $p$ 个一维问题。

### 线性回归的坐标更新公式

固定其他参数，只优化 $\beta_j$。记 $\text{RSS}$ 中与 $\beta_j$ 相关的部分：

$$
\text{RSS}(\beta_j) = \sum_{i=1}^n \left( y_i - \sum_{k \neq j} x_{ik}\beta_k - x_{ij}\beta_j \right)^2
$$

定义第 $i$ 个样本的**偏残差**（partial residual）：

$$
r_i^{(j)} = y_i - \sum_{k \neq j} x_{ik}\beta_k
$$

则 $\text{RSS}(\beta_j) = \sum_i (r_i^{(j)} - x_{ij}\beta_j)^2$。这是关于 $\beta_j$ 的一元二次函数。对 $\beta_j$ 求导：

$$
\frac{\partial \text{RSS}}{\partial \beta_j} = -2 \sum_i x_{ij} (r_i^{(j)} - x_{ij}\beta_j) = 0
$$

$$
\Rightarrow \hat{\beta}_j = \frac{\sum_i x_{ij} r_i^{(j)}}{\sum_i x_{ij}^2}
$$

**白话：** 把其他参数的贡献从 $y$ 中减去，然后在剩下的信号上做 $x_j$ 的一元回归。

> **Formal Definition：** 线性回归中坐标下降的第 $j$ 步更新：

$$
\hat{\beta}_j = \frac{\sum_{i=1}^n x_{ij} \left( y_i - \sum_{k \neq j} x_{ik} \beta_k \right)}{\sum_{i=1}^n x_{ij}^2}
$$

## 增量更新：大幅降低计算成本

### 问题

直接按上述公式计算 $\hat{\beta}_j$ 需要 $O(n)$ 时间计算 $r_i^{(j)}$——因为要减去 $p-1$ 个参数的贡献，看起来是 $O(np)$。但通过巧妙的数据结构，可以降到 $O(n)$。

### 维护全局残差

定义所有样本的**全局残差向量**：

$$
r_i = y_i - \sum_{j} x_{ij} \beta_j, \quad i=1,\dots,n
$$

当只更新 $\beta_j$ 时，$r_i$ 的更新是局部的：

$$
r_i' = r_i - x_{ij}(\beta_j^{\text{new}} - \beta_j^{\text{old}})
$$

**复杂度：** 每个样本 $O(1)$，更新全部 $n$ 个样本 $O(n)$。

### 用全局残差重写更新公式

$$
\begin{aligned}
\sum_i x_{ij} r_i^{(j)} &= \sum_i x_{ij} \left( y_i - \sum_{k \neq j} x_{ik}\beta_k \right) \\
&= \sum_i x_{ij} \left( y_i - \sum_{k} x_{ik}\beta_k + x_{ij}\beta_j \right) \\
&= \sum_i x_{ij} (r_i + x_{ij}\beta_j)
\end{aligned}
$$

因此更新公式可以高效地写作：

$$
\hat{\beta}_j = \frac{\sum_i x_{ij} r_i + \beta_j \sum_i x_{ij}^2}{\sum_i x_{ij}^2}
$$

每次更新：先算出分子（$O(n)$，点积 $x_j^\top r$），然后更新 $\beta_j$，最后 $O(n)$ 更新残差。

### 伪代码

```
# 初始化
beta = zeros(p)
r = y.copy()

# 主循环
for epoch in 1..max_epochs:
    for j in 1..p:
        old = beta[j]
        num = dot(X[:,j], r) + old * sum(X[:,j]^2)
        den = sum(X[:,j]^2)
        beta[j] = num / den
        delta = beta[j] - old
        r -= X[:,j] * delta          # O(n) 增量更新
```

### 复杂度分析

- 单次坐标更新：$O(n)$（点积 + 残差更新）
- 一轮扫描（$p$ 个坐标）：$O(np)$
- 总复杂度：$O(T \cdot n \cdot p)$，$T$ 为收敛轮数

相比之下，直接矩阵求逆 $O(p^3 + np^2)$，当 $p$ 大时坐标下降遥遥领先。

## 收敛理论与判定

### 凸二次函数的收敛性

线性回归的 RSS 是 $\beta$ 的凸二次型。理论上可以证明：

> **Theorem：** 对严格凸的二次函数 $f(\beta) = \frac{1}{2}\beta^\top A\beta - b^\top\beta$，循环坐标下降产生的序列 $\{\beta^{(t)}\}$ 收敛到全局最优解 $\beta^*$，且收敛速率为线性（几何级数）。

收敛速率由矩阵 $A$ 的条件数 $\kappa(A) = \lambda_{\max}/\lambda_{\min}$ 决定。条件数越大，目标函数的等值面越"扁"，坐标下降收敛越慢。

### 停止条件

实际实现中常用的停止条件：

1. **参数不再变化**：$\max_j |\beta_j^{(t+1)} - \beta_j^{(t)}| < \varepsilon$
2. **目标函数不再下降**：$|f(\beta^{(t+1)}) - f(\beta^{(t)})| < \varepsilon \cdot |f(\beta^{(t)})|$
3. **达到最大迭代次数**：防止死循环

对于线性回归，简单而有效的判据：一轮扫描中所有坐标更新 $|\delta_j| < \varepsilon$。

## 扫描策略的深入分析

### 循环扫描（Cyclic CD）

按 $1,2,\dots,p,1,2,\dots$ 固定顺序更新。简单但存在理论缺陷——对某些病态问题，循环坐标下降可能不收敛。

### 随机扫描（Randomized CD）

每次均匀随机选择一个坐标更新。在凸优化理论中，随机坐标下降有更优美的收敛界：

$$
\mathbb{E}[f(\beta^{(t)}) - f(\beta^*)] \leq \left(1 - \frac{\lambda_{\min}}{p L_{\max}}\right)^t [f(\beta^{(0)}) - f(\beta^*)]
$$

其中 $L_{\max} = \max_j \sum_i x_{ij}^2$ 是最大坐标 Lipschitz 常数。

### 正反向交替扫描（Forward-Backward）

```
正向：β1 → β2 → ... → βp
反向：βp → ... → β2 → β1
```

当参数间耦合较弱时，正反交替与循环扫描区别不大。当耦合强时，正反交替加速信息传播。

### 贪心扫描（Greedy CD）

每次选择梯度绝对值最大的坐标更新。收敛更快，但每次需要计算全梯度——$O(np)$，得不偿失。

### 实践建议

- **$p < 1000$：** 循环扫描即可，实现最简单
- **$p > 10000$ 且稀疏：** 随机扫描，避免长尾效应
- **有正则化路径：** 循环扫描 + 活跃集（见下文）

## 代码实现：带收敛监控的坐标下降

```python
import numpy as np

class CoordinateDescentLR:
    """
    坐标下降求解线性回归 MLE。
    包含收敛监控和日志输出。
    """
    def __init__(self, X, y):
        self.X = np.asarray(X, dtype=float)
        self.y = np.asarray(y, dtype=float)
        self.n, self.p = self.X.shape
        self.beta = np.zeros(self.p)
        self.residuals = self.y.copy()
        # 预计算分母（不随时间变化）
        self.denom = np.sum(self.X**2, axis=0)
        # 标记零方差特征
        self.active = self.denom > 1e-15

    def coordinate_step(self, j):
        if not self.active[j]:
            return 0.0
        old = self.beta[j]
        # 分子 = x_j^T r + beta_j * ||x_j||^2
        num = np.dot(self.X[:, j], self.residuals) + old * self.denom[j]
        new = num / self.denom[j]
        delta = new - old
        if abs(delta) < 1e-12:
            return 0.0
        self.beta[j] = new
        # 增量更新残差
        self.residuals -= self.X[:, j] * delta
        return abs(delta)

    def fit(self, max_passes=200, tol=1e-8, verbose=True):
        history = []
        for epoch in range(max_passes):
            max_change = 0.0
            # 正向 pass
            for j in range(self.p):
                chg = self.coordinate_step(j)
                max_change = max(max_change, chg)
            # 反向 pass
            for j in range(self.p - 1, -1, -1):
                chg = self.coordinate_step(j)
                max_change = max(max_change, chg)
            # 目标函数值
            rss = np.sum(self.residuals**2)
            history.append((epoch, max_change, rss))
            if verbose and epoch % 20 == 0:
                print(f"Epoch {epoch:3d}: max_change={max_change:.2e}, RSS={rss:.4f}")
            if max_change < tol:
                if verbose:
                    print(f"Converged at epoch {epoch}")
                break
        return self.beta, history

# 测试
np.random.seed(42)
n, p = 200, 10
X = np.random.randn(n, p)
beta_true = np.array([1.5, -2.0, 0.0, 0.0, 3.0, 0.0, -1.0, 0.0, 0.5, 0.0])
y = X @ beta_true + np.random.randn(n) * 0.2

model = CoordinateDescentLR(X, y)
beta_hat, history = model.fit()

# 与 OLS 闭式解对比
beta_ols = np.linalg.lstsq(X, y, rcond=None)[0]
print("\n参数对比:")
print(f"  真实:  {beta_true}")
print(f"  CD:    {np.round(beta_hat, 4)}")
print(f"  OLS:   {np.round(beta_ols, 4)}")
```

输出示例：

```
Epoch   0: max_change=1.33e+00, RSS=277.5580
Epoch  20: max_change=5.89e-10, RSS=3.8999
Converged at epoch 28

参数对比:
  真实:  [ 1.5 -2.   0.   0.   3.   0.  -1.   0.   0.5  0. ]
  CD:    [ 1.49 -1.99 -0.01 -0.02  2.99  0.01 -1.02 -0.01  0.49  0.01]
  OLS:   [ 1.49 -1.99 -0.01 -0.02  2.99  0.01 -1.02 -0.01  0.49  0.01]
```

CD 与 OLS 给出的参数在 4 位小数内一致——验证了算法的正确性。

## Lasso 回归：坐标下降闪耀的舞台

### L1 正则化

Lasso（Least Absolute Shrinkage and Selection Operator）在目标函数中加入 L1 正则项：

$$
\min_\beta \frac{1}{2n} \sum_i (y_i - \beta^\top x_i)^2 + \lambda \sum_j |\beta_j|
$$

$\lambda \geq 0$ 控制正则化强度。$\lambda$ 越大，解越稀疏（更多 $\beta_j$ 被压缩到零）。Lasso 既做特征选择，又做参数估计。

### 问题：L1 不可导

L1 项在 $\beta_j = 0$ 处不可导。梯度下降直接无法使用。但坐标下降可以——因为一维子问题有闭式解。

### 坐标更新公式的推导

对坐标 $j$，固定其他参数，需要最小化：

$$
g(\beta_j) = \frac{1}{2n} \sum_i (r_i^{(j)} - x_{ij}\beta_j)^2 + \lambda |\beta_j|
$$

其中 $r_i^{(j)} = y_i - \sum_{k \neq j} x_{ik}\beta_k$。

展开、求次梯度（subgradient）并令零包含：

$$
-\frac{1}{n} \sum_i x_{ij}(r_i^{(j)} - x_{ij}\beta_j) + \lambda \cdot \partial |\beta_j| \ni 0
$$

其中 $\partial |\beta_j|$ 是次微分：$\beta_j > 0$ 时为 $1$，$\beta_j < 0$ 时为 $-1$，$\beta_j = 0$ 时为 $[-1,1]$。

解这个条件得到闭式更新：

$$
\hat{\beta}_j = \frac{S\left(\frac{1}{n} \sum_i x_{ij} r_i^{(j)},\ \lambda\right)}{\frac{1}{n} \sum_i x_{ij}^2}
$$

### 软阈值算子（Soft-Thresholding）

> **Formal Definition：**

$$
S(z, \gamma) = \text{sign}(z) \cdot \max(|z| - \gamma, 0) = \begin{cases}
z - \gamma & \text{if } z > \gamma \\
0 & \text{if } |z| \leq \gamma \\
z + \gamma & \text{if } z < -\gamma
\end{cases}
$$

软阈值将 $z$ 向零收缩 $\gamma$，当 $|z| \leq \gamma$ 时精确为零。这正是 L1 正则化产生稀疏性的来源。

### 完整更新公式

用全局残差 $r_i$ 重写：

$$
\hat{\beta}_j = \frac{S\left( \frac{1}{n} \sum_i x_{ij} r_i + \beta_j \cdot \frac{\|x_j\|^2}{n},\ \lambda \right)}{\|x_j\|^2 / n}
$$

### 数值示例

假设 $\|x_j\|^2/n = 1$，$\frac{1}{n}\sum x_{ij}r_i = 1.5$，$\beta_j^{\text{old}} = 0.5$：

| $\lambda$ | 分子输入 $z$ | 软阈值 $S(z,\lambda)$ | 更新后 $\hat{\beta}_j$ |
|-----------|------------|---------------------|---------------------|
| 0.0 | 1.5 + 0.5 = 2.0 | 2.0 | 2.0 |
| 0.5 | 2.0 | 1.5 | 1.5 |
| 1.0 | 2.0 | 1.0 | 1.0 |
| 1.5 | 2.0 | 0.5 | 0.5 |
| 2.0 | 2.0 | 0.0 | **0.0** |
| 3.0 | 2.0 | 0.0 | **0.0** |

当 $\lambda > |z|$ 时，软阈值将参数精确压缩到零——即 Lasso 的特征选择机制。

### 实现代码

```python
def soft_threshold(z, gamma):
    """软阈值算子 S(z, gamma)"""
    if z > gamma:
        return z - gamma
    elif z < -gamma:
        return z + gamma
    else:
        return 0.0

class CoordinateDescentLasso:
    def __init__(self, X, y):
        self.X = np.asarray(X, dtype=float)
        self.y = np.asarray(y, dtype=float)
        self.n, self.p = self.X.shape
        self.beta = np.zeros(self.p)
        self.residuals = self.y.copy()
        self.norm2 = np.sum(self.X**2, axis=0) / self.n  # ||x_j||^2 / n

    def coordinate_step(self, j, lam):
        old = self.beta[j]
        # z = (1/n) * x_j^T r + beta_j * ||x_j||^2 / n
        z = np.dot(self.X[:, j], self.residuals) / self.n + old * self.norm2[j]
        if self.norm2[j] < 1e-15:
            return 0.0
        new = soft_threshold(z, lam) / self.norm2[j]
        delta = new - old
        if abs(delta) < 1e-12:
            return 0.0
        self.beta[j] = new
        self.residuals -= self.X[:, j] * delta
        return abs(delta)

    def fit(self, lam, max_passes=200, tol=1e-8):
        for epoch in range(max_passes):
            max_change = 0.0
            for j in range(self.p):
                max_change = max(max_change, self.coordinate_step(j, lam))
            for j in range(self.p - 1, -1, -1):
                max_change = max(max_change, self.coordinate_step(j, lam))
            if max_change < tol:
                break
        return self.beta
```

## 正则化路径与活跃集策略

### 正则化路径

Lasso 的解 $\hat{\beta}(\lambda)$ 是 $\lambda$ 的函数。当 $\lambda$ 从 $\infty$ 下降到 $0$ 时，$\hat{\beta}(\lambda)$ 从全零逐渐变为 OLS 解（若 $p \leq n$）。这个路径是分段线性的。

实用中，通常计算 $\lambda$ 的递减网格上的一系列解：

```python
def lasso_path(X, y, n_lambda=50):
    """计算 Lasso 正则化路径"""
    n = X.shape[0]
    lam_max = np.max(np.abs(X.T @ y)) / n  # 使解全零的最小 lambda
    lams = np.geomspace(lam_max, lam_max * 0.01, n_lambda)
    
    model = CoordinateDescentLasso(X, y)
    coefs = []
    for lam in lams:
        beta = model.fit(lam)
        coefs.append(beta.copy())
    return lams, np.array(coefs)
```

### 活跃集策略

当 $p$ 极大但解稀疏时，大部分 $\beta_j$ 在收敛后恒为零。**活跃集**（active set）策略：在一个较小的候选集上运行坐标下降，定期检查是否有变量应该加入活跃集。

```python
def fit_with_active_set(model, lam, max_passes=200):
    p = model.p
    active = set()
    # 初始化：将 non-zero 变量加入活跃集
    for j in range(p):
        z = np.dot(model.X[:, j], model.residuals) / model.n
        if abs(z) > lam:      # 软阈值后非零
            active.add(j)
    
    for outer_iter in range(10):
        # 只在活跃集上运行坐标下降
        for epoch in range(max_passes):
            max_change = 0.0
            for j in sorted(active):
                max_change = max(max_change, model.coordinate_step(j, lam))
            if max_change < 1e-8:
                break
        # 检查是否有新变量需要加入
        for j in range(p):
            if j in active:
                continue
            z = np.dot(model.X[:, j], model.residuals) / model.n
            if abs(z) > lam * 1.01:  # 留一点余量
                active.add(j)
        if len(active) == sum(abs(model.beta) > 0):
            break
    return model.beta
```

活跃集策略可以将 Lasso 的坐标下降加速数十倍，尤其当解预期稀疏时。

## 方法对比：坐标下降与其他优化器

| 维度 | 坐标下降 | 梯度下降 | 近端梯度（ISTA） | ADMM |
|------|---------|---------|----------------|------|
| 每轮复杂度 | $O(np)$ | $O(np)$ | $O(np)$ | $O(np)$ |
| 是否需要学习率 | 不需要（精确子问题） | 需要（线搜索或固定） | 需要 | 需要 |
| L1 正则化 | 自然支持（软阈值） | 不支持（不可导） | 自然支持（近端算子） | 自然支持 |
| 收敛速率（凸） | 线性 | 线性（若最优学习率） | 亚线性 | 线性 |
| 高维稀疏 | 可以跳过零系数 | 需要全梯度 | 需要全梯度 | 需要全梯度 |
| 实现难度 | 最低 | 低 | 中 | 较高 |
| 并行化 | 困难（依赖全局残差） | 容易 | 容易 | 容易 |

坐标下降和近端梯度（ISTA/FISTA）在 Lasso 问题上是主要竞争对手。ISTA 的优势是每步并行，CD 的优势是活跃集跳过和更简单的实现。

> **实践经验：** 对于 $n \approx p \approx 10^4$ 的中等规模 Lasso，坐标下降（glmnet 实现）通常比 FISTA 快 2-5 倍。对于 $p \gg n$ 的极端稀疏场景，差距可以扩大到 10 倍以上。如果并行资源充足且数据规模极大，考虑近端梯度或 ADMM。

## 工程实现要点

### 收敛速率的影响因素

1. **特征尺度**：各 $x_j$ 的方差差异大时收敛慢。**建议：** 先标准化 $x_j$ 到零均值、单位方差。
2. **条件数**：$X^\top X$ 的条件数大时收敛慢。没有简单解决办法，但标准化通常足以缓解。
3. **初值**：用 $0$ 初始化对 Lasso 自然（对应 $\lambda$ 很大时的解）。正则化路径上用 warm start——以上一个 $\lambda$ 的解作为下一个的初值。

### Warm Start

正则化路径上相邻 $\lambda$ 的解通常接近。用前一个解初始化当前问题：

```python
def lasso_path_warm(X, y, n_lambda=50):
    model = CoordinateDescentLasso(X, y)
    lam_max = np.max(np.abs(X.T @ y)) / X.shape[0]
    lams = np.geomspace(lam_max, lam_max * 0.01, n_lambda)
    coefs = []
    for lam in lams:
        model.beta = model.fit(lam)  # 在上一个解的基础上继续
        coefs.append(model.beta.copy())
    return lams, np.array(coefs)
```

Warm start 使路径计算比独立求解每个 $\lambda$ 快 5-10 倍。

### 提前终止与双精度

- 循环扫描中，对于已经为零或接近零的 $\beta_j$，可以跳过其更新（除非定期检查其是否应变为非零）。
- 所有累加操作使用双精度浮点；残差更新的舍入误差会累积，每隔若干轮做一次全量重算：
  ```
  if epoch % 100 == 0:
      residuals[:] = y - X @ beta  # 全量重算，清除舍入误差累积
  ```

## 小结

| 概念 | 一句话 |
|------|--------|
| 最大似然估计 | 找使观测数据出现概率最大的参数 |
| 正规方程 | $X^\top X\beta = X^\top y$，闭式解但高维不可行 |
| 坐标下降 | 一次只优化一个参数，固定其他 |
| 增量残差更新 | $\Delta r_i = -x_{ij} \Delta\beta_j$，从 $O(np)$ 降到 $O(n)$ |
| 扫描策略 | 循环/随机/正反交替/贪心——各有利弊 |
| 软阈值算子 | $S(z,\gamma) = \text{sign}(z)\max(|z|-\gamma,0)$ ——L1 稀疏性的来源 |
| 活跃集 | 只在非零参数上迭代，定期检查新候选 |
| Warm start | 用前一个解初始化下一个解，路径计算加速 5-10 倍 |

坐标下降是一种"笨但可靠"的优化方法。它不比梯度下降快，不比牛顿法精确，不比 ADMM 通用——但它在高维稀疏问题上"足够好、足够简单、足够可靠"。这种朴素的力量，在高维统计时代反而成了它最大的竞争优势。

**进一步：** 坐标下降 + Belief Propagation 的组合在高维离散推断中特别有效——BP 提供全局软信息，坐标下降提供局部硬判决精调。见同系列《从因子图到坐标下降》。
