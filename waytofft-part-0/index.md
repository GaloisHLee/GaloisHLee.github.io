# WayToFFT Part 0


从拉格朗日插值法到 FFT. Part-0.

<!--more-->

# 系数表示法和点值表示法

## 系数表示法
多项式可定义为形如
$$
\sum_{i=0}^{n} a_{i}x^{i}
$$
的有限和式，记作
$$
f(x) = \sum_{i=0}^{n} a_{i}x^{i}
$$
其中 $a_i$ 称为 $i$ 次项的**系数**。  
这种表示方法称为**系数表示法**。

## 点值表示法

将 $x = t_i$ 代入，得到序列 $(t_i, y_i)$。当此序列满足 $0 \le i \le n$ 且 $t_i$ 互不相等时，可以唯一确定一个次数不超过 $n$ 的多项式，这种描述方法称为**点值表示法**。

> 在给定 $n$ 个点值 $(t_0, y_0), (t_1, y_1), \dots, (t_{n-1}, y_{n-1})$ 且 $t_i$ 互不相等时，唯一确定的多项式次数至多为 $n-1$。

证明：考虑 $n$ 阶 Vandermonde 方阵
$$
V = \begin{bmatrix}
1 & t_0 & t_0^2 & \cdots & t_0^{n-1} \\\\
1 & t_1 & t_1^2 & \cdots & t_1^{n-1} \\\\
1 & t_2 & t_2^2 & \cdots & t_2^{n-1} \\\\
\vdots & \vdots & \vdots & \ddots & \vdots \\\\
1 & t_{n-1} & t_{n-1}^2 & \cdots & t_{n-1}^{n-1}
\end{bmatrix},
\quad
\mathbf{a} = \begin{bmatrix}
a_0 \\\\ a_1 \\\\ a_2 \\\\ \vdots \\\\ a_{n-1}
\end{bmatrix}
$$
由于 $t_i$ 互不相同，$\det V \ne 0$，故方程有唯一解，从而唯一确定系数向量 $\mathbf{a}$。

- 系数表示 $\to$ 点值表示：**求值（evaluation)**
- 点值表示 $\to$ 系数表示：**插值（interpolation)**

# 拉格朗日插值

## 定义

对于某个次数不超过 $n$ 的多项式 $f$，已知点值表示 $(t_i, y_i)$，其拉格朗日插值公式为：
$$
\mathscr{L}(x) := \sum_{i=0}^{n} y_{i} \,\ell_{i}(x)
$$
其中 $\ell_i(x)$ 称为**拉格朗日基本多项式（插值基函数）**：
$$
\ell_{i}(x) := \prod_{\substack{m=0 \\ m \ne i}}^{n} \frac{x - t_{m}}{t_{i} - t_{m}}
$$

### 存在性

由点值表示知，目标多项式一定存在。对于 $\ell_i(x)$，在 $x = t_s$ 时：
$$
\ell_{i}(t_s) =
\begin{cases}
1, & s = i, \\
0, & s \ne i.
\end{cases}
$$
于是：
$$
\mathscr{L}(t_s) = \sum_{i=0}^{n} y_{i} \,\ell_{i}(t_s) = y_s
$$

### 唯一性

次数不超过 $n$ 的拉格朗日多项式 $\mathscr{L}(x)$ 至多只有一个。  
设 $P_1$ 与 $P_2$ 均为满足插值条件的多项式，作差：
$$
\Delta(x) = P_1(x) - P_2(x)
$$
则 $\Delta(x)$ 在 $n+1$ 个互异点 $t_0,\dots,t_n$ 上为零，故：
$$
\Delta(x) = k \prod_{i=0}^{n} (x - t_i)
$$
若 $k = 0$，则 $P_1 \equiv P_2$；若 $k \ne 0$，则 $\deg \Delta > n$，与已知矛盾。

对应到 Vandermonde 矩阵：
- $\operatorname{rank} V = n+1$ 时有唯一解
- $\operatorname{rank} V < n+1$ 时有无穷多解

### 向量空间观点

设 $\mathbb{K}_{n}[X]$ 表示系数域为 $\mathbb{K}$、次数不超过 $n$ 的多项式空间。  
由 $\{\ell_0, \ell_1, \dots, \ell_n\}$ 构成的集合为该空间的一组基，因为：
1. 它们线性无关  
2. 元素个数为 $n+1$（等于空间维数）

---

## 核心思想总结

利用点值的可加性：每次构造一个多项式在某个插值点处取所需值、在其他插值点处取 0，然后将这些多项式相加。

插值公式可写为：
$$
f(x) = \sum_{i=0}^{n} y_{i} \prod_{\substack{m=0 \\ m \ne i}}^{n} \frac{x - t_{m}}{t_{i} - t_{m}}
$$

为了得到 $f$ 的系数，需要 $O(n^2)$ 时间计算
$$
F(x) = \prod_{i=0}^{n} (x - t_i)
$$
然后可通过解线性方程组（Vandermonde 系数矩阵）得到多项式系数。

---

# 范德蒙矩阵与拉格朗日逆

## 范德蒙行列式

定义 $n \times n$ 的 Vandermonde 矩阵：
$$
V = \begin{bmatrix}
1 & a_1 & a_1^2 & \cdots & a_1^{n-1} \\\\
1 & a_2 & a_2^2 & \cdots & a_2^{n-1} \\\\
\vdots & \vdots & \vdots & \ddots & \vdots \\\\
1 & a_n & a_n^2 & \cdots & a_n^{n-1}
\end{bmatrix}
$$
其行列式为：
$$
\det V = \prod_{1 \le j < i \le n} (a_i - a_j)
$$
只要 $a_i$ 两两不同，该行列式非零，矩阵可逆。

---

## 克拉默法则

若 $A\mathbf{x} = \mathbf{b}$ 且 $\det A \ne 0$，则唯一解为：
$$
x_j = \frac{\det A_j}{\det A}, \quad j = 1, \dots, n
$$
其中 $A_j$ 为将 $A$ 的第 $j$ 列替换为列向量 $\mathbf{b}$ 后得到的矩阵。

---

## 多项式与对称多项式

设多项式
$$
f(x) = \sum_{i=0}^{n} a_i x^i
$$
若为首一多项式（最高次项系数 $a_n = 1$），根据代数基本定理：
$$
f(x) = \prod_{i=1}^{n} (x - r_i)
$$
韦达定理给出系数与根的关系：
$$
a_{n-k} = (-1)^k \,\sigma_k(r_1, r_2, \dots, r_n), \quad k=1,\dots,n
$$
其中 $\sigma_k$ 为第 $k$ 阶初等对称多项式。

---

## 拉格朗日逆矩阵形式

设 $V$ 为由插值点 $t_0,\dots,t_n$ 构成的 $(n+1) \times (n+1)$ Vandermonde 矩阵：
$$
V_{ij} = t_i^{\,j}, \quad 0 \le i,j \le n
$$
插值过程等价于解：
$$
V \mathbf{a} = \mathbf{y}
$$
其中 $\mathbf{a} = (a_0, \dots, a_n)^\mathrm{T}$，$\mathbf{y} = (y_0, \dots, y_n)^\mathrm{T}$。

由克拉默法则：
$$
a_j = \frac{\det V_j}{\det V}
$$
展开可得：
$$
f(x) = \sum_{j=0}^{n} \frac{\det V_j}{\det V} \, x^j
$$
进一步化简后，$V^{-1}$ 的第 $(j,i)$ 元素正是 $\ell_i$ 展开式中 $x^j$ 的系数，从而：
$$
\ell_i(x) = \frac{\prod_{\substack{m=0 \\ m \ne i}}^{n} (x - t_m)}{\prod_{\substack{m=0 \\ m \ne i}}^{n} (t_i - t_m)}
$$
矩阵形式：
$$
\mathbf{a} = V^{-1} \mathbf{y}
$$
即给出了 **拉格朗日逆矩阵** 的具体含义。

---

# Reference

- https://www.luogu.com.cn/blog/AlexWei/Polynomial---Lagrange-Interpolation-and-Fast-Fourier-transform
- https://zhuanlan.zhihu.com/p/397342283

