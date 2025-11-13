# Way-To-MPKC From UOV


<!--more-->



## UOV



### Notation

本文将使用以下变量：

- **\(m\) 个“油”变量 (Oil):** \(o_1, o_2, \dots, o_m\)
- **\(v\) 个“醋”变量 (Vinegar):** \(v_1, v_2, \dots, v_v\) (其中 \(v = n-m\))
- **\(m\) 个方程** \(F_1, \dots, F_m\)
- **\(m\) 个目标哈希值** \(z_1, \dots, z_m\)



我们要处理的系统是 \(F(x) = z\)，即 \(F_k(o, v) = z_k\)，对 \(k = 1, \dots, m\) 成立。

------



### \(F\) 的特殊结构



UOV的核心秘密在于 \(F\) 的特殊构造：**\(F\) 的方程中可以有任意的“醋-醋”项 (\(v_i v_j\)) 和“油-醋”项 (\(o_i v_j\))，但绝对没有“油-油”项 (\(o_i o_j\))。**

因此，我们的 \(m\) 个方程中，第 \(k\) 个方程 \(F_k\) 的**最一般形式**可以写成：
$$
F_k(o, v) = \underbrace{\sum_{i=1}^{v} \sum_{j=i}^{v} \alpha_{k,i,j} v_i v_j}_{\text{醋-醋 (二次)}} + \underbrace{\sum_{i=1}^{m} \sum_{j=1}^{v} \beta_{k,i,j} o_i v_j}_{\text{油-醋 (混合)}} + \underbrace{\sum_{i=1}^{m} \gamma_{k,i} o_i + \sum_{j=1}^{v} \delta_{k,j} v_j + C_k}_{\text{所有线性项和常数项}}
$$



- \(\alpha, \beta, \gamma, \delta, C\) 都是私钥中已知的常数系数。
- 注意看，这个方程对于 \(v\) 变量是**二次**的，但对于 \(o\) 变量是**线性**的（\(o_i\) 只以 1 次方出现）。



### 签名者的目标



签名者的目标是找到一组 \((o, v)\) 使得 \(F_k(o, v) = z_k\) 对所有的 \(k=1, \dots, m\) 成立。

完整的方程组（共 \(m\) 个）看起来是这样的：
$$
\begin{cases}    F_1(o, v) = z_1 \\\\    F_2(o, v) = z_2 \\\\    \quad \vdots \\\\    F_m(o, v) = z_m \end{cases}
$$



### 关键操作



签名者执行以下操作：

1. **随机选择“醋”：** 签名者为所有 \(v\) 个“醋”变量 \(v_1, \dots, v_v\) 随机挑选一组具体的值，我们称之为 \(\bar{v}_1, \dots, \bar{v}_v\)。
2. **代入方程：** 将这些已知的 \(\bar{v}\) 值代入到第 1 步的 \(F_k\) 方程中。

我们来看代入后，第 \(k\) 个方程 \(F_k(o, \bar{v}) = z_k\) 变成了什么：

$$
z_k = \sum_{i=1}^{v} \sum_{j=i}^{v} \alpha_{k,i,j} \bar{v}_i \bar{v}_j + \sum_{i=1}^{m} \sum_{j=1}^{v} \beta_{k,i,j} o_i \bar{v}_j + \sum_{i=1}^{m} \gamma_{k,i} o_i + \sum_{j=1}^{v} \delta_{k,j} \bar{v}_j + C_k
$$




### 推导与重组



现在，方程中唯一的未知数只剩下 \(m\) 个“油”变量 \(o_1, \dots, o_m\)。

我们把上式中所有包含 \(o_i\) 的项（未知项）移到左边，所有纯常数项（已知项）移到右边。

**1. 找出所有未知项（包含 \(o_i\) 的项）：**

$$
\sum_{i=1}^{m} \sum_{j=1}^{v} \beta_{k,i,j} o_i \bar{v}_j + \sum_{i=1}^{m} \gamma_{k,i} o_i
$$

我们可以按 \(o_i\) 合并同类项：

$$
= \sum_{i=1}^{m} o_i \cdot \left( \sum_{j=1}^{v} \beta_{k,i,j} \bar{v}_j + \gamma_{k,i} \right)
$$

**2. 找出所有已知项（纯常数项）：**

$$
\sum_{i=1}^{v} \sum_{j=i}^{v} \alpha_{k,i,j} \bar{v}_i \bar{v}_j + \sum_{j=1}^{v} \delta_{k,j} \bar{v}_j + C_k
$$

**3. 重组第 \(k\) 个方程：**

$$
\sum_{i=1}^{m} o_i \cdot \underbrace{\left( \sum_{j=1}^{v} \beta_{k,i,j} \bar{v}_j + \gamma_{k,i} \right)}_{\text{这是 \(o_i\) 的系数, 记为 } A_{k,i}} = z_k - \underbrace{\left( \sum_{i=1}^{v} \sum_{j=i}^{v} \alpha_{k,i,j} \bar{v}_i \bar{v}_j + \sum_{j=1}^{v} \delta_{k,j} \bar{v}_j + C_k \right)}_{\text{这是一个大常数, 记为 } B_k}
$$



### 线性系统显现



因为所有的 \(\bar{v}_j\) 都是已知值，所以 \(A_{k,i}\) 和 \(B_k\) 都是可以计算出来的具体常数。

我们再定义一个常数 \(R_k = z_k - B_k\)。

现在，第 \(k\) 个方程被**严格地**简化为：

$$
A_{k,1} o_1 + A_{k,2} o_2 + \dots + A_{k,m} o_m = R_k
$$

这，就是一个关于 \(o_1, \dots, o_m\) 的**一元线性方程**。



### 最终的 \(m \times m\) 线性系统



我们有 \(m\) 个这样的方程（\(k=1\) 到 \(m\)），把它们全部写在一起，就得到了一个 \(m\) 个方程、 \(m\) 个未知数（\(o_1, \dots, o_m\)）的标准线性方程组：

$$
\begin{cases}    A_{1,1}o_1 + A_{1,2}o_2 + \dots + A_{1,m}o_m = R_1 \\\\    A_{2,1}o_1 + A_{2,2}o_2 + \dots + A_{2,m}o_m = R_2 \\\\    \quad \vdots \\\\    A_{m,1}o_1 + A_{m,2}o_2 + \dots + A_{m,m}o_m = R_m \end{cases}
$$

**这就是推导的最终结果。** 签名者通过高斯消元法等标准方法解这个 \(m \times m\) 线性系统，得到“油”变量 \(\bar{o}_1, \dots, \bar{o}_m\) 的值。

(如果这个 \(m \times m\) 系统恰好无解，签名者只需回到第 3 步，重新随机选择一组 \(\bar{v}\) 值，再试一次即可。)

### 验证过程：纯粹的“代入求值”



我们来梳理一下验证者（Verifier）的工作：

**验证者拥有：**

1. **公钥 \(G\)：** 那组 \(m\) 个公开的、看起来非常复杂的二次方程。
2. **消息 \(\mu\)：** 原始消息。
3. **签名 \(y\)：** 签名者提供的 \(n\) 个值的向量， \(y = (y_1, y_2, \dots, y_n)\)。

**验证步骤：**

1. 计算目标哈希值：

   验证者计算 \(z = H(\mu)\)。 ( \(z\) 是一个 \(m\) 个值的向量)

2. 核心步骤：计算签名结果 \(z'\)

   验证者将收到的签名 \(y\)（这 \(n\) 个数值）代入到公钥 \(G\) 的 \(m\) 个方程中，计算出结果 \(z'\)。

   $$
   z' = G(y)
   $$

   展开来说，就是：

   $$
   \begin{cases}    z'_1 = G_1(y_1, y_2, \dots, y_n) \\\\    z'_2 = G_2(y_1, y_2, \dots, y_n) \\\\   \quad \vdots \\\\    z'_m = G_m(y_1, y_2, \dots, y_n) \end{cases}
   $$

   这里**没有任何“求解”**。\(G\) 是已知的方程，\(y\) 是已知的数值。这只是一个纯粹的**算术计算**。

3. 比对：

   验证者检查 \(z'\) 是否等于 \(z\)。

   - 如果 \(z' = z\)，签名有效。
   - 如果 \(z' \neq z\)，签名无效。



| **特征 (Feature)**      | **签名过程 (Signing Process)**                               | **验证过程 (Verification Process)**                          |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **目标 (Goal)**         | **求解** \(y\) 使得 \(G(y)=z\)                                   | **检查** \(G(y)\) 是否等于 \(z\)                                 |
| **所用密钥 (Key Used)** | **私钥** (\(F\) 和 \(A\))                                        | **公钥** (\(G\))                                               |
| **核心操作 (Core Op)**  | **解方程 (Solve Equations)**                                 | **函数求值 (Evaluate Function)**                             |
| **"线性系统"在何处**    | **有 (Exists)**：通过固定“醋”变量，将 \(F(x)=z\) 转化为 \(m \times m\) 线性系统来**求解**“油”变量。 | **无 (Does not exist)**：只是将 \(y\) 代入 \(G\) 中进行二次**计算**。 |
| **\(A\) 的作用**          | 用 \(A^{-1}\) 将解 \(x\) **变换**为签名 \(y\) (\(y = A^{-1}(x)\))。  | **不使用** \(A\) 或 \(A^{-1}\)。                                 |



### 总结

- **签名**的“魔法”在于利用秘密 \(F\) 和 \(A\) 将一个（对他人而言）困难的二次问题**转化**为一个（对自己而言）简单的**线性问题**来**求解**。
- **验证**的“朴实”在于它不进行任何变换，它只是**代入**签名 \(y\) 到公钥 \(G\) 中，看结果是否匹配。

这正是UOV方案（以及所有公钥密码）安全性的基础：正向计算 \(G(y)\) 很容易，但反向求解 \(y\)（即签名）在没有私钥“后门”的情况下极其困难。





## 矩阵视角

### 二次型的一般矩阵表示



任何一个 \(n\) 元二次方程 \(f(x) = z\) 都可以被写成矩阵形式：

$$f(x) = x^T P x + L^T x + C = z$$

- \(x\) 是 \(n \times 1\) 的变量向量。
- \(P\) 是一个 \(n \times n\) 的对称矩阵，代表所有二次项（如 \(x_i x_j\)）。
- \(L\) 是一个 \(n \times 1\) 的向量，代表所有线性项（如 \(x_i\)）。
- \(C\) 是一个常数。

------



### 签名者的视角：\(F\) 的矩阵结构



签名者使用**私有变量** \(x\)，它被**结构化**地分为“油” \(o\) ( \(m \times 1\) 向量) 和“醋” \(v\) ( \(v \times 1\) 向量)：

$$x = \begin{bmatrix} o \\\\ v \end{bmatrix}$$

私钥 \(F\) 的 \(m\) 个方程中的第 \(k\) 个 \(F_k(x) = z_k\) 具有 **“无油-油”** 的特殊结构。

这意味着 \(F_k\) 的二次型矩阵 \(P_k\)（一个 \(n \times n\) 矩阵）在 \(x = \begin{bmatrix} o \\\\ v \end{bmatrix}\) 这个基下，具有一个**特定的块(block)结构**：

$$
P_k = \begin{bmatrix} P_{oo} & P_{ov} \\\\ P_{vo} & P_{vv} \end{bmatrix}
$$


- \(P_{oo}\) 是 \(m \times m\) 矩阵，代表“油-油”项。
- \(P_{vv}\) 是 \(v \times v\) 矩阵，代表“醋-醋”项。
- \(P_{ov}\) 和 \(P_{vo}\) 是 \(m \times v\) 和 \(v \times m\) 矩阵，代表“油-醋”混合项 (且 \(P_{vo} = P_{ov}^T\))。

**UOV的核心秘密就是：\(P_{oo} = 0_{m \times m}\) ( \(m \times m\) 的零矩阵)。**

同时，线性部分的向量 \(L_k\) 也可以被分块： \(L_k = \begin{bmatrix} L_o \\\\ L_v \end{bmatrix}\)。

------



### 签名者的线性系统变换演示



签名者的第 \(k\) 个方程 \(F_k(x) = z_k\) 写作：

$$
\begin{bmatrix} o^T & v^T \end{bmatrix} \begin{bmatrix} 0 & P_{ov} \\\\ P_{vo} & P_{vv} \end{bmatrix} \begin{bmatrix} o \\\\ v \end{bmatrix} + \begin{bmatrix} L_o^T & L_v^T \end{bmatrix} \begin{bmatrix} o \\\\ v \end{bmatrix} + C_k = z_k
$$


第 1 步：固定“醋” \(v = \bar{v}\)

签名者随机选择一个常数向量 \(\bar{v}\) 并代入。\(o\) 成为唯一的变量。

$$
\begin{bmatrix} o^T & \bar{v}^T \end{bmatrix} \begin{bmatrix} 0 & P_{ov} \\\\ P_{vo} & P_{vv} \end{bmatrix} \begin{bmatrix} o \\\\ \bar{v} \end{bmatrix} + \begin{bmatrix} L_o^T & L_v^T \end{bmatrix} \begin{bmatrix} o \\\\ \bar{v} \end{bmatrix} + C_k = z_k
$$


**第 2 步：展开矩阵乘法 (二次项部分)**

$$
(o^T \cdot 0 \cdot o) + (o^T P_{ov} \bar{v}) + (\bar{v}^T P_{vo} o) + (\bar{v}^T P_{vv} \bar{v})
$$


**第 3 步：化简**

- \(o^T \cdot 0 \cdot o = 0\)

  这就是“魔法”发生的地方。 \(o\) 的所有二次项 \(o_i o_j\) 全部消失了，因为它们对应的系数矩阵是 \(0\)。

- \((o^T P_{ov} \bar{v}) + (\bar{v}^T P_{vo} o)\)

  因为 \(P_{vo} = P_{ov}^T\)，这两项是相等的（它们是标量，互为转置）。

  这部分等于 \(2 (\bar{v}^T P_{vo}) o\)。这是一个 \(o\) 的线性项。

- \((\bar{v}^T P_{vv} \bar{v})\)

  \(\bar{v}\) 是常数，\(P_{vv}\) 是常数矩阵。这是一个常数。

**第 4 步：展开线性项部分**
$$
L_o^T o + L_v^T \bar{v}
$$


- \(L_o^T o\) 是 \(o\) 的**线性项**。
- \(L_v^T \bar{v}\) 是一个**常数**。

第 5 步：重组方程

我们将所有 \(o\) 的项（都是线性的）留在左边，所有常数项移到右边：

$$
\underbrace{[ 2 (\bar{v}^T P_{vo}) + L_o^T ]}_{\text{一个 } 1 \times m \text{ 的常数行向量}} \cdot o = \underbrace{z_k - (\bar{v}^T P_{vv} \bar{v}) - (L_v^T \bar{v}) - C_k}_{\text{一个常数}}
$$


第 6 步：最终的线性系统

我们把 \(m\) 个这样的方程（\(k=1\) 到 \(m\)）堆叠起来。

- 左侧的 \(m\) 个 (\(1 \times m\)) 行向量组成了一个 \(m \times m\) 的**常数系数矩阵 \(M'\)**。
- 右侧的 \(m\) 个常数组成了一个 \(m \times 1\) 的**常数向量 \(R\)**。

签名者得到了一个易于求解的 \(m \times m\) 线性系统：

$$
M' \cdot o = R
$$


------



### 攻击者的（失败）变换演示



攻击者只知道公钥 \(G\) 和公开变量 \(y\)。

第 1 步：公钥 \(G\) 的矩阵是什么？

\(G(y) = F(A(y))\)。代入 \(x = Ay\)：

$$
G_k(y) = (Ay)^T P_k (Ay) + L_k^T (Ay) + C_k
$$

$$
G_k(y) = y^T (A^T P_k A) y + (L_k^T A) y + C_k
$$


公钥 \(G_k\) 的矩阵是 \(P'_k = A^T P_k A\)。

即使 \(P_k = \begin{bmatrix} 0 & P_{ov} \\\\ P_{vo} & P_{vv} \end{bmatrix}\) 是稀疏的，但 \(A\) 是一个稠密的 \(n \times n\) 矩阵。

\(P'_k = A^T P_k A\) 将是一个稠密的 \(n \times n\) 矩阵，它完全隐藏了 \(P_k\) 中那个 \(0\) 块的结构。

第 2 步：攻击者模仿“固定醋”

攻击者不知道 \(A\)，他只能猜测 \(y\) 的分块，比如 \(y = \begin{bmatrix} y_{oil} \\\\ y_{vin} \end{bmatrix}\)。

他将 \(P'_k\)（稠密矩阵）进行相应分块：

$$
P'_k = \begin{bmatrix} P'_{oo} & P'_{ov} \\\\ P'_{vo} & P'_{vv} \end{bmatrix}
$$


**关键失败点：** 由于 \(A\) 的“搅浑”作用，\(P'_k\) 是稠密的。**\(P'_{oo}\) 几乎不可能是零矩阵**。

第 3 步：代入 \(\bar{y}_{vin}\) 并展开

攻击者得到第 \(k\) 个方程（只看二次项）：

$$
(y_{oil}^T P'_{oo} y_{oil}) + (y_{oil}^T P'_{ov} \bar{y}_{vin}) + (\bar{y}_{vin}^T P'_{vo} y_{oil}) + (\bar{y}_{vin}^T P'_{vv} \bar{y}_{vin})
$$

$$ (y_{oil}^T P'{oo} y{oil}) + (y_{oil}^T P'{ov} \bar{y}{vin}) + (\bar{y}{vin}^T P'{vo} y_{oil}) + (\bar{y}{vin}^T P'{vv} \bar{y}_{vin}) $$


结论：

由于 \(P'_{oo} \neq 0 \) ，第一项 \(y_{oil}^T P'_{oo} y_{oil}\) 并不会消失。

这是一个关于 \(y_{oil}\) 的二次型。

因此，攻击者得到的系统仍然是一个**多元二次 (MQ) 方程组**，而不是线性方程组。他无法简化问题。




