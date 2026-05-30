# AboutXL


<!--more-->

# 

## 第一部分：引言与背景综述

### 1. 多变量公钥密码学与代数分析的现状

在后量子密码学（Post-Quantum Cryptography, PQC）的宏大叙事中，多变量公钥密码学（Multivariate Public Key Cryptography, MPKC）占据着独特的地位。不同于基于格（Lattice）或纠错码（Code-based）的方案，MPKC的安全性直接构建在有限域上求解非线性多项式方程组的困难性之上，即著名的PoSSo（Polynomial System Solving）问题[^1]。

#### 1.1 PoSSo问题的计算复杂性

PoSSo问题，具体而言，是指给定一组定义在有限域 $\mathbb{F}_q$ 上的多项式 $f_1, \dots, f_m \in \mathbb{F}_q[x_1, \dots, x_n]$，寻找一个向量 $\mathbf{x} \in \mathbb{F}_q^n$ 使得所有 $f_i(\mathbf{x}) = 0$。

- **通用硬度（Generic Hardness）：** 该问题已被证明是NP-hard的[^1]。对于通用的随机生成系统，目前已知的最佳算法（如F4、F5、XL及其变体）在 $m \approx n$ 时均表现出指数级的时间复杂度。
- **极端情况的可解性：** 当方程数量远多于变量数量（超定，overdetermined）或远少于变量数量（欠定，underdetermined）时，问题往往可以在多项式时间内求解[^1]。例如，在极度超定的情况下，可以通过线性化（Linearization）技术，将每一个单项式视为独立的线性变量，从而将非线性问题转化为线性代数问题求解。

#### 1.2 代数分析的核心逻辑

代数密码分析（Algebraic Cryptanalysis）的核心目标是攻击那些看似随机但实则包含隐藏结构的密码系统。攻击者试图生成新的方程，这些方程在代数理想 $\mathcal{I}(\mathcal{F}) = \langle f_1, \dots, f_m \rangle$ 中，且能够降低求解所需的“操作度数”（operating degree）。

- **线性化攻击（Linearization Attacks）：** 这是最基础的方法。如果即使在扩展单项式后，线性无关的方程数量仍不足以解出所有变量，我们需要更高阶的算法。
- **Gröbner基与XL算法：** 为了系统地消元，Buchberger提出了Gröbner基算法。随后，Lazard将其与Macaulay矩阵的线性代数操作联系起来。Faugère进一步优化了这一过程，提出了F4和F5算法，利用稀疏线性代数和避免无用计算（Syzygies）来提升效率[^1]。与此同时，Courtois等人重新发明了类似的概念，命名为XL（Extended Linearization）算法，即“扩展线性化”[^1]。

### 2. 结构化系统的挑战：通用算法的局限

上述算法（XL, F4, F5）在处理**半正则（Semi-regular）系统**时表现出高度的可预测性。半正则系统是指除了平凡的代数关系（如 $f_i f_j - f_j f_i = 0$）之外，不存在其他低次代数关系的系统。对于这类系统，我们可以精确计算其Hilbert级数，从而准确预估算法的运行时间[^1]。

然而，现实中的密码系统设计绝非“通用”或“随机”。为了构建陷门（Trapdoor）以实现高效的签名或解密，设计者必须引入特定的代数结构。

- **隐蔽结构：** 例如UOV（Unbalanced Oil and Vinegar）、Rainbow签名方案、HFE（Hidden Field Equations）等，它们通过线性变换隐藏了内部的代数结构（如多层结构、特定的变量分组等）[^1]。
- **通用算法的低效性：** 当我们将针对随机系统设计的通用算法（如标准XL）应用于这些结构化系统时，往往会遭遇效率瓶颈。通用算法倾向于忽略变量之间的内在分组和差异，盲目地生成庞大的Macaulay矩阵，导致计算资源（时间和内存）的巨大浪费。例如，对于具有双线性结构的系统，标准XL会将所有变量混合处理，计算所有可能的单项式乘积，而忽略了许多乘积在特定度数下并不产生线性无关的新方程[^1]。

正是在这种背景下，Ning, Ran, 和 Samardjiska 提出了**多齐次XL（Multi-homogeneous XL, MXL）**算法。这是一项专门针对具有多重齐次结构（Multi-homogeneous structure）的多项式系统而设计的改进算法。它不仅填补了理论上的空白，更在实际攻击中展现了显著的性能提升。

------

## 第二部分：多齐次系统的理论基础

本部分将深入探讨多齐次系统的数学定义及其性质。理解这些概念是掌握MXL算法的前提。这是对经典代数几何中齐次理想理论的重要推广。

### 3. 多重分级环与多齐次多项式

在许多密码分析场景中，变量天然地分为若干个不相交的集合。例如，在MinRank问题的Kipnis-Shamir建模中，变量被分为“核变量”（Kernel variables）和“线性组合系数”（Linear combination coefficients）。

#### 3.1 核心定义

设 $n_1, \dots, n_c \in \mathbb{N}$ 为正整数。令 $x_i = \{x_{i1}, \dots, x_{in_i}\}$ 表示第 $i$ 组变量，其中 $i \in \{1, \dots, c\}$。

- **多重分级环（Multi-graded Ring）：** 我们考虑的多项式环是 $\mathbb{F}_q[x_1, \dots, x_c]$。
- **多齐次多项式：** 一个多项式 $f$ 被称为关于多重度数（multi-degree） $d = (d_1, \dots, d_c) \in \mathbb{Z}_{\ge 0}^c$ 是多齐次的，如果它的每一项在变量组 $x_i$ 上的总次数都恰好为 $d_i$。
  - 例如，若 $x_1 = \{u, v\}, x_2 = \{y, z\}$，则多项式 $f = u^2 y + v^2 z$ 是多齐次的，其多重度数为 $(2, 1)$。因为 $u^2$ 和 $v^2$ 的次数均为2（针对 $x_1$），$y$ 和 $z$ 的次数均为1（针对 $x_2$）。
  - 反之，$g = u y + v^2 z$ 不是多齐次的，因为第一项针对 $x_1$ 的次数是1，第二项是2[^1]。

#### 3.2 形状（Shape）与系统

一个多项式系统 $\mathcal{F} = (f_1, \dots, f_m)$ 的形状（Shape） $\mathcal{S}$ 定义为所有方程多重度数的集合：



$$\mathcal{S} = (deg(f_1), \dots, deg(f_m)) \in (\mathbb{Z}_{\ge 0}^c)^m$$



理解系统的形状至关重要，因为它决定了Macaulay矩阵的构建方式和潜在的代数关系（Syzygies）[^1]。

### 4. 边界正则性（Border-Regularity）：理论的核心

在单变量集合的经典理论中，“半正则性”（Semi-regularity）假设几乎所有的随机系统都具有某种标准的Hilbert级数行为。对于多齐次系统，作者提出了一个新的概念——**边界正则性（Border-Regularity）**。

#### 4.1 推广Hilbert级数

对于一个理想 $\mathcal{I}$，其Hilbert级数 $\mathcal{H}(t)$ 描述了理想在各个度数上的维数。在多齐次情形下，变量 $t$ 变为向量 $t = (t_1, \dots, t_c)$，级数变为多重幂级数。

作者提出，对于一个边界正则的系统，其Hilbert级数应由下式给出（在特定范围内）：

$$ \mathcal{H}(t) = \left[ \frac{\prod_{j=1}^m (1 - t^{deg(f_j)})}{\prod_{i=1}^c (1 - t_i)^{n_i}} \right]_+ $$

这里 $t^{d}$ 使用多重指数记号，即 $\prod t_i^{d_i}$。公式中的分母 $\prod (1 - t_i)^{n_i}$ 反映了多重分级环的生成元结构，分子则反映了方程引入的关系[^1]。

#### 4.2 “边界”的含义与Cramer规则Syzygies

为什么叫“边界”正则？因为这种“通用”的级数行为会在多重度数空间的某个边界处失效。

- **结构性Syzygies（Structural Syzygies）：** 在多齐次系统中，除了平凡的 $f_i f_j = f_j f_i$ 关系外，还存在由Cramer规则（克莱姆法则）引发的必然代数关系。

- Jacobian矩阵的子式： 考虑系统关于某一变量组 $x_i$ 的Jacobian矩阵。如果方程数量 $m > n_i$，该Jacobian矩阵的行之间必然线性相关（因为行数多于列数，列数为 $n_i$）。这种线性相关性可以通过Jacobian矩阵的 $(n_i + 1) \times n_i$ 子矩阵的行列式（Minors）显式构造出来。

  

  $$\text{Jac}_{x_i} \cdot \mathbf{x}_i^T = \text{diag}(deg_{x_i}(f_j)) \cdot \mathbf{f}$$

  

  利用这一关系，可以构造出度数为 $e_i + \sum (deg(f_j) - e_i)$ 的Syzygies。

- **边界集合 $\mathcal{B}(\mathcal{S})$：** 这些由Cramer规则导出的Syzygies所在的多重度数集合，构成了所谓的“边界”。
- **定义：** 一个系统被称为**边界正则**的，如果它的Hilbert级数在所有严格小于边界的多重度数（$\mathcal{B}_<(\mathcal{S})$）上，都与上述理论公式吻合[^1]。

  - **猜想1：** 对于给定的形状 $\mathcal{S}$，随机生成的多齐次系统以极高概率是边界正则的。这为算法的性能分析提供了理论基石。

### 5. 粘合（Gluing）与可容许序理想

这是该论文最具创新性的理论贡献之一。在传统XL中，我们只关注单一的总度数 $D$。而在多齐次XL中，单一的多重度数可能不足以捕捉所有的代数信息，或者会导致矩阵过大。

#### 5.1 为什么要“粘合”？

考虑一个简单的例子：假设我们在多重度数 $d=(1, 1, 2)$ 处构建Macaulay矩阵，发现秩亏（Rank defect），产生了一些新的低次方程（Degree falls），这些方程落在 $(1, 1, 1)$。同时，如果在 $(1, 2, 1)$ 处构建矩阵，也会产生落在 $(1, 1, 1)$ 的新方程。

- **问题：** 如果我们只分别处理 $(1, 1, 2)$ 或 $(1, 2, 1)$，我们可能会错过这两个来源的方程在 $(1, 1, 1)$ 处合并后产生的更深层的线性相关性。
- **解决方案：** 我们需要将这些不同多重度数对应的向量空间“粘合”在一起处理，形成一个更大的Macaulay矩阵，其列空间由一组多重度数 $\mathcal{D}$ 生成。

#### 5.2 序理想（Order Ideal）

集合 $\mathcal{D} \subset \mathbb{Z}_{\ge 0}^c$ 被称为一个**序理想**，如果对于任意 $d \in \mathcal{D}$，所有满足 $d' \le d$（按分量比较）的 $d'$ 也都在 $\mathcal{D}$ 中。

- 这意味着如果我们要考虑某个高次单项式，我们必须同时考虑所有整除它的低次单项式。这是构建Macaulay矩阵的自然要求。
- **粘合Macaulay矩阵 $\mathcal{M}_{\mathcal{D}}$：**
  - **列：** 对应所有多重度数在 $\mathcal{D}$ 中的单项式。
  - **行：** 对应所有形式为 $t \cdot f_i$ 的多项式，满足 $deg(t \cdot f_i) \in \mathcal{D}$。

#### 5.3 粘合性（Gluability）与定理1

- **定义（粘合性）：** 一个系统是“可粘合”的，如果来自不同多重度数 $D_1, \dots, D_k$ 的降次方程（Degree falls）在合并时表现良好——即它们要么线性无关，要么共同张成整个空间，不存在意外的线性相关。

- **猜想2：** 边界正则系统以高概率是可粘合的。

- 定理1（零度定理）： 对于一个可粘合的边界正则系统，如果序理想 $\mathcal{D}$ 位于边界之内 $ ( \mathcal{D} \subseteq \mathcal{B}\_{<} (\mathcal{S}))$   且没有子序理想是“可解的”（admissible），那么矩阵 $ \mathcal{M}_{\mathcal{D}}$ 的右零度（Right Nullity，即解空间维数或秩亏）可以由Hilbert级数的系数和精确给出：

  

  $$\text{Nullity}(\mathcal{M}_{\mathcal{D}}) = \sum_{d \in \mathcal{D}} [t^d]\,\mathcal{H}(t)$$

  

  这一定理直接指导了我们如何选择最优的 $\mathcal{D}$ 来最小化矩阵尺寸[^1]。

------

## 第三部分：多齐次XL算法详解

基于上述理论，作者提出了**Multi-homogeneous XL (MXL)** 算法。本部分将详细拆解算法步骤，并讨论其参数选择策略。

### 6. 算法流程 (Algorithm 2 & 3)

MXL算法的目标是求解多齐次多项式系统 $\mathcal{F}$。

#### 步骤 1：输入与预处理

- **输入：** 多项式系统 $\mathcal{F} = \{f_1, \dots, f_m\}$，及其形状 $\mathcal{S}$。
- **确定参数：** 寻找一个**可容许的序理想（Admissible Order Ideal）** $\mathcal{D}$。
  - 所谓“可容许”，是指理论预测在该序理想下，Macaulay矩阵的秩足够大，能够产生唯一的解（或解空间的基）。
  - 这通常意味着 Hilbert 级数在 $\mathcal{D}$ 上的系数和 $\le 1$（对于寻找唯一解的情况）。

#### 步骤 2：构建粘合Macaulay矩阵 $\mathcal{M}_{\mathcal{D}}$

这是算法中最耗内存的一步。

- **列索引：** 遍历 $\mathcal{D}$ 中所有多重度数 $d$，生成所有对应的单项式。例如，若 $\mathcal{D}$ 包含 $(1,1)$，则生成 $x_{1,i} x_{2,j}$ 等所有组合。
- **行生成：** 对于每个原始方程 $f_k$（设其度数为 $d_{f_k}$），寻找所有单项式 $u$，使得 $deg(u) + d_{f_k} \in \mathcal{D}$。计算 $u \cdot f_k$，将其系数填入矩阵。
- **稀疏性：** 由于每个 $f_k$ 只有有限项，生成的矩阵 $\mathcal{M}_{\mathcal{D}}$ 是极度稀疏的。每行非零元素的数量大致等于 $f_k$ 的项数，远小于列数。

#### 步骤 3：求解线性系统

目标是找到 $\mathcal{M}_{\mathcal{D}} \cdot \mathbf{y} = 0$ 的非平凡解向量 $\mathbf{y}$。

- **算法选择：** 由于矩阵巨大且稀疏，不能使用稠密高斯消元（$O(N^3)$）。必须使用稀疏线性代数算法，如 **Block Lanczos** 或 **Block Wiedemann** 算法。
- **左核与右核：**
  - 如果目的是直接找解，通常计算右核（Right Kernel）。
  - 但在某些变体（如算法3）中，为了消除特定的列（例如高次单项式），可能需要计算左核（Left Kernel）来寻找行之间的线性组合[^1]。
  - 实际上，对于求解MinRank等问题，通常需要找到 $k$ 个线性无关的核向量。

#### 步骤 4：解的提取

获得的核向量 $\mathbf{y}$ 实际上是对所有单项式的赋值。

- 如果系统有唯一解，$\mathbf{y}$ 的分量直接给出了单项式的值。由于 $\mathcal{D}$ 是序理想，它必然包含线性单项式（如 $x_{1,1}$），因此可以直接读取变量的值。
- 如果解不唯一，需要对核空间进行进一步的Groebner基转换或使用FGLM类算法（但这通常不在XL的主循环中）。

### 7. 寻找最优粘合策略（Heuristic Search）

MXL的效率关键在于选择最优的 $\mathcal{D}$。这是一项非平凡的任务，因为多重度数空间是一个格（Lattice），没有单一的“度数”概念。

#### 7.1 启发式搜索算法

作者采用了一种类似Dijkstra的最短路径搜索策略：

1. **初始化：** 从原点 $d=(0, \dots, 0)$ 开始。
2. **扩展：** 在每一步，考虑当前 $\mathcal{D}$ 的所有“邻居”多重度数（即 $d + e_i$）。
3. **评估：** 利用Hilbert级数公式 $\mathcal{H}_{\mathcal{S}}(t)$，计算若将该邻居加入 $\mathcal{D}$ 后，理论上的零度（Nullity）变化。
   - 目标是找到一个 $\mathcal{D}$，使得 $\sum_{d \in \mathcal{D}} [\mathcal{H}(t)]_d \le 1$（即系统可解），同时使得 $\|\text{Cols}(\mathcal{M}\_{\mathcal{D}})\|$ 最小。

4. **剪枝：** 利用系统的对称性（如MinRank中变量的对称性）来减少搜索空间。如果形状在某些变量置换下不变，则最优的 $\mathcal{D}$ 也应具有相应的对称性。

#### 7.2 实际案例分析

在表1[^1]中，作者展示了对于MinRank参数 $(3, 6, 5, 3)$，最优的单一多重度数是 $(1, 1, 3)$，矩阵大小为320。而通过粘合策略，最优集合是 $\{(1, 2, 1), (1, 1, 2)\}$，矩阵大小降为256。虽然在这个小例子中差异不大，但在大规模参数下，这种优化是指数级的。

------


------

### 8. Idea

Structure is Weakness.

任何偏离“随机”的代数特征，一旦被精确建模，都可能成为攻破系统的突破口。

### 参考文献

[^1]: Multi-homogeneous XL 与 Border-Regularity 原论文与开源实现。提出多齐次框架、边界正则性、粘合的 Macaulay 矩阵以及零度预测公式。参见 [Cryptology ePrint Archive 2025/2060](https://eprint.iacr.org/2025/2060).

