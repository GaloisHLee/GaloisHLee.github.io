# Solving LWE with Independent Hints about Secret and Errors — Lu, Feng, Pan (2025)


Reading: Lu, Feng, Pan (2025). *Solving LWE with Independent Hints about Secret and Errors.*

**问题设定：** 给定 LWE 实例 \((A, \mathbf{b} = \mathbf{s}A + \mathbf{e} \bmod q)\) 和一组精确的侧信道 hint（关于 \(\mathbf{s}\) 或 \(\mathbf{e}\) 的内积值），构造 primal attack 格基进行密钥恢复。

**本文的改进：** 将 Nowakowski-May (ASIACRYPT 2023) 嵌入 hint 时使用的 LLL 约简替换为 Hermite 标准型（HNF）——一个多项式次数更低的整数线性代数操作。Kyber512 上 234 个完美 hint 的基构造从 2.16 小时降至 0.35 小时。格基维度与行列式与 MN23 等价，对完美 hint 行列式略有增大。

**本笔记关注：** (1) 构造链——hint 转为 lattice hint 后如何通过矩阵乘法嵌入 primal attack 格基；(2) HNF 比 LLL 快在哪——多项式次数的差距及其工程含义；(3) 论文未说但值得追问的部分——带噪 hint、联合 hint、attack pipeline 完整评测的缺失。

<!--more-->

## 1. 从「证明安全」到「攻击模型」：Hint 攻击的学术路线

### 1.1 预设知识

本笔记假设读者已熟悉 primal attack 格基（详见 lattice-part-9）的构造与 BKZ block-size 估计：

$$
B^{\mathrm{LWE}} = \begin{pmatrix} qI_m & 0 & 0 \\ A & I_n & 0 \\ \mathbf{b} & 0 & 1 \end{pmatrix},\quad
\boldsymbol{\tau} = (-\mathbf{e}, \mathbf{s}, -1),\quad \det = q^m.
$$

Gaussian heuristic 预测 \(\lambda_1\) 与 \(\|\boldsymbol{\tau}\|\) 的 gap 决定攻击所需 BKZ block size \(\beta\)。Hint 的作用是缩小这个 gap——缩减维度或增大行列式——从而降低 \(\beta\)。

### 1.2 Hint 模型的动机：侧信道与代数泄漏

物理实现的侧信道（功耗、时序、电磁）给出的是关于密钥的 **额外代数约束**。 在 LWE 设定下，这典型地表现为关于 \(\mathbf{s}\) 或 \(\mathbf{e}\) 的部分内积信息：

- **完美 hint：** \(\langle \mathbf{v}, \mathbf{s} \rangle = \ell\)（精确整数值）。
- **模 q hint：** \(\langle \mathbf{v}, \mathbf{s} \rangle \equiv \ell \pmod{q}\)。
- **模任意数 hint：** \(\langle \mathbf{v}, \mathbf{s} \rangle \equiv \ell \pmod{m}\)。

这些 hint 的来源可以是多样化的：模板攻击恢复出的单个系数的 Hamming weight（经过一些代数处理后变成内积约束），乘法运算中的中间值泄漏，或者冷启动攻击中恢复出的部分密钥字节。 本文不讨论 hint 的来源，而是假设已经拥有了一组代数上精确的 hint，然后回答：**有了这些信息后，如何最快地构造出能够恢复剩余未知部分的格基？**

### 1.3 攻击路线的演化（关键节点）

- **DSDGR20 (CRYPTO):** 奠基性统一框架。将 hint 作为等式约束嵌入 DBDD 格基，每个 hint 增加一行一列，缩小 uSVP gap。局限：需要大量 hint 才能显著改善攻击参数；嵌入经过对偶格基，对 primal attack 的影响间接。
- **DSGHK23 (CRYPTO):** 引入不等式 hint（\(\langle \mathbf{v}, \mathbf{s} \rangle \in [L, U]\)），提供更几何化的 gap 分析框架。
- **MN23 (ASIACRYPT):** 突破性节点。直接嵌入 LWE 实例而非 DBDD，维度增长更小，统一处理完美和 modular hint。**核心工具是 LLL**——给定 hint 后，将 hint 矩阵与 LWE 格基组合成一个更大的矩阵，用 LLL 约简提取等效格基。缺陷：hint 数量增加时（如 Kyber512 的 200+ 个 hint），LLL 本身成为瓶颈。
- **LFP25 (本文):** 拒绝 LLL。用 HNF 将 hint 转换为 compact 核格基，通过矩阵乘法嵌入 primal attack 格基。全程只有整数矩阵运算——HNF、乘法、求逆、高斯消元——没有格约简。
- **HSMP25 (EUROCRYPT):** 平行的泛化框架，覆盖更多泄漏模型（含带噪泄漏）。在宽度上扩展，Lu 等人在深度（构造效率）上优化。
- **Belief propagation 路线:** 用统计推断处理带噪软信息。与 lattice reduction 线（擅长精确值泄漏）各有侧重，未来大概率收敛。

---

## 2. Lattice Hint：用格的语言描述侧信息

### 2.1 一个统一的观察

几乎所有的 LWE hint——不论是关于 \(\mathbf{s}\) 还是 \(\mathbf{e}\)——都可以被重新表述为：

> **有一个格 \(L\)，我们已知 secret（或 error，或其拼接）属于这个格。**

这个观察看似 trivial，但它将 hint 从散落的线性等式提升为统一的代数对象。 一旦你接受这个视角，嵌入问题就变成了一个纯粹的格操作问题：**如何将两个格基组合，使得新格基包含原 LWE 格的所有信息，同时嵌入了 hint 格的约束。**

> **Lattice Hint (Formal Definition).** 一个 lattice hint 是有序对 \((B, \text{target})\)，其中 \(B\) 为整数矩阵，\(\text{target} \in \{ \mathbf{s}, (\mathbf{s}, -1), \mathbf{e}, (\mathbf{e}, -1) \}\)，满足 target 属于 \(L(B)\)。

### 2.2 转化规则

**齐次 hint：\(\mathbf{s}V = 0\)（整系数或模 \(q\)）。**
所有满足这个等式的 \(\mathbf{s}\) 构成一个整数格。 求这个格的基 \(B_s\)，则有 \(\mathbf{s} \in L(B_s)\)。

**非齐次 hint：\(\mathbf{s}V = \boldsymbol{\ell}\)。**
通过将常数项吸收进变量：
$$
(\mathbf{s}, -1) \begin{pmatrix} V \\ \boldsymbol{\ell} \end{pmatrix} = 0,
$$

转化成了关于 \((\mathbf{s}, -1)\) 的齐次形式。
因此 \((\mathbf{s}, -1) \in L(B_{(\mathbf{s}, -1)})\)，
其中 \(B_{(\mathbf{s}, -1)}\) 是上述齐次系统的核格基。

**多个 modular hint：\(\langle \mathbf{v}_i, \mathbf{s} \rangle \equiv \ell_i \pmod{m_i}\)。**
每个 hint 定义一个格 \(L_i\)，所有格的交集仍是格。 因此 \((\mathbf{s}, -1) \in \bigcap_{i=1}^k L(B_i)\)。

### 2.3 HNF 求核格基

问题的核心变成了：**给定一个整数矩阵 \(V \in \mathbb{Z}^{n \times k}\)，如何高效地求出 \(\{\mathbf{s} \in \mathbb{Z}^n \mid \mathbf{s}V = 0\}\) 的格基。**

最自然的方法是通过 Smith 标准型或 Hermite 标准型。 HNF 给出幺模矩阵 \(U \in \mathbb{Z}^{n \times n}\) 使得

$$
UV = H,
$$

其中 \(H\) 是 \(V\) 的上三角 HNF，且后 \(n-k\) 行全为零（因为 \(V\) 是 \(n \times k\) 且通常列满秩，只有 \(k\) 个独立约束）。 因此 \(U\) 的后 \(n-k\) 行就是核格 \(B_s\) 的一组基。

对这个公式，我们需要一个关键的观察：**\(U\) 是幺模矩阵。** 这意味着它定义了一个 \(\mathbb{Z}^n\) 上的基变换。 将 \(V\) 的列向量在这个新基下重新表示，就自然把零空间分离出来了——这正是 HNF 的代数本质。

对于非齐次情况，先求任意一个特解 \(\mathbf{s}_0\) 满足 \(\mathbf{s}_0 V = \boldsymbol{\ell}\)，
然后所有满足 \(\mathbf{s}V = \boldsymbol{\ell}\) 的向量构成仿射空间 \(\mathbf{s}_0 + L(B_s)\)。
因此

$$
B_{(\mathbf{s}, -1)} = \begin{pmatrix} B_s & 0 \\ \mathbf{s}_0 & -1 \end{pmatrix},\quad
(\mathbf{s}, -1) = (\mathbf{z}, 1) \, B_{(\mathbf{s}, -1)}.
$$

### 2.4 复杂度分析：为什么 HNF 赢

这是全文的理论分水岭。 我们来对比两者的渐近复杂度：

- **LLL**（May-Nowakowski 用的工具）：\(\widetilde{O}(n^5 k \log^3 B)\)。
  这里 \(n\) 是输入维度（secret 维度），\(k\) 是 hint 数量，\(B\) 是矩阵元素的最大绝对值。
  \(n^5\) 因子意味着当 \(n\) 达到数百（正是 Kyber 和 Dilithium 所在的区域）时，即使 \(k\) 很小，LLL 的运行时间也可能迅速膨胀。

- **HNF**（论文用的替代方案）：\(\widetilde{O}(n k^3 \log^2 B)\)。
  注意这里的指数分布完全不同：\(n\) 的幂次从 \(5\) 降到了 \(1\)，\(k\) 的幂次保持在 \(3\)。
  在典型的侧信道攻击场景下——\(n \approx 256\text{--}1024\)，\(k \ll n\)（通常几十到几百个 hint）——HNF 的多项式次数是压倒性的优势。

更具体地说，当 \(n=256, k=128\) 时，\(n^5 k \approx 2.7 \times 10^{14}\)，而 \(n k^3 \approx 5.3 \times 10^8\)——差了 **六个数量级**。 即使在常数因子上 LLL 的实现可能更优化（由于几十年的工程积累），这个多项式差距也是 LLL 无法追赶的。

Note: 在 MN23 的框架中，LLL 并非攻击的核心计算瓶颈（BKZ 通常更重），但它成为了 **基构造** 的瓶颈。 即使 BKZ block size 很小，也需要先在 LLL 上等两个小时才能开始真正的 attack——基构造工具与攻击需求之间的不匹配。

---

## 3. Secret Hint 的嵌入：分块矩阵与行列式分析

这一节展开论文最核心的三条构造链：一般嵌入公式、完美 hint 的特化、mod-q hint 的特化。 每条链都从一个 lattice hint 出发，以一个新的 primal attack 格基结束。

### 3.1 一般嵌入公式

一旦有了 lattice hint \(B_s\) 使得 \(\mathbf{s} \in L(B_s)\)（即存在 \(\mathbf{z}\) 使 \(\mathbf{s} = \mathbf{z}B_s\)），
我们可以直接将 \(\mathbf{s} = \mathbf{z}B_s\) 代入 LWE 定义式 \(\mathbf{b} = \mathbf{s}A + \mathbf{e} + q\mathbf{k}\)：

$$
\mathbf{b} = \mathbf{z}B_s A + \mathbf{e} + q\mathbf{k}
\quad\Longrightarrow\quad
(-\mathbf{e}, \mathbf{s}, -1) = (\mathbf{k}, \mathbf{z}, -1)
\begin{pmatrix}
qI_m & 0 & 0 \\
B_s A & B_s & 0 \\
\mathbf{b} & 0 & 1
\end{pmatrix}.
$$

这个 \(B_s^{\mathrm{LWE}}\) 就是我们的新 primal attack 格基。 它本质上是用 \(B_s\) 对 \(A\) 做了一次行空间的线性变换——将 \(n\) 维的 secret 空间压缩到了 \(n-k\) 维（因为 \(B_s\) 是 \((n-k) \times n\) 的矩阵）。

对于非齐次情况 \((\mathbf{s}, -1) \in L(B_{(\mathbf{s}, -1)})\)，构造更为紧凑。
令 \(\overline{A} = \begin{pmatrix} A \\ \mathbf{b} \end{pmatrix}\)（将 \(\mathbf{b}\) 叠在 \(A\) 下方），则
$$
B_{(\mathbf{s}, -1)}^{\mathrm{LWE}} =
\begin{pmatrix}
qI_m & 0 \\
B_{(\mathbf{s}, -1)} \overline{A} & B_{(\mathbf{s}, -1)}
\end{pmatrix}.
$$

一个很有用的视角：\(B_{(\mathbf{s}, -1)} \overline{A}\) 这个乘积同时处理了 \(A\) 和 \(\mathbf{b}\)，
不需要像 Case 1 那样分三行处理。 这是因为 \((\mathbf{s}, -1)\) 天然地耦合了 secret 和常数项，使得 LWE 方程 \(\mathbf{b} - \mathbf{s}A = \mathbf{e} \pmod{q}\) 可以直接写为 \((\mathbf{s},-1) \overline{A} = -\mathbf{e} \pmod{q}\)。

### 3.2 完美 hint 的特化（九步推导链）

这是论文中最干净的一条构造链。 我们逐步展开，既呈现公式也解释每一步的代数理由。

**场景设定：**
已知 \(k\) 个完美整数 hint：\(\mathbf{s}V = \boldsymbol{\ell}\)，其中 \(V \in \mathbb{Z}^{n \times k}\)，
\(\boldsymbol{\ell} \in \mathbb{Z}^k\)。

**Step 1 —— 求核格基 \(B_s\)。**
计算 \(V\) 的 HNF。 存在幺模 \(U \in \mathbb{Z}^{n \times n}\) 使得 \(UV = H\)，
\(H\) 的后 \(n-k\) 行全零。 取 \(U\) 的后 \(n-k\) 行组成 \(B_s \in \mathbb{Z}^{(n-k) \times n}\)。

这一步的 key insight 是：HNF 不需要约简质量，它只需要矩阵的标准形。 不需要 shortest basis，只需要 *any* basis——因为我们用这组基的目的是在下一步嵌入 LWE 格基，而 BKZ 会在最终的格基上完成约简工作。

**Step 2 —— 求特解 \(\mathbf{s}_0\)。**
从 \(k\) 个线性方程中解出一个满足 \(\mathbf{s}_0 V = \boldsymbol{\ell}\) 的特解。 因为 \(V\) 的秩为 \(k\)（hint 向量线性无关——否则有冗余 hint 可以先剔除），任意特解都可以。

**Step 3 —— 参数化所有满足 hint 的 secret。**
这是线性代数的标准结论：齐次方程的所有解构成一个线性空间（格），加上一个特解后成为一个仿射空间：

$$
\mathbf{s} = \mathbf{s}_0 + \mathbf{z}B_s,\quad \mathbf{z} \in \mathbb{Z}^{n-k}.
$$

此时 \(\mathbf{z}\) 是新的「未知量」——维度从 \(n\) 降到了 \(n-k\)，但系数的取值范围从 \(\mathbb{Z}_q\) 变成了 \(\mathbb{Z}\)（因为没有模约简，hint 是精确的整数等式）。

**Step 4 —— 构造 \(B_{(\mathbf{s}, -1)}\)。**

$$
B_{(\mathbf{s}, -1)} =
\begin{pmatrix}
B_s & 0 \\
\mathbf{s}_0 & -1
\end{pmatrix}
\in \mathbb{Z}^{(n-k+1) \times (n+1)}.
$$

底行的 \(-1\) 编码了 \((\mathbf{s}, -1)\) 中 \(-1\) 分量恒为常数的约束。 验证 \((\mathbf{s}, -1) = (\mathbf{z}, 1) B_{(\mathbf{s}, -1)}\) 成立。

**Step 5 —— 嵌入 LWE 格基。**
将 \(B_{(\mathbf{s}, -1)}\) 代入 Case 2 的嵌入公式：

$$
B_{(\mathbf{s},-1)}^{\mathrm{LWE}}
= \begin{pmatrix}
qI_m & 0 & 0 \\
B_s A & B_s & 0 \\
\mathbf{b} - \mathbf{s}_0 A & -\mathbf{s}_0 & 1
\end{pmatrix}.
$$

注意底行 \(\mathbf{b} - \mathbf{s}_0 A\) 是 **「使用特解 \(\mathbf{s}_0\) 作为近似 secret 时对应的 adjusted \(\mathbf{b}\)」**。
这意味着我们将 \(\mathbf{b}\) 中已知的部分（由 \(\mathbf{s}_0\) 贡献的部分）从系统中扣除，
剩下待恢复的是 \(\mathbf{z}\) 对应的残差分量。

**Step 6 —— 维度分析。**
$$
\dim(B_{(\mathbf{s},-1)}^{\mathrm{LWE}}) = m + n + 1 - k.
$$

每个完美 hint 从系统中消除了一个自由度（将 \(n\) 维 secret 的搜索空间缩小了一维），
因此最终的格基维度从原始的 \(m+n+1\) 降到 \(m+n+1-k\)。
这等价于 MN23 中处理方法得到的维度——在这个指标上两者相同。

**Step 7 —— 行列式与几何分析。**
\(B_s\) 不是方阵，所以行列式需要从几何角度分析。
将基的前 \(m+n-k\) 行张成的子格记为 \(L_{\mathrm{sub}}\)。
底行 \((\mathbf{b} - \mathbf{s}_0A, -\mathbf{s}_0, 1)\) 在垂直于 \(L_{\mathrm{sub}}\) 的方向上的投影给出

$$
\det(B_{(\mathbf{s},-1)}^{\mathrm{LWE}})
= \det(L_{\mathrm{sub}}) \cdot
\|\mathrm{proj}_{\perp}(\mathbf{b} - \mathbf{s}_0 A,\, -\mathbf{s}_0,\, 1)\|.
$$

> **Key Observation.** \(\|\mathrm{proj}_{\perp}\| \ge 1\) 严格成立：投影向量的分量均为整数，在整数格的垂直投影方向上，任何非零整数向量的投影长度至少为 \(1\)（它是格本身的初等体积的约化因子）。

而在 MN23 中，相应的行列式为 \(\det(L_{\mathrm{sub}})\)（没有 \(\|\mathrm{proj}_{\perp}\|\) 因子）。
因此：
$$
\det_{\text{LFP}} = \det_{\text{MN23}} \cdot \|\mathrm{proj}_{\perp}\| \ge \det_{\text{MN23}}.
$$

对于 primal attack，行列式越大越好——Gaussian heuristic 预测的最短向量长度 \(\propto \det^{1/d}\)，
而行列式增加意味着 lattice 的「稀疏度」增加，使 target vector 更「突出」。

所以对于完美 hint，Lu 等人的方法不仅在构造速度上优于 MN23，在基的 **攻击质量**上也有优势：**相同维度，更大行列式。**

**Step 8 —— 与 MN23 在完美 hint 场景下的对比。**

| 指标 | MN23 (LLL) | Lu 等人 (HNF) | 差异 |
|------|-----------|---------------|------|
| 维度 | \(m+n+1-k\) | \(m+n+1-k\) | 相同 |
| 行列式 | \(\det(L_{\mathrm{sub}})\) | \(\det(L_{\mathrm{sub}}) \cdot \|\mathrm{proj}_{\perp}\|\) | \(\ge\) |
| 构造复杂度 | \(\widetilde{O}(n^5 k \log^3 B)\) | \(\widetilde{O}(n k^3 \log^2 B)\) | HNF 更优 |

**Step 9 —— 直觉总结。**
整个完美 hint 的构造链可以理解为：用 \(k\) 个线性方程将 \(n\) 维 secret 空间压到 \(n-k\) 维超平面上，然后将这个超平面作为一个格嵌入 LWE 实例中。 过程中唯一「非平凡」的操作是 HNF（求出超平面的基底），其余全是矩阵乘法和加法。

### 3.3 Mod-q hint 的特化（九步链）

完美 hint 是强的：它要求攻击者知道精确的整数值。 更现实的情况是模 \(q\) 下的等价类信息——模 \(q\) hint \(\mathbf{s}H \equiv \boldsymbol{\ell} \pmod{q}\)。

**Step 1 —— 转化。**
将模等式转为 \(\mathbb{Z}\) 上的齐次形式：

$$
(\mathbf{s}, -1) \begin{pmatrix} H \\ \boldsymbol{\ell} \end{pmatrix} \equiv 0 \pmod{q}.
$$

此时 \(\mathbf{s} = \mathbf{s}_0 + \mathbf{z}B_s\)，其中 \(B_s\) 是 \(\mathbf{s}H \equiv 0 \pmod{q}\) 的核格基。

**Step 2 —— 假设与分块。**
假设 \(k\) 个 hint 向量线性无关模 \(q\)。 则存在 \(k \times k\) 可逆子块。
重新排列 \(\mathbf{s}\) 的坐标，使前 \(k\) 个坐标对应的 \(k \times k\) 子矩阵 \(H_0\) 可逆：

$$
H = \begin{pmatrix} H_0 \\ H_1 \end{pmatrix},\quad
H_0 \in \mathbb{Z}_q^{k \times k},\; H_1 \in \mathbb{Z}_q^{(n-k) \times k}.
$$

**Step 3 —— 构造核格基 \(B_s\)。**
从 \(\mathbf{s}_k H_0 + \mathbf{s}_{n-k} H_1 \equiv 0 \pmod{q}\;\text{解出}\;\mathbf{s}_k \equiv -\mathbf{s}_{n-k} H_1 H_0^{-1} \pmod{q}\)。这意味着前 \(k\) 个坐标由后 \(n-k\) 个坐标模 \(q\) 决定，但可以在 \(\mathbb{Z}\) 上自由变化 \(q\) 的倍数。
因此核格基是一个 \((n) \times n\) 的矩阵：

$$
B_s = \begin{pmatrix}
qI_k & 0_{k \times (n-k)} \\
-H_1 H_0^{-1} & I_{n-k}
\end{pmatrix}.
$$

注意这里 \(B_s\) 是方阵（维度 \(n \times n\)），与前一种情形 \(B_s\) 是 \((n-k) \times n\) 不同。 这是因为模 \(q\) 约束不压缩维度——它只将前 \(k\) 维锁定为后 \(n-k\) 维的模 \(q\) 线性函数。

**Step 4 —— 划分 \(A\)。**
按照 \((k, n-k)\) 的分块：

$$
A = \begin{pmatrix} A_{11} & A_{12} \\ A_{21} & A_{22} \end{pmatrix},
$$

其中 \(A_{11} \in \mathbb{Z}_q^{k \times k}, A_{12} \in \mathbb{Z}_q^{k \times (m-k)}, A_{21} \in \mathbb{Z}_q^{(n-k) \times k}, A_{22} \in \mathbb{Z}_q^{(n-k) \times (m-k)}\)。

**Step 5 —— 构造 \(B_{q,H_s}^{\mathrm{LWE}}\)（\(5 \times 5\) 分块矩阵）。**
将 \(B_s\) 和分块的 \(A\) 代入 Case 1 嵌入公式（此时 \(\mathbf{s} \in L(B_s)\)），得到论文中最复杂的单个公式：

$$
B_{q,H_s}^{\mathrm{LWE}} =
\begin{pmatrix}
qI_k & 0 & 0 & 0 & 0 \\
0 & qI_{m-k} & 0 & 0 & 0 \\
qA_{11} & qA_{12} & qI_k & 0 & 0 \\
\widetilde{A}_0 & \widetilde{A}_1 & -H_1 H_0^{-1} & I_{n-k} & 0 \\
\widetilde{b}_0 & \widetilde{b}_1 & -s_{00} & -s_{01} & 1
\end{pmatrix}.
$$

这个矩阵看起来吓人，但它的块结构有清晰的逻辑：

- 前两行 \((qI_k, 0, \ldots)\) 和 \((0, qI_{m-k}, \ldots)\) 编码模 \(q\) 关系。
- 第三行 \((qA_{11}, qA_{12}, qI_k, \ldots)\) 来自 \(B_s A\) 乘积的前 \(k\) 行。
- 第四行 \((\widetilde{A}_0, \widetilde{A}_1, -H_1 H_0^{-1}, I_{n-k}, 0)\) 来自 \(B_s A\) 的后 \(n-k\) 行和 \(B_s\) 自身的下半部分。
- 第五行来自 \(\mathbf{b}\) 和 \(\mathbf{s}_0\)。

**Step 6 —— 行变换消去。**
用第一行消去第三行中的 \(qA_{11}\) 部分，用第二行消去第三行中的 \(qA_{12}\) 部分。
这是完全合法的格基变换（行变换对应左乘幺模矩阵，不改变格）：

$$
B_1 = \begin{pmatrix}
qI_k & 0 & 0 & 0 & 0 \\
0 & qI_{m-k} & 0 & 0 & 0 \\
0 & 0 & qI_k & 0 & 0 \\
\widetilde{A}_0 & \widetilde{A}_1 & -H_1 H_0^{-1} & I_{n-k} & 0 \\
\widetilde{b}'_0 & \widetilde{b}'_1 & -s'_{00} & 0 & 1
\end{pmatrix}.
$$

**Step 7 —— 消去冗余列，降维。**
\(B_1\) 的第 \(m+1\) 至 \(m+k\) 列（对应 \(qI_k\)）只包含 \(q\) 的倍数。
因为 BKZ 在格上操作时不依赖特定的列排列，
这些列与它们的行可以在不影响格结构的情况下被消去：

$$
B_{q,H_s}^{\prime\mathrm{LWE}} = \begin{pmatrix}
qI_k & 0 & 0 & 0 \\
0 & qI_{m-k} & 0 & 0 \\
A_{e0} & A_{e1} & I_{n-k} & 0 \\
b'_0 & b'_1 & 0 & 1
\end{pmatrix}.
$$

维度从 \(m+n+k+1\) 降到了 \(m+n+1-k\)，与 MN23 一致。

**Step 8 —— 行列式。**
\(\det(B_{q,H_s}^{\prime\mathrm{LWE}}) = q^m\)，与原始 LWE 格基相同。 这与 MN23 一致。

**Step 9 —— 恢复被消去的 \(\mathbf{s}_k\)。**
BKZ 恢复出 \((\mathbf{e}, \mathbf{s}_{n-k}, -1)\) 后，\(\mathbf{s}_k\) 可以通过模等式反推：

$$
\mathbf{s}_k \equiv -\mathbf{s}_{n-k} H_1 H_0^{-1} + \boldsymbol{\ell} H_0^{-1} \pmod{q}.
$$

这一步不需要额外的格操作。

### 3.4 Mod-q hint 对比总结

对于模 \(q\) hint，Lu 等人的方法与 Nowakowski-May 在格基的维度、行列式、和攻击质量上完全等价。 差异在构造速度上——HNF 替代了 LLL。 因为维度相同、行列式相同，这 5-7 倍的加速是纯粹工程上的收益，不影响攻击的可行性条件。

### 3.5 一个容易被忽略的细节：\(H_0\) 的可逆性

在模 \(q\) hint 的构造中，\(H_0\) 的可逆性假设是关键的。 如果 hint 向量集合恰好不能形成 \(k \times k\) 可逆子块，
行列置换后总可以找到一个可逆子块（因为 hint 向量是线性无关的），但这意味着需要重新排列 \(\mathbf{s}\) 的坐标——在实际的 SageMath 实现中需要处理这个置换。

如果 hint 向量线性相关呢？ 此时 \(H\) 的秩小于 \(k\)，表示有冗余 hint。 可以预处理：求出最大无关 hint 组，丢弃多余的。 这个预处理可以用高斯消元在 \(O(nk^2)\) 时间内完成，不会成为瓶颈。

---

## 4. Error Hint 的嵌入：三变量线性系统

### 4.1 前置：Secret 嵌入后的基

在 secret hint 嵌入完成后（§3 的输出），我们拥有一个形如

$$
B_s^{\mathrm{LWE}} =
\begin{pmatrix}
qI_m & 0 & 0 \\
T & V & 0 \\
\mathbf{t} & \mathbf{v} & 1
\end{pmatrix}
$$

的格基，且存在整数向量 \((\mathbf{k}, \mathbf{z}, -1)\) 满足

$$
(-\mathbf{e},\; \mathbf{s},\; -1) = (\mathbf{k},\; \mathbf{z},\; -1) \cdot B_s^{\mathrm{LWE}}.
$$

将矩阵乘法展开为分量等式：

$$
\begin{aligned}
-\mathbf{e} &= \mathbf{k} \cdot qI_m + \mathbf{z} \cdot T + (-1) \cdot \mathbf{t}
= q\mathbf{k} + \mathbf{z}T - \mathbf{t}, \\
\mathbf{s} &= \mathbf{z} \cdot V + (-1) \cdot \mathbf{v}
= \mathbf{z}V - \mathbf{v}.
\end{aligned}
$$

第一个等式是我们处理 error hint 的起点：它把 \(\mathbf{e}\) 表示成了两个自由变量
\(\mathbf{k}\)（mod-\(q\) 消去向量，每一条 LWE 样本对应一个自由度）
和 \(\mathbf{z}\)（secret 的参数化系数）的线性组合。

### 4.2 三变量系统：\((\mathbf{k}, \mathbf{z}, \mathbf{w})\) 的来源

现在引入 error hint。以 Case 1 为例：已知整数矩阵 \(B\) 使得 \(\mathbf{e} \in L(B)\)。
这意味着存在整数向量 \(\mathbf{w}\) 满足 \(\mathbf{e} = \mathbf{w}B\)。

将 \(\mathbf{e} = \mathbf{w}B\) 代入 \(-\mathbf{e} = q\mathbf{k} + \mathbf{z}T - \mathbf{t}\)：

$$
-\mathbf{w}B = q\mathbf{k} + \mathbf{z}T - \mathbf{t}
\quad\Longrightarrow\quad
q\mathbf{k} + \mathbf{z}T + \mathbf{w}B = \mathbf{t}.
$$

> **三变量线性系统。** 方程 \(q\mathbf{k} + \mathbf{z}T + \mathbf{w}B = \mathbf{t}\) 包含三个未知块：
> \(\mathbf{k}\)（长度 \(m\)，编码每条 LWE 样本的模 \(q\) 自由量）、
> \(\mathbf{z}\)（长度取决于 secret hint 的数量，编码 \(\mathbf{s}\) 的参数化）、
> \(\mathbf{w}\)（长度取决于 error hint 容器的维度，编码 \(\mathbf{e}\) 的参数化）。
>
> 相比之下，§3 的 secret hint 嵌入中只有 \(\mathbf{z}\) 一个变量。
> 这额外的两个变量——\(\mathbf{k}\) 和 \(\mathbf{w}\)——使得 error hint 嵌入的代数复杂度高出一个小阶。

### 4.3 Case 1：\(\mathbf{e} \in L(B)\) —— 五步构造

**Step 1 —— 整理方程。**
令齐次部分为 \(q\mathbf{k} + \mathbf{z}T + \mathbf{w}B = 0\)，
则原方程是它的一个非齐次偏移（右端是 \(\mathbf{t}\)）。

**Step 2 —— 求齐次核格基。**
求解齐次线性系统 \(q\mathbf{k} + \mathbf{z}T + \mathbf{w}B = 0\)，得到核格基 \((K, Z, W)\)。
\(K\) 的行数（核的维数）决定了最终格基的行数。
这一步可以用整数线性代数（高斯消元或 HNF）完成——不涉及格约简。

**Step 3 —— 求特解。**
找一个特解 \((\mathbf{k}_0, \mathbf{z}_0, \mathbf{w}_0)\) 满足整方程
\(q\mathbf{k}_0 + \mathbf{z}_0 T + \mathbf{w}_0 B = \mathbf{t}\)。
因为方程是线性且一致（hint 来自真实 secret/error，一定存在解），特解总是存在的。

**Step 4 —— 参数化 \((\mathbf{k}, \mathbf{z})\)。**
所有解可写为

$$
(\mathbf{k},\; \mathbf{z},\; \mathbf{w}) = (\mathbf{k}_0,\; \mathbf{z}_0,\; \mathbf{w}_0) + \boldsymbol{\omega} \cdot (K,\; Z,\; W),
$$

其中 \(\boldsymbol{\omega}\) 是新的自由整数向量（维度等于核的维数）。
由此提取出最终需要的 \((\mathbf{k}, \mathbf{z})\) 部分：

$$
(\mathbf{k},\; \mathbf{z}, -1)
= (\boldsymbol{\omega},\; 1) \cdot
\begin{pmatrix}
K & Z & 0 \\
\mathbf{k}_0 & \mathbf{z}_0 & -1
\end{pmatrix}.
$$

**Step 5 —— 乘回 \(B_s^{\mathrm{LWE}}\)，得到最终的 primal attack 基。**

$$
\begin{aligned}
B_e^{\mathrm{LWE}}
&= \begin{pmatrix}
K & Z & 0 \\
\mathbf{k}_0 & \mathbf{z}_0 & -1
\end{pmatrix}
\cdot
\begin{pmatrix}
qI_m & 0 & 0 \\
T & V & 0 \\
\mathbf{t} & \mathbf{v} & 1
\end{pmatrix} \\
&= \begin{pmatrix}
qK + ZT & ZV & 0 \\
q\mathbf{k}_0 + \mathbf{z}_0 T - \mathbf{t} & \mathbf{z}_0 V - \mathbf{v} & -1
\end{pmatrix}.
\end{aligned}
$$

注意最末行的 \(-1\) 保持完整——确保 target vector \((-\mathbf{e}, \mathbf{s}, -1)\) 在新基中仍以 \((\boldsymbol{\omega}, 1)\) 的整数组合形式存在。构造全程只用矩阵乘法、HNF（核格求解）、和加减法——无 LLL 参与。

### 4.4 Case 2：\((\mathbf{e}, -1) \in L(\overline{B})\) —— 仅符号变化

当 lattice hint 的目标是 \((\mathbf{e}, -1)\) 而非单独的 \(\mathbf{e}\) 时，
使用 \(\overline{B}\) 表示该 hint 的格基。记 \(\overline{B}'\) 为 \(\overline{B}\) 的前 \(m\) 列（提取 \(\mathbf{e}\) 的坐标部分）。

从 \((\mathbf{e}, -1) = \mathbf{w}\overline{B}\) 得 \(\mathbf{e} = \mathbf{w}\overline{B}'\)。
代入 \(-\mathbf{e} = q\mathbf{k} + \mathbf{z}T - \mathbf{t}\)：

$$
-\mathbf{w}\overline{B}' = q\mathbf{k} + \mathbf{z}T - \mathbf{t}
\quad\Longrightarrow\quad
q\mathbf{k} + \mathbf{z}T + \mathbf{w}\overline{B}' = \mathbf{t}.
$$

与 Case 1 的方程 \(q\mathbf{k} + \mathbf{z}T + \mathbf{w}B = \mathbf{t}\) 相比，
唯一的差异是 hint 矩阵从 \(B\) 换成了 \(\overline{B}'\)，
\(\mathbf{w}\) 项的符号保持不变（都是 \(+\)，因为我们消去了 \(-\mathbf{e}\) 与 \(-\mathbf{w}\overline{B}'\) 之间的负号）。
后续的核格求解、特解、参数化、和矩阵乘法步骤与 Case 1 完全相同：

$$
\overline{B}_e^{\mathrm{LWE}} =
\begin{pmatrix}
q\overline{K} + \overline{Z}T & \overline{Z}V & 0 \\
q\overline{\mathbf{k}}_0 + \overline{\mathbf{z}}_0 T - \mathbf{t} & \overline{\mathbf{z}}_0 V - \mathbf{v} & -1
\end{pmatrix}.
$$

本质上，Case 2 是 Case 1 的「把 \(\mathbf{e}\) 改写成 \((\mathbf{e}, -1)\) 的格成员关系」的语法变形，
不引入额外的代数结构。

### 4.5 嵌入成本与边际收益

与 secret hint 嵌入相比，error hint 嵌入的核心额外开销是求解三变量线性系统。
这一步仍然是整数线性代数（HNF / 高斯消元），不是格约简——因此额外开销是多项式级别的，
随 hint 数量多项式增长而非指数增长。

但从攻击者的角度，一个更实际的问题是边际收益。
每个 hint（不论是关于 \(\mathbf{s}\) 还是 \(\mathbf{e}\)）消去一个自由度，格基维度减 1，
行列式不变，Gaussian heuristic 预测的 \(\lambda_1 \propto \det^{1/(d-1)}\) 略微增大——有助于攻击。
然而 \(\mathbf{e}\) 在许多 LWE 参数选择中比 \(\mathbf{s}\) 更短，
\(\mathbf{e}\) 自身的短向量属性在 BKZ 阶段已经构成自然优势。
因此从 \(\mathbf{e}\) 中消去自由度的边际改善可能不如从 \(\mathbf{s}\) 中消去来得大。
这本质上是一个实验问题——论文没有提供 \(\mathbf{s}\)-only 与 \(\mathbf{e}\)-only hint 的对比攻击数据。

### 4.6 独立假设的局限

论文框架要求每个 hint 只涉及 \(\mathbf{s}\) **或** \(\mathbf{e}\) 之一，
不处理形如 \(\langle \mathbf{v}_s, \mathbf{s} \rangle + \langle \mathbf{v}_e, \mathbf{e} \rangle = \ell\)
的联合 hint。
这个限制是构造性的——一旦 hint 同时涉及两者，
lattice hint 的 target 就无法简单设为 \(\mathbf{s}\) 或 \(\mathbf{e}\) 之一，
对应的三变量线性系统也会退化为不同形式。

在实际攻击中，许多侧信道泄漏确实只涉及一方（例如 Kyber 加密中只有 \(\mathbf{s}\) 参与多项式乘法）。
但解密过程的模板攻击可能同时恢复两方的部分信息。
处理联合 hint 需要更一般的格基构造方法——这是本路线的一个自然延伸方向。

---

## 5. 实验与数据分析

### 5.1 实验设置

所有实验在 SageMath 10.3 上运行，MacBook Pro M1，最高 CPU 频率 3.2 GHz。
对比基准是 Nowakowski & May (2023) 的方法在相同参数下的实现。

需要注意的一点是：MN23 的原始实现未必是针对 hint 大量场景优化的——如果对 MN23 的代码做工程优化（例如使用更好的 LLL 实现、或针对 hint 矩阵的特殊结构做预处理），LLL 的绝对运行时间可能有所下降。 但多项式级别的差距（\(n^5\) vs \(n^1\)）不会因为常数因子优化而消失——这正是理论分析的价值所在。

### 5.2 Kyber512 数据

| Hint 数量 | MN23 (LLL) | Lu et al. (HNF) | 加速比 |
|----------:|-----------:|----------------:|-------:|
| 32 | 51.2 秒 | 11.2 秒 | **4.57×** |
| 64 | 73.9 秒 | 23.5 秒 | **3.14×** |
| 128 | 488.0 秒 | 67.7 秒 | **7.21×** |
| 234 | 7788.8 秒 | 1243.6 秒 | **6.26×** |

### 5.3 Falcon512 数据

| Hint 数量 | MN23 (LLL) | Lu et al. (HNF) | 加速比 |
|----------:|-----------:|----------------:|-------:|
| 32 | 47.8 秒 | 13.6 秒 | **3.51×** |
| 64 | 82.1 秒 | 27.9 秒 | **2.94×** |
| 128 | 550.1 秒 | 101.0 秒 | **5.45×** |
| 233 | 5494.8 秒 | 1612.6 秒 | **3.41×** |

### 5.4 趋势分析

几条值得注意的观察：

**加速比的非单调性。** 在 Kyber512 上，128 hints 的加速比（7.21×）优于 234 hints（6.26×）。 Falcon512 上同样的模式。 这暗示 HNF 的 \(k^3\) 因子在 hint 数量很大时开始显现——当 \(k\) 从 128 增长到 234（约 1.8×），\(k^3\) 增长约 6×，但 LLL 的 \(n^5 k\) 中 \(k\) 是线性的。 所以加速比在 \(k\) 的中等区间达到峰值，之后 HNF 的相对优势略微缩小。 这并不意味着 HNF 变慢了——它的绝对时间仍然远小于 LLL——只是 LLL 在这个区间「追回」了一些。

**Falcon512 的加速比整体低于 Kyber512。** Falcon 的 secret 分布和维数参数与 Kyber 不同，这可能影响 LLL 在 MN23 中的内部行为（LLL 的性能高度依赖输入基的「质量」和结构）。 但在所有测试点上 HNF 方法都保持了显著的速度优势。

**234 hints 场景的绝对时间对比。** MN23 在 Kyber512 上用了 2.16 小时——仅仅是为了构造格基，还没有开始 BKZ。 而 Lu 等人的方法只需 20.7 分钟。 在实际攻击的 pipeline 中（包括 BKZ 的后续阶段），这近两个小时的节省可能意味着攻击的可行性差异——特别是在需要重复运行（尝试不同的 hint 组合、或者验证 hint 的正确性）的场景下。

### 5.5 实验报告的局限

实验仅报告了基构造时间，没有报告：

- **BKZ 阶段的运行时间。** 基构造完成后，实际攻击仍需要运行 BKZ（block size \(\beta\) 由 target vector 和 Gaussian heuristic 的 gap 决定）。 如果 BKZ 阶段的运行时间是 \(2^{0.292\beta}\)，基构造的 2 小时对 20 分钟的差异在 \(\beta\) 较大时可能被 BKZ 的运行时间淹没。 但对于 \(\beta\) 较小的情况（hint 很多时 \(\beta\) 降低），基构造时间的差异就更显著。

- **内存消耗。** HNF 的峰值内存使用可能与 LLL 不同——这对于在有限资源上运行大规模实验的用户很重要。

- **数值精度。** LLL 通常在浮点近似下运行（\(L^2\) 或 floating-point LLL），而 HNF 是精确的整数运算。 精确运算在大数系数下可能产生大的多精度整数，影响实际运行时间。

---

## 6. 代码架构（实现正在落实）

这一节是 API 签名和模块结构的占位——实际的 SageMath/Python 实现将在后续版本中填充。

### 6.1 模块树

```text
lwe_hints/
  __init__.py
  lwe_instance.py          # A, b, q, dimension, validation
  lattice_hints.py         # hint → lattice hint 的转换
  hnf_kernel.py            # HNF 核格求解与整数线性代数
  secret_embedding.py      # B_s^LWE 和 B_(s,-1)^LWE 的构造
  error_embedding.py       # secret 嵌入后的 error hint 嵌入
  primal_attack.py         # BKZ/LLL 编排与向量验证
  estimates.py             # 行列式、Gaussian heuristic、目标范数、beta 估计
  experiments.py           # 复现计时实验
```

### 6.2 核心数据结构

```python
from dataclasses import dataclass
from typing import Literal

# Matrix 和 Vector 类型别名在实际实现中使用 SageMath 的 Matrix 或 numpy

@dataclass
class LWEInstance:
    """LWE 实例 (A, b = sA + e mod q)。"""
    A: 'Matrix'       # n × m 在 Z_q 上
    b: 'Vector'       # 长度 m 在 Z_q 上
    q: int
    n: int
    m: int

@dataclass
class PerfectHint:
    """完美 hint ⟨v, s⟩ = ℓ。"""
    v: 'Vector'       # 长度 n
    ell: int

@dataclass
class ModQHint:
    """模 q hint ⟨v, s⟩ ≡ ℓ (mod q)。"""
    v: 'Vector'
    ell: int
    q: int

@dataclass
class ModularHint:
    """模 m hint ⟨v, s⟩ ≡ ℓ (mod m)。"""
    v: 'Vector'
    ell: int
    modulus: int

@dataclass
class LatticeHint:
    """Lattice hint：目标向量属于 L(B)。"""
    B: 'Matrix'
    target: Literal["s", "s_minus_one", "e", "e_minus_one"]
```

### 6.3 主要函数 API

```python
# ---- lattice_hints.py ----

def lattice_hint_from_perfect_secret_hints(V, ell):
    """
    将完美 hint sV = ℓ 转换为 lattice hint。

    Input:
        V:  n × k 整数矩阵（hint 向量）
        ell: 长度 k 的整数向量（目标值）

    Output:
        Bs: (n-k) × n 核格基，{s | sV = 0} = L(Bs)
        s0: 特解 s0·V = ell
        B_s_minus_one: (n-k+1) × (n+1) 基，(s,-1) ∈ L(B_s_minus_one)
    """
    # 实现正在落实
    pass


# ---- secret_embedding.py ----

def embed_secret_s_in_lattice(A, b, q, Bs):
    """
    构造 B_s^LWE：s ∈ L(Bs) 的情况。

    Input:
        A:  n × m (Z_q)
        b:  长度 m (Z_q)
        q:  模数
        Bs: (n-k) × n 核格基

    Output:
        B_s_LWE: (m+n-k+1) × (m+n-k+1) 格基
    """
    # 实现正在落实
    pass


def embed_secret_s_minus_one_in_lattice(A, b, q, B_s_minus_one):
    """
    构造 B_(s,-1)^LWE：(s,-1) ∈ L(B_s_minus_one) 的情况。
    """
    # 实现正在落实
    pass


# ---- hnf_kernel.py ----

def construct_Bs_from_modq_hints(H, q):
    """
    从模 q hint sH ≡ 0 (mod q) 构造 Bs。

    假设 H = [H0; H1] 且 H0 在 Z_q 上可逆。
    返回 Bs = [[q I_k, 0], [-H1·H0^{-1}, I_{n-k}]]。
    """
    # 实现正在落实
    pass


def recover_eliminated_secret_part(sn_minus_k, H1, H0_inv, ell):
    """
    从 BKZ 恢复的 s_{n-k} 反推 s_k：
        s_k ≡ -s_{n-k}·H1·H0^{-1} + ell·H0^{-1}  (mod q)
    """
    # 实现正在落实
    pass


# ---- error_embedding.py ----

def embed_error_hint_e_in_L(B_secret, B_error):
    """
    在 secret hint 嵌入后嵌入 error hint：e ∈ L(B_error)。
    """
    # 实现正在落实
    pass


def embed_error_hint_e_minus_one_in_L(B_secret, B_error_bar):
    """
    在 secret hint 嵌入后嵌入 error hint：(e,-1) ∈ L(B_error_bar)。
    """
    # 实现正在落实
    pass


# ---- primal_attack.py ----

def validate_recovered_vector(v, A, b, q, hints):
    """
    验证 BKZ 恢复的候选向量：
    1. 形状与 (-e, s, -1) 或约简形式兼容
    2. 最后一个坐标为 ±1
    3. 恢复的 s 满足 b ≡ sA + e (mod q)
    4. s 和 e 范数符合期望分布
    5. 所有输入 hint 被满足
    """
    # 实现正在落实
    pass
```

---

## 7. 贡献、局限与后续方向

### 7.1 贡献

MS23 的 LLL 嵌入方法在 hint 数量增加时，基构造本身成为瓶颈——LLL 的 \(\widetilde{O}(n^5 k)\) 与本应受益的其他攻击阶段不成比例。

Lu 等人的核心洞察：**hint 是一组精确的线性约束，其解空间是一个可以用整数线性代数——而非格约简——计算的格。** 接受这一点后，构造链退化为 HNF 求核格基、矩阵乘积做嵌入、高斯消元做降维。每步的代数依据都是标准的整数线性代数。

实验上 5–7× 的加速是实质性的。Kyber512 在 234 个完美 hint 下基构造从 2.16 小时降至 20.7 分钟——从「需要专门规划计算资源」到「笔记本上可运行」的粒度切换。

### 7.2 局限

**（a）精确 hint 假设。** 论文假设 hint 完全精确。真实侧信道 hint 通常带噪声。HNF 对错误等式不具有容错性——一个错误 hint 会将格基推向错误方向。需要额外的预处理层（belief propagation 或多次采样与投票）来过滤。

**（b）评测不完整。** 仅报告基构造时间，未报告完整 attack pipeline（含 BKZ）。若 BKZ 占总时间的 95%，基构造加速对整体时间的影响会相应稀释。

**（c）缺乏 hint 选择策略。** 论文回答「给定一组 hint 后如何最快嵌入」，不回答「哪些 hint 对攻击帮助最大」。对于攻击者可以主动选择测量目标的场景，hint 的信息论价值分析是缺失的 prior。

**（d）联合 hint 不在框架内。** 如 §4.5。双变量泄漏（\(\mathbf{s}\) 与 \(\mathbf{e}\) 同时暴露的场景）需要额外处理。

### 7.3 后续方向

- **BP + HNF 融合：** BP 做带噪 hint 预处理（去噪、软判决）→ 清理后的 hint 送入 HNF 管道做格基构造。对 HSMP25 框架（EUROCRYPT 2025）的实现级补充。
- **Hint selection 优化问题：** 固定测量预算下选择最优 hint 子集，最大化攻击效果。组合优化 + hint 之间的信息冗余分析。
- **Ring/Module-LWE 推广：** Module 结构的代数约束在转换为 \(\mathbb{Z}\)-格时会损失代数效率增益——需要特殊的格基构造技巧。
- **代码开源：** MN23 和 Lu 等人的方法目前缺乏标准实现。一旦有可靠的 SageMath/Python 库，这条攻击路线将从学术工具进入安全评估和 CTF 社区。

### 7.4 继续阅读

- lattice-part-9：primal attack 框架与 BKZ block-size 估计
- lattice-part-10：侧信道泄漏类型与实现断层线
- Toy exercise：用 \(n=4, m=10, q=17\) 手动走一遍完美 hint 的 HNF 构造链——小维度上的矩阵变化即大维度版本的 scale-down
- 取 Kyber512 标准参数，用 lwe-estimator 估算需要多少 hint 才能将 BKZ block size \(\beta\) 降至安全边界以下

---

## 参考文献

- **DSDGR20:** Dachman-Soled, Ducas, Gong, Rossi. *LWE with Side Information: Attacks and Concrete Security Estimation.* CRYPTO 2020.
- **DSGHK23:** Dachman-Soled, Gong, Hanson, Kippen. *Revisiting Security Estimation for LWE with Hints from a Geometric Perspective.* CRYPTO 2023.
- **MN23:** Nowakowski, May. *Too Many Hints — When LLL Breaks LWE.* ASIACRYPT 2023.
- **HSMP25:** Hermelink, Streit, Mårtensson, Petri. *A Generic Framework for Side-Channel Attacks against LWE-based Cryptosystems.* EUROCRYPT 2025.
- **LFP25:** Lu, Feng, Pan. *Solving LWE with Independent Hints about Secret and Errors.* 2025.
- **CN11:** Chen, Nguyen. *BKZ 2.0: Better Lattice Security Estimates.* ASIACRYPT 2011.
- **APS15:** Albrecht, Player, Scott. *On the Concrete Hardness of Learning with Errors.* 2015.
- **KB79:** Kannan, Bachem. *Polynomial Algorithms for Computing the Smith and Hermite Normal Forms of an Integer Matrix.* SIAM Journal on Computing, 1979.

---

*关联阅读：lattice-part-9（Primal/Dual 攻击与 BKZ 具体安全性估计）；lattice-part-10（格子实现中的断层线与侧信道泄漏类型）。*

