# 从零开始理解 Belief Propagation：贝叶斯网络上的消息传递


**Reading:** 假设你有一个报警系统：入室盗窃（burglary）或地震（earthquake）都可能触发报警器（alarm）。听到 alarm 后，邻居 John 和 Mary 可能会打电话给你。现在你接到 John 的电话——到底发生了什么事？是 burglary 还是 earthquake？这个问题本质上是要计算 $\Pr(\text{burglary} \mid \text{John calls})$。本文从概率论第一原理出发，用这个经典例子引出 **Belief Propagation（信念传播）**——一种在概率图模型上高效做推断的算法。

<!--more-->

## 两个基础概率定理

Belief Propagation 只用到了两条概率定理。整个算法都是这两条规则的反复应用。

### 加法规则（边缘化）

如果知道联合分布 $\Pr(X, Y)$，但只关心 $X$，就加掉 $Y$：

$$
\Pr(X = x) = \sum_{y} \Pr(X = x, Y = y)
$$

这叫**边缘化**。名字很形象——把概率表的"边"加起来，消去一个维度。

### 乘法规则（条件概率）

$$
\Pr(X, Y) = \Pr(X \mid Y) \cdot \Pr(Y)
$$

等价形式——贝叶斯定理：

$$
\Pr(Y \mid X) = \frac{\Pr(X \mid Y) \cdot \Pr(Y)}{\Pr(X)}
$$

> **Key Observation：** 加法规则负责"消去"隐变量；乘法规则负责"组合"独立证据。BP 的全部消息更新公式都来自这两条规则的交替使用。

## 一个经典例子：Pearl 的报警网络

### 网络结构

这是 Judea Pearl 在 1988 年提出的经典贝叶斯网络，也是概率图模型教学中的 Hello World。

五个二元变量，结构如下：

```
Burglary → Alarm ← Earthquake
              ↓
         JohnCalls  MaryCalls
```

箭头方向表示因果依赖：
- 入室盗窃（$B \in \{\text{true}, \text{false}\}$）以一定概率触发报警
- 地震（$E \in \{\text{true}, \text{false}\}$）也以一定概率触发报警
- 报警（$A \in \{\text{true}, \text{false}\}$）触发后，John 和 Mary 各自以一定概率打电话

### 条件概率表（CPT）

先验概率：

$$
\Pr(B = \text{true}) = 0.001, \qquad \Pr(E = \text{true}) = 0.002
$$

报警器的触发概率：

$$
\Pr(A = \text{true} \mid B, E) =
\begin{cases}
0.95 & \text{if } B=\text{T}, E=\text{T} \\
0.94 & \text{if } B=\text{T}, E=\text{F} \\
0.29 & \text{if } B=\text{F}, E=\text{T} \\
0.001 & \text{if } B=\text{F}, E=\text{F}
\end{cases}
$$

邻居打电话的概率：

$$
\Pr(J = \text{true} \mid A) = 0.90, \qquad
\Pr(M = \text{true} \mid A) = 0.70
$$

### 推断问题

John 打电话来了。问：入室盗窃的概率是多少？

$$
\Pr(B = \text{true} \mid J = \text{true})
$$

直接从定义出发，用加法规则展开：

$$
\Pr(B \mid J) \propto \sum_{E} \sum_{A} \sum_{M} \Pr(B, E, A, J, M)
$$

联合分布按网络结构因子化：

$$
\Pr(B, E, A, J, M) = \Pr(B) \cdot \Pr(E) \cdot \Pr(A \mid B, E) \cdot \Pr(J \mid A) \cdot \Pr(M \mid A)
$$

代入后对 $E, A, M$ 求和——一共 $2^3 = 8$ 项，手工可算。但当网络扩大到几百个节点时，暴力求和就是天文数字。

> **Formal Definition：** 给定变量集合 $\mathcal{X} = \{X_1, \dots, X_N\}$ 和观测 $\mathcal{E}$，推断目标是边缘后验 $\Pr(X_i \mid \mathcal{E})$。在贝叶斯网络中，联合分布因子化为 $\Pr(\mathcal{X}) = \prod_i \Pr(X_i \mid \text{parents}(X_i))$。直接边缘化的复杂度对变量数指数增长。

## 因子图：把联合分布画成二分图

BP 更自然地工作在一个无向的二分图结构上——**因子图（factor graph）**。

### 转化规则

将贝叶斯网络转化为因子图：每个条件概率 $\Pr(X_i \mid \text{parents}(X_i))$ 变成一个**因子节点**，每个变量变成一个**变量节点**。因子节点 $\psi_F$ 的值等于对应的条件概率值。

报警网络的因子图结构：

```
   B — ψ_A — A — ψ_J — J
   │         │
   E    ψ_M — M
```

（这里 $\psi_B$ 和 $\psi_E$ 是先验因子，各自只连一个变量，通常省略不画出。）

每个因子节点对应一个局部函数：

$$
\psi_B(b) = \Pr(B = b), \quad \psi_E(e) = \Pr(E = e)
$$
$$
\psi_A(a, b, e) = \Pr(A = a \mid B = b, E = e)
$$
$$
\psi_J(j, a) = \Pr(J = j \mid A = a), \quad \psi_M(m, a) = \Pr(M = m \mid A = a)
$$

全局联合分布就是所有因子之积：

$$
\Pr(B, E, A, J, M) = \psi_B(B) \cdot \psi_E(E) \cdot \psi_A(A, B, E) \cdot \psi_J(J, A) \cdot \psi_M(M, A)
$$

> **Formal Definition：** 因子图 $\mathcal{G} = (\mathcal{V}, \mathcal{F}, \mathcal{E})$ 是一个**二分图**。$\mathcal{V}$ 是变量节点集，$\mathcal{F}$ 是因子节点集，$\mathcal{E}$ 是边集。边 $(X_i, \psi_F) \in \mathcal{E}$ 表示 $\psi_F$ 是 $X_i$ 的函数。全局函数 $\Phi(\mathcal{V}) = \prod_{F \in \mathcal{F}} \psi_F(\text{neighbors}(F))$。

### 为什么用二分图

二分图的优势在于清晰地分离了"变量"和"函数"两种角色：

- 变量节点负责存储和转发消息（不做计算）
- 因子节点负责接收消息并计算新的消息（做真正的计算）

这种分离使消息更新规则变得极其对称。

## 树状图上的精确边缘化：手工推导

报警网络的因子图是一棵树——没有环。在树上，边缘概率可以通过两轮消息传递精确计算。

### Collect 阶段：从叶子到根

以计算 $A$ 的边缘分布为例，固定 $J = \text{true}$ 作为观测。写出边缘化公式并重新排列求和顺序是理解 BP 的关键一步。

从全联合分布开始：

$$
\Pr(A \mid J) \propto \sum_{B,E,M} \psi_B(B) \cdot \psi_E(E) \cdot \psi_A(A, B, E) \cdot \psi_J(J, A) \cdot \psi_M(M, A)
$$

注意到 $\psi_J(J, A)$ 和 $\psi_M(M, A)$ 都只依赖于 $A$（加上它们各自的观测变量），而 $\psi_B, \psi_E, \psi_A$ 一组涉及 $B, E$。利用乘法分配律重排求和：

$$
\begin{aligned}
\Pr(A \mid J) \propto \; &\psi_J(J, A) \\
&\times \underbrace{\left[\sum_M \psi_M(M, A)\right]}_{\text{消息 } \mu_{\psi_M \to A}} \\
&\times \underbrace{\left[\sum_B \sum_E \psi_B(B) \cdot \psi_E(E) \cdot \psi_A(A, B, E)\right]}_{\text{消息 } \mu_{\psi_A \to A}}
\end{aligned}
$$

**这就是 BP 的全部秘密：** 全局求和被拆成了三个局部求和，每个只涉及原始问题中的一个小子集。三个求和可以独立完成，然后结果相乘。

### 消息 1：从 $\psi_M$ 到 $A$

$$
\mu_{\psi_M \to A}(a) = \sum_{m \in \{\text{T},\text{F}\}} \psi_M(m, a)
$$

对 $a=\text{true}$：

$$
\mu_{\psi_M \to A}(\text{T}) = \Pr(M=\text{T} \mid A=\text{T}) + \Pr(M=\text{F} \mid A=\text{T}) = 0.70 + 0.30 = 1.0
$$

对 $a=\text{false}$：

$$
\mu_{\psi_M \to A}(\text{F}) = \Pr(M=\text{T} \mid A=\text{F}) + \Pr(M=\text{F} \mid A=\text{F}) = 0.01 + 0.99 = 1.0
$$

都是 1.0——因为 $\psi_M$ 这条支路没有观测约束时，它对 $A$ 的中立信息就是均匀的。

### 消息 2：从 $B, E$ 方向到 $A$

$$
\mu_{\psi_A \to A}(a) = \sum_{b \in \{\text{T},\text{F}\}} \sum_{e \in \{\text{T},\text{F}\}} \psi_A(a, b, e) \cdot \psi_B(b) \cdot \psi_E(e)
$$

对 $a=\text{true}$，代入数字：

$$
\begin{aligned}
\mu_{\psi_A \to A}(\text{T}) =& \; 0.95 \times 0.001 \times 0.002 \\
&+ 0.94 \times 0.001 \times 0.998 \\
&+ 0.29 \times 0.999 \times 0.002 \\
&+ 0.001 \times 0.999 \times 0.998
\end{aligned}
$$

逐项计算：

$$
\begin{aligned}
&= 0.0000019 \\
&\quad + 0.0009381 \\
&\quad + 0.0005794 \\
&\quad + 0.0009970 \\
&\approx 0.0025164
\end{aligned}
$$

对 $a=\text{false}$：

$$
\begin{aligned}
\mu_{\psi_A \to A}(\text{F}) =& \; 0.05 \times 0.001 \times 0.002 \\
&+ 0.06 \times 0.001 \times 0.998 \\
&+ 0.71 \times 0.999 \times 0.002 \\
&+ 0.999 \times 0.999 \times 0.998
\end{aligned}
$$

$$
\begin{aligned}
&= 0.0000001 \\
&\quad + 0.0000599 \\
&\quad + 0.0014186 \\
&\quad + 0.9960060 \\
&\approx 0.9974846
\end{aligned}
$$

### 合并消息

$$
\begin{aligned}
\Pr(A=\text{T} \mid J) &\propto \psi_J(J=\text{T}, A=\text{T}) \times 1.0 \times 0.0025164 \\
&= 0.90 \times 0.0025164 = 0.002265 \\
\Pr(A=\text{F} \mid J) &\propto \psi_J(J=\text{T}, A=\text{F}) \times 1.0 \times 0.9974846 \\
&= 0.10 \times 0.9974846 = 0.099748
\end{aligned}
$$

归一化：

$$
\Pr(A=\text{T} \mid J=\text{T}) = \frac{0.002265}{0.002265 + 0.099748} \approx 0.0222
$$

即约 **2.22%**——John 打电话来了，但报警器响的概率仍然很低。这是因为 burglary 和 earthquake 本身的先验概率极小（0.1% 和 0.2%），John 的误报率（10%）也使得"John 打来电话"本身不是一个很强的信号。

### 进一步：计算 $B$ 的后验

通过 $A$ 的结果可以继续向上传播，计算 $\Pr(B \mid J)$。从 $A$ 到 $\psi_A$ 的消息是上面算出的后验 $\Pr(A \mid J)$，再结合 $\psi_B$ 和 $\psi_E$ 的消息即可得到 $B$ 的边缘。

这个"接力"过程清晰地展示了 BP 的核心思想：**每条消息都是对上游信息的总结，下游节点只需要乘上新的消息，不需要重新处理原始数据。**

## Sum-Product BP 算法的正式定义

### 两种消息

**变量 → 因子消息**（乘法规则的应用）：

$$
\mu_{X \to \psi_F}(x) = \prod_{\psi_{F'} \in \mathcal{N}(X) \setminus \{\psi_F\}} \mu_{\psi_{F'} \to X}(x)
$$

这是"除了你之外，其他因子对我的看法"——乘积运算组合了条件独立的证据源。

**因子 → 变量消息**（加法 + 乘法规则的应用）：

$$
\mu_{\psi_F \to X}(x) = \sum_{\text{neighbors} \setminus \{X\}} \psi_F(\text{all neighbors}) \cdot \prod_{Y \neq X} \mu_{Y \to \psi_F}(y)
$$

这是"固定 $X=x$ 时，综合其他变量的当前信念，这个因子对 $x$ 的认可程度"。先乘（组合其他变量的信念）再加（消去它们的值）。

### 边缘分布

消息收敛后，变量 $X$ 的边缘分布：

$$
\Pr(X = x) \propto \prod_{\psi_F \in \mathcal{N}(X)} \mu_{\psi_F \to X}(x)
$$

### 算法流程

```
1. 初始化所有变量→因子消息为均匀分布
2. 对叶子节点（只连一个因子的变量），可直接发送先验消息
3. 重复直到收敛（树上只需两轮）：
   a. 对每个因子 ψ_F，对其每个邻居 X：
       收集所有其他邻居发来的变量→因子消息
       计算 ψ_F → X（先乘后加）
   b. 对每个变量 X，对其每个邻居 ψ_F：
       收集所有其他因子发来的因子→变量消息
       计算 X → ψ_F（乘积）
4. 用因子→变量消息的乘积计算各变量的边缘分布
```

## 树 vs 有环：Loopy BP

报警网络是树，一轮消息传递即可精确收敛。但大多数现实问题的因子图含有环。

### 环的影响

考虑一个三角形环：$X, Y, Z$ 两两相连，因子 $\psi_1(X,Y), \psi_2(Y,Z), \psi_3(Z,X)$。消息从 $X$ 出发，经过 $Y \to Z \to X$ 绕一圈回来——$X$ 收到了自己之前发出去的消息，相当于**证据被重复计数**。

- **收敛性**：不保证。消息可能震荡、发散或在两个状态间周期跳跃。
- **精确性**：收敛到的是近似后验，不是精确解。

这称为 **Loopy BP**。

### 收敛诊断与缓解

实践中常用的改进：

**阻尼（Damping）：** 消息更新时不完全替换旧值，而是做加权平均：

$$
\mu_{\psi_F \to X}^{(t+1)} = \alpha \cdot \mu_{\psi_F \to X}^{\text{new}} + (1 - \alpha) \cdot \mu_{\psi_F \to X}^{(t)}
$$

$\alpha \in (0, 1]$ 是阻尼系数。$\alpha=0.5$ 是常用起点。阻尼可以抑制震荡，但会减慢收敛速度。

**残差调度（Residual Scheduling）：** 不等所有消息都更新，而是优先更新"变化最大"的消息。定义消息残差：

$$
\text{Res} = \|\mu_{\psi_F \to X}^{\text{new}} - \mu_{\psi_F \to X}^{\text{old}}\|_1
$$

每次选择残差最大的消息所在的边进行更新。这种调度方式在有环图上通常比同步更新收敛更快。

**收敛判据：** 当所有消息残差的最大值低于阈值（如 $10^{-6}$）时停止。

### 一个数值实验

在三角形环上运行 Loopy BP：

- $\psi_1(x,y) = \exp(-(x-y)^2/2\sigma^2)$（鼓励 $x$ 和 $y$ 接近）
- $\psi_2(y,z) = \exp(-(y-z)^2/2\sigma^2)$
- $\psi_3(z,x) = \exp(-(z-x)^2/2\sigma^2)$

当 $\sigma$ 较小时（强约束），BP 可能在两个对称解之间震荡。引入阻尼后收敛到靠近真实后验的近似解。当 $\sigma$ 较大时（弱约束），BP 快速收敛且近似误差很小。

## Log-Domain BP：数值稳定性的关键

### 为什么要取对数

消息是一堆概率的乘积。当因子较多时，消息值会指数级趋近于零——很快就超出浮点数精度范围。

以报警网络为例，$\Pr(B=\text{T}) = 0.001$ 本身已经很小。如果网络有 20 个低概率事件相乘，结果会下溢到 0.0。

**解决方案：** 在对数域中运行 BP。定义：

$$
\ell_{X \to \psi_F}(x) = \log \mu_{X \to \psi_F}(x), \quad
\ell_{\psi_F \to X}(x) = \log \mu_{\psi_F \to X}(x)
$$

### 对数域中的消息更新

变量→因子消息的乘法变成加法：

$$
\ell_{X \to \psi_F}(x) = \sum_{\psi_{F'} \neq \psi_F} \ell_{\psi_{F'} \to X}(x)
$$

因子→变量消息的"先乘后加"变成"先加后指数-对数"：

$$
\ell_{\psi_F \to X}(x) = \log \sum_{\text{neighbors} \setminus \{X\}} \exp\left( \log \psi_F + \sum_{Y \neq X} \ell_{Y \to \psi_F}(y) \right)
$$

其中 $\log \sum \exp$ 需要用 **LogSumExp** 技巧稳定计算：

$$
\log(e^{a_1} + e^{a_2}) = \max(a_1, a_2) + \log(1 + e^{-|a_1 - a_2|})
$$

### Log-Domain BP 的Python 骨架

```python
import numpy as np

def log_sum_exp(log_vec):
    """稳定计算 log(sum(exp(log_vec)))"""
    m = np.max(log_vec)
    return m + np.log(np.sum(np.exp(log_vec - m)))

def log_bp_iteration(log_factor_to_var, log_var_to_factor,
                     log_potentials, neighbors):
    """
    一轮对数域 BP 更新。
    log_factor_to_var[f][v] = 因子 f 发给变量 v 的消息 (log domain)
    log_var_to_factor[v][f] = 变量 v 发给因子 f 的消息 (log domain)
    log_potentials[f] = 因子 f 的势函数表
    neighbors[f] = 因子 f 的邻居变量列表
    """
    # Step 1: 更新因子 → 变量消息
    for f_idx in range(len(log_potentials)):
        nb = neighbors[f_idx]
        for v_local, v_idx in enumerate(nb):
            # 其他邻居的 log 消息求和
            other_log_sum = sum(log_var_to_factor[nb[k]][f_idx]
                              for k in range(len(nb)) if k != v_local)
            # 对目标变量 x 的每种取值，计算 LogSumExp
            for x in range(2):  # 假设二元变量
                terms = []
                # 枚举其他邻居的所有取值组合
                # （这里仅为示意，实际应向量化或卷积优化）
                for configs in ...:
                    log_val = log_potentials[f_idx][configs + (x,)]
                    log_val += other_log_sum[configs]
                    terms.append(log_val)
                log_factor_to_var[f_idx][v_idx][x] = log_sum_exp(np.array(terms))

    # Step 2: 更新变量 → 因子消息（简单求和）
    for v_idx in range(len(log_var_to_factor)):
        for f_idx in log_var_to_factor[v_idx]:
            total = sum(log_factor_to_var[v_idx][other_f]
                       for other_f in log_var_to_factor[v_idx]
                       if other_f != f_idx)
            log_var_to_factor[v_idx][f_idx] = total

    # Step 3: 计算信念（归一化）
    beliefs = {}
    for v_idx in range(len(log_var_to_factor)):
        log_belief = sum(log_factor_to_var[v_idx].values())
        log_belief -= log_sum_exp(log_belief)  # 归一化
        beliefs[v_idx] = np.exp(log_belief)    # 转回概率域
    return beliefs
```

关键点：全程只在最后一步转回概率域，中间所有消息都在对数域中处理。

## 从 BP 到信念

BP 收敛后，每个变量 $X_i$ 得到一个 $k$ 维概率向量 $\mathbf{b}_i$（belief），称为**信念**：

$$
\mathbf{b}_i = [\Pr(X_i = \text{val}_1), \Pr(X_i = \text{val}_2), \dots, \Pr(X_i = \text{val}_k)]
$$

### 信念的三种用途

1. **最大后验（MAP）估计**：$\arg\max \mathbf{b}_i$——最可能的值。在报警网络中，$B$ 的 MAP 估计是"false"（因为 $\Pr(B=\text{T} \mid J) \approx 0.02$，远小于 0.98）。

2. **置信度**：$\max \mathbf{b}_i$——对估计的确定程度。$\mathbf{b}_i$ 越尖（一个值概率接近 1），置信度越高。

3. **软信息**：整个分布作为下游算法的输入。例如，用 BP 的输出初始化 MCMC 采样器可以大幅加速收敛。

### 报警网络的信念变化

| 变量 | 先验 ($\Pr$) | 给定 $J=\text{T}$ 后 ($\Pr$) | 变化倍数 |
|------|-------------|---------------------------|---------|
| $B=\text{T}$ | 0.001 | ~0.02 | **20x** |
| $E=\text{T}$ | 0.002 | ~0.05 | **25x** |
| $A=\text{T}$ | ~0.003 | ~~0.022 | **~7x** |
| $M=\text{T}$ | ~0.0027 | ~0.017 | **~6x** |

有趣的现象：$B$ 和 $E$ 的概率增长倍数最大（~20-25x），但由于先验极低，后验绝对值仍然很小。这就是贝叶斯推断的直观体现——"证据支持了某种可能性，但不一定足以翻转先验判断"。

## BP 在 LDPC 译码中的应用

### LDPC 码简介

低密度奇偶校验码（LDPC）是一种线性纠错码。一个 $(n,k)$ LDPC 码用 $m = n-k$ 个奇偶校验方程定义，每个方程涉及一小部分码字比特。这些校验方程自然地构成一个因子图：

- 变量节点：$n$ 个码字比特 $c_1, \dots, c_n$
- 因子节点：$m$ 个校验方程 $f_1, \dots, f_m$
- 边：如果比特 $c_j$ 参与了校验 $f_i$，则连边

### BP 译码

接收端收到带噪信号 $y$。译码任务是找到最可能的发送码字 $c$。

因子节点 $f_i$ 的势函数是"校验方程是否满足"的指示函数：

$$
\psi_i(c_{\mathcal{N}(i)}) = \begin{cases}
1 & \text{if } \bigoplus_{j \in \mathcal{N}(i)} c_j = 0 \\
0 & \text{otherwise}
\end{cases}
$$

其中 $\bigoplus$ 是二进制异或。

BP 消息在此上下文中被称为"置信传播译码"：消息 $\mu_{c_j \to f_i}$ 是比特 $c_j$ 的软判决（"我认为自己是 0 的概率是 0.8，是 1 的概率是 0.2"）；消息 $\mu_{f_i \to c_j}$ 是校验 $f_i$ 对 $c_j$ 的外部约束（"基于其他比特的当前信念和你必须满足的校验方程，你应该更偏向 0"）。

LDPC 码的因子图虽然含有环，但设计良好的 LDPC 码环长至少为 6，且图极其稀疏。BP 译码在这些图上接近最优性能。

### 一个具体例子

一个简单的 (3,1) 重复码：$c_1 = c_2 = c_3$。三个比特，两个校验方程：$c_1 \oplus c_2 = 0$，$c_1 \oplus c_3 = 0$。

因子图：

```
    c1
   /  \
  f1  f2
  |    |
  c2  c3
```

假设接收到的软信息为 $y = [0.8, -2.1, 1.5]$（正数偏向 0，负数偏向 1，绝对值表示置信度）。

初始化：$\mu_{c_j \to f_i}$ 从 $y$ 计算得到：

$$
\Pr(c_j = 0 \mid y_j) = \frac{1}{1 + \exp(-2y_j/\sigma^2)}
$$

第一轮 BP 迭代：

- $f_1$ 收到 $c_1$ 和 $c_2$ 的消息，计算：$c_1=0$ 且 $c_2=0$，或者 $c_1=1$ 且 $c_2=1$ 才能使校验成立。$f_1$ 据此向两端发送更新的消息。
- $f_2$ 类似地在 $c_1$ 和 $c_3$ 之间交换信息。
- 两轮迭代后，三个比特的信念趋于一致——译码成功。

> **Key Observation：** LDPC 译码是 BP 最具规模的工业应用。5G NR 标准中使用的 LDPC 码块长度为上千比特，BP 译码器在几十轮迭代内完成译码。这得益于因子图的稀疏性（每个校验只涉及 3-6 个比特）以及 BP 消息运算的局部性。

## BP 与 MCMC 的对比

BP 不是唯一的推断方法。另一种主流方法是马尔可夫链蒙特卡洛（MCMC）采样。

| 维度 | BP | MCMC（Metropolis-Hastings） |
|------|----|---------------------------|
| 输出 | 精确/近似边缘分布 | 来自后验的样本集合 |
| 收敛保证 | 树上精确；有环图上无保证 | 渐近保证收敛到真实后验 |
| 收敛速度 | 通常 10-100 轮 | 可能需要 $10^4$-$10^6$ 次采样 |
| 环的影响 | 导致近似误差 | 不影响正确性（只影响混合速度） |
| 高维问题 | 节点数增长时消息数线性增长 | 维度增加时混合时间指数增长 |
| 确定性 | 确定性算法（给定初值） | 随机算法（每次运行结果不同） |
| 调试难度 | 低（可逐消息追踪） | 高（需诊断收敛性） |

**何时用 BP：** 图是树状的、或者环长但稀疏、或者需要快速得到合理估计。

**何时用 MCMC：** 图高度有环且约束极强、或者后验多峰需要全面探索、或者需要保证渐近精确。

在实践中，BP 和 MCMC 可以互补：先运行 BP 得到信念，用信念构造一个好的提议分布（proposal distribution），再用 MCMC 做精调——这正是"近似推断 + 精调"的两阶段范式。

## 消息调度的艺术

BP 的收敛速度和质量高度依赖于消息更新的顺序。主要有三种调度策略：

### 1.  flooding（同步更新）

每轮同时更新所有消息。简单、并行性好，但可能收敛慢。

### 2.  serial（串行更新）

每轮按固定顺序逐个更新消息。刚更新的消息立即在本轮后续计算中被使用——信息传播更快。串行通常比同步收敛快约 2 倍。

### 3.  residual belief propagation（RBP）

优先更新残差最大的消息。RBP 在有环图上往往收敛更快，但每步需要额外计算残差排序。

```
# RBP 伪代码
queue = priority_queue()
for each edge (v, f):
    queue.push(edge, priority=inf)  # 全部初始化

while not converged:
    (v, f) = queue.pop()  # 取出残差最大的边
    old_msg = message[v][f]
    new_msg = compute_message(v, f)
    residual = ||new_msg - old_msg||
    message[v][f] = new_msg
    # 将受影响的邻居边加入队列
    for (v', f') in affected_edges(v, f):
        queue.push((v', f'), residual)
```

## 应用图景

BP 的应用远不止教学示例和 LDPC 译码：

| 领域 | 应用 | 图规模 | 图结构 |
|------|------|--------|--------|
| 通信 | Turbo 码、LDPC 码译码 | $10^3$-$10^4$ 节点 | 稀疏，有长环 |
| 计算机视觉 | 立体匹配、图像分割、去噪 | $10^5$-$10^6$ 节点 | 网格图（大量短环） |
| NLP | 句法分析、词性标注 | $10^2$-$10^3$ 节点 | 链状/树状 |
| 生物信息学 | 系统发育树、蛋白质结构预测 | $10^2$-$10^4$ 节点 | 树为主 |
| 统计物理 | Ising 模型、自旋玻璃 | $10^3$-$10^6$ 节点 | 网格/随机图 |
| 密码学 | 侧信道分析、LWE 私钥恢复 | $10^2$-$10^3$ 节点 | 随机双二部图 |

> **Key Observation：** BP 在图的环结构和变量稀疏性之间取得了一种微妙的平衡。当因子度数小（变量对之间的约束是局部的）且环长足够大（消息在绕环回来时已经足够"衰减"），BP 的近似质量接近精确。

## 总结

| 概念 | 一句话 |
|------|--------|
| 边缘化 | 加掉不关心的变量 |
| 因子图 | 把联合分布拆成局部函数乘积，画成二分图 |
| 消息 | 一个节点对另一个节点关于变量取值的"看法" |
| Sum-Product 规则 | 乘法组合独立证据，加法消去隐变量 |
| 树 vs 有环 | 树上精确；有环上近似，但实践中常有效 |
| Log-domain BP | 对数域中运行避免数值下溢 |
| 调度 | 串行更新比同步快；残差调度在有环图上更稳 |
| 信念 | BP 输出的软概率分布，可用于硬判决或下游算法 |

Belief Propagation 的核心思想极其简洁：**利用条件独立性把全局求和拆成局部消息传递。** 它不是一个万能的推断算法——有环图上的近似误差在某些问题中不可接受——但它在通信（LDPC/Turbo 译码）、计算机视觉（立体匹配）、统计物理（Ising 模型）等领域都做出了里程碑式的贡献。

**进一步：** BP 输出软信息。当需要做硬判决时——从一组离散候选值中选出最优组合——坐标下降是一个高效的精调工具。见同系列《从零开始理解坐标下降》。
