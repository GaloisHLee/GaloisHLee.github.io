# WayToFFT Part 0


从拉格朗日插值法到FFT. Part-0.

<!--more-->

# 系数表示法和点值表示法

## 系数表示法
多项式可简略定义为，形如$\Sigma_{i=0}^{n}a_{i}x^{i}$的有限和式为多项式，记作$f(x)= \Sigma_{i=0}^{n}a_{i}x^{i}$其中$a_{i}$称为$i$次项目的**系数**。

这种表示方法称为系数表示法

## 点值表示法

将$x = x_{i}$代入，得到序列$(x_{i},y_{i})$，此序列用于描述多项式时，称为点值表示法$0 \le i \le n+1$,可以唯一描述一个$n$次多项式。

考虑这两种表示法之间的联系。

>在给定$n$个点值$(x_{0},y_{0}),(x_{1},y_{1}),\dots,(x_{n-1},y_{n-1})$其中$x_{i}$互不相等时，所唯一确定的多项式最高次数为 $n-1$次。

证明:考虑$n$阶方阵

$$A =\begin{bmatrix}1&x_0&x_0^1&\cdots&x_0^{n-1} \\\\ 1&x_1&x_1^1&\cdots&x_1^{n-1} \\\\ 1&x_2&x_2^1&\cdots&x_2^{n-1} \\\\ \vdots&\vdots&\vdots&\ddots&\vdots \\\\ 1&x_{n-1}&x_{n-1}^2&\cdots&x_{n-1}^{n-1}\end{bmatrix}
,x=\begin{bmatrix} a_{0} \\\\ a_{1} \\\\ a_{2} \\\\ \vdots \\\\ a_{n-1} \end{bmatrix}$$

$x_{i}$互不相同，$\det A \ne 0$，故方程有唯一解，则可唯一确定一组系数$a_{i}$

- 系数表示 -> 点值表示 **求值（evaluation)**
- 点值表示 -> 系数表示 **插值(interpolation)**

# 拉格朗日插值

## 定义

对于某个$n$次多项式函数$f$，已知给定点值表示$(x_{i},y_{i})$

$$\mathscr{L} (x) := \sum_{i=0}^{n} y_{i} \mathscr{l}_{i} (x)$$

其中每个$\mathscr{l}$称为
**拉格朗日基本多项式（插值基函数）**:

$$\mathscr{l_{i}} (x) := \prod_{i=0,j \ne i}^{n} \frac{x-x_{j}}{x_{i}-x_{j}}= \frac{(x-x_{0})\cdots(x-x_{n})}{(x_{i}-x_{0})\cdots (x-x_{i-1})(x-x_{i+1})\cdots(x-x_{n}) }$$

### 存在性

由点值表示得知目标多项式一定存在，那么对于拉格朗日基本多项式, 对于$i=s,(x_{s},y_{s})$得到

$$\mathscr{l_{i}} (x_{s}) = 1$$

那么可以满足

$$y_{s}\mathscr{l_{i}} (x_{s}) = y_{s}$$

进而构造

$$\mathscr{L} (x) := \sum_{i=0}^{n} y_{i} \mathscr{l_{i}} (x)$$

### 唯一性

次数不超过$n$的拉格朗日多项式$\mathscr{L} (x)$至多只有一个。

证明：
对于在$n+1$个点上取值均为零的多项式，有
$$P_{i}(x)= k \prod_{i=0}^{n}(x-x_{i}) \tag{1}$$
对任意两个次数不超过$n$的拉格朗日多项式:$P_{1}$和$P_{2}$,作差得
$$\Delta(x)=P_{1}(x)-P_{2}(x) \tag{2}$$
那么由$(1),(2)$式知，
$i.$ 若$\Delta(x) =0$， 则$P_{1}(x) =P_{2}(x)$，即多项式系数$k$唯一
$ii.$ 若$\Delta(x) \ne 0$,  则$\min(\deg(P_{1},P_{2})) \gt n$

对应范德蒙矩阵

- $rank = n$时有唯一解

- $rank \lt n$ 有无穷多解

### 拉格朗日插值与向量空间

**线性空间**：
$\mathbb{K_{n}}[X]$, 经由拉格朗日插值法，可以找到一组基，由拉格朗日基本多项式$\mathscr{l_{0}},\mathscr{l_{1}},\dots,\mathscr{l_{n}}$组成,使得
$$P=\prod_{i = 0}^{n} \lambda_{i} \mathscr{l}_{i}=0$$
那么，对于多项式$P(x_{i})=\lambda_{i}$的拉格朗日插值多项式，与零多项式$P$。
可得:

$$\lambda_{0} = \lambda_{1} = \dots =\lambda_{n} = 0$$

则$\mathscr{l_{0}},\mathscr{l_{1}},\dots,\mathscr{l_{n}}$线性无关，同时包含$n+1$个多项式：
故可作为$\mathbb{K_{n}}[X]$的一组基底，且构造了一组齐次基。

## 核心思想

利用点值的可加性，每次仅考虑一个点值，其他值均为0，由此构造n个多项式$f_{i}(x)$，使得它们在$x_{i}$对应处值为$y_{i}$。则$f = \sum^{n-1}_{0} f_{i}(x)$。

- 构造其他点值为$0$, 必含有因子$\prod_{i \ne j}^{}(x-x_{j})$
- 构造$f_{i}(x) = y_{i}$,调整系数，数乘$\frac{y_{i}}{f_{i}(x)}$
- 各构造多项式累加

 得到插值最终表达式：
 $$ f(x) = \sum_{i=0}^{n-1}y_{i} \prod_{j \ne i}^{} \frac{x - x_{j}}{x_{i} - x_{j}}$$
  
为了得到$f$的各项系数，需要$O(n^{2})$求出
$$F(x)=\prod_{i = 0}^{n-1}(x-x_{i})$$。

已知$n-1$, 那么各个$x_{i}$均已知，行列式系数范德蒙矩阵可知。
若已知$y_{i}$则求系数问题可转化为线性方程组求解问题。

真对上述问题作变式：  $n-1$ 、序列$1,2,3,\dots,x_{n-1}$,$y_{0},\dots,y_{n-1}$,已知，可唯一确定多项式。

那么可以知晓，拉格朗日插值可求得范德蒙矩阵的逆矩阵。

# 范德蒙矩阵的拉格朗日逆

拉格朗日插值根据多项式的点值表示，以及多项式次数唯一确定多项式系数。
$$V ·A = Y$$
即根据Vandermonde矩阵，点值向量$V,Y$确定系数向量$A=(a_{0},\dots ,a_{n})$
$$A = V^{-1}Y$$
本节将会探究这样求值的具体形式化表达。

## 范德蒙行列式

范德蒙行列式，即范德蒙矩阵的行列式

$$M = \begin{vmatrix}1&1&\cdots&1 \\\\ a_1&a_2&\cdots&a_n \\\\ a_1^2&a_2^2&\cdots&a_n^2 \\\\ \vdots&\vdots&\ddots&\vdots \\\\ a_1^{n-1}&a_2^{n-1}&\cdots&a_n^{n-1}\end{vmatrix}$$
即:
$$M = M^{T}=\prod_{1\leq j<i\leq n}(a_i-a_j)$$
  
> **克拉默法则:**
> $n$元线性方程组的系数行列式$|A|\ne {0}$,则有唯一解，形式表达为:
>$$\begin{align}AX&= B \\\\ A &=(a_{0},a_{1},a_{2},\dots,a_{n}) \tag{0} \\\\  a_{i}&= ( a_{0i},a_{1i},\dots,a_{(n-1)i})^{T} \\\\ B &= ( b_{0}, b_{1}, \cdots ,b_{n})^{T}\end{align}$$

>可以确定唯一解的形式：
> $$X =\left( \frac{|\mathbf{B_1}|}{|\mathbf{A}|},\frac{|\mathbf{B_2}|}{|\mathbf{A}|},\cdots,\frac{|\mathbf{B_n}|}{|\mathbf{A}|}\right)^{T}$$
> 其中
> $$B_{i} = (a_{0},a_{1},a_{i-1},B,a_{i+1},a_{n})$$

## 范德蒙方阵及其转置的行列式

对于多项式:
$$f(x)=\sum_{i=0}^{n} a_{i}x^{i}\tag{1}$$

首先改写为首一多项式

$$f(x)= \sum_{i=0}^{n-1} a_{i}^{'}x^{i} + x^{n}, a_{i}^{'}= \frac{a_{i}}{a_{n}}\tag{2}$$

应用代数基本定理，则首一多项式$f(x)$又可写作：

$$f(x)= \prod_{i=0}^{n}(x-x_{i}) \tag{3}$$

韦达定理对称多项式表达：

$$\begin{cases} a_{0} &= (-1)^{n} \sigma_{n}(x_{1},x_{2},\dots,x_{n}) \\\\ \cdots \\\\ a_{k} &= (-1)^{n-k} \sigma_{k}(x_{1},x_{2},\dots,x_{n}) \\\\ \cdots \\\\ a_{n-1} &= (-1)^{1}\sigma_{1}(x_{1},x_{2},\dots,x_{n}) \end{cases}$$

可以归纳为:

$$a_{i}=(-1)^{n-i}\sigma_{n-i}(x_{1},x_{2},\dots,x_{n})\tag{4}$$

## 推导拉格朗日逆

### 推出拉格朗日插值表达

由克拉默法则导出线性方程组$(0)$的一般解：
$$a_{j}=  \frac{|B_{j}|}{|A|}$$

从而
$$f(x)= \sum_{j=0}^{n} \frac{|B_{j}|}{|A|}x^{j}$$
将$|B+j|$按照第$j+1$列展开：
$$B_{j}= \sum_{j=0}^{n}y_{i}A_{ij}$$

反代$f(x)$中，并作二次求和的交换：
$$f(x)=\sum_{j=0}^n\frac{\sum_{i=0}^{n}y_i\mathbf{A_{ij}}}{|\mathbf{A}|}x^j=\sum_{i=0}^{n} y_{i} \frac{\sum_{j=0}^{n}x^{j}\mathbf{A_{ij}}}{|\mathbf{A}|}$$

胜利的曙光即将到来：

$$\sum_{j=0}^{n} x^{j}\mathbf{A_{ij}} = \begin{vmatrix} 1 & x_{0}&x_{0}^{2}&\cdots&x_{0}^{n} \\\\ \vdots&\vdots&\vdots&\ddots&\vdots \\\\ 1&x_{i-1}&x_{i-1}^{2}&\cdots&x_{i-1}^{n} \\\\ 1&x&x^{2}&\cdots&x^{n} \\\\ 1&x_{i+1}&x_{i+1}^{2}&\cdots&x_{i+1}^{n} \\\\ \vdots&\vdots&\vdots&\ddots&\vdots \\\\ 1&x_{n}&x_{n}^{2}&\cdots&x_{n}^{n}\end{vmatrix}$$

推导并,调整下标$i,j$得到：

$$\mathscr{l_{i}}(x)=\frac{\sum_{i=0}^{n}x^i\mathbf{A_{ji}}}{|\mathbf{A}|}$$

从而推出拉格朗日表达：

$$\mathscr{L} (x):= \sum_{i=0}^{n} y_{i} \mathscr{l_{i}} (x)$$

### 推出拉格朗日逆

范德蒙方阵逆矩阵：

对于范德蒙方阵$V$,其对角线元素$a_{0},\dots,a_{n-1} \ne 0$时，有唯一逆矩阵$V^{-1}$。

可计算知：
$$\det V^{-1} =  \prod_{1 \le i \lt j \le n}^{} \frac{1}{a_{i}-a_{j}}$$
而对于工程上，无论是克拉默法则，还是更易计算机实践的高斯消元，在求解$V^{-1}$时，都过于复杂，难以应用。

利用韦达定理对称多项式表达展开拉格朗日插值公式：
$$\begin{aligned}
\mathscr{L}(x) &= \sum_{i=0}^{n} y_{i} \mathscr{l_{i}}(x) \\\\
\mathscr{l_{i}}(x) &= \prod_{i=0,j \ne i}^{n} \frac{x-x_{j}}{x_{i}-x_{j}}= \frac{(x-x_{0})\cdots(x-x_{n})}{(x_{i}-x_{0})\cdots (x-x_{i-1})(x-x_{i+1})\cdots(x-x_{n}) } \\\\
\mathscr{L}(x) &= \sum_{j=0}^{n}  y_{i} \frac{\sum_{i=0}^{n}  \sigma_{i}x^{n-i}}{\prod_{i=0}^{n}(a_{i}-a_{j})}
\end{aligned}$$



# Reference

- https://www.luogu.com.cn/blog/AlexWei/Polynomial---Lagrange-Interpolation-and-Fast-Fourier-transform
- https://zhuanlan.zhihu.com/p/397342283

