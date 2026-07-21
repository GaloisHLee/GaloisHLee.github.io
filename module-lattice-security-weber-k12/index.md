# Module Lattice Security (Part I) 论文讲义：从 Weber 猜想到 RLWE/MLWE 安全性


Reading: *Module Lattice Security (Part I): Unconditional Verification of Weber's Conjecture for k<=12* (arXiv:2604.15858v2).

**论文主结论：** 作者证明了对 \(k \le 12\)，与 \(2^{k+1}\)-次单位根相关的 maximal real cyclotomic subfield 的 class number 满足 \(h_k = 1\)。这给出了一个对当前密码学相关参数区间足够强的、无条件的 Weber conjecture 验证结果。

**这份讲义的重点：** 不是把论文从头翻成中文，而是回答四个更难的问题：为什么 class number one 会突然进入 RLWE / MLWE / module lattice 语境；Section 3 的证明链到底如何接起来；Algorithm 1 为什么足以给出 unconditional verification；以及一个代数基础一般的密码学读者究竟该补哪些知识，才可能独立读懂这篇论文。

**建议读法：** 先读本文的“导读、阅读路线、数学补丁、符号表”，再进入逐章精读。若直接冲进论文 Section 2，大概率会在 class group eigenspace、cyclotomic units、generalized Bernoulli numbers 和 Herbrand theorem 这些点上同时卡住。

<!--more-->

## 1. 导读：作者到底证明了什么

这篇论文表面上在做一个 cyclotomic field 的 class number 问题，深层上却在碰一个 structured-lattice cryptography 里长期悬着的代数前提。

把记号先摆清楚。令

$$
K_n = \mathbb{Q}(\zeta_{2^{n+2}}), \qquad
K_n^+ = \mathbb{Q}(\zeta_{2^{n+2}} + \zeta_{2^{n+2}}^{-1}),
$$

其中 \(\zeta_m\) 是原始 \(m\)-次单位根。论文记 \(h_n\) 为 \(K_n^+\) 的 class number。作者最终证明的是：对于 \(n \le 12\)，有 \(h_n = 1\)。也就是说，在这些层数上，\(K_n^+\) 的 ideal class group 是平凡的，所有 ideals 都是 principal。

如果只从纯数论的角度看，这当然已经是一个很完整的问题。但对密码学读者而言，更值得抓住的是另外一句话：**class number one 不是装饰性的背景，它会改变某些 ideal-lattice 和 module-lattice reduction 中“底层对象是否自由”“理想是否可由单个元素生成”“codifferent 是否更易处理”“RLWE 与 PLWE 是否更容易无条件对齐”这一类结构问题。**

所以读这篇论文时，真正的任务不是死记一个结论，而是理解下面这条翻译链：

> cyclotomic real subfield 的 class number  
> \(\Downarrow\)  
> principal ideal / free module / simpler codifferent structure  
> \(\Downarrow\)  
> ideal-lattice 与 module-lattice reductions 的某些技术前提  
> \(\Downarrow\)  
> RLWE / MLWE / ML-KEM / ML-DSA 所依赖的 structured ring or module 叙事更稳固

论文的主证明不是一条直接的“算出 class number”路线，而是一条三阶段的排除链：

1. 先用 Fukuda-Komatsu sieve 和 Wieferich-type 条件，排除大部分可能整除 \(h_n\) 的奇素数。
2. 再利用 tower 里的 norm-coherence 和已有归纳信息，杀掉低阶 characters 对应的 class-group eigenspaces。
3. 最后把剩余 full-order characters 的问题转译为 generalized Bernoulli number 的整除性，再通过一个有限候选集与显式计算完成收尾。

这也是这份讲义的核心结构。我们不会一开始就扎进 Theorem 3.1 到 Theorem 3.3，而是先把“为什么会有这三步”这件事讲清楚。

为了便于校验，这份讲义会明确覆盖论文研究问题、背景、重要性与核心贡献；提供适合基础较弱读者的阅读路线与依赖图；按论文章节逐段讲解；并在末尾补上每章练习与延伸阅读地图。

## 2. 为什么这个数论结果会碰到 RLWE / MLWE

第一次看到这篇论文时，密码学读者最自然的困惑通常不是证明细节，而是：

> 这篇论文明明没有出现 RLWE samples，也没有出现 BKZ、KEM、decryption failure。为什么它会被说成和 module lattice security 有关？

原因在于很多 structured-lattice reduction 并不是直接在“矩阵样本”层面工作，而是在更底层的 number field、ring of integers、ideals、fractional ideals、codifferent 和 module structure 上工作。只要 reduction 真正使用了这些对象，它就会关心两件事：

1. 这些对象是不是足够好处理。
2. 某些“看起来应该成立”的自由性或主理想性质，到底是条件性的还是无条件的。

在 ideal-lattice 和 RLWE 的经典叙事里，cyclotomic rings 之所以占据中心位置，不只是因为它们支持快速乘法，还因为它们的代数结构异常整齐。论文 Section 4 明确强调：当某个相关 real cyclotomic subfield 的 class number 为 \(1\) 时，许多理想与模结构上的麻烦会显著减少。更直接地说，某些“module over \(R_q\) 是否自由”“理想是否必为 principal”“codifferent 是否更容易给出显式生成元”“PLWE 和 RLWE 的比较是否更干净”之类的问题，都更容易在 class number one 的情形下得到无条件处理。

这并不等于“如果没有这个结果，RLWE 就不安全”。正确的理解是：

- 这篇论文不是在分析攻击，而是在清理 reduction 背后的代数地基。
- 它不是直接给 ML-KEM 增加或删除安全比特，而是在缩小某类 structured-lattice proofs 依赖的条件性假设空间。
- 它也解释了为什么 modern module-lattice constructions 经常强调“module over \(R_q\)”而不是停留在纯 ideal-lattice 语境：后者在更深层数论上会暴露更多结构性风险。

所以这份讲义的密码学部分，不会只说“和后量子密码有关”。我们会把这条结构翻译链写细，直到读者能回答：

1. class group 在 cryptographic reduction 里到底出现在哪。
2. class number one 消除了哪些技术摩擦。
3. 为什么 Module-LWE 线最终没有沿 ideal-lattice 的某些最脆弱方向走下去。

## 3. 这份讲义怎么读

这不是一篇适合从第一行顺序硬啃到最后一行的论文。更现实的读法是分层推进。

第一层先建立对象感：

- ideal、principal ideal、fractional ideal 到底是什么；
- class group 衡量的到底是什么失败；
- cyclotomic field 和 maximal real subfield 的塔结构长什么样。

第二层再建立工具感：

- characters 和 idempotents 为什么能把 class group 切成 eigenspaces；
- generalized Bernoulli numbers 为什么会突然出现；
- norm maps 和 tower coherence 为什么可以配合归纳使用。

第三层才进入主证明：

- Theorem 3.1 做了什么筛除；
- Proposition 3.1 / Corollary 3.1 怎样消去低阶 characters；
- Theorem 3.2 / Proposition 3.2 怎样把问题压缩成有限候选集；
- Algorithm 1 怎样把剩余情况穷尽掉；
- Theorem 3.3 为什么由此成立。

如果把这三层顺序颠倒，读者几乎必然会在 proof stack 中失去方向。因此本文后续会先给一份明确的阅读路线，再补数学背景，再逐章精读。

## 4. 阅读路线：如果没有 algebraic number theory，应先补哪些知识

这部分不追求“把代数数论学完”，而追求 **尽快达到能读懂这篇论文的最低阈值**。对目标读者而言，最容易浪费时间的方式是从头啃一本完整教材；更合理的方式是先围绕论文真正使用的对象和定理反向补。

我建议分三轮读。

### 4.1 第一轮：先建立对象感，不碰证明细节

目标只有一个：看到论文 Section 2 时，不再把每个名词都当成第一次见。

第一轮必须掌握：

1. group、ring、field、module 的最小定义。
2. ideal、principal ideal、fractional ideal、class group、class number 的语义。
3. cyclotomic field、maximal real subfield、Galois group、character、idempotent 的最小图景。
4. norm、trace、ring of integers 这三个基本算术对象。

第一轮可以先接受为黑箱的内容：

- Herbrand-Ribet theorem 的完整证明。
- Ferrero-Washington theorem 的完整证明。
- Thaine theorem 的技术细节。
- Iwasawa invariants 的一般理论。

换句话说，第一轮不是为了“会证明”，而是为了“知道作者为什么会用这些东西”。

### 4.2 第二轮：对照原论文读 Section 2

原论文 Section 2 的真实结构是：

1. `2.1 Number fields and rings of integers`
2. `2.2 Ideals, fractional ideals, and the class group`
3. `2.3 Galois theory of cyclotomic fields`
4. `2.4 Character theory`
5. `2.5 Eigenspace decomposition of the class group`
6. `2.6 Cyclotomic units`
7. `2.7 Generalized Bernoulli numbers`
8. `2.8 The Cyclotomic \(\mathbb{Z}_2\)-Tower`
9. `2.9 Iwasawa theory`

读法上不要平均用力。

必须真正看懂的部分：

- `2.1` 到 `2.5`：这是后面 theorem chain 的语言基础。
- `2.6` 的 Sinnott formula 在这里不是背景，而是 Section 3.1 的 formal exit。
- `2.7` 的 generalized Bernoulli numbers 是 Section 3.3 的可计算接口。
- `2.8` 的 tower norm 结构是 Section 3.2 的核心。

可以先只理解用途、暂不深究完整证明的部分：

- `2.9 Iwasawa theory` 的一般框架。
- `2.6` 和 `2.7` 背后的历史发展脉络。

### 4.3 第三轮：带着问题读 Section 3 和 Section 4

当你进入主证明时，脑中要持续问四个问题：

1. 当前这一段在排除什么样的奇素数 \(\ell\)。
2. 这一段排除的是“素数大小”、"Frobenius 同余类" 还是 “class-group eigenspace”。
3. 当前使用的是哪条桥：Sinnott、tower norm、Herbrand-Ribet，还是显式计算。
4. 这一段怎样回到 \(h_k^+ = 1\) 或密码学含义。

### 4.4 最小依赖图

```text
group/ring/field/module
    ->
ideal / principal ideal / class group
    ->
number field / ring of integers / norm / trace
    ->
cyclotomic field K_n, maximal real subfield K_n^+, Galois group G_k
    ->
characters chi, idempotents e_chi, eigenspace decomposition
    ->
cyclotomic units + Sinnott formula
    ->
generalized Bernoulli numbers + Herbrand-Ribet
    ->
Section 3 theorem chain
    ->
Section 4: RLWE / MLWE / module freeness / PIP / PLWE equivalence
```

### 4.5 必学与可跳过

**必学**

- ideal 与 principal ideal 的区别。
- class group 测量什么失败。
- \(K_n\)、\(K_n^+\)、\(G_k\)、\(\chi\)、\(e_\chi\)。
- cyclotomic units、Sinnott formula 的使用方式。
- generalized Bernoulli number 与 candidate primes 的联系。
- why full-order characters are the only survivors after Section 3.2.

**可暂时跳过**

- Iwasawa theory 的一般历史背景。
- Thaine 优化的完整证明。
- 参考文献中的更一般 Herbrand / Stickelberger 理论。

## 5. 数学补丁：从 ideal 到 class group，再到 cyclotomic field

下面这几组对象，是读这篇论文最小不可少的数学补丁。这里不追求面面俱到，而追求“够用而且不误导”。

### 5.1 Group、Ring、Field、Module：语言层为什么先出现

**为什么提出它**

- Group 负责抽象“可组合的对称性”。
- Ring 负责同时讨论加法与乘法。
- Field 让除法在非零元素上可行。
- Module 是“把向量空间里的标量域换成一般环”后的对象，是 MLWE 语境的真正语言。

**直观理解**

- Group 像“只有一种运算的对称系统”。
- Ring 像“整数那样可以加减乘”的宇宙。
- Field 像“有除法的 ring”。
- Module 像“系数来自 ring 的向量空间”，但不一定有基，也不一定自由。

**小例子**

- \(\mathbb{Z}\) 是 ring，不是 field。
- \(\mathbb{Q}\)、\(\mathbb{F}_p\) 是 field。
- \(R_q^d\) 是 MLWE 中最直接出现的 \(R_q\)-module。

**在论文中的位置**

- Section 4.2 讨论 MLWE 时，作者显式写出 \(A \in R_q^{d \times d}\)、\(s,e \in R_q^d\)。
- “underlying module free” 是 Section 4 最关键的 module 级条件。

**与密码学关系**

- 现代 lattice KEM 不只是“矩阵 over \(\mathbb{Z}_q\)”；它们常常是 “module over \(R_q\)”。
- 一旦你不理解 module，就会误读“module freeness”在 reduction 里的分量。

### 5.2 Ideal 与 Principal Ideal：为什么元素分解不够

**为什么提出它**

在一般整数环里，元素未必仍有唯一分解。ideal 是用来挽救“分解理论”的对象。

**直观理解**

- ideal 不是一个单独元素，而是一整个对 ring 乘法封闭的“倍数集合”。
- principal ideal 是由单个元素生成的 ideal，记为 \((\alpha)\)。

**小例子**

- 在 \(\mathbb{Z}\) 中，每个 ideal 都是 \((n)\)，所以所有 ideals 都是 principal。
- 在一般 number field 的整数环中，这个性质可能失败，于是就有了 class group。

**常见误区**

- “理想就是元素的别名”是错的。
- “principal ideal obvious enough, no need to distinguish it from ideal” 也是错的；这正是 class number 的核心分界线。

**在论文中的位置**

- Section 2.2 先讲 ideals、fractional ideals、class group。
- Section 4 又回到 “all ideals are principal” 对 codifferent 和 RLWE 分析的简化作用。

**与密码学关系**

- PIP 本质上就是“给定 principal ideal，恢复一个生成元”的问题。
- ideal-lattice attack line 正是围绕 principal ideals 与短生成元展开。

### 5.3 Class Group 与 Class Number：论文到底在证明什么

**为什么提出它**

因为“哪些 ideals 不是 principal”需要一个全局量来测量。

**形式化对象**

令 \(I_K\) 为 \(K\) 的 fractional ideals 乘法群，\(P_K\) 为 principal fractional ideals 子群，则

$$
\mathrm{Cl}(K) = I_K / P_K.
$$

class number 就是这个有限群的大小：

$$
h(K) = |\mathrm{Cl}(K)|.
$$

**直观理解**

- \(\mathrm{Cl}(K)\) 记录“离 principal 还差多少类”。
- \(h(K) = 1\) 等价于 every ideal is principal。

**在论文中的位置**

- 论文真正证明的是 \(h_k^+ = 1\)。
- Theorem 3.3 的最终结论不是一句模糊的“猜想成立”，而是对 \(k \in \{9,10,11,12\}\) 明确给出 \(h_k^+ = 1\)。

**与密码学关系**

- class number one 让 principal ideal / free module / codifferent simplification 这几件事同时变干净。

### 5.4 Cyclotomic Field 与 Maximal Real Subfield：为什么恰好是这些域

**为什么提出它**

因为 RLWE / MLWE 常用的环来自 cyclotomic integer rings，而 Weber conjecture 正是这条塔上的 class number 问题。

**论文中的特定对象**

$$
K_n = \mathbb{Q}(\zeta_{2^{n+2}}), \qquad
K_n^+ = \mathbb{Q}(\zeta_{2^{n+2}} + \zeta_{2^{n+2}}^{-1}).
$$

这里 \(K_n\) 是 full cyclotomic field，\(K_n^+\) 是其 maximal real subfield。

**直观理解**

- \(K_n\) 带有所有 \(2^{n+2}\)-次单位根信息。
- \(K_n^+\) 只保留“实部方向”的子域。

**小例子**

- Section 4 直接指出 ML-KEM 和 ML-DSA 使用的
  \(R_q = \mathbb{Z}_q[x] / (x^{256} + 1)\)
  来自 \(K_9 = \mathbb{Q}(\zeta_{512})\) 的整数环商。

**与密码学关系**

- 这就是为什么论文特别强调 \(h_9^+ = 1\)：它对应当前标准中 degree-256 的现实参数带。

### 5.5 Character、Idempotent、Eigenspace：为什么 class group 会被切片

**为什么提出它**

因为 class group 太大太粗，必须按照 Galois action 分解成小块，才能逐块排除。

**直观理解**

- \(\chi\) 是 Galois group 的一维表示。
- \(e_\chi\) 是群环里的投影算子。
- \(\mathrm{Cl}(K_k^+)[\ell]^{e_\chi}\) 是“只看 character \(\chi\) 这一块”的 eigenspace。

**论文中的角色**

- Section 2.5 建立 eigenspace decomposition。
- Proposition 3.1 和 Corollary 3.1 的核心含义就是：低阶 characters 那些块可以被归纳消掉，只剩 full-order characters 需要继续追。

**常见误区**

- 不要把 “class group 非平凡” 理解成一个不可分解的整体事件。
- 在这篇论文里，更准确的问题是：**哪一个 \(\chi\)-eigenspace 还可能非零。**

### 5.6 Cyclotomic Units、Sinnott Formula：Section 3.1 的 formal exit

**为什么提出它**

Fukuda-Komatsu sieve 想排除 \(\ell \mid h_k^+\)，需要把 class number 和某个可测试的单位结构联系起来。

**论文中的关键桥**

Theorem 2.1 给出 Sinnott formula，把
\(h_k^+\)
与 cyclotomic unit index 联系起来。Section 3.1 再用 Wieferich-type 条件说明某些 \(\ell\) 不可能整除这个 index，因此也不可能整除 \(h_k^+\)。

**直观理解**

- class number 本来太全局。
- cyclotomic units 提供了一个更接近“显式可算”的入口。

**一个小例子**

先看最小 real cyclotomic toy field：
\(\mathbb{Q}(\zeta_8)^+=\mathbb{Q}(\sqrt{2})\)。
这时

$$
\xi_3=\frac{\zeta_8^3-\zeta_8^{-3}}{\zeta_8-\zeta_8^{-1}}
=\frac{\sin(3\pi/8)}{\sin(\pi/8)}
=1+\sqrt{2}.
$$

这个例子教给读者两件事：

1. cyclotomic unit 不是抽象存在性对象，而是真的能写成一个熟悉 number field 里的具体单位；
2. 论文里抓 \(\xi_5\) 去做 residue test，方法论上和这里抓住 \(1+\sqrt2\) 这种显式单位是同一路数。

**常见误区**

- 不要把 “cyclotomic units” 误解成 “全部 units”。它们只是单位群里的一个显式子群。
- 也不要把 Sinnott 公式误读成 “每个 unit 都能直接读出 class number”。真正可控的是指数
  \([\mathcal{O}_{K_k^+}^{\times}:C^+]\)，不是随手挑一个单位就够。

**在论文中的位置**

- `2.6 Cyclotomic units` 提供对象 \(\xi_a\) 与 \(C^+\)。
- `3.1 The Fukuda-Komatsu sieve` 把 \(\xi_5\) 变成可检验的 local residue probe。

### 5.7 Generalized Bernoulli Numbers 与 Herbrand：Section 3.3 的可计算接口

**为什么提出它**

当 Section 3.2 只剩 full-order characters 后，作者需要一个对象把“某个 eigenspace 非零”转译成“某个整数可被 \(\ell\) 整除”。

**论文中的桥**

- Theorem 3.2 是 Herbrand-Ribet 型结论。
- Proposition 3.2 进一步说明候选集合 \(S_k\) 是 finite and computable。

**直观理解**

- class-group information 被送入 Bernoulli-number norms。
- 这一步让抽象代数对象最终落到显式整数因子分解和 residue test 上。

**一个小例子**

先看 conductor \(4\) 上唯一的非平凡奇 character
\(\psi\)，满足
\(\psi(1)=1\)、\(\psi(3)=-1\)。
按定义，

$$
B_{1,\psi}
=\frac{1}{4}\bigl(\psi(1)\cdot1+\psi(3)\cdot3\bigr)
=\frac{1}{4}(1-3)
=-\frac12.
$$

这个例子虽然非常小，但它已经说明 generalized Bernoulli number 在本文里的真正气质：

1. 它不是“神秘常数表”中的一个编号；
2. 它是 character 加权后的一个显式有理数；
3. 一旦你以后再取 norm、看素因子，就已经进入了 paper 的 candidate-generation 语言。

**常见误区**

- 不要把这里的 \(B_{1,\psi}\) 和初等分析里熟悉的 \(B_1=-1/2\)、\(B_2=1/6\) 那串 classical Bernoulli numbers 直接混为一谈。这里是 **character-twisted** 版本。
- 也不要把
  \(\ell\mid \operatorname{N}(B_{1,\psi})\)
  误读成
  “\(B_{1,\psi}\) 本身是一个被 \(\ell\) 整除的整数”。
  真正可分解、可谈素因子的对象是 rational norm，而不是原始 field element 自身。

**在论文中的位置**

- `2.7 Generalized Bernoulli numbers` 给出 \(B_{1,\psi}\) 与 Stickelberger evaluation。
- `3.3 Herbrand's theorem bounds candidate primes` 用这些 norms 生成有限候选素数集。

### 5.8 Number Field、Ring of Integers、Integral Basis：为什么整数环不是凭空出现

**为什么提出它**

- RLWE/MLWE 里出现的环，不是随手写一个模多项式商环就结束了；在证明层，真正的母对象往往是某个 number field \(K\) 及其整数环 \(\mathcal{O}_K\)。
- 论文 Section 2.1 一上来就固定这些对象，是因为 class group、units、norm、trace 都住在这里。

**形式化定义**

- number field 是 \(\mathbb{Q}\) 的有限扩张。
- \(\mathcal{O}_K\) 是 \(K\) 中所有对首一整系数多项式整的元素组成的环。
- integral basis 指一组 \(\omega_1,\dots,\omega_n\)，使得
  $$
  \mathcal{O}_K = \mathbb{Z}\omega_1 \oplus \cdots \oplus \mathbb{Z}\omega_n.
  $$

**直观理解**

- \(\mathbb{Z}\) 是 \(\mathbb{Q}\) 的整数环。
- \(\mathcal{O}_K\) 则是 “\(K\) 里最像整数的元素全集”。
- integral basis 让你知道：\(\mathcal{O}_K\) 虽然复杂，但仍是一个有限秩 \(\mathbb{Z}\)-lattice。

**几何理解**

- 经过 Minkowski embedding，\(\mathcal{O}_K\) 会变成 \(\mathbb{R}^{r_1} \times \mathbb{C}^{r_2}\) 里的一个格。
- 这也是为什么 number theory 会和 lattice geometry 接上。

**小例子**

- \(K=\mathbb{Q}(i)\) 时，\(\mathcal{O}_K=\mathbb{Z}[i]\)。
- \(K=\mathbb{Q}(\sqrt{2})\) 时，\(\mathcal{O}_K=\mathbb{Z}[\sqrt{2}]\)。
- 对本文最重要的 toy field，\(\mathbb{Q}(\zeta_{32})^+=\mathbb{Q}(\zeta_{32}+\zeta_{32}^{-1})\) 的次数是 \(8\)；Sage 可直接构出其 defining polynomial。

**常见误区**

- “\(\mathcal{O}_K\) 就是 \(\mathbb{Z}[\alpha]\)” 不是总成立。
- “选个 polynomial quotient 就等于完全理解了 ring of integers” 也不对；很多 reduction 真正用的是整数环而不是某个 presentation。

**在论文中的位置**

- `2.1 Number fields and rings of integers`。
- Section 4 里 \(R_q\) 被解释为某个整数环模掉 \(q\) 的商，这正是从 \(\mathcal{O}_{K_9}\) 落到标准实现的路径。

### 5.9 Prime Ideal、Dedekind Domain、Fractional Ideal：为什么“素数分解”要升级

**为什么提出它**

- 在 \(\mathcal{O}_K\) 中，元素的唯一分解可能失败。
- 但 ideals 的分解在 Dedekind domain 里会恢复唯一性，所以 class group 才有稳定基座。

**直观理解**

- prime ideal 是 “ideal 版的素数”。
- fractional ideal 允许分母，因此 ideal multiplication 才能形成群。
- Dedekind domain 可以理解成“ideal factorization 特别干净的 Noetherian 一维整闭整环”。

**小例子**

- 在 \(\mathbb{Z}\) 中，\((p)\) 就是 prime ideal。
- 在 \(\mathbb{Z}[\sqrt{-5}]\) 里，元素唯一分解失败，但 prime ideal factorization 仍然工作。

**图示**

```text
元素唯一分解可能失败
    ->
改看 ideal 分解
    ->
prime ideals 唯一分解
    ->
class group 衡量“哪些 ideals 不是 principal”
```

**与密码学关系**

- RLWE 与 ideal-lattice reduction 关心的 lattice 往往来自 ideals 的 canonical embedding。
- PIP、SGP、codifferent 等对象都默认你能在 ideal language 中工作。

**在论文中的位置**

- `2.2 Ideals, fractional ideals, and the class group` 是全篇最不可跳过的预备节之一。

### 5.10 Norm、Trace、Minkowski 直觉：为什么 finite computation 有机会成立

**为什么提出它**

- norm 和 trace 让 field 元素、ideals、Bernoulli-number norms 都能落回整数或较小域。
- 没有这些映射，你很难把 “抽象 eigenspace 非零” 变成 “显式整数被某个 \(\ell\) 整除”。

**形式化定义**

若 \(L/K\) 是有限扩张，则

$$
\operatorname{Tr}_{L/K}(\alpha)=\sum_{\sigma:L\hookrightarrow \overline{K}} \sigma(\alpha), \qquad
\operatorname{N}_{L/K}(\alpha)=\prod_{\sigma:L\hookrightarrow \overline{K}} \sigma(\alpha).
$$

对 ideal 也有 norm，记为 \(\operatorname{N}(\mathfrak{a}) = |\mathcal{O}_K / \mathfrak{a}|\)。

**直观理解**

- trace 把“所有共轭之和”收集起来。
- norm 把“所有共轭之积”收集起来。
- 它们是把高维对象压回低维算术数据的机器。

**小例子**

在 \(K=\mathbb{Q}(\sqrt{2})\) 中，

$$
\operatorname{Tr}(a+b\sqrt{2}) = 2a, \qquad
\operatorname{N}(a+b\sqrt{2}) = a^2 - 2b^2.
$$

**与密码学关系**

- canonical embedding 下的 Gaussian 参数、smoothing parameter、codifferent 大小分析，都和 norm/trace 或其诱导结构缠在一起。

**在论文中的位置**

- Section 3.3 真正使用的是 Bernoulli number 的 norm。
- Proposition 3.2 之所以能给出有限候选集，就是因为这些 norms 是显式有界整数。

### 5.11 Units、Dirichlet Unit Theorem、Cyclotomic Units：为什么单位群突然变成攻击面和证明面

**为什么提出它**

- Section 3.1 的 sieve 经由 Sinnott formula 进入 cyclotomic unit index。
- Section 4.1 的 ideal-lattice quantum attack 线也会碰到 log-unit lattice。

**Dirichlet 单位定理的直觉**

若 \(K\) 有 \(r_1\) 个实嵌入和 \(r_2\) 对复嵌入，则

$$
\mathcal{O}_K^\times \cong \mu_K \times \mathbb{Z}^{r_1+r_2-1}.
$$

这说明单位群不是随机长出来的，而是“有限 torsion \(\times\) 一个显式秩的自由阿贝尔群”。

**直观理解**

- units 是可逆整数。
- cyclotomic units 是单位群里一批特别可写、特别适合计算的单位。
- Sinnott formula 说明：虽然全体单位群太大，但 cyclotomic units 已经足以捕捉 class number 的关键指数信息。

**图示**

```text
unit group O_K^x
    ->
cyclotomic units C^+
    ->
index [O_K^x : C^+]
    ->
via Sinnott formula touches h_k^+
```

**与密码学关系**

- Cramer 等攻击 ideal lattices 时，依赖 cyclotomic units 在 log-unit lattice 里的特殊地位。
- 这也是为什么“class number one 让 cyclotomic units 满秩”会在 Section 4.1 里变成一把双刃剑：证明更干净，攻击条件也更清楚。

**在论文中的位置**

- `2.6 Cyclotomic units`。
- `3.1 The Fukuda-Komatsu sieve`。
- `4.1 Quantum attacks on ideal lattices`。

### 5.12 Hilbert Class Field 与 Herbrand Theorem：为什么 class group 会和 Bernoulli numbers 发生关系

**为什么提出它**

- 光知道 class group 是一个商群还不够。你还需要理解：它为什么能被 Galois action、characters、Bernoulli divisibility 控制。
- Hilbert class field 给出 class group 的 Galois avatar；Herbrand theorem 则给出 cyclotomic eigenspace 与 Bernoulli 数之间的桥。

**直观理解**

- Hilbert class field \(H_K\) 是 \(K\) 的“最大 everywhere unramified abelian extension”。
- 类域论告诉你：\(\operatorname{Gal}(H_K/K)\) 与 \(\mathrm{Cl}(K)\) 紧密对应。
- 这解释了为什么 class group 不是一个孤零零的组合对象，而是 Galois-theoretic object。

**论文真正使用的版本**

- 论文不重证 Herbrand theorem。
- 它使用的是 Herbrand-Ribet 型结论：若某个 full-order eigenspace 非零，则对应 generalized Bernoulli norm 必被 \(\ell\) 整除。

**图示**

```text
nonzero class-group eigenspace
    ->
Herbrand / Ribet bridge
    ->
Bernoulli divisibility
    ->
finite candidate prime set
```

**常见误区**

- 不要把 Herbrand theorem 误解为“它直接算出 class number”。
- 它在本文里的角色更像一个转译器：把难以直接算的 class-group 事件翻译成可以做整数分解的整除事件。

**在论文中的位置**

- `2.7 Generalized Bernoulli numbers` 给出输入对象。
- `3.3 Herbrand's theorem bounds candidate primes` 给出真正的输出。

### 5.13 Codifferent、Dual Lattice、Canonical Embedding：为什么 RLWE 的几何母语不是“系数向量”

**为什么提出它**

- 一旦进入 RLWE / ideal-lattice reduction，光知道 \(\mathcal{O}_K\) 是个环还不够。
- 你还需要知道：它在 canonical embedding 下对应什么格，它的对偶格是什么，以及这个对偶格为什么会由一个 field-side 对象控制。
- 这正是 codifferent 的角色。

**形式化定义**

对 number field \(K\) 的整数环 \(\mathcal{O}_K\)，定义

\[
\begin{aligned}
\mathcal{O}_K^\vee
&=
\{x\in K:\operatorname{Tr}_{K/\mathbb{Q}}(x\mathcal{O}_K)\subseteq \mathbb{Z}\}.
\end{aligned}
\]

这就是 codifferent。更一般地，对一个 fractional ideal \(I\)，它的 dual ideal 写作

\[
I^\vee = \mathcal{O}_K^\vee I^{-1}.
\]

**直观理解**

- trace pairing
  \(\langle x,y\rangle=\operatorname{Tr}_{K/\mathbb{Q}}(xy)\)
  是 number field 上最自然的“整数值内积”来源。
- codifferent 收集的正是：哪些元素 \(x\) 能和整个 \(\mathcal{O}_K\) 做 trace pairing，且结果始终落在 \(\mathbb{Z}\)。
- 所以它不是附属对象，而是“整数环在对偶方向上长什么样”的代数答案。

**几何理解**

- canonical embedding 把 \(\mathcal{O}_K\) 送成欧氏空间里的一个格。
- 这个格的 dual lattice，恰好对应 \(\mathcal{O}_K^\vee\) 的 embedding。
- 因而：

```text
field-side codifferent
    <->
Euclidean-space dual lattice
```

- 这也是为什么 RLWE 的几何母语更接近 “embedding + dual lattice + trace pairing”，而不只是“多项式系数向量”。

**小例子**

在
\(K=\mathbb{Q}(\sqrt{2})\)
中，\(\mathcal{O}_K=\mathbb{Z}[\sqrt2]\)。
写
\(x=a+b\sqrt2\)。
若 \(x\in \mathcal{O}_K^\vee\)，则必须同时满足

\[
\operatorname{Tr}(x)=2a\in\mathbb{Z},
\qquad
\operatorname{Tr}(x\sqrt2)=4b\in\mathbb{Z}.
\]

所以
\[
a\in \frac12\mathbb{Z},
\qquad
b\in \frac14\mathbb{Z},
\]
从而
\[
\begin{aligned}
\mathcal{O}_K^\vee
&=
\frac12\mathbb{Z}\oplus \frac{\sqrt2}{4}\mathbb{Z}.
\end{aligned}
\]

这个例子教给读者两件事：

1. dual object 一般不会等于原来的 \(\mathcal{O}_K\)；
2. codifferent 真的是“积分对偶条件”算出来的，不是拍脑袋附会出来的。

**图示**

```text
ring of integers O_K
    ->
canonical embedding sigma(O_K)
    ->
dual lattice sigma(O_K)^*
    <->
sigma(O_K^vee)
    ->
RLWE / ideal-lattice geometry
```

**与密码学关系**

- RLWE 的误差大小、dual ideal、pairing compatibility、quotient map 自然性，都和 codifferent 相关。
- 当 class number one 让 principal ideal / inverse ideal / codifferent 的表示都更显式时，RLWE 与 PLWE 的比较会少一层 ideal-class bookkeeping。
- 这也是为什么 Section 4 不把 codifferent 当枝节，而把它视作 reduction cleanliness 的一部分。

**常见误区**

- 不要把 codifferent 误解成“又一个随便挑的理想”；它是由 trace pairing 强制定义出来的。
- 不要把 dual lattice 和矩阵转置类比得太直接；这里的 dual 是由 canonical embedding 与 trace pairing 共同决定的。
- 不要以为 class number one 自动推出 RLWE 与 PLWE 完全等价；它清理的是 ideal-class obstruction，不是所有表示差异。

**在论文中的位置**

- Section 4.2 里 RLWE / MLWE reduction 的结构翻译会显式依赖这层 dual-language。
- 讲义 `10.3` 里 “Principal Ideals 与 Codifferent” 正是在把这页数学补丁重新翻译回密码学语言。

### 5.14 PIP、SGP、Ideal-LWE 与 Module-LWE：为什么攻击对象要先分家

**为什么提出它**

- 一旦读到 Section 4.1 的 quantum attack 讨论，很多人会把 “PIP 更透明” 直接误读成 “标准 MLWE 也更危险”。
- 这一步最容易错，不是因为 theorem 难，而是因为攻击输入对象已经换了。
- 所以在谈攻击前，必须先把 PIP、SGP、Ideal-LWE、Module-LWE 分清。

**最小定义**

- **PIP**：给定一个 principal ideal \(I\)，恢复某个生成元 \(\alpha\)，使得
  \[
  I=(\alpha).
  \]
- **SGP**：在知道 \(I\) 是 principal 的前提下，进一步找一个在 canonical embedding 意义下足够短的生成元。
- **Ideal-LWE**：困难对象更贴近单个 ideal 或 rank-one ideal lattice。
- **Module-LWE**：困难对象是
  \(R_q^d\)
  上的向量 / 子模 / noisy linear relation。

**直观理解**

- PIP 问的是：这个 ideal 背后有没有一个“单元素句柄”。
- SGP 问的是：即使句柄存在，你能不能找到一个几何上真正有用的短句柄。
- Ideal-LWE 的世界，很多结构可以压成“一个 ideal + 一个 generator”的语言。
- Module-LWE 的世界，基础对象已经变成高维 module，不再天然压成单个生成元问题。

**几何理解**

- ideal-lattice 更像一条由单个 ideal 控制的 rank-one 几何对象；
- module-lattice 更像若干份 ring geometry 拼成的高维自由模；
- 所以“恢复一个 generator”与“解一个 noisy module system”在攻击接口上不是同一类任务。

**小例子**

在
\(K=\mathbb{Q}(\sqrt2)\)
中，单位
\(u=1+\sqrt2\)
满足
\(u\in\mathcal{O}_K^\times\)。
因此同一个 principal ideal
\[
I=(2)
\]
也可以写成
\[
I=(2u^m)
\qquad
(m\in\mathbb{Z}).
\]

这个例子说明：

1. PIP 一旦输出某个生成元，例如 \(2\)，就算成功；
2. 但 SGP 还要在所有
   \(2u^m\)
   中找一个 embedding 意义下足够短的生成元；
3. 这也是为什么 “PIP 容易了” 并不自动等于 “ideal-lattice attack 已经完全闭环”。

而在 Module-LWE 里，secret 更像
\[
s=(s_1,\dots,s_d)\in R_q^d,
\]
这里根本没有一个天然的 “单个 ideal generator” 入口。

**图示**

```text
principal ideal I=(alpha)
    ->
many generators u alpha via units
    ->
PIP: recover some generator
SGP: recover a short generator

Module-LWE instance (A, b = As + e)
    ->
vector/module object
    ->
not canonically a single-generator problem
```

**与密码学关系**

- Biasse-Song 一类量子算法利用的是 cyclotomic ideal structure 与 generator recovery。
- 现代标准用的是 module-over-\(R_q\) 语言，这会打断“从单个 principal ideal 直接下手”的最自然接口。
- 所以 class number one 会让 ideal world 的结构更透明，但不会自动把 MLWE world 也降成 PIP。

**常见误区**

- 不要把 “PIP 在 principality 层面更透明” 误解成 “SGP 也自动变简单”。
- 不要把 RLWE、Ideal-LWE、MLWE 混成同一个问题；它们共享底层环，但攻击接口不同。
- 不要把 “module free” 误解成 “每个 module secret 都等价于某个单生成 ideal”。
- 不要把已知 ideal-lattice 量子攻击直接外推到 ML-KEM / ML-DSA；中间还差一条从 module instance 到 generator problem 的桥。

**在论文中的位置**

- Section 4.1：ideal-lattice quantum attack 线。
- Section 4.2：Module-LWE reduction 与 freeness 的结构条件。
- 讲义 `10.4`：解释为什么这条攻击线没有自动穿透到标准 MLWE。

### 5.15 Monogenicity、Polynomial Presentation、RLWE vs PLWE：为什么“同一个环”会有两种完全不同的噪声视角

**为什么提出它**

- 很多密码学读者第一次接触 RLWE/PLWE 时，会把
  \[
  R_q=\mathbb{Z}_q[x]/(f(x))
  \]
  当成唯一现实对象。
- 但在 reduction 语言里，更原生的对象往往是 number field \(K\)、整数环 \(\mathcal{O}_K\)、canonical embedding 与 dual ideal。
- 要理解 “为什么 class number one 让 RLWE/PLWE 比较更干净，但还不等于自动完全等价”，就必须把 polynomial presentation、monogenicity 与几何噪声视角分开。

**最小定义**

- **PLWE**：更接近“在某个固定多项式基下的系数向量上加噪声”的问题。
- **RLWE**：更接近“在整数环 / ideal lattice 的 canonical embedding 几何里加噪声”的问题。
- **monogenicity**：若存在某个元素 \(\theta\in\mathcal{O}_K\) 使得
  \[
  \mathcal{O}_K=\mathbb{Z}[\theta],
  \]
  就说这个整数环是 monogenic。

**直观理解**

- PLWE 先固定一个“写多项式系数”的坐标系，再谈误差分布。
- RLWE 先固定一个“把整个 ring 嵌进欧氏空间”的几何视角，再谈误差分布。
- monogenicity 的作用是告诉你：这个整数环能不能真的由一个单元素生成，从而写成某个 polynomial quotient 的形式。

也就是说：

```text
monogenic
    ->
there exists a natural single-generator presentation
    ->
coefficient view becomes available

but

coefficient view
    !=
canonical-embedding view
```

所以就算 \(\mathcal{O}_K=\mathbb{Z}[\theta]\) 成立，PLWE 与 RLWE 也还没自动变成同一个问题，因为噪声“长相”仍然可能不同。

**几何理解**

- PLWE 的噪声先在 coefficient basis 里被描述；
- RLWE 的噪声先在 canonical embedding 里被描述；
- 两者之间隔着一个 basis-change map。

如果这个变换在几何上严重扭曲，那么：

1. 在系数基里看起来“圆”的分布，到了 embedding 里可能变得很拉伸；
2. 在 embedding 里自然的高斯，回到系数基后也未必仍然自然。

这就是为什么 RLWE/PLWE 比较除了 ideal-class obstruction 之外，还要关心 embedding distortion。

**小例子**

在最熟悉的 cyclotomic 型环里，
\[
R=\mathbb{Z}[x]/(x^2+1)\cong \mathbb{Z}[i].
\]

这里
\(\mathbb{Z}[i]\)
确实是 monogenic，因为它就是
\(\mathbb{Z}[i]\)。
所以 coefficient presentation 完全存在。

但即使如此，你仍然要区分两件事：

1. 在 basis
   \(\{1,i\}\)
   下把元素写成系数向量 \((a,b)\)；
2. 在 canonical embedding 下把它看成复平面中的点
   \(a+bi\)。

对 \(\mathbb{Q}(i)\) 这种极其对称的小例子，这两种视角几乎贴合；但在更一般的 number field 里，二者之间的变换可能远不这么温和。

这个例子教给读者的不是“它们总是等价”，而是：

> 就连在 monogenic 场景里，也要先问清楚“噪声是按哪个坐标系定义的”。

**图示**

```text
number field K
    ->
ring of integers O_K
    ->            \
if monogenic       \ canonical embedding
    ->              \
polynomial presentation  coefficient basis
    \               /
     \             /
      RLWE vs PLWE comparison
```

**与密码学关系**

- class number one 主要清理的是 ideal-class / codifferent / freeness 这层结构包袱；
- monogenicity 决定你能不能把整数环自然地放进 polynomial presentation；
- embedding distortion 决定 coefficient-noise 与 geometric-noise 之间会不会差得太远。

因此更准确的链条是：

```text
class number one
    ->
ideal-class obstruction removed

monogenicity + controlled basis distortion
    ->
PLWE/RLWE comparison becomes technically cleaner
```

而不是

```text
class number one
    ->
automatic RLWE = PLWE
```

**常见误区**

- 不要把 “存在 polynomial quotient presentation” 误解成 “已经完全理解了整数环几何”。
- 不要把 monogenicity 和 class number one 混为一谈；前者是表示问题，后者是 ideal-class 问题。
- 不要把 RLWE 与 PLWE 的区别仅仅理解成“一个在环里、一个在多项式里”；真正的差别在噪声视角与 basis change。

**在论文中的位置**

- Section 4.2 讨论 RLWE / PLWE 比较时，这一层虽然没有被大篇幅展开，但正是 “为什么 class number one 只清理一部分障碍” 的背景。
- 讲义 `10.3` 的最后一段和 `10.5` 的对照表，都是在把这页基础补丁翻译回论文语境。

### 5.16 Different、Discriminant、Basis Distortion：为什么“同一个 ring”在不同基下可能几何差很多

**为什么提出它**

- `5.13` 里我们已经看到 codifferent 控制 dual lattice。
- `5.15` 里又提到 RLWE/PLWE 比较还要关心 basis distortion。
- 这两件事在更底层都和 different / discriminant 连在一起：它们衡量的正是整数环与其对偶、以及某组基在几何上有多“扭”。

**最小定义**

- codifferent 已定义为
  \[
  \begin{aligned}
  \mathcal{O}_K^\vee
  &=
  \{x\in K:\operatorname{Tr}_{K/\mathbb{Q}}(x\mathcal{O}_K)\subseteq \mathbb{Z}\}.
  \end{aligned}
  \]
- **different ideal** 可以记为
  \[
  \mathfrak{D}_{K/\mathbb{Q}} = (\mathcal{O}_K^\vee)^{-1}.
  \]
- 对一个整基
  \(\omega_1,\dots,\omega_n\)，
  **discriminant** 是
  \[
  \begin{aligned}
  \operatorname{disc}(K)
  &=
  \det\bigl(\operatorname{Tr}_{K/\mathbb{Q}}(\omega_i\omega_j)\bigr)_{1\le i,j\le n}.
  \end{aligned}
  \]

**直观理解**

- codifferent 说的是“谁和整数环做 trace pairing 还能回到整数”；
- different 则是它的反向尺度，记录“这个对偶对象离原整数环有多远”；
- discriminant 把这种尺度压成一个整数，粗略刻画这个整数环在几何上有多拥挤、多倾斜、多难坐标化。

所以可以把它们看成一条链：

```text
trace pairing
    ->
codifferent / different
    ->
discriminant
    ->
how distorted a basis/geometric model may become
```

**几何理解**

- canonical embedding 下，\(\mathcal{O}_K\) 是一个格。
- discriminant 的绝对值控制这个格的 covolume 量级。
- 如果你换到某个 coefficient basis 或 polynomial basis，basis-change matrix 的条件数和“长瘦程度”会受到这层几何影响。

教学上最值得记住的不是精确常数，而是：

> discriminant 大，往往意味着你更该警惕“看起来简单的系数基”与“自然的 embedding 几何”之间可能存在明显扭曲。

这正是 RLWE/PLWE 比较里 “embedding distortion” 这四个字背后的对象感。

**小例子**

对二次域
\(K=\mathbb{Q}(\sqrt2)\)，
取整基
\(\{1,\sqrt2\}\)。
其 trace matrix 是

\[
\begin{aligned}
\begin{pmatrix}
\operatorname{Tr}(1) & \operatorname{Tr}(\sqrt2) \\
\operatorname{Tr}(\sqrt2) & \operatorname{Tr}(2)
\end{pmatrix}
&=
\begin{pmatrix}
2 & 0 \\
0 & 4
\end{pmatrix},
\end{aligned}
\]

所以
\[
\operatorname{disc}(\mathbb{Q}(\sqrt2))=8.
\]

这个小例子说明两件事：

1. discriminant 真的是由 trace pairing 算出来的；
2. 它不是抽象标签，而是能落成一个显式整数的几何量。

#### 一个零跳步 notebook：把 coefficient basis 真的送到 canonical embedding 里

如果你想把 “basis distortion” 这四个字真正变成手感，最好的办法不是继续背定义，而是把矩阵直接写出来。

仍然取
\(K=\mathbb{Q}(\sqrt2)\)，
\(\mathcal{O}_K=\mathbb{Z}[\sqrt2]\)。
它的两个实嵌入是
\[
\sigma_1(a+b\sqrt2)=a+b\sqrt2,\qquad
\sigma_2(a+b\sqrt2)=a-b\sqrt2.
\]

对任意元素
\[
x=a+b\sqrt2,
\]
你可以同时用两种坐标记它：

1. coefficient basis 坐标：
   \[
   c(x)=
   \begin{pmatrix}
   a\\
   b
   \end{pmatrix};
   \]
2. canonical embedding 坐标：
   \[
   \sigma(x)=
   \begin{pmatrix}
   a+b\sqrt2\\
   a-b\sqrt2
   \end{pmatrix}.
   \]

两者之间正好差一个显式 basis-change matrix
\[
\sigma(x)=B\,c(x),\qquad
B=
\begin{pmatrix}
1 & \sqrt2\\
1 & -\sqrt2
\end{pmatrix}.
\]

这一步有三个值得立刻记住的事实。

**Fact 1：monogenicity 只保证矩阵存在，不保证它“几何上几乎等于单位阵”**

这里
\(\mathcal{O}_K=\mathbb{Z}[\sqrt2]\)
当然是 monogenic，所以 coefficient basis
\(\{1,\sqrt2\}\)
完全合法。
但合法不等于几何上无扭曲，因为
\[
B\neq I.
\]
也就是说，就算 polynomial presentation 天然存在，coefficient coordinates 和 embedding coordinates 也仍然是两套不同的几何语言。

**Fact 2：discriminant 已经在这个矩阵里现身了**

\[
\det(B)=-2\sqrt2,
\qquad
\det(B)^2=8=\operatorname{disc}(\mathbb{Q}(\sqrt2)).
\]

这正是前面 “discriminant 在测坐标几何复杂度” 的第一次落地：

> discriminant 不是悬空定义；它就藏在 coefficient basis 到 canonical embedding 的体积伸缩里。

**Fact 3：圆形噪声一过矩阵就会变成椭圆**

假设你在 coefficient basis 里先写一个各向同性误差
\[
e_{\mathrm{coeff}}\sim \text{mean }0,\quad \operatorname{Cov}(e_{\mathrm{coeff}})=s^2 I_2.
\]

送到 embedding 里之后，
\[
e_{\mathrm{emb}}=B\,e_{\mathrm{coeff}},
\]
于是协方差矩阵变成
\[
\begin{aligned}
\operatorname{Cov}(e_{\mathrm{emb}})
&=
s^2BB^\top \\
&=
s^2
\begin{pmatrix}
3 & -1\\
-1 & 3
\end{pmatrix}.
\end{aligned}
\]

这个矩阵的特征值是
\[
4s^2,\qquad 2s^2.
\]

所以原来在 coefficient basis 里“半径差不多一样”的圆形云团，到了 canonical embedding 里会被拉成一条主轴长度不同的椭圆。

```text
coefficient basis:   round cloud
        |
        | multiply by B
        v
embedding space:     tilted / stretched ellipse
```

这就是 RLWE / PLWE 比较里最该警惕的那件事：

> “同一个误差” 一旦换基，分布的长相就可能已经不是同一个几何对象。

反过来看，如果你先在 embedding 空间里放一个各向同性高斯，再拉回 coefficient basis，则会得到
\[
\begin{aligned}
\operatorname{Cov}(B^{-1}e_{\mathrm{emb}})
&=
s^2 B^{-1}(B^{-1})^\top \\
&=
s^2
\begin{pmatrix}
\tfrac12 & 0\\
0 & \tfrac14
\end{pmatrix}.
\end{aligned}
\]

在这个二次域 toy model 里，它恰好还没有出现坐标相关项，但两个方向的尺度已经不同了；到了更高维的 number field，通常连这种“仍然轴对齐”的好运都未必保留。

这页 notebook 想让你真正看到的结论只有一句：

> monogenicity 给你的是“可以写成多项式坐标”；  
> basis distortion 问的是“这些坐标和自然几何到底差多远”。

前者解决表示问题，后者解决几何比较问题；二者绝不是一回事。

#### 一个 bridge notebook：把这个 \(2\times 2\) 小矩阵重新接回 \(K_9\)、\(K_9^+\) 与 \(R_q\)

上面的 \(\mathbb{Q}(\sqrt2)\) 例子只是让你先摸到手感。真正和本文、以及 ML-KEM / ML-DSA 直接相接的，是下面这条更大的链：

\[
K_9=\mathbb{Q}(\zeta_{512}),
\qquad
K_9^+=\mathbb{Q}(\zeta_{512}+\zeta_{512}^{-1}),
\qquad
R_q=\mathbb{Z}_q[x]/(x^{256}+1).
\]

这一页的目标不是把 \(256\times256\) 的矩阵全写出来，而是让你看清：前面 toy model 里出现的 basis-change 现象，在现实 cyclotomic family 里到底对应什么对象。

**Step 1：为什么标准里的 coefficient view 本来就来自 full cyclotomic field**

记
\[
\zeta=\zeta_{512}.
\]
因为
\[
\Phi_{512}(x)=x^{256}+1,
\]
所以 power-of-two cyclotomic ring 有最熟悉的单生成表示：
\[
\mathcal{O}_{K_9}=\mathbb{Z}[\zeta]
\cong
\mathbb{Z}[x]/(x^{256}+1).
\]

模掉 \(q\) 之后，就得到
\[
\begin{aligned}
\mathcal{O}_{K_9}/q\mathcal{O}_{K_9}
&\cong
\mathbb{Z}_q[x]/(x^{256}+1) \\
&=
R_q.
\end{aligned}
\]

这说明标准实现里最熟悉的 coefficient vector
\[
(c_0,\dots,c_{255})
\]
本质上就是把
\[
f(\zeta)=c_0+c_1\zeta+\cdots+c_{255}\zeta^{255}
\]
写在 power basis
\[
\{1,\zeta,\zeta^2,\dots,\zeta^{255}\}
\]
下得到的坐标。

所以：

> ML-KEM / ML-DSA 里看到的 “polynomial coefficients” 不是凭空发明的工程表示；  
> 它们正是 full cyclotomic integer ring 的一个自然坐标系。

**Step 2：canonical embedding 在这条链上到底长什么样**

full cyclotomic field
\(K_9\)
的 Galois embeddings 由奇数指数给出：
\[
\begin{aligned}
\sigma_a(\zeta)=\zeta^a,
\qquad
a\in(\mathbb{Z}/512\mathbb{Z})^\times
&=
\{1,3,5,\dots,511\}.
\end{aligned}
\]

因此对
\[
f(\zeta)=\sum_{j=0}^{255}c_j\zeta^j,
\]
canonical embedding 的坐标就是
\[
\begin{aligned}
\sigma(f(\zeta))
&=
\bigl(f(\zeta^a)\bigr)_{a\in(\mathbb{Z}/512\mathbb{Z})^\times}.
\end{aligned}
\]

如果你把 coefficient vector 记成列向量
\[
c=
\begin{pmatrix}
c_0\\
\vdots\\
c_{255}
\end{pmatrix},
\]
那么这一步正是一个高维版的
\[
\sigma(f(\zeta)) = Bc,
\]
其中矩阵元素长成
\[
B_{a,j}=(\zeta^a)^j.
\]

和前面的
\(\mathbb{Q}(\sqrt2)\)
例子相比，差别只在维数：

- toy model 里是 \(2\times 2\) 的 basis-change；
- 这里是一个由 Galois conjugates 组织起来的 \(256\times256\) Fourier/Vandermonde 型 basis-change。

因此 “coefficient view vs canonical embedding view” 在现实参数里的真正含义，其实就是：

> 把一个多项式在 power basis 下的系数， simultaneously 改写成它在所有 Galois conjugates 上的取值。

**Step 3：为什么论文的主定理落在 \(K_9^+\)，而标准实现却常写在 \(K_9\)**

这正是很多读者第一次接触 Section 4 时最容易混淆的地方。

标准实现最常写的是 full cyclotomic side：
\[
R_q=\mathcal{O}_{K_9}/q\mathcal{O}_{K_9}.
\]

但论文的 class-number 结论落在 maximal real subfield：
\[
K_9^+=\mathbb{Q}(\zeta+\zeta^{-1}).
\]

二者并不矛盾，因为 complex conjugation 会把 embeddings 成对配起来：
\[
a \longleftrightarrow -a.
\]

在 real side，这一对会压成同一个实坐标：
\[
\begin{aligned}
\tau_a(\zeta+\zeta^{-1})
&=
\zeta^a+\zeta^{-a} \\
&=
2\cos\frac{2\pi a}{512}.
\end{aligned}
\]

所以如果说 full cyclotomic side 给你的是 `256` 个复方向，那么 maximal real subfield 给你的就是把共轭成对压缩后的 `128` 个实方向。

这页最关键的对象感是：

```text
full cyclotomic field K_9
    ->
power basis / polynomial coefficients / R_q presentation

maximal real subfield K_9^+
    ->
real embeddings paired by complex conjugation
    ->
class-number statement h_9^+ = 1
```

也就是说，本文主定理不是在一个和标准无关的平行宇宙里发生；它发生在标准所依赖的 full cyclotomic family 的 real half 上。

**Step 4：前面的 \(2\times 2\) toy model 到这里到底保留了什么**

前面
\(\mathbb{Q}(\sqrt2)\)
例子里，你亲眼看到了三件事：

1. coefficient basis 与 embedding basis 是两套不同坐标；
2. basis-change matrix 的行列式会碰到 discriminant；
3. 圆形噪声换基后会椭圆化。

到了 \(K_9\) 这边，三件事一个都没有消失，只是规模突然变大：

1. coefficient basis 变成
   \(\{1,\zeta,\dots,\zeta^{255}\}\)；
2. embedding coordinates 变成所有 odd conjugates 上的 simultaneously evaluation；
3. basis distortion 变成一个高维 Fourier/Vandermonde 型矩阵的几何问题。

所以不应该把 power-of-two cyclotomic 的 monogenic presentation 误读成：

> “既然已经有了 \(x^{256}+1\) 这个很自然的多项式，RLWE 与 PLWE 的几何就自动完全一致。”

真正正确的理解是：

> 你已经拥有了一个非常自然的 coefficient model；  
> 但 canonical embedding 仍然是另一套由 Galois conjugates 决定的几何坐标。

**Step 5：class number one 在这张图里到底清理了什么，又没有清理什么**

把这页和 `10.2`、`10.3` 连起来看，最稳妥的总结是：

class number one 会帮助你清理的是：

1. ideal-class twisting；
2. torsion-free module 的 freeness；
3. principal ideal / inverse ideal / codifferent 的显式性；
4. RLWE proof language 里那层额外的 ideal-class baggage。

但它不会自动清理的是：

1. coefficient basis 到 canonical embedding 的 basis-change 本身；
2. 噪声换基后的椭圆化或各向异性；
3. “有一个自然 polynomial quotient” 与 “几何视角完全等同” 之间的差距。

这也是为什么 Section 4.2 最值得教材化地读成：

> 论文无条件清理了结构层的 obstruction；  
> 至于 coefficient-noise 与 embedding-noise 之间还差多少几何控制，仍然是另一个问题。

如果你能把这句话和上面的 \(256\times256\) DFT/Vandermonde 型图像一起记住，那么前面的 toy model 就真正和本文的现实环 \(R_q\) 接上了。

**图示**

```text
integral basis
    ->
trace matrix
    ->
discriminant
    ->
geometric size / skew of the embedded lattice
    ->
basis distortion concerns in RLWE/PLWE comparison
```

**与密码学关系**

- 在 RLWE 里，误差自然写在 canonical embedding 的几何里；
- 在 PLWE 里，误差自然写在某个 polynomial coefficient basis 里；
- 不同基之间是否“温和”等价，取决于 basis-change 的几何质量，而 discriminant 正是最先提醒你这件事不会自动良性的对象之一。

这也是为什么：

1. class number one 只清理 ideal-class obstruction；
2. monogenicity 只保证 polynomial presentation 有意义；
3. 还必须额外关心 basis distortion，才能谈 RLWE/PLWE 是否真的干净对齐。

**常见误区**

- 不要把 discriminant 误解成“越小越安全、越大越不安全”的单调分数；它更像几何复杂度信号，而不是直接安全参数。
- 不要把 different / codifferent 仅仅当作互为逆的形式符号；它们恰好承载了 dual lattice 的尺度信息。
- 不要把 “有一个很自然的 polynomial basis” 误读成 “这个 basis 在几何上也天然接近 canonical embedding”。

**在论文中的位置**

- Proposition 2.2 显式给出
  \(\operatorname{disc}(K_k^+)=2^{(k-1)n-1}\)。
- Section 4 里虽然不把 discriminant 当主角反复展开，但 RLWE/PLWE / codifferent / embedding cleanliness 的整条翻译链都默认你知道它在测几何复杂度。

### 5.17 Short Generator、Unit Orbit、Log-Unit Lattice：为什么“找到一个生成元”之后攻击还没结束

**为什么提出它**

- `5.14` 已经把 PIP 和 SGP 分开了，但读者仍然可能会问：既然 ideal 是 principal，为什么恢复一个生成元还不够？
- 答案是：同一个 principal ideal 有无穷多个 generators，它们的几何长度可能差很多。
- 这正是 short-generator 问题和 log-unit lattice 出现的原因。

**最小定义**

若
\(I=(\alpha)\)
是 principal ideal，而
\(u\in\mathcal{O}_K^\times\)
是单位，则

\[
(\alpha)=(u\alpha).
\]

所以 principal ideal 的 generators 不是唯一的，而是沿着单位轨道

\[
\alpha,\ u\alpha,\ u^2\alpha,\dots
\]

不停变化。

**SGP** 要问的正是：在所有这些 generators 中，能不能找到一个在 canonical embedding 意义下足够短的那个。

**直观理解**

- PIP 只负责回答“有没有一个句柄”；
- SGP 才关心“句柄里哪个几何上最好用”。

这一区别的核心在于：

```text
principal ideal
    ->
many equivalent generators under units
    ->
geometry depends on which generator you choose
```

所以一旦单位群秩大于零，generator recovery 和 short-generator recovery 就不再是一回事。

**几何理解**

在 canonical embedding 下，乘以单位不会改变 ideal 本身，但会沿不同嵌入方向拉伸或压缩坐标。
把单位的绝对值取对数后，这种乘法会被线性化成一个 lattice，这就是 log-unit lattice 的直觉来源。

也就是说：

1. 单位乘法在原空间里是乘法作用；
2. 到 log 空间里，它变成加法平移；
3. 因而“在单位轨道里找更短 generator”会变成一个几何最近点/短向量风格的问题。

这就是为什么 ideal-lattice attack 论文常常一边谈 units，一边谈 lattice reduction。

**小例子**

在
\(K=\mathbb{Q}(\sqrt2)\)
里，单位
\(u=1+\sqrt2\)
满足
\[
\operatorname{N}(u)=-1.
\]

对 principal ideal
\[
I=(2),
\]
生成元
\(2\)
、
\(2u\)
、
\(2u^{-1}\)
都定义同一个 ideal。

但在两个实嵌入下，
\[
u\mapsto 1+\sqrt2\approx 2.414,
\qquad
u'\mapsto 1-\sqrt2\approx -0.414.
\]

所以乘以
\(u\)
会在一个方向放大，在另一个方向缩小。
这说明：

1. “同一个 ideal” 不等于 “所有 generators 几何长度差不多”；
2. short generator 的难点，正是要在单位轨道里找到真正平衡、真正短的代表。

**图示**

```text
principal ideal I
    ->
choose one generator alpha
    ->
multiply by units u
    ->
orbit {u alpha}
    ->
search for a geometrically short representative
```

**与密码学关系**

- 量子 PIP 算法如果只恢复到某个生成元，还不一定足以直接触发 ideal-lattice cryptanalysis。
- 很多攻击真正还需要一个 short generator，或者至少需要把 generator 推到足够有用的几何范围。
- log-unit lattice 正是组织这一步的标准语言，因为单位群在 log 空间里表现成一个真实的 lattice。

这也解释了为什么 class number one 是一把双刃剑：

1. 它让 principality 不再成问题；
2. 于是攻击者更可以把精力集中在 short-generator / unit-lattice 这一步；
3. 但这仍然属于 ideal world 的攻击接口，不会自动变成 module-LWE secret recovery。

**常见误区**

- 不要把 “PIP solved” 误解成 “SGP solved”。
- 不要把单位只当成“可逆元素的集合”；在攻击语境里，它们还是生成元轨道的来源。
- 不要把 log-unit lattice 想成论文里额外冒出来的技巧；它本质上是在把单位作用线性化。

**在论文中的位置**

- Section 4.1 的 ideal-lattice quantum attack 讨论，会把 units / short generators / principal structure 放在同一条攻击线里。
- 讲义 `10.4` 之所以强调 ideal world 和 module world 的区别，就是因为这一整套 SGP / unit-orbit 语言并没有自然迁移到标准 MLWE 实例上。

## 6. 全量符号表：不要让记号本身变成障碍

讲义里不会把 glossary 当附录摆设，而会在第一次重度进入 Section 2 前先给出。这里先放一版最小工作表。

| 符号 | 含义 | 在本文里负责什么 |
| --- | --- | --- |
| \(K_n\) | \( \mathbb{Q}(\zeta_{2^{n+2}}) \) | full cyclotomic field |
| \(K_n^+\) | \( \mathbb{Q}(\zeta_{2^{n+2}} + \zeta_{2^{n+2}}^{-1}) \) | maximal real subfield，真正关心的 class number 所在地 |
| \(\mathcal{O}_K\) | \(K\) 的整数环 | ideals、units、class group 的母对象 |
| \(\mathcal{O}_K^\vee\) | codifferent | canonical embedding 下的 dual lattice 母对象 |
| \(\mathfrak{D}_{K/\mathbb{Q}}\) | different ideal | codifferent 的逆，记录对偶尺度 |
| \(\operatorname{disc}(K)\) | discriminant | 测量整数环几何复杂度与基扭曲程度 |
| \(I^\vee\) | ideal \(I\) 的 dual ideal | 用 \(\mathcal{O}_K^\vee I^{-1}\) 表示对偶理想格 |
| monogenic | \(\mathcal{O}_K=\mathbb{Z}[\theta]\) 的性质 | 决定 polynomial presentation 是否自然可用 |
| log-unit lattice | 单位群取对数后的格 | 组织 short-generator / unit orbit 的几何搜索 |
| \(I_K\) | fractional ideals 群 | 构造 class group 的分子 |
| \(P_K\) | principal fractional ideals 子群 | 构造 class group 的分母 |
| \(\mathrm{Cl}(K)\) | \(I_K / P_K\) | 衡量哪些 ideals 不是 principal |
| \(h(K)\) | \(|\mathrm{Cl}(K)|\) | class number |
| \(h_k^+\) | \(|\mathrm{Cl}(K_k^+)|\) | 论文主目标 |
| \(G_k\) | \(\mathrm{Gal}(K_k^+ / \mathbb{Q})\) | Galois action 所在的循环 2-群 |
| \(\hat G_k\) | \(G_k\) 的 character group | 切 eigenspaces 用 |
| \(\chi\) | 一个 character | 标记某个 eigenspace |
| \(e_\chi\) | 对应于 \(\chi\) 的 idempotent | 从 class group 中投影出 \(\chi\)-部分 |
| \(C^+\) | cyclotomic unit group | 通过 Sinnott formula 接到 \(h_k^+\) |
| \(\xi_5\) | 特定 cyclotomic unit | Section 3.1 Wieferich test 的核心对象 |
| \(B_{1,\psi_j}\) | generalized Bernoulli number | Section 3.3 的可计算对象 |
| \(S_k\) | 有限可计算候选素数集 | Proposition 3.2 的产物 |
| \(R_q\) | \(\mathbb{Z}_q[x]/(x^{256}+1)\) | ML-KEM / ML-DSA 使用的现实环 |
| RLWE | Ring-LWE | Section 4.2 的 reduction 语境 |
| MLWE | Module-LWE | Section 4.1-4.2 的标准语境 |
| PIP | Principal Ideal Problem | ideal-lattice quantum attack 主线 |
| SGP | Short Generator Problem | PIP 平凡后剩下的 ideal-lattice 难题 |

再补两个容易混淆的记号：

- \(\zeta_m\) 是 primitive \(m\)-th root of unity，不是任意单位根。
- \(\Phi_n(x)\) 是 cyclotomic polynomial，定义 cyclotomic ring 时经常出现，但在这篇论文里更核心的是 field 和 class-group 侧，而不是 polynomial presentation 本身。

### 6.1 Operators 与函数记号

| 记号 | 读法 | 作用 |
| --- | --- | --- |
| \(\operatorname{Gal}(L/K)\) | Galois 群 | 跟踪对称性、characters 与 eigenspaces |
| \(\operatorname{ord}(\chi)\) | character 的阶 | Section 3.2 用它区分 low-order 与 full-order characters |
| \(\operatorname{cond}(\chi)\) | conductor | 决定 character 活在哪一层 cyclotomic data 上 |
| \(\operatorname{Tr}_{L/K}\) | trace | 把扩张中的元素加总回基域 |
| \(\operatorname{N}_{L/K}\) | field norm | 把扩张中的乘法信息压回基域 |
| \(\operatorname{N}(\mathfrak{a})\) | ideal norm | 记录商环大小 |
| \(\gcd(\cdot,\cdot)\) | greatest common divisor | 3.5 Optimizations 里用多项式 GCD 替换大整数分解 |
| \(e_\chi\) | idempotent projector | 从 class group 中抽取 \(\chi\)-块 |
| \([\mathcal{O}_K^\times : C^+]\) | 单位指数 | 由 Sinnott formula 接到 class number |

### 6.2 缩写与密码学翻译层

| 缩写 | 展开 | 这篇论文里为什么重要 |
| --- | --- | --- |
| Ideal-LWE | Ideal Learning With Errors | rank-one ideal / ideal-lattice 语境，用来和 MLWE 分家 |
| RLWE | Ring Learning With Errors | Section 4.2 讨论其 worst-case to average-case reduction 的无条件化 |
| PLWE | Polynomial Learning With Errors | class number one 时与 RLWE 的等价更干净 |
| MLWE | Module Learning With Errors | ML-KEM / ML-DSA 的直接问题模型 |
| PIP | Principal Ideal Problem | ideal-lattice quantum attack 的入口问题 |
| SGP | Short Generator Problem | 当 PIP 平凡后剩下的核心难题 |
| LUL | Log-Unit Lattice | 单位作用线性化后的攻击几何对象 |
| NTT | Number Theoretic Transform | Phase B 复杂度分析中提到可用于加速多项式算术 |
| ML-KEM | NIST FIPS 203 标准 KEM | 使用 \(R_q=\mathbb{Z}_q[x]/(x^{256}+1)\) |
| ML-DSA | NIST FIPS 204 标准签名 | 同样工作在 module-over-\(R_q\) 语境 |

### 6.3 读论文时最常见的三种“翻译错位”

1. 把 \(K_k\) 与 \(K_k^+\) 混为一谈。前者是 full cyclotomic field，后者是本文主结论所在的 maximal real subfield。
2. 把 “\(\ell \mid h_k^+\)” 理解成纯粹的整数整除，而忘记它等价于 class-group \(\ell\)-torsion 非平凡。
3. 把 \(R_q = \mathbb{Z}_q[x]/(x^{256}+1)\) 当成唯一现实对象，而忽略它背后是 \(\mathcal{O}_{K_9}\) 的 quotient 这一数论来源。

## 7. 逐章精读 Section 2：为什么作者先铺这些背景

这一节最容易被初读者低估，因为它看起来像“背景综述”。但对这篇论文来说，Section 2 其实是把后面三段主证明分别装上接口：

- `2.2` 负责把目标整数 \(h_k^+\) 重新解释成 class-group torsion 问题。
- `2.5` 负责把整个 class group 切成 characters 管得住的 eigenspaces。
- `2.6` 负责给 `3.1` 的筛法提供 \(\xi_5\) 和 unit-index 出口。
- `2.7` 负责给 `3.3` 的 finite computation 提供 Bernoulli-norm 入口。
- `2.8` 负责给 `3.2` 的 tower argument 提供 \(N=1+\sigma^{n/2}\) 这个杀手算子。

如果你想把原论文读成“每一步为什么非做不可”，最好的办法不是背定义，而是对每个 subsection 都问五个问题：

1. 作者在这一小节要固定什么对象？
2. 这个对象在 Section 3 哪个定理里第一次真正发力？
3. 如果删掉这一小节，后文哪一步会失去语言或失去可计算性？
4. 这里最容易误解的地方是什么？
5. 这小节究竟是在为“筛素数”“杀 eigenspace”，还是为“做有限计算”铺路？

### 7.1 `2.1 Number fields and rings of integers`

**作者在解决什么**

- 作者先不谈 class number，而是先把“我们究竟在哪个环里工作”说清楚。
- Definition 2.1 固定 number field 与 ring of integers。
- Definition 2.2 说明什么叫 totally real。

**关键定义与公式**

- number field 是 \(\mathbb{Q}\) 的有限扩张。
- 整数环 \(\mathcal{O}_K\) 是扩张里“最像整数”的元素集合。
- totally real 的意思是：所有嵌入都落在 \(\mathbb{R}\) 里，而不是出现成对复嵌入。

**为什么这一节非要先来**

- 因为后文所有对象都住在 \(\mathcal{O}_{K_k^+}\) 上：ideal、unit、class group、residue field、prime ideal、norm map、cyclotomic unit 都没有离开它。
- 论文最终关心的是 \(K_k^+\)，不是 full cyclotomic field \(K_k\) 本身的任意算术量。
- “totally real” 也不是修辞词。它会影响 unit rank、Minkowski 估计、以及为什么 \(K_k^+\) 正好与 RLWE/MLWE 后面的真实数嵌入结构相接。

**一眼直觉**

- 如果把 \(\mathbb{Z}\) 看成 \(\mathbb{Q}\) 里的整数，那么 \(\mathcal{O}_{K_k^+}\) 就是 \(K_k^+\) 里的“整数世界”。
- 这一步是在告诉你：后文不是在某个随手写下的 quotient ring 里做技巧，而是在一个有完整 arithmetic structure 的母对象里做事。

**小例子**

- 对 \(K=\mathbb{Q}(i)\)，有 \(\mathcal{O}_K=\mathbb{Z}[i]\)。
- 对本文更贴近的 toy field，\(K_5^+=\mathbb{Q}(\zeta_{32}+\zeta_{32}^{-1})\) 是一个次数 \(8\) 的 totally real field；前面 Sage 例子已经验证它的 class number 就是 \(1\)。

**隐藏前置**

- integral element 是什么。
- 为什么 \(\mathcal{O}_K\) 不是任意 \(\mathbb{Z}[\alpha]\) 都能代替。
- embeddings 与 field degree 的关系。

**在 Section 3 的用法**

- `3.1` 里 prime \(\mathfrak l \mid \ell\) 是 \(\mathcal{O}_{K_k^+}\) 里的 prime ideals。
- `3.2` 里 norm of ideals 是从 \(\mathcal{O}_{K_k^+}\) 往 \(\mathcal{O}_{K_{k-1}^+}\) 送。
- `4.2` 里 \(R_q=\mathbb{Z}_q[x]/(x^{256}+1)\) 被重新解释成 \(\mathcal{O}_{K_9}/q\mathcal{O}_{K_9}\) 的现实投影。

### 7.2 `2.2 Ideals, fractional ideals, and the class group`

**作者在解决什么**

- 这一小节第一次把 Weber conjecture 的目标写成可操作的 algebraic statement。
- Definition 2.3 给 fractional ideals。
- Definition 2.4 与 Eq. (1) 给出
  $$
  \mathrm{Cl}(K)=I(K)/P(K).
  $$
- Definition 2.5 给 ideal norm。
- Eq. (2) 给 Minkowski bound。
- Definition 2.6 与 Eq. (3) 给 \(\mathrm{Cl}(K)[\ell^\infty]\) 这样的 \(\ell\)-primary/torsion 记号。

**为什么这一节是全篇第一根承重梁**

- 论文最终要证 \(h_k^+=1\)。
- 这句话的真正意思不是“某个解析公式等于 1”，而是“\(\mathrm{Cl}(K_k^+)\) 是平凡群”。
- 一旦这样翻译，所有后续步骤就都变成“证明没有 prime \(\ell\) 能制造非平凡 \(\ell\)-torsion”。

**一眼直觉**

- ideal 是在唯一分解失效时，替元素分解补位的对象。
- principal ideals 是“由单个元素生成”的那些 ideal。
- class group 记录的正是：有多少 ideal 无法被元素单独解释。
- \(h(K)=1\) 的意思就是：每个 ideal 最终都能落回 principal ideal。

**为什么 Minkowski 会突然出现**

- Eq. (2) 告诉你：每个 ideal class 里都能找到 norm 不太大的代表。
- 这解释了“为什么 class group 是 finite”的几何来源。
- 教学上最重要的不是记住常数，而是理解它在本篇里的叙事位置：
  没有 finite-ness，后面“只剩有限多个 prime divisors 需要排除”的气氛就毫无根据。

**隐藏前置**

- 为什么 fractional ideals 才能形成群，而 integral ideals 只能形成幺半群。
- 为什么 \(\ell \mid h_k^+\) 等价于 \(\mathrm{Cl}(K_k^+)[\ell]\neq 0\)。

**在 Section 3 的用法**

- `3.1` 的结论 \(\ell \nmid h_k^+\) 实际上是在说 \(\ell\)-torsion 根本不存在。
- `3.2` 直接操作的是 \(\mathrm{Cl}(K_k^+)[\ell]^{e_\chi}\)。
- `3.4` 最终证明 \(h_k^+=1\) 的方式就是“把所有可能的 prime divisors 全部耗尽”。

#### 一个零跳步 notebook：为什么这里一定要先换到 fractional ideal → class group → \ell-torsion 的语言

这一小节看起来像一串定义，但对整篇论文来说，它其实是在做一次非常关键的“问题重编码”：
把
\(h_k^+\)
这个整数问题，翻译成一个可以逐个 prime、逐个 eigenspace 拆解的群论问题。

**Step 1：为什么只讲 integral ideals 不够，必须放宽到 fractional ideals**

如果只看 \(\mathcal{O}_K\) 里的 integral ideals，那么它们在乘法下是封闭的，但一般没有逆元。

最简单的例子甚至就在 \(\mathbb{Z}\) 里：
\((2)\)
当然是一个 integral ideal，但它的“乘法逆元”应该是
\[
(2)^{-1}=\frac{1}{2}\mathbb{Z},
\]
这已经不再包含在 \(\mathbb{Z}\) 里，因此不是 integral ideal。

这说明了一个结构事实：

1. integral ideals 只形成幺半群；
2. 若你想讨论 “ideal class” 这种带有可逆性的对象，就必须允许分母出现；
3. 这正是 fractional ideals 的角色。

更正式一点，对一个非零 ideal \(\mathfrak a\)，它的逆可写成
\[
\mathfrak a^{-1}=\{x\in K: x\mathfrak a\subseteq \mathcal{O}_K\}.
\]
在 number field 的整数环 \(\mathcal{O}_K\) 中，非零 fractional ideals 都是可逆的，于是它们组成一个交换群 \(I(K)\)。

所以 Definition 2.3 不是“把 ideal 稍微推广一下”而已；它是在把后文真正要操作的对象，从一个不能取逆的幺半群，升级成一个可做 quotient 的群。

**Step 2：为什么商掉的是 principal ideals，而不是别的子群**

一旦进入 \(I(K)\)，下一步要问的就是：哪些 ideals 在本质上“不该算不同”？

论文的答案是：如果两个 fractional ideals 只差一个 field element 的缩放，那么它们应该视为同一类。也就是
\[
\mathfrak a \sim \mathfrak b
\qquad\Longleftrightarrow\qquad
\mathfrak a=(\alpha)\mathfrak b
\quad\text{for some }\alpha\in K^\times.
\]

这里的
\((\alpha)\)
就是 principal fractional ideal。它表示的不是新的算术障碍，而只是“拿一个元素整体缩放了一下”。

因此
\[
\mathrm{Cl}(K)=I(K)/P(K)
\]
测量的正是：

- 哪些 ideal 可以被单个元素解释；
- 哪些 ideal 即使允许整体缩放，仍然无法写成 principal。

换句话说，class group 不是随手定义出来的 quotient；它恰好是“principal ideal 失败到什么程度”的障碍群。

于是
\(h(K)=1\)
的真正含义就变成：

> 每个 nonzero fractional ideal 都和某个 principal ideal 属于同一类。

特别地，这等价于每个 integral ideal 也是 principal。论文后面要证明的，从头到尾都是这件事，而不是某个孤立的整数恒等式。

**Step 3：为什么 \(\ell \mid h_k^+\) 等价于 \(\mathrm{Cl}(K_k^+)[\ell]\neq 0\)**

Minkowski bound 的作用之一，是保证
\(\mathrm{Cl}(K_k^+)\)
是一个有限阿贝尔群。记
\[
A=\mathrm{Cl}(K_k^+),\qquad h_k^+=|A|.
\]

一旦进入“有限阿贝尔群”语境，整除问题就立刻变成 torsion 问题。由 Cauchy 定理：

\[
\ell \mid |A|
\qquad\Longleftrightarrow\qquad
A \text{ 中存在阶为 }\ell\text{ 的元素}.
\]

而“存在阶为 \(\ell\) 的元素”正等价于
\[
A[\ell]=\{a\in A:\ell a=0\}\neq 0.
\]

所以
\[
\ell \mid h_k^+
\qquad\Longleftrightarrow\qquad
\mathrm{Cl}(K_k^+)[\ell]\neq 0.
\]

若用 Definition 2.6 的 primary-part 记号来讲，这也等价于
\(\mathrm{Cl}(K_k^+)[\ell^\infty]\neq 0\)。
但教学上最重要的是先抓住第一层翻译：

```text
prime divides class number
    <->
class group has surviving ell-torsion
```

这一步一旦看清，后文所有 “固定一个 odd prime \(\ell\)” 的动作就都不再神秘。

**Step 4：为什么这正好是 Section 3 的 prime-divisor exhaustion 入口**

现在再回看论文主目标
\(h_k^+=1\)：
它不再只是“证明一个整数等于 1”，而是

1. 先利用有限性，说明若 \(h_k^+>1\)，就必有某个 prime \(\ell\) 整除它；
2. 再把“\(\ell \mid h_k^+\)”翻译成“\(\mathrm{Cl}(K_k^+)[\ell]\neq 0\)”；
3. 接着把这一团 \(\ell\)-torsion 用 `2.5` 的 eigenspace decomposition 切成 \(\chi\)-块；
4. 最后在 Section 3 里逐步证明这些块不可能存活。

对这篇论文，真正的执行形态其实就是下面这张工作流：

```text
fix a candidate prime ell
    ->
assume Cl(K_k^+)[ell] is nonzero
    ->
push it into chi-eigenspaces
    ->
kill low-order characters
    ->
force full-order survivors into explicit Bernoulli divisibility
    ->
use residue tests / finite candidate checks to eliminate ell
```

所以 `2.2` 的意义不是“先讲一些理想论背景”；它是在给 Section 3 设定打击目标。
没有
\[
\mathrm{Cl}(K)=I(K)/P(K)
\]
这层重编码，后面的 “prime elimination”“character elimination”“candidate set exhaustion” 都没有一个统一的着力点。

### 7.3 `2.3 Galois theory of cyclotomic fields`

**作者在解决什么**

- 这一节固定后文所有 group action 的底层群。
- Proposition 2.1 给出
  \(\Gamma_k=\operatorname{Gal}(K_k/\mathbb{Q})\cong(\mathbb{Z}/2^k\mathbb{Z})^\times\)。
- Eq. (4) 把 maximal real subfield 的 Galois 群写成
  $$
  G_k=\operatorname{Gal}(K_k^+/\mathbb{Q})\cong \Gamma_k/\langle\sigma_{-1}\rangle \cong \mathbb{Z}/n\mathbb{Z},
  $$
  其中 \(n=2^{k-2}\)。
- 作者选 \(\sigma=\sigma_5\) 作为生成元。
- Proposition 2.2 与 Eq. (5) 给出
  $$
  \operatorname{disc}(K_k^+) = 2^{(k-1)n-1}.
  $$

**为什么这一节决定了后面 proof skeleton**

- Section 3 里 character order、full-order character、Frobenius order、norm element \(1+\sigma^{n/2}\)，全部依赖“\(G_k\) 是一个循环 2-群”这一事实。
- 如果 \(G_k\) 不是这么干净，后面的 eigenspace 分解与 low-order/full-order 二分法都不会这么锋利。

**一眼直觉**

- 这一步是在告诉你：虽然 field 很大，但它的对称群 surprisingly simple。
- “simple” 的具体含义不是对象小，而是群结构几乎就是一个单生成的二进塔。

**小例子**

- 当 \(k=9\) 时，\(n=2^{7}=128\)，所以 \(G_9\cong \mathbb{Z}/128\mathbb{Z}\)。
- 这解释了为什么 `3.2` 里“full-order character”就是阶恰为 \(128\) 的 character。

**隐藏前置**

- complex conjugation 为什么对应 \(\sigma_{-1}\)。
- fixed field 与 quotient Galois group 的关系。
- discriminant 为什么能粗略反映域的“算术复杂度”。

**在 Section 3 的用法**

- `3.1` 里 Frobenius element 的阶，就是 prime \(\ell\) 在 \(G_k\) 中的 order。
- `3.2` 里 \(N=1+\sigma^{n/2}\) 只在 cyclic 2-group 的语境下这么自然。
- `3.3` 里 full-order characters 的计数，归根结底来自 \(G_k\) 的循环结构。

#### 一个零跳步 notebook：为什么 \(G_k\) 会变成由 \(\sigma_5\) 生成的循环 2-群，以及这正好喂给 `3.1` 的 Frobenius 语言

第一次读 Eq. (4) 时，很多读者都会有一种“作者是不是突然跳步了”的感觉：

> 为什么一个看起来很大的 cyclotomic field，最后会把所有 Galois action 都压成一个由 \(\sigma_5\) 控制的循环群？

而后面的 `3.1`、`3.2` 又进一步默认你已经接受了一件更强的事：

> 每个 odd prime \(\ell\) 在这座 \(2\)-power tower 里，都有一个明确的 “Frobenius 位置”。

这页 notebook 的作用，就是把这两句话接起来。

**Step 1：为什么 full cyclotomic field 的自同构天然就是幂次映射**

固定一个 primitive \(2^k\)-th root of unity
\(\zeta_{2^k}\)。
任何 \(\mathbb{Q}\)-自同构都由它对
\(\zeta_{2^k}\)
的作用唯一决定；而它必须把 primitive root 送到另一个 primitive root，所以只能是

\[
\sigma_a(\zeta_{2^k})=\zeta_{2^k}^{\,a},
\qquad
a\in (\mathbb{Z}/2^k\mathbb{Z})^\times.
\]

这里
\((\mathbb{Z}/2^k\mathbb{Z})^\times\)
正是模 \(2^k\) 的 odd residue classes。
因此

\[
\Gamma_k=\operatorname{Gal}(K_k/\mathbb{Q})
\cong
(\mathbb{Z}/2^k\mathbb{Z})^\times.
\]

这一步最该记住的不是同构本身，而是它给了你一个非常具体的 mental model：

```text
Galois action
    =
"raise zeta to an odd power"
```

后文一切关于 Frobenius、character、order 的语言，都是在这个模型上运转的。

**Step 2：为什么 complex conjugation 恰好就是 \(\sigma_{-1}\)，以及为什么要对它取 quotient**

因为
\[
\overline{\zeta_{2^k}}=\zeta_{2^k}^{-1},
\]
所以 complex conjugation 在 \(\Gamma_k\) 中对应的正是
\(\sigma_{-1}\)。

而 maximal real subfield
\(K_k^+\)
就是被 complex conjugation 固定的那个子域。
由 Galois correspondence，

\[
\begin{aligned}
G_k
&=
\operatorname{Gal}(K_k^+/\mathbb{Q}) \\
&\cong
\Gamma_k/\langle \sigma_{-1}\rangle.
\end{aligned}
\]

这条 quotient 有一个非常直观的含义：

1. 在 full field \(K_k\) 里，\(a\) 与 \(-a\) 是不同的 automorphisms；
2. 一旦降到 real side，它们的差别正好被 complex conjugation 抹掉；
3. 所以后面在 \(K_k^+\) 上谈 Frobenius 时，真正重要的是 \(\pm a\) 这一类，而不是带符号的单个 \(a\)。

这也解释了为什么后文很多 real-side 结论最终只记住 “\(\ell\) 模 \(2\)-power 的位置”，而不再敏感于正负号。

**Step 3：为什么 quotient 之后会变成一个循环 2-群，以及作者为什么偏偏选 5**

对 \(k\ge 3\)，模 \(2^k\) 的 odd units 有一个非常整齐的分解：

\[
(\mathbb{Z}/2^k\mathbb{Z})^\times
\cong
\{\pm1\}\times \langle 5\rangle.
\]

这里真正承重的是：
\(5\)
在模 \(2^k\) 意义下有尽可能大的阶。更准确地说，

\[
\operatorname{ord}_{2^k}(5)=2^{k-2}.
\]

一个紧凑证明可以记成

\[
v_2(5^{2^t}-1)=t+2,
\]

它可由 repeated squaring 或 LTE 看出。于是第一次出现
\(5^{2^t}\equiv1\pmod{2^k}\)
的时刻，正好是
\(t=k-2\)。

所以在除去 \(\{\pm1\}\) 这个 sign part 之后，\(5\) 的像仍然保有精确的阶
\(2^{k-2}\)。
这就得到

\[
\begin{aligned}
G_k
&=
\langle \sigma_5\rangle \\
&\cong
\mathbb{Z}/2^{k-2}\mathbb{Z}.
\end{aligned}
\]

也就是说，作者选
\(\sigma=\sigma_5\)
不是审美问题，而是因为它确实给出了整个 real-side Galois group 的一个 generator。

**小例子**

当 \(k=5\) 时，模 \(32\) 的 \(5\) 的幂次链是

\[
5,\ 25,\ 29,\ 17,\ 21,\ 9,\ 13,\ 1 \pmod{32}.
\]

它走满了长度 \(8=2^{5-2}\) 的整个循环，这就是
\(\operatorname{ord}_{32}(5)=8\)
的最直观版本。

**Step 4：为什么每个 odd prime \(\ell\) 都会落到这个循环群里的某个位置**

只要
\(\ell\neq 2\)，
它就不 ramify 在这个 \(2\)-power cyclotomic extension 里。
于是对 full field \(K_k\)，Frobenius 元素满足

\[
\operatorname{Frob}_\ell=\sigma_\ell,
\]

因为在 residue field 里，“把 root 提到 \(\ell\)-次方” 正是 Frobenius 的定义动作。

降到 real subfield 之后，由于我们已经把
\(\sigma_{-1}\)
quotient 掉，所以真正留下的是
\(\sigma_\ell\)
在
\(G_k\)
中的像；等价地说，是 \(\pm\ell\) 在那个循环群里的位置。

因此对每个 odd prime \(\ell\)，都存在唯一的
\(t\in\{0,\dots,n-1\}\)
使得

\[
\operatorname{Frob}_\ell=\sigma^t,
\qquad
\sigma=\sigma_5,\quad n=2^{k-2}.
\]

这就是后文 repeatedly 使用的 “prime \(\ell\) 在 tower 里的 Frobenius 位置”。

`Step 5：为什么这个位置的阶，正好就是 `3.1` 要用的 residue degree`

对任意
\(\mathfrak l\mid \ell\)
在 \(K_k^+\) 上方的 prime ideal，Galois number theory 告诉你：
unramified prime 的 Frobenius 元素的阶，正好等于 residue degree

\[
f=[\kappa(\mathfrak l):\mathbb{F}_\ell].
\]

也就是说，

\[
f=\operatorname{ord}_{G_k}(\operatorname{Frob}_\ell).
\]

这就是 `3.1` 背后最关键的 group-theoretic bridge：

```text
odd prime ell
    ->
Frobenius class in G_k
    ->
order of that class = residue degree f
    ->
small f makes the residue-test exponent explicit
```

所以后面当论文施加某个 congruence condition on \(\ell\) 时，它本质上不是在玩整数同余技巧，而是在逼迫
\(\operatorname{Frob}_\ell\)
落进
\(G_k\)
里的某个小阶子群。
一旦阶小，\(f\) 就小，局部测试指数就会塌成
\((\ell^f-1)/2^{k-1}\)
这类可计算的量。

教学上把 `7.3` 压成一句话，就是：

> Section 2 先把所有 odd primes 放进一个由 \(\sigma_5\) 编号的循环 2-塔里；Section 3 再利用 prime 所处的位置，决定它能否威胁 \(h_k^+\)。

### 7.4 `2.4 Character theory`

**作者在解决什么**

- 这一节把“群的作用”转换成“可逐个频率分析的 characters”。
- Definition 2.7 定义 character。
- Definition 2.8 定义 character 的阶与 full order。
- Definition 2.9 定义 even/odd。
- Lemma 2.1 给出 full-order characters 的数量与共轭配对数。

**为什么作者现在就讲 characters**

- 因为后文不是直接和 \(G_k\) 斗，而是把 \(G_k\) 在 class group 上的作用分频。
- 没有这一层语言，`3.2` 中“先消去 low-order characters，再只剩 full-order characters”这句话根本无法表达。

**一眼直觉**

- 把 \(G_k\cong\mathbb{Z}/n\mathbb{Z}\) 看成一个旋转群。
- character 就像“测某个向量在第几个频率上振动”的探头。
- full-order character 对应最细的频率；low-order character 对应粗糙的低频模式。

**小例子**

- 若 \(G_9=\langle \sigma \rangle\cong\mathbb{Z}/128\mathbb{Z}\)，则每个 character 由 \(\chi(\sigma)\) 决定。
- 当 \(\chi(\sigma)\) 是 primitive \(128\)-th root of unity 时，它就是 full-order character。
- 当 \(\chi(\sigma)\) 只落在 \(64\)-th roots 或更低时，它就是 `3.2` 会被整批杀掉的对象。

**隐藏前置**

- dual group \(\hat G_k\) 的概念。
- even/odd 最好不要和函数奇偶性混淆；这里说的是 character 在 \(-1\) 或 complex conjugation 上的行为。

**在 Section 3 的用法**

- Lemma 2.1 的 “共轭 pair 数至多 \(n/4\)” 会直接进入 `3.3` 的 candidate counting。
- full-order 的概念会成为 Corollary 3.1 之后唯一剩下的风险层。

#### 一个零跳步 notebook：full-order、even/odd、conjugate pairs 为什么正好长成后文要的样子

如果你第一次读到 `Lemma 2.1`，很容易只记住一句结果：

> full-order characters 有若干个，共轭 pair 大约只有一半。

但后文 `3.2`、`3.3`、`3.5` 真正需要的不是这个口号，而是你能自己把这件事重新算出来。

**Step 1：cyclic 群上的 character 其实只由一个值决定**

既然
\(G_k=\langle \sigma\rangle\cong \mathbb{Z}/n\mathbb{Z}\)，
其中
\(n=2^{k-2}\)，
那任何 character \(\chi\) 都由
\(\chi(\sigma)\)
唯一决定。

而 \(\sigma^n=1\)，所以
\(\chi(\sigma)^n=1\)。
这意味着：

$$
\chi(\sigma)=\zeta_n^j
\qquad
(0\le j<n)
$$

对某个整数 \(j\)。

因此 character 可以直接编号成
\(\chi_j\)，其中
\(\chi_j(\sigma)=\zeta_n^j\)。

**Step 2：为什么 full-order 等价于 “j 是奇数”**

\(\chi_j\) 的阶就是 \(\zeta_n^j\) 的阶，也就是

$$
\operatorname{ord}(\chi_j)=\frac{n}{\gcd(j,n)}.
$$

而这里
\(n=2^{k-2}\)
是纯 \(2\)-幂，所以：

1. 若 \(j\) 是偶数，则 \(\gcd(j,n)\ge2\)，于是
   \(\operatorname{ord}(\chi_j)\le n/2\)；
2. 若 \(j\) 是奇数，则 \(\gcd(j,n)=1\)，于是
   \(\operatorname{ord}(\chi_j)=n\)。

所以对本论文的 cyclic \(2\)-group 来说，

$$
\chi_j \text{ 是 full-order }
\qquad\Longleftrightarrow\qquad
j \text{ 是奇数}.
$$

这条等价在教学上非常值钱，因为它把一个看似抽象的 “full-order” 直接变成了一个最朴素的奇偶判别。

**Step 3：为什么 full-order characters 恰好有 n/2 个**

在 \(0,1,\dots,n-1\) 里，恰好一半是奇数，一半是偶数。

因此 full-order characters 的个数就是

$$
\varphi(n)=\varphi(2^{k-2})=2^{k-3}=n/2.
$$

这和上面 “\(j\) 必须是奇数” 的直接计数完全一致。

也就是说，后文所谓 “只剩 full-order layer” 时，表面上还剩下的频道数其实仍然不小；这也是为什么后面必须继续做 orbit reduction，而不是天真地逐个 character 分开跑。

**Step 4：为什么共轭会把 n/2 个 characters 压成 n/4 对**

在 character 值域里，复共轭把
\(\zeta_n^j\)
送到
\(\zeta_n^{-j}=\zeta_n^{n-j}\)。
所以
\(\chi_j\)
的共轭 character 正是
\(\chi_{n-j}\)。

于是：

1. \(\chi_j\) 与 \(\chi_{n-j}\) 有相同的阶；
2. 若 \(j\) 是奇数，则 \(n-j\) 也是奇数，所以 full-order character 在共轭下封闭；
3. 当 \(n\ge4\) 时，不会有
   \(j=n-j\)
   的 full-order 奇数解，因此这些 full-order characters 都两两配成 distinct pairs。

因此 full-order characters 的共轭 pair 数就是

$$
\frac{n/2}{2}=n/4.
$$

这就是 `Lemma 2.1` 和后面 `3.5` repeatedly 使用的计数来源。

**Step 5：even/odd 到底是在说什么，为什么别和 full-order 混为一谈**

这也是最容易混乱的一点。

- full-order 说的是 character 的**阶有多大**；
- even/odd 说的是 character 在 \(-1\) 或 complex conjugation 上的**符号行为**。

所以这两件事是不同维度的信息：

```text
full-order
    = frequency is as fine as possible

even / odd
    = sign behavior under -1 / conjugation
```

在本文的 real-subfield class-group 语境里，真正住在
\(G_k=\Gamma_k/\langle\sigma_{-1}\rangle\)
上的那一层，天然已经把 complex conjugation quotient 掉了，所以后面承载 class-group eigenspace 的 \(\chi\) 本质上是 even side 的角色。

而到了 `3.3`，作者又需要一个 **primitive odd Dirichlet character**
\(\psi\)
去喂 generalized Bernoulli numbers，这才会出现

$$
\psi=\chi^{-1}\omega_\ell
$$

这样的扭曲。也就是说：

1. `3.2` / Corollary 3.1 里真正筛出来的是 full-order \(\chi\)；
2. `3.3` 里真正拿去构造 \(B_{1,\psi}\) 的，是由这个 \(\chi\) 再扭出来的 odd primitive \(\psi\)。

如果把这两层混成一个对象，就会误以为 “full-order” 和 “odd” 是同一个条件；其实不是。

**Step 6：为什么这一页会直接喂给后面的算法**

一旦你把上面五步吃透，后文几处最容易显得突兀的地方会立刻顺下来：

1. `3.2` 为什么只需要消掉 order dividing \(2^{k-3}\) 的 characters；
2. `3.3` 为什么 Phase A 表面要看的是 full-order representatives；
3. `3.5` 为什么 orbit reduction 后只剩 \(n/4\) 对 conjugate pairs；
4. `9.7` 里为什么对 `k=9` 恰好会扫到 `64` 个 primitive odd full-order characters，也就是 `32` 对共轭代表。

把这页压成一句话，就是：

```text
characters on G_k are indexed by j mod n
    ->
full-order means j odd
    ->
there are n/2 of them
    ->
complex conjugation pairs j with n-j
    ->
only n/4 conjugate pairs need separate attention
```

这就是 `2.4` 真正给后文留下的 computational shape。

### 7.5 `2.5 Eigenspace decomposition of the class group`

**作者在解决什么**

- 这一小节把 class group 从一个整体切成一堆 \(\chi\)-块。
- Definition 2.10 给出 idempotent projector \(e_\chi\)。
- Proposition 2.3 给出四条核心代数关系：\(e_\chi^2=e_\chi\)、\(e_\chi e_\psi=0\)、\(\sum e_\chi=1\)、以及 \(g\cdot e_\chi=\chi(g)e_\chi\)。
- Eq. (7) 给出
  $$
  \mathrm{Cl}(K_k^+)[\ell] = \bigoplus_{\chi\in\hat G_k}\mathrm{Cl}(K_k^+)[\ell]^{e_\chi}.
  $$
- Corollary 2.1 告诉你：当 \(\ell\nmid 2n\) 时，
  \(\ell \mid h_k^+\) 当且仅当某个 nontrivial eigenspace 非零。

**为什么这是 Section 2 最关键的一节**

- 论文不直接证明“整个 class group 为零”。
- 它证明的是：“每一个非平凡 \(\chi\)-块都为零”。
- 这就是为什么整个 proof 看起来像一堆 theorem stack，其实本质是把一个大群逐块清空。

**一眼直觉**

- \(e_\chi\) 是投影算子。
- 它把一个“混合了很多频率”的 class-group element 投到某个固定频率的分量上。
- 后文每次说“取 \(c\in \mathrm{Cl}(K_k^+)[\ell]^{e_\chi}\)”时，实际上就是把问题缩到一维频带里处理。

**隐藏前置**

- 为什么要要求 \(\ell\nmid |G_k|=2^{k-2}\)。
- 这背后是 \(\mathbb{F}_\ell[G_k]\) 的 semisimplicity；在 `3.2` 的证明里会显性冒出来。

**在 Section 3 的用法**

- `3.2` 只对一个固定 \(\chi\)-eigenspace 证明它为零。
- `3.3` 则说明如果某个 full-order eigenspace 非零，那么某个 Bernoulli norm 必被 \(\ell\) 整除。
- 所以 Eq. (7) 是从“global class number question”转向“local frequency question”的总闸门。

#### 一个零跳步 notebook：`Definition 2.10 + Proposition 2.3 + Corollary 2.1` 到底怎样把 class group 切开

很多读者在这里会被一句话卡住：

> “取 \(c\in \mathrm{Cl}(K_k^+)[\ell]^{e_\chi}\)。”

如果你还没把 \(e_\chi\) 真正看成投影算子，这句话会像凭空变出一个新宇宙。最稳妥的读法是把这段当成一页 Fourier-style notebook。

**Step 1：先把 \(e_\chi\) 当成群环里的频率投影**

在 cyclic 群 \(G_k\) 上，一个标准 idempotent 的形状可以记成

$$
e_\chi = \frac{1}{|G_k|}\sum_{g\in G_k}\chi(g^{-1})\,g.
$$

这条公式最重要的不是背下来，而是看懂它在做什么：

1. 它把每个群元素 \(g\) 按 \(\chi(g^{-1})\) 加权；
2. 这个加权方式会保留 \(\chi\)-频率，消掉别的频率；
3. 因而 \(e_\chi\) 的角色和 Fourier analysis 里的“抽取某一频率分量”是同一种结构。

也就是说，若把 class-group element 想成“混合了很多 character 频率的信号”，那
\(e_\chi\) 就是在做：

```text
mixed signal
    ->
project to the chi-frequency channel
    ->
keep only the chi-part
```

**Step 2：为什么这里可以除以 \(|G_k|=2^{k-2}\)**

这也是初学者最容易略过去、但后面会真正咬你的地方。

- 论文固定在 odd prime \(\ell\) 的语境里工作。
- 而 \(G_k\) 的大小是 \(2\)-幂。
- 因此当 \(\ell\nmid 2^{k-2}\) 时，\(|G_k|\) 在 \(\mathbb{F}_\ell\) 或 \(\mathbb{Z}_\ell\)-语境下都是可逆的。

这就是为什么 idempotent decomposition 是合法的；也是为什么 Corollary 2.1 里要专门写
\(\ell\nmid 2n\)。

如果这件事失败，群环就可能不再 semisimple，你也就不能理直气壮地把 class group 切成一堆互相独立的 \(\chi\)-块。

**Step 3：Eq. (7) 为什么真的是 direct sum decomposition**

Proposition 2.3 给的四条关系里，真正承重的是三条：

1. \(e_\chi^2=e_\chi\)：说明它真的是投影，而不是一般线性算子；
2. \(e_\chi e_\psi=0\)（当 \(\chi\neq\psi\)）：说明不同频率之间彼此正交；
3. \(\sum_{\chi\in\hat G_k} e_\chi = 1\)：说明所有频率加起来刚好还原整体。

这三条一合起来，Eq. (7)

$$
\mathrm{Cl}(K_k^+)[\ell] = \bigoplus_{\chi\in\hat G_k}\mathrm{Cl}(K_k^+)[\ell]^{e_\chi}
$$

就不再神秘了。它说的无非是：

```text
every ell-torsion class
    =
sum of its chi-frequency pieces

different chi-pieces do not overlap
```

所以这里的 “direct sum” 不是抽象装饰，而是在给后文开工作票：
以后你可以一块一块地消风险，而不必每次对整个 class group 动手。

**Step 4：一个最小 toy picture**

先别看真实的 \(G_9\cong \mathbb{Z}/128\mathbb{Z}\)，先把脑内图景缩到
\(G=\mathbb{Z}/4\mathbb{Z}\)。

- 这时 characters 由
  \(\chi(\sigma)\in\{1,-1,i,-i\}\)
  决定。
- 一个 class-group 元素如果同时含有 “\(1\)-频率”和 “\(i\)-频率” 成分，
  那 \(e_{\chi_i}\) 作用之后，就只会留下 \(i\)-频率那一块。
- 再对别的 \(\chi\) 用对应的 \(e_\chi\)，就会抽出别的块。

真实论文里只是把这张图从 `4` 个频率升级到 `2^{k-2}` 个频率，但逻辑完全一样。

**Step 5：Corollary 2.1 为什么能把“ell | h_k^+”翻成“某个非平凡 eigenspace 非零”**

一旦 Eq. (7) 成立，剩下的逻辑其实是有限阿贝尔群的常识加上 semisimplicity：

1. 若 \(\ell \mid h_k^+\)，则
   \(\mathrm{Cl}(K_k^+)[\ell]\neq 0\)；
2. 既然整个 \(\ell\)-torsion 群非零，而它又分解成一堆 \(\chi\)-块的直和，
   那么至少有一个块非零；
3. 反过来，只要某个非平凡 eigenspace 非零，整个 \(\ell\)-torsion 当然也非零。

所以 Corollary 2.1 真正做的是把一个粗粒度事件：

```text
ell divides the class number
```

翻译成一个细粒度事件：

```text
some specific chi-channel is still alive
```

而这正是 `3.2` 和 `3.3` 真正会处理的对象。

**Step 6：为什么这一步会直接喂给后面的 theorem chain**

一旦你接受这页 notebook，后面的三段主证明就全都更自然了：

1. `3.2` 不再是在“证明一个怪命题”，而是在说：
   低阶 \(\chi\)-频道全部静音；
2. `3.3` 不再是在“突然引入 Bernoulli 数”，而是在说：
   如果某个 full-order 频道还活着，就把它翻译成一个整数整除事件；
3. `3.4` 则是在说：
   每个可能还活着的频道，最后都对应到一个会被筛空的 prime list。

所以对于真正想跟住 theorem 的读者，这一页最该 internalize 的不是群环公式本身，而是下面这句：

```text
class number divisibility
    ->
some chi-channel survives
    ->
kill low-order channels
    ->
translate surviving full-order channels to Bernoulli norms
```

这就是 `2.5` 之所以是整篇论文第一道“proof language gate”的原因。

### 7.6 `2.6 Cyclotomic units`

**作者在解决什么**

- 这一节给 `3.1` 的 sieve 找到一个显式可测试的单位。
- Definition 2.11 与 Eq. (8) 定义
  $$
  \xi_a=\frac{\zeta^a-\zeta^{-a}}{\zeta-\zeta^{-1}}
  =\frac{\sin(\pi a/2^k)}{\sin(\pi/2^k)}.
  $$
- Definition 2.12 定义 cyclotomic unit group \(C^+\)。
- Theorem 2.1 与 Eq. (9) 给出 Sinnott 公式：
  $$
  h_k^+ = [\mathcal{O}_{K_k^+}^\times : C^+].
  $$

**为什么 \(\xi_5\) 这么重要**

- 因为论文的筛法不是在整个 unit group 上做“存在性论证”，而是抓住一个具体单位 \(\xi_5\) 去做 power-residue test。
- Eq. (9) 告诉你：如果 \(\ell\) 不可能整除单位指数，就不可能整除 \(h_k^+\)。
- 这就把 class number 问题接成了一个显式 residue computation。

**一眼直觉**

- 全体单位群太大，不适合直接算。
- cyclotomic units 是一批“有公式、可下手”的特殊单位。
- Sinnott 公式说：这批可写下来的单位，已经足够测出 class number 的关键信息。

**一个特别容易忽略的点**

- \(\xi_a=\xi_{2^k-a}\) 不是闲笔。
- 它会在 `3.5` 的 conjugate-pair/orbit reduction 里回收一半甚至更多的重复计算。

**隐藏前置**

- unit group 与 index 的基本语言。
- 为什么显式单位能控制 class number，这是 Sinnott 定理提供的深层输入。

**在 Section 3 的用法**

- `3.1` 的 Eq. (18) 与 Eq. (19) 都直接围绕 \(\xi_5\) 写。
- proof 的最后一步是把“\(\xi_5\) 的 power residue symbol 非平凡”翻译成 \(\ell\nmid [\mathcal{O}_{K_k^+}^\times:C^+]\)，再由 Eq. (9) 得 \(\ell\nmid h_k^+\)。

### 7.7 `2.7 Generalized Bernoulli numbers`

**作者在解决什么**

- 这一节为 `3.3` 准备一个“可以取 norm、可以分解素因子”的整数型对象。
- Definition 2.13 引入 Dirichlet character 与 conductor。
- Definition 2.14 与 Eq. (10) 定义 generalized Bernoulli number：
  $$
  B_{1,\psi}=\frac{1}{f}\sum_{a=1}^{f}\psi(a)a.
  $$
- Definition 2.15 引入 Stickelberger element \(\theta_k\)。
- Theorem 2.2 说明 \(2\theta_k\) 消去 \(\mathrm{Cl}(K_k)\)。
- Eq. (13) 给出最关键的 evaluation bridge：
  \(\chi(\theta_k)=B_{1,\chi^{-1}}\)。

**为什么这一节是从抽象群论通往显式整数算术的桥**

- 如果没有 Bernoulli numbers，nonzero eigenspace 仍然只是“某个 abstract class-group event”。
- 有了 Eq. (13)，character evaluation 会把 group-ring 里的 annihilator 变成一个具体数。
- 再取 norm，就能落到 \(\mathbb{Z}\) 里讨论 prime divisibility。

**一眼直觉**

- \(B_{1,\psi}\) 可以先粗看成“按 character 加权的 residue average”。
- 它看起来像解析数论对象，但在这篇论文里最重要的角色是：给 class-group eigenspace 提供一个可计算的 arithmetic shadow。

**一个关键观察**

- 对 even nontrivial characters，很多 Bernoulli 贡献会通过配对直接消失。
- 这也解释了为什么论文后面真正盯的是 odd primitive Dirichlet characters \(\psi\)。

**隐藏前置**

- Dirichlet character 的 primitive/imprimitive 区别。
- group ring 元素在 character 下取值是什么意思。

**在 Section 3 的用法**

- `3.3` 的 Theorem 3.2 会把 “full-order eigenspace 非零” 送成
  \(\ell \mid \operatorname{N}(B_{1,\psi})\)。
- 这就是 candidate set \(S_k\) 的来源。

#### 一个最容易混淆的双账本：为什么 \(\xi_5\) 与 \(B_{1,\psi}\) 不是同一种证据

很多第一次读论文的人，会把 `2.6` 与 `2.7` 混成一句话：

> “反正都是从 class group 推到某个显式量，然后算一算。”

这句话太粗，会直接毁掉对 `Section 3` 的理解。更准确的说法是：作者同时在维护 **两本账**，而且两本账负责的任务完全不同。

**第一本账：unit-side ledger**

- 这里的核心对象是 \(\xi_5\)。
- 它通过 cyclotomic units 与 Sinnott 公式接到
  \([\mathcal{O}_{K_k^+}^{\times}:C^+]\) 和 \(h_k^+\)。
- 它回答的问题是：
  对一个 **已经给定的** 候选素数 \(\ell\)，能不能直接在每个
  \(\mathfrak l\mid\ell\) 的 residue field 里把它排掉？

压缩成链条就是：

```text
fix a prime ell
    ->
look at xi_5 mod every prime ideal l | ell
    ->
residue test is nontrivial
    ->
ell ∤ [O_{K_k^+}^× : C^+]
    ->
ell ∤ h_k^+
```

这本账的功能是 **prime elimination**。它不负责找出全部嫌疑 prime，但一旦给你一个具体 \(\ell\)，它就可能把这个 \(\ell\) 直接删掉。

**第二本账：eigenspace-to-Bernoulli ledger**

- 这里的核心对象是 \(B_{1,\psi}\)。
- 它通过 Herbrand-Ribet / Stickelberger 语言，把 full-order eigenspace 的非零性翻译成 Bernoulli norm 的整除性。
- 它回答的问题是：
  如果某个 odd prime \(\ell\) 真的整除 \(h_k^+\)，那么它 **必须出现在哪些显式整数的素因子表里**？

压缩成链条就是：

```text
assume some full-order eigenspace is nonzero
    ->
attach an odd primitive Dirichlet character psi
    ->
form B_{1,psi}
    ->
take the rational norm N(B_{1,psi})
    ->
ell | N(B_{1,psi})
```

这本账的功能是 **candidate generation**。它不会直接证明
\(\ell\nmid h_k^+\)，但会告诉你：任何真坏素数都逃不出一张有限候选表。

**为什么两本账缺一不可**

只用第一本账不够，因为你没法对“所有奇素数”逐个做 residue test。必须先有第二本账，把无限 prime universe 压成有限 suspect list。

只用第二本账也不够，因为
\(\ell\mid \operatorname{N}(B_{1,\psi})\)
只是 **必要条件**，不是最后的排除证书。一个 prime 出现在 Bernoulli norm 的素因子表里，只能说明“它值得怀疑”，还不能说明它真的整除 \(h_k^+\)。

因此把 `Algorithm 1` 教材化以后，最该记住的不是某条长伪代码，而是这句：

```text
Bernoulli norms generate suspects
    ->
xi_5 residue tests eliminate suspects
```

如果你把这两本账分清楚，再看后面的 `Phase A / Phase B`，会立刻更清楚：

1. `Phase A` 主要使用的是 \(B_{1,\psi}\) 这一侧。
2. `Phase B` 主要使用的是 \(\xi_5\) 这一侧。
3. `Theorem 3.3` 的闭环，不是靠任何一边单独完成，而是靠这两边首尾拼接。

也正因为如此，`2.6` 与 `2.7` 在 Section 2 里都不是“背景补充材料”。

- 没有 `2.6`，你不知道为什么一个局部 residue test 能真的退出到
  \(\ell\nmid h_k^+\)；
- 没有 `2.7`，你不知道为什么 abstract eigenspace 风险会变成一个可分解素因子的整数问题。

对讲义读者来说，这正是 Section 2 后半段最该 internalize 的结构分工。

### 7.8 `2.8 The Cyclotomic \(\mathbb{Z}_2\)-Tower`

**作者在解决什么**

- 这一节把不同层之间的关系写成可用的 tower language。
- Eq. (14) 给出
  \(\operatorname{Gal}(K_\infty^+/\mathbb{Q})\cong \mathbb{Z}_2\)。
- Eq. (15) 把整条塔写成
  \(\mathbb{Q}=K_2^+\subset K_3^+\subset K_4^+\subset \cdots\)。
- Eq. (16) 给出二次层间 norm：
  $$
  N_{k+1/k}(\alpha)=\alpha\cdot \sigma^{n_{k+1}/2}(\alpha).
  $$
- Lemma 2.2 说：当 \(\operatorname{ord}(\chi)\mid n_{k+1}/2\) 时，norm element
  \(N=1+\sigma^{n_{k+1}/2}\) 在 \(\chi\)-块上作用成乘以 \(2\)。

`为什么这一节是 `3.2` 的完整预告`

- `3.2` 的真正杀招不是高深 Iwasawa machinery，而是“同一个 \(N\) 同时等于 \(2\) 和 \(0\)”。
- 这一节先告诉你：为什么会有这么一个 \(N\)，以及为什么它和上下一层的 norm map 是同一个对象。

**一眼直觉**

- 你可以把 tower 想成一层层升级的 cyclotomic real fields。
- norm map 是“把上一层看不到的相位折叠回去”。
- 当某个 character 太低频时，这个折叠既不会改变它，又会因为上一层 class number one 而把它压成零。

**隐藏前置**

- quadratic extension 上 ideals 的 norm。
- 为什么 \(\sigma^{n/2}\) 正好是层间非平凡自同构。

**在 Section 3 的用法**

- `3.2` 会直接把 Lemma 2.2 实例化为 \(N\cdot c=2c\)。
- 再结合上层到下层的 ideal norm 得到 \(N\cdot c=0\)。

### 7.9 `2.9 Iwasawa theory`

**作者在解决什么**

- 这一节不是主证明的手术刀，而是告诉你这整个问题为什么天然属于 cyclotomic tower arithmetic。
- Theorem 2.3 与 Eq. (17) 给出 class-number growth 的 Iwasawa 形状：
  $$
  e_n=\mu p^n+\lambda n+\nu.
  $$
- Theorem 2.4 用 Ferrero-Washington 给出 \(\mu=0\)。
- 文中还特别提醒：在本文关心的 \(k\le 12\) 范围内，\(2\nmid h_k^+\) 已知。

**为什么作者要放这一节，但后面又不靠它硬推主定理**

- 因为 Weber conjecture 本来就是 cyclotomic \(\mathbb{Z}_2\)-tower 里的经典问题。
- 这节的作用更像“解释这片地形”，而不是替你走完最后一公里。
- 它让读者明白：论文不是孤立发明了几个筛法，而是在 Iwasawa-theoretic landscape 上选了一个对 \(k\le 12\) 真正可执行的路线。

**读者该如何把握轻重**

- 初读时，不需要把 \(\mu,\lambda,\nu\) 的全部理论背下来。
- 但要牢牢记住两点：
  1. 这问题本来就属于塔上的 class-number growth；
  2. 本文最终选择的是“tower + eigenspace + Bernoulli norms + finite computation”的无条件路线。

**这一节在 Section 3 的真实用法**

- 它不直接证明 Theorem 3.3。
- 它负责解释为什么 “按层归纳” 与 “只看 odd primes” 是自然的。

### 7.10 把 `Section 2` 直接改写成 `Algorithm 1` 的前置接口图

很多读者在读完 `Section 2` 之后，仍然会有一种强烈的断裂感：

> 我知道了 class group、characters、cyclotomic units、Bernoulli numbers，
> 但为什么下一节突然就能开始“枚举 candidate primes 再做 residue tests”？

这不是读者粗心，而是因为原论文默认你会把 `Section 2` 自动压成一张 **algorithm preflight map**。教材化地读，最好把它显式写出来。

**第一步：先看 Algorithm 1 到底需要什么输入**

把原文算法的数学输入压缩一下，其实只需要五类东西：

1. 一个要验证的层数 \(k\)；
2. 上一层结论 \(h_{k-1}^+=1\)；
3. 一个把 class-group 风险切成 \(\chi\)-块的语言；
4. 一个把 full-order 风险翻译成显式整数的语言；
5. 一个把单个候选 prime \(\ell\) 直接删掉的局部测试。

而这五类输入，恰好就是 `Section 2` 后半段逐节准备出来的。

**第二步：把 Section 2 的对象逐项对齐到 Algorithm 1**

```text
2.5 eigenspace decomposition
    ->
告诉你“坏事发生在哪个 chi-块”
    ->
为 3.2 / Corollary 3.1 准备 character-space elimination

2.6 cyclotomic units + Sinnott
    ->
告诉你“怎么把单个 ell 的局部测试退出到 h_k^+”
    ->
为 3.1 / Phase B 准备 prime elimination

2.7 Bernoulli + Stickelberger evaluation
    ->
告诉你“怎么把 full-order eigenspace 风险翻译成整数整除”
    ->
为 3.3 / Phase A 准备 candidate generation

2.8 Z_2-tower + layer norm
    ->
告诉你“为什么低阶 characters 会被上一层结论杀掉”
    ->
为 3.2 准备 full-order reduction

2.9 Iwasawa landscape + known 2-part facts
    ->
告诉你“为什么可以按层归纳且先把 odd primes 单独处理”
    ->
为 3.4 的 induction framing 准备全局语境
```

如果你把这张图读顺，就会发现：`Algorithm 1` 不是突然出现的工程模块，而是 `Section 2` 全部接口被接好之后的自然落点。

**第三步：把这张图压成一页真正可用的前置清单**

对一个准备进入 `Theorem 3.3` 或 `Algorithm 1` 的读者，最有用的不是背 theorem 编号，而是先问自己下面五个问题。

1. 我是否已经知道 class-group 风险会先落到某个
   \(\mathrm{Cl}(K_k^+)[\ell]^{e_\chi}\)？
   如果不知道，就说明 `2.5` 还没吃透。
2. 我是否知道为什么
   \(\xi_5\) 的局部非平凡性，真的能推出
   \(\ell\nmid h_k^+\)？
   如果不知道，就说明 `2.6` 还只是“看过名词”。
3. 我是否知道为什么某个 full-order eigenspace 非零，会逼出
   \(\ell\mid \operatorname{N}(B_{1,\psi})\)？
   如果不知道，就说明 `2.7` 还没变成可计算接口。
4. 我是否知道为什么上一层的
   \(h_{k-1}^+=1\)
   足以干掉 low-order characters？
   如果不知道，就说明 `2.8` 和 `3.2` 之间还没接上。
5. 我是否知道为什么主定理是按层归纳、并且主要只追 odd primes？
   如果不知道，就说明 `2.9` 的地形图还没 internalize。

只要这五个问题里有两个以上答不上来，直接冲 `Algorithm 1` 往往都会把人打回 “怎么又来了一个新对象” 的状态。

**第四步：于是 Algorithm 1 的两相就不再神秘**

当上面这张 preflight map 建好以后，原文算法的两相会突然变得非常直白：

**Phase A**

- 先用 `2.5 + 2.8` 知道只需看 full-order layer；
- 再用 `2.7` 把 full-order 风险翻译成 Bernoulli norms；
- 最后对这些 norms 做 factorization 与筛法，得到有限 suspect list。

**Phase B**

- 取 Phase A 留下的单个 suspect prime \(\ell\)；
- 用 `2.6` 给出的 cyclotomic unit / Sinnott 出口；
- 对每个 \(\mathfrak l\mid\ell\) 跑 \(\xi_5\) residue test；
- 若局部测试全部失败，就删掉这个 \(\ell\)。

于是整条算法的骨架其实就是：

```text
Section 2 builds the interfaces
    ->
Section 3 composes the interfaces
    ->
Algorithm 1 executes the composed interfaces
```

**这一小节最该记住的结论**

1. `Section 2` 不是 theorem 之前的“知识背景区”，而是 `Algorithm 1` 的接口定义区。
2. `Phase A` 和 `Phase B` 分别吃的是 `2.7` 与 `2.6` 这两套 completely different evidence chains。
3. 真正把它们接起来的是 `2.5` 的 eigenspace language、`2.8` 的 tower norm，以及 `2.9` 给出的归纳地形。

## 8. 逐章精读 Section 3：主证明链的真正承重位置

Section 3 真正做的是三次降维：

```text
先从“所有奇素数”降到“满足特定同余的候选素数”
    ->
再从“所有 characters”降到“只有 full-order characters 还可能出事”
    ->
最后从“抽象 eigenspace 非零”降到“某个显式整数 norm 被 ell 整除”
```

如果你读这节时只记结论，会觉得作者像在不断换工具；如果你抓住这三次降维，就会发现整篇证明其实非常直线。

### 8.1 `3.1 The Fukuda-Komatsu sieve`

**这一节到底在做什么**

- 它先不碰 characters，也不碰 Bernoulli numbers。
- 它做的是最前端的 prime filtering：把绝大多数奇素数直接挡在门外。

**关键对象**

- Definition 3.1 把 Wieferich criterion 写成一个对 \(\xi_5\) 的 residue test。Eq. (18) 是
  $$
  \xi_5^{(\ell-1)/\operatorname{ord}_{\mathfrak l}(\xi_5)} \not\equiv 1 \pmod{\mathfrak l}.
  $$
- Lemma 3.1 在附加条件 \(\ell\equiv \pm 1 \pmod{2^{k-1}}\) 下，把它简化成更便于实际检验的 Eq. (19)：
  $$
  \xi_5^{(\ell-1)/2^{k-1}} \not\equiv 1 \pmod{\mathfrak l}.
  $$
- 论文还把这件事改写为 power residue symbol 的语言，这就是 Eq. (20) 的角色。

**为什么作者一定要抓 \(\xi_5\)**

- 因为 \(h_k^+=[\mathcal{O}_{K_k^+}^\times:C^+]\)。
- 一旦能证明 \(\xi_5\) 在相应 residue quotient 中的行为与 “\(\ell\) 不可能整除该 index” 相冲突，就能直接推出 \(\ell\nmid h_k^+\)。
- 换句话说，\(\xi_5\) 不是随手挑的一个单位，而是 cyclotomic unit group 里足够显式、又足够灵敏的探针。

**为什么 \(\ell\equiv \pm 1 \pmod{2^{k-1}}\) 会对应 residue degree \(f\le 2\)**

- 先回忆 \(G_k\cong \mathbb{Z}/2^{k-2}\mathbb{Z}\)，Frobenius \(\operatorname{Frob}_\ell\) 的阶就是 residue degree \(f\)。
- 如果 \(\ell\equiv 1 \pmod{2^{k-1}}\)，那它在 \(G_k\) 里的像已经是单位元，所以 \(f=1\)。
- 如果 \(\ell\equiv -1 \pmod{2^{k-1}}\)，则 \(\ell^2\equiv 1 \pmod{2^{k-2}}\)，于是 \(\operatorname{Frob}_\ell\) 的阶至多是 \(2\)。
- 这就是 Theorem 3.1(ii) 里同余条件的群论本质：作者要把 residue field degree 压到 \(1\) 或 \(2\)，这样测试指数才会落成 \((\ell-1)/2^{k-1}\) 或 \((\ell^2-1)/2^{k-1}\) 这种可算量。

**Lemma 3.1 的证明该怎么读**

先把它读成“反证法版的 order-divisibility argument”。

1. 设 \(d=\operatorname{ord}_{\mathfrak l}(\xi_5)\)。
2. 若 \(\ell\) 失败了 Wieferich criterion，那么 \(d\) 必须整除某个更小的指数。
3. 当 \(f=1\) 时，作者证明
   \((\ell-1)/\operatorname{ord}_\ell(5)\mid (\ell-1)/2^{k-1}\)，因此 \(d\mid (\ell-1)/2^{k-1}\)。
4. 于是得到
   \(\xi_5^{(\ell-1)/2^{k-1}}\equiv 1 \pmod{\mathfrak l}\)，这与 Eq. (19) 矛盾。
5. 当 \(f=2\) 时，同样逻辑变成
   \((\ell-1)/\operatorname{ord}_\ell(5)\mid (\ell^2-1)/2^{k-1}\)，从而得到
   \(\xi_5^{(\ell^2-1)/2^{k-1}}\equiv 1 \pmod{\mathfrak l}\)。

教学上最该盯住的不是细枝常数，而是这条结构：

```text
Frobenius order small
    ->
residue field degree small
    ->
test exponent becomes explicit
    ->
failure of Wieferich criterion forces a forbidden congruence
```

**Theorem 3.1 的真正产出**

- 产出一：所有 \(\ell<10^9\) 都被筛掉。
- 产出二：任何还活着的奇素数都必须满足
  \(\ell\equiv \pm 1 \pmod{2^{k-1}}\)。

第一条是 computation-heavy 的范围清空，第二条是 structure-heavy 的同余刚性。
两者一起的效果，不是“已经证明完了”，而是把后面要面对的 prime universe 压成一个又大又稀薄、但仍然可控的集合。

**这一节之后还剩什么**

- 还剩的大素数并没有消失。
- 它们只是被压到了 very sparse congruence classes 里。
- 下一节会说明：即便这些大素数存在，它们也不可能躲在 low-order characters 里。

### 8.2 `3.2 Norm-coherence eliminates low-order characters`

**这一节到底在做什么**

- `3.1` 消的是 prime space。
- `3.2` 消的是 character space。
- 它的目标是证明：只要上一层已经有 \(h_{k-1}^+=1\)，那本层所有 non-full-order eigenspaces 都自动归零。

**主结论**

- Proposition 3.1：若 \(h_{k-1}^+=1\)，则对每个满足 \(\operatorname{ord}(\chi)\mid 2^{k-3}\) 的 character，
  \(\mathrm{Cl}(K_k^+)[\ell]^{e_\chi}=0\)。
- Corollary 3.1：因此若 \(\ell\mid h_k^+\)，那么非平凡 \(\ell\)-torsion 只能落在 full-order characters 上。

**这段 proof 的核心不是“高深”，而是“同一个算子同时等于 2 和 0”**

令 \(n=|G_k|=2^{k-2}\)，并取
$$
N = 1 + \sigma^{n/2}.
$$

然后固定一个元素
\(c\in \mathrm{Cl}(K_k^+)[\ell]^{e_\chi}\)。

第一半来自 character information：

1. 因为 \(c\) 在 \(\chi\)-eigenspace 中，\(\sigma\cdot c=\chi(\sigma)c\)。
2. 因为 \(\operatorname{ord}(\chi)\mid n/2\)，有 \(\chi(\sigma)^{n/2}=1\)。
3. 所以
   $$
   N\cdot c = (1+\sigma^{n/2})\cdot c = c+\sigma^{n/2}\cdot c = 2c.
   $$
   这就是原文的 Eq. (29)。

第二半来自 tower norm：

1. 取代表 ideal \(\mathfrak a\) 使 \(c=[\mathfrak a]\)。
2. group-ring action 给出
   \(N\cdot c=[\mathfrak a\cdot \sigma^{n/2}(\mathfrak a)]\)。
3. 对二次扩张 \(K_k^+/K_{k-1}^+\)，有
   $$
   \mathfrak a\cdot \sigma^{n/2}(\mathfrak a)
   = \mathrm{Nm}_{k/(k-1)}(\mathfrak a)\cdot \mathcal{O}_{K_k^+}.
   $$
   这是原文 Eq. (31) 的意思。
4. 由于假设 \(h_{k-1}^+=1\)，下层 class group 平凡，所以
   \(\mathrm{Nm}_{k/(k-1)}(\mathfrak a)\) 是 principal ideal。
5. 因而
   $$
   N\cdot c = 0.
   $$
   这就是原文 Eq. (33)。

于是两边合起来：
$$
2c = N\cdot c = 0.
$$

同时因为 \(c\in \mathrm{Cl}(K_k^+)[\ell]\)，还有
\(\ell c=0\)。
而 \(\ell\) 是奇素数，所以 \(\gcd(2,\ell)=1\)。Bézout 恒等式给出整数 \(a,b\) 满足
\(2a+\ell b=1\)，进而
$$
c=(2a+\ell b)c = a(2c)+b(\ell c)=0.
$$

这就是原文最后那一步为何不是“显然”，而是一个非常具体的 annihilator argument。

**为什么这一步在全篇里极其重要**

- 它第一次真正利用了归纳假设 \(h_{k-1}^+=1\)。
- 它把“所有 nontrivial characters 都可能出事”砍成“只剩 full-order characters 还有嫌疑”。
- 对 \(k=9\) 而言，\(G_9\cong \mathbb{Z}/128\mathbb{Z}\)，所以 orders
  \(\{1,2,4,8,16,32,64\}\) 的 characters 全部被清空，只剩 order \(128\) 的 characters。

**最容易误解的地方**

- 不是因为 full-order characters “更危险”，而是因为 low-order characters 与 tower norm 相容得太强，反而会被 \(N\) 直接杀掉。
- 所以 full-order 不是天生特殊，而是这套 \(N=1+\sigma^{n/2}\) 机制无法再触及它们。

**这一节之后还剩什么**

- prime space 已经被 `3.1` 稀疏化。
- character space 现在又被压成了 full-order layer。
- 接下来只差最后一次翻译：把“某个 full-order eigenspace 非零”变成“某个具体整数有 \(\ell\) 因子”。

### 8.3 `3.3 Herbrand's theorem bounds candidate primes`

**这一节到底在做什么**

- 这是整篇论文从 abstract class-group statement 进入 explicit finite computation 的转轴。
- Theorem 3.2 给出 Herbrand-Ribet 型桥。
- Definition 3.2 与 Proposition 3.2 把这座桥压缩成有限候选集 \(S_k\)。

**先看 Theorem 3.2 在说什么**

如果某个 full-order character \(\chi\) 的 eigenspace 非零，作者构造
$$
\psi = \chi^{-1}\omega_\ell,
$$
其中 \(\omega_\ell\) 是 Teichmüller character。

Lemma 3.3 先把这个 \(\psi\) 的性质整理干净：

1. \(\psi\) 是 primitive odd Dirichlet character。
2. 它的 conductor 是 \(2^k\ell\)。
3. 对 \(\ell\)-divisibility 来说，
   \(\operatorname{N}(B_{1,\psi})\) 与 \(\operatorname{N}(B_{1,\chi^{-1}})\) 是等价的。

原文把这条等价性写成 Eq. (39)：
\(\ell\mid \operatorname{N}(B_{1,\psi})\) 当且仅当
\(\ell\mid \operatorname{N}(B_{1,\chi^{-1}})\)。

教学上这一步的重点不是记住 \(\omega_\ell\) 的定义细节，而是理解它的作用：

```text
class-group eigenspace data
    ->
attach a Dirichlet character psi
    ->
form B_{1,psi}
    ->
take rational norm
    ->
obtain an ordinary integer divisible by ell
```

**为什么这一步突然能落到整数上**

- \(B_{1,\psi}\) 本来住在 cyclotomic field 里，不是整数。
- 但把所有 Galois conjugates 一乘，rational norm 就回到 \(\mathbb{Z}\)。
- 一旦回到 \(\mathbb{Z}\)，你就能做 prime factorization，而不是继续困在 abstract eigenspace 里。

**Definition 3.2 与 Proposition 3.2 的真正含义**

作者把 candidate set 定义成同时满足三条的 primes：

1. \(\ell>10^9\)；
2. \(\ell\equiv \pm 1 \pmod{2^{k-1}}\)；
3. 对某个 full-order character，有
   \(\ell\mid \operatorname{N}_{\mathbb{Q}(\zeta_m)/\mathbb{Q}}(B_{1,\psi_j})\)。

于是 Proposition 3.2 说：

- \(S_k\) 是 finite；
- \(S_k\) 是 computable；
- 每个 odd prime divisor of \(h_k^+\) 都在 \(S_k\) 里。

这句话如果只机械地读，会觉得像“定义了一个集合然后说它包含坏素数”。真正的内容其实是三层桥的合成：

```text
ell | h_k^+
    ->
Corollary 3.1: some full-order eigenspace is nonzero
    ->
Theorem 3.2: ell divides a Bernoulli norm
    ->
Definition 3.2: ell enters S_k
```

**为什么 \(S_k\) 真的是有限而且可算**

这里有两个原因。

第一，character orbit 数本来就有限。

- 由 Lemma 2.1，full-order characters 只会按共轭成对出现。
- 所以真正需要看的 pair 数至多是 \(n/4\)，而不是无限多。

第二，每个对应的 norm 都有显式上界。

- 原文 Eq. (41) 用 odd-character pairing 把 Bernoulli 和式对半整理。
- Eq. (42) 导出
  $$
  |B_{1,\psi}| \le 2^{k-2}.
  $$
- Eq. (43) 再把 rational norm 写成所有共轭的乘积。

于是 rational norm 不但落在 \(\mathbb{Z}\) 里，而且其大小还有明确上界。只要一个非零整数有上界，它的 prime divisors 就只可能来自一个 finite list。

**为什么这些上界在计算上还算现实**

- 论文后面说明，紧的 second-moment bound 让最关键的整数在相关层数上只有可控位数。
- 对 \(k=10\)，tight bound 约在 \(10^{143}\) 量级；worst-case 粗界可到 \(10^{309}\) 量级。
- 这仍然不是“小整数”，但已经是现代 ECM/number-theory software 真能处理的对象。

**这一节最该记住的教学结论**

- Theorem 3.2 不是在“证明 Weber conjecture”，而是在把最后的风险变成整数分解问题。
- Proposition 3.2 也不是单独工作的；它必须建立在 `3.1` 的 prime sieve 和 `3.2` 的 eigenspace elimination 之后才成立。
- 到了这一节结束，论文已经把无限风险压成了一个有限、明确、可枚举的候选表。

### 8.4 `3.4 h_k^+ = 1 for k \le 12`

这里作者把前三步收束成主定理和算法。

- `Theorem 3.3`：对 \(k \in \{9,10,11,12\}\)，有 \(h_k^+ = 1\)。
- 证明结构：
  1. base case 是 Miller 给出的 \(h_8^+ = 1\)；
  2. Theorem 3.1 负责消去所有小奇素数与错误同余类；
  3. Corollary 3.1 负责把剩余风险压到 full-order characters；
  4. Proposition 3.2 负责把风险压进有限候选集 \(S_k\)；
  5. Algorithm 1 对 \(S_k\) 逐个做最终验证。

### 8.5 `3.5 Optimizations`：为什么 paper-scale computation 没有爆炸

如果说 `3.1` 到 `3.4` 回答的是“为什么主定理为真”，那么 `3.5` 回答的是“为什么这个 theorem 在现实层数上还能算得完”。这一节最重要的，不是记住所有实现细节，而是先把 **正确性层** 和 **优化层** 分开。

**先把边界划清楚**

- 到 `Proposition 3.2` 为止，论文已经把风险压成有限候选集 \(S_k\)。
- 到 `Algorithm 1` 为止，论文已经给出了足以完成 `unconditional verification` 的闭环。
- 因此 `3.5` 不是“补证明漏洞”，而是把 computation 从 “理论上可做” 压到 “现实上更容易做”。

如果你第一次读这篇 paper，把 `3.5` 误解成“没有这些优化，Theorem 3.3 就不成立”，就会把证明逻辑和工程代价混在一起。正确理解应当是：

```text
Theorem 3.1 + Corollary 3.1 + Proposition 3.2 + Algorithm 1
    = 正确性闭环

Section 3.5
    = 把这个闭环做得更便宜
```

**第一层优化：Galois orbit reduction**

在 `3.3` 结束时，表面上我们还要看 \(n/4\) 对 full-order conjugate pairs。一个 naive 的想法是：

1. 对每一对 character \((\psi_j,\psi_{n-j})\) 分别算 \(B_{1,\psi_j}\)；
2. 对每个对应的 rational norm 分别做一次整数分解；
3. 再把所有 odd prime divisors 合并起来做 sieve。

如果真这么做，代价会随 \(n=2^{k-2}\) 线性增长。`3.5` 的第一条消息正是：这一步里有大量重复工作。

原文 `Proposition 3.3` 说：

> 所有 \(n/4\) 对 full-order conjugate pairs 给出的
> \(|\operatorname{N}(B_{1,\psi_j})|\) 都相同。

这句话为什么成立？证明骨架其实很干净：

1. \(\operatorname{Gal}(\mathbb{Q}(\zeta_m)/\mathbb{Q}) \cong (\mathbb{Z}/m\mathbb{Z})^\times\) 会作用在 characters 上，作用方式是
   \(\psi \mapsto \psi^t\)。
2. 在这个作用下，
   \(B_{1,\psi^t}\) 正好是 \(B_{1,\psi}\) 的 Galois 共轭。
3. rational norm 对 Galois 共轭不变，所以
   \(\operatorname{N}(B_{1,\psi^t})=\operatorname{N}(B_{1,\psi})\)。
4. 再除去 complex conjugation 给出的 \(\{\pm1\}\) 冗余以后，所有 full-order characters 模掉共轭后落在同一个 Galois orbit 里。

所以教材化结论是：

```text
full-order characters 看起来有很多
    ->
它们在 Galois 作用下其实属于同一个 orbit
    ->
对应的 Bernoulli norms 完全一样
    ->
只需要 factor 一个整数，而不是 n/4 个整数
```

这一步非常值得和前面的本地 notebook 联系起来看。我们在 `9.7` 对 `k=9` 的 full-order primitive odd characters 做 merged recomputation 时，确实看到全部 \(64\) 个 characters 给出的 rational norm 都相同。这不是偶然现象，而正是 `Proposition 3.3` 的一个具体影子。

**它到底节省了多少**

- 原文 `Corollary 3.2` 明确说：Algorithm 1 里只需要 **一次** 整数分解，而不是 \(n/4\) 次。
- 对 `k=10`，有 \(n=256\)，于是 factorization count 从
  \(n/4=64\) 次直接压到 \(1\) 次。
- 这就是为什么 paper 可以把真正的瓶颈描述成 “factor one integer of at most 143 decimal digits”，而不是 “factor dozens of unrelated huge integers”。

**这条优化没有做什么**

- 它没有消去任何新的 class-group torsion。
- 它没有替代 `Corollary 3.1` 的 full-order reduction。
- 它也没有改变 candidate set 的定义。

它只是在说：**产生 candidate set 所需的那批 norms，有非常强的对称性，所以不要傻算很多遍。**

**第二层优化：Thaine 的额外 annihilators**

第一层优化仍然保留了一个硬骨头：你还是要 factor 一个很大的 Bernoulli norm。于是论文继续给出第二层优化，它更激进，因为它试图把“整数分解”换成“有限域上的多项式运算”。

原文调用的是 Thaine 的 annihilator theorem。对一个满足
\(q\equiv1\pmod{2^k}\) 的 auxiliary prime \(q\)，再取一个模 \(q\) 阶正好为 \(2^k\) 的元素 \(\eta\)，作者定义群环元素

$$
\theta_q =
\sum_{\substack{1\le a\le 2^k\\(a,2)=1}}
\left\lfloor\frac{a\eta^a}{q}\right\rfloor \sigma_a^{-1}
\in \mathbb{Z}[\Gamma_k].
$$

`Theorem 3.4` 说，在 \(q\nmid h_k^+\) 的前提下，这个 \(\theta_q\) 会 annihilate \(\operatorname{Cl}(K_k^+)\)。

这句如果只按“存在一个 annihilator”来读，仍然很抽象。真正要抓住的是它对 eigenspace elimination 的用途：

1. 固定一个 even character \(\chi\) 和一个候选 odd prime \(\ell\)。
2. 取两个 auxiliary primes \(q_1,q_2\)。
3. 把对应 annihilator 投影到 \(\chi\)-分量上，得到
   \(e_\chi \theta_{q_1}\) 和 \(e_\chi \theta_{q_2}\)。
4. 如果这两个投影在模 \(\ell\) 意义下生成 unit ideal，也就是它们的 GCD 在 \(\mathbb{F}_\ell\) 上等于 \(1\)，那么
   \(\operatorname{Cl}(K_k^+)[\ell]^{e_\chi}=0\)。

换句话说，第二层优化的逻辑不是
“先 factor 一个巨大整数，再问 \(\ell\) 会不会整除它”，而是：

```text
先选辅助素数 q1, q2
    ->
构造两个额外 annihilators
    ->
把它们投影到 chi-eigenspace
    ->
在 F_ell 上算 gcd
    ->
若 gcd = 1，则直接杀掉该 eigenspace 的 ell-torsion
```

这就把最贵的一步从 **大整数分解**，改成了 **有限域上的多项式 GCD**。

**为什么这条优化不会偷偷引入新假设**

这里读者很容易紧张：`Theorem 3.4` 不是还要求 \(q\nmid h_k^+\) 吗？这会不会变成循环依赖？

原文 `Remark 3.2` 专门解释了这个问题。思路是：

1. 辅助素数 \(q\) 满足 \(q\equiv1\pmod{2^k}\)，所以至少有 \(q\ge 2^k+1\)。
2. 对本论文的目标层数 \(k\le12\)，这个规模仍然远小于 `Theorem 3.1(i)` 的 \(10^9\) cutoff。
3. 因而，对选取的这些小辅助素数，可以直接用 Fukuda-Komatsu sieve 先验证 \(q\nmid h_k^+\)。

所以这里不是“为了验证 \(h_k^+\) 又假设了一个新的 class-number fact”，而是把第二层优化安全地接回了第一层 sieve。

**这一层优化和主证明是什么关系**

教学上最值得明确写出来的一点是：

- Galois orbit reduction 是 **确定性的 compression**；
- Thaine + GCD 路线是 **更强的实现优化**；
- 但哪怕你完全不用第二层优化，只保留 `Algorithm 1` 的 factorization 路线，`Theorem 3.3` 依然成立。

所以对第一次精读的读者，一个健康的阅读顺序是：

1. 先把 `3.1` 到 `3.4` 看成完整证明；
2. 再把 `3.5` 看成“如何把 proof-of-existence 变成更轻的 computation”；
3. 最后才去区分哪些优化是 deterministic symmetry，哪些优化带有“随机选 \(q_1,q_2\) 通常就足够”的实践味道。

**这一节最该记住的三句话**

1. `Proposition 3.3` 说的不是 “Bernoulli numbers 相同”，而是 **它们的 rational norms 相同**。
2. `Corollary 3.2` 真正压缩的是 **factorization count**，把按 \(n\) 线性增长的代价压成每层常数次。
3. Thaine 的额外 annihilators 不改变 theorem 的逻辑结构，但它给了你一条“用 polynomial GCD 替代 integer factorization” 的更狠实现路线。

### 8.6 一张证明依赖图

```text
base case h_8^+ = 1
    +
Section 2 language (class group, characters, units, Bernoulli, tower)
    ->
Theorem 3.1
    -> eliminate all odd primes < 10^9
    -> force l ≡ ±1 mod 2^(k-1)
    ->
Proposition 3.1
    ->
Corollary 3.1
    -> only full-order character eigenspaces remain
    ->
Theorem 3.2
    ->
Proposition 3.2
    -> finite computable set S_k
    ->
Algorithm 1
    ->
Theorem 3.3
```

### 8.7 把 Theorem 3.3 的证明按原文顺序拆开

如果你第一次读 `3.4`，很容易以为作者只是把前面结果“引用一下”就结束了。更好的读法是把这段 proof 理解成一个 **prime-divisor exhaustion argument**。

#### Step 1：先固定归纳框架，而不是直接算 \(h_k^+\)

原文第一句就是：

> base case 用 Miller 的 \(h_8^+ = 1\)，然后假设 \(h_{k-1}^+ = 1\)，证明 \(h_k^+ = 1\)。

这一步的作用不是形式主义，而是给 `Proposition 3.1` 提供入口，因为它正是“在上一层 class number one 的条件下”消去低阶 characters。

#### Step 2：先把 \(\ell = 2\) 单独拿走

原文接着引用 Fukuda-Komatsu 的旧结果：对 \(k \le 12\)，有

$$
2 \nmid h_k^+.
$$

为什么这句必须单独写？

- 因为 Section 3 的 machinery 主要围绕 odd primes \(\ell\) 工作。
- 如果你不先把 \(2\)-part 拿走，最后的 “no prime divides \(h_k^+\)” 会缺一块。

#### Step 3：用 Theorem 3.1 消去所有小奇素数

接下来作者说：由 Theorem 3.1(i)，所有 \(\ell < 10^9\) 都不能整除 \(h_k^+\)。

这一步的逻辑很值得停一下：

1. 它没有证明 \(h_k^+=1\)。
2. 它只是把“所有可能的奇素数因子”压到非常大的一段。
3. 这使得后面 Proposition 3.2 的 finite candidate set 才值得算。

如果你把这一步看成“只是个数值筛法”，会低估它的结构作用。它真正做的是把 candidate space 从“所有奇素数”砍成“很大但稀薄的素数”。

#### Step 4：用 Corollary 3.1 与 Proposition 3.2 把大奇素数压进有限集

原文下一步说：若有 odd prime \(\ell > 10^9\) 整除 \(h_k^+\)，则由 Proposition 3.2 知 \(\ell \in S_k\)。

这句话其实压缩了三层桥：

```text
ell | h_k^+
    ->
Corollary 3.1: some full-order eigenspace is nonzero
    ->
Theorem 3.2: corresponding Bernoulli norm is divisible by ell
    ->
Proposition 3.2: ell lies in a finite computable set S_k
```

也就是说，`Proposition 3.2` 单独并不神奇；它的前提是前面已经用 tower argument 把低阶 characters 全部扫掉。

#### Step 5：对 \(S_k\) 中每个候选 \(\ell\) 做最后的 residue elimination

原文最后说：对每个 \(\ell \in S_k\)，计算

$$
\xi_5^{(\ell-1)/2^{k-1}} \bmod \mathfrak{l}
$$

对每个 \(\mathfrak{l} \mid \ell\) 的取值。如果对所有 \(\mathfrak{l}\) 都不等于 \(1\)，则由 Lemma 3.1 得 \(\ell \nmid h_k^+\)。

这一段的教学重点是：

- Phase A 给的是“候选素数表”；
- Phase B 给的是“候选表清零”；
- 二者缺一不可。

只有当你理解这点，才会明白为什么论文说自己给出的是 **unconditional verification**，而不是“把问题约化到一堆看起来可算的整数”。

#### Step 6：为什么结尾真的推出 \(h_k^+ = 1\)

原文最后一句其实是一个有限群论事实：

1. \(2 \nmid h_k^+\)。
2. 没有 odd prime \(\ell\) 能整除 \(h_k^+\)。
3. 因此没有任何 prime 能整除 \(h_k^+\)。
4. 唯一可能就是 \(h_k^+ = 1\)。

这听起来平凡，但教学上值得明确写出来，因为很多读者在前面被 class groups、characters、Bernoulli numbers 淹没之后，反而会忘记最终落点只是一个整数的素因子耗尽。

#### Step 7：为什么这个归纳没有循环

这是读者最常担心的点之一，尤其在 `k=9,10,11,12` 接力时。

正确理解是：

1. \(k=9\) 只依赖 base case \(h_8^+ = 1\)。
2. 一旦 \(k=9\) 被彻底关掉，才能把 \(h_9^+ = 1\) 喂给 \(k=10\)。
3. 然后 \(k=10\) 的结论再喂给 \(k=11\)，依此类推。

所以每一步都 independently terminate：

- Stage 1 给出 sieve；
- Stage 2 给出 eigenspace elimination；
- Stage 3 给出 finite computation；
- 最终候选集被实际算空。

归纳结构只负责“允许你消掉低阶 characters”，并不替你完成最后 computation。这正是它不循环的原因。

### 8.8 把 `Theorem 3.2 + Definition 3.2 + Proposition 3.2` 改写成一个 theorem notebook

如果说 `8.7` 已经把 `Theorem 3.3` 的主证明顺序拆开了，那么这里要补的是其中最容易被一笔带过的那一段：

> 为什么一个真的会整除 \(h_k^+\) 的坏奇素数 \(\ell\)，最后一定会落进作者定义的 finite candidate set \(S_k\)？

原文在 `Proposition 3.2` 的 proof 里把这段压得非常紧。教材化地读，最好把它当成一个 **input-output notebook**。

**输入是什么**

从最坏情况出发，假设：

$$
\ell \mid h_k^+,
\qquad
\ell \text{ 是奇素数}.
$$

我们的任务不是立刻证明矛盾，而是先说明：

$$
\ell \in S_k.
$$

也就是先把“抽象的坏素数”压成“一个明确候选表中的元素”。

**Step 1：先用 Theorem 3.1 把 prime-level 必要条件钉死**

只要 \(\ell\mid h_k^+\)，`Theorem 3.1` 立刻给出两个出口：

1. 由 `Theorem 3.1(i)`，
   \(\ell>10^9\)。
2. 由 `Theorem 3.1(ii)`，
   \(\ell\equiv \pm1 \pmod{2^{k-1}}\)。

这一步在 `Definition 3.2` 里正对应条件 `(i)` 和 `(ii)`。也就是说，candidate set 的前两条不是凭经验拍出来的 filter，而是 paper 前半段已经证明的 **必要条件**。

如果你读到这里还把 \(S_k\) 看成“作者自己定义的一个搜索空间”，就会漏掉最关键的逻辑方向：

```text
不是先定义 S_k 再希望坏素数落进去
而是先证明坏素数必须满足两条刚性约束
然后才把这两条写进 S_k 的定义
```

**Step 2：再用 Corollary 3.1 把风险压到某个 full-order eigenspace**

`Theorem 3.1` 只告诉你 prime 自己必须长什么样，但还没有告诉你 class group 里“坏事发生在哪一层”。这时接 `Corollary 3.1`：

若 \(\ell\mid h_k^+\)，则存在某个 **full-order** even character \(\chi\)，使得

$$
\operatorname{Cl}(K_k^+)[\ell]^{e_\chi}\neq 0.
$$

这一步的含义是：

- 低阶 eigenspaces 已经被 tower norm-coherence 清空；
- 真正可能承载 \(\ell\)-torsion 的，只剩 full-order layer。

所以到这里，输入被进一步压成：

```text
一个大奇素数 ell
    +
某个 full-order even character chi
使得 ell-torsion 在 chi-eigenspace 中非零
```

**Step 3：为什么突然要引入 \(\psi=\chi^{-1}\omega_\ell\)**

这是很多读者第一次读时最困惑的一步。直观看，\(\chi\) 已经是我们关心的 eigenspace label 了，为什么还要再扭一次 character？

答案是：**Herbrand-Ribet 型结论真正认识的对象，不是裸的 \(\chi\)，而是与 \(\ell\) 的 Teichmüller 数据正确耦合后的 odd character。**

于是作者设

$$
\psi=\chi^{-1}\omega_\ell,
$$

其中 \(\omega_\ell\) 是 Teichmüller character。

这一步的作用不是“制造一个更复杂的符号”，而是把：

```text
class-group eigenspace data
```

翻译成

```text
一个适合喂给 generalized Bernoulli numbers 的 primitive odd Dirichlet character
```

**Step 4：Lemma 3.3 在这条链里到底承担什么职责**

`Lemma 3.3` 并不直接给出 finite candidate set，但它把新构造的 \(\psi\) 整理成了可用输入。对我们的 notebook 来说，最关键的是三件事：

1. \(\psi\) 是 primitive。
2. \(\psi\) 是 odd。
3. \(\ell\)-divisibility 最终可以落到某个 rational norm 上。

更具体地说，论文在这一步里要确保：

- \(\chi\) 是 even，而 \(\omega_\ell(-1)=-1\)，所以
  \(\psi(-1)=-1\)，即 \(\psi\) 是 odd；
- \(\chi\) 的 conductor 来自 \(2^k\) 层，而 \(\omega_\ell\) 的 conductor 是 \(\ell\)；
  由于 \(\gcd(2^k,\ell)=1\)，二者相乘后 \(\psi\) 的 conductor 变成
  \(2^k\ell\)；
- 然后把 \(\chi\)-eigenspace 中的非零 \(\ell\)-torsion 与
  \(B_{1,\psi}\) 的整除信息连起来。

读者最容易卡住的是最后这点。最稳妥的理解方式是：

```text
Lemma 3.3 不是在独立证明一个新现象
它是在把 Theorem 3.2 需要的输入格式整理好
```

**Step 5：Theorem 3.2 真正输出了什么**

一旦有了 full-order \(\chi\) 和由它扭出来的 \(\psi\)，`Theorem 3.2` 才能接管：

- 若
  \(\operatorname{Cl}(K_k^+)[\ell]^{e_\chi}\neq0\)，
- 则
  \(\ell\) 必须整除与 \(B_{1,\psi}\) 相关的 rational norm。

教材化地说，`Theorem 3.2` 并不是在“再证明一次 class-group 非平凡”，而是在完成这条最关键的翻译：

```text
nonzero ell-torsion in one full-order eigenspace
    ->
ell divides a concrete rational integer built from B_{1,psi}
```

这正是整篇 paper 从抽象代数对象切到有限计算对象的最后闸门。

**Step 6：为什么这就足以推出 \(\ell\in S_k\)**

现在把前几步收束起来：

1. 从 `Step 1` 知道
   \(\ell>10^9\)。
2. 从 `Step 1` 知道
   \(\ell\equiv\pm1\pmod{2^{k-1}}\)。
3. 从 `Step 2 + Step 3 + Step 5` 知道：
   存在某个 full-order character \(\psi_j\)，使得
   \[
   \ell\mid
   \operatorname{N}_{\mathbb{Q}(\zeta_m)/\mathbb{Q}}(B_{1,\psi_j}).
   \]

而这三条，恰好就是 `Definition 3.2` 里 \(S_k\) 的三条定义条件。

所以并不是“作者声称坏素数应该进 \(S_k\)”；
而是 **每一条定义条件都已经被前面的 theorem chain 强迫出来了**。

最终结论就是：

$$
\ell\in S_k.
$$

**把这条 proof chain 压成一张图**

```text
assume ell | h_k^+
    ->
Theorem 3.1(i): ell > 10^9
    +
Theorem 3.1(ii): ell ≡ ±1 mod 2^(k-1)
    ->
Corollary 3.1: some full-order eigenspace e_chi is nonzero
    ->
set psi = chi^(-1) omega_ell
    ->
Lemma 3.3: psi is the right primitive odd character
    ->
Theorem 3.2: ell divides N(B_{1,psi})
    ->
Definition 3.2: ell satisfies all three candidate-set conditions
    ->
ell ∈ S_k
```

**Step 7：为什么 Proposition 3.2 还要再说“finite”和“computable”**

到目前为止，我们只证明了 “every bad odd prime enters \(S_k\)”。
但 `Proposition 3.2` 的 statement 还多了两件事：

- \(S_k\) 是 finite；
- \(S_k\) 是 computable。

这两点分别靠下面两条补上：

1. `finite`
   - full-order conjugate pairs 最多只有 \(n/4\) 个；
   - 每个对应的 norm 都是一个有显式上界的非零整数；
   - 一个有界非零整数只会有有限多个 prime divisors。
2. `computable`
   - \(B_{1,\psi}\) 可由 Definition 2.14 的显式求和计算；
   - norm 可用 resultant 或等价代数算法计算；
   - bounded integer 的 factorization 是可执行步骤。

所以 `Proposition 3.2` 的完整输出其实是三层：

```text
bad primes are all inside S_k
    +
S_k is finite
    +
S_k is effectively computable
```

只有这三层合在一起，`Algorithm 1` 才真正有意义。否则你最多得到“坏素数满足某个存在性描述”，还不能进入机器验证。

**这一节最容易误解的三个点**

1. \(\omega_\ell\) 不是多余装饰。
   它是把 eigenspace 信息翻译进 Bernoulli/Herbrand-Ribet 语言的必要扭曲。
2. `Theorem 3.2` 不是直接说 “\(\ell\mid h_k^+\Rightarrow \ell\mid N(B_{1,\chi})\)”。
   中间要先通过 full-order \(\chi\)，再构造 \(\psi=\chi^{-1}\omega_\ell\)。
3. `Definition 3.2` 不是“算法工程师临时定的过滤条件”。
   它其实是把前三条 theorem-level 必要条件打包成一个候选表定义。

**这一小节最该记住的教学结论**

1. `Proposition 3.2` 的真正作用，不是“又得到一个定理”，而是把坏素数从抽象 class-group 语言送进一个有限列表。
2. `Theorem 3.2` 是整篇 paper 从 proof world 进入 computation world 的最后一座桥。
3. 一旦你把这条 notebook 真正跟下来，`Algorithm 1` 的 Phase A 就不再像黑箱，而更像是把 theorem chain 直接改写成代码。

### 8.9 把 `Proposition 3.1 + Corollary 3.1` 改写成一个 layer-by-layer notebook

前面的 `8.2` 已经讲过 `Proposition 3.1` 的局部 annihilator 机制：同一个 \(N=1+\sigma^{n/2}\) 在 low-order eigenspace 上一边等于 \(2\)，一边等于 \(0\)。但如果读者要真正顺着 `Theorem 3.3` 往下走，还需要把 `Corollary 3.1` 的用法看成一个 **分层的 notebook**，而不是一句 “因此只剩 full-order characters”。

**这条链真正要解决的问题**

假设：

$$
\ell \mid h_k^+,
\qquad
k\in\{9,10,11,12\},
\qquad
\ell \text{ 是奇素数}.
$$

我们想从这里推出的，不是马上得到矛盾，而是先得到一个更精确的 statement：

> 若 \(\ell\)-torsion 真的存在，那么它一定落在某个 full-order character eigenspace 里。

这句话就是 `Corollary 3.1` 的真正工作内容。

**Step 1：为什么一开始就可以做 eigenspace decomposition**

`Corollary 3.1` 的 proof 一上来会用到一个看起来很“表示论”的事实：

$$
\operatorname{Cl}(K_k^+)[\ell] =
\bigoplus_{\chi\in\widehat{G_k}}
\operatorname{Cl}(K_k^+)[\ell]^{e_\chi}.
$$

教学上一定要先把这一步说白：

1. \(\ell\) 是奇素数；
2. \(G_k\cong \mathbb{Z}/2^{k-2}\mathbb{Z}\)，所以
   \(|G_k|=2^{k-2}\)；
3. 因而
   \(\ell\nmid |G_k|\)；
4. 所以 \(\mathbb{F}_\ell[G_k]\) 是 semisimple；
5. 因此 \(\ell\)-torsion class group 可以按 characters 完整分解。

这一步的作用是把“有 nontrivial \(\ell\)-torsion”翻译成：

```text
至少有一个 character eigenspace 非零
```

如果没有这一步，后面根本无从谈“排空 low-order，再留下 full-order”。

**Step 2：什么叫 non-full-order，其实就是 order dividing 2^(k-3)**

对本论文的 \(G_k\)，所有 character 的阶都是 \(2\) 的幂，因为
\(G_k\) 本身就是 cyclic \(2\)-group。

而 full-order 的定义是：

$$
\operatorname{ord}(\chi)=|G_k|=2^{k-2}.
$$

因此 **非 full-order** 并不神秘，它等价于：

$$
\operatorname{ord}(\chi)\mid 2^{k-3},
$$

也就是阶至多是最大真因子 \(2^{k-3}\)。

这正是 `Proposition 3.1` 的适用范围。它不是对“某些碰巧的低阶 character”起作用，而是正好覆盖了所有 non-full-order 情形。

**Step 3：Proposition 3.1 真正消掉了什么**

一旦你知道上一层已经满足
\(h_{k-1}^+=1\)，
`Proposition 3.1` 就说：

$$
\operatorname{Cl}(K_k^+)[\ell]^{e_\chi}=0
\quad
\text{for every }
\chi
\text{ with }
\operatorname{ord}(\chi)\mid 2^{k-3}.
$$

也就是说：

- trivial character 被杀掉；
- 所有 proper \(2\)-power orders 的 characters 都被杀掉；
- 任何还能承载 \(\ell\)-torsion 的位置，只可能是 full-order layer。

这一步是整篇 paper 中最标准的 “先把 support 缩到最薄的一层” 操作。

**Step 4：Corollary 3.1 为什么不是新 theorem，而是 bookkeeping theorem**

现在把 `Step 1` 和 `Step 3` 接起来：

1. 由 eigenspace decomposition，若 \(\ell\mid h_k^+\)，则
   \(\operatorname{Cl}(K_k^+)[\ell]\neq0\)。
2. 因而至少有一个 eigenspace
   \(\operatorname{Cl}(K_k^+)[\ell]^{e_\chi}\)
   非零。
3. 但 `Proposition 3.1` 已经把所有 non-full-order \(\chi\) 全部清空。
4. 所以剩下唯一可能的 \(\chi\)，只能是 full-order。

这就是 `Corollary 3.1` 的全部逻辑。

因此它不是在引入新的 arithmetic mechanism，而是在做一件非常关键的 bookkeeping：

```text
nontrivial ell-torsion exists
    ->
some eigenspace is nonzero
    ->
all non-full-order eigenspaces are already zero
    ->
a full-order eigenspace must be nonzero
```

**Step 5：为什么这一步对后文是刚需，而不是可选整理**

如果没有 `Corollary 3.1`，后面的 `Theorem 3.2` 和 `Definition 3.2` 就会遇到一个根本问题：

- 你不知道该给哪个 character 构造
  \(\psi=\chi^{-1}\omega_\ell\)；
- 你也不知道该看哪些 Bernoulli norms；
- 那么 Phase A 的 candidate-set construction 就没有输入。

所以 `Corollary 3.1` 的真正角色是：

> 它不是证明的终点，而是把 `Section 3.2` 的 tower argument 变成 `Section 3.3` 的输入接口。

**Step 6：按层写，k=9 到 k=12 实际上在做同一件事**

这一点论文原文其实已经明说了，但很容易被读者忽略。`Corollary 3.1` 在 `k=9,10,11,12` 上的使用模板完全一致，只是参数不同。

**k=9**

- 归纳输入：base case \(h_8^+=1\)。
- 群：
  \(G_9\cong \mathbb{Z}/128\mathbb{Z}\)。
- 被清空的所有阶：
  \(\{1,2,4,8,16,32,64\}\)。
- 唯一残留的阶：
  \(128\)。

因此若 \(\ell\mid h_9^+\)，必有某个 order-\(128\) 的 full-order eigenspace 非零。

**k=10**

- 归纳输入：已经先证明好的 \(h_9^+=1\)。
- 群：
  \(G_{10}\cong \mathbb{Z}/256\mathbb{Z}\)。
- 被清空的所有阶：
  所有整除 \(128\) 的 characters。
- 唯一残留的阶：
  \(256\)。

因此若 \(\ell\mid h_{10}^+\)，坏 torsion 只能躲在 order-\(256\) 的 eigenspace 里。

**k=11**

- 归纳输入：\(h_{10}^+=1\)。
- 群阶：
  \(512\)。
- 被清空的所有阶：
  所有整除 \(256\) 的 characters。
- 唯一残留的阶：
  \(512\)。

**k=12**

- 归纳输入：\(h_{11}^+=1\)。
- 群阶：
  \(1024\)。
- 被清空的所有阶：
  所有整除 \(512\) 的 characters。
- 唯一残留的阶：
  \(1024\)。

所以这四层不是四个不同 proof，而是同一个 proof schema 在更高参数上重复。

**Step 7：为什么这条归纳真的不循环**

读者在这里最常见的怀疑是：

> 你在 `k=10` 用了 \(h_9^+=1\)，而 \(h_9^+=1\) 又来自主定理本身，这会不会是循环？

正确答案是：不会。因为 `Theorem 3.3` 在写法上是一个 tower induction，但在执行上是 **严格向前** 的：

1. 先独立完成 `k=9`；
2. 再把 `k=9` 的结论喂给 `k=10`；
3. 再把 `k=10` 的结论喂给 `k=11`；
4. 最后把 `k=11` 的结论喂给 `k=12`。

每一步里，`Corollary 3.1` 只负责把 torsion support 压到 full-order layer；
真正让这一层结束的，还得靠 `Section 3.3` 的 finite candidate set 和最终 residue elimination。

所以归纳链的角色是：

```text
它不替你完成本层 computation
它只替你保证：
本层不需要再看低阶 characters
```

**把整条链压成一张图**

```text
assume ell | h_k^+
    ->
Cl(K_k^+)[ell] != 0
    ->
Maschke decomposition over F_ell[G_k]
    ->
some eigenspace e_chi is nonzero
    ->
Proposition 3.1 kills every chi with ord(chi) | 2^(k-3)
    ->
the surviving chi must have full order 2^(k-2)
    ->
Corollary 3.1
    ->
Section 3.3 can now attach psi = chi^(-1) omega_ell
```

**这一小节最该记住的教学结论**

1. `Proposition 3.1` 负责“清空 low-order layer”，`Corollary 3.1` 负责“证明坏 torsion 必落在 full-order layer”。
2. 这一步真正引入的不是新数论，而是把 class-group torsion 的 support 重新组织成后续 Herbrand bridge 可处理的输入。
3. 一旦你把这条 layer-by-layer notebook internalize，`Section 3.2` 就不再像一个抽象的 tower trick，而会变成 `Section 3.3` 的必要前处理。

### 8.10 把 `Definition 3.1 + Lemma 3.1 + Theorem 3.1` 改写成一个 residue-side notebook

前面的 `8.1` 给的是概览版读法；这里补的是更接近 proof notebook 的版本。要抓住的不是“又出现一个新筛法公式”，而是下面这条局部到全局的桥：

```text
在每个 residue field 里检查 xi_5 的 2^(k-1)-power behavior
    ->
得到一个局部 obstruction
    ->
由 Fukuda-Komatsu + Sinnott 的桥推出 ell 不可能整除 unit index
    ->
所以 ell 不可能整除 h_k^+
```

**Step 1：先把局部对象和全局对象分开**

`3.1` 同时在说两件不同层面的事：

1. 全局目标是
   $$
   h_k^+ = [\mathcal{O}_{K_k^+}^{\times}:C^+].
   $$
2. 局部输入是：固定一个奇素数 \(\ell\)，再固定一个上方素理想 \(\mathfrak l\mid\ell\)，考察 residue field
   \(\kappa(\mathfrak l)=\mathcal{O}_{K_k^+}/\mathfrak l\)
   里 \(\xi_5\) 的行为。

最稳定的局部量是
$$
d=\operatorname{ord}_{\mathfrak l}(\xi_5).
$$

因为 \(\kappa(\mathfrak l)^\times\) 是一个有限循环群，阶为
\(N(\mathfrak l)-1\)，所以只要给定任何整数 \(E\)，都有：

$$
\xi_5^E \equiv 1 \pmod{\mathfrak l}
\qquad\Longleftrightarrow\qquad
d \mid E.
$$

因此 `3.1` 里真正反复使用的结构其实很简单：

```text
若某个坏素数 ell 真的存在
    ->
Fukuda-Komatsu 的论证会逼出 d | E
    ->
于是 xi_5^E ≡ 1 mod l

而 sieve 想要验证的正好是 xi_5^E ≠ 1 mod l
```

矛盾一旦建立，\(\ell\) 就被排除了。

**Step 2：Definition 3.1 不要读成“怪指数”，要读成“xi_5 不能太早塌缩”**

第一次读 Definition 3.1，读者很容易把注意力全放在 Eq. (18) 的表面指数上。教材化地读，更稳妥的方式是：

- `Definition 3.1` 的核心不是某一行指数长什么样，而是它在测试：
  \(\xi_5\) 在每个 \(\kappa(\mathfrak l)^\times\) 里，是否会“过早地”落进一个太小的子群。
- proof 里后来又引入
  \(\operatorname{ord}_\ell(5)\)。
  这个量不是 \(d\) 的替身，而是用来记录 \(\ell\) 在 \(2\)-power cyclotomic tower 里的 Frobenius 位置，从而比较“坏情况指数”和“真正要测的指数”。
- 真正负责产生矛盾的永远是：
  某个 divisor relation 最终迫使
  \(d\mid E\)，
  然后推出
  \(\xi_5^E\equiv1\pmod{\mathfrak l}\)。

所以这一小节里最该记住的不是单条公式，而是：

> 所有局部筛法最终都落回“\(\xi_5\) 在 residue field 里的阶 \(d\) 是否整除测试指数”。

**Step 3：为什么 \(\ell\equiv\pm1\pmod{2^{k-1}}\) 会把问题压到 \(f\le2\)**

令
$$
G_k = \operatorname{Gal}(K_k^+/\mathbb{Q}) \cong \mathbb{Z}/2^{k-2}\mathbb{Z},
$$
Frobenius 元素 \(\operatorname{Frob}_\ell\) 在这个群里的阶，就是 residue degree
\(f=[\kappa(\mathfrak l):\mathbb{F}_\ell]\)。

于是：

1. 若
   \(\ell\equiv1\pmod{2^{k-1}}\)，
   那么它在 \(G_k\) 里的像就是单位元，所以
   \(f=1\)。
2. 若
   \(\ell\equiv-1\pmod{2^{k-1}}\)，
   那么
   \(\ell^2\equiv1\pmod{2^{k-2}}\)，
   因而 \(\operatorname{Frob}_\ell\) 的阶至多为 \(2\)，于是
   \(f=2\)。

这一步极其关键，因为一旦 \(f\le2\)，就有
$$
N(\mathfrak l)=\ell^f,
$$
于是测试指数会坍缩成非常显式的量。

**Step 4：真正该测的指数是什么**

对教学来说，最好把 Eq. (19) 和 Eq. (20) 一起读。更 invariant 的写法是：

$$
E_{\mathfrak l} = \frac{N(\mathfrak l)-1}{2^{k-1}}.
$$

在 \(f\le2\) 时，这个指数具体变成：

1. 若 \(f=1\)，
   $$
   E_{\mathfrak l} = \frac{\ell-1}{2^{k-1}}.
   $$
2. 若 \(f=2\)，
   $$
   E_{\mathfrak l} = \frac{\ell^2-1}{2^{k-1}}.
   $$

而
$$
\xi_5^{E_{\mathfrak l}} \pmod{\mathfrak l}
$$
正是在检查 \(\xi_5\) 的
\(2^{k-1}\)-power residue symbol 是否非平凡。

因此 `Section 3.1` 的 residue test 不是“随便幂一下看结果”，而是在问：

```text
xi_5 在 kappa(l)^× 里
是否落进了 2^(k-1)-th powers 这个禁止子群
```

**Step 5：Lemma 3.1 的 contradiction 到底怎么发生**

现在固定一个 \(\mathfrak l\mid\ell\)，并设
\(d=\operatorname{ord}_{\mathfrak l}(\xi_5)\)。
proof 的骨架可以压成下面两种情形。

**情形 A：f=1**

这对应
\(\ell\equiv1\pmod{2^{k-1}}\)。
此时 residue field 就是 \(\mathbb{F}_\ell\)，测试指数是
\((\ell-1)/2^{k-1}\)。

Fukuda-Komatsu 的失败情形会迫使 \(d\) 去整除一个比这更“坏”的辅助指数；论文引入
\(\operatorname{ord}_\ell(5)\)
正是为了完成这一步比较。对读者真正重要的结论只有一句：

$$
d \mid \frac{\ell-1}{2^{k-1}}
\qquad\Longrightarrow\qquad
\xi_5^{(\ell-1)/2^{k-1}} \equiv 1 \pmod{\mathfrak l}.
$$

但这和要验证的局部测试
\(\xi_5^{(\ell-1)/2^{k-1}}\not\equiv1\pmod{\mathfrak l}\)
正面冲突。

**情形 B：f=2**

这对应
\(\ell\equiv-1\pmod{2^{k-1}}\)。
此时 residue field 的规模是
\(N(\mathfrak l)=\ell^2\)，
测试指数要改成
\((\ell^2-1)/2^{k-1}\)。

proof 的本质仍然一样：

1. 因为 \(d\mid|\kappa(\mathfrak l)^\times|=\ell^2-1\)；
2. 又因为
   \(2^{k-1}\mid(\ell+1)\)；
3. 坏情形最终会迫使
   \(d\mid(\ell^2-1)/2^{k-1}\)；
4. 所以
   $$
   \xi_5^{(\ell^2-1)/2^{k-1}} \equiv 1 \pmod{\mathfrak l},
   $$
   这再次和局部 nontriviality test 冲突。

所以 `Lemma 3.1` 真正做的事不是“算一个神秘幂”，而是：

```text
坏素数若存在
    ->
xi_5 的 residue order d 必须整除测试指数
    ->
测试值就会变成 1

而我们验证的正是“测试值不等于 1”
```

**Step 6：为什么局部 nontriviality 足以推出 \(\ell\nmid h_k^+\)**

这里 paper 并没有从零重证一个新的 class-number 定理，而是调用了两条已有桥：

1. Fukuda-Komatsu 的命题把
   \(\xi_5\) 的 nontrivial power-residue behavior
   与
   \(\ell\nmid[\mathcal{O}_{K_k^+}^\times:C^+]\)
   连接起来；
2. Sinnott 公式给出
   $$
   h_k^+ = [\mathcal{O}_{K_k^+}^\times:C^+].
   $$

所以 `Lemma 3.1` 的逻辑出口其实是：

```text
对每个 l | ell
    xi_5 的 2^(k-1)-power residue symbol 都非平凡
    ->
Fukuda-Komatsu bridge
    ->
ell ∤ [O_{K_k^+}^× : C^+]
    ->
ell ∤ h_k^+
```

这也是为什么 `3.1` 虽然完全没谈 characters、Bernoulli numbers、Herbrand-Ribet，却已经能先砍掉一大批奇素数。

**Step 7：Theorem 3.1 到底产出了什么**

有了上面的 notebook，再看 `Theorem 3.1` 就不会把它误读成“纯数值筛法”。

**Part (i)**

- 对 \(k\le12\)，把所有
  \(\ell<10^9\)
  的候选奇素数做掉。
- 这一步是 computation-heavy，但它验证的对象非常明确：
  就是前面那套 residue-side test。

**Part (ii)**

- 任何真能整除 \(h_k^+\) 的奇素数，都必须满足
  \(\ell\equiv\pm1\pmod{2^{k-1}}\)。
- 为什么？因为若
  \(\ell\not\equiv\pm1\pmod{2^{k-1}}\)，
  `Lemma 3.2` 给出
  \(f\ge4\)；
  而 Fukuda-Komatsu 的结果正是：一旦 Frobenius 阶已经这么大，\(\xi_5\) 的局部行为就不允许 \(\ell\) 去整除这个 unit index。

因此 `Theorem 3.1` 的真正产出不是一句筛法口号，而是两层信息：

1. 一个已经跑完的小素数清空结果；
2. 一个把幸存坏素数压进极稀薄同余类的结构性必要条件。

**把这条 residue-side chain 压成一张图**

```text
fix ell and l | ell
    ->
look at xi_5 in kappa(l)^×
    ->
set d = ord_l(xi_5)
    ->
if local test is nontrivial, then d does not divide the forbidden exponent
    ->
if ell were bad, Fukuda-Komatsu would force d to divide that exponent
    ->
contradiction
    ->
ell ∤ [O_{K_k^+}^× : C^+]
    ->
ell ∤ h_k^+
```

**这一小节最该记住的教学结论**

1. `Section 3.1` 不是“先做一个 heuristic sieve”，而是把 class number 问题先转成了一个非常局部的 residue obstruction。
2. 在 proof notebook 里，真正反复出现的核心量不是某条长指数，而是
   \(d=\operatorname{ord}_{\mathfrak l}(\xi_5)\)。
3. `9.8` 的 `k=5,\ell=97` toy walkthrough，本质上就是把这里的 `Step 4` 到 `Step 6` 真正落到一个具体 residue field 上跑一遍。

## 9. 算法与计算：为什么这份证明是 unconditional 的

这篇论文的关键不只是 “存在一个 finite set”，而是作者把剩余工作收束成了可执行算法。

原文 `Algorithm 1` 的逻辑可以压成两相：

### 9.1 Phase A：先算候选素数

输入：

- \(k \ge 9\)；
- 已知 \(h_{k-1}^+ = 1\)。

主流程：

1. 设 \(n = 2^{k-2}\)。
2. 选一个 full-order character 代表 \(j\)。
3. 计算 generalized Bernoulli number \(B_{1,\psi_j}\)。
4. 计算其 norm \(\mathrm{N}(B_{1,\psi_j})\)。
5. 分解这个整数，保留满足 sieve 和 congruence filter 的奇素数。
6. 得到候选集 \(S_k\)。

作者在 `3.5 Optimizations` 里还进一步说明：实际只需 **一次** 整数分解，而不是对 \(n/4\) 个 character pair 全做一遍。

### 9.2 Phase B：逐个验证候选素数

对每个 \(\ell \in S_k\)，测试对应的 Wieferich-style 条件。如果对所有 \(\mathfrak l \mid \ell\) 都通过，则 \(\ell \nmid h_k^+\)。

所以整个闭环是：

```text
抽象 class-group 风险
    ->
full-order eigenspace
    ->
Bernoulli norm divisibility
    ->
finite candidate primes
    ->
explicit residue test on xi_5
```

### 9.3 原文给出的复杂度信息

根据 `3.4`：

- Phase A 中 Bernoulli 数和 norms 的算术成本大约是 \(\tilde O(n^2)\)。
- 实际瓶颈是整数分解：
  论文说实际遇到的是至多 \(143\) 位整数，worst-case bound 来自 Lemma 3.4 可到 \(309\) 位。
- Phase B 对每个 prime 的代价大约是
  \(O(n \log n \cdot \log^2 \ell)\)，可借助 NTT 完成。

这也是为什么作者敢把“unconditional verification”写成一个现实可执行而非纯存在性的结论。

### 9.4 教材化伪代码：把 Algorithm 1 翻译成人能跟的流程

```text
Input:
    integer k >= 9
    previously established fact h_{k-1}^+ = 1

Output:
    True  (if h_k^+ = 1 is verified)
    or a residual set of unresolved odd primes

Procedure:
1. Set n <- 2^(k-2).
2. Enumerate representatives of conjugate pairs of full-order characters.
3. For each representative psi_j:
   compute B_{1,psi_j} and the integer N_j = |N(B_{1,psi_j})|.
4. Collect odd prime divisors of the N_j into a raw candidate set.
5. Apply the Stage-1 sieve:
   remove every ell <= 10^9,
   remove every ell not congruent to +-1 mod 2^(k-1).
6. For each surviving ell:
   for each prime ideal l | ell,
   compute xi_5^((ell-1)/2^(k-1)) mod l.
   If every residue is != 1, discard ell.
   Otherwise keep ell as unresolved.
7. If no unresolved ell remain, conclude h_k^+ = 1.
```

这里最重要的理解不是代码细节，而是两种不同的“缩减”在交替工作：

- Phase A 用 Herbrand bridge 把抽象 torsion 风险压到有限候选素数上。
- Phase B 用 Wieferich-style residue test 把每个候选素数做最终消元。

### 9.5 从定义手算一个 \(B_{1,\psi}\)：`k=5` 的超小 toy case

前面的伪代码仍然有一个教学缺口：读者知道要“算 generalized Bernoulli number”，但不知道这一步真正长什么样。先做一个完全缩小版的例子。

取 conductor \(f=2^5=32\)，并选一个 primitive odd full-order Dirichlet character \(\psi\) 满足：

$$
\psi(31)=-1, \qquad \psi(5)=\zeta_8.
$$

这里 “full-order” 的意思是 \(\psi\) 在 \(\langle 5\rangle\subset (\mathbb{Z}/32\mathbb{Z})^\times\) 上的阶正好是 \(8\)。由于 \(\psi\) 是奇 character，\(B_{1,\psi}\) 不会像 even nontrivial character 那样直接退化成 \(0\)。

按 Definition 2.14，

$$
B_{1,\psi}=\frac{1}{32}\sum_{\substack{1\le a\le 32\\ \gcd(a,32)=1}}\psi(a)a.
$$

把 \(32\) 以内与 \(32\) 互素的奇数全部代进去，Sage 验证得到：

$$
\sum_{\gcd(a,32)=1}\psi(a)a = 32\zeta_8^3+32\zeta_8^2-32\zeta_8-32,
$$

因此

$$
B_{1,\psi}=\zeta_8^3+\zeta_8^2-\zeta_8-1.
$$

这个结果有两个非常重要的教学意义。

第一，它说明 generalized Bernoulli number 不是“黑箱常数”，而是一个按 character 加权后的显式和。

第二，它已经能让我们演示 Stage A 里最关键的一步：取 rational norm。对上面的 \(B_{1,\psi}\)，最小多项式与范数分别是

\[
\operatorname{minpoly}(B_{1,\psi}) = x^4 + 4x^3 + 4x^2 + 8.
\]

\[
\operatorname{N}_{\mathbb{Q}(\zeta_8)/\mathbb{Q}}(B_{1,\psi}) = 8 = 2^3.
\]

于是这个 toy case 的 odd prime divisor 集合为空。换句话说，哪怕还没用到 paper 里的大规模 sieve，你也已经看到：

```text
primitive odd character
    ->
explicit B_{1,psi}
    ->
rational integer norm
    ->
prime divisors to inspect
```

这就是 `3.3` 的计算主线最小原型。

### 9.6 一个更像论文的 Stage-A toy walkthrough：`k=7`

上一个例子太小，导致 norm 只有 \(2\)-part。为了真正看到 “先因式分解，再筛候选 prime” 的 workflow，再看一层更接近论文气质的例子。

取 conductor \(f=2^7=128\)，再选一个 primitive odd full-order character \(\psi\) 满足

$$
\psi(127)=-1, \qquad \psi(5)=\zeta_{32}.
$$

SageMath 可以直接把 \(B_{1,\psi}\)、它的 rational norm 以及素因子分解算出来：

```python
G = DirichletGroup(128)
psi = [
    chi for chi in G
    if chi.multiplicative_order() == 32
    and chi.is_odd()
    and chi.conductor() == 128
][0]

B = psi.bernoulli(1)
N = abs(B.norm())

print(B)
print(N)
print(factor(N))
```

本地验证输出的关键信息是：

$$
\operatorname{N}(B_{1,\psi}) = 692092928 = 2^{15}\cdot 21121.
$$

这时就能把 Phase A 真的走一遍：

1. raw prime divisors 是 \(\{2,21121\}\)。
2. 论文关心 odd prime divisors，所以先丢掉 \(2\)。
3. 剩下的 odd prime 只有 \(21121\)。
4. 对 `k=7` 的 congruence filter，要检查
   \(\ell \equiv \pm 1 \pmod{2^{k-1}} = \pmod{64}\)。
5. 这里
   \(21121 \equiv 1 \pmod{64}\)，所以它通过同余筛。
6. 但它仍然满足
   \(21121 < 10^9\)，因此会被 Theorem 3.1(i) 的 cutoff 直接删除。

于是这个 toy example 的 candidate set 仍然是空集：

$$
S_7^{\mathrm{toy}} = \varnothing.
$$

这个例子比 `k=5` 更贴近原论文真正的 workflow，因为它已经包含了：

- 非平凡 odd prime factor；
- congruence filter；
- cutoff filter；
- 最终候选集清空。

你也可以把它和论文在 `k\le 7` 的现象对应起来看：在小层数上，Stage A 往往已经足够把 candidate set 压空，真正变难的是 `k=9,10,11,12` 这些密码学相关层数。

### 9.7 一个更接近原论文的 Stage-A 重算：合并 `k=9` 的全部 full-order characters

前两个例子都停在小层数。为了让读者真正看到“为什么 paper-scale computation 仍然是可落地的”，这里直接把 `k=9` 的 full-order primitive odd characters 全部扫一遍，做一版 **merged Stage-A recomputation**。

取 \(k=9\)，于是 conductor 是 \(f=2^9=512\)，full-order 的大小是

$$
m = 2^{k-2} = 128.
$$

先从单个 full-order primitive odd character \(\psi\) 开始，取

$$
\psi(511)=-1, \qquad \psi(5)=\zeta_{128}.
$$

对这个单样例，本地 Sage 验证可直接算出它的 Bernoulli number norm：

```python
G = DirichletGroup(512)
psi = [
    chi for chi in G
    if chi.multiplicative_order() == 128
    and chi.is_odd()
    and chi.conductor() == 512
][0]

B = psi.bernoulli(1)
N = abs(B.norm())

print(len(str(N)))
print(N)
print(factor(N))
```

关键输出是：

$$
\operatorname{N}(B_{1,\psi}) =
5527622451448555262320005717721937139739376956982951936 =
2^{63}\cdot 76532353\cdot 7830753969553468937988617089.
$$

这一层先告诉你：单个 paper-scale norm 确实已经会长成一个几十位的大整数。

但真正更关键的，不是这个单样例，而是把全部 full-order characters 合并后会发生什么。对 `k=9`，本地 Sage 验证 primitive odd full-order characters 一共有 \(64\) 个，也就是 \(32\) 对共轭代表：

```python
G = DirichletGroup(512)
chars = [
    chi for chi in G
    if chi.multiplicative_order() == 128
    and chi.is_odd()
    and chi.conductor() == 512
]
print(len(chars))      # 64
print(len(chars) // 2) # 32 conjugate-pair reps
```

接着把每个 character 的 \(B_{1,\psi}\) norm 全算出来。这里出现了一个非常漂亮、也非常适合讲义解释的现象：

- 这 \(64\) 个 characters 给出的 rational norms **全部相同**。
- 因此 merged raw set 的 odd prime divisors，和上面单样例的 odd prime divisors 完全一致。

可以直接用下面的脚本验证：

```python
from collections import Counter

norms = []
odd_primes = set()
for chi in chars:
    N = abs(chi.bernoulli(1).norm())
    norms.append(int(N))
    for p, e in factor(N):
        p = int(p)
        if p % 2 == 1:
            odd_primes.add(p)

print(len(set(norms)))        # 1
print(sorted(odd_primes))     # [76532353, 7830753969553468937988617089]
```

这一步有两个非常重要的教学价值。

第一，它把 `3.5 Optimizations` 里“为什么一次分解就够”的直觉落到了一个真实层数上。对 `k=9` 的这次本地重算而言，不只是共轭 pair 可以合并，连全部 \(64\) 个 primitive odd full-order characters 都落到了同一个 rational norm 上，因此你只需要 factor **一个** \(55\) 位整数。

第二，它能让你直接看到 merged Stage A 的两道必要筛如何工作。对 \(k=9\)，Theorem 3.1 的 congruence filter 是

$$
\ell \equiv \pm 1 \pmod{2^{k-1}} = \pmod{256}.
$$

而 merged raw set 里的两个 odd factors 表现分别是：

- \(76532353 < 10^9\)，因此先被 cutoff 删掉。
- \(76532353 \equiv 129 \pmod{256}\)，所以它本来也不满足同余筛。
- \(7830753969553468937988617089 > 10^9\)，但
  \(7830753969553468937988617089 \equiv 129 \pmod{256}\)，同样不满足
  \(\pm 1 \pmod{256}\)。

因此，从这次 **完整 `k=9` full-order sweep** 产生的 odd prime divisors 里，没有任何一个能穿过 Stage-A filters。换句话说，在这次本地 merged recomputation 中，

$$
S_9^{\mathrm{Stage\ A,\ local}} = \varnothing.
$$

**如何理解这件事**

- 从计算角度看，它说明 `k=9` 的 full-order Bernoulli norms 在本地重算里并没有留下任何 odd prime survivor。
- 从讲义角度看，它第一次把 paper-scale Stage A 变成了一个读者真能跑出来的实验，而不是只看 theorem statement。
- 从 proof 结构看，它也解释了为什么作者在 `3.5` 里如此强调 orbit/conjugacy reduction 与“只做一次分解”。

**但为什么这还不等于“我已经重证了 Theorem 3.3”**

因为严格按论文结构，主定理还要用到另外两层逻辑：

1. `3.2` 的 low-order eigenspace elimination，保证真正只需看 full-order layer。
2. Phase B 的 residue test，哪怕在这次 `k=9` 本地重算中它已经变成 vacuous step，也仍然属于 theorem 的正式闭环。

所以这一小节最准确的定位是：

```text
不是完整复现 Theorem 3.3
而是把 k=9 的 full-order Stage-A merged workflow
真正跑出来一次：
枚举 -> 取 norm -> 分解 -> 合并 odd primes -> 应用筛法 -> candidate set clears
```

### 9.8 一个真正的 Phase-B toy walkthrough：`k=5, \ell=97`

上面的 `k=9` merged recomputation 已经把 Stage A 跑得很像论文了，但还没有真正展示 Phase B 的 residue elimination 长什么样。这里补一个更小、但完全可执行的例子。

取 `k=5`，于是 Theorem 3.1 / Lemma 3.1 的 congruence modulus 是

$$
2^{k-1} = 16.
$$

选一个满足同余条件的素数

$$
\ell = 97 \equiv 1 \pmod{16}.
$$

这使得 Lemma 3.1 里的测试指数变成

$$
\frac{\ell-1}{2^{k-1}} = \frac{96}{16} = 6.
$$

现在在 real cyclotomic subfield
\(K_5^+=\mathbb{Q}(\zeta_{32}+\zeta_{32}^{-1})\)
里取
\(\xi_5\)，并把它约化到每个 \(\mathfrak l \mid 97\) 的 residue field 中。下面的 Sage 代码可以直接做这件事：

```python
K = CyclotomicField(32)
Kplus, emb = K.maximal_totally_real_subfield()
z = K.gen()
xi5 = emb.preimage((z^5 - z^-5) / (z - z^-1))

exp = (97 - 1) // 16
for P in Kplus.primes_above(97):
    kP = P.residue_field()
    val = kP(xi5)^exp
    print(P, val, val == 1)
```

本地验证结果是：

```text
exp = 6
(97, zeta320 + 12)  -> 33   != 1
(97, zeta320 + 17)  -> 50   != 1
(97, zeta320 + 32)  -> 85   != 1
(97, zeta320 + 43)  -> 8    != 1
(97, zeta320 - 43)  -> 8    != 1
(97, zeta320 - 32)  -> 85   != 1
(97, zeta320 - 17)  -> 50   != 1
(97, zeta320 - 12)  -> 33   != 1
```

也就是说，对 `k=5` 的这个 didactic prime，所有 \(\mathfrak l \mid 97\) 都满足

$$
\xi_5^{(\ell-1)/16} \not\equiv 1 \pmod{\mathfrak l}.
$$

因此按 Lemma 3.1 的逻辑，可以直接推出

$$
97 \nmid h_5^+.
$$

**这一小节的教学价值**

- 它第一次把 Phase B 从 “一句 residue test” 变成了真正逐 prime-ideal 检验的过程。
- 它也解释了为什么论文里必须写 “for every \(\mathfrak l \mid \ell\)”：你不是只看一个 residue，而是要把 \(\ell\) 上方所有 prime ideals 都验过。
- 在这个 toy example 里，Stage A 并不重要，重点是让你看清 Phase B 的输入输出格式：

```text
Input:
    one surviving or artificially chosen prime ell
    a field K_k^+
    the cyclotomic unit xi_5

Work:
    factor ell in O_{K_k^+}
    reduce xi_5 modulo each prime ideal above ell
    test the required exponent

Output:
    either some residue is 1 and ell remains unresolved
    or every residue is != 1 and ell is eliminated
```

这也补上了前面 `9.7` 还没展示的最后一跳：**Stage A 负责产生或清空 candidate set，Stage B 负责对残余 prime 做 prime-ideal level 的最终逐点消元。**

### 9.9 把 `k=9` 的验证闭环写成一页 notebook

到这里为止，讲义里已经分别有：

- `3.2` 的 full-order reduction 解释；
- `k=9` 的 merged Stage-A recomputation；
- 一个真正跑通的 Phase-B toy residue test。

还差最后一步：把这些零件按 Theorem 3.3 的顺序拼起来，写成一页能顺着读下去的 `k=9` notebook。

**输入**

1. base case：Miller 已知 \(h_8^+=1\)。
2. 已知 \(2\nmid h_9^+\)。
3. `3.2` 告诉我们：若 odd prime \(\ell\mid h_9^+\)，则非平凡 \(\ell\)-torsion 只能落在 full-order characters 上。
4. 对 `k=9`，有
   \(G_9\cong \mathbb{Z}/128\mathbb{Z}\)，所以 low-order characters 的阶都整除 \(64\)，full-order characters 的阶是 \(128\)。

**Stage A：把 odd prime candidates 压空**

1. 枚举全部 primitive odd full-order characters。
   本地验证：一共有 \(64\) 个，即 \(32\) 对共轭代表。
2. 对每个 character 计算 \(B_{1,\psi}\) 的 rational norm。
3. 本地验证：这 \(64\) 个 characters 给出的 norm 全部相同。
4. 该共同 norm 的 odd prime divisors 只有
   \(76532353\) 和
   \(7830753969553468937988617089\)。
5. 对 `k=9` 的 cutoff 与 congruence filter：
   - \(76532353 < 10^9\)，直接删除；
   - \(76532353 \equiv 129 \pmod{256}\)，本来也不满足 \(\pm 1 \pmod{256}\)；
   - \(7830753969553468937988617089 > 10^9\)，但
     \(7830753969553468937988617089 \equiv 129 \pmod{256}\)，仍不满足 \(\pm 1 \pmod{256}\)。
6. 因此 merged Stage-A candidate set 已经是空集：

$$
S_9^{\mathrm{Stage\ A,\ local}} = \varnothing.
$$

**Stage B：为什么这一步在 k=9 本地 notebook 里变成 vacuous**

- Theorem 3.3 的正式算法仍然包含 residue elimination。
- 但在这次 `k=9` 的本地 merged recomputation 里，Stage A 已经没有留下任何 odd candidate prime。
- 因此 Stage B 没有输入对象，也就变成了 vacuous step。

**结论是怎样闭环的**

1. \(2\nmid h_9^+\)。
2. 没有 odd prime 能通过 full-order Stage-A candidate generation。
3. 所以没有任何 prime 能整除 \(h_9^+\)。
4. 因而

$$
h_9^+ = 1.
$$

把这页 notebook 压成一张图，就是：

```text
base case h_8^+ = 1
    ->
Proposition 3.1 / Corollary 3.1
    -> only order-128 characters remain
    ->
compute all 64 primitive odd full-order character norms
    ->
merged odd prime set = {76532353, 7830753969553468937988617089}
    ->
cutoff + congruence filters
    ->
empty Stage-A candidate set
    ->
no odd prime divisors remain
    +
2-part already excluded
    ->
h_9^+ = 1
```

这不是对整篇论文的替代品，但它已经足够接近一个真正的 theorem notebook：读者现在可以不只“相信 Theorem 3.3”，而是至少能顺着一个具体层数把证明链走完一遍。

#### `k=10` 的规模预告：为什么后两层开始明显更贵

为了避免读者误以为 `k=9` 的空候选集意味着后面各层也同样轻松，补一个本地尺度信号。

对 `k=10` 取一个 primitive odd full-order character，Sage 本地可算出其 \(B_{1,\psi}\) rational norm 是一个 **130 位** 整数。也就是说，仅仅把层数从 `9` 提到 `10`，单个 character 的 norm 就从 `55` 位跃迁到 `130` 位。

更关键的是：在这台机器上，直接对这个 `130` 位整数调用朴素的 `factor(N)`，在几十秒内没有完成，我手动中断了计算。这件事与论文 `3.4-3.5` 的叙事正好吻合：

- Bernoulli number 与 norm 的构造本身还可做；
- 真正的现实瓶颈迅速转向 **大整数分解**；
- 这也是为什么 orbit reduction、一次分解、GCD tricks 与 NTT 风格优化在 `k=10,11,12` 上不再只是锦上添花。

所以对读者而言，一个健康的理解顺序是：

1. 先在 `k=5`、`k=7` 看清每一步在做什么。
2. 再在 `k=9` 看一遍接近主定理的完整 notebook。
3. 最后把 `k=10` 视作“规模开始真实咬人”的门槛层。

### 9.10 把 `k=9,10,11,12` 的 paper-scale verification 写成一张核对表

到这里，讲义里已经有两个层次的例子：

- `k=5`、`k=7` 这类教学级 toy cases；
- `k=9` 这类已经接近主定理的 local notebook。

但读者通常还会卡在最后一个问题上：

> 真正按论文去验证 `k=9,10,11,12` 时，每一层到底还剩哪些事要做？

最有效的办法不是继续堆结论，而是把 Theorem 3.3 的 **paper-scale verification contract** 写成一张分层核对表。

**先把“哪些东西是一次性证明的，哪些东西是逐层重算的”分开**

读原文时，最好把所有工作分成三类：

1. **一次性证明的结构事实**
   - `Theorem 3.1`：给 cutoff 与同余筛；
   - `Proposition 3.1 / Corollary 3.1`：把 low-order characters 全部消掉；
   - `Theorem 3.2 / Proposition 3.2`：把残余风险压进有限候选集；
   - `Proposition 3.3 / Corollary 3.2`：说明每层只需 factor 一个 shared norm。
2. **每一层都要重算的量**
   - 该层的一个 full-order Bernoulli shared norm；
   - 该 norm 的 odd prime factors；
   - sieve 后得到的 candidate set \(S_k\)。
3. **每个 surviving prime 都要重做的测试**
   - 对每个 \(\ell \in S_k\)，对每个 \(\mathfrak l\mid \ell\) 检验
     \(\xi_5^{(\ell-1)/2^{k-1}} \bmod \mathfrak l\)。

这三类东西一旦分开，Theorem 3.3 的阅读难度会明显下降，因为你会看到：

```text
数学结构并没有在每一层重新发明
真正随层数变化的
只是参数、shared norm 的大小、以及 residue tests 的 bookkeeping
```

**统一模板：每一层实际都在执行同一张 checklist**

对 \(k\in\{9,10,11,12\}\)，paper-level verification 都可以压成下面七步：

1. 拿到上一层结论 \(h_{k-1}^+=1\) 作为归纳输入。
2. 先单独记住 \(2\nmid h_k^+\) 已由旧结果处理。
3. 用 `Proposition 3.1 / Corollary 3.1` 消掉所有 low-order eigenspaces，只保留 full-order characters。
4. 用 `Proposition 3.3 / Corollary 3.2` 把 Phase A 压到 **一个** shared norm \(N_k\)。
5. factor \(N_k\)，只保留满足
   \(\ell>10^9\) 且 \(\ell\equiv \pm1\pmod{2^{k-1}}\) 的 odd primes。
6. 对 surviving \(\ell\) 做 prime-ideal level 的 residue elimination。
7. 若所有 \(\ell\) 都被删掉，则 odd part 为空，再结合 \(2\nmid h_k^+\)，推出 \(h_k^+=1\)。

如果你把这七步背下来，再去看原论文 `3.4`，就不会再把 Theorem 3.3 误读成“突然跳到计算机结果”。

**按层展开：四层真正变化的只是参数**

下面这张表把论文 `Table 2` 与前面的 proof skeleton 合并在一起看：

| 层 | \(n=[K_k^+:\mathbb{Q}]\) | full-order 阶 | 合并前 conjugate pairs | 需 factor 的 shared norm 最大位数 | 论文给出的量级 |
|---|---:|---:|---:|---:|---|
| \(k=9\) | \(128\) | \(128\) | \(32\) | \(63\) 位 | 分钟级，ECM |
| \(k=10\) | \(256\) | \(256\) | \(64\) | \(143\) 位 | 小时级，ECM |
| \(k=11\) | \(512\) | \(512\) | \(128\) | \(325\) 位 | 天级，ECM/NFS |
| \(k=12\) | \(1024\) | \(1024\) | \(256\) | \(726\) 位 | 周级，NFS 或 Euler-system 路线 |

这张表最该教给读者的，不是“机器跑多久”，而是两个更本质的判断：

1. **概念难度几乎没有随层数上升。**
   - `k=10` 不是引入了新数学；
   - `k=11`、`k=12` 也不是又换了一套 proof skeleton；
   - 真正上升的是 shared norm 的位数，以及因此带来的 arithmetic backend 压力。
2. **orbit reduction 的收益会随着层数上升越来越大。**
   - `k=9` 时从 `32` 对降到 `1` 个 shared norm；
   - `k=10` 时从 `64` 次 factorization 降到 `1` 次；
   - `k=12` 时如果没有这条对称性，计算会从“艰难但可能”直接变成“几乎不可管理”。

**把每层的 proof/computation ledger 分别写出来**

对于第一次想自己跟 theorem 的读者，下面这四个 layer note 比泛泛地说“后面类似”更有用。

**k=9**

- 归纳输入：base case \(h_8^+=1\)。
- full-order 阶：\(128\)。
- 本地讲义里已经真正跑通了一个 merged Stage-A recomputation。
- 该层在本地 notebook 中的核心现象是：shared norm 的 odd factors 经 cutoff 与 congruence filter 后已经清空，因此 Phase B 变成 vacuous。

所以 `k=9` 是最适合第一次完整跟 theorem 的层，因为它已经足够像 paper，但还没有把你拖进特别重的大整数分解。

**k=10**

- 归纳输入：上一层 \(h_9^+=1\)。
- full-order 阶：\(256\)。
- sieve 模数：\(2^{k-1}=512\)，因此 surviving primes 必须满足
  \(\ell\equiv\pm1\pmod{512}\)。
- 论文的全局描述是：在 orbit reduction 之后，这一层只需 factor 一个最多 `143` 位的整数；过滤之后，最终 residue checks 的规模最多是 `10` 次 modular exponentiations。

教学上，`k=10` 是最关键的一层，因为它第一次把 theorem notebook 从“概念可跟”推进到“算术后端真的开始吃力”。

**k=11**

- 归纳输入：上一层 \(h_{10}^+=1\)。
- full-order 阶：\(512\)。
- sieve 模数：\(2^{k-1}=1024\)。
- 论文 `Table 2` 报告的 shared norm 最大位数是 `325` 位，量级从小时推进到天级。

这时最重要的教学判断是：**证明结构没有变，变化的是 factorization regime**。也就是说，读者不需要再学一套新 theorem，只需要接受“同一条 verified pipeline 在更大的整数上继续运行”。

**k=12**

- 归纳输入：上一层 \(h_{11}^+=1\)。
- full-order 阶：\(1024\)。
- sieve 模数：\(2^{k-1}=2048\)。
- `Table 2` 给出的 shared norm 最大位数是 `726` 位，作者把这一层描述成周级计算，并提示可能需要 NFS 或 Euler-system 风格的替代路线。

这层最值得学的并不是“如何手算”，而是理解 paper 的 reach：

- Theorem 3.3 到这里并没有在逻辑上断掉；
- 断点只会出现在 arithmetic backend 是否足够强；
- 这正是 `3.5` 要讨论更强优化路线的原因。

**什么才算真正完成了一层**

很多读者会误以为“只要 factor 出一个 norm，就差不多了”。这其实不够。对任意固定层 \(k\)，真正的 completion condition 是：

1. 你已经有上一层的 \(h_{k-1}^+=1\)；
2. 你已经把 low-order characters 排空；
3. 你已经得到并 factor 了该层的 shared norm；
4. 你已经把不满足 cutoff / congruence 的 odd primes 删掉；
5. 对所有剩余 \(\ell\)，你已经把 prime-ideal residue tests 做完；
6. 没有任何 prime 留下。

只做到第 `3` 步，顶多说明“已经把无限风险变成有限列表”；只有做到第 `6` 步，才真的闭合了 Theorem 3.3 的一个 layer proof。

**这一小节最该记住的结论**

1. `k=9,10,11,12` 的区别主要是参数和算术规模，不是 theorem skeleton 变了。
2. 论文真正的 computational miracle 不是“算得很快”，而是“每层只需追一个 shared norm，再追一个有限 prime list”。
3. 如果你能把这张分层 checklist internalize，下次再读原论文 `3.4` 与 `3.5`，你会更清楚哪里是证明，哪里是计算，哪里是优化。

**把 later-layer 的 final residue elimination 写成一页工作单**

到这里为止，读者最容易还差的一步是：虽然知道 `k=10,11,12` 也要做 Phase B，但还不知道对一个 surviving \(\ell\) 到底要准备哪些数据、跑哪些局部测试。把 `8.10` 的 residue-side notebook 与这里的 paper-scale ledger 拼起来，later-layer 的最后一跳其实可以压成下面这张工作单。

1. 固定层数 \(k\) 与一个 surviving odd prime \(\ell\)。
   这里 “surviving” 已经包含三层先验：
   - 上一层结论 \(h_{k-1}^+=1\) 已知；
   - low-order characters 已由 `3.2` 排空；
   - \(\ell\) 已通过 shared norm、cutoff 与 congruence filter，因此必满足 \(\ell>10^9\) 且 \(\ell\equiv \pm1\pmod{2^{k-1}}\)。
2. 把 \(\ell\) 在 \(K_k^+\) 中分解成所有 \(\mathfrak l\mid \ell\)。
   这一步不是重新做 global proof，而是把一个剩余整数候选转成有限个 local test sites。
3. 对每个 \(\mathfrak l\)，计算 residue degree \(f\) 与测试指数
   \(E_{\mathfrak l}=(\operatorname{N}(\mathfrak l)-1)/2^{k-1}\)。
   由于 \(\ell\equiv \pm1\pmod{2^{k-1}}\)，`8.10` 已经解释过这里强迫 \(f\le2\)，所以
   \(\operatorname{N}(\mathfrak l)\) 只会是 \(\ell\) 或 \(\ell^2\)。
4. 把固定的 cyclotomic unit \(\xi_5\) 约化到每个 residue field，并检查
   \(\xi_5^{E_{\mathfrak l}}\bmod \mathfrak l\) 是否等于 \(1\)。
5. 如果对所有 \(\mathfrak l\mid \ell\) 都得到“不等于 \(1\)”的结果，就按 Lemma 3.1 / Theorem 3.1 删掉这个 \(\ell\)。
   只有当某个 \(\mathfrak l\) 真正给出 \(1\) 时，这个 \(\ell\) 才继续悬而未决。

这张工作单最关键的教学意义有三点。

- 它说明 later-layer 的 Phase B 不是“再来一次大整数分解”，而是把 Stage A 留下的极小候选表逐个做局部验证。
- 它说明 `k=10,11,12` 的 arithmetic pain 主要集中在 shared norm factorization；一旦进入 residue elimination，逻辑结构反而比 Stage A 更短。
- 它也解释了为什么论文会把 later-layer residue cost 记成很小的 modular exponentiation 数量：真正昂贵的是筛前端，最后的 theorem closure 只是有限个 local certificates。

把这一步压成一句话，就是：

```text
global shared norm gave you a finite suspect list
    ->
Theorem 3.1 turns each suspect prime into finitely many local checks
    ->
every local check fails
    ->
the suspect prime is removed
    ->
no suspects remain
    ->
h_k^+ = 1
```

**把这张工作单再压成一页逐公式 notebook 模板**

如果你真的要把某个 later-layer 候选 \(\ell\) 写进自己的 proof notebook，最省力的方式不是散着记，而是固定成下面五栏。

1. `global assumptions`
   记下这一层已经完成的全局前提：
   \(h_{k-1}^+=1\)、\(2\nmid h_k^+\)、low-order characters 已排空，以及
   \(\ell\) 已通过 shared norm 与 cutoff / congruence filter。
2. `branch choice`
   先只看
   \(\ell\equiv 1\) 还是 \(-1 \pmod{2^{k-1}}\)。
   这一步立即决定你走的是 \(f=1\) 还是 \(f=2\) 分支。
3. `local data`
   对每个 \(\mathfrak l\mid\ell\)，只记录三件事：
   \(N(\mathfrak l)\)、测试指数 \(E_{\mathfrak l}\)、以及
   \(r_{\mathfrak l}=\xi_5^{E_{\mathfrak l}}\bmod\mathfrak l\)。
4. `prime verdict`
   若所有 \(r_{\mathfrak l}\neq 1\)，则由 Lemma 3.1 删掉整个 \(\ell\)。
   注意删掉的是 prime \(\ell\)，不是单个 prime ideal \(\mathfrak l\)。
5. `layer closure`
   只有当所有 surviving \(\ell\) 都被这样删空，这一层才真正闭合成
   \(h_k^+=1\)。

照这个结构抄一页，later-layer 的 residue elimination 就不会再显得“只有一句话但不知道怎么算”：

```text
Layer k = ...
Candidate prime ell = ...
Global assumptions:
    h_{k-1}^+ = 1
    2 ∤ h_k^+
    low-order eigenspaces already eliminated
    ell > 10^9
    ell ≡ ±1 mod 2^(k-1)

Branch:
    if ell ≡ 1 mod 2^(k-1):
        f = 1
        N(l) = ell
        E_l = (ell - 1) / 2^(k-1)
    if ell ≡ -1 mod 2^(k-1):
        f = 2
        N(l) = ell^2
        E_l = (ell^2 - 1) / 2^(k-1)

For each prime ideal l | ell:
    compute r_l = xi_5^(E_l) mod l
    record whether r_l == 1

Verdict for ell:
    all r_l != 1
        -> ell ∤ h_k^+
    some r_l == 1
        -> ell not yet eliminated by this local certificate

Layer closure:
    every surviving ell eliminated
        -> no odd prime divisor remains
        -> h_k^+ = 1
```

这张模板和 `9.8` 的 `k=5,\ell=97` toy walkthrough 是一一对应的。区别只在于：

1. `9.8` 真的把一个小层数 prime 全跑出来了；
2. 这里把同样的动作提升成适用于 `k=10,11,12` 的 paper-scale记录格式；
3. 因而读者以后如果要复核论文某个 surviving \(\ell\)，至少知道 notebook 应该长成什么样，而不是只剩一句“做 residue test”。

**为什么这张模板足够**

真正的原因是：Phase B 已经不再承担“找候选”的职责。进入这一页 notebook 之前，所有 global work 都已经完成，剩下的只是把一个具体 prime \(\ell\) 做成 finitely many local yes/no tests。所以 later-layer final residue elimination 的证明感，来自于：

1. 每个局部域上的测试对象都同一个，始终是 \(\xi_5\)；
2. 每个局部域上的测试指数都来自同一个公式，只因 \(f=1\) 或 \(f=2\) 分支而变化；
3. 每个 prime 的结论标准也完全固定：不是看 residue 长得“像不像随机数”，而是只看它是否等于 \(1\)；
4. 一旦全部 local checks 都失败，这个 \(\ell\) 就获得了一个完整的排除证书。

#### 一个 later-layer final residue elimination theorem notebook：把 “every surviving prime is locally impossible” 真正写成 layer proof

上面的工作单已经告诉读者：

> later-layer 的最后一跳  
> 不是重新做 global search，  
> 而是把每个 surviving \(\ell\)  
> 逐个做 local residue elimination。

但很多读者到这里 still 会差最后半步：

> 我知道怎么算一个 surviving \(\ell\)，  
> 但我还没有把  
> “每个 surviving \(\ell\) 都被删掉”  
> 真正听成  
> “这一层 theorem 闭合了”。

也就是说，工作单已经解决了
`how to compute`，
却还没有完全解决
`why this computation closes the proof`。

这一页就是专门补这最后一层 theorem notebook。

**先把 later-layer 的目标句压成真正的量词结构**

对固定层数 \(k\in\{10,11,12\}\)，later-layer final residue elimination 真正想交出的不是一句模糊的

> “把 residue tests 都跑完了”。

而是下面这条更硬的量词链：

```text
for every odd prime ell
    if ell | h_k^+
    then ell must lie in the finite surviving set S_k

for every ell in S_k
    for every prime ideal l | ell
    compute the prescribed xi_5-residue certificate

if every such local certificate fails
    then ell is impossible

if every ell in S_k is impossible
    then no odd prime divides h_k^+

since 2 ∤ h_k^+
    conclude h_k^+ = 1
```

这条链真正难的地方，
不是局部公式本身，
而是读者必须始终分清：

1. 哪一句是在说 theorem；
2. 哪一句是在说 finite computation；
3. 哪一句是在做 layer closure。

**把 theorem / computation / closure 三件事强行拆开**

| 句子层级 | 真正内容 | 由什么来证明 | 绝对不要误听成什么 |
| --- | --- | --- | --- |
| theorem statement | odd divisor \(\ell\) 若存在，必须先进入有限候选表 \(S_k\) | 3.1–3.3 的筛法链 | “shared norm factorization 本身就等于结论” |
| finite computation | 对每个 surviving \(\ell\)，把所有 \(\mathfrak l\mid \ell\) 的 residue certificates 真正算出来 | factorization + local residue arithmetic | “作者随手验了几个 prime ideal 就差不多了” |
| layer closure | 每个 surviving \(\ell\) 都被删空，因此无 odd prime divisor 留下 | 对有限候选表做 exhaustively complete 的 yes/no audit | “某个代表性 \(\ell\) 被删掉，所以这一层大概也完成了” |

这张表最该防住的，
就是 later-layer 最常见的三种过快结论：

1. `factor 出 shared norm` 就当成 theorem 完成；
2. `算完一个 prime ideal` 就当成一个 prime 完成；
3. `删掉几个大 prime` 就当成这一层完成。

真正的 theorem notebook 必须把这三层强制分开。

**先把单个 surviving prime 的证明账本写完整**

固定一个 later-layer surviving prime \(\ell\)。
一页最稳的 theorem notebook，
其实只需要下面六行。

| proof row | incoming fact | action | outgoing fact | if skipped, you will overclaim |
| --- | --- | --- | --- | --- |
| R1 suspect entry | 假设 \(\ell\) 是 odd prime 且 \(\ell\mid h_k^+\) | 调用前半段筛法链，把 \(\ell\) 压进 finite suspect set \(S_k\) | \(\ell\) 已是 later-layer finite suspect | “只要 factor 了一个 norm，就已经抓住了全部 odd divisors” |
| R2 branch choice | \(\ell\in S_k\) 且 \(\ell\equiv \pm 1 \pmod{2^{k-1}}\) | 决定 \(f=1\) 或 \(f=2\) 分支 | residue degree branch fixed | “反正指数公式都差不多，分支无关紧要” |
| R3 local exponent setup | 分支已定 | 对每个 \(\mathfrak l\mid \ell\) 写出 \(N(\mathfrak l)\) 与 \(E_{\mathfrak l}=(N(\mathfrak l)-1)/2^{k-1}\) | local certificate format fixed | “只要有一个模指数看起来不像 \(1\) 就够了” |
| R4 local elimination | 每个 \(\mathfrak l\mid \ell\) 的格式已固定 | 真正计算 \(r_{\mathfrak l}=\xi_5^{E_{\mathfrak l}}\bmod \mathfrak l\) | 每个 local site 都拿到 yes/no verdict | “抽样验几个 \(\mathfrak l\) 也足以判断整个 \(\ell\)” |
| R5 prime verdict | 对所有 \(\mathfrak l\mid \ell\) 都得到 \(r_{\mathfrak l}\neq 1\) | 调用 Lemma 3.1 / Theorem 3.1 删掉整个 \(\ell\) | \(\ell \nmid h_k^+\) | “删掉了某个 \(\mathfrak l\)，因此 prime \(\ell\) 也就没了” |
| R6 layer closure | 对 every surviving \(\ell\) 都完成 R1-R5 | 对 finite suspect set 做 exhaustively complete audit | no odd prime divisor remains | “删掉了最难的几个大 prime，剩下的应该也没问题” |

这六行里最容易被低估的是 `R6`。
因为很多读者觉得前五行都只是 “单个 prime 的事”，
最后一句像纯收尾。

实际上 `R6` 才是 later-layer 从

> “我会算一个 residue test”

变成

> “这一层 theorem 真闭合了”

的那一步。

**把 ±1 分支真正写成可对照的 case split**

later-layer 的 residue elimination 看起来常常像只是在“套同一个公式”，
但对 theorem notebook 而言，
最关键的 still 是先把两条分支分清：

| branch | congruence condition | residue degree \(f\) | norm of local site | exponent formula | 真正作用 |
| --- | --- | --- | --- | --- | --- |
| Branch + | \(\ell \equiv 1 \pmod{2^{k-1}}\) | \(f=1\) | \(N(\mathfrak l)=\ell\) | \(E_{\mathfrak l}=(\ell-1)/2^{k-1}\) | 把 prime test 直接压在 \(\mathbb{F}_\ell\)-size residue field 上 |
| Branch - | \(\ell \equiv -1 \pmod{2^{k-1}}\) | \(f=2\) | \(N(\mathfrak l)=\ell^2\) | \(E_{\mathfrak l}=(\ell^2-1)/2^{k-1}\) | 把 test 提到 degree-2 residue field 上 |

这张表的教学重点不是记住公式，
而是明确：

> later-layer 的 local proof  
> 并不是 “固定一个万能指数”；
> 它首先是一个 branch-sensitive certificate。

所以成熟的 theorem notebook 不该只写：

> **compute xi_5^E mod l**

而应该先写：

> **which branch are we in, and why does this branch decide E?**

**把单个 prime 的 notebook 真正压成最短 theorem skeleton**

如果你以后想复核某个 `k=10`、`k=11` 或 `k=12` 的 surviving \(\ell\)，
最短可以只写下面这张骨架：

```text
Layer k = ...
Prime ell = ...

Global entry:
    ell is a surviving odd suspect
    h_{k-1}^+ = 1
    2 ∤ h_k^+

Branch:
    ell ≡ +1 or -1 mod 2^(k-1)
    therefore f = ...
    therefore N(l) = ...
    therefore E_l = ...

Local certificates:
    for every l | ell
    compute r_l = xi_5^(E_l) mod l

Prime verdict:
    all r_l != 1
        -> ell ∤ h_k^+

Layer verdict:
    every surviving ell eliminated
        -> no odd prime divisor remains
        -> h_k^+ = 1
```

这张 skeleton 比上一页模板更“theorem notebook”一点，
因为它强迫你把

1. `surviving suspect`；
2. `branch-fixed local certificate`；
3. `prime verdict`；
4. `layer verdict`

四层结论分开写。

**为什么 later-layer 的最后一跳本质上是一个 finite universal statement**

很多读者对 later-layer final residue elimination 仍然会残留一种心理错觉：

> 既然 prime 很大、计算很重，  
> 这一段好像更像工程后端，  
> theorem 成分反而变少了。

其实正相反。
later-layer 的最后一跳之所以值得教材化，
恰恰因为它把 theorem 压成了一个非常干净的 finite universal statement：

```text
there is a finite suspect set S_k
and every element of S_k receives a complete local no-certificate
therefore S_k contains no actual odd divisor of h_k^+
```

真正增加的只是 backend 的算术重量；
并不是 logical shape 变脏了。

这也是为什么 `k=10,11,12` 的教学重点不该是“手算一个超大 prime”，
而该是：

> 你能不能清楚地区分  
> theorem closure  
> 和  
> arithmetic backend。

**再给一张最短的 false-closure 对照表**

| 你若把 later-layer 读快了 | 实际漏掉了什么 |
| --- | --- |
| “shared norm 已分解，所以这一层差不多了” | 漏掉了 R4-R6 的局部证书与全称闭环 |
| “一个 \(\mathfrak l\) 给出非 \(1\) 就说明 prime 被删掉” | 漏掉了 for every \(\mathfrak l\mid \ell\) |
| “一个 surviving \(\ell\) 被删掉，剩下也类似” | 漏掉了 finite suspect set 的 exhaustiveness |
| “later-layer 只是算得更慢，没有新 proof” | 漏掉了 theorem/computation/closure 三层分工 |

这张表最该带走的判断是：

> later-layer 最后一跳  
> 不是“把几个大数算完”；  
> 而是“把一个 finite suspect universe 做完备审计”。

**最后把这一页压成一句最该带走的话**

later-layer final residue elimination 的证明感，
不在于你会不会手算一个 `300` 位或 `700` 位对象，
而在于你能不能把下面这句话真正说完整：

> 每个 surviving prime 都进入了正确 branch；  
> 每个 branch 都给出了完整 local certificate；  
> 每个 local certificate 都把对应 prime 删掉；  
> 所以 finite suspect set 被 exhaustively 清空；  
> 因而这一层真的闭合成 \(h_k^+=1\)。

#### 一张 branch-by-branch local-certificate verification page：把 surviving \(\ell\) 真正压成逐公式 audit

上一页已经把 later-layer 的最后一跳压成了 theorem notebook：

1. 先有 finite suspect set；
2. 再给每个 surviving \(\ell\) 做 local no-certificate；
3. 最后把整个 suspect set 清空。

但如果读者真的准备自己拿一页纸去复核 later-layer，
通常还会卡在更硬的一层：

> 我知道 branch 有 `+` / `-` 两种；  
> 我也知道要算 \(E_{\mathfrak l}\)；  
> 但我还没有把  
> **which quantity depends on the branch**
> 真正压成一张逐公式 audit。

这正是这一页要补的东西。

它不再讲 theorem closure，
而只做一件事：

> 把单个 surviving \(\ell\) 的 local certificate  
> 压成可逐格核对的公式页。

**先把 later-layer local certificate 真正依赖的量列出来**

固定 \(k\in\{10,11,12\}\) 与一个 surviving odd prime \(\ell\)。
真正要核的不是一大段 prose，
而是下面这几行量之间的依赖关系：

\[
n_k = [K_k^+:\mathbb{Q}] = 2^{k-2}.
\]

\[
\ell \equiv \pm 1 \pmod{2^{k-1}}
\quad\Longrightarrow\quad
f \in \{1,2\}.
\]

\[
g = \frac{n_k}{f},
\]

其中 \(g\) 是 later-layer 要逐个核的 prime ideals 数量，
也就是 \(\ell\) 上方 local sites 的个数。

接着对每个 \(\mathfrak l\mid \ell\)，
要固定：

\[
N(\mathfrak l)=\ell^f,
\qquad
E_{\mathfrak l}=\frac{N(\mathfrak l)-1}{2^{k-1}},
\qquad
r_{\mathfrak l}=\xi_5^{E_{\mathfrak l}} \bmod \mathfrak l.
\]

最后判断的不是 “residue 看起来像不像随机”，
而是这句最硬的 yes/no：

\[
r_{\mathfrak l} \stackrel{?}{=} 1.
\]

这一串公式最该防的是下面这种模糊读法：

> “反正就是选一个指数，  
> 然后把 \(\xi_5\) 幂一幂。”

真正成熟的 audit 必须继续问：

1. branch 决定的是哪几个量；
2. local site 数量为什么也跟 branch 有关；
3. prime verdict 到底是从 “一个 local site” 还是 “所有 local sites” 领出来的。

**先给一张 branch-sensitive formula ledger**

| quantity | Branch + | Branch - | 谁决定它 | 若读快了最容易偷掉什么 |
| --- | --- | --- | --- | --- |
| congruence | \(\ell \equiv 1 \pmod{2^{k-1}}\) | \(\ell \equiv -1 \pmod{2^{k-1}}\) | surviving-prime branch choice | “\(\pm1\) 只是同余装饰，不影响后面公式” |
| residue degree \(f\) | \(1\) | \(2\) | branch | “只换指数，不换 local field size” |
| local site count \(g\) | \(2^{k-2}\) | \(2^{k-3}\) | \(g=n_k/f\) | “两条 branch 只是算同一批 \(\mathfrak l\)” |
| local norm \(N(\mathfrak l)\) | \(\ell\) | \(\ell^2\) | \(f\) | “\(N(\mathfrak l)\) 永远就是 \(\ell\)” |
| exponent \(E_{\mathfrak l}\) | \((\ell-1)/2^{k-1}\) | \((\ell^2-1)/2^{k-1}\) | \(N(\mathfrak l)\) | “\(\ell^2\) 只是 cosmetic square” |
| certificate host | degree-1 residue field | degree-2 residue field | branch-fixed local geometry | “local certificate 的宿主大小没变” |

这张表真正想讲的是：

> **Branch -**
> 不是把 `Branch +` 的公式里  
> 多写一个平方而已；  
> 它真的把  
> residue degree、local field size、local site count、指数公式  
> 一起改了。

**把 k=10,11,12 三层的 branch data 真正摊开**

为了让 later-layer audit 不再停在符号层，
下面把三层真正要抄的 branch data 直接列出来。

| layer | \(n_k=[K_k^+:\mathbb{Q}]\) | Branch + local sites \(g_+\) | Branch + exponent | Branch - local sites \(g_-\) | Branch - exponent |
| --- | --- | --- | --- | --- | --- |
| k=10 | \(2^8=256\) | \(256\) | \((\ell-1)/512\) | \(128\) | \((\ell^2-1)/512\) |
| k=11 | \(2^9=512\) | \(512\) | \((\ell-1)/1024\) | \(256\) | \((\ell^2-1)/1024\) |
| k=12 | \(2^{10}=1024\) | \(1024\) | \((\ell-1)/2048\) | \(512\) | \((\ell^2-1)/2048\) |

这张表的教学价值很直接：

1. 它把 later-layer 的 branch 差别从“抽象理论”压成了可抄写的数字；
2. 它让读者看到 `Branch -` 不只改指数，也把 local site 数量砍半；
3. 它解释了为什么 later-layer 的局部验证虽然逻辑短，但实际 notebook 仍然可能很长:
   branch 决定了你到底要写多少个 \(\mathfrak l\)-rows。

**把单个 surviving prime 的公式级 audit 真正写成六格页**

如果你真的要对某个 later-layer surviving \(\ell\) 做最短核对，
最省力的不是散着记公式，
而是固定写下面六格：

```text
Layer:
Branch:
f:
number of local sites g:
for each l | ell:
    N(l) =
    E_l =
    r_l = xi_5^(E_l) mod l
prime verdict:
```

看起来很短，
但这六格已经强迫你回答 later-layer local certificate 里最容易漏的四件事：

1. 你到底在哪一层；
2. 你到底走的是哪个 branch；
3. 你到底要验多少个 local sites；
4. 你到底是凭一个 \(r_{\mathfrak l}\) 还是全部 \(r_{\mathfrak l}\) 领出 prime verdict。

**把最危险的四个 false equalities 单独写出来**

later-layer 的公式级核对，
最容易在下面四个位置偷并：

| false equality | 为什么错 |
| --- | --- |
| \(\ell \equiv \pm 1 \pmod{2^{k-1}}\) = branch already harmless | 这只说明 branch 已定，还没有任何 local certificate |
| \(f\) fixed = prime verdict almost done | 这只说明宿主大小与指数公式已定，还没算 residues |
| 某个 \(r_{\mathfrak l}\neq 1\) = \(\ell\) 被删掉 | prime verdict 需要 for every \(\mathfrak l\mid \ell\) |
| 一个 prime \(\ell\) 被删掉 = layer closed | layer verdict 需要对整个 finite suspect set 做 exhaustive audit |

这张表和上一页 theorem notebook 的关系是：

1. 上一页管 theorem closure；
2. 这一页管单个 prime 的公式级 local certificate；
3. 两页拼起来，later-layer 才真正从“会复述流程”变成“能自己抄 proof notebook”。

**最后把这一页压成一句最该带走的话**

later-layer final residue elimination 走到公式级时，
最重要的不是多记一个符号，
而是始终分清：

> branch 决定 \(f\)、\(g\)、\(N(\mathfrak l)\)、\(E_{\mathfrak l}\)；  
> 所有 \(\mathfrak l\)-rows 合起来才决定 prime verdict；  
> 所有 prime verdict 合起来才决定 layer closure。

如果这三层被混成一句“反正最后都算 \(\xi_5^{E_{\mathfrak l}}\)”，
later-layer 的 proof 感就会再次塌回成一段算术黑箱。

#### 一张 prime-ideal-row audit page：不要让一条 \(\mathfrak l\)-row 假装已经删掉整个 \(\ell\)

上一页已经把 later-layer 的公式级 notebook 压到了：

1. branch 决定 \(f\)；
2. \(f\) 决定 \(g\)、\(N(\mathfrak l)\)、\(E_{\mathfrak l}\)；
3. 所有 local rows 合起来才决定 prime verdict。

但真到读者自己抄 notebook 时，
还会残留最后一种很容易犯的错：

> 我已经算出某一个 \(\mathfrak l\)-row 的  
> \(r_{\mathfrak l}\neq 1\)，  
> 于是脑子自动把它听成  
> “这个 prime 差不多已经被删掉了”。

这正是 later-layer 里最危险的 row-level overclaim。

因为 theorem 真正要求的是：

\[
\forall\,\mathfrak l \mid \ell,\quad r_{\mathfrak l}\neq 1,
\]

而不是：

\[
\exists\,\mathfrak l \mid \ell,\quad r_{\mathfrak l}\neq 1.
\]

所以这一页只做一件更窄、但更实用的事：

> 把单个 \(\mathfrak l\)-row  
> 能证明什么、  
> 还绝对不能证明什么、  
> 以及它下一步要交给谁  
> 写成真正的 row audit。

**先把 row-level 的最小量词差别写死**

later-layer 的 prime-elimination 闭环里，
最该盯住的不是复杂公式，
而是下面这条最小量词差别：

```text
one row cleared
    !=
prime ell cleared

every row above ell cleared
    =>
prime ell cleared
```

如果这一步不在脑中写死，
later-layer 的 local arithmetic 就会再次被误听成
“只要出现几个非 \(1\) residue，证明就差不多了”。

**先给一张真正的 single-row audit ledger**

固定一层 \(k\)、一个 surviving prime \(\ell\)、以及一条具体的
\(\mathfrak l\mid \ell\)。
这一条 row 最短该写成下面七格：

| row field | 这一格到底写什么 | 这一格真能证明什么 | 这一格绝对还不能证明什么 |
| --- | --- | --- | --- |
| branch | + 或 - | 当前 row 的宿主和指数公式已固定 | 其他 rows 也都在同一 verdict 下通过 |
| row id | 第几个 \(\mathfrak l\) | 你现在在 audit 哪个 local site | 这个 row 具有代表性 |
| N(\mathfrak l) | \(\ell\) 或 \(\ell^2\) | 当前 row 的 residue-field size | prime verdict 已近完成 |
| E_{\mathfrak l} | \((N(\mathfrak l)-1)/2^{k-1}\) | 当前 row 的测试指数已确定 | 这个指数能替其他 rows 通用 |
| r_{\mathfrak l} | \(\xi_5^{E_{\mathfrak l}}\bmod \mathfrak l\) | 当前 row 的实际 certificate value | 整个 \(\ell\) 的证书值都非 \(1\) |
| row verdict | r_{\mathfrak l}\neq 1 或 =1 | 当前 local site 通过或未通过 | prime \(\ell\) 已删掉或保留 |
| passed next | “still need every other row” 或 “prime remains unresolved” | 下一步该把什么信息交回 prime-level audit | layer closure 已经发生 |

这七格真正想训练的是：

> row verdict  
> 只能把信息交回 prime-level ledger；  
> 它自己从来不是 prime verdict。

**把 Branch + / Branch - 的 row 长相并排放出来**

如果把上一页的 branch table 再往下压一层，
单条 row 的最短长相其实是下面这样。

| row shape | Branch + | Branch - |
| --- | --- | --- |
| local host | residue field of size \(\ell\) | residue field of size \(\ell^2\) |
| row count inside one prime | \(g_+=2^{k-2}\) rows | \(g_-=2^{k-3}\) rows |
| row exponent | \(E_{\mathfrak l}=(\ell-1)/2^{k-1}\) | \(E_{\mathfrak l}=(\ell^2-1)/2^{k-1}\) |
| row pass condition | \(r_{\mathfrak l}\neq 1\) | \(r_{\mathfrak l}\neq 1\) |
| what one passed row still means | one local site cleared | one local site cleared |
| what one passed row still does NOT mean | \(\ell\) cleared | \(\ell\) cleared |

这张对照表最重要的地方不是 `\ell` 还是 `\ell^2`，
而是最后两行完全一样：

> 无论 branch 是 `+` 还是 `-`，  
> 一条通过的 row  
> 都只等于  
> `one local site cleared`。

这也是为什么 later-layer 的 row audit 必须先写量词，
再写公式。

**把 prime-level accumulation 真正写成一个 row counter**

如果你要把 later-layer 真抄成 notebook，
最实用的往往不是一大段 prose，
而是一张最硬的 row counter：

```text
Prime ell:
branch = ...
total rows required = g

row 1: r_l1 ?= 1   -> pass/fail
row 2: r_l2 ?= 1   -> pass/fail
...
row g: r_lg ?= 1   -> pass/fail

prime verdict:
    all rows pass    -> ell eliminated
    some row fails   -> ell unresolved by this certificate
```

这张 counter 最有价值的地方，
是它把 later-layer 最容易被 prose 糊过去的一层，
直接压成了：

1. 这一层一共欠多少行；
2. 现在已经清掉了多少行；
3. 还差哪一行没交。

**再给一张最短的 false-row-closure 对照表**

| 你若把 row audit 读快了 | 实际偷掉了哪层量词 |
| --- | --- |
| “这个 \(\mathfrak l\) 已经给出非 \(1\)，所以 \(\ell\) 基本没了” | for all \(\mathfrak l\mid \ell\) |
| “Branch - 的 rows 只有一半，所以大概更容易结束” | row 数量变少不等于 prime verdict 自动得到 |
| “这一行的 \(E_{\mathfrak l}\) 已经对了，所以剩下只是在机械计算” | 你还没有证明其余 rows 也都通过 |
| “这个 prime 被删掉后，这一层应该就差不多了” | finite suspect set 的 exhaustive audit |

这张表真正最该带走的是第一行：

> one row pass  
> 从来不等于  
> one prime eliminated。

**给读者一张最短可手抄的 prime-ideal row card**

如果以后你真回到某个 surviving \(\ell\) 的 later-layer notebook，
最短可以只写下面八格：

```text
layer:
prime ell:
branch:
row id:
N(l):
E_l:
r_l == 1 ?
what still remains before prime verdict:
```

这张 card 的价值不在于它公式多，
而在于最后一格强迫你每算完一行都追问：

> 我现在只是清掉了一个 row，  
> 还是已经有资格领出整个 prime verdict？

**最后把这一页压成一句最该带走的话**

later-layer final residue elimination 真正细到 row 级以后，
proof 最危险的错读就不再是公式看不懂，
而是量词听错。
更稳的读法必须始终把下面三层分开：

> 一条 \(\mathfrak l\)-row 只清一个 local site；  
> 一个 prime verdict 需要清空该 \(\ell\) 上方所有 rows；  
> 一个 layer verdict 需要清空 finite suspect set 中所有 primes。

如果这三层在脑中被压成一层，
later-layer 的 row-level audit 就会重新塌回算术黑箱。

#### 一个 concrete local-field row walkthrough：把 `k=5, \ell=97` 的八条 row 真抄成一页完整 counter

上面的 `prime-ideal-row audit page`
已经把 row-level 量词关系讲清楚了，
但它 still 偏抽象。

如果读者真的想把 later-layer 的 row 审计肌肉练出来，
最稳的下一步不是再看一张更抽象的模板，
而是拿一个 **已经有真实 residue 值** 的局部域实例，
把所有 rows 一条一条抄完。

本讲义里最适合做这件事的，
就是前面 `9.8` 已经真正跑通的例子：

\[
k=5,\qquad \ell=97.
\]

它当然不是 later-layer。
但它在 row grammar 上和 later-layer 完全同型：

1. 先定 branch；
2. 再定 \(f\)、\(g\)、\(N(\mathfrak l)\)、\(E_{\mathfrak l}\)；
3. 再把所有 rows 做完；
4. 最后才领出 prime verdict。

所以这一页的定位非常明确：

> 先用 `k=5,\ell=97`  
> 把 row-level audit 真做成一页完整账本；  
> 然后再把同一个读法回带到 `k=10,11,12`。

**先把这个 concrete instance 的 branch data 一次写死**

对 `k=5,\ell=97`，
前面 `9.8` 已经给出：

\[
97 \equiv 1 \pmod{16},
\]

因此这次走的是 `Branch +`。

这一页最先要写的不是 residue 值，
而是四个 branch-fixed facts：

\[
n_5=[K_5^+:\mathbb{Q}] = 2^{5-2}=8,
\qquad
f=1,
\qquad
g=\frac{n_5}{f}=8,
\qquad
N(\mathfrak l)=97.
\]

同时测试指数也已经固定成

\[
E_{\mathfrak l}=\frac{97-1}{16}=6.
\]

也就是说，
在真正开始看 rows 之前，
这页最重要的 branch summary 已经是：

```text
layer = k=5
prime ell = 97
branch = +
f = 1
total rows required = g = 8
every row uses N(l) = 97 and E_l = 6
```

如果这六行不先写出来，
后面的 row table 只会看起来像“八个随机 residue”。

**现在把八条 rows 真抄成一个完整 counter**

下面这张表直接重用 `9.8` 已经验证过的本地输出，
不新增任何猜测值。

| row id | local site \(\mathfrak l_i\) | \(N(\mathfrak l_i)\) | \(E_{\mathfrak l_i}\) | \(r_{\mathfrak l_i}=\xi_5^{E_{\mathfrak l_i}}\bmod \mathfrak l_i\) | row verdict | what still remains |
| --- | --- | --- | --- | --- | --- | --- |
| R1 | \((97,\zeta_{320}+12)\) | \(97\) | \(6\) | \(33\) | pass: \(33\neq 1\) | still need rows R2-R8 |
| R2 | \((97,\zeta_{320}+17)\) | \(97\) | \(6\) | \(50\) | pass: \(50\neq 1\) | still need rows R3-R8 |
| R3 | \((97,\zeta_{320}+32)\) | \(97\) | \(6\) | \(85\) | pass: \(85\neq 1\) | still need rows R4-R8 |
| R4 | \((97,\zeta_{320}+43)\) | \(97\) | \(6\) | \(8\) | pass: \(8\neq 1\) | still need rows R5-R8 |
| R5 | \((97,\zeta_{320}-43)\) | \(97\) | \(6\) | \(8\) | pass: \(8\neq 1\) | still need rows R6-R8 |
| R6 | \((97,\zeta_{320}-32)\) | \(97\) | \(6\) | \(85\) | pass: \(85\neq 1\) | still need rows R7-R8 |
| R7 | \((97,\zeta_{320}-17)\) | \(97\) | \(6\) | \(50\) | pass: \(50\neq 1\) | still need row R8 |
| R8 | \((97,\zeta_{320}-12)\) | \(97\) | \(6\) | \(33\) | pass: \(33\neq 1\) | no row remains; now prime verdict may be claimed |

这张表最该教给读者的，
不是这些具体 residue 数字；
而是：

> 前七条 row 全都通过时，  
> 你仍然只能说  
> `almost done but not yet prime verdict`。

只有 `R8` 交完以后，
你才第一次有资格把量词从

\[
\text{some rows have passed}
\]

升级成

\[
\forall\,\mathfrak l\mid 97,\quad r_{\mathfrak l}\neq 1.
\]

**把这页最关键的 overclaim 点单独钉出来**

如果这页只允许你带走一句页边话，
应该是：

> **R7 is not weaker than R1; it is just not last enough.**

也就是说，
`R8` 的特殊性不在于它算出了一个更强的 residue，
而只在于：

> 它是最后一条尚未交出的 row。

这是 later-layer row audit 里非常重要的一种成熟感：

1. row 强度不靠“看起来更大更复杂”；
2. row 强度只靠它在量词链里的位置；
3. 最后一条 row 之所以关键，不是数学上更强，而是逻辑上补齐了 `for all`。

**把 concrete row counter 真正压成 prime verdict**

对这个 `k=5,\ell=97` 实例，
prime-level ledger 其实可以压成下面四行：

```text
branch fixed:
    +

row counter:
    8 / 8 rows cleared

prime verdict:
    every row above 97 passes
    -> 97 ∤ h_5^+

what made the verdict legal:
    not any single residue value,
    but the completion of the whole 8-row counter
```

这里最值得 later-layer 读者学的，
不是 `97` 这个具体 prime，
而是：

> prime verdict  
> 永远是  
> **completed row counter**
> 的产物。

**最后把这个 concrete page 回带到 later-layer**

现在再回头看 `k=10,11,12`，
你应该把它们听成：

1. 不是在换一种不同的 row 逻辑；
2. 只是把这张 `k=5,\ell=97` 的完整 row counter，
   放到更大的 \(g\)、
   更大的 residue field、
   以及更重的 arithmetic backend 上。

也就是说，later-layer 真正新增的不是 row semantics，
而是 row count 和数值规模。

所以更成熟的 later-layer 读法不该只是说：

> “那里也要做 residue tests。”

而该更准确地说：

> “later-layer 只是把 `k=5,\ell=97` 这张 8-row counter  
> 推广成 branch-sensitive 的 128/256/512/1024-row style audit。”

**最后把这一页压成一句最该带走的话**

如果你已经能把 `k=5,\ell=97` 的八条 rows
真的抄成一张完整 counter，
later-layer 的 row audit 就不再只是抽象口号。
你会更清楚地知道：

> 一条 row 只是一个 local site；  
> 一个 prime verdict 是一张完成的 row counter；  
> later-layer 真正变大的，不是逻辑形状，而是这张 counter 的尺寸。

#### 一个更接近 later-layer 的 branch-aware training walkthrough：`k=9` 的 `257` 与 `1279`

上面的 `k=5,\ell=97` 页已经把一张完整 row counter
真的抄出来了。

但它 still 有一个局限：

> 它只有 `Branch +`，  
> 而且 row 总数只有 `8`。

如果读者下一步直接跳到 later-layer，
仍然可能低估两件事：

1. `Branch -` 不是把 `Branch +` 指数里平方一下就完了；
2. row 数一旦从 `8` 长到 `64 / 32 / 256 / 128 ...`，阅读节奏会明显变。

所以这里再补一页 **更接近 later-layer 的训练样例**，
但仍然坚持一个原则：

> 不伪造 theorem candidate，  
> 只用本地真能算出来的 branch-aware artificial primes  
> 训练 later-layer 的 row grammar。

这一页选用的不是 paper 里的 surviving primes，
而是两个人工训练 prime：

\[
k=9,\qquad \ell_+ = 257 \equiv 1 \pmod{256},
\qquad
\ell_- = 1279 \equiv -1 \pmod{256}.
\]

它们都只是 **branch-aware training primes**，
不是 theorem candidate。
这里故意不要求它们来自 Stage A surviving set，
也不要求满足 `>10^9` cutoff。
这页唯一目标是：

> 让你看到 later-layer 附近的 `Branch +` 与 `Branch -`  
> 在真实局部域行长上到底长什么样。

**先把两条训练样例的 branch data 一次写清**

对 `k=9`，

\[
n_9=[K_9^+:\mathbb{Q}] = 2^{9-2}=128.
\]

但这里要先分清两层 bookkeeping：

1. `mod 2^{k-1}` 的 `+/-` branch 决定 **exponent formula**；
2. `mod 2^k` 的 lift 决定 Frobenius 在
   \[
   G_k^+=\left(\mathbb{Z}/2^k\mathbb{Z}\right)^\times/\{\pm1\}
   \]
   里的阶，从而决定 \(K_k^+\) 里 **真的有多少个 real local sites**。

对这两个人工训练 prime，

\[
257 \equiv 257 = 1+256 \pmod{512},
\qquad
1279 \equiv 255 = -(1+256) \pmod{512}.
\]

也就是说，它们虽然分别属于 `Branch +` 与 `Branch -` 的 exponent family，
但在 `mod 512` 的 lift 层面都属于 **dirty lift**。
因此本地 Sage 真正看到的 real row count，
两边都是 `64`。

把这两层分开以后，
这页的训练样例 summary 更准确地应该写成：

| training prime | exponent branch | lift mod \(2^9\) | observed row count in \(K_9^+\) | local norm / exponent |
| --- | --- | --- | --- | --- |
| \(\ell_+=257\) | Branch + because \(257\equiv1\pmod{256}\) | dirty: \(257\equiv1+256\pmod{512}\) | \(64\) | \(N(\mathfrak l)=257,\ E_{\mathfrak l}=(257-1)/256=1\) |
| \(\ell_-=1279\) | Branch - because \(1279\equiv-1\pmod{256}\) | dirty: \(1279\equiv-(1+256)\pmod{512}\) | \(64\) | \(N(\mathfrak l)=1279^2,\ E_{\mathfrak l}=(1279^2-1)/256=6390\) |

这张表最值得 later-layer 读者记住的是：

1. row 数已经不再是 `8`；
2. `Branch +` 与 `Branch -` 真正改变的是 exponent family；
3. 但 **actual row count 并不只由 `+/-` branch 决定**，还要看 `mod 2^k` 的 lift；
4. 因而这两条 `k=9` 训练样例虽然分属不同 exponent branch，却恰好都落在 `64`-row dirty-lift regime。

**现在看 Branch + 样例：257 的前四条真实 rows**

本地 Sage 在 `K_9^+` 中对 \(\ell_+=257\) 的前四条 rows
给出的真实输出是：

```text
ell 257 count 64 exp 1
1 Fractional ideal (257, zeta5120^2 + 3)   -> 19   != 1
2 Fractional ideal (257, zeta5120^2 + 6)   -> 55   != 1
3 Fractional ideal (257, zeta5120^2 + 10)  -> 131  != 1
4 Fractional ideal (257, zeta5120^2 + 20)  -> 204  != 1
```

把它压成 row ledger，
最短该写成：

| row id | local site | \(E_{\mathfrak l}\) | \(r_{\mathfrak l}\) | row verdict | what still remains |
| --- | --- | --- | --- | --- | --- |
| P1 | \((257,\zeta_{5120}^2+3)\) | \(1\) | \(19\) | pass | still need 63 more rows |
| P2 | \((257,\zeta_{5120}^2+6)\) | \(1\) | \(55\) | pass | still need 62 more rows |
| P3 | \((257,\zeta_{5120}^2+10)\) | \(1\) | \(131\) | pass | still need 61 more rows |
| P4 | \((257,\zeta_{5120}^2+20)\) | \(1\) | \(204\) | pass | still need 60 more rows |

这四条 rows 真正最重要的教学点，
不是 `19 / 55 / 131 / 204` 这些数本身，
而是：

> 即便前四条全部 pass，  
> 你离 prime verdict 仍然还差 `60` 条 rows。

所以 later-layer 的 `Branch +` 读法，
绝对不能再保留 `k=5` 时那种
“八条差不多一眼能数完”的直觉。

**再看 Branch - 样例：1279 的前四条真实 rows**

对 \(\ell_-=1279\)，
本地 Sage 给出的前四条 rows 则是：

```text
ell 1279 count 64 exp 6390
1 Fractional ideal (1279, zeta5120^2 + 7)    -> 1
2 Fractional ideal (1279, zeta5120^2 + 8)    -> 1
3 Fractional ideal (1279, zeta5120^2 + 63)   -> 1
4 Fractional ideal (1279, zeta5120^2 + 143)  -> 1
```

这里最关键的不是 “很多 row 都是 `1`”，
而是：

> 第一条 row 一旦已经等于 `1`，  
> 你就已经知道  
> 这个训练 prime **不会** 被这张 local no-certificate 删掉。

压成 row ledger，
最短可以写成：

| row id | local site | \(E_{\mathfrak l}\) | \(r_{\mathfrak l}\) | row verdict | what this already means |
| --- | --- | --- | --- | --- | --- |
| M1 | \((1279,\zeta_{5120}^2+7)\) | \(6390\) | \(1\) | fail | prime already unresolved by this certificate |
| M2 | \((1279,\zeta_{5120}^2+8)\) | \(6390\) | \(1\) | fail | nothing can upgrade this to a no-certificate |
| M3 | \((1279,\zeta_{5120}^2+63)\) | \(6390\) | \(1\) | fail | more rows may be computed, but elimination is already lost |
| M4 | \((1279,\zeta_{5120}^2+143)\) | \(6390\) | \(1\) | fail | branch remains unresolved for this training prime |

这张表真正最值得 later-layer 读者带走的是：

> `Branch -` 的阅读困难  
> 不只是指数变成了 `6390`；  
> 而是你必须立刻听懂  
> `first failing row already blocks prime elimination`。

这和 `Branch +` 的阅读节奏完全不同：

1. `Branch +` 更像 long counter；
2. `Branch -` 更像 early failure gate。

**把两条训练样例真正放到一张对照表里**

| question | Branch + training prime \(257\) | Branch - training prime \(1279\) |
| --- | --- | --- |
| observed row count | \(64\) | \(64\) |
| first four rows | all pass | all fail immediately |
| what the first row tells you | almost nothing beyond one local pass | prime already unresolved by this certificate |
| reading rhythm | long counter, no early verdict | first failure already decides no-certificate is lost |
| common overclaim to resist | “前几行都 pass 了，prime 大概快删掉了” | “既然已经 fail，就不必再分清这是 row verdict 还是 prime verdict 的入口” |

这张对照表最有价值的地方，
是它第一次把 later-layer 附近的两种阅读节奏并排放到同一页：

1. `slow accumulation`；
2. `early obstruction`。

而且两者都是真实局部域输出，
不是 synthetic sentence。

**最后把这页回带到真正 later-layer**

现在再回去看 `k=10,11,12`，
更成熟的读法应该是：

1. 不要只问 “这一层有多少 rows？”；
2. 还要问 “这一层更像 `257` 这种 long counter，还是更像 `1279` 这种 early obstruction？”；
3. 更不要把 `Branch -` 听成 “只是把 \(\ell\) 换成 \(\ell^2\)”。

因为对 later-layer 真正重要的，
不是符号上看到了平方，
而是：

> row semantics、row count、以及 first failing row 的逻辑位置  
> 都变了。

**最后把这一页压成一句最该带走的话**

`k=5,\ell=97` 教你什么叫完整 row counter；  
`k=9,\ell=257` 教你 later-layer 附近的 `Branch +` 会怎样把 counter 拉长；  
`k=9,\ell=1279` 则教你 `Branch -` 的第一条 failing row 就足以让这张 local no-certificate 失效。

如果这三页能连起来读，
later-layer final residue elimination 就不再只是公式模板，
而会开始有真正的 branch-aware row 手感。

#### 一张 lift-aware row-budget sanity page：branch 决定 exponent，lift 决定 real row count

上面的 `k=9` branch-aware training page
已经把 later-layer 附近最关键的两种 row rhythm
都演给你看了：

1. `257` 这种 `long accumulation`；
2. `1279` 这种 `early obstruction`。

但这页 still 留着一个隐藏 debt：

> **Branch + / Branch -**  
> 只告诉了你 exponent 该用哪一族公式；  
> 它还没有单独告诉你  
> `K_k^+` 里到底会出现多少条 real rows。

这正是这一页要补的地方。

**先把 clean lift / dirty lift 分开**

对后文真正相关的 suspect primes，
我们已经知道：

\[
\ell \equiv \pm1 \pmod{2^{k-1}}.
\]

但这只固定了 exponent branch。
若继续往 `mod 2^k` 再看一层，
还会出现两种不同的 Frobenius lift：

| lift type in \(G_k^+\) | congruence mod \(2^k\) | Frobenius order in \(G_k^+\) | real row count in \(K_k^+\) |
| --- | --- | --- | --- |
| clean lift | \(\ell \equiv \pm1 \pmod{2^k}\) | \(1\) | \(g=n_k=2^{k-2}\) |
| dirty lift | \(\ell \equiv \pm(1+2^{k-1}) \pmod{2^k}\) | \(2\) | \(g=n_k/2=2^{k-3}\) |

所以 later-layer 的更成熟读法必须把两句话分开：

1. `+/-` branch 决定 exponent formula；
2. clean / dirty lift 决定 real row count 是 `n_k` 还是 `n_k/2`。

**先用已经算出来的三条真证据把这件事钉死**

下面三条都是本地 Sage 已经真正跑出来的 later-layer-style evidence：

| layer-prime | exponent branch | lift class | observed row count |
| --- | --- | --- | --- |
| k=9, \ell=257 | Branch + | dirty since \(257\equiv257\pmod{512}\) | 64 |
| k=9, \ell=1279 | Branch - | dirty since \(1279\equiv255\pmod{512}\) | 64 |
| k=10, \ell=7681 | Branch + | dirty since \(7681\equiv513\pmod{1024}\) | 128 |

这张表正好解释了前面最容易让人困惑的现象：

> `257` 明明属于 `Branch +`，  
> 为什么本地输出 still 是 `count 64`？

答案不是 Sage 算错了，
而是：

> `Branch +` 只决定 exponent 用 \((\ell-1)/2^{k-1}\)；  
> `257` 这个具体 training prime 又恰好是 dirty lift，  
> 所以 real row count 被再砍半。

**现在把 k=10,11,12 的 row budget 真正改写成 lift-aware bookkeeping**

对真正 later layers，
最该抄在 notebook 顶部的不是一句 “`Branch +` 就是 256 rows”，
而是下面这张更稳的表：

| layer | \(n_k=[K_k^+:\mathbb{Q}]\) | clean-lift row count | dirty-lift row count |
| --- | --- | --- | --- |
| k=10 | \(256\) | \(256\) | \(128\) |
| k=11 | \(512\) | \(512\) | \(256\) |
| k=12 | \(1024\) | \(1024\) | \(512\) |

如果你只记得 branch，
而忘了再看一眼 clean / dirty lift，
later-layer 的 row budget 很容易会在第一行就记错一半。

**再把前四条 rows 的 bookkeeping 改成 clean / dirty 两列**

如果只做一个最短 sanity check，
真正该写成下面这样：

| layer-lift | after 1 passed row | after 4 passed rows |
| --- | --- | --- |
| k=10, clean | cleared \(1/256 \approx 0.39\%\), still need 255 rows | cleared \(4/256 = 1.5625\%\), still need 252 rows |
| k=10, dirty | cleared \(1/128 \approx 0.78\%\), still need 127 rows | cleared \(4/128 = 3.125\%\), still need 124 rows |
| k=11, clean | cleared \(1/512 \approx 0.20\%\), still need 511 rows | cleared \(4/512 = 0.78125\%\), still need 508 rows |
| k=11, dirty | cleared \(1/256 \approx 0.39\%\), still need 255 rows | cleared \(4/256 = 1.5625\%\), still need 252 rows |
| k=12, clean | cleared \(1/1024 \approx 0.10\%\), still need 1023 rows | cleared \(4/1024 = 0.390625\%\), still need 1020 rows |
| k=12, dirty | cleared \(1/512 \approx 0.20\%\), still need 511 rows | cleared \(4/512 = 0.78125\%\), still need 508 rows |

这才是 later-layer 里真正稳的 partial-audit 尺度感。

**这张表该怎样和 early obstruction 并排读**

上表仍然只是在说：

> 如果前缀 rows 一路都 pass，  
> 那么 bookkeeping 上还剩多少。

它并没有取消前一页 `Branch -` 的核心手感：

> 第一条 fail  
> 仍然可能立刻让这张 local no-certificate 失效。

所以 later-layer 的 row reading
要同时保留两种 reflex：

1. `long accumulation`：前缀 rows 全过时，记账 still 离 prime verdict 很远；
2. `early obstruction`：某条 row 一旦 fail，prime 立刻转入 `unresolved by this certificate`。

**把 later-layer partial audit 的两句合法状态话写死**

如果你以后真拿着 `k=10,11,12` 的 notebook 在读，
最稳的不是凭感觉说“应该快了”或“应该没了”，
而是固定只允许自己说下面两种句子。

对任意 clean / dirty lift 的 long-counter 前缀，
合法状态话应该是：

```text
p rows have cleared p local sites only;
still need g - p more rows before any prime verdict.
```

对 `Branch -` 的 first-failure 情形，
合法状态话应该是：

```text
this row makes the prime unresolved by this local no-certificate;
later rows may still be inspected, but they cannot upgrade it back to elimination.
```

这两句话看起来朴素，
但它们正好把 later-layer 最容易偷并的两种错话堵死了：

1. “算了几条，prime 大概快删掉了”；
2. “既然已经 fail 了，后面就等于整个 proof 也没信息了”。

第一句错在把 row prefix 假装成 prime verdict；
第二句错在把 `certificate lost` 和 `proof useless` 混成一件事。

**把 k=9 训练页真正翻译成 k=10,11,12 的阅读 reflex**

现在可以把前面的三页 training walkthrough
再向 later-layer 自身翻译半步：

1. `k=5,\ell=97` 教你：什么叫 **complete counter**。
2. `k=9,\ell=257` 教你：什么叫 **long accumulation**。
3. `k=9,\ell=1279` 教你：什么叫 **early obstruction**。
4. 这一页则把前三页统一翻译成一句更接近 `k=10,11,12` 的 later-layer reflex：

> 先看 exponent branch，  
> 再看 clean / dirty lift，  
> 然后只允许自己说合法的 partial-audit 状态话。

如果这个 reflex 没形成，
later-layer 的 row audit
就 still 会在 “已经算了几条 / 已经出现一个 fail / 差不多懂了”
这类口头直觉里打转。

**最后把这一页压成一句最该带走的话**

对 `k=10,11,12`，
later-layer 最危险的错觉
不是“不知道公式”，
而是低估了 partial audit 的贫弱程度。

更稳的读法必须始终记住：

> exponent branch 决定公式是哪一族；  
> clean / dirty lift 决定 real row count 是整份还是半份；  
> 一条 fail 又仍然可能立刻让这张 no-certificate 失效；  
> 所以 later-layer 真正该练的，不是“看几条 row 猜结论”，而是 branch-aware, lift-aware bookkeeping。

#### 一个 clean-lift 对照页：同一个 prime `7681` 在 `k=9` 里为什么回到整份 `128` rows

上面那页是 `k=10` 的 dirty-lift walkthrough。
但现在已经可以补上它的 clean-lift 对照页了。

同一个 prime `7681`，
在 `k=9` 时恰好满足

\[
7681 \equiv 1 \pmod{512},
\]

所以它不是 dirty lift，而是 **clean lift**。
这件事的教学价值很强，
因为它直接说明：

> **row count = 128**
> 这件事本身并不告诉你它是 clean 还是 dirty；
> 你还要同时看 `k` 和 `mod 2^k` 的 lift。

**先把 k=9 的 clean-lift summary 写死**

对 `k=9`，

\[
n_9=[K_9^+:\mathbb{Q}] = 2^{9-2}=128.
\]

对这条训练 prime `7681`，
本地 Sage 给出的真实输出是：

```text
ell 7681 count 128 exp 30
1 Fractional ideal (7681, zeta5120 + 220) -> 1848 != 1
2 Fractional ideal (7681, zeta5120 + 284) -> 6090 != 1
3 Fractional ideal (7681, zeta5120 + 451) -> 528 != 1
4 Fractional ideal (7681, zeta5120 + 457) -> 6090 != 1
```

把它压成 row ledger，
最短该写成：

| row id | local site | \(E_{\mathfrak l}\) | \(r_{\mathfrak l}\) | row verdict | what still remains |
| --- | --- | --- | --- | --- | --- |
| C1 | \((7681,\zeta_{5120}+220)\) | \(30\) | \(1848\) | pass | still need 127 more rows |
| C2 | \((7681,\zeta_{5120}+284)\) | \(30\) | \(6090\) | pass | still need 126 more rows |
| C3 | \((7681,\zeta_{5120}+451)\) | \(30\) | \(528\) | pass | still need 125 more rows |
| C4 | \((7681,\zeta_{5120}+457)\) | \(30\) | \(6090\) | pass | still need 124 more rows |

这张表补上的不是一个新技巧，
而是一个很关键的 lift lesson：

1. `k=9` 里 `7681` 是 clean lift，所以 rows 回到整份 `128`；
2. `k=10` 里同一个 `7681` 会变成 dirty lift，所以 rows 只剩 `128` 的另一种来路；
3. 所以 later-layer 的 `row count` 不是只看一个 prime 的 residue 类，而是看 `k`、branch、lift 三者一起决定的。

**把 clean-lift 和 dirty-lift 放到同一张最短对照表里**

| case | exponent branch | lift class | observed row count | what this teaches |
| --- | --- | --- | --- | --- |
| k=9, \ell=7681 | Branch + | clean | 128 | clean lift can restore the full row budget |
| k=10, \ell=7681 | Branch + | dirty | 128 | dirty lift can also land on 128, but for a different reason |

这张对照表最值得读者记住的是：

> `128` 不是一个自动等于 clean 或 dirty 的数；  
> 它只是一个 row count。  
> 你必须继续追问它是从哪一层 `k`、哪一类 lift、哪一个 branch 里来的。

**最后把这一页压成一句最该带走的话**

同一个 prime `7681`，
在 `k=9` 里是 clean lift 的 `128` rows，
在 `k=10` 里又可以是 dirty lift 的 `128` rows。
所以 later-layer 读法真正要练的，
不是先记住一个数字，
而是先确认：

> 这个数字是由哪一层 `k`、哪一族 exponent branch、哪一种 lift 共同决定的。

#### 一个真正更像 later-layer 自身的 dirty-lift local-field walkthrough：`k=10` 的 `7681`

上面的 lift-aware row-budget page
已经把 later-layer 的 bookkeeping
从 “只看 `+/-` branch” 修正成了：

1. 先看 exponent branch；
2. 再看 clean / dirty lift；
3. 最后才谈 real row count 与 partial audit。

但它 still 主要停在 counting 层。

如果想把这种修正真正压成 local-field 手感，
最直接的方法就是看一页 **真的已经进入 `k=10` 的 dirty-lift rows**。

这一页选的训练 prime 是：

\[
k=10,
\qquad
\ell = 7681.
\]

它不是 theorem candidate，
只是一条 later-layer training prime。
之所以选它，
是因为它刚好把这两层 bookkeeping 同时摆在台面上：

\[
7681 \equiv 1 \pmod{512},
\qquad
7681 \equiv 513 = 1+512 \pmod{1024}.
\]

因此：

1. 在 exponent family 上，它属于 `Branch +`；
2. 在 `mod 1024` 的 lift 上，它又是 dirty lift；
3. 所以它的 actual real row count 不是 `256`，而是 `128`。

**先把这页最该抄的 branch/lift summary 写死**

对 `k=10`，

\[
n_{10}=[K_{10}^+:\mathbb{Q}] = 2^{10-2}=256.
\]

但对这条训练 prime，
真正进 notebook 的 summary 应该是：

```text
layer = k=10
prime ell = 7681
exponent branch = +
lift class = dirty
observed real row count = 128
E_l = (7681 - 1)/512 = 15
```

这六行最重要的教学点是：

> **Branch +**
> 并不 automatically 等于
> `256 real rows`.

如果你不把 dirty lift 这一格单独抄出来，
later-layer 的第一页就会在 row budget 上记错。

**现在把前四条真 rows 真抄出来**

本地 Sage 在 `K_{10}^+` 中对这条 training prime
给出的真实输出是：

```text
ell 7681 count 128 exp 15
1 Fractional ideal (7681, zeta10240^2 + 218) -> 5685 != 1
2 Fractional ideal (7681, zeta10240^2 + 282) -> 202  != 1
3 Fractional ideal (7681, zeta10240^2 + 449) -> 6637 != 1
4 Fractional ideal (7681, zeta10240^2 + 455) -> 6825 != 1
```

压成最短 row ledger，
应当写成：

| row id | local site | \(E_{\mathfrak l}\) | \(r_{\mathfrak l}\) | row verdict | what still remains |
| --- | --- | --- | --- | --- | --- |
| D1 | \((7681,\zeta_{10240}^2+218)\) | \(15\) | \(5685\) | pass | still need 127 more rows |
| D2 | \((7681,\zeta_{10240}^2+282)\) | \(15\) | \(202\) | pass | still need 126 more rows |
| D3 | \((7681,\zeta_{10240}^2+449)\) | \(15\) | \(6637\) | pass | still need 125 more rows |
| D4 | \((7681,\zeta_{10240}^2+455)\) | \(15\) | \(6825\) | pass | still need 124 more rows |

这张表最值得 later-layer 读者带走的，
不是 `5685 / 202 / 6637 / 6825` 这些数本身，
而是：

> 这已经不是 `k=9` 的近 later-layer 训练页，  
> 而是真正在 `k=10` 里的 real local-field page；  
> 但即便这样，前四条 pass 仍然只清掉了 `4/128 = 3.125%`。

也就是说，
later-layer 的 row audit 一旦真的进入 `k=10`，
读者必须 simultaneously 记住两件事：

1. 这是 actual local rows，不再只是 formula skeleton；
2. 但 partial audit 的信息量仍然极其有限。

**把这页和 k=9,257 放成一张最短对照表**

| question | k=9,\ell=257 | k=10,\ell=7681 |
| --- | --- | --- |
| exponent branch | + | + |
| lift class | dirty | dirty |
| observed row count | 64 | 128 |
| first four rows | all pass | all pass |
| what four passes still mean | still far from prime verdict | still far from prime verdict |
| what got harder | row budget doubled | local page now truly sits in later-layer itself |

这张表真正想训练的是：

> later-layer 的“更难”  
> 不只是 residue 数字更大，  
> 而是你开始同时背负  
> **true local page**  
> 和  
> **still tiny partial progress**
> 这两种感觉。

**最后把这页压成一句最该带走的话**

`k=9,\ell=257` 还像 near later-layer 的 branch-aware training；  
`k=10,\ell=7681` 则已经把读者真正推进到 later-layer 自身。

但这页最硬的 lesson 仍然不是某个 residue 值，
而是：

> 先看 exponent branch，  
> 再看 clean / dirty lift，  
> 再看 actual row count，  
> 最后才允许自己说“还差多少 rows 才有 prime verdict”。

#### 一个 computation-boundary completeness page：更大 `k` 没有快速 row 输出时，讲义仍然应该完整

到 `k=10` 以后，最容易出现的写作诱惑是：

> 既然还想给读者一个更像 paper-scale 的 local-field walkthrough，  
> 那就继续找 prime，继续跑 Sage，直到拿到一整页 rows。

这当然是理想路线，但它不是讲义完成度的唯一标准。
对 `k=10,11,12` 这种 later-layer page，更成熟的写法必须承认一个现实边界：

> 若实验在短时间内没有产出完整 row list，  
> 不应把计算等待伪装成证明进展；  
> 应优先把已经确定的 branch / lift / row-budget / certificate obligation 写完整。

这不是降低标准，而是防止两类更严重的问题：

1. 把没有跑完的局部计算写成好像已经有 no-certificate。
2. 为了补“更大 \(k\)”的视觉完整性，误写或臆造 \(\mathfrak l\)-rows。

**以 k=10, ell=12289 为边界例子**

一个自然候选是
\[
k=10,\qquad \ell=12289.
\]

它满足
\[
12289\equiv 1\pmod{1024},
\]
所以它是 `Branch + / clean lift` 的训练候选。
因此在 purely local-certificate bookkeeping 上，已经可以无歧义地写出：

\[
n_{10}=2^{10-2}=256,
\qquad
g=256,
\qquad
E_{\mathfrak l}=\frac{12289-1}{512}=24.
\]

这些都是 **可写入 notebook 的确定事实**。
它们不依赖 `primes_above(12289)` 是否已经返回完整列表。

但真正的 Phase-B no-certificate 还需要另一组事实：

\[
\begin{aligned}
\forall\,\mathfrak l\mid 12289,\qquad
r_{\mathfrak l}
&=
\xi_5^{24}\bmod \mathfrak l \\
&\neq 1.
\end{aligned}
\]

这一句目前不能写成结论。
原因很简单：
本地实验已经完成 field、real subfield、\(\xi_5\) 与 exponent 的准备，
但 `K_{10}^+.primes_above(12289)` 没有在短时间内返回完整 row list。
所以这页最多只能写成 **computation boundary record**，
不能写成 local-field walkthrough。

**边界页应该怎样写才算完整**

对一个没有快速产出 rows 的 later-layer prime，讲义可以、也应该写满下面六格：

| slot | 可以写什么 | 不能越界写什么 |
| --- | --- | --- |
| layer | \(k\)、\(n_k\)、目标 field \(K_k^+\) | 不要暗示这一层 theorem 已闭合 |
| prime | \(\ell\) 的同余类与是否为训练 prime | 不要暗示它来自论文 surviving set，除非已验证 |
| branch | Branch + 或 Branch - | 不要把 branch 当作 prime verdict |
| lift | clean / dirty lift 与 expected row count \(g\) | 不要把 expected \(g\) 当作已经枚举了 \(g\) 条 rows |
| setup | \(\xi_5\)、\(E_{\mathfrak l}\)、local certificate 公式 | 不要写任何未计算的 residue 值 |
| boundary | 卡在哪个确定步骤，例如 primes_above 未返回 | 不要把 “未返回” 翻译成 “可能失败” 或 “可能通过” |

这张表的教学价值在于：

> 即使没有完整 rows，读者仍然能学会  
> later-layer local certificate 到底欠哪些数据。

换句话说，文档完整性不是“强行补满实验输出”，而是把 proof obligation 的形状写到不能误读。

**把 completed walkthrough 与 boundary record 分开**

成熟讲义应该强制区分三种页面：

| page type | 允许 claim 什么 | 例子 |
| --- | --- | --- |
| complete row counter | 所有 rows 已列出或已审计，可以推出 prime verdict | k=5,\ell=97 |
| partial training walkthrough | 只展示真实前缀 rows，并持续声明 still need rows | k=9,\ell=257、k=10,\ell=7681 |
| computation boundary record | 只写 branch、lift、\(g\)、\(E_{\mathfrak l}\) 与阻塞点，不写 residue verdict | k=10,\ell=12289 |

这三类页面不能混用。

- complete counter 可以承担 prime-level no-certificate。
- partial walkthrough 只能训练 row-reading reflex。
- boundary record 只能说明下一步实验应该从哪里继续。

如果把第三类写成第二类，读者会以为已经有真实 rows。
如果把第二类写成第一类，读者会以为 prime 已经被删掉。
这两种误读都比“少一页更大 \(k\) 的实验例子”更糟。

**更大 k 的优先级规则**

因此后续补 `k=10,11,12` 时，应采用下面这条更稳的策略：

```text
first write the certificate obligation completely
then add real rows only when they are actually available
never let slow computation weaken the proof-state bookkeeping
```

对读者来说，这条规则反而更接近原论文的精神。
因为 Theorem 3.3 的无条件性，本来就不是靠“看起来算了很多 rows”获得的，
而是靠：

1. theorem chain 把风险压成 finite suspect set；
2. 每个 suspect prime 都有明确 local certificate obligation；
3. 只有完整 local audit 才能产生 prime verdict；
4. 所有 prime verdict 合在一起才产生 layer closure。

所以在更大的 \(k\) 上，如果实验没有快速结果，讲义最应该保住的不是“多一页漂亮输出”，而是：

> 每一条尚未完成的计算，  
> 都被准确标成尚未完成；  
> 每一个已经确定的 branch / lift / exponent / row-budget，  
> 都被准确写进 proof notebook；  
> 每一个 prime verdict，  
> 都只在完整 row audit 后才被允许出现。

#### 一张 `k=11/12` no-fast-output audit ledger：没有 rows 时也要把 proof state 写完整

前一页用 `k=10,\ell=12289` 说明了一个具体边界：

> branch、lift、\(g\)、\(E_{\mathfrak l}\) 都已经确定，  
> 但完整 \(\mathfrak l\)-row list 没有快速产出，  
> 因而正文只能写成 boundary record。

如果继续往 `k=11`、`k=12` 推，这条规则反而更重要。
因为这里的 shared norm、local site count、field construction 与 ideal decomposition 都更重；
一旦没有快速完整输出，最容易发生的不是“证明慢一点”，而是 proof-state 语言开始变脏：

1. 把 expected row count 听成 observed row list。
2. 把 setup completed 听成 local certificate completed。
3. 把 partial rows 听成 prime verdict。
4. 把一个 prime 的 boundary record 听成 layer-level progress。

所以更大的 \(k\) 上，讲义优先追求的不是“看起来又多算了一个大例子”，而是把下面这张 audit ledger 先写完整。

**先把 k=11/12 不会变的参数骨架写死**

对 `k=11`：

\[
n_{11}=2^{11-2}=512,\qquad
2^{k-1}=1024,\qquad
2^k=2048.
\]

对 `k=12`：

\[
n_{12}=2^{12-2}=1024,\qquad
2^{k-1}=2048,\qquad
2^k=4096.
\]

这三项分别回答三件完全不同的问题：

1. \(n_k\) 控制 clean lift 时完整 row counter 的最大长度；
2. \(2^{k-1}\) 控制 `Branch + / Branch -` 的 exponent family；
3. \(2^k\) 控制 clean / dirty lift，从而控制实际 row count。

如果这三件事没有分开写，后面的 notebook 很容易一开始就把 branch 和 lift 混成一格。

**把 k=11/12 的 boundary skeleton 直接列成表**

| layer | branch modulus | lift modulus | clean-lift rows | dirty-lift rows | Branch + exponent | Branch - exponent |
| --- | ---: | ---: | ---: | ---: | --- | --- |
| k=11 | \(1024\) | \(2048\) | \(512\) | \(256\) | \((\ell-1)/1024\) | \((\ell^2-1)/1024\) |
| k=12 | \(2048\) | \(4096\) | \(1024\) | \(512\) | \((\ell-1)/2048\) | \((\ell^2-1)/2048\) |

这张表已经是一个合格 boundary record 的核心。
它没有给出任何 residue value，
也没有 claim 任何 prime verdict；
但它让读者知道：

> 一旦拿到某个 surviving \(\ell\)，  
> 首先该核对哪几个 deterministic slots。

**再给一张真正可填的 no-fast-output card**

以后如果拿到一个 `k=11` 或 `k=12` 的候选 prime，
但本地实验没有快速返回完整 rows，
不要写“实验未完成”四个字就结束。
应该至少填下面这张 card：

```text
layer:
candidate prime ell:
candidate source:
    from paper surviving set / local Stage-A run / artificial training prime
branch evidence:
    ell mod 2^(k-1) =
    Branch + or Branch -
lift evidence:
    ell mod 2^k =
    clean or dirty
expected full row count:
    g =
exponent obligation:
    E_l = ...
setup completed:
    field built? real subfield built? xi_5 represented? exponent fixed?
blocking step:
    factorization / primes_above(ell) / row residue computation / verification audit
allowed claim:
    boundary record only / partial walkthrough only / complete counter
forbidden claim:
    no prime verdict until all rows are audited
next reproducible action:
    exact command or script entry to resume
```

这张 card 的关键是 `candidate source` 和 `allowed claim` 两格。
如果 \(\ell\) 只是人工训练 prime，
就不能把它写成 paper surviving candidate。
如果只完成了 field setup，
就不能把它写成 partial walkthrough。
如果只拿到前几条 rows，
就不能把它写成 complete counter。

**把三种“没有快速结果”的状态分清**

| state | 已经拿到什么 | 正文允许怎么写 | 仍然欠什么 |
| --- | --- | --- | --- |
| setup-only boundary | \(k\)、\(\ell\)、branch、lift、\(g\)、\(E_{\mathfrak l}\) | boundary record | 完整 \(\mathfrak l\)-list 与 residues |
| row-prefix partial | 前 \(p<g\) 条真实 rows | partial training walkthrough，并持续写 still need g-p rows | prime verdict |
| complete counter | 全部 \(g\) 条 rows，且每条 verdict 已核 | 可以推出 prime-level no-certificate | 若要 layer closure，还欠其余 surviving primes |

这三种状态必须在标题、表格和结论句里同时保持一致。
标题写 `boundary`，正文就不能突然说 “therefore \(\ell\nmid h_k^+\)”。
标题写 `partial`，最后一列就必须持续出现 `still need ... rows`。
只有标题写 `complete counter`，才允许把 prime verdict 写进正文。

**为什么这仍然是在推进讲义，而不是回避计算**

对 `k=11`、`k=12`，没有快速完整 rows 时，补这张 ledger 仍然是有效进展。
原因是这份讲义的目标不是伪装成一个计算日志仓库，
而是让读者能独立审计论文证明。

独立审计至少包括两种能力：

1. 看见完整 local certificate 时，知道它为什么足以删掉一个 prime。
2. 看见不完整实验记录时，知道它到底缺哪一格，不能 claim 什么。

第二种能力在大 \(k\) 上尤其重要。
因为真正的 proof discipline 不是“只在成功输出时才清楚”，
而是：

> 即使计算暂时停在边界处，  
> 读者仍然能准确说出  
> 当前 proof state 已经赚到什么、  
> 尚未赚到什么、  
> 下一步必须补哪一个可复现证据。

**把这页压成后续工作的优先级规则**

后面若继续补 later-layer local pages，
优先级应该固定成：

```text
1. complete the proof-obligation card
2. record branch / lift / row budget / exponent
3. attach only real rows that actually exist
4. label the page as boundary / partial / complete
5. claim prime verdict only after the complete row counter exists
```

这条规则比继续盲跑更重要。
因为一个没有完成 rows 的 `k=11/12` 页面，
只要边界写得清楚，
仍然能提升读者的 proof audit 能力；
而一个把 incomplete computation 讲得像 complete certificate 的页面，
会直接破坏整篇讲义最核心的无条件性纪律。

### 9.11 一个玩具级 Python 过滤器

下面这段不是原论文的完整实现，只是把 `Theorem 3.1` 的两个出口变成最小可运行形状：

```python
def sieve_candidates(candidates, k, cutoff=10**9):
    modulus = 1 << (k - 1)
    kept = []
    for ell in candidates:
        if ell <= cutoff:
            continue
        residue = ell % modulus
        if residue in (1, modulus - 1):
            kept.append(ell)
    return kept


toy_candidates = [21121, 1_000_000_193, 1_000_000_257, 1_000_000_321]
print(sieve_candidates(toy_candidates, k=9))
```

这段代码教的不是数论本身，而是 proof workflow：

1. 先产生候选。
2. 再按 theorem 给出的必要条件筛掉大多数。
3. 最后只对极小残余集合做贵价测试。

### 9.12 SageMath / PARI-GP 最小入口

**SageMath**

下面的 toy example 不复现论文规模，只是让读者确认“maximal real subfield”是可计算对象，而不是抽象名词：

```python
K = CyclotomicField(32)
Kplus, emb = K.maximal_totally_real_subfield()
print(Kplus.defining_polynomial())   # x^8 - 8*x^6 + 20*x^4 - 16*x^2 + 2
print(Kplus.degree())                # 8
print(Kplus.class_number())          # 1
```

**PARI/GP**

同一个 toy field 在 PARI/GP 里可以直接这样起步：

```gp
P = polsubcyclo(32, 8)[3];
bnf = bnfinit(P, 1);
bnf.no    \\ class number, here 1
```

这两段示例的教学作用是：

- 让读者知道“real cyclotomic subfield”可以被显式拿到；
- 让读者感受到 class number computation 的最小接口；
- 为后续自己尝试更小层数的 \(k\) 准备工具入口。
- 与上面的 `9.5`、`9.6` 合起来看，就能把 “域对象 -> character -> Bernoulli number -> norm -> candidate primes” 这条计算链拆成可独立上手的几段。

## 10. 回到密码学：Section 4 不只是应用场景点缀

Section 4 的真正价值，不是“顺手提一下后量子密码”，而是把前面那些看似纯数论的对象重新翻译回 structured lattices 的实际语言。

### 10.1 为什么 \(h_9^+ = 1\) 会正中 NIST 标准

论文明确指出，ML-KEM 与 ML-DSA 使用

$$
R_q = \mathbb{Z}_q[x]/(x^{256}+1),
$$

它来自 \(K_9 = \mathbb{Q}(\zeta_{512})\) 的整数环商。对应的 maximal real subfield 是 \(K_9^+\)，其次数为 \(128\)。因此主定理给出的 \(h_9^+ = 1\) 并不是一个遥远层数的纯理论结论，而是刚好落在现实标准正在使用的 degree-256 family 上。

这点很重要，因为它说明：

1. 论文没有只解决一个“将来也许有用”的数论层数。
2. 它覆盖的是 today 的标准参数族，而不是边缘实验参数。

### 10.2 Module Freeness：为什么 MLWE reduction 关心这个

论文 Section 4.2 直接把 MLWE 写成：

$$
(A, b = As + e \bmod q), \qquad
A \in R_q^{d \times d}, \quad s,e \in R_q^d.
$$

这套写法看起来像普通线性代数，但 reduction 真正在意的是：这些对象作为 \(\mathcal{O}_{K_k^+}\)-module 或其 mod-\(q\) quotient，到底是不是 free。

为什么这会变成 proof obligation？

- 如果 module 不自由，就未必存在你熟悉的“坐标化”。
- 许多采样、均匀性、basis 切换和 reduction map 会变得有额外扭曲。
- Langlois-Stehle 这类 worst-case to average-case reduction 需要 underlying module free。

论文给出的翻译是：当 \(h_k^+ = 1\) 时，所有 finitely generated torsion-free \(\mathcal{O}_{K_k^+}\)-modules 由 Steinitz theorem 都是 free，且这种良性结构会 descend 到 mod-\(q\) quotient。

#### 一个零跳步 notebook：为什么 `class number one` 会把 “有没有坐标基” 变成一个 class-group 问题

这一句如果只当成结论记，很容易变成又一个 “高级代数黑箱”。教材化地读，最好把它拆成五步。

**Step 1：先分清 vector space 直觉在哪一步失效**

在线性代数里，只要对象是 field 上的有限维向量空间，你就自动知道它有基，也就自动知道它同构于
\(F^r\)。

但 module 把标量从 field 换成了一般环
\(R\)。
一旦 \(R\) 不是 field，“有 rank” 不再自动推出“有全局基”。

最简单的对比就是：

1. 在 \(\mathbb{Z}\) 上，任何有限生成 torsion-free module 都是
   \(\mathbb{Z}^r\)；
2. 在一般 number ring 上，可能存在 rank \(1\)、torsion-free、但 **不是 free** 的 module。

这正是密码学读者最容易漏掉的地方：

```text
vector space intuition:
rank known
    ->
basis exists

module reality over number rings:
rank known
    -/-> basis exists automatically
```

所以 Section 4 里 “underlying module free” 不是修辞词，而是一个真正需要证明的结构条件。

**Step 2：为什么 nonprincipal ideal 本身就是一个 rank-one、torsion-free、但不 free 的 module**

把一个 ideal
\(I\subset \mathcal{O}_K\)
直接看成 \(\mathcal{O}_K\)-module。
它天然满足：

1. 它是 torsion-free，因为
   \(\mathcal{O}_K\)
   是整环；
2. 它的 rank 是 \(1\)，因为
   \(I\otimes_{\mathcal{O}_K}K\cong K\)；
3. 但若 \(I\) 不是 principal，它就不可能是 free of rank \(1\)。

最后一点可以一句话证明：
如果
\(I\cong \mathcal{O}_K\)
作为 module，那么
\(I\)
中某个元素的像必须生成整个
\(I\)，
于是
\(I=(\alpha)\)
就成了 principal ideal，矛盾。

所以：

> nonprincipal ideals 就是最具体的 “torsion-free 但不 free” 的 module 反例。

这也解释了为什么 class group 会突然进入 module freeness 的叙事。因为 class group 本来就在记录“ideal 离 principal 还差多远”。

**Step 3：Steinitz theorem 到底把什么压缩了**

对 Dedekind domain 上的 finitely generated torsion-free module
\(M\)，
Steinitz theorem 告诉你：

\[
M \cong \mathcal{O}_K^{\,r-1}\oplus I
\]

其中
\(r=\operatorname{rank}(M)\)，
\(I\)
是某个 fractional ideal。

这条分解最值得记住的，不是它的证明细节，而是它对“复杂度”的压缩方式：

1. 一个看起来可能很扭曲的高秩 module；
2. 最终只剩下一个 rank-one ideal \(I\) 来承载全部 global twisting；
3. 换句话说，**所有不自由的障碍，最后都浓缩进一个 ideal class**。

所以 module freeness 的问题，被压成了：

\[
I \text{ 是否 principal?}
\]

而这又正好是 class group 的问题。

**Step 4：为什么 \(h(K)=1\) 会直接杀掉这个障碍**

若
\(\mathrm{Cl}(K)=0\)，
也就是
\(h(K)=1\)，
那么每个 fractional ideal 都是 principal。
因此上式中的
\(I\)
必可写成
\(I=(\alpha)\cong \mathcal{O}_K\)。

于是

\[
M \cong \mathcal{O}_K^{\,r-1}\oplus I
\cong
\mathcal{O}_K^{\,r-1}\oplus \mathcal{O}_K
\cong
\mathcal{O}_K^r.
\]

这就是 Section 4 真正依赖的代数链：

```text
class number one
    ->
every rank-one ideal is principal
    ->
Steinitz obstruction disappears
    ->
every finitely generated torsion-free module is free
```

所以这里不是 “class number one 看起来让世界更整齐”；
而是它精确地消灭了 torsion-free module 没有全局基的唯一全局障碍。

**Step 5：为什么这件事会落到 MLWE reduction 的坐标语言上**

reduction 真正想用的是
\(R_q^d\)
这种有显式坐标的 free module 语言。
一旦底层 module 已经在整数环层面同构于
\(\mathcal{O}_K^d\)，
再 mod 掉
\(q\)
之后就得到

\[
M/qM \cong (\mathcal{O}_K/q\mathcal{O}_K)^d,
\]

这正是标准 MLWE 写法里
\(R_q^d\)
的形状。

这一步的教学重点不是“公式长得像”而是：

1. 坐标化不是免费送的；
2. 它依赖你先在整数环层面没有 hidden ideal-class twist；
3. class number one 正是在最底层保证了这件事。

因此 Section 4.2 的 claim 可以读成：

> 论文不是在发明一个新的 MLWE reduction；  
> 它是在清理这个 reduction 背后本来可能是条件性的 module-structure 假设。

所以 class number one 在这里的意义不是“看起来更整齐”，而是：

> reduction 想使用 free module language；  
> class number one 让这种语言在相关 cyclotomic real subfield 上无条件成立。

### 10.3 Principal Ideals 与 Codifferent：为什么 RLWE / PLWE 会变干净

RLWE 的证明语言不只涉及 \(R_q\) 本身，也涉及其 dual、codifferent、canonical embedding 下的误差分布、以及 ideal lattices 的几何。

#### 一个零跳步 notebook：为什么 codifferent 会控制 dual lattice，而 class number one 会把这层控制变成显式坐标

如果读者之前只接触过“多项式商环版”的 RLWE，很容易把这段话读成一句模糊口号：

> dual、codifferent、canonical embedding 这些词看起来很高级，但和样本 \((a,b=a\cdot s+e)\) 到底有什么关系？

最稳妥的读法，是把它拆成五步。

**Step 1：为什么 trace pairing 会自然地产生一个 dual lattice**

在 number field \(K\) 里，最自然的双线性 pairing 不是随便写出来的点积，而是

\[
\langle x,y\rangle = \operatorname{Tr}_{K/\mathbb{Q}}(xy).
\]

原因很简单：

1. trace 把 \(K\) 中的乘法信息压回到 \(\mathbb{Q}\)；
2. 经过 canonical embedding，它变成欧氏空间里真正可计算的内积数据；
3. 因而 ideal lattice 的对偶对象，会自动由这个 pairing 来定义。

于是对整数环 \(\mathcal{O}_K\)，最自然的 dual module 就不是“把矩阵转置一下”，而是

\[
\begin{aligned}
\mathcal{O}_K^\vee
&=
\{x\in K:\operatorname{Tr}_{K/\mathbb{Q}}(x\mathcal{O}_K)\subseteq \mathbb{Z}\},
\end{aligned}
\]

这就是 codifferent。

所以 codifferent 不是附属定义，而是：

> “哪些元素能和整个整数环做 trace pairing 且始终落回整数” 的全集。

**Step 2：为什么它正好是 canonical embedding 下的对偶格**

把
\(\mathcal{O}_K\)
送进 canonical embedding 后，你得到一个欧氏格
\(\sigma(\mathcal{O}_K)\)。
这个格的对偶格，按定义是所有和它做内积都得到整数的向量。

而上面的 codifferent 定义正是在 field language 中表达同一件事。
因此：

\[
\sigma(\mathcal{O}_K)^\ast
\cong
\sigma(\mathcal{O}_K^\vee).
\]

更一般地，对一个 fractional ideal \(I\)，它在理想格意义下的 dual 会是

\[
I^\vee = \mathcal{O}_K^\vee I^{-1}.
\]

这条公式非常关键，因为它告诉你：

1. dual 不只是“把 \(I\) 再看一遍”；
2. 它同时依赖两层信息：
   codifferent \(\mathcal{O}_K^\vee\) 和 ideal inverse \(I^{-1}\)；
3. 所以后面一旦 principal ideals 与 codifferent 都能显式写出来，dual lattice 也会随之显式化。

**Step 3：为什么 principal ideal 会把 dual 从“对象”压成“一个标量”**

若
\(I=(\alpha)\)
是 principal ideal，那么
\[
I^{-1}=(\alpha^{-1}).
\]

于是
\[
\begin{aligned}
I^\vee
&=\mathcal{O}_K^\vee I^{-1} \\
&=
\alpha^{-1}\mathcal{O}_K^\vee.
\end{aligned}
\]

如果 codifferent 本身也 principal，例如
\[
\mathcal{O}_K^\vee=(\delta^{-1}),
\]
那么立刻得到
\[
I^\vee=(\delta^{-1}\alpha^{-1}).
\]

这就是 Section 4 所说“表示更统一”的真正含义：

```text
without principality:
dual object may carry ideal-class bookkeeping

with principality:
dual object is generated by one explicit scalar
```

所以 class number one 之所以重要，不是因为它让 dual lattice 消失，而是因为它把 dual lattice 的全局扭曲压成一个可写下来的生成元。

**Step 4：为什么这会进入 RLWE 的样本与误差语言**

RLWE 并不只是 “在某个商环里加一个小误差”。
它自然活在 canonical embedding 与 trace pairing 控制的几何里：

1. secret / error 的“短”是 embedding 空间里的几何短；
2. dual 结构决定了哪些离散点集与 pairing / mod-\(q\) quotient 配合自然；
3. codifferent 因而会进入误差分布、对偶理想格以及 reduction map 的定义。

换句话说，RLWE 的几何母语更接近

```text
ideal lattice + canonical embedding + dual via trace
```

而不是一开始就只是“系数向量上的噪声”。

一旦 codifferent 与相关 ideals 都 principal，很多原本需要额外注明“在某个 ideal class 里选代表”“dual 侧还要再扭一次”的地方，就能统一写成显式标量乘法。

**Step 5：为什么这会让 RLWE / PLWE 的比较更干净，但又不能被过度解读**

PLWE 更接近 “固定一个多项式基下的系数分布”；
RLWE 更接近 “canonical embedding / ideal lattice / dual pairing 下的几何分布”。

要把两者比较干净，至少要处理两类差异：

1. **表示差异**：coefficient basis 与 canonical embedding 不是同一个视角；
2. **理想差异**：dual / codifferent / ideal-class twisting 会不会引入额外全局障碍。

class number one 主要清理的是第二类差异。也就是说：

- 它让 principal ideal、inverse ideal、codifferent 这些对象都更容易显式化；
- 它减少了 RLWE 侧 “dual object 落在某个非主类里” 这类条件性包袱；
- 因而 RLWE 与 PLWE 的桥少了一层 class-group 方向的技术噪声。

但这不应被过度读成：

> 只要 class number one，就自动得到 RLWE = PLWE。

还需要其他结构输入，例如具体的多项式表示、monogenicity、embedding distortion 控制等。
对本论文的语境，更准确的说法是：

> class number one 把 RLWE/PLWE 比较里一整层 ideal-class obstruction 无条件拿掉了。

当所有 ideals 都是 principal 时，会发生三件事：

1. codifferent 更容易写出显式生成元。
2. ideal lattices 的表示更统一，不必频繁处理“非主类”带来的额外 bookkeeping。
3. RLWE 与 PLWE 之间的桥更少受 class-group 侧的条件假设牵制。

这也是 Section 4.2 说 reduction 在 class number one 时更紧、更干净的原因。它不是说 “有了 \(h_k^+=1\) RLWE 才安全”，而是说：

- 以前某些结构性前提要么默认、要么条件性成立；
- 现在在 \(k \le 12\) 的范围里，这些前提变成了无条件事实。

#### 一个 proof-language notebook：同一个 RLWE 实例在四套语言里到底分别长什么样

很多读者读 Section 4.2 时真正卡住的，不是某个单独定义，而是下面这种反复切换：

- 一会儿写
  \((a,b=a\cdot s+e\bmod q)\)；
- 一会儿谈
  \(\mathcal{O}_K/q\mathcal{O}_K\)；
- 一会儿又跳到 canonical embedding；
- 再往后还会出现 dual ideal、codifferent、pairing compatibility。

如果没有一张总图，你很容易把这些对象误读成“作者用了四种差不多的说法在描述同一件事”。其实它们各自负责的不是同一层任务。

最稳妥的读法，是把同一个 RLWE 实例按四套语言拆开。

**Language 1：coefficient presentation**

这是实现层和 PLWE 叙事里最熟悉的写法：
\[
R_q=\mathbb{Z}_q[x]/(f(x)),
\qquad
(a,b=a\cdot s+e\bmod q).
\]

在本文真正相关的 family 里，
\[
f(x)=x^{256}+1,
\qquad
R_q=\mathbb{Z}_q[x]/(x^{256}+1).
\]

这套语言最适合说的是：

1. 元素如何写成系数向量；
2. 乘法如何实现；
3. NTT / 多项式模乘 / 压缩编码怎样落地；
4. PLWE 风格的 “coefficient noise” 直觉长什么样。

也就是说，coefficient presentation 主要回答的是：

> 我怎样把 ring element 写成机器最容易操作的坐标？

它的强项是表示与实现，而不是几何。

**Language 2：quotient-of-integers language**

一旦你回到 Section 4 的 proof 语境，作者更关心的是
\[
R_q \cong \mathcal{O}_K/q\mathcal{O}_K.
\]

对现实标准，\(K\) 可以取 \(K_9=\mathbb{Q}(\zeta_{512})\)，于是
\[
\mathcal{O}_{K_9}=\mathbb{Z}[\zeta_{512}],
\qquad
\mathcal{O}_{K_9}/q\mathcal{O}_{K_9}\cong \mathbb{Z}_q[x]/(x^{256}+1).
\]

这套语言最适合说的是：

1. 这个商环背后真正的整数环是谁；
2. ideals、fractional ideals、inverse ideals 在哪里生活；
3. module freeness、Steinitz obstruction、principality 这些结构条件到底是在谁身上说的；
4. class number one 为什么会对 reduction 的合法性产生影响。

也就是说，这一层主要回答的是：

> 这个看起来像“多项式商环”的对象，底层真正继承了哪一个 number-theoretic mother object 的结构？

它的强项是结构来源，而不是噪声几何。

**Language 3：canonical-embedding geometry**

一旦你开始问 “short 到底是什么意思”“误差为什么是高斯”“为什么 RLWE 不只是系数噪声”，语言就必须切到
\[
\sigma:\mathcal{O}_K \hookrightarrow K_\mathbb{R}
\]
这一边。

这时同一个 ring element 不再主要被看成系数向量，而被看成 embedding 空间里的一个格点。

这一层最适合说的是：

1. error / secret 的“短”到底在哪个欧氏空间里测；
2. Gaussian 与 smoothing parameter 的自然几何母语是什么；
3. RLWE 为什么先天更接近 ideal lattice geometry，而不只是 polynomial coefficients；
4. PLWE 与 RLWE 的差异为什么会表现成 basis distortion。

也就是说，这一层主要回答的是：

> 当 proof 说 “small error” 或 “short vector” 时，那个几何到底住在哪个空间里？

它的强项是噪声与长度的自然定义，而不是实现坐标。

**Language 4：dual ideal / codifferent / pairing language**

再往下一层，光有 embedding 还不够。你还必须说明：

1. quotient 和 pairing 为什么彼此兼容；
2. “mod \(q\)” 之后哪些 characters / torus-like quotient 是自然的；
3. 对偶对象应该相对于哪个 lattice 来定义。

这正是
\[
\begin{aligned}
\mathcal{O}_K^\vee
&=
\{x\in K:\operatorname{Tr}_{K/\mathbb{Q}}(x\mathcal{O}_K)\subseteq \mathbb{Z}\}
\end{aligned}
\]
和更一般的
\[
I^\vee=\mathcal{O}_K^\vee I^{-1}
\]
出场的地方。

这套语言最适合说的是：

1. trace pairing 下谁是自然的 dual lattice；
2. 为什么 RLWE reduction 喜欢把 quotient、pairing、discrete Gaussian 放在同一套对偶结构里；
3. codifferent / inverse ideal / principal generator 是否显式，为什么会影响 proof cleanliness。

也就是说，这一层主要回答的是：

> proof 里那些看似抽象的 dual / codifferent，到底是在保证哪一种“模掉以后仍然自然”的兼容性？

它的强项是对偶性与 reduction map 的自然性。

**把四套语言压成一张表**

| 语言 | 典型写法 | 最自然回答的问题 | 容易误读成什么 |
| --- | --- | --- | --- |
| coefficient presentation | \(R_q=\mathbb{Z}_q[x]/(x^{256}+1)\), coefficient vectors | 怎么表示、怎么实现、怎么算乘法 | “这已经是全部几何” |
| quotient-of-integers | \(\mathcal{O}_{K_9}/q\mathcal{O}_{K_9}\) | 底层整数环是谁，module / ideal 结构住在哪 | “只是 coefficient presentation 的文雅重写” |
| canonical embedding | \(\sigma(\mathcal{O}_K)\subset K_\mathbb{R}\) | short / Gaussian / smoothing 在哪定义 | “只是把系数向量换个基而已” |
| dual / codifferent | \(\mathcal{O}_K^\vee,\ I^\vee\) | pairing、quotient、对偶格怎样兼容 | “又多引入一个可有可无的理想” |

这张表最想纠正的误解是：

> Section 4.2 不是在同一层语言里反复改写同一个公式；  
> 它是在不同层次上分别处理表示、结构、几何与对偶性。

**现在把这四层重新贴回同一个样本**

假设你看到
\[
(a,b=a\cdot s+e \bmod q).
\]

最稳妥的阅读顺序不是一下子把所有层混在一起，而是按下面四问往下走：

1. **coefficient 问题**：这里的 \(a,s,e\) 在 power basis 下怎么写？实现如何操作它们？
2. **ring-origin 问题**：这些元素其实来自哪个 \(\mathcal{O}_K/q\mathcal{O}_K\)？
3. **geometry 问题**：proof 说 \(e\) 小，到底是 coefficient 小，还是 embedding 几何里小？
4. **duality 问题**：为什么 quotient、pairing 与 error distribution 能放在同一套 reduction 里自然兼容？

如果把这四问分开，你就不会再把 Section 4.2 读成一团术语雾。

**class number one 在这四层里只直接清理哪几层**

最后还要把“论文主结论到底作用在哪一层”说清楚。

class number one 最直接清理的是：

1. quotient-of-integers language 里的 ideal-class / module-structure 障碍；
2. dual / codifferent language 里的 principal / inverse / codifferent bookkeeping。

它并不直接替你清理的是：

1. coefficient presentation 与 canonical embedding 之间的 basis distortion；
2. 实现层坐标和几何层坐标之间的数值条件数问题；
3. PLWE 噪声与 RLWE 几何噪声之间仍然存在的表示差异。

所以 Section 4.2 最准确的一句总括是：

> class number one 先把“结构和对偶性这两层的障碍”无条件清掉；  
> 至于 coefficient view 和 embedding view 之间还差多少几何控制，那仍是另一层问题。

如果你把这页和前面的 `5.16`、`10.2`、`10.3` 连起来看，Section 4.2 的语言切换就会顺很多：

```text
coefficient presentation
    ->
see implementation coordinates

quotient of O_K
    ->
recover arithmetic structure

canonical embedding
    ->
define shortness / Gaussian geometry

dual ideal / codifferent
    ->
make pairing and reduction maps natural
```

这正是这篇论文里 “为什么一个 class-number 结果会跑进 RLWE/MLWE reduction 讨论” 的最底层答案。

#### 一个 reduction-reading notebook：如果按证明流程读 Section 4.2，freeness、codifferent 与 coefficient view 分别在哪一步出场

到这里，很多读者已经能分清四套语言是什么了，但还会剩下一个更实操的问题：

> 真正读 reduction 的时候，我该怎样判断  
> “这一句是在用 freeness”，  
> “这一句是在用 dual/codifferent”，  
> “这一句其实只是实现层的 coefficient 写法”？

最有效的方法，是把 Section 4.2 暂时读成一条 proof pipeline，而不是一段连续 prose。

这页 notebook 不重证 Langlois-Stehle 那条 reduction，只做一件事：

> 给你一张“阅读分层图”，让你知道每种对象到底在哪一类步骤里承担工作。

**Phase 0：先固定 reduction 想得到什么**

在教学上，先别急着钻进符号。
先固定 reduction 想说的是：

```text
structured worst-case lattice problem
    ->
average-case Module-LWE hardness
    ->
scheme-level confidence for module-over-R_q constructions
```

所以它不是在证明“某个实现代码安全”，也不是在证明“某个具体 NIST 参数一定有多少 bits”。
它做的是 hardness transfer。

因此只要某一步突然引入了新对象，你都应该先问：

> 这个对象是在帮助 hardness transfer 的哪一层成立？

**Phase 1：实例接口层，用 coefficient presentation 把问题写出来**

reduction 一开头最容易看到的，通常还是 familiar 的
\[
A\in R_q^{d\times d},\qquad s,e\in R_q^d,
\qquad b=As+e \bmod q.
\]

这一层为什么必须保留 coefficient presentation？

因为你总得先有一个 average-case instance interface，才能谈“给定 oracle / solver 怎么办”。

这一层最自然负责的是：

1. 把输入输出接口写成 \(R_q\)-module 形式；
2. 说明 average-case 问题长什么样；
3. 保持和现实 schemes 的写法一致；
4. 给后面所有 lifting 提供出发点。

所以如果你看到的是：

- \(A,s,e\) 的维数；
- mod-\(q\) 线性关系；
- 系数表示；
- matrix-over-\(R_q\) 的写法；

那么大概率你还停留在 **instance interface layer**，还没有真正触到 deepest arithmetic structure。

**Phase 2：结构抬升层，把“看起来像多项式商环”的对象重新抬回整数环**

下一步 reduction 必须回答：

> 这个 \(R_q\) 到底只是一个实现方便的商环，  
> 还是它背后真的有一个能承载理想、模、对偶与几何的 arithmetic parent？

这时语言会切到
\[
R_q \cong \mathcal{O}_K/q\mathcal{O}_K
\]
以及相关的 \(\mathcal{O}_K\)-module。

这一层最自然负责的是：

1. 解释问题为什么不是“任意商环上的任意噪声线性系统”；
2. 把 structure 拉回 number field / ring of integers；
3. 为后面谈 freeness、ideals、duals、embedding 准备合法母对象。

如果没有这一层，后面的 class number、principal ideal、codifferent 统统无处安放。

所以这一层的阅读关键词是：

```text
where does the module actually live?
```

**Phase 3：坐标合法化层，freeness 真正出场的地方**

当 reduction 想把某个底层 module 当成
\[
\mathcal{O}_K^r
\quad\text{或}\quad
(\mathcal{O}_K/q\mathcal{O}_K)^r
\]
来操作时，它就不再只是“想写得整齐”。

它是在要求：

1. 这个 module 真的有全局基；
2. 不存在 hidden ideal-class twist；
3. 你写下的每个坐标都不是局部拼出来的假坐标。

这正是 `10.2` 里讲的 freeness 问题。

所以如果你读到某一步在做：

- choose a basis；
- identify the module with a free rank-\(r\) object；
- descend that identification mod \(q\)；
- 把结构写成 \(R_q^d\) 而不是某个扭曲 module；

那这一步真正依赖的就是：

```text
torsion-free module
    +
class number one
    ->
free module coordinates become legitimate
```

也就是说，freeness 不是在解释噪声大小；
它是在解释：

> 为什么你有权把底层对象真正写成“一个标准坐标 module”。

**Phase 4：几何赋义层，canonical embedding 决定“短”和“高斯”在谁身上定义**

有了合法坐标，不代表你已经知道 error distribution 应该按什么几何来读。

reduction 一旦开始谈：

- short secret；
- discrete Gaussian；
- smoothing parameter；
- ideal lattice hardness；

语言就必须切到 canonical embedding。

这一层最自然负责的是：

1. 给 “small noise” 一个欧氏几何意义；
2. 把 worst-case lattice hardness 与 number-field objects 接上；
3. 说明 RLWE / MLWE proof 里为什么不是只看 coefficient size。

所以这一层不是在问“怎么编码”，而是在问：

> 哪个几何空间才是 hardness statement 真正发生的地方？

这也是为什么 `5.16` 里 coefficient basis 与 embedding geometry 的区别，不能被省略成一句“换个基而已”。

**Phase 5：对偶兼容层，dual ideal / codifferent 保证 quotient、pairing 与 Gaussian 的接口是自然的**

再往下走，reduction 还需要处理一个更细的问题：

> 你在 embedding 里定义的几何对象，  
> 和你在 mod-\(q\) quotient 里写下的代数对象，  
> 到底怎样彼此兼容？

这就是 dual ideal 与 codifferent 真正承重的地方。

它们负责的不是“再增加一些高级术语”，而是：

1. 指定 trace pairing 下自然的 dual lattice；
2. 说明某些 quotient / character / torus-like object 为什么定义得干净；
3. 确保离散高斯、pairing 与模 \(q\) 结构在同一条 proof 里不是互相打架的三套定义。

因此如果你读到某一步在说：

- dual module；
- inverse ideal；
- codifferent；
- pairing naturality；
- reduction map compatibility；

就应该立刻把它归类到 **duality layer**，而不是误以为作者只是在“换一种更抽象的记号”。

**Phase 6：再落回实例层，coefficient presentation 只是最后的可执行外壳**

当前面这些结构层都建立完后，proof 最终仍然会回到
\[
R_q^d
\]
或 matrix-over-\(R_q\) 的写法。

但这时你应该已经知道：

1. 这不是“什么都没发生，只是又写回多项式系数”；
2. 而是因为前面已经证明，这个 coefficient-level interface 背后确实有合法的 arithmetic / geometric support；
3. 所以 average-case instance 的书写，才不只是工程表示，而是有 reduction legitimacy 的表示。

换句话说，coefficient presentation 在 reduction 的末端重新出现时，扮演的是：

> executable shell

而不是：

> foundational justification

**把整条阅读流程压成一张操作表**

| 读到的步骤类型 | 你应优先问什么 | 真正承重的对象 |
| --- | --- | --- |
| \(A,s,e,b\) 的实例写法 | average-case 接口是什么 | coefficient presentation |
| \(R_q \cong \mathcal{O}_K/q\mathcal{O}_K\) | 底层母对象是谁 | quotient-of-integers structure |
| 选基、写成 rank-\(r\) free module | 这些坐标合法吗 | freeness / class number one |
| short / Gaussian / smoothing | 长度在哪个空间里定义 | canonical embedding geometry |
| dual / inverse / codifferent | quotient 与 pairing 怎样兼容 | duality layer |
| 再回到 matrix-over-\(R_q\) | 前面的抽象结构怎样落回可执行接口 | coefficient shell backed by proof |

这张表的用途很直接：

> 以后你读 Section 4.2 的任一段落，都可以先判定它属于哪一行，  
> 然后再问“这一步真正需要哪类结构假设”。

**class number one 只在这条 pipeline 的哪几段直接起作用**

把责任再压缩得更清楚一点：

class number one 直接加固的是：

1. `Phase 3` 的 freeness legitimacy；
2. `Phase 5` 的 principal / inverse / codifferent explicitness。

它不会直接替你完成的是：

1. `Phase 1` 的实例接口定义；
2. `Phase 4` 的全部 embedding 几何控制；
3. coefficient presentation 与 embedding geometry 之间的全部 distortion analysis。

所以如果你在 Section 4.2 里看到一句“因此在 \(h_k^+=1\) 时 reduction 更干净”，最稳妥的自动翻译不是：

> 所有层次都 simultaneously 变简单了。

而是：

> 结构坐标是否合法、对偶对象是否显式，这两层最直接受益；  
> 其他几何或表示问题仍要单独处理。

这页 notebook 的目的，就是把那句看似抽象的话，压回一个能实际跟着 proof 走的阅读手柄。

#### 一个 margin-annotation notebook：拿到 Section 4.2 原文段落时，应该怎样在页边标“这里到底在用什么”

如果你真的准备回去逐段读 Section 4.2，最容易再次迷路的情况是：

> 我明明知道 freeness、codifferent、embedding、coefficient presentation 各自是什么，  
> 但读到具体一段 prose 时，还是不知道这一段到底在证明链里负责哪一层。

这时最有用的不是再背定义，而是给自己一个固定的页边标注协议。

下面这套协议故意不依赖某个 theorem 编号。
它假设你拿到的是一段 reduction prose，然后逐段问同一组问题。

**Tag A：instance-writing**

如果一段话主要在做下面这些事：

- 写出 \(A,s,e,b\) 的形状；
- 说明 oracle 的输入输出；
- 指定 \(R_q^d\)、\(R_q^{d\times d}\) 这类接口；
- 维持和现实 scheme 写法一致；

就在页边标：

```text
[instance-writing]
this paragraph fixes the average-case interface
```

这类段落的关键不是深层数论，而是：

> proof 先要有一个能交给 solver 的实例接口。

如果把这类段落删掉，后面连“average-case problem 到底是什么”都无从谈起。

**Tag B：arithmetic-lift**

如果一段话在强调：

- \(R_q\) 不是 arbitrary ring；
- 它来自 \(\mathcal{O}_K/q\mathcal{O}_K\)；
- 某些 module / ideal / quotient 其实 living over \(\mathcal{O}_K\)；
- reduction 必须回到底层整数环；

就在页边标：

```text
[arithmetic-lift]
move from implementation shell back to the arithmetic parent
```

这类段落的关键不是噪声，而是：

> 你要先证明自己站在一个真正能谈 ideals、duals、embedding 的母对象上。

如果把这类段落删掉，后面的 class number、principal ideal、codifferent 都会变成无来源的术语漂浮。

**Tag C：freeness-legitimacy**

如果一段话在做：

- choose a basis；
- identify a torsion-free module with a free one；
- descend that identification mod \(q\)；
- justify coordinates over \(R_q^d\)；

就在页边标：

```text
[freeness-legitimacy]
this is where coordinates stop being merely notational
```

这类段落真正承重的是：

> 为什么这些坐标真的是全局合法坐标，而不是藏着 ideal-class twist 的假坐标。

如果把这类段落删掉，proof 仍可能写得像在线性代数里一样顺，但你已经失去了“为什么有权这么写”的关键合法性。

**Tag D：geometry-lift**

如果一段话开始谈：

- discrete Gaussian；
- short vectors；
- smoothing parameter；
- ideal-lattice geometry；
- worst-case hardness living in Euclidean space；

就在页边标：

```text
[geometry-lift]
coefficient symbols are now being re-read in canonical-embedding geometry
```

这类段落的关键不是代数坐标，而是：

> 哪个空间才是“短”和“高斯”真正发生的地方。

如果把这类段落误读成纯 coefficient 叙事，你就会低估 RLWE / MLWE reduction 里 embedding geometry 的分量。

**Tag E：dual-compatibility**

如果一段话在处理：

- dual module / dual ideal；
- codifferent；
- inverse ideal；
- trace pairing；
- quotient 与 pairing 的自然兼容；

就在页边标：

```text
[dual-compatibility]
this is where the reduction aligns quotient structure with geometric duality
```

这类段落的关键不是“再加一点抽象层次”，而是：

> 让 mod-\(q\) 代数、embedding 几何、以及 Gaussian / pairing 的接口真正拼起来。

如果把这类段落删掉，proof 里的 quotient map 与 dual lattice 关系就会开始像“定义很多但不知为何兼容”的黑箱。

**Tag F：shell-return**

如果一段话最后又回到：

- matrix-over-\(R_q\)；
- coefficient-level samples；
- solver interface；
- scheme-like notation；

就在页边标：

```text
[shell-return]
abstract structure is being pushed back into an executable average-case shell
```

这类段落最容易被误会成“作者又回到了开头，所以中间那些结构只是装饰”。
其实恰好相反：

> 正是因为中间那些结构层已经被建立，  
> 现在写回 coefficient shell 才是合法的。

**把六种标记压成一张页边协议表**

| 页边标记 | 你应立刻问的问题 | 如果这层缺失，最先坏掉什么 |
| --- | --- | --- |
| instance-writing | average-case 接口到底怎么写 | 连 solver 输入输出都不明确 |
| arithmetic-lift | 这些对象真正住在哪个 \(\mathcal{O}_K\) 上 | ideals / class number / codifferent 无处着陆 |
| freeness-legitimacy | 这些坐标为什么是全局合法的 | module 可能只是看起来像 \(R_q^d\) |
| geometry-lift | “小噪声”到底在哪个空间里小 | shortness / Gaussian 失去自然定义 |
| dual-compatibility | quotient 与 pairing 为什么能一起工作 | dual lattice 与 mod-\(q\) 结构脱节 |
| shell-return | 前面的抽象层怎样落回实例接口 | proof 无法回到真正的 average-case problem |

这张表其实就是一把速查钥匙。
以后你每读一段 Section 4.2，都可以先问：

1. 这一段最像哪一个 tag？
2. 它是在建立接口、合法性、几何，还是对偶兼容？
3. 如果没有这一层，proof 最先会在哪里失真？

**最后再把 class number one 放回这套批注协议里**

在这六个 tags 里，最直接受 \(h_k^+=1\) 影响的是：

1. `freeness-legitimacy`；
2. `dual-compatibility`。

原因已经在前文反复出现，但这里值得再压成一句话：

> class number one 不是在替你定义 average-case instance；  
> 它是在保证你写下来的 free coordinates 和 dual objects 真的是无条件合法且显式的。

如果你用这套页边协议回去读 Section 4.2，那么很多原本看起来只是“高级名词切换”的段落，会立刻变成一张职责很清楚的证明流程图。

#### 一个 synthetic annotated walkthrough：把六种标签真的贴到一条 reduction skeleton 上

上面的页边协议已经告诉你“看到什么句型就该打什么标签”。
但很多读者第一次真正动手时，还是会卡在一个很现实的问题：

> 我知道这些 tags 的定义，  
> 但要我自己在纸上标第一轮，仍然不知道一段典型 prose 到底长什么样。

这时最好的练习不是立刻硬啃原论文全文，而是先看一条 **synthetic but faithful** 的 skeleton。

注意下面六段不是在冒充原论文原句。
它们只是把一条典型 structured-lattice reduction 会经历的 prose 流程，压成最短可批注版本。

**Paragraph 1**

> Let \(A\in R_q^{d\times d}\), \(s,e\in R_q^d\), and suppose we are given oracle access to an algorithm that solves the corresponding average-case Module-LWE problem on samples \((A,As+e\bmod q)\).

页边标签应该打：

```text
[instance-writing]
```

因为这一段只是在固定：

1. average-case problem 的输入输出；
2. solver/oracle 的接口；
3. 实例写法仍然工作在 \(R_q^d\) 的 coefficient shell 上。

它还没有开始解释这些对象的 arithmetic source，也还没谈几何。

**Paragraph 2**

> We now regard \(R_q\) as a quotient \(\mathcal{O}_K/q\mathcal{O}_K\), so that the module instances inherit the arithmetic structure of the ambient ring of integers.

页边标签应该打：

```text
[arithmetic-lift]
```

因为这一段真正干的是：

1. 把看起来像实现接口的对象抬回 \(\mathcal{O}_K\)；
2. 给 ideals、fractional ideals、class number、duality 等后续对象腾出合法落脚点。

如果这一段没有出现，后面任何关于 class number one 的句子都会像突然插进来的外来设定。

**Paragraph 3**

> Under the relevant class-number-one hypothesis, the torsion-free modules that arise here may be identified with free modules of the same rank, so the coordinate description over \(R_q^d\) is a genuine global description rather than a merely local convenience.

页边标签应该打：

```text
[freeness-legitimacy]
```

因为这一段是在回答：

> 为什么我有权把这些对象真正写成标准坐标 module？

它不是在重新定义实例，也不是在谈 Gaussian。
它是在消灭 hidden ideal-class twisting 这层结构障碍。

**Paragraph 4**

> To measure shortness and to formulate the relevant Gaussian distributions, we pass to the canonical embedding, where the structured module is read as a lattice in Euclidean space.

页边标签应该打：

```text
[geometry-lift]
```

因为这一段的职责是把：

- coefficient symbols

重新解释成

- embedding geometry 里的长度与分布。

这就是 proof 真正开始接触 “worst-case lattice hardness lives in Euclidean geometry” 的位置。

**Paragraph 5**

> The dual objects are then described via the trace pairing, with the codifferent and inverse ideals ensuring that the quotient structure and the geometric duality remain compatible throughout the reduction.

页边标签应该打：

```text
[dual-compatibility]
```

因为这一段已经不只是谈“格在哪里”，而是在谈：

1. dual lattice 如何定义；
2. pairing 为什么自然；
3. quotient / Gaussian / duality 为什么能放在同一条 proof 里不互相打架。

这正是 codifferent 真正承重的地方。

**Paragraph 6**

> Finally, the transformed problem is pushed back to an average-case interface written over \(R_q^d\), so that the solver hypothesis can be invoked on an executable instance rather than on an abstract lattice object.

页边标签应该打：

```text
[shell-return]
```

因为这一段的工作是：

1. 把前面建立的 arithmetic / geometric structure 再压回可执行接口；
2. 让 oracle hypothesis 真正能被调用。

这一段最容易被读者误会成“作者绕了一大圈又回到了原点”。
其实并不是回到原点，而是：

> 先在抽象层证明这套接口有合法支撑，  
> 再把它还原成 solver 真能接收的实例壳。

**把六段 skeleton 压成一张一眼能用的表**

| 段落编号 | 典型句意 | 应贴标签 | 它真正在做什么 |
| --- | --- | --- | --- |
| 1 | 写出 \((A,As+e\bmod q)\) 与 solver interface | instance-writing | 固定 average-case problem |
| 2 | 把 \(R_q\) 重新读成 \(\mathcal{O}_K/q\mathcal{O}_K\) | arithmetic-lift | 恢复底层 arithmetic parent |
| 3 | 用 class number one / freeness 合法化坐标 | freeness-legitimacy | 证明这些坐标不是假的 |
| 4 | 用 canonical embedding 定义短与高斯 | geometry-lift | 给 hardness 与噪声一个欧氏几何舞台 |
| 5 | 用 codifferent / inverse / trace 处理 duality | dual-compatibility | 对齐 quotient、pairing 与 dual lattice |
| 6 | 把抽象结构压回 \(R_q^d\) 接口 | shell-return | 让 solver hypothesis 真正可调用 |

这张表最值得带走的一点是：

> 一条 reduction prose 从来不只是“把问题越写越复杂”；  
> 它是在不同层之间来回搬运同一个实例，直到既保住结构，又保住可执行接口。

**怎样把这页练习真正用到原文上**

如果你下一步真的回到 Section 4.2 原文，我建议这样做：

1. 先不急着读每个数学细节；
2. 只在每个自然段旁边写一个 tag；
3. 再问这一段是在修“接口、结构、几何、还是对偶兼容”；
4. 最后再看相邻两段之间为什么要切层。

只要这样做两三页，Section 4.2 原本那种“好像什么都懂一点，但不知道每段在干嘛”的感觉会明显减轻。

#### 一个 reduction worksheet：真正回到原文时，每段先勾哪几个框

如果前面的 synthetic walkthrough 已经让你知道“标签怎么打”，下一步最容易遇到的新问题就是：

> 真回到原文时，我是不是每段都要同时想六七件事？  
> 如果我基础还不稳，第一遍到底先看什么，先不看什么？

这时最省力的办法不是硬记，而是把阅读动作本身模板化。

下面这张 worksheet 可以直接拿来用。第一遍读 Section 4.2 时，你不需要每格都写长答案；很多时候只要打勾、写两个关键词就够了。

| 读到的段落特征 | 先勾这个 tag | 第一遍必须确认什么 | 第一遍可以暂缓什么 | 最直接回链到本讲义哪一节 |
| --- | --- | --- | --- | --- |
| 写 \(A,s,e,b\) 与 solver/oracle 接口 | instance-writing | average-case 问题到底怎样表述 | 具体几何还不用急着想 | 10.2 开头、10.3 前的接口图 |
| 把 \(R_q\) 重新读成 \(\mathcal{O}_K/q\mathcal{O}_K\) | arithmetic-lift | 底层整数环是谁，为什么不是 arbitrary ring | 先不急着追每个 ideal 细节 | 5.8、10.1、5.16 bridge notebook |
| 说明 underlying module free / choose a basis | freeness-legitimacy | 坐标是否合法，是否在消 hidden twist | 先不急着展开全部 Steinitz 证明 | 10.2 的五步 notebook |
| 谈 shortness / Gaussian / smoothing | geometry-lift | “小”到底在哪个空间里定义 | 先不急着算具体参数 | 5.13、5.16、10.3 |
| 谈 dual / inverse / codifferent / trace pairing | dual-compatibility | quotient 与 pairing 怎么兼容 | 先不急着追所有显式生成元 | 5.13、10.3 的 dual-lattice notebook |
| 又回到 \(R_q^d\) / coefficient shell | shell-return | 这一段是在把结构压回可执行接口 | 不要误以为前面抽象层白做了 | 10.3 proof-language notebook、synthetic walkthrough |

这张表的重点不是“替你读完原文”，而是帮你管理注意力。

第一遍时，建议每段只问四件事：

1. 这段最像哪一个 tag？
2. 这段是在建接口、建合法性、建几何，还是建对偶兼容？
3. 这段如果删掉，proof 最先会坏在哪里？
4. 这段和讲义里哪一页 notebook 最直接对应？

只要先把这四问稳定住，你就不会在第一轮阅读里同时被：

- theorem statement；
- module 结构；
- embedding geometry；
- codifferent 记号；
- coefficient implementation

五件事一起拖走。

**再补一列：哪几段最直接吃到 \(h_k^+=1\)**

如果你要更有针对性地看 “Weber conjecture 的结果到底在哪些 reduction 句子里起作用”，可以再加一列二值判断：

| tag | 是否最直接吃到 \(h_k^+=1\) | 理由 |
| --- | --- | --- |
| instance-writing | 否 | 这是 average-case 问题的接口层，不是 class-number 结论作用点 |
| arithmetic-lift | 间接 | 只是在把对象抬回 arithmetic parent，尚未使用 principality/freeness |
| freeness-legitimacy | 是 | 这里最直接用到 class number one 消掉 module twisting |
| geometry-lift | 否 | embedding geometry 仍需额外控制，不会被 \(h_k^+=1\) 自动解决 |
| dual-compatibility | 是 | principal / inverse / codifferent 的显式性在这里最直接受益 |
| shell-return | 间接 | 这是把前面的结构收益压回接口，而不是收益本身产生的地方 |

这张二值表非常适合防止一个常见误读：

> 只要 Section 4.2 哪儿提到了 class number one，  
> 那一整页的所有技术层都 simultaneously 被它解决了。

正确的读法更细：

> \(h_k^+=1\) 最直接解决的是坐标合法性与对偶显式性；  
> 而不是替你自动解决全部 embedding 几何或 coefficient distortion。

**如果你只允许自己在原文页边写三个符号**

对第一次回看原文的读者，我建议你把批注压缩到最小工作集：

1. 写一个主 tag；
2. 写一个 “this paragraph is for ...” 的英文短语；
3. 若这一段直接受 \(h_k^+=1\) 影响，就额外打一个 `H=1` 标记。

例如：

```text
[freeness-legitimacy]
global coordinates become legal
H=1
```

或者：

```text
[geometry-lift]
shortness moves to embedding space
```

这套极简格式的价值在于：它足够轻，不会打断阅读；但又足够强，能强迫你分清“这段究竟在干嘛”。

如果你能把这张 worksheet 用熟，那么下一步再进入真正更细的 reduction-by-reduction 注释页时，就不会只是在原文旁边堆满术语，而是真的在读证明的职责分层。

#### 一个 reduction-obligation ledger：第二遍读 Section 4.2 时，每类段落到底背着什么数学义务

上面的 `worksheet` 已经解决了第一遍阅读最现实的问题：

> 这段 prose 最像哪个 tag？

但如果你真的想从“看懂段落在干嘛”走到“能判断这一步到底有没有证明到位”，还需要再压一层。

第二遍读的时候，真正该问的已经不是：

> 这一段是在谈 freeness 还是 codifferent？

而是：

> 这一段如果要在 proof 里站住脚，  
> 它到底必须补出哪一种数学义务？  
> 如果这层义务没补上，reduction 会先断在哪里？

这就是下面这张 `obligation ledger` 的用途。

**先给一张六类段落的证明义务总表**

| tag | 这一类段落真正必须证明什么 | 如果这层站不住，proof 最先坏在哪里 | \(h_k^+=1\) 在这里能直接帮你免掉什么 |
| --- | --- | --- | --- |
| instance-writing | 你写下的 average-case instance 真的是后续 oracle / solver 要接收的对象 | reduction 最后根本没有可调用的目标问题 | 基本不直接帮忙；这层首先是接口定义问题 |
| arithmetic-lift | coefficient shell 背后确实有合法的 \(\mathcal{O}_K/q\mathcal{O}_K\) arithmetic parent | ideals、modules、class number、codifferent 全部失去落脚点 | 只间接受益；\(h_k^+=1\) 还没开始真正出场 |
| freeness-legitimacy | 这些坐标真的是全局合法坐标，而不是藏着 ideal-class twist 的局部写法 | 你写成 \(R_q^d\) 的对象可能根本不是自由模意义下的标准实例 | 直接免掉“还要另外验证 module 是否 free / 是否被某个 ideal class 扭曲” |
| geometry-lift | “小”“高斯”“worst-case lattice” 确实是在 canonical embedding 的欧氏几何里定义 | proof 会退化成纯 coefficient manipulation，失去真实 hardness geometry | 不直接帮你完成；几何控制与 distortion 仍要单独处理 |
| dual-compatibility | quotient、trace pairing、dual lattice、codifferent 在同一套对象上自然兼容 | reduction map、duality、Gaussian language 会彼此脱节 | 直接免掉“inverse ideal / codifferent 是否显式、是否落在非主类里”的一层负担 |
| shell-return | 前面的 arithmetic/geometric 变换最后真能压回 solver 可执行的 \(R_q^d\) 接口 | 你得到的只是抽象结构结论，不是可喂给 oracle 的 average-case instance | 间接受益；收益来自前面合法性与对偶性已经被清理 |

这张表最重要的一列其实不是“这一段在说什么”，而是：

> 如果这层没被证明，最先坏掉的是什么？

因为这能迫使你把 prose 当成 proof obligation，而不是当成背景说明。

**Obligation A：instance-writing 段落不是在热身，而是在固定 reduction target**

很多读者会下意识觉得：

> 写 \(A,s,e,b\) 这些对象的段落，只是在把记号铺开。

但第二遍读时，你必须更严格：

1. 这是不是 reduction 最后真正要调用的 average-case oracle interface？
2. 这里的分布、维度、模数、module 结构，是否和后文 solver 假设一致？
3. 后面所有 arithmetic lift 与 geometry lift，是否最终都要回到这张接口壳？

如果这层没站住，后面再漂亮的 number-theoretic 讨论都可能只是在分析一个 **并不是最终目标实例** 的对象。

**Obligation B：arithmetic-lift 段落必须证明“这不是任意商环上的巧合”**

第二遍读到
\[
R_q \cong \mathcal{O}_K/q\mathcal{O}_K
\]
时，不能只满足于“原来它们同构”。
你应该继续追问：

1. 这个同构是不是在证明里真正被用来搬运 ideals、modules、duals？
2. 后面谈到的 \(\mathcal{O}_K\)-module、fractional ideal、codifferent，是否都通过这一步得到合法母对象？
3. 若把这一步删掉，后面的 class-number 语言是否会失去宿主？

所以这类段落背着的义务是：

> 不只是给一个优雅改写，  
> 而是给后面所有 arithmetic structure 一个合法出生地。

**Obligation C：freeness-legitimacy 段落必须把“能写坐标”升级成“有权写这种坐标”**

这是 Section 4.2 最容易被读快的一层。
表面上它常常只是在说：

1. choose a basis；
2. identify the module with a free one；
3. descend mod \(q\)；
4. write everything in coordinates。

但第二遍读时，你真正要核对的是：

> 这里是不是已经消掉了 hidden ideal-class twist？

也就是说，这类段落背着的义务不是“坐标写出来了”这么弱，而是：

\[
\text{torsion-free arithmetic module}
\Longrightarrow
\text{genuinely free global coordinates}
\]

在本文语境里，\(h_k^+=1\) 的最直接收益就落在这里：

1. 你不必再额外追某个秩一扭曲模是不是来自非平凡 ideal class；
2. 也不必在后面一直背着 “coordinates are only locally chosen” 这种隐性包袱。

**Obligation D：geometry-lift 段落必须把 hardness statement 接回真实欧氏空间**

一旦 prose 开始谈 shortness、Gaussian、smoothing，第二遍阅读就不能只问“作者是不是切到 embedding 了”。
你还要问：

1. 这里的“短”是否已经从 coefficient size 切换成 embedding norm？
2. reduction 需要的 worst-case lattice problem 是否确实 living in this Euclidean model？
3. 这一层是否只是在讲 intuition，还是已经在建立后面分布与长度比较所必需的几何语义？

如果这层没被真正补上，proof 会退化成：

> 用 coefficient shell 写着一个格密码证明，  
> 但真正的 hardness geometry 从未被接回实例。

这也是为什么 `geometry-lift` 虽然不直接被 \(h_k^+=1\) 解决，却仍然是 Section 4.2 里不可省的一层。

**Obligation E：dual-compatibility 段落必须证明“这些自然对象真在同一套范畴里兼容”**

很多读者第二遍最容易在这里失去耐心，因为 codifferent、inverse ideal、trace pairing 看起来像在同时引入三层抽象。

但真正该问的是：

1. quotient structure 与 dual lattice 是否通过同一个 pairing 接在一起？
2. 这里的 dual object 是不是 reduction 后面真正要用的 natural dual，而不是随便挑的一种对偶？
3. 若没有这层，Gaussian language、mod-\(q\) quotient、以及 arithmetic duality 会不会分裂成互不相认的三套对象？

因此这类段落背着的义务是：

> 证明 quotient、pairing 与 dual lattice 不是偶然同名，  
> 而是在同一条 reduction 里 genuinely compatible。

在本文语境下，\(h_k^+=1\) 在这里最直接帮你免掉的是：

1. inverse ideals 还要不要再追非主类代表；
2. codifferent 是否只能抽象存在、却难以显式搬运；
3. 对偶对象是不是一直背着额外的 ideal-class bookkeeping。

**Obligation F：shell-return 段落必须把“抽象合法性”重新压回“可执行实例”**

最后又写回 \(R_q^d\) 或 matrix-over-\(R_q\) 时，第二遍阅读最容易偷懒地想：

> 哦，作者只是又换回工程记号了。

但真正要核对的是：

1. 前面建立的 free coordinates、embedding geometry、dual compatibility，是不是都已经被安全地压回这个 shell？
2. 这个 shell-return 后的实例，是否仍然具有 solver 假设所要求的分布与结构？
3. 如果这里说不清，前面那些结构收获是不是就停留在抽象层，无法真正调用 oracle？

所以这一层的证明义务是：

> 把抽象结构收益重新翻译成 solver 真能吃的接口，  
> 而不是只留下一个 “存在某种同构 / 某种格结构” 的软结论。

**把第二遍阅读动作压成一个三问协议**

如果你真的准备回到 Section 4.2 原文做第二遍，我建议每一段都固定问这三句：

1. 这段的主 tag 是什么？
2. 这段背着的 **数学义务** 是什么，而不只是它表面在说什么？
3. 如果这段失败，proof 最先坏掉的是接口、坐标合法性、几何语义，还是对偶兼容？

你会发现，第二遍和第一遍最大的区别不是“看得更细”，而是：

> 第一遍在识别层次；  
> 第二遍在审计每一层到底有没有把自己的 burden 真背起来。

**最后把这一页压成一句最该带走的话**

Section 4.2 最容易被误读成“一连串高层术语切换”。
第二遍阅读时，最稳妥的矫正句应该是：

> 每一段 reduction prose 都不只是“在换语言”；  
> 它都在承担一个具体的 proof obligation。  
> \(h_k^+=1\) 最直接替你清掉的，是其中关于 free coordinates 与 dual explicitness 的那两层；  
> 其余层次仍然需要逐段核对。

#### 一个 reduction-invariant notebook：Section 4.2 从头到尾真正不能丢的是哪几类 invariant

到这里，第一遍阅读的 `tag`，第二遍阅读的 `proof obligation`，都已经有了。
但很多读者回到原文时还会继续卡在一个更“全局”的问题：

> 我知道每一段局部在干什么，  
> 但整条 reduction 从头走到尾，到底有哪些东西是 **一步都不能丢** 的？

这是因为 reduction proof 不是单纯把对象从一种记号改写成另一种记号。
它更像是在一路搬运若干条必须同时保持的 invariant。

只要其中任何一条在中途断掉，最后就算还写得出一个类似
\[
(A,b=As+e\bmod q)
\]
的壳，也不再是原来那个能调用 solver 的 average-case problem 了。

**先给一张 invariant map**

| invariant | 它真正要求什么 | 若中途丢失，最后会坏成什么 | 在现有六个 tags 里主要由谁承重 |
| --- | --- | --- | --- |
| oracle-family invariant | 最后送进 solver 的仍然是它承诺处理的 average-case MLWE family | 你只能得到“像某个实例”的对象，却不能合法调用 oracle | instance-writing、shell-return |
| arithmetic-legitimacy invariant | objects genuinely live over the intended \(\mathcal{O}_K\) / quotient / module parent | ideals、class number、codifferent 语言全部变成空转 | arithmetic-lift |
| coordinate-legitimacy invariant | 写成 \(R_q^d\) 的坐标真是全局合法坐标，不藏 ideal-class twist | 你在用一个看似线性的 shell 讨论一个其实被扭曲的 module | freeness-legitimacy |
| geometry-of-shortness invariant | “短”“高斯”“worst-case lattice” 一直在同一套 canonical-embedding 几何里解释 | proof 会退化成只剩 coefficient manipulations，没有真实几何难题 | geometry-lift |
| duality-compatibility invariant | quotient、trace pairing、dual lattice、codifferent 始终彼此兼容 | mod-\(q\) 代数、Gaussian 语言、对偶结构各说各话 | dual-compatibility |
| distribution-fidelity invariant | 经过 lifting / pushforward / shell-return 后，样本分布仍在 solver 可接受的范围里 | 你得到的是“同类型公式”，但不是同类 average-case law | instance-writing、geometry-lift、shell-return |

这张表和前面的 `obligation ledger` 的区别在于：

> `obligation ledger` 盯的是“这一个段落要证明什么”；  
> `invariant map` 盯的是“整条 reduction 从头到尾不能丢什么”。

**Invariant A：oracle-family invariant**

很多初读者会觉得，只要最后还能写成
\[
(A,b=As+e\bmod q)
\]
就说明 reduction 成功回到了平均情形。

这其实不够。
真正要保住的是：

1. solver 预设接受的实例 family 没被偷偷换掉；
2. 维度、模数、底层环/模接口仍在承诺范围内；
3. 最后一跳调用 oracle 时，送进去的确实是“同一个问题族”的样本。

所以 `instance-writing` 和 `shell-return` 真正共同维护的是：

> not just a familiar syntax, but the actual oracle-facing family.

**Invariant B：arithmetic-legitimacy invariant**

这一条 invariant 解释了为什么 reduction 不能只停留在 coefficient shell 上。

当 proof 说
\[
R_q \cong \mathcal{O}_K/q\mathcal{O}_K
\]
时，真正要保的不是“记号更优雅”，而是：

1. 后面出现的 ideals、fractional ideals、class number 都有合法宿主；
2. dual/codifferent 不是悬空定义；
3. 你讨论的 module 确实来自 number-field arithmetic，而不是 arbitrary ring gadget。

如果这条 invariant 丢了，Section 4.2 后面所有数论句子都会变成：

> 看起来像在解释底层结构，  
> 但实际上没有证明这些结构真的属于当前实例。

**Invariant C：coordinate-legitimacy invariant**

这条 invariant 是 `freeness-legitimacy` 那一页真正的全局版本。

写成 \(R_q^d\) 很容易；
真正困难的是证明：

\[
\text{the coordinates are globally legal, not merely locally convenient.}
\]

也就是说，reduction 必须一路保住：

1. 这些坐标不是临时挑出来的局部 chart；
2. 不存在 hidden ideal-class twisting 在后台改写对象类型；
3. 后面所有 matrix-over-\(R_q\) 的线性代数，都是在 genuinely free module 上做的。

在本文语境里，\(h_k^+=1\) 的最大直接收益之一，就落在这条 invariant 上：

> 它把“我是不是还欠一个 freeness proof”这层长期包袱，无条件压掉了。

**Invariant D：geometry-of-shortness invariant**

Section 4.2 一旦谈到 shortness、Gaussian、smoothing、worst-case lattice hardness，reduction 就必须继续保住：

1. “小”的含义没有偷偷从 embedding norm 滑回 coefficient size；
2. 难题真正 living in the Euclidean lattice geometry that the reduction needs；
3. 后面的分布语言不是 detached intuition，而是和 hardness statement 共用同一套几何。

这也是为什么很多看似“只是解释直觉”的几何段落，其实在全局上是在保一个非常硬的 invariant：

> the semantic meaning of shortness.

如果这条 invariant 断掉，你看到的将不再是 RLWE/MLWE proof，而更像是：

> 把格密码的字眼贴在一个 coefficient-level 代数变形上。

**Invariant E：duality-compatibility invariant**

dual/codifferent 段落最容易被误解成“技术记号偏好”。
从 invariant 视角看，它们真正保的是：

1. mod-\(q\) quotient 结构；
2. trace pairing；
3. natural dual lattice；
4. Gaussian / error language

这四样东西始终属于同一套兼容框架。

也就是说，这条 invariant 不允许 proof 变成：

```text
algebra over here
geometry over there
duality somewhere else
```

然后最后再口头说“它们应该是相容的”。

在本文里，\(h_k^+=1\) 对这条 invariant 的直接帮助同样非常明确：

1. inverse ideal 更容易显式；
2. codifferent 的 bookkeeping 更少；
3. dual objects 不再长期拖着非主类方向的附加包袱。

**Invariant F：distribution-fidelity invariant**

这一条其实最容易被忽略，因为原文常常不会每段都大声提醒“这里在保分布”。

但 worst-case to average-case reduction 真正想成立，不能只保对象类型，还要保：

> the transformed samples still live in the right law class for the solver hypothesis.

也就是说，第二遍读时你应始终追问：

1. 这个 map 只是把对象写成另一种壳，还是也把分布安全地 push forward 了？
2. 这里的 Gaussian / noise language 是否仍和 solver 假设所需的 family 对齐？
3. `shell-return` 之后的样本，究竟只是“外形相似”，还是仍是 oracle promise 能吃的 average-case law？

这条 invariant 之所以要单独列出来，是因为它和 `oracle-family invariant` 不完全相同：

- `oracle-family` 更像“问题类型对不对”；
- `distribution-fidelity` 更像“样本 law 对不对”。

两者常常一起出现，但不能混成一件事。

**把六个 tags 重新投影到这几条 invariant 上**

前面的 tags 现在可以重新读成一张更全局的投影表：

| tag | 它主要在保哪条 invariant | 次要碰到哪条 invariant |
| --- | --- | --- |
| instance-writing | oracle-family | distribution-fidelity |
| arithmetic-lift | arithmetic-legitimacy | coordinate-legitimacy 的前置条件 |
| freeness-legitimacy | coordinate-legitimacy | shell-return 的合法前提 |
| geometry-lift | geometry-of-shortness | distribution-fidelity |
| dual-compatibility | duality-compatibility | arithmetic-legitimacy 的延伸 |
| shell-return | oracle-family、distribution-fidelity | 前面所有 invariant 的汇总落点 |

这张表的教学价值在于：

> 你现在不只知道一段 prose 属于哪个 tag，  
> 还知道它在全局上到底是在给哪条 invariant 续命。

**第二遍读原文时，怎样把 tag / obligation / invariant 三层一起用起来**

如果你回到 Section 4.2 原文，我建议每段做一个三行极简批注：

```text
[main tag]
obligation: ...
invariant: ...
```

例如：

```text
[freeness-legitimacy]
obligation: justify global free coordinates
invariant: coordinate-legitimacy
H=1
```

或者：

```text
[shell-return]
obligation: push abstract structure back to an oracle-legal sample family
invariant: oracle-family + distribution-fidelity
```

这比只写一个 tag 强得多，因为它会迫使你区分：

1. 这段表面在干什么；
2. 它必须证明什么；
3. 它全局上在保哪条 invariant。

**把这一页压成一句最该记住的话**

Section 4.2 真正难读，不是因为术语多，而是因为它在同时保多条 invariant。
最稳妥的读法不是：

> 作者又换了一层语言。

而是：

> 作者正在保护某条 reduction invariant 不在这一段里断掉。  
> tag 告诉我这段在哪一层；  
> obligation 告诉我它必须证明什么；  
> invariant 告诉我整条 reduction 为什么非保住这件事不可。

#### 一个 distribution-fidelity notebook：为什么 “最后还能写成 \(A,b=As+e\bmod q\)” 仍然不等于 reduction 可合法调用 solver

前面的 `invariant map` 已经把 `oracle-family invariant` 和 `distribution-fidelity invariant` 分开了。
但很多读者第一次看到这里时，仍然会下意识地把两者混成一句：

> 既然最后还是
> \[
> (A,b=As+e\bmod q),
> \]
> 那不就说明 reduction 已经成功回到 average-case MLWE 了吗？

不够。

这句话至多说明：

1. 你还保留着一个熟悉的 **语法壳**；
2. 公开关系仍像一个带噪声线性方程。

它并没有自动说明：

1. 这仍然是 solver 允诺接收的那一族样本；
2. 噪声 law 仍在同一类分布里；
3. “小”的含义没有在中途被改写掉；
4. 你送回去的 instance 不是一个只在外形上像 MLWE、但在 law 上已经偏离的对象。

这正是 `distribution-fidelity` 需要单独拉出来讲的一页。

**先给一句最短的判断**

在 reduction 里，

> **same formula shape**  
> 不等于  
> **same average-case law**。

前者只是在说你还能写出熟悉的等式；
后者才是在说你真的还能合法调用 solver hypothesis。

**先看一个最容易误导人的 toy picture**

回想前面 `5.16` 的 \(\mathbb{Q}(\sqrt2)\) toy model。
同一个元素既能写成 coefficient basis 坐标，也能写成 canonical embedding 坐标。

如果你在 coefficient basis 里拿一个“看起来各向同性”的噪声向量
\[
\mathbf{e}_{\mathrm{coeff}}\sim D
\]
然后经过一个非正交的 basis-change matrix \(B\)，得到
\[
\mathbf{e}_{\mathrm{emb}}=B\mathbf{e}_{\mathrm{coeff}},
\]
那么在 embedding 空间里它通常不再是“同一个球形 law”，而会变成某种椭球化分布。

注意这时公式外形仍可能完全没变：

\[
b=as+e\bmod q
\]

依旧成立。
但你已经不能再偷懒地说：

> 噪声还是同一种分布，只是换了个记号。

因为 law 已经随 basis geometry 被拉伸了。

这就是 Section 4.2 里最该时刻记住的一件事：

> syntactic shell 可以不变；  
> sample law 却可能已经变了。

**把 Section 4.2 里最常见的四类“保壳不保 law”风险单独列出来**

| 风险类型 | 表面看起来没变什么 | 真正可能变掉的是什么 | 为什么这会动到 solver hypothesis |
| --- | --- | --- | --- |
| 非正交 basis rewrite | 还是同一个 \(a,s,e\) 关系 | noise / shortness 的几何语义 | solver 可能需要的是某类几何上定义的 error family |
| arithmetic lift 只做对象重写 | 还是同一个商环元素 | 你还没说明这些元素在什么 law 下被抽样 | solver 假设不只关心对象落在谁上面，也关心它们怎样被抽到 |
| geometry lift 后没控制 pushforward | 只是去 embedding 里解释一下 | Gaussian / smoothing family 是否仍被正确运输 | worst-case to average-case 依赖的正是这类分布兼容 |
| shell-return 只保住等式外形 | 又写回 \(R_q^d\) 了 | 返回壳里的样本是否仍是 oracle promise 的 law class | oracle 不是“见到相似公式就工作”，而是对样本族有承诺 |

这张表最想纠正的误解是：

> reduction 最难的地方，常常不是“还能不能写出熟悉等式”；  
> 而是“还能不能把正确的分布一起带回去”。

**Case A：instance-writing 为什么一开始就已经在悄悄背 law 的问题**

很多人把 `instance-writing` 只理解成：

1. 指定 \(A,s,e,b\) 的类型；
2. 写出平均情形接口。

但它其实已经隐含了一个更强的问题：

> 你后面最终要调用的 solver，  
> 到底承诺接受哪一种样本 law？

也就是说，Section 4.2 一开头并不是只在固定语法；
它其实也在固定：

1. 环/模的 family；
2. 参数范围；
3. 噪声或误差项的抽样模型；
4. oracle 认为“这是同一问题”的判据。

如果这一层不先记住，后面 `shell-return` 时你就会误把“公式长得像”当成“law 也回来了”。

**Case B：arithmetic-lift 为什么还不等于 law 已经被保住**

当作者把
\[
R_q
\]
抬回
\[
\mathcal{O}_K/q\mathcal{O}_K
\]
时，你当然获得了 arithmetic parent。

但这一步本身只说明：

1. 对象生活在哪个更自然的 arithmetic universe；
2. ideals、modules、duals 有了合法宿主。

它还没有自动说明：

1. 原来的 error distribution 在这个 universe 里如何被解释；
2. 这种解释是否与 solver 假设的样本族兼容；
3. 后面从 arithmetic parent 再压回 shell 时，law 是否还能被正确带回去。

所以 `arithmetic-lift` 更像是在给 distribution 问题搭舞台，而不是直接解决它。

**Case C：geometry-lift 是 distribution-fidelity 最容易真正出问题的地方**

一旦进入 canonical embedding，很多“只是换个视角”的直觉都必须变严格。

因为此时你谈的已经不是单纯元素身份，而是：

1. shortness 在哪种欧氏范数下定义；
2. Gaussian 在哪种 lattice geometry 上定义；
3. 变换前后的 law 是否仍处在 reduction 允许的 family 里。

也就是说，`geometry-lift` 这层最该追问的是：

\[
\text{did we preserve the semantic law of the error, or only its notation?}
\]

如果这里没有明确控制，proof 就很容易退化成：

> 我把对象抬进了更高级的几何空间，  
> 但没有证明 solver 需要的那种 average-case law 也被一起搬过去了。

这正是 `distribution-fidelity invariant` 为什么和 `geometry-of-shortness invariant` 经常缠在一起，却又不是同一件事。

**Case D：shell-return 是最容易让人误判“已经回来了”的一步**

最后写回
\[
(A,b=As+e\bmod q)
\]
时，最常见的误判是：

> 既然已经又回到了 \(R_q^d\)，  
> 那就当然可以喂给 MLWE solver。

这仍然不够。
第二遍读到这一层时，你应该强制自己问：

1. 回来的只是对象壳，还是连抽样 law 也回来了？
2. 这个 \(e\) 还是 solver 假设里的那类 error 吗？
3. 若前面在 embedding 里或 arithmetic parent 里做过变换，这些变换有没有给出 law-level control，而不只是对象级同构？

只有当这些问题也被回答时，`shell-return` 才不只是“又写回熟悉记号”，而是真正回到了 oracle-legal sample family。

**把“保壳”和“保 law”压成一张对照卡**

| 你看到的现象 | 只能推出什么 | 还不能推出什么 |
| --- | --- | --- |
| 末尾又写成 \(A,b=As+e\bmod q\) | 公开关系外形仍像 MLWE | 这仍是 solver 假设中的同一 average-case law |
| 有一个很自然的 ring/module 同构 | 对象可在不同语言里搬运 | 样本分布也自动被无失真地搬运 |
| embedding 里 shortness 解释得更自然 | 几何语义更清楚 | solver-facing error family 没有变化 |
| free coordinates 合法了 | 坐标壳不再是假的 | law-level distortion 自动消失 |

这张卡片最值得背住的一句是：

> object transport is not yet distribution transport.

**class number one 在这里到底帮了什么，又没帮什么**

这一页也非常适合再次压实 \(h_k^+=1\) 的边界。

它直接帮你的是：

1. arithmetic objects 更显式；
2. free coordinates 更合法；
3. dual / codifferent 的 bookkeeping 更干净。

它没有直接替你完成的是：

1. 任意 basis change 下的 law preservation；
2. embedding/quotient/shell 之间一切 noise family 的无损运输；
3. “看起来还是同一公式” 到 “确实还是同一 average-case distribution” 的最后论证。

因此关于 `distribution-fidelity` 最稳妥的话不是：

> \(h_k^+=1\) 让这条 invariant 自动成立。

而是：

> \(h_k^+=1\) 清掉了若干结构障碍，  
> 从而让你更有希望把 law-level argument 讲干净；  
> 但 law 本身是否被保住，仍需单独证明。

**最后给一套最短的阅读自问句**

以后你回到 Section 4.2，只要看到一段在“把对象搬来搬去”，都可以先问四句：

1. 这里保住的是对象身份，还是连 distribution 也一起保住了？
2. 如果我只知道最后还能写成 \(A,b=As+e\bmod q\)，我是否已经知道 solver 可以被合法调用？
3. 这一步在动的是 arithmetic parent、coordinate shell，还是 error law 本身？
4. 若这一步没有额外分布控制，最可能断的是 `distribution-fidelity invariant` 还是 `oracle-family invariant`？

如果你把这四句问顺了，Section 4.2 里最微妙的那类“看似没变，实则可能已经变了 law”的段落，就不那么容易从你眼前滑过去。

**把这一页压成一句最终判断**

Section 4.2 里最危险的阅读偷懒，不是看不懂公式，而是太快相信：

> “公式壳已经回来了，所以 reduction 也已经回来了。”

更稳妥的判断是：

> reduction 想合法调用 average-case solver，  
> 不只要把对象写回熟悉壳里；  
> 还要把正确的 sample law 一起带回来。  
> `same shell` 从来不自动推出 `same law`。

#### 一个 reduction-step audit card：真正回到 Section 4.2 原文时，五次对象搬运各要逐项核对什么

如果把前面的 `tag`、`obligation`、`invariant`、`distribution-fidelity` 四页都看完，你其实已经具备了读懂 Section 4.2 的所有零件。

但真正回到原文时，很多读者还是会卡在一个非常具体的地方：

> 我知道这一段在做 arithmetic lift，  
> 也知道它大概要保某条 invariant，  
> 但如果让我逐句核对，我还是不知道“这一跳到底要查哪几项”。

所以这一页故意不再引入新概念。
它只做一件事：

> 把 Section 4.2 里最常见的五次对象搬运，压成一张真正可手持的 step-audit card。

你可以把它理解成：

```text
tag / obligation / invariant
    ->
turn into a concrete review checklist per transition step
```

**先给一张五步审计总表**

| step | 典型搬运 | 主 tag | 这一跳最该核对的 claim | 主 invariant | \(h_k^+=1\) 是否直接出场 | 若这一步失败，proof 最先坏在哪 |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | \(R_q^d\) 实例壳 -> arithmetic parent | arithmetic-lift | 这些对象确实来自 \(\mathcal{O}_K/q\mathcal{O}_K\) 的自然结构，而不是随便找的文雅重写 | arithmetic-legitimacy | 否，通常只是间接前置 | ideals / class number / codifferent 无处着陆 |
| 2 | arithmetic modules -> free coordinates | freeness-legitimacy | 这些坐标是全局合法的，不藏 hidden ideal-class twist | coordinate-legitimacy | 是，这里最直接吃到 \(h_k^+=1\) | 你后面写的 \(R_q^d\) 线性代数可能根本不是合法的标准坐标 |
| 3 | free coordinates -> embedding geometry | geometry-lift | shortness / Gaussian / hardness 语义确实被抬到 canonical embedding，而不是只换术语 | geometry-of-shortness | 否 | 证明会退化成只剩 coefficient shell 的代数变形 |
| 4 | embedding/arithmetic objects -> dual compatibility | dual-compatibility | quotient、pairing、codifferent、dual lattice 仍在同一套对象上兼容 | duality-compatibility | 是，这里最直接受 principal / inverse explicitness 影响 | Gaussian language、quotient 结构与对偶对象开始脱节 |
| 5 | abstract structure -> oracle-facing shell | shell-return | 返回去的不只是熟悉公式，而是 oracle-legal sample family | oracle-family + distribution-fidelity | 间接 | 你得到的只是“长得像 MLWE”的壳，而不是 solver 真能吃的实例 |

这张总表最重要的不是顺序本身，而是最后两列：

1. \(h_k^+=1\) 在哪里真的直接出场；
2. 每一步若失败，proof 会先坏在哪里。

因为这两列决定了你第二遍读原文时，注意力该放在哪。

**Step 1 审计卡：从 coefficient shell 抬回 arithmetic parent**

当你看到作者把
\[
R_q
\]
重新读成
\[
\mathcal{O}_K/q\mathcal{O}_K,
\]
请不要只在页边写一个 `arithmetic-lift` 就结束。
真正该核对的是：

1. 这一步是否只是在说“有一个同构存在”，还是已经说明 proof 里后续要用的 ideals/modules 都沿这一步有自然宿主？
2. 后面提到的 codifferent、inverse ideal、module structure，是否都真的 living over this arithmetic parent？
3. 如果删掉这一步，后面的 class-number 语言会不会立刻悬空？

最短页边批注可以写成：

```text
[arithmetic-lift]
obligation: justify the arithmetic parent of all later objects
invariant: arithmetic-legitimacy
```

**Step 2 审计卡：从 arithmetic modules 压成 free coordinates**

这一跳是 Section 4.2 最容易“看起来像线性代数常规操作”的地方。
但你真正要核对的是：

1. 这里的 basis 选择是不是全局合法，而不是局部 convenience？
2. 是否已经排除了 rank-one twist / ideal-class obstruction？
3. 后面所有 \(R_q^d\) matrix language，是否都建立在 genuinely free module 上？

在本文里，这正是 \(h_k^+=1\) 最直接承担 proof burden 的位置之一。

最短页边批注可以写成：

```text
[freeness-legitimacy]
obligation: turn arithmetic modules into genuine global coordinates
invariant: coordinate-legitimacy
H=1
```

如果这一步站不住，后面最先坏掉的不是某个 Gaussian 细节，而是：

> 你整个 \(R_q^d\) 壳都可能只是看起来像标准坐标。

**Step 3 审计卡：从 free coordinates 抬到 embedding geometry**

这一跳最常见的误读是：

> 反正只是把对象放到 canonical embedding 里看一眼。

真正该核对的是：

1. “短”是否已从 coefficient-size 语义切到 embedding-norm 语义？
2. Gaussian / smoothing / hardness 语句是否都在同一几何母语里？
3. 这一步有没有只保对象身份，却没保住 shortness 的真实语义？

最短页边批注可以写成：

```text
[geometry-lift]
obligation: move hardness and error semantics into embedding geometry
invariant: geometry-of-shortness
```

如果这一步失败，proof 会先坏成：

> 你还在写格密码的词，  
> 但真正的格几何已经没有被接回去。

**Step 4 审计卡：从 embedding/arithmetic objects 接到 dual compatibility**

这一跳最容易让人因为 codifferent 记号多而只做“名词识别”。
但第二遍读时真正该问的是：

1. quotient 与 pairing 是否仍通过同一套对象连接？
2. dual lattice 是否真是当前 reduction 需要的 natural dual，而不是随手挑的一种对偶？
3. inverse ideal / codifferent 的显式性是否在这里真的被用来消掉额外 bookkeeping？

这也是 \(h_k^+=1\) 另一处最直接减负的位置。

最短页边批注可以写成：

```text
[dual-compatibility]
obligation: align quotient, pairing, codifferent, and the natural dual lattice
invariant: duality-compatibility
H=1
```

若这一步没讲清，proof 最先坏成的不是“看起来不够优雅”，而是：

> mod-\(q\) 代数、Gaussian 语言与 dual object 已经不再处于同一条证明链里。

**Step 5 审计卡：从抽象结构再压回 oracle-facing shell**

这一步是最危险的，因为所有读者都天然会对熟悉的
\[
(A,b=As+e\bmod q)
\]
放松警惕。

但真正该核对的是：

1. 回来的是否不仅是公式壳，还是连 solver-facing family 也一起回来了？
2. distribution law 是否仍在 oracle promise 的范围内？
3. 前面所有 structure gains 是否都真的落回了一个可执行实例，而不只是留在抽象层？

最短页边批注可以写成：

```text
[shell-return]
obligation: push the proved structure back to an oracle-legal sample family
invariant: oracle-family + distribution-fidelity
```

若这一步失败，proof 最先坏掉的就是：

> 你只能说“我得到了一个长得像 MLWE 的实例”，  
> 却不能说“我得到了 solver 假设真正覆盖的那个 average-case family”。

**把五步再压成一张真正可抄在纸上的极简卡**

如果你想把这套东西带回原文页边，最短版本可以压成下面五行：

```text
1. arithmetic parent legit?
2. free coordinates legit?     H=1
3. shortness moved to embedding?
4. duality still compatible?   H=1
5. shell came back with the law?
```

读 Section 4.2 时，每遇到一个自然段，只要先判断它主要在这五行里的哪一行服务，阅读就会稳很多。

**把这一页压成一句真正可带走的话**

真正回到 Section 4.2 原文时，最有用的不是再记更多术语，而是记住：

> 这条 reduction 不是“一步把问题写复杂了”，  
> 而是分五次搬运同一个实例；  
> 每次搬运都必须同时回答  
> “这一跳是否合法、保住了什么、若失败会先坏在哪”。  
> 这才是逐段审计 proof 的正确手感。

#### 一个 reduction phrase decoder：读到这些句型时，作者其实在偷偷搬运什么 claim

到这里，其实已经有了四层阅读手柄：

1. `tag`；
2. `proof obligation`；
3. `invariant`；
4. `step-audit card`。

但很多读者真回到 Section 4.2 时，还是会在最原始的一步卡住：

> 我拿到的是英文 prose，  
> 还没来得及分段，就先被诸如  
> “regard as”, “identify with”, “under the canonical embedding”,  
> “via the trace pairing”, “thus given a solver...”  
> 这种句型冲走了。

所以这一页的目标更朴素：

> 不先问 theorem，不先问全局结构，  
> 先问“看到这种句型时，你的大脑应该立即弹出哪种 hidden claim”。

**先给一张 phrase decoder 总表**

| 典型 prose 句型 | 你应立即判成哪一层 | 这句话真正偷运的 claim | 最容易被误读成什么 | 若要继续审计，下一问该问什么 |
| --- | --- | --- | --- | --- |
| “regard \(R_q\) as \(\mathcal{O}_K/q\mathcal{O}_K\)” | arithmetic-lift | 这些对象现在有了 arithmetic parent，可合法谈 ideals/modules/duals | “只是把多项式商环换个更高级的名字” | 后面要用到的 ideal / module / codifferent 是否都沿这一步真正着陆了？ |
| “identify the module with a free rank-\(r\) object” | freeness-legitimacy | 坐标壳不再只是方便写法，而是全局合法坐标 | “作者只是为了排版整齐才写成 \(R_q^d\)” | 这里是否已经消掉 hidden ideal-class twist？\(h_k^+=1\) 是否在此直接减负？ |
| “under the canonical embedding” / “in Euclidean space” | geometry-lift | shortness / Gaussian / hardness 语义被抬到 embedding geometry | “只是换个基，几何不变” | 这里保住的是对象身份，还是连 shortness 的真实语义也被保住了？ |
| “via the trace pairing / codifferent / dual ideal” | dual-compatibility | quotient、pairing、dual lattice 正在被对齐成同一套对象 | “又多引入一层抽象记号而已” | 这里是否在保证 natural dual truly matches the quotient structure？ |
| “thus any solver for Module-LWE over \(R_q^d\) ...” | shell-return | 前面抽象结构已被压回 oracle-facing family | “作者只是把记号写回实现层” | 回来的到底是同一 sample law，还是只回来了熟悉公式壳？ |

这张表最重要的不是记忆英文短语，而是建立一种条件反射：

> 某些 prose 句型一出现，  
> 你不该只看字面意思，  
> 而该立刻追问它在偷运哪一层合法性。

**Phrase A：regard / realize / view as**

原文里只要出现类似：

> regard \(R_q\) as a quotient of the ring of integers

你就不该只把它当成“数学家更喜欢这种写法”。
这类句型的标准译码应当是：

```text
we are establishing the arithmetic parent of later objects
```

也就是说，它通常不是装饰性 prose，而是在声明：

1. 后面出现的 ideals/duals 不再悬空；
2. 当前对象不再只是 implementation shell；
3. number-theoretic language 现在终于有合法宿主。

如果你读到这种句型却没自动想到 `arithmetic-legitimacy invariant`，后面就很容易把 class number、codifferent 读成“突然冒出来的高层术语”。

**Phrase B：identify with / choose a basis / free module**

这类句型是 Section 4.2 里最容易被轻读的。
例如：

> identify the module with a free module of the same rank

或者：

> choose coordinates over \(R_q^d\)

标准译码不该只是：

> 好，作者选了一组基。

而应当是：

```text
the proof is paying off a coordinate-legitimacy debt
```

因为这类句型背后真正要解决的是：

1. 这些坐标是否 genuinely global；
2. 是否仍有 ideal-class twist 藏在后台；
3. 写成 \(R_q^d\) 后，你后面做的线性代数是否已被合法化。

在本文语境里，这也是你最该自动问：

> 这里是不是 \(h_k^+=1\) 直接出场替作者免债了？

的地方。

**Phrase C：under the canonical embedding / in the Euclidean norm / discrete Gaussian**

这类句型是很多读者最容易误判成“只是切到几何直觉”之处。

标准译码应当是：

```text
the proof is now moving the semantics of shortness and error into its real geometry
```

也就是说，作者不只是在换视角，而是在要求：

1. hardness statement 真正 living in this Euclidean model；
2. Gaussian / smoothing 也共用同一套几何母语；
3. 你后面再谈 “small error” 时，small 已经不是 coefficient-size 的朴素意思。

如果读者把这类句型只当成“换个基”，就会漏掉 Section 4.2 里最硬的一层几何 burden。

**Phrase D：via the trace pairing / with respect to the codifferent / dual ideal**

这类句型最容易让人产生“抽象层数又+1”的疲劳。

标准译码应该是：

```text
the proof is aligning quotient structure with the natural dual object
```

也就是说，这里真正要问的不是“这个定义会不会考试”，而是：

1. 现在的 dual object 是否真是 current reduction 所需的 natural dual？
2. quotient 与 pairing 是否还在同一条证明链里？
3. inverse ideal / codifferent 的显式性是不是正在被拿来消掉 bookkeeping？

读到这种句型时，如果你没有自动跳到 `duality-compatibility invariant`，就会把最承重的一层误读成纯记号偏好。

**Phrase E：thus / therefore / given a solver for Module-LWE ...**

这类句型通常出现在段落收尾，非常容易让人产生一种错误轻松感：

> 哦，终于又回到 solver 了。

但标准译码应当是：

```text
the proof is attempting to cash out all previous structure into an oracle-legal family
```

也就是说，你此时真正该问的是：

1. 回来的到底只是 familiar shell，还是连 law 也一起回来了？
2. 这个 `therefore` 是对象级 therefore，还是分布级 therefore？
3. 若这里偷换了 average-case family，前面所有结构努力是不是都白费了？

这类句型几乎总应该触发你去检查：

1. `oracle-family invariant`；
2. `distribution-fidelity invariant`。

**把五类句型再压成一张最短速查卡**

如果你打算真的把这套东西带到原文页边，最短版可以压成：

```text
regard as        -> arithmetic parent?
identify with    -> coordinates really legal?
under embedding  -> shortness moved where?
via pairing      -> natural dual still aligned?
thus solver      -> same family or only same shell?
```

这张速查卡的价值在于：

> 它把一整页英文 prose  
> 压成几个一眼就能触发正确问题的句型开关。

**最后把这一页压成一句最该带走的话**

真正读 Section 4.2 原文时，最有效的技巧往往不是更会算，而是更会识别句型。
最稳妥的读法不是：

> 这句话听起来很自然，所以应该只是过渡句。

而是：

> 在 reduction prose 里，很多最像过渡句的句子，  
> 恰恰正在偷偷搬运最重的 claim。  
> 先把句型译码对，再去看 theorem 细节，速度反而更快。

#### 一个 paragraph-audit template：真正回到 Section 4.2 原文时，每个自然段先写哪六格

前面的 `worksheet`、`obligation ledger`、`invariant map`、`step-audit card` 与 `phrase decoder` 已经把读 Section 4.2 的零件拆开了。
但真正回到原文时，很多读者还是会卡在一个非常具体的问题：

> 我知道这一段大概在谈 freeness 或 duality，  
> 也知道某个句型在偷运 hidden claim，  
> 但如果我只允许自己在页边写最少的话，  
> 到底先写哪几格，才能既不失真、又不把页面写炸？

这时最有效的办法不是再发明一套新术语，而是把批注动作本身压成一个六格模板。

这页模板的定位非常克制：

1. 它不是替你改写原文。
2. 它不是伪造一段“作者本来应该怎样写”的 prose。
3. 它只是把你对某个自然段的第一轮审计，压成六个最有信息量的槽位。

**先给六格最小模板**

| 你要先写的格子 | 建议长度 | 这格真正想抓住什么 | 如果这一格写不出来，先回跳哪页 |
| --- | --- | --- | --- |
| literal claim | 1 句 | 这段字面上到底在声称什么，不掺你的解释 | phrase decoder，先把句型译码 |
| hidden import | 1 句 | 这段若想成立，还偷偷借了哪层合法性或分布控制 | obligation ledger |
| bounce page | 1 个节号或短标签 | 这段一旦卡住，你该回本讲义哪一页补背景 | 10.2、distribution-fidelity notebook、step-audit card 等 |
| H=1 status | direct / indirect / no | \(h_k^+=1\) 在这里是直接出力，还是只在更早处清了路 | Step 2 / Step 4 那张审计卡 |
| transport type | object / law / both | 这段只是在搬对象，还是连 solver-facing law 也在搬 | distribution-fidelity notebook |
| failure mode | 4 到 8 个词 | 这段若站不住，proof 第一处会坏在哪里 | invariant map |

这六格里，最容易混淆的是前两格：

1. `literal claim` 只准写作者这段表面正在做的事。
2. `hidden import` 才写 “这件事若要成立，还得额外保住什么”。

如果你一上来就把两格混在一起，后面就会很难分清：

> 作者到底真的在这段里证明了什么；  
> 哪些只是你脑中帮他自动补上的东西。

**页边最短写法**

如果你想把这六格压成真正能手写在纸边的格式，可以直接抄这一版：

```text
literal:
imports:
bounce:
H=1:
transport:
watch:
```

这里的 `watch` 就是 `failure mode` 的极简版。
它不要求你写长解释，只要求你明确指出：

> 这段若不成立，最先坏的是 coordinates、duality、geometry，还是 solver law。

**三种最常见的填写方式**

下面不伪造论文原句，只拿三类最常见的 Section 4.2 段落原型，示范这六格应该怎么填。

| 你读到的段落原型 | 六格里最该怎么填 | 这张卡最想防的误读 |
| --- | --- | --- |
| “identify the module with a free rank-\(r\) object” 一类句子 | literal: choose global coordinates; imports: no hidden ideal-class twist remains; bounce: 10.2 freeness notebook; H=1: direct; transport: object; watch: fake coordinates | 把“选一组基”误读成纯排版动作 |
| “under the canonical embedding / discrete Gaussian” 一类句子 | literal: shortness now lives in Euclidean geometry; imports: error semantics must survive the lift; bounce: 5.16 + distribution-fidelity notebook; H=1: no; transport: law; watch: geometry/law drift | 以为作者只是换个视角，并没有在动 hardest part |
| “thus given a solver for Module-LWE ...” 一类句子 | literal: cash out the reduction through an oracle; imports: returned samples stay inside the promised family; bounce: step-audit Step 5; H=1: indirect; transport: both; watch: oracle mismatch | 以为熟悉公式壳一回来，solver 就当然可调用 |

这三行示范真正想训练的是一种顺序：

1. 先把这段表面 claim 压成一句短句。
2. 再把它暗中借走的 burden 单独写出来。
3. 最后才判断 \(h_k^+=1\) 是否真的在这里直接出手。

**为什么这一页特别强调 H=1 要写成 direct / indirect / no**

读 Section 4.2 时，一个非常常见的过度归功是：

> 只要这段附近正在讨论 freeness 或 codifferent，  
> 就把整段都标成 “this is where Weber enters”.

更稳妥的做法是把判断压成三档：

1. `direct`：这段若没有 class number one，作者就还欠一层 freeness / principality / explicit dual bookkeeping。
2. `indirect`：这段本身不直接吃 \(h_k^+=1\)，但它站在 earlier structure gains 之上。
3. `no`：这段主要在搬几何语义、分布控制或 solver 接口，Weber 结果没有直接替它免债。

这样标的好处是，你不会把 `H=1` 神化成：

> 它只要出现一次，就同时解决同页所有技术层。

**为什么 transport type 必须单列一格**

这一格专门防一个阅读错觉：

> 既然这段还在谈同一个对象，  
> 那 distribution 应该也还跟着没变。

这在 Section 4.2 里经常不安全。
你最好强迫自己明确写成三类之一：

1. `object`：这段主要在确认对象宿主、坐标或对偶对象，没有直接在 claim law preservation。
2. `law`：这段最敏感的 burden 是 shortness / Gaussian / average-case law 是否被正确运输。
3. `both`：这段既在搬对象，也在试图把对象与 law 一起压回 oracle-facing family。

只要你把这一格单列出来，`same shell` 与 `same law` 就不那么容易在脑子里自动合并。

**最后给一个真正可重复使用的空白卡**

以后你回看 Section 4.2 的任何自然段，都可以先只填这一张：

```text
[paragraph audit]
literal:
imports:
bounce:
H=1: direct / indirect / no
transport: object / law / both
watch:
```

如果第一遍只能填出 `literal` 和 `bounce`，也完全没问题。
真正重要的是：

> 你开始把“阅读自然段”这件事，从被 prose 推着走，  
> 变成主动审计每段到底在搬什么 burden。

**把这一页压成一句最该带走的话**

回到 Section 4.2 原文时，最有用的往往不是再记一个新术语，而是学会把每个自然段拆成：

> 它字面说了什么；  
> 它暗中借了什么；  
> \(h_k^+=1\) 在这段是直接、间接，还是根本没出手；  
> 以及这段若失败，proof 第一处会坏在哪里。

#### 一个 worked paragraph-audit page：把 Section 4.2 的五类原型段落真的逐格批完

上面的 `paragraph-audit template` 已经给了你六格空白卡。
但真正第一次回原文时，很多读者仍会卡在：

> 我知道该填哪六格，  
> 可我还不知道一段典型 reduction prose  
> 被填完以后应该长成什么样。

所以这页不再解释模板，而是直接给五个 **source-like but synthetic** 的段落原型。
它们不是论文原句，只是故意压成最接近 Section 4.2 阅读手感的 paragraph shapes。

每个原型都只做两件事：

1. 先给一段最短 prose 原型；
2. 再把六格审计卡真的填完。

这样你回到原文时，就不必先从空白页边起步。

**原型 A：arithmetic-lift 段落到底该怎么批**

最常见的原型之一长得像：

> We regard the quotient ring as descending from \(\mathcal{O}_K/q\mathcal{O}_K\), so that the relevant samples inherit the ambient arithmetic structure of the ring of integers.

第一轮页边批注最稳妥的写法是：

```text
[paragraph audit]
literal: move the quotient shell back to an arithmetic parent
imports: later ideals/modules/duals must all live over the same O_K parent
bounce: reduction-reading notebook + phrase decoder
H=1: indirect
transport: object
watch: arithmetic parent floats
```

这张卡最关键的是 `literal` 和 `imports` 不能混写。

- `literal` 只说这段正在把对象抬回 \(\mathcal{O}_K\)；
- `imports` 才说：后面的 ideal、module、codifferent 语言都得真的沿这一步着陆。

所以这类段落虽然常常看起来只是一个 “regard as” 句型，但它真正承担的是：

> 给后面的 class-number / duality 语言找合法宿主。

**原型 B：freeness-legitimacy 段落到底该怎么批**

另一类最承重的段落常会写成：

> Under the class-number-one hypothesis, the torsion-free module may be identified with a free module of the same rank, and we may therefore work in genuine global coordinates.

这类段落的第一轮页边卡最适合写成：

```text
[paragraph audit]
literal: turn the arithmetic module into genuine global coordinates
imports: no hidden ideal-class twist remains behind the chosen basis
bounce: 10.2 freeness notebook + step-audit Step 2
H=1: direct
transport: object
watch: fake coordinates
```

这里最值得死记的一点是：

> **H=1: direct**

因为这正是 \(h_k^+=1\) 最典型、最不该被轻读成“只是选了个好基”的位置。
如果这一段站不住，proof 最先坏掉的不是 Gaussian 细节，而是：

> 你后面所有写成 \(R_q^d\) 的线性代数  
> 都可能只是看起来像标准坐标。

**原型 C：geometry-lift 段落到底该怎么批**

第三类原型经常会把读者骗成“只是换个视角”：

> Under the canonical embedding, the error distribution is interpreted geometrically, so that shortness and Gaussian behavior are read in the ambient Euclidean space rather than coefficientwise.

这类段落的六格卡最稳妥的是：

```text
[paragraph audit]
literal: move shortness and error semantics into embedding geometry
imports: the law of the error must survive the lift, not only the object identity
bounce: 5.16 bridge notebook + distribution-fidelity notebook
H=1: no
transport: law
watch: geometry/law drift
```

这里最容易犯的错，是把 `transport: law` 偷偷脑补成 `object`。
但这类段落真正危险的地方恰恰在于：

> 你不仅要把对象送到 embedding 里，  
> 还要把 “small / Gaussian / short” 的语义一起送过去。

所以这类段落通常最该防的，不是坐标错误，而是：

> error law 在换语言时已经悄悄变形。

**原型 D：dual-compatibility 段落到底该怎么批**

第四类原型常常带有 trace pairing、codifferent、dual ideal 一类词：

> Via the trace pairing and the codifferent, the quotient structure can be aligned with the natural dual object required by the reduction.

这类段落的第一轮审计卡可以直接写成：

```text
[paragraph audit]
literal: align quotient structure with the natural dual object
imports: principal / inverse bookkeeping is explicit enough to keep pairing natural
bounce: codifferent notebook + step-audit Step 4
H=1: direct
transport: object
watch: duality mismatch
```

这里 `H=1` 再次是 `direct`，但原因和 freeness 那页不同。
在这里，\(h_k^+=1\) 减掉的主要不是“有没有合法坐标”，而是：

> inverse ideal / codifferent / principal bookkeeping  
> 能不能被写成无条件、显式、同一条证明链上的对象。

如果读者把这类段落只当成“抽象记号又多了一层”，就会直接错过 reduction 里最承重的 glue layer。

**原型 E：shell-return 段落到底该怎么批**

最后一类最危险的段落通常出现在收束处：

> Hence, given a solver for Module-LWE over the resulting \(R_q^d\)-shell, one obtains the corresponding hardness transfer for the original structured family.

这类段落最该写成：

```text
[paragraph audit]
literal: cash out the previous structure through an oracle-facing shell
imports: returned samples must stay inside the promised average-case family
bounce: step-audit Step 5 + distribution-fidelity notebook
H=1: indirect
transport: both
watch: oracle mismatch
```

这张卡里最关键的是：

> **transport: both**

因为这类段落既在把对象写回 familiar shell，也在尝试把 solver-facing law 一起写回去。
如果你只盯着公式壳
\((A,b=As+e\bmod q)\)，
就很容易漏掉真正该查的那件事：

> 回来的到底是同一个 average-case family，  
> 还是只回来了同一种外观。

**把五张 worked cards 再压成一张“段落看到什么就先写什么”的顺序表**

| 你先看到的 prose 线索 | 第一反应先写什么 | 最该防的误读 |
| --- | --- | --- |
| regard as / quotient of O_K | literal: arithmetic parent | “只是更文雅的记号” |
| identify with a free module | H=1: direct + watch: fake coordinates | “只是选了个基” |
| under the canonical embedding | transport: law | “对象没变所以分布也没变” |
| via the trace pairing / codifferent | watch: duality mismatch | “只是又加了一层术语” |
| thus given a solver | transport: both + watch: oracle mismatch | “公式壳回来就能直接调 solver” |

这张顺序表真正想训练的是一种固定动作：

> 先判这段最像哪一类原型，  
> 再从六格里挑两格最关键的先写。  
> 不需要第一眼就六格全满，  
> 但一定要先把最危险的那一格写出来。

**最后把这一页压成一句最该带走的话**

Section 4.2 的 paragraph audit 不是在考你会不会复述 prose，而是在考你能不能立刻分清：

> 这一段是在搬对象、搬 law，还是两者一起搬；  
> \(h_k^+=1\) 是直接出手、间接出手，还是根本没出手；  
> 以及它一旦失手，proof 第一处会先坏在哪。

#### 一个 paragraph-chain notebook：不是每段都看懂就够了，还要看 debt 怎样一段段接力

到这里，读者通常已经能做两件事：

1. 给单个自然段打 tag；
2. 给单个自然段填六格 audit card。

但真正回到 Section 4.2 原文时，仍然会有一个更深的困难：

> 就算每段单独都能看懂，  
> 我还是不确定这几段连起来  
> 到底是在怎样接力完成 reduction。

换句话说，上一页解决的是：

> 单段 paragraph 在搬什么 burden？

这一页要解决的是：

> 前一段把什么 debt 传给下一段，  
> 下一段又到底替谁还债？

这一步很重要，因为很多 reduction prose 的误读并不发生在单句，而发生在相邻两段之间：

1. 读者以为前一段已经证明完了；
2. 实际上它只是把对象送到下一段该工作的地方；
3. 下一段若没读懂，前一段也会被误听成更强 claim。

**先给一条六段 reduction strip**

下面这条 strip 不是论文原文，只是把 Section 4.2 最典型的一条阅读路径压成六段连续 prose：

1. `instance-writing`：把 average-case shell 写出来；
2. `arithmetic-lift`：说明它不是 arbitrary shell，而有 \(\mathcal{O}_K\) 母对象；
3. `freeness-legitimacy`：把 arithmetic module 真正合法化成 global coordinates；
4. `geometry-lift`：把 shortness / Gaussian 语义抬到 embedding；
5. `dual-compatibility`：让 quotient、pairing、dual object 回到同一条证明链；
6. `shell-return`：把前面的结构成果压回 solver-facing family。

如果你只把这六段当成六个互不相干的知识点，阅读会很碎。
真正更稳的读法是：

> 逐段问  
> “这一段是从上一段接到了什么 debt，  
> 又把什么 debt 传给下一段？”

**先给一张 debt-handoff 总表**

| strip step | 这段表面在做什么 | 它从上一段接到什么 debt | 它在这一段真正还掉什么 debt | 它还留给下一段什么 debt | \(h_k^+=1\) | 若这一段被轻读，下一段最容易怎么被误听 |
| --- | --- | --- | --- | --- | --- | --- |
| 1. instance-writing | 写出 \(A,s,e,b\) 与 solver interface | 无；这里只是起点 | average-case problem 至少有了明确输入输出 | 这些对象到底住在哪个 arithmetic parent 上 | no | 后面看见 \(\mathcal{O}_K\) 语言时，误以为作者突然换题 |
| 2. arithmetic-lift | 把 shell 抬回 \(\mathcal{O}_K/q\mathcal{O}_K\) | “这个壳必须有真正的数论宿主” | ideal / module / codifferent 终于有合法落脚点 | 坐标是否 genuinely global，而非扭曲 module 的假外观 | indirect | 把下一段 freeness 误听成“只是选了个方便基” |
| 3. freeness-legitimacy | 合法化 global coordinates | “这些 arithmetic modules 还不能自动写成标准坐标” | hidden ideal-class twist 被显式清掉，\(R_q^d\) 不再只是外观 | “短” 与 Gaussian 该在哪个几何里解释 | direct | 把 geometry 段误听成“只是在标准坐标里再加一点直觉” |
| 4. geometry-lift | 把 error / shortness 送进 embedding | “坐标合法了，但 small 还没有几何语义” | coefficient shell 与 Euclidean hardness 之间建立真正语义桥 | quotient、pairing 与 natural dual 是否仍兼容 | no | 把 duality 段误听成“纯记号偏好”，而不是维持几何接口 |
| 5. dual-compatibility | 把 quotient、pairing、dual object 对齐 | “几何语义有了，但不同对象还未必在同一链上兼容” | natural dual 与 quotient structure 被重新粘回同一 proof line | 能否把这一切合法压回 solver family | direct | 把最后的 thus solver 段误听成“当然能调 solver 了” |
| 6. shell-return | 写回 solver-facing family | “前五段的结构成果必须真的落回 oracle promise” | 对象壳与 law 一起回到可调用的 average-case family | 无；这是 cash-out 段 | indirect | 直接把 same shell 听成 same law |

这张表真正重要的不是第二列，而是中间三列：

1. `接到什么 debt`
2. `这段还掉什么 debt`
3. `又把什么 debt 传下去`

因为这三列会强迫你停止把相邻段落读成互相独立的说明书。

**为什么 Step 2 和 Step 3 必须分开读**

第一次读 Section 4.2 时，最常见的粗读就是把：

1. `arithmetic-lift`
2. `freeness-legitimacy`

合并听成一句：

> “作者把 \(R_q\) 重新解释成更有结构的对象，然后顺便选了坐标。”

这会直接错掉两件事：

1. `arithmetic-lift` 负责的是 **母对象合法性**；
2. `freeness-legitimacy` 负责的是 **全局坐标合法性**。

也就是说，前者在回答：

> 后面的 ideal / module / codifferent 到底住在哪？

后者在回答：

> 既然住在这里，为什么你有权把它们真写成标准坐标？

如果把这两段糊成一段，你就会错过 \(h_k^+=1\) 第一次真正直接替作者免债的位置。

**为什么 Step 4 和 Step 5 也不能塌成同一层**

第二个最常见的合并误读，是把：

1. `geometry-lift`
2. `dual-compatibility`

都听成“接下来作者在做更抽象的格几何”。

但这两段的 debt 完全不同：

- `geometry-lift` 负责把 “small / Gaussian / short” 搬到真正的欧氏语义里；
- `dual-compatibility` 负责把这些几何对象与 quotient / pairing / dual lattice 粘回同一条 reduction。

所以更准确的顺序是：

```text
first:
decide where shortness really lives

then:
make that geometry compatible with the algebraic quotient machinery
```

一旦把这两段混成一锅，读者就会把 codifferent 一类对象误听成：

> 作者只是想把 embedding 记号写得更高级。

实际上它在这里承担的是 glue layer，而不是 decorative layer。

**为什么最后一段最该反着读**

`shell-return` 段最危险，因为形式上它最像“终于回到熟悉公式了”。
教学上最稳的读法不是顺着它读，而是反着问：

1. 这一段想 cash out 到哪个 solver promise？
2. 它必须继承前面哪几段已经还掉的 debt，才能合法 cash out？
3. 如果前一段 duality glue 没站住，这里究竟还能不能说 `thus given a solver`？

所以最后一段最好的页边动作不是写：

```text
solver call
```

而是写：

```text
what exact law is now being claimed back?
```

只有这样，你才不容易把：

> “能写回熟悉壳”

误听成：

> “能写回 solver 真正承诺的样本族”。

**给读者一张真正可回带到原文上的三格链式卡**

如果你不想每次都抄完整张大表，可以把 sequence-level 审计压成下面三格：

```text
received debt:
paid here:
passed next:
```

这张三格卡的作用不是替代六格卡，而是补六格卡缺的那半边：

- 六格卡让你看清单段；
- 三格链式卡让你看清段与段之间的接力。

以后你读 Section 4.2 的任意连续两三段，都可以先给每段写：

1. 这段从上一段继承了什么；
2. 这段真正清掉了什么；
3. 这段又把什么遗留问题送给下一段。

**最后把这一页压成一句最该带走的话**

Section 4.2 真正难的地方，不是每一段都很深，而是：

> 每一段都只还一部分债。  
> 读者若不看 debt 怎样逐段接力，  
> 就会把“只是把问题送到下一段”  
> 误听成“这一段已经把问题解决了”。

#### 一个 reduction-certificate notebook：读 Langlois-Stehle-style MLWE proof 时，每一步你手里到底该拿到什么证书

前面的 `paragraph-audit template`、`worked paragraph-audit page` 和 `paragraph-chain notebook`
已经能帮助你回答：

1. 单段在搬什么 burden；
2. 相邻两三段之间怎样交接 debt。

但如果你真的想把 Section 4.2 读成一条完整 proof，还有一个更硬的阅读动作必须补上：

> 每走完一段关键 proof job，  
> 你手里到底该拿到哪张 “已经证明到这里了” 的 certificate？

这一步为什么重要？
因为很多读者虽然能说出：

- 这里在谈 freeness；
- 那里在谈 codifferent；
- 最后又回到了 solver；

但仍然分不清：

> 这些段落到底已经证明了什么，  
> 以及下一步为什么现在才有资格发生。

所以这一页故意把 Section 4.2 的阅读方式再收紧一层：

> 不再按 paragraph 或 debt 来看，  
> 而是按 “proof 到这一步结束时，我必须能写出的 certificate” 来看。

**先给一条 certificate chain**

把 Langlois-Stehle-style 的 MLWE hardness proof 读成教材时，最稳妥的 certificate chain 大致是：

```text
certificate 1:
this is the exact average-case family we are trying to solve

certificate 2:
these instances genuinely live over the intended arithmetic parent

certificate 3:
the coordinates are globally legitimate free-module coordinates

certificate 4:
shortness and Gaussian semantics now live in the right geometry

certificate 5:
the quotient, pairing, and natural dual object are aligned

certificate 6:
the resulting shell still belongs to the solver's promised family
```

如果某一张 certificate 你写不出来，就不应该急着相信下一张。
因为下一步常常只是：

> 站在上一张 certificate 已经成立的前提上继续走。

**先给一张六证书总表**

| proof job | 这一段真正该产出的 certificate | 你最该问的判据 | \(h_k^+=1\) | 若没有这张 certificate，下一步为什么没有资格成立 |
| --- | --- | --- | --- | --- |
| 1. instance shell | the target average-case family is fixed | solver/oracle 的输入输出是否已经清楚，不再只是模糊提 average-case MLWE | no | 连“后面到底要还原到哪个 family”都没固定，后续一切都像换题 |
| 2. arithmetic parent | the shell truly descends from the intended O_K-parent | ideals/modules/codifferent 是否都有共同宿主，而不是只换了记号 | indirect | freeness、class number、duality 都会失去合法落点 |
| 3. freeness legitimacy | the coordinates are globally legal, not merely convenient | 写成 \(R_q^d\) 是否已经排除 hidden ideal-class twist | direct | 后面谈 embedding 或 solver 时，你用的壳可能根本不是合法标准坐标 |
| 4. geometry lift | shortness and Gaussian law are now interpreted in the right Euclidean model | small / short / Gaussian 是否已经脱离 coefficient-size 的朴素语义 | no | 后面所有 hardness / Gaussian 句子都可能只剩空洞术语 |
| 5. dual compatibility | the quotient machinery and the natural dual object are aligned | trace pairing、codifferent、dual lattice 是否仍在同一套对象上兼容 | direct | 最后的 shell-return 只会回到表面公式，回不到合法的 reduction chain |
| 6. oracle cash-out | the returned shell still satisfies the solver promise as a law, not only as a syntax shell | 回来的是否真是 solver 承诺的 average-case family，而不是长得像它 | indirect | 你最多只能说“公式像 MLWE”，还不能说“可以调用 MLWE solver” |

这张表最重要的不是第一列，而是第二列：

> 每一段 proof job 结束时，  
> 你手里到底该拿到什么一句话证书。

只要你强迫自己把这句话写出来，很多 prose 里“似乎已经证明很多”的段落会自动降到更准确的位置。

**证书 3 和证书 5 为什么是 H=1 的两次真正落地**

教材化阅读时，最值得单独钉住的是：

1. `certificate 3`：global free coordinates legitimate；
2. `certificate 5`：natural dual / codifferent machinery aligned.

这两张证书不是因为名字高级才重要，而是因为：

> 它们正是 \(h_k^+=1\) 最直接替 proof 免债的两个位置。

更准确地说：

- `certificate 3` 解决的是“我到底有没有权利把底层 module 当成标准坐标对象来算”；
- `certificate 5` 解决的是“我在 embedding、quotient、pairing、dual 之间来回切换时，有没有藏着额外 ideal-class bookkeeping”。

所以以后你在 Section 4.2 里看到一句

> “class number one makes the reduction cleaner”

更稳妥的自动翻译应当是：

> 它大概率是在帮作者拿到 `certificate 3` 或 `certificate 5`，  
> 而不是在替整条 proof 自动省掉所有技术步骤。

**为什么证书 4 和证书 6 最容易被读者偷换**

读者最常见的两种错觉是：

1. 坐标一旦合法，就以为几何语义也自动合法；
2. 公式壳一旦回来，就以为 solver promise 也自动回来。

这恰好就是 `certificate 4` 和 `certificate 6` 必须分开的原因。

`certificate 4` 真正说的是：

> “small / short / Gaussian”  
> 已经在 canonical embedding 的欧氏模型里被正确解释。

而 `certificate 6` 真正说的是：

> 这些已经被几何化、对偶化、再压回去的对象，  
> 仍然落在 solver 假设覆盖的 average-case law 里。

因此这两张证书中间，隔着一个必须反复警惕的陷阱：

```text
right geometry
    !=
automatically right oracle family
```

也正因为如此，`shell-return` 段最不应该被读成“收尾句”，而应被读成：

> 整条 proof 最后一次、也是最危险的一次合法性 cash-out。

**给读者一张真正可手抄的 certificate card**

如果你想把这一页带回原文页边，最短可以压成下面四格：

```text
certificate:
why earned here:
which next step now becomes legal:
if missing, next step is fake because:
```

这张卡的使用方式很直接：

1. 每遇到一段看上去像“阶段收束”的 prose；
2. 先逼自己写一句 `certificate:`；
3. 再问下一段到底是在消费这张证书，还是还在重新证明它。

只要这一步做对，你就会更少把“过渡句”误读成“结论句”。

**最后把这一页压成一句最该带走的话**

Section 4.2 不只是把对象一层层搬过去，而是在每个关键节点都必须交出一张新的 proof certificate。
如果你不能明确说出：

> “这一段结束后，我手里究竟多了一张什么证书，  
> 所以下一段现在才有资格继续”；

那就说明你还没有真正跟住这条 reduction。

#### 一个 reduction I/O notebook：真正贴近 proof 读法时，每一步都要写出 in → move → out

到这里，Section 4.2 已经有了三种阅读手柄：

1. `paragraph audit`：单段在搬什么 burden；
2. `paragraph chain`：相邻段落怎样传递 debt；
3. `certificate chain`：每个 proof job 结束时你手里多了哪张证书。

但如果你真的要把一条 reduction prose 当成 proof 来跟，还有最后一层最硬的动作：

> 不只问 “这段想说什么”，  
> 还要问 “它到底把什么对象送进来，经过什么合法 move，送出了什么对象”。

也就是说，你需要的不只是 paragraph notes，而是一张真正的：

```text
in
  ->
move
  ->
out
```

卡片。

这张卡为什么重要？
因为很多读者在 Section 4.2 真正卡住的点，不是术语不认识，而是：

1. 看见对象换了名字；
2. 却不知道 proof 到底有没有合法地把旧对象变成新对象；
3. 更不知道这一步应该保什么、不该默认什么。

所以这一页的目标非常具体：

> 把 Langlois-Stehle-style 的 MLWE reduction  
> 压成六个 proof moves；  
> 每个 move 都写出  
> `incoming object`、`allowed move`、`outgoing object`。

**先给一张六步 I/O 总表**

| proof move | in | move | out | 这一步真正依赖什么 | 最该防的偷换 |
| --- | --- | --- | --- | --- | --- |
| 1. shell fixation | average-case samples written over \(R_q^d\) | fix the exact solver-facing family | a precise target shell | problem statement discipline | “反正就是某种 MLWE” |
| 2. arithmetic realization | \(R_q^d\)-shell | realize \(R_q\) as descending from \(\mathcal{O}_K/q\mathcal{O}_K\) | arithmetic parented modules/quotients | quotient-of-integers structure | “只是把记号写得更高雅” |
| 3. coordinate legitimization | torsion-free arithmetic modules | use freeness to choose genuine global coordinates | free coordinates over the intended quotient | Steinitz + \(h_k^+=1\) | “选了个方便基而已” |
| 4. geometric reinterpretation | free coordinates carrying candidate errors/secrets | reinterpret shortness and Gaussian semantics in canonical embedding | geometric objects with Euclidean meaning | embedding geometry / ideal-lattice semantics | “对象没变，所以 small 的意义也没变” |
| 5. dual alignment | geometric + arithmetic objects | align quotient, pairing, dual object, codifferent | a duality-compatible proof state | trace pairing + codifferent + inverse bookkeeping | “又多了一层可选抽象” |
| 6. oracle cash-out | duality-compatible structured state | push everything back to an oracle-facing average-case shell | solver-legal family, not only shell-shaped samples | distribution-fidelity + oracle-family preservation | “长得像 MLWE 就能调 solver” |

这张表最有价值的地方，是它把 “proof move” 从 prose 印象压成了对象流：

> 这一段之前你手里拿着什么对象；  
> 这一步允许你做什么；  
> 做完之后你手里究竟换成了什么对象。

**把六步压成真正可手写的 I/O 卡**

如果你想把这层方法直接带回原文页边，最短可以只写下面四格：

```text
in:
move:
out:
what must still be invariant:
```

这四格和前面的六格 audit card、四格 certificate card 不冲突，反而互补：

- six-box audit card 回答 “这段在背什么 burden”；
- certificate card 回答 “这段结束后证明到哪了”；
- I/O card 回答 “对象到底怎样被合法搬过去了”。

**为什么 Step 2 和 Step 3 最该用 I/O 卡读**

Section 4.2 里最容易在对象流上犯错的，就是：

1. `arithmetic realization`
2. `coordinate legitimization`

因为这两步在 prose 上都很像“对象重新解释了一下”。
但 I/O 写法一旦展开，差别就会非常硬：

```text
Step 2
in:  R_q-shell
move: realize shell as quotient of O_K
out: arithmetic-parented module object

Step 3
in:  arithmetic-parented torsion-free module
move: discharge Steinitz obstruction / choose genuine coordinates
out: truly free coordinate shell
```

所以 `Step 2` 还没有权利产出“合法坐标”；
它最多只产出：

> “这些对象终于住到了一个能谈 class number / ideals / codifferent 的地方。”

而 `Step 3` 才第一次真正产出：

> “现在写成 \(R_q^d\) 已经不只是看起来像坐标，而是合法坐标。”

只要你把这两步都写成 I/O，就不容易再把它们误听成同一个动作。

**为什么 Step 4 到 Step 6 最该盯住 out-object**

后半段最危险的不是 `in`，而是你太快相信了错误的 `out`。

尤其是：

1. `Step 4` 的 `out` 不是 “同一对象换个几何眼镜”，而是
   “带有真正 Euclidean shortness / Gaussian meaning 的几何对象”；
2. `Step 5` 的 `out` 不是 “把 dual 也顺便提了一下”，而是
   “已经和 quotient / pairing 接好的 duality-compatible state”；
3. `Step 6` 的 `out` 不是 “又写回 familiar shell”，而是
   “solver 假设实际承认的 average-case family”。

也就是说，后半段更适合这样读：

```text
ask less:
what sentence did the author just say?

ask more:
what stronger out-object is the author now claiming to have earned?
```

如果这张 `out` 写不清，就说明这一段的 cash-out 还没被你真正跟住。

**给一个真正的 Langlois-Stehle-style route card**

如果你要把 Section 4.2 的一条 reduction prose 压成最短 route card，可以直接抄这一版：

```text
1. R_q-shell
   -> arithmetic realization
   -> O_K/qO_K parent

2. O_K-parented torsion-free module
   -> freeness discharge
   -> genuine free coordinates

3. free coordinates
   -> embedding reinterpretation
   -> geometric shortness semantics

4. geometric arithmetic state
   -> dual / codifferent alignment
   -> duality-compatible quotient state

5. duality-compatible state
   -> oracle cash-out
   -> solver-legal average-case family
```

这张卡最该拿来做的，不是背，而是对照原文逐段查：

> 当前 paragraph 到底在这五条里的哪一箭头上？

只要这个箭头判断错了，后面的 audit、certificate、H=1 status 很可能都会跟着错位。

**最后把这一页压成一句最该带走的话**

Section 4.2 真正要读成 proof，而不是读成 prose，总得落到一句很硬的话上：

> 每一段关键 reduction prose，  
> 你都应该能写出  
> in → move → out。  
> 写不出来，就说明你还没有真正看见  
> 这一步到底合法搬了什么对象。

#### 一张 reduction route sheet：把连续六个 proof moves 压成真正可回带原文的路线页

到这里，Section 4.2 已经有了几层越来越硬的阅读手柄：

1. `paragraph audit`：单段在背什么 burden；
2. `paragraph chain`：相邻段怎样接力 debt；
3. `certificate chain`：每个 proof job 结束时手里多了哪张 certificate；
4. `reduction I/O`：某一步到底把什么对象送进来，又把什么对象送出去。

但读者一旦真的回到 Langlois-Stehle-style reduction prose，还是会差最后一张中间页：

> 不是只盯单段，  
> 也不是只记单张卡，  
> 而是把连续五六个 proof moves 压成一条 route，  
> 明确看到“这一箭头为什么现在才合法”。

这一页的目标 therefore 很单纯：

1. 不伪造论文原句；
2. 不重写整条证明；
3. 只把 Section 4.2 最常见的一条 reduction 路线压成可手抄的 route step → earned state → next debt 页面。

**先给一条最短 route strip**

```text
solver family
  -> arithmetic parent
  -> free coordinates
  -> geometric shortness
  -> dual compatibility
  -> oracle-legal law
```

这六格看起来很短，但它逼着你把两类最容易混在一起的动作拆开：

1. object 真的被合法搬家；
2. object 只是在同一壳上被换了新的 law。

**再给一张真正可回带原文的 route sheet 总表**

| route step | 这一步想拿到什么 | main object before | main object after | 为什么还不能直接跳到下一步 | bounce page | \(h_k^+=1\) | 当前最该盯的 invariant |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1. family fixation | 固定 solver 真正承诺的 average-case family | 只是“某种看起来像 MLWE 的壳” | 精确的 solver-facing shell | 现在只是把题目固定下来，还没有算术宿主 | reduction worksheet | no | oracle-family |
| 2. arithmetic parent | 给壳找到 \(\mathcal{O}_K/q\mathcal{O}_K\) 这一层 arithmetic host | solver-facing shell | 带 arithmetic parent 的 torsion-free object | 有宿主不等于已有合法全局坐标 | reduction I/O notebook | indirect | arithmetic-legitimacy |
| 3. coordinate legitimization | 把“可写成坐标”升级成“合法 free coordinates” | arithmetic-parented torsion-free module | genuinely free coordinates | 坐标合法了，small / Gaussian 语义还没有自动合法 | reduction-certificate notebook | direct | coordinate-legitimacy |
| 4. geometric reinterpretation | 让 shortness / Gaussian 落到 canonical embedding 的欧氏语义 | free coordinates with candidate secrets/errors | 几何语义合法的 Euclidean state | 有了几何语义，还没有处理 dual / pairing / quotient 的兼容性 | distribution-fidelity notebook | no | geometry-of-shortness |
| 5. dual alignment | 把 quotient、trace pairing、dual object、codifferent 接到同一链上 | geometric arithmetic state | duality-compatible proof state | duality 对齐了，solver promise 还没有 cash out 回来 | reduction-invariant notebook | direct | duality-compatibility |
| 6. oracle cash-out | 把前面所有合法性真的兑回 solver 允许调用的 law | duality-compatible state | solver-legal average-case family | 到这里才有资格 honest 地说 “given a solver ...” | reduction-step audit card | indirect | distribution-fidelity |

这张表最重要的不是第一列，而是第四列与第五列一起看：

> **main object after**  
> 决定你这一箭头到底真拿到了什么；  
> **为什么还不能直接跳到下一步**  
> 则强迫你承认这一步还没替 proof 还完所有 debt。

**把四类依赖直接贴到 route 上**

如果你只想记最硬的 dependency map，这页可以再压成下面四句：

1. `Step 1` 与 `Step 2` 主要还是 shell discipline 与 arithmetic hosting；它们看上去像“只是重写记号”，但实际是在把对象送进可以谈 ideals、class number、codifferent 的宿主。
2. `Step 3` 才是 freeness 真正落地的箭头；这里的 \(h_k^+=1\) 不是装饰，而是在帮 proof 消掉 “Steinitz obstruction 还没还” 这层债。
3. `Step 5` 才是 dual/codifferent 真正落地的箭头；这一步不只是多引入一个 fancy noun，而是在保证 quotient、pairing 与 inverse-ideal bookkeeping 仍属于同一对象链。
4. `Step 6` 则主要在保 distribution-fidelity；它最危险的失败方式不是公式写错，而是 “same shell” 被偷换成了 “same oracle law”。

也就是说，route sheet 最该帮你记住的是：

```text
Step 2 != Step 3
Step 4 != Step 6
```

前一组在区分 arithmetic host 与 free coordinates；
后一组在区分 right geometry 与 right solver law。

**下面给一条 source-like but synthetic 的六箭头路线**

下面这六句都不是论文原句。
它们只是故意写成接近 reduction prose 的样子，方便你把 route sheet 真带回原文页边。

1. `synthetic fragment A`

   > We first fix the target average-case Module-LWE family over the standard quotient shell.

   **route reading**
   `trying to obtain:` solver promise 被精确钉死，不再只是 “某种 average-case flavor”。  
   `before:` 模糊的 MLWE-looking target。  
   `after:` 精确的 solver-facing family。  
   `why this is still not enough:` 现在只是定题，还没说明这些样本壳从哪套 arithmetic parent 下降而来。  
   `bounce:` `reduction worksheet`。  
   `H=1:` `no`。  
   `watch:` 不要把 “题目名字写清楚了” 误听成 “算术结构已经合法了”。

2. `synthetic fragment B`

   > Regard this quotient shell as descending from the appropriate \(\mathcal{O}_K\)-module quotient.

   **route reading**
   `trying to obtain:` 对象终于住进能谈 ideals、codifferent、class number 的 arithmetic parent。  
   `before:` solver-facing shell。  
   `after:` arithmetic-parented torsion-free object。  
   `why this is still not enough:` 你现在只知道“住址对了”，还不知道“坐标是否合法”。  
   `bounce:` `reduction I/O notebook`。  
   `H=1:` `indirect`。  
   `watch:` 不要把 “宿主对了” 偷换成 “已经能无代价选标准基”。

3. `synthetic fragment C`

   > Since the relevant module is free in this setting, we may pass to genuine global coordinates.

   **route reading**
   `trying to obtain:` global coordinates 不再只是 convenient notation，而是 proof-legitimate coordinates。  
   `before:` arithmetic-parented torsion-free module。  
   `after:` genuinely free coordinates。  
   `why this is still not enough:` 有合法坐标，还没有把 shortness / Gaussian 送进 canonical embedding 的几何 law。  
   `bounce:` `reduction-certificate notebook`。  
   `H=1:` `direct`。  
   `watch:` 这是 route 上第一处真正不能拿 “只是换个基” 来糊过去的地方。

4. `synthetic fragment D`

   > Under the canonical embedding, the error and secret now carry the intended Euclidean shortness semantics.

   **route reading**
   `trying to obtain:` 把 small、short、Gaussian 从 coefficient-size 直觉升级成欧氏几何语义。  
   `before:` free coordinates with candidate secret/error vectors。  
   `after:` geometric state with honest shortness meaning。  
   `why this is still not enough:` 现在只是几何 law 对了，dual object、trace pairing 与 quotient 兼容性还没审完。  
   `bounce:` `distribution-fidelity notebook`。  
   `H=1:` `no`。  
   `watch:` 不要把 “对象还写成同一个向量壳” 误听成 “shortness 的 law 没变”。

5. `synthetic fragment E`

   > Via the trace pairing and the codifferent, the quotient and the natural dual object align in the required way.

   **route reading**
   `trying to obtain:` dual / codifferent / pairing / quotient 终于在同一条链上接稳。  
   `before:` geometric arithmetic state。  
   `after:` duality-compatible proof state。  
   `why this is still not enough:` 对偶性对齐不等于 solver 已经能 honest 接手；还差最后一次 law cash-out。  
   `bounce:` `reduction-obligation ledger`。  
   `H=1:` `direct`。  
   `watch:` 这一箭头若省掉，后面的 shell-return 往往只剩“长得像”，不再有合法 pairing/dual bookkeeping。

6. `synthetic fragment F`

   > Hence, given a solver for the target Module-LWE family, we obtain the desired contradiction / break.

   **route reading**
   `trying to obtain:` 把前面所有 host、coordinate、geometry、dual 的合法性全部兑成 solver 真承认的 average-case law。  
   `before:` duality-compatible state。  
   `after:` solver-legal family。  
   `why this is finally enough:` 到这里才能 honest 地调用 solver，因为 shell、law、distribution promise 才第一次同时对齐。  
   `bounce:` `reduction-step audit card`。  
   `H=1:` `indirect`。  
   `watch:` 这一步最危险的错觉是 “公式已经长回 familiar shell，所以 solver 自动可用”。

**为什么这张 route sheet 比单张 I/O 卡更硬**

单张 I/O 卡的重点是：

> 这一段里面，  
> **in -> move -> out**  
> 到底怎么走。

而 route sheet 逼你多做一步：

> 这一箭头结束后，  
> 为什么还不能假装后面那一箭头也已经免费完成。

这正是 Section 4.2 最容易被读滑过去的地方。
因为很多 prose 表面上都像：

1. “对象又被解释了一次”；
2. “然后作者接着说了下一句”；
3. “所以我猜这两句差不多是一件事”。

route sheet 专门反过来要求你写：

1. 这一箭头刚刚还掉的是哪一层 debt；
2. 下一箭头还没还的是哪一层 debt；
3. 这两层 debt 为什么不能偷并。

**给读者一张最短可手抄的 route card**

如果你真的要把这一页带回 paper 页边，最短可以只抄下面七格：

```text
route step:
trying to obtain:
before:
after:
not-yet:
H=1:
main invariant:
```

这七格卡最适合拿来处理那种“连续三四段看起来都在说 reduction 正在前进”的 prose。
用法非常简单：

1. 先判当前段在 route 上属于哪一箭头；
2. 再写一句 `after:`；
3. 最后强迫自己补一句 `not-yet:`。

只要 `not-yet:` 写不出来，你大概率还是把两箭头偷并了。

**最后把这一页压成一句最该带走的话**

Section 4.2 最危险的误读，不是没看见术语，而是把：

> “对象已经有了合适的 shell”

误听成：

> “对象已经有了后面那一箭头所需的 law”。

route sheet 的作用，就是逼你在每一箭头都写清楚：

> 这一箭头刚刚还掉的是 host、coordinates、geometry、dual，  
> 还是 distribution 这一层 debt。

#### 一张 route-to-paragraph margin sheet：把连续六段 reduction prose 直接钉回 route arrow

到这里，Section 4.2 已经有了：

1. `paragraph audit`：看单段在背什么 burden；
2. `paragraph chain`：看 debt 怎样段与段接力；
3. `certificate notebook`：看 proof 到这里究竟多了哪张证书；
4. `reduction I/O notebook`：看对象怎样 in → move → out；
5. `reduction route sheet`：看整条 proof 被压成哪几根箭头。

但真正回到原论文页边时，读者还会差最后一张更贴近手感的页面：

> 不只是知道有六根箭头，  
> 而是面对一段具体 prose 时，  
> 能立刻判断  
> “它究竟属于上一箭头、这一箭头，还是下一箭头”。

这一页 therefore 只做一件事：

> 把连续六段 source-like but synthetic 的 reduction prose  
> 逐段钉回 route sheet，  
> 并强迫你给每段写一句  
> `why not previous / why not next`。

因为这正是 Section 4.2 最常见、也最伤理解的一种误读：

> 读者不是完全没看懂这一段，  
> 而是把它误归到了相邻那一箭头上。

**先给一张六段 strip 的最短标签图**

| synthetic paragraph | route arrow | 最短标签 | 这一段真正承重的是什么 |
| --- | --- | --- | --- |
| A | 1 | coefficient-shell presentation | 把 solver-facing family 精确钉死 |
| B | 2 | arithmetic hosting | 给壳补上 \(\mathcal{O}_K\) 母对象 |
| C | 3 | freeness discharge | 把“可写成坐标”升级成“合法 free coordinates” |
| D | 4 | geometry/law transport | 把 shortness / Gaussian 送进正确几何 |
| E | 5 | dual/codifferent glue | 让 quotient、pairing、natural dual 接到同一链上 |
| F | 6 | distribution-fidelity cash-out | 把整条结构链真的兑回 solver promise |

这张表真正想训练的动作是：

1. 先判断当前 paragraph 在哪一箭头上；
2. 再判断它为什么还没有资格算成下一箭头。

**下面给一条连续六段的 source-like strip**

下面六段都不是论文原句。
它们只是故意写得足够像 reduction prose，好让你把前面的 route sheet 真带回页边。

1. `synthetic paragraph A`

   > Let the target average-case Module-LWE instances be written over the standard quotient shell, and regard the solver as acting on this precise family.

   **margin audit**
   `route arrow:` `1. family fixation`。  
   `why not previous:` 这是 strip 起点，前面还没有更早箭头可消费。  
   `why not next:` 这里只把题目钉死，还没有给这些 samples 找到 arithmetic parent。  
   `literal:` 精确固定 solver-facing family。  
   `hidden import:` 后面所有 object transport 都必须最终回到这个 family，而不是回到某个长得像它的壳。  
   `H=1:` `no`。  
   `transport:` `object`。  
   `failure mode:` `wrong target family`。  
   `bounce:` `reduction worksheet`。

2. `synthetic paragraph B`

   > We then regard this shell as descending from the relevant \(\mathcal{O}_K\)-module quotient, so that ideals, modules, and congruence data share one arithmetic parent.

   **margin audit**
   `route arrow:` `2. arithmetic parent`。  
   `why not previous:` 这里已经不只是 naming the family，而是在给对象补上真正的 \(\mathcal{O}_K\) 宿主。  
   `why not next:` 有 arithmetic host 还不等于已经拿到 genuinely free coordinates。  
   `literal:` 把 solver-facing shell 落回共同的 arithmetic parent。  
   `hidden import:` 后面谈 ideals、class number、codifferent 时，不会出现“对象其实并不住在同一宿主里”的漂浮问题。  
   `H=1:` `indirect`。  
   `transport:` `object`。  
   `failure mode:` `arithmetic parent floats`。  
   `bounce:` `reduction I/O notebook`。

3. `synthetic paragraph C`

   > Since the torsion-free module is free in the class-number-one setting, we may pass to genuine global coordinates and work in a standard free \(R_q\)-module model.

   **margin audit**
   `route arrow:` `3. coordinate legitimization`。  
   `why not previous:` 这段已经在 discharge freeness obstruction，而不是只说“对象现在住在对的地方”。  
   `why not next:` 坐标合法化之后，shortness / Gaussian 语义仍未自动搬进 canonical embedding。  
   `literal:` 把 arithmetic-parented module 合法化成 genuine free coordinates。  
   `hidden import:` 当前使用的 basis 不再暗藏 ideal-class twist；写成 \(R_q^d\) 现在是 proof-legal，而不只是 notation-convenient。  
   `H=1:` `direct`。  
   `transport:` `object`。  
   `failure mode:` `fake coordinates`。  
   `bounce:` `reduction-certificate notebook`。

4. `synthetic paragraph D`

   > Under the canonical embedding, the secret and error are now read in the Euclidean geometry where shortness and Gaussian behavior carry their intended meaning.

   **margin audit**
   `route arrow:` `4. geometric reinterpretation`。  
   `why not previous:` 这里真正新得到的不是 coordinates，而是 shortness / Gaussian 的几何 law。  
   `why not next:` 几何语义对了，并不等于 quotient、pairing、natural dual 已经兼容。  
   `literal:` 把 shortness 与 Gaussian semantics 从 coefficient shell 抬进 Euclidean geometry。  
   `hidden import:` 需要保住的是 distribution meaning，而不只是 object identity。  
   `H=1:` `no`。  
   `transport:` `law`。  
   `failure mode:` `geometry/law drift`。  
   `bounce:` `distribution-fidelity notebook`。

5. `synthetic paragraph E`

   > Via the trace pairing and the codifferent, the quotient description aligns with the natural dual object required by the reduction.

   **margin audit**
   `route arrow:` `5. dual alignment`。  
   `why not previous:` 这段不再只谈几何语义，而是在把 quotient、pairing、dual object 粘回同一条 proof line。  
   `why not next:` duality glue 站住了，solver promise 还没有 cash out 回 oracle-facing law。  
   `literal:` 对齐 quotient structure 与 natural dual object。  
   `hidden import:` inverse-ideal / codifferent bookkeeping 现在是显式、自然、可继续消费的，而不是散落在不同语言里。  
   `H=1:` `direct`。  
   `transport:` `object`。  
   `failure mode:` `duality mismatch`。  
   `bounce:` `reduction-obligation ledger`。

6. `synthetic paragraph F`

   > Hence the resulting structured state can be returned to the target Module-LWE oracle family, so that the solver hypothesis applies to the promised law rather than merely to a shell with familiar syntax.

   **margin audit**
   `route arrow:` `6. oracle cash-out`。  
   `why not previous:` 这里只在做最后一次 solver-facing cash-out，而不是继续整理 dual/codifferent glue。  
   `why not next:` 这已经是 strip 终点；后面没有更强箭头可再消费。  
   `literal:` 把前面的 structured state 兑回 target oracle family。  
   `hidden import:` 回来的必须同时是 `same shell` 与 `same law`，否则 solver 只对外观负责，不对 distribution promise 负责。  
   `H=1:` `indirect`。  
   `transport:` `both`。  
   `failure mode:` `oracle mismatch`。  
   `bounce:` `reduction-step audit card`。

**为什么这张页边表最该盯住 why-not-previous / why-not-next**

如果只写 `literal / hidden import / H=1 / transport`，
你仍然可能会把相邻两箭头偷并。
真正更硬的动作，是强迫自己写出：

1. 这段为什么已经比上一箭头更强；
2. 这段为什么还没有强到下一箭头。

这两句一旦写出来，Section 4.2 里最容易混掉的三组边界会立刻清楚很多：

1. A → B：不是 “同一个壳换更文雅的记号”，而是从 target naming 真切切入 arithmetic hosting。
2. B → C：不是 “有宿主就自然有标准坐标”，而是必须再过一次 freeness discharge。
3. D → E → F：不是 “都属于抽象格几何后半段”，而是 geometry、dual glue、oracle cash-out 三次不同 burden。

**把四类最想分开的 burden 再压成一句话**

如果你只想把最硬的四个判断背下来，这页其实只需要记下面四句：

1. `A` 最像 `coefficient-shell presentation`；它在定 solver 目标，不在谈 arithmetic legality。
2. `C` 才是 freeness 真正直接落地的位置；这一段若站不住，后面的 \(R_q^d\) 只是外观坐标。
3. `E` 才是 dual/codifferent 真正直接落地的位置；它不是 decorative abstraction，而是 proof glue。
4. `F` 才是 distribution-fidelity 的最终 cash-out；它要保的不是“还能写回同一公式”，而是“还能回到 solver 真承认的 law”。

因此教材化阅读时，最常见的三种错位刚好是：

```text
arithmetic host
    !=
free coordinates

right geometry
    !=
duality already aligned

same shell
    !=
same oracle law
```

**给读者一张真正可抄到页边的最短卡**

以后你看到一段像 Section 4.2 的 reduction prose，最短可以只写下面八格：

```text
route arrow:
why not previous:
why not next:
literal:
hidden import:
H=1:
transport:
failure mode:
```

这张卡比 earlier six-box audit 更靠近 proof reading。
因为它不是只问：

> “这段说了什么？”

而是问：

> “这段在整条 route 上到底站在哪一箭头，  
> 它为什么还不是前一箭头，也还不是后一箭头？”

只要这一步能做对，很多看似“差不多”的 prose 段落就会被你自动拆开。

**最后把这一页压成一句最该带走的话**

Section 4.2 最难的地方，往往不是你完全看不懂某段，而是你把它放错了箭头。
所以真正最值得训练的页边动作不是复述句子，而是立刻写出：

> **why not previous**  
> 与  
> `why not next`。

一旦这两句写清，freeness、dual/codifferent、coefficient-shell presentation 与 distribution-fidelity 就不那么容易再在脑子里塌成一层。

#### 一张 four-burden overlay sheet：把同一条 reduction strip 按四种真正不同的 burden 重新着色

到这里，读者已经能做三件比最初硬很多的事：

1. 看出一段 prose 属于哪一根 route arrow；
2. 写出它为什么不是前一箭头、也还不是后一箭头；
3. 粗分它是在搬 object、law，还是 both。

但 Section 4.2 还有最后一种特别容易残留的糊感：

> 虽然知道这六段不是同一箭头，  
> 但还是会把它们都统称成  
> “在做 reduction 的技术整理”。

这会导致一个更深的误读：

> freeness、dual/codifferent、coefficient-shell presentation、distribution-fidelity  
> 明明是四种很不一样的 burden，  
> 读者脑中却仍把它们听成  
> “后半段那些抽象操作”。

这一页 therefore 的目标非常窄，也非常硬：

> 不再按箭头顺序看，  
> 而是把同一条六段 strip  
> 用四种 burden 重新着色；  
> 强迫自己承认  
> 每一段到底主做哪一类工作，  
> 又绝对不是哪一类工作。

**先把四种 burden 单独命名**

这一页只追四种最容易被混成一锅的 burden：

1. `coefficient-shell presentation`
   重点不是证明新结构，而是把 solver-facing family、quotient shell、notation boundary 钉清楚。
2. `freeness discharge`
   重点是把 “可写成坐标” 升级成 “合法 free coordinates”，真正清掉 hidden ideal-class twist。
3. `dual/codifferent glue`
   重点是让 quotient、pairing、natural dual、codifferent 落回同一条对象链。
4. `distribution-fidelity cash-out`
   重点是保证回到 solver 时，回去的不只是 familiar shell，而是真正被假设覆盖的 average-case law。

这四种 burden 之所以值得单列，是因为它们分别回答四个完全不同的问题：

```text
presentation: 我们到底在解哪个 family？
freeness:     这些坐标到底是否合法？
duality:      pairing / quotient / natural dual 还能不能接上？
fidelity:     回到 solver 时 law 还在不在？
```

**先给一张六段 overlay 总表**

| paragraph / arrow | coefficient-shell presentation | freeness discharge | dual/codifferent glue | distribution-fidelity cash-out | 如果这段被涂错色，最容易坏在哪里 |
| --- | --- | --- | --- | --- | --- |
| A / 1 | main | no | no | secondary | 把 target family 说糊，后面会变成 “反正某种 MLWE” |
| B / 2 | secondary | no | secondary | no | 把 arithmetic host 误听成 freeness，后面会提前偷到假坐标 |
| C / 3 | secondary | main | no | no | 把合法坐标误听成 notation convenience，整个 \(R_q^d\) 线性代数会失根 |
| D / 4 | no | no | secondary | main-prep | 把几何 law 误听成 object rewrite，error semantics 会悄悄飘掉 |
| E / 5 | no | secondary | main | secondary | 把 dual glue 当成装饰抽象，最后 cash-out 只剩“壳像”，不再有合法 proof chain |
| F / 6 | secondary | no | secondary | main | 把 familiar shell 误听成 promised law，solver 调用会失去假设覆盖 |

这里最重要的不是 `main` 那一列，而是：

> 同一段虽然可能会轻触别的 burden，  
> 但真正的主 burden 只有一个。

教材式阅读若不逼自己给每段选一个 `main burden`，
就会重新滑回那种：

> “这里也有 freeness、那里也有 codifferent、最后也有 solver，  
> 所以后半段大概都差不多。”

**把六段逐一重着色**

下面不再重复整段 prose，
而是只解释为什么每段该被涂成这个颜色，而不是另一个颜色。

1. `A / family fixation` 为什么是 `presentation-main`

   这一段真正承担的是：

   > 让 solver-facing family 精确落地。

   它当然会轻微碰到后面的 fidelity 问题，
   因为后面一切都得回到这个 target family；
   但这还不是 `distribution-fidelity cash-out` 本身。

   所以最准确的页边句不是：

   > “这里已经在保 distribution”

   而是：

   > “这里先把 distribution promise 的名字钉死，  
   > 但还没有兑现它。”

2. `B / arithmetic parent` 为什么不是 freeness

   这一段最常见的错色，是被涂成 `freeness-ish`。
   但它真正做的只是：

   > 给对象找共同的 arithmetic host。

   也就是说，它解决的是：

   > ideals、modules、congruence data 到底住在哪。

   它还没有解决：

   > 既然住在这里，为什么这些对象就能合法写成 genuinely free coordinates。

   所以 `B` 最值得写的反误读句是：

   ```text
   host gained
       !=
   free coordinates earned
   ```

3. `C / freeness discharge` 为什么必须单独染成主色

   `C` 是整条 strip 里最该毫不含糊写成 `freeness-main` 的一段。
   因为它第一次真正把：

   > “这些对象可以在纸面上写成坐标”

   推进到：

   > “这些坐标是 proof-legitimate 的。”

   教材化读法里，这段最不该被涂成：

   1. `presentation`
   2. `duality`
   3. `generic algebra cleanup`

   它真正清掉的是一笔非常具体的债：

   > hidden ideal-class twist 不能继续留在 basis 选择后面。

4. `D / geometric reinterpretation` 为什么最该染成 `fidelity-prep`

   `D` 表面上不像 `cash-out`，
   但它已经是 `distribution-fidelity` 的第一处真正主战场。
   因为这一步开始处理的不是：

   > 对象长什么样，

   而是：

   > “small / short / Gaussian” 这些 law 到底住在哪套语义里。

   所以 `D` 最容易被误涂成 `object rewrite`，
   但更准确的理解其实是：

   > 这一步还没把 law 送回 solver，  
   > 可它已经在为最后的 law-preserving cash-out 做最关键的前置合法化。

5. `E / dual alignment` 为什么不是“顺手加一层术语”

   `E` 是最容易被弱背景读者涂错的一段之一。
   因为一旦看到 trace pairing、codifferent、dual object，
   很多人会自动以为：

   > “作者只是把 embedding 后的对象再写得更高级一点。”

   但这页要强迫读者改成下面这句：

   > `E` 不是 decorative abstraction，  
   > 而是 glue burden。

   它真正在修的是：

   > quotient / pairing / natural dual 是否还能在同一条 reduction chain 里继续互相消费。

   如果这一步不被染成 `duality-main`，
   最后 `F` 就会被误读成：

   > “反正对象最后又写回 familiar shell 了。”

6. `F / oracle cash-out` 为什么必须是 `fidelity-main`

   `F` 是整条 strip 里最不该含糊的 `distribution-fidelity` 主色。
   因为这里真正被 claim 的不是：

   > “对象还能写回 \(R_q^d\) 的外观。”

   而是：

   > “对象与 law 一起回到了 solver 假设真正承诺的 average-case family。”

   所以 `F` 若被涂成 `presentation-main`，
   读者就会把最后的关口降格成：

   > “公式看起来终于像 MLWE 了。”

   这正好错过整条 proof 最危险的一次合法性 cash-out。

**再给一张真正可手抄的错误重着色表**

如果你想更快定位自己在什么地方最容易把段落涂错色，
可以直接用下面这张错色表：

| 当前 paragraph | 最常见的错误重着色 | 为什么错 |
| --- | --- | --- |
| B | 染成 freeness-main | host 只解决宿主，不解决合法 coordinates |
| C | 染成 presentation-main | 这里不是换 notation，而是在清 Steinitz/ideal-class 债 |
| D | 染成 object-main | 这里主修的是 shortness / Gaussian 的 law，不是壳的外观 |
| E | 染成 geometry-main | 这里主修的是 glue，不是继续描述几何 |
| F | 染成 presentation-main | familiar shell 不是 solver law；最终关口在 fidelity，不在外观 |

这张表的作用很直接：

> 一旦你发现自己在脑中给某段涂了错误颜色，  
> 就要立刻回查  
> “这段现在到底在还哪一种 debt”。

**把这页压成一张最短 overlay card**

以后你遇到一段 Section 4.2 式的 prose，
除了前面的 route-card 以外，
还可以再补一张四格 overlay：

```text
main burden:
secondary burden:
definitely not:
if miscolored, next break:
```

这张卡专门防一种比“没看懂”更隐蔽的问题：

> 你其实看懂了句子，  
> 但把它放进了错误的 burden 抽屉。

一旦抽屉放错，
后面整条 proof 的 reading order 也会跟着歪掉。

**最后把这一页压成一句最该带走的话**

Section 4.2 真正难的地方，不只是段落太密，
而是不同 paragraph 正在偿还的 proof debt 根本不是同一种。
所以更成熟的读法，不只是问：

> “这一段属于哪一箭头？”

还要继续问：

> “这一段主修的到底是 presentation、freeness、duality，  
> 还是 distribution-fidelity？”

只要主 burden 染对色，整条 reduction 的骨架就会清楚很多。

#### 一张 worked reduction-by-reduction audit page：把同一条 A-F strip 真的压成 proof ledger

到这里，Section 4.2 已经不缺“卡片类型”了。
它现在真正缺的是：

> 把前面这些卡片  
> 在同一条六段 strip 上  
> 一次性真正填完。

因为很多读者到这一步仍会停在：

1. 我知道有哪些 notebook；
2. 我也知道每张卡大概怎么用；
3. 但我还没有看过一条连续 reduction prose 被完整批成一页 proof ledger。

这一页 therefore 的目标非常明确：

> 不再新增新术语；  
> 也不再只给空白模板；  
> 而是把前面的 A-F 六段 strip  
> 真正压成一张 worked audit page。

这张页最重要的训练动作不是“复述段意”，而是同时写出：

1. 这段接收的 proof state；
2. 这段交出的 proof state；
3. 这段主还的是哪一种 burden；
4. 如果这段失效，后面第一处哪句话会先失去合法性。

**先给一张六段 proof-ledger 总表**

| paragraph | route arrow | incoming proof state | outgoing proof state | main burden | \(h_k^+=1\) | if false, first downstream collapse |
| --- | --- | --- | --- | --- | --- | --- |
| A | 1 | 只知道“要解某种 Module-LWE-looking family” | 精确 solver-facing family 被钉死 | coefficient-shell presentation | no | B 里 “this shell descends from ...” 会先变成无目标漂浮句 |
| B | 2 | 有了 target shell，但还没有 arithmetic host | objects now share one arithmetic parent | arithmetic hosting | indirect | C 里 “we may pass to genuine global coordinates” 会先失去落脚点 |
| C | 3 | arithmetic-parented torsion-free module | proof-legitimate free coordinates | freeness discharge | direct | D 里所有 embedding / Euclidean semantics 都只会作用在假坐标上 |
| D | 4 | legal free coordinates carrying secret/error objects | law-bearing geometric state | distribution-fidelity prep | no | E/F 里谈 solver-relevant shortness 时会只剩 object shell、没有 honest law |
| E | 5 | geometric state already carrying intended law | duality-compatible reduction state | dual/codifferent glue | direct | F 里最后一次 cash-out 会退化成“壳像 MLWE”，不再是合法 reduction return |
| F | 6 | duality-compatible state still waiting to be cashed out | solver-legal average-case family | distribution-fidelity cash-out | indirect | 整条 proof 不能 honest 调 solver，只能停在“syntax looks right” |

这张总表最该盯的不是第二列，而是最后一列：

> 如果这一段站不住，  
> 后面第一处  
> 哪一句就不再有资格被说出来？

这会强迫你把 paragraph 理解成 proof dependency，
而不是 prose summary。

**下面把六段真的逐段填完**

1. `Paragraph A`

   `incoming state:` 你只有一个很宽的目标：后面某处想调用某个 Module-LWE solver。  
   `paragraph job:` 把这个宽目标压成精确的 solver-facing family。  
   `outgoing state:` 现在后文终于可以明确区分 “target family” 与 “只是看起来像它的壳”。  
   `main burden:` `coefficient-shell presentation`。  
   `secondary burden:` future oracle-boundary discipline。  
   `H=1:` `no`。  
   `transport:` `object`。  
   `why not previous:` 这是 strip 起点，没有更早 proof state 可消费。  
   `why not next:` 现在只是定题，还没给 objects 落到共同的 arithmetic host。  
   `first downstream collapse if false:` `Paragraph B` 里 “descends from the relevant \(\mathcal{O}_K\)-module quotient” 会先失去 target anchor。  
   `bounce:` `reduction worksheet` + `reduction route sheet`。

2. `Paragraph B`

   `incoming state:` target family 已被钉死，但它还只是 solver-facing shell。  
   `paragraph job:` 把 shell 落回 \(\mathcal{O}_K\) 宿主，使 ideals / modules / congruence data 真住在同一 arithmetic parent。  
   `outgoing state:` 现在你第一次有资格认真谈 class number、codifferent、duals 的宿主合法性。  
   `main burden:` `arithmetic hosting`。  
   `secondary burden:` future dual-language admissibility。  
   `H=1:` `indirect`。  
   `transport:` `object`。  
   `why not previous:` 这不再只是 naming the family；它已经在补共同宿主。  
   `why not next:` 有 host 还没有 legal free coordinates。  
   `first downstream collapse if false:` `Paragraph C` 里 “pass to genuine global coordinates” 会先失去对象宿主，变成无根的 basis language。  
   `bounce:` `reduction I/O notebook` + `paragraph-chain notebook`。

3. `Paragraph C`

   `incoming state:` objects 终于有了共同 arithmetic host，但还只是 torsion-free module。  
   `paragraph job:` 清掉 hidden ideal-class twist，把对象合法化成 genuine free coordinates。  
   `outgoing state:` 写成 \(R_q^d\) 现在不只是 convenient shell，而是 proof-legitimate coordinates。  
   `main burden:` `freeness discharge`。  
   `secondary burden:` later linear-algebra legitimacy。  
   `H=1:` `direct`。  
   `transport:` `object`。  
   `why not previous:` 这一段已经在还 Steinitz / ideal-class 债，而不是只说宿主是谁。  
   `why not next:` 坐标合法了，但 shortness / Gaussian 还没有被送进几何 law。  
   `first downstream collapse if false:` `Paragraph D` 里任何 canonical-embedding shortness claim 都会先退化成“只是在假坐标上做几何解释”。  
   `bounce:` `reduction-certificate notebook` + `four-burden overlay sheet`。

4. `Paragraph D`

   `incoming state:` legal free coordinates 已经到手，但 law 仍停留在 coefficient intuition。  
   `paragraph job:` 把 shortness、smallness、Gaussian behavior 真正抬进 canonical embedding 的 Euclidean semantics。  
   `outgoing state:` 现在后文终于可以消费一个带 honest geometric law 的 state，而不只是带向量外观的 state。  
   `main burden:` `distribution-fidelity prep`。  
   `secondary burden:` future duality compatibility。  
   `H=1:` `no`。  
   `transport:` `law`。  
   `why not previous:` 这里新得到的是 law-bearing geometry，不是再一次 coordinate cleanup。  
   `why not next:` 几何 law 对了，仍未说明 quotient、pairing、natural dual 能和它继续接轨。  
   `first downstream collapse if false:` `Paragraph E` 里任何 “required by the reduction” duality glue 都会变成 glue 到一个没有 honest shortness law 的空壳上。  
   `bounce:` `distribution-fidelity notebook` + `route-to-paragraph margin sheet`。

5. `Paragraph E`

   `incoming state:` 已有 honest geometric law，但 objects 之间的 dual / quotient / pairing 接口仍可能散。  
   `paragraph job:` 把 codifferent、trace pairing、natural dual、quotient structure 粘回同一条 reduction chain。  
   `outgoing state:` 现在你拿到的是一个可继续 cash out 的 duality-compatible state。  
   `main burden:` `dual/codifferent glue`。  
   `secondary burden:` final solver-facing admissibility。  
   `H=1:` `direct`。  
   `transport:` `object`。  
   `why not previous:` 这里不再只是保护 law，而是在修对象接口之间的 glue。  
   `why not next:` dual glue 站住了，还没有把这整套 state honest 兑回 solver promise。  
   `first downstream collapse if false:` `Paragraph F` 里 “returned to the target Module-LWE oracle family” 会先退化成 syntax return，而不是 reduction-legal return。  
   `bounce:` `reduction-obligation ledger` + `four-burden overlay sheet`。

6. `Paragraph F`

   `incoming state:` 你终于有了 duality-compatible state，但它还没回到 solver 假设亲自承认的 law。  
   `paragraph job:` 把前面的 host、coordinates、geometry、duality 一起兑回 target average-case family。  
   `outgoing state:` 这时才第一次拿到 honest solver-legal family。  
   `main burden:` `distribution-fidelity cash-out`。  
   `secondary burden:` shell-recognition discipline。  
   `H=1:` `indirect`。  
   `transport:` `both`。  
   `why not previous:` 这一步不是继续修 glue，而是在做最后一次 oracle-facing cash-out。  
   `why not next:` 这是 strip 终点；后面没有更强状态可再争取。  
   `first downstream collapse if false:` 整条 reduction 无法 legitimate 地写出 “given a solver ...”；证明只能停在 “shape looks MLWE-like”。  
   `bounce:` `reduction-step audit card` + `distribution-fidelity notebook`。

**为什么这张 worked page 比前面所有卡更接近真正读 proof**

前面的卡片各自都在回答一个局部问题：

1. `paragraph audit` 说这段在背什么 burden；
2. `I/O notebook` 说对象怎样合法搬家；
3. `route sheet` 说这段处在哪根箭头上；
4. `overlay sheet` 说这段主修哪一类 debt。

而这张 worked page 第一次强迫你把它们合成一句真正 proof-reader 会问的话：

> “如果这一段失效，  
> 后面第一句  
> 哪一句就会立刻失去资格？”

这一步一旦能做出来，
你就不再只是在“理解内容”，
而是在真正地检查 proof dependency。

**给读者一张最短可带回原文页边的 worked-ledger 卡**

如果你要把这种 worked audit 压成最短手抄版，
只需保留下面六格：

```text
incoming state:
outgoing state:
main burden:
why not next:
first downstream collapse if false:
bounce:
```

这六格已经足够把一段 reduction prose
从“可复述”推进到“可审计”。

**最后把这一页压成一句最该带走的话**

Section 4.2 真正成熟的读法，不只是能说出：

> “这一段在谈 freeness / duality / solver。”

而是能继续写出：

> “这一段到底交出了什么新的 proof state；  
> 如果它没交出来，  
> 后面第一句会先在哪里塌掉。”

只要这一步开始稳定做到，读者就真的开始在跟 proof，而不只是跟 prose。

#### 一个 sentence-surgery notebook：不要让一条长句把 host、freeness、law、cash-out 一起偷带过去

到这里，Section 4.2 的阅读困难已经从“我看不懂这些名词”变成了更细的一种困难：

> 我大概知道这一整段在干什么，  
> 但一条长句里到底哪一半是在补合法性，  
> 哪一半才真的在把 proof 往前推，  
> 我还会混。

这正是很多数学论文 reduction prose 的常见压缩方式：

1. 先用半句把对象重新放到对的宿主里；
2. 再用半句借出某个合法性；
3. 最后才用一句 `thus / hence / therefore` 做真正的 cash-out。

如果把整条句子一口气吞掉，读者最容易犯的错不是完全没看懂，
而是把：

> legalizer clause

误听成：

> already earned next proof state.

所以这一页的目标很具体：

> 不再按自然段切，  
> 而是按 clause 切；  
> 让你看到一条长句里  
> 哪个子句只是在补 host / freeness / glue，  
> 哪个子句才真的把下一层 state 领出来。

**先给一张 clause-role 小表**

Section 4.2 风格的一条长句，最常见会混着下面四类 clause：

| clause role | 最常见触发词 | 真正作用 | 最容易被误听成什么 |
| --- | --- | --- | --- |
| host clause | regard as, descends from, view as a quotient of ... | 给对象补共同宿主 | “已经有合法 coordinates 了” |
| freeness / legality clause | since ... is free, we may choose coordinates, identify with ... | 给坐标或对象语言补合法性 | “已经进入下一层几何/solver 语义了” |
| law clause | under the canonical embedding, Gaussian behavior, shortness is read ... | 给 distribution / geometry 补 honest semantics | “对象没变，所以这只是重写” |
| cash-out clause | hence, thus given a solver, therefore the target family ... | 把先前积攒的合法性真正兑回接口 | “只是收尾重述一下而已” |

这张小表真正想防的是：

> 一条句子里有三个 clause，  
> 但你脑中只留下一个模糊结论。

**下面给三条真正需要 sentence surgery 的 synthetic sentences**

它们都不是论文原句。
它们只是故意写成接近 Section 4.2 手感的样子，
用来训练“同一句里不同 clause 承担不同 proof debt”这件事。

1. `synthetic long sentence A`

   > Viewing the target shell as descending from the relevant \(\mathcal{O}_K\)-module quotient, and using the class-number-one freeness of the resulting torsion-free module, we may work in genuine global coordinates over a standard free \(R_q\)-module.

   **clause surgery**

   | clause | visible text | route / burden | 真正领出的 state | H=1 | 若在这里就信过头，会偷成什么 |
   | --- | --- | --- | --- | --- | --- |
   | A1 | `Viewing the target shell as descending from ... quotient` | `B / arithmetic hosting` | objects now share one arithmetic parent | `indirect` | “宿主对了，所以坐标也已经合法了” |
   | A2 | `using the class-number-one freeness ...` | `C / freeness discharge` | hidden ideal-class twist can be discharged | `direct` | “提了一句 freeness，只是背景说明” |
   | A3 | `we may work in genuine global coordinates ...` | `C / outgoing state` | proof-legitimate free coordinates | consumes A1+A2 | “这只是 notation choice，不是新 state” |

   **most important cut**

   ```text
   host clause
       !=
   freeness clause
       !=
   coordinates already earned
   ```

   这条句子最危险的地方，不是最后半句，而是中间那个
   **using ... freeness**
   很容易被读者当成“顺口提一下假设”。
   但它其实正是让最后一句
   **we may work in genuine global coordinates**
   合法的那张许可证。

2. `synthetic long sentence B`

   > Under the canonical embedding the error distribution acquires its intended Euclidean shortness semantics, and via the trace pairing and the codifferent the resulting state remains aligned with the natural dual object required by the reduction.

   **clause surgery**

   | clause | visible text | route / burden | 真正领出的 state | H=1 | 若在这里就信过头，会偷成什么 |
   | --- | --- | --- | --- | --- | --- |
   | B1 | `Under the canonical embedding ...` | `D / law transport` | we are now reading law in the right geometry | `no` | “只是对象换了眼镜，law 没有新内容” |
   | B2 | `the error distribution acquires ... shortness semantics` | `D / outgoing law state` | honest geometric law becomes available | `no` | “还只是同一个 vector shell” |
   | B3 | `via the trace pairing and the codifferent ... remains aligned` | `E / dual glue` | duality-compatible state | `direct` on the glue side | “既然几何语义对了，dual glue 当然顺便也对了” |

   **most important cut**

   ```text
   right geometry
       !=
   duality already aligned
   ```

   这条句子最容易被误读成“一整句都在继续谈 embedding geometry”。
   实际上它横跨了两根不同箭头：

   1. 先把 law 放到 honest geometry；
   2. 再把 dual / codifferent glue 接上。

3. `synthetic long sentence C`

   > Hence, once the dual-compatible state has been returned to the standard shell, the target Module-LWE solver applies to the promised average-case family.

   **clause surgery**

   | clause | visible text | route / burden | 真正领出的 state | H=1 | 若在这里就信过头，会偷成什么 |
   | --- | --- | --- | --- | --- | --- |
   | C1 | `once the dual-compatible state has been returned to the standard shell` | `F / shell-facing pre-cash-out` | familiar syntax shell is back | `indirect` | “回到 familiar shell 就已经等于回到 solver family” |
   | C2 | `the target Module-LWE solver applies ...` | `F / actual cash-out` | solver-legal average-case family | consumes C1 + earlier fidelity proofs | `indirect` | “solver apply” 只是收尾重述，不是最危险 claim |

   **most important cut**

   ```text
   same shell
       !=
   solver may already apply
   ```

   这条句子里最需要下刀的地方，
   就是把
   **returned to the standard shell**
   和
   **solver applies to the promised average-case family**
   分开。
   前者只是 syntax-side recovery；
   后者才是 law-side cash-out。

**再给一张真正可回带原文页边的 sentence-surgery 总表**

| long sentence | 你最该下刀的切口 | 如果不切，最容易塌成什么一句错话 |
| --- | --- | --- |
| A | host → freeness → coordinates | “重新解释对象并顺便选了基” |
| B | law-bearing geometry → dual glue | “继续在讲 embedding，所以后半句只是术语展开” |
| C | shell comes back → solver may apply | “公式长回 MLWE，所以 solver 当然能调” |

这张总表真正要训练的，是一种很具体的页边习惯：

> 每看到一条长句，  
> 不先问“整句大意是什么”，  
> 而先问“这里最危险的 cut 在哪”。

**给读者一张最短可手抄的 clause card**

如果你要把这页方法压成最短的手抄版，只需写下面五格：

```text
clause:
what it legalizes:
what it does NOT yet earn:
which later clause consumes it:
if merged away, overclaim becomes:
```

这五格特别适合拿来处理那种
`since ...`, `under ...`, `via ...`, `hence ...`
挤在同一句里的 paragraph。

**最后把这一页压成一句最该带走的话**

Section 4.2 更接近真实 paper prose 的困难，常常不是整段太深，
而是一条长句里混着三种不同 proof debt。
所以真正成熟的读法，不只是能给整段打 tag，
而是能继续把一句话切开，明确说出：

> 哪个 clause 只是在补合法性，  
> 哪个 clause 才真的把下一层 proof state 领出来。

#### 一张 near-paper paragraph crosswalk：把一整段连续 reduction prose 逐句压回 proof state

到这里，Section 4.2 的阅读手柄已经从：

1. 段落级；
2. 箭头级；
3. burden 级；
4. clause 级

一路压到了很细。

但读者真正回到 paper 页面时，最常遇到的 still 不是单句，而是一整个连续 paragraph：

> 四五句话连在一起，  
> 每一句都好像在推动 proof；  
> 可我不确定到底哪一句只是在补许可证，  
> 哪一句才真的把新 state 交出来。

这就是为什么还需要最后再补一页：

> 不再只拆单句，  
> 而是拿一整段连续 synthetic paragraph，  
> 逐句写出  
> `visible job / hidden dependency / earned state / not yet justified`。

这一页的目标不是新增概念，
而是把前面所有 notebook 压到最接近真实读 paper 的动作上：

> 你现在看到的不是一个 isolated sentence，  
> 而是一段连续 prose；  
> 你必须知道每一句在 proof chain 里到底只赚到了哪一点点合法性。

**先给一段真正接近 paper 手感的四句 synthetic paragraph**

下面这一段不是论文原文。
它只是故意写成接近 Section 4.2 的连续 prose，方便做真正的页边 crosswalk。

> `S1` We first regard the target Module-LWE shell as descending from the appropriate \(\mathcal{O}_K\)-module quotient, so that the relevant arithmetic objects share a common ambient parent.  
> `S2` Since the resulting torsion-free module is free in the class-number-one setting, we may pass to genuine global coordinates and express the reduction in a standard free \(R_q\)-module model.  
> `S3` Under the canonical embedding, the secret and error are then interpreted in the Euclidean geometry carrying the intended shortness semantics, and via the trace pairing and the codifferent this state remains aligned with the natural dual object required by the reduction.  
> `S4` Hence the resulting dual-compatible shell may be returned to the target average-case family, and the promised Module-LWE solver applies.

这一段的难点不在术语量，
而在它把：

1. arithmetic host
2. freeness legality
3. geometric law
4. dual glue
5. solver cash-out

压成了只用四句写完。

**先给一张逐句 crosswalk 总表**

| sentence | visible job | hidden dependency it consumes | earned state at sentence end | still not justified after this sentence | if over-read, you will hallucinate |
| --- | --- | --- | --- | --- | --- |
| S1 | 补共同 arithmetic host | target family 已经先被钉死 | objects share one arithmetic parent | legal free coordinates | “既然住进 \(\mathcal{O}_K\) 里，坐标就已经合法” |
| S2 | 兑现 freeness，产出合法 coordinates | S1 的共同宿主 + \(h_k^+=1\) | proof-legitimate free coordinates | honest geometric law | “坐标一合法，shortness / Gaussian 也自动合法” |
| S3 | 先给 geometry law，再给 dual glue | S2 的合法 coordinates | law-bearing, duality-compatible state | solver-facing cash-out | “几何和 duality 都讲了，所以已经等于 solver 可用” |
| S4 | 把先前积攒的合法性真正兑回 solver promise | S3 的 law + glue 同时成立 | solver-legal average-case family | 无；这是本段 cash-out 终点 | “这句只是收尾复述，没有新 claim” |

这张表最该盯的不是 `visible job`，
而是第四列与第五列一起看：

> 每一句到底真交出了什么；  
> 以及它交完以后  
> 还绝对没有资格说什么。

**把四句逐句写成真正的页边批注**

1. `S1` 该怎么批

   `visible job:` 把 target shell 落回共同 arithmetic parent。  
   `hidden dependency consumed:` 你必须已经有一个明确 target family，否则 “the target shell” 这件事本身是漂浮的。  
   `earned state:` ideals、modules、congruence data 终于可被认真看作住在同一宿主里。  
   `not yet justified:` 这还完全不等于 `we may pass to genuine global coordinates`。  
   `route arrow:` `B / arithmetic hosting`。  
   `main burden:` 宿主合法性，不是坐标合法性。  
   `margin note:` `host yes, coordinates not yet`。

2. `S2` 该怎么批

   `visible job:` 用 \(h_k^+=1\) 背后的 freeness 合法化 coordinate language。  
   `hidden dependency consumed:` `S1` 必须先把对象放进能谈 freeness 的宿主；否则 “is free” 根本没有落脚点。  
   `earned state:` \(R_q^d\) 现在是 proof-legitimate coordinate shell，而不再只是 notation shell。  
   `not yet justified:` 这里还没有给 shortness / Gaussian 一个 honest Euclidean law。  
   `route arrow:` `C / freeness discharge`。  
   `main burden:` hidden ideal-class twist 被清掉。  
   `margin note:` `legal coordinates earned, law not yet`。

3. `S3` 该怎么批

   `visible job:` 前半句在给 geometry law，后半句在给 dual glue。  
   `hidden dependency consumed:` `S2` 的 coordinates 必须已经合法，否则 embedding 解释与 dual pairing 都在假坐标上展开。  
   `earned state:` 句末你真正拿到的是一个 law-bearing, duality-compatible state，而不只是 “对象看起来更几何了”。  
   `not yet justified:` 直到句末为止，都还没有兑现 `solver applies`。  
   `route arrow:` D → E 横跨两箭头。  
   `main burden:` 先 `distribution-fidelity prep`，后 `dual/codifferent glue`。  
   `margin note:` `law first, glue second, solver still no`。

4. `S4` 该怎么批

   `visible job:` 做真正的 oracle-facing cash-out。  
   `hidden dependency consumed:` `S3` 末尾必须已经同时给出 honest law 与 dual-compatible state；缺任一项，这里都只能回到 syntax shell。  
   `earned state:` 这时才第一次拿到 solver-legal average-case family。  
   `not yet justified:` 无；这是该 paragraph 的 cash-out 终点。  
   `route arrow:` `F / oracle cash-out`。  
   `main burden:` `distribution-fidelity cash-out`。  
   `margin note:` `same shell is finally upgraded to same promised law`。

**为什么 S3 最值得单独盯住**

四句里最危险的通常不是 `S4`，而是 `S3`。
因为 `S4` 至少显眼地写了 `solver`；
而 `S3` 更像一条“技术准备句”，最容易被低估。

但恰恰是 `S3`，
在同一句里同时做了两层很重的工作：

1. 让 law 真正住进 honest geometry；
2. 让那个几何 state 和 natural dual / quotient machinery 接回同一条链。

如果这句被读滑过去，
`S4` 看上去就会像一句理所当然的收尾。
实际上它依赖的是：

```text
S2 legal coordinates
    ->
S3 honest law
    ->
S3 duality-compatible glue
    ->
S4 solver may apply
```

所以更成熟的页边笔记不该只写：

> **S3 = geometry sentence**

而该写：

> **S3 = geometry law + dual glue, still no solver yet**

**再给一张最短的逐句错读表**

| sentence | 最常见错读 | 更准确的纠正句 |
| --- | --- | --- |
| S1 | “已经进入 canonical arithmetic model，所以坐标也差不多了” | host only |
| S2 | “既然 free 了，几何语义当然顺便好了” | coordinates yes, law no |
| S3 | “继续讲几何，所以后半句只是术语扩展” | law first, glue second |
| S4 | “只是最后把上面的结果重复说一遍” | this is the first actual solver cash-out |

这张表真正想训练的是：

> 不要让一句“差不多懂了”  
> 覆盖掉  
> proof chain 实际上还差哪一层 state。

**给读者一张真正可带回原文页边的最短 sentence-crosswalk 卡**

如果你真要把这页方法带回 paper 页面，最短可以只写下面五格：

```text
sentence:
earned state:
still not justified:
consumes:
overclaim if read too fast:
```

这张卡正好夹在前面的 `route card` 和 `clause card` 中间：

1. `route card` 管整条箭头；
2. `sentence-crosswalk card` 管一句在 paragraph 里的确切进度；
3. `clause card` 管一句内部最危险的切口。

**最后把这一页压成一句最该带走的话**

Section 4.2 真正更接近真实 paper prose 的地方，不是段落里术语多，
而是每一句通常都只赚到一点点 state。
所以更成熟的读法，不只是问：

> “这一句大概在讲什么？”

而是继续问：

> “这一句句末到底真赚到了什么；  
> 它句末以后，proof 还绝对没有资格说什么？”

#### 一张 micro actual-sentence crosswalk：把更像 paper 的三句 bridge 压成 transport ledger

上一页的 `near-paper paragraph crosswalk`，
已经把一整段四句 prose 压成了句级账本。

但真正回到论文页面时，
读者还会碰到另一种更滑的句面：

> 每句更短，  
> 可每句里塞进的 proof job 反而更密；  
> 一个 `under`、一个 `via`、一个 `and remain compatible`，  
> 就把 `H=1` 的落点、law 的落点、glue 的落点  
> 全部压进去了。

所以还需要再补一页更小、更贴近 paper 句法的 notebook：

> 不再按 “整段四句” 做 crosswalk，  
> 而是拿一组更像真实 paper bridge 的三句 source-like prose，  
> 逐句写出  
> `literal claim / hidden import / bounce page / H=1 status / transport type / first failure`。

这里仍然不是论文原句。
只是故意把句法压得更像论文里的强句，
方便训练真正回带到原文页边时的最小审计动作。

**先给一组更像 paper bridge 的三句 source-like prose**

> `T1` After fixing the class-number-one regime, we identify the relevant quotient with a free \(R_q\)-module presentation and write the target samples in standard coordinates.  
> `T2` Via the canonical embedding and the trace-dual identification, these coordinates carry the intended Gaussian/shortness law and remain compatible with the dual object required by the reduction.  
> `T3` Consequently the transformed instance belongs to the promised average-case Module-LWE family, so an oracle for that family yields the desired solver.

这三句比上一页更难，
不是因为术语更多，
而是因为它们把：

1. host → free coordinates
2. coordinates → honest law + dual glue
3. law-bearing state → solver-legal family

压成了只有三次句号。

**先给一张真正能回带到 paper 页边的逐句 ledger**

| sentence | literal claim | hidden import | bounce page | H=1 status here | transport type | first failure if false |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | target quotient 已可写成 free \(R_q\)-module coordinates | 共同 arithmetic host 已先落稳；free 不是语气词，而是在兑现 freeness debt | phrase decoder + route sheet + worked paragraph-audit page | direct | object transport | T2 会在 notation-only coordinates 上假装谈 geometry |
| T2 | 同一组合法 coordinates 现在承载 honest shortness/Gaussian law，并接回 dual object | T1 必须已给出 proof-legitimate coordinates；否则 canonical embedding 与 trace pairing 都只是壳上的解释 | distribution-fidelity notebook + sentence-surgery notebook + four-burden overlay sheet | indirect | law transport + dual glue | T3 会把 “same shell” 错当成 “same promised family” |
| T3 | reduction 最终回到 solver 可接受的 average-case Module-LWE family | T2 末尾必须同时成立 honest law 与 dual compatibility；缺一项都只能回到 syntax shell | reduction-certificate notebook + worked reduction-by-reduction audit page | no | distribution transport / oracle cash-out | 整段只剩 “对象看起来像 MLWE”，却没有 solver promise |

这张表最该看的不是第一列，
而是最后三列一起看：

1. 这句到底在搬对象、搬 law，还是在做真正的 solver cash-out；
2. `H=1` 在这里是直接出手、间接被消费，还是根本没有新出手；
3. 如果这句是假的，下一句最先哪一层会塌。

**把三句逐句压成最短页边批注**

1. `T1` 该怎么批

   `literal:` 现在不仅在“重写对象”，而是在声称 free coordinates 已经合法。  
   `danger:` 这句最容易被读成纯 notation change。  
   `actual debt paid:` class-number-one regime 在这里第一次被真正花掉，用来清掉 hidden ideal-class twist。  
   `margin note:` host settled → coordinates legal, law not yet。

2. `T2` 该怎么批

   `literal:` 同一坐标系下，shortness/Gaussian law 与 dual object compatibility 同时被接上。  
   `danger:` 句中间那个 `and remain compatible` 最容易被当成附带说明。  
   `actual debt paid:` 前半句在补 honest geometry，后半句在补 dual glue；两半句不能并成一句 “继续解释 embedding”。  
   `margin note:` `law first, glue second, solver still no`。

3. `T3` 该怎么批

   `literal:` 这不是总结句，而是第一次真正宣称 solver-facing family 已恢复。  
   `danger:` 只要 `T2` 少了 law 或少了 glue，这句就会从 `solver promise` 退化成 `same shell slogan`。  
   `actual debt paid:` distribution-fidelity 在这里第一次完成 cash-out。  
   `margin note:` `promised family restored, solver may apply`。

**为什么这页比上一页更接近真实 paper 句法**

上一页的 `S1-S4` 还保留了“每句大致只做一项主工作”的善意。
这一页的 `T1-T3` 则更接近真实论文里常见的压缩写法：

1. `T1` 把 `host` 与 `freeness discharge` 挤进同一句；
2. `T2` 把 `law transport` 与 `dual glue` 挤进同一句；
3. `T3` 用一句看似平静的 `Consequently ...` 直接把前两句积攒的 state 全部兑回 solver promise。

所以真正成熟的读法，不该只在页边写：

> **T2 = geometry sentence**

而该明确写成：

> **T2 = law transport + dual glue; still no solver yet**

**给读者一张最短可手抄的 actual-sentence card**

```text
surface sentence:
literal claim:
hidden import:
bounce page:
H=1 status:
transport type:
first next sentence that breaks if false:
```

这张卡的用途不是做摘要，
而是强迫自己在读一条更像论文原句的 bridge 时，
立刻分清：

1. 这句表面上在说什么；
2. 它暗地里已经偷用了哪张前置许可证；
3. 它究竟是在搬 object、搬 law，还是在做 cash-out。

**最后把这一页压成一句最该带走的话**

Section 4.2 最难的地方，很多时候不是术语，
而是语法压缩。
一条真正像 paper 的句子，
常常会把 `H=1` 的直接落点、geometry 的 honest law、以及 dual glue
塞进同一句里。
所以更成熟的读法，不只是问这句“在讲什么”，
而是继续问：

> 这句现在到底在运哪一种 transport；  
> 如果它是假的，下一句最先坏掉的是哪一层 proof state？

#### 一张 dependency-colored source strip：不要把 presentation、freeness、dual glue、cash-out 染成同一种灰色

到这里，读者通常已经会做两件事：

1. 看一句 prose 时，分出 `literal claim` 和 `hidden import`；
2. 看一整段 prose 时，问清 `earned state` 和 `still not justified`。

但真正回到论文页面时，
还会剩下一种更细的错法：

> 我知道这句不只是表面那点意思，  
> 可我还是分不清  
> 这句主要是在还 `presentation` 债、`freeness` 债、`dual glue` 债，  
> 还是已经在做 `distribution-fidelity cash-out`。

这正是很多 reduction prose 最容易把人带偏的地方。
因为句面上常常都长得很像：

1. 都像在 “重新解释对象”；
2. 都像在 “换一种坐标说法”；
3. 都像在 “回到标准 MLWE 壳”；

但 proof 里它们做的主工作其实完全不同。

所以这里再补一页，
专门训练一种更接近真实读 paper 的动作：

> 先给一句 source-like prose 选一个  
> `表面颜色`，  
> 再强迫自己改判成  
> `真正主依赖颜色`。

这里的句子依然不是论文原句，
只是故意写成更像 Section 4.2 paper bridge 的短句。

**先给一条四句 source-like strip**

> `D1` We first rewrite the target samples in the customary coefficient shell so that the reduction interface is syntactically comparable with the standard Module-LWE presentation.  
> `D2` Because the relevant torsion-free quotient is free in the class-number-one regime, these coordinates are not merely cosmetic notation but proof-legitimate global coordinates.  
> `D3` Via the trace pairing and the codifferent, the resulting coordinate state remains aligned with the dual object that the reduction must feed downstream.  
> `D4` Hence the transformed samples still obey the promised average-case law, and the target Module-LWE oracle may now be invoked.

这四句如果读快了，
很容易被脑子自动染成一片灰：

> “反正都在把对象改写成标准 MLWE 样子。”

但更成熟的读法必须继续问：

> 这句的主 burden  
> 到底是 `presentation`、`freeness`、`dual glue`，  
> 还是 `distribution-fidelity cash-out`？

**先给一张真正的 dependency-color ledger**

| sentence | 表面最像哪种颜色 | 真正主依赖颜色 | 为什么不能按表面颜色读 | H=1 status | transport type | first fake next sentence if miscolored |
| --- | --- | --- | --- | --- | --- | --- |
| D1 | presentation | presentation | 这里只是在把句法外壳对齐，还没赚到 proof-legitimate coordinates | no | object transport | “既然已经长得像标准 MLWE，solver 当然快能调了” |
| D2 | presentation | freeness | 真工作不是 “坐标长出来了”，而是 class-number-one regime 让这些 coordinates 变成合法 proof state | direct | object transport | “既然能写坐标，geometry law 也就自动合法” |
| D3 | geometry | dual/codifferent glue | 句面虽然提了 coordinate state，但真正危险 claim 是它与 downstream dual object 仍然兼容 | indirect | law/object glue | “既然 dual object 对齐了，oracle family 也自然没变” |
| D4 | shell return | distribution-fidelity cash-out | 这里第一次真正宣称的是 promised average-case law 没断，不只是语法壳回来了 | no | distribution transport / oracle cash-out | “只要样子还像 MLWE，average-case solver 就都能接” |

这张表真正要训练的，
是把一句 sentence 的
**surface color**
和
**true burden color**
硬分开。

**逐句解释为什么最容易染错**

1. `D1` 为什么虽然是 `presentation`，却 still 很危险

   因为很多读者一看到  
   **standard Module-LWE presentation**
   就会下意识把它往 solver 方向听。

   但这句其实只够交出：

   > **syntax shell is now comparable**

   它完全还没交出：

   > **coordinates are proof-legitimate**  
   > 或  
   > `promised law is preserved`。

2. `D2` 为什么最常被误染成 `presentation`

   因为句面上它也在讲 coordinates。
   可真正花掉的不是“换了一种写法”，
   而是：

   > class-number-one regime  
   > 让 hidden ideal-class twist 被清掉，  
   > 因而这些 coordinates 第一次变得真合法。

   所以更准确的页边短批不该写：

   > **D2 = coordinate sentence**

   而该写：

   > **D2 = freeness discharge, coordinates become legal**

3. `D3` 为什么最常被误染成 `geometry`

   因为句面里出现了
   **coordinate state**
   和
   `trace pairing`，
   很像还在继续做“技术解释”。

   但这里真正重的 burden 是：

   > 这个 state  
   > 还能不能和 reduction 真要吃的 dual object  
   > 对得上。

   若这里染错，
   后面最自然的假句就是：

   > “dual object 都已经接上了，  
   > 所以 oracle family 当然也没变。”

   这一步正是 false jump。

4. `D4` 为什么最常被误染成 `shell return`

   因为它句面看起来像一句平静的收尾句，
   好像只是说：

   > “现在我们又回到了熟悉形式。”

   但更危险的真实 claim 是：

   > familiar shell  
   > 不只是 syntax-level familiar，  
   > 而是 law-side 仍属于 promised average-case family。

   这才是 solver 可调用的真正门槛。

**再给一张最短的 miscoloring 对照表**

| sentence | 最常见误染 | 更准确的更正句 |
| --- | --- | --- |
| D1 | “已经开始在替 solver 铺路了” | syntax comparable only |
| D2 | “继续讲坐标表示，所以还是 presentation” | freeness is being spent here |
| D3 | “继续讲 geometry / pairing 术语” | this is dual-compatibility glue |
| D4 | “只是把前面结果收束成标准外壳” | this is the first law-side cash-out |

这一页真正最该防住的，
不是“完全读不懂”；
而是：

> 句子大概看懂了，  
> 但 burden 染错了颜色。

一旦颜色染错，
下一句通常就会被脑子自动补成一条假的自然过渡句。

**给读者一张最短可手抄的 dependency-color card**

```text
surface sentence:
tempting color:
true burden color:
why not the tempting one:
H=1 status:
first fake next sentence if miscolored:
```

这张卡与前面的 `sentence-crosswalk card` 和 `clause card`
不是重复关系，而是分工关系：

1. `sentence-crosswalk card` 问这句到底赚到了什么 state；
2. `clause card` 问一句内部最危险的切口在哪；
3. `dependency-color card` 问这句主债主到底是谁。

**最后把这一页压成一句最该带走的话**

Section 4.2 的很多难句，
表面上都像在“把对象写回标准样子”。
但真正成熟的读法，必须继续分清：

> 这句是在对齐 presentation，  
> 在花 freeness，  
> 在补 dual/codifferent glue，  
> 还是已经在做 distribution-fidelity cash-out。

如果这四种颜色被染成一种灰，
读者就会把真正的 proof ladder
误听成“只是一路同义改写到 MLWE”。

### 10.4 为什么 ideal-lattice 攻击线没有直接打穿 ML-KEM / ML-DSA

这一点非常容易被误读。Section 4.1 的正确结论不是：

> class number one 既然让 PIP 平凡，那现代标准也危险。

而是：

1. 对 ideal-lattice schemes，PIP 与 short generator 确实是核心结构问题。
2. Biasse-Song 这类量子算法能在 cyclotomic fields 上高效解 PIP。
3. 当 \(h_k^+ = 1\) 时，PIP 在 principality 层面甚至更透明。
4. 但 ML-KEM / ML-DSA 不是 ideal-LWE schemes，而是 Module-LWE schemes。
5. 已知的 PIP-based attack line 并没有自然延伸到 standards 采用的 MLWE setting。

#### 一个零跳步 notebook：为什么 “PIP 变容易” 并不会自动变成 “MLWE 被攻破”

读到这里，很多密码学读者会下意识地做一个过快推理：

```text
class number one
    ->
principal ideals 更透明
    ->
PIP 更容易
    ->
structured lattice standard 也该更危险
```

这条链最关键的问题是：它把 **输入对象** 偷换了。

**Step 1：PIP / SGP 攻击线真正吃的是什么对象**

PIP 的输入不是一个 RLWE/MLWE 样本，而是一个 principal ideal
\(I=(\alpha)\)，
目标是恢复某个生成元 \(\alpha\)。

SGP 更进一步：即便知道这个 ideal 是 principal，也还要找一个 **足够短** 的生成元。

所以 ideal-lattice 攻击线最自然的工作流是：

```text
ideal object
    ->
recover generator
    ->
try to recover short generator
    ->
use that generator to attack the scheme
```

注意这里每一步都在 ideal language 里发生。攻击真正咬住的是：

1. 你有一个单独的 ideal；
2. 这个 ideal 的 principal structure 对攻击有用；
3. 一个短生成元会直接泄露某种 trapdoor-like information。

**Step 2：MLWE 标准实例真正给出的不是一个 ideal，而是一个 module-valued LWE instance**

ML-KEM / ML-DSA 的基础对象更接近：

\[
(A,b=As+e\bmod q),
\qquad
A\in R_q^{d\times d},\ s,e\in R_q^d.
\]

这里最重要的不是公式本身，而是对象类型已经变了：

1. secret 是一个 **向量**
   \(s\in R_q^d\)，不是单个 ideal generator；
2. public structure 是一个 module-valued linear system，不是“给你一个 principal ideal，求它的生成元”；
3. 难题是 LWE-style search/decision，而不是 PIP-style generator recovery。

所以即使底层相关整数环满足 class number one，也不会自动把一个 MLWE 实例改写成 “给定 ideal，求 generator” 的问题。

**Step 3：module lift 真正挡住了哪一步**

module lift 的关键效果，不是“把一切都变安全”，而是把 ideal world 的某些最直接攻击入口打断了。

在 ideal-lattice 语境里，很多结构可以被压成一个 rank-one ideal 或其生成元。
但在 module 语境里，哪怕底层每个 rank-one ideal 都 principal，你面对的仍然是：

\[
R_q^d
\quad\text{中的向量、子模、线性关系与噪声。}
\]

换句话说：

- class number one 消除了 ideal-class twisting；
- 它没有把一个 \(d\)-维 module instance 降成单个 principal ideal；
- 也没有白送一个“从 MLWE 样本恢复短生成元”的映射。

这就是为什么 “PIP 变透明” 和 “MLWE 被攻破” 中间还隔着一道非常实的模型差异。

**Step 4：真正缺失的桥是什么**

如果某人想把 ideal-lattice attack line 直接推到 ML-KEM / ML-DSA，上述差异迫使他至少补上一条新桥：

```text
MLWE sample
    ->
extract a canonically relevant ideal or principal ideal
    ->
recover a short generator that matters cryptanalytically
    ->
turn it back into secret/key recovery
```

而这条桥目前并不是现成的。

也就是说，已知的 PIP-based quantum algorithms 说明的是：

- 在 certain ideal-lattice settings，principal structure 可能被高效利用；
- 但它们并没有直接给出一个从标准 MLWE 样本到 “有用的 principal ideal generator” 的黑盒 reduction。

**Step 5：为什么这篇论文反而让边界更清楚**

这篇论文的重要性之一，不是简单地替某一边“站队”，而是把边界画得更清楚：

1. 在 ideal-lattice / RLWE proof language 里，class number one 确实会让 principal ideals、codifferent、module freeness 更显式；
2. 这会让某些 proof 变干净，也会让某些 attack prerequisite 变得更可检视；
3. 但标准 ML-KEM / ML-DSA 所处的是 module-LWE 语境，攻击者仍然需要跨过从 ideal objects 到 module instances 的那道桥。

因此最准确的结论不是：

> class number one 让标准更危险。

而是：

> class number one 让 ideal world 的结构更透明；  
> 而 module world 是否会因此变脆，取决于你能否额外建立一条从 MLWE 实例到 ideal-generator attack 的新桥。

#### 一个 attack-flow notebook：从 PIP → short generator → scheme break 到底哪一步在 module world 前断掉

如果你想把 Section 4.1 真正读成一条攻击线，而不是几句结论，最稳妥的方法是把它压成下面五步。

**Step 1：先固定攻击真正想拿到的不是“某个 ideal”，而是“对密码实例有用的短 generator”**

对 ideal-lattice scheme 来说，principal ideal 本身往往还不是终点。
攻击真正想要的是：

1. 先拿到一个与实例相关的 ideal object；
2. 再确认它是 principal；
3. 再恢复一个几何上足够短的 generator；
4. 然后把这个 short generator 变成某种 trapdoor-like leverage。

所以完整接口更像：

```text
ideal from the scheme
    ->
PIP solves principality/generator existence
    ->
SGP/unit search finds a useful short representative
    ->
short generator feeds the cryptanalytic step
```

这里真正承重的是最后一步：**短** 这个条件通常不是装饰，而是攻击是否真正闭合的关键。

**Step 2：为什么 PIP 只解决“存在句柄”，SGP 才解决“句柄好不好用”**

PIP 的产出是某个 generator
\(\alpha\)
满足
\(I=(\alpha)\)。
但由 units 作用，你立刻得到整条轨道
\[
\alpha,\ u\alpha,\ u_1u_2\alpha,\dots
\]
它们定义同一个 ideal，却可能有完全不同的 embedding length profile。

因此：

- PIP 回答的是 “我能不能把这个 ideal 还原成单元素对象？”；
- SGP 回答的是 “在所有等价 generators 中，我能不能找到真正短、真正对攻击有用的那个？”。

这就是为什么 log-unit lattice 会自然接到这条链上：它组织的正是“在单位轨道里找好代表”这一步。

**Step 3：为什么这条链在 ideal-lattice scheme 上是有密码学接口的**

在 ideal-lattice scheme 里，很多公开对象、秘密对象或 trapdoor 结构，本来就以单 ideal 或其 generator 的语言出现。

于是 short generator 一旦被找出来，就可能直接进入：

1. basis recovery；
2. trapdoor reconstruction；
3. short-vector style key material extraction；
4. 对实例的几何约束进行显式利用。

教学上最重要的不是逐一背攻击论文，而是抓住这条接口事实：

> ideal-lattice scheme 本来就把“单个 ideal + 其 generator”暴露成了有密码学意义的对象。

所以 PIP / SGP 线能够自然咬住实例，不是巧合，而是因为模型接口本来就对齐。

**Step 4：为什么同样的接口在标准 MLWE 上并没有现成出现**

到了 ML-KEM / ML-DSA 这边，公开数据更像
\[
(A,b=As+e\bmod q),\qquad s\in R_q^d.
\]

这时攻击者首先面对的是：

1. 一个 module-valued noisy linear system；
2. 一个向量 secret，而不是单个 ideal generator；
3. 一个天然以矩阵与向量写下的实例，而不是“给定 principal ideal”。

因此 ideal-world 的攻击线如果想继续往前走，至少要多补一层：

```text
module-LWE instance
    ->
extract a canonically meaningful ideal object
    ->
show that recovering a short generator of that object is enough
    ->
turn that into secret/key recovery
```

这条桥并不是已有 PIP/SGP 结果自动送的。

**Step 5：真正的断点应该怎么记**

把整条链压成一句可操作的话，就是：

```text
ideal world:
instance already speaks the language of ideals/generators

module world:
instance speaks the language of noisy module linear algebra
```

所以 class number one 确实会：

- 让 principal structure 更透明；
- 让 PIP 入口更清晰；
- 让 short-generator / unit-lattice 那一半攻击前提更容易被单独分析。

但它不会自动给出：

1. 从 MLWE 样本到单个 relevant ideal 的 canonical extraction；
2. 从 short generator 到 module secret 的直接密码学映射；
3. 从 ideal-lattice attack paper 到 ML-KEM / ML-DSA break 的黑盒 reduction。

这就是为什么 Section 4.1 的最准确阅读方式不是“论文暗示标准有危险”，而是：

> 论文把 ideal world 的结构条件讲得更透明了；  
> 至于这种透明性是否会穿过 module lift 传递到标准实例，仍然需要额外的新攻击桥。

也就是说，这篇论文一方面让某些 ideal-lattice attack prerequisite 更清楚，另一方面也更清楚地区分了：

- 哪些风险属于 ideal-lattice 世界；
- 哪些构造已经通过 module lift 躲开了那条最脆弱的攻击线。

#### 一个 scheme-level interface notebook：为什么“拿到短 generator”还不等于“已经打到标准实例”

前面的 attack-flow notebook 说清了 object mismatch，但很多读者还会追问一句：

> 就算 module world 里没有现成的 principal ideal，  
> 如果某种方法真的给了我一个相关 short generator，离 concrete break 到底还差什么？

这时最有效的做法不是空谈“攻击可能有效/无效”，而是把 scheme-level 接口拆开看。

**Interface A：ideal-lattice trapdoor 模型里，short generator 往往本来就是有密码学意义的秘密结构**

在许多经典 ideal-lattice 或 ring-trapdoor 叙事里，实例背后真正重要的秘密对象常常就是：

1. 一个相关 principal ideal 的 generator；
2. 或者由 short generator 可以直接展开出来的 short basis；
3. 或者再经过 inverse ideal / codifferent 调整后得到的等价短表示。

因此一旦你恢复了“对的那个 short generator”，常见的下一步往往就是：

```text
short generator
    ->
explicit short basis / good basis
    ->
inversion, decoding, preimage sampling, or trapdoor-style leverage
```

这条链未必对所有具体构造都完全同形，但它有一个共同点：

> scheme secret 与 ideal/generator language 本来就高度对齐。

所以攻击者拿到 short generator 之后，离真正的 cryptanalytic leverage 通常只差一层 relatively direct 的 basis conversion，而不是还差一次新的问题翻译。

**Interface B：为什么“可展开成 short basis”本身就已经很危险**

一个 short generator 危险，不只是因为它“看起来短”，而是因为它常常意味着：

1. 你获得了 associated ideal lattice 的紧凑短描述；
2. 你可以把乘法结构翻译成一组显式短向量；
3. 某些原本依赖隐藏 trapdoor 的操作，会开始变得可模拟、可逆或可预测。

教材视角下，最重要的不是背攻击细节，而是记住下面这句话：

> 在 ideal-lattice trapdoor 模型里，  
> “short generator” 往往已经非常接近 “good basis”。

这也是为什么 PIP 之后还要追 SGP：前者只告诉你“存在一个 generator”，后者才逼近“存在一个对攻击真正有用的短 generator”。

**Interface C：为什么到了 ML-KEM / ML-DSA 风格的 module 实例，接口突然不再直通**

标准 module-lattice 实例更像是在公开：
\[
(A,t=As+e \bmod q),\qquad s\in R_q^d,
\]
或者与之同类的 noisy module relation。

这时秘密对象的接口形状是：

1. 一个 module vector；
2. 若干 coordinate slots；
3. 一份被噪声包裹的线性关系；
4. 可能再叠加 rounding、compression 或 scheme-specific encoding。

它并不是天然以
“给定一个 principal ideal，求其短 generator”
的形式出现。

所以即便某个理想世界分析告诉你：

- 某个相关环里的 ideal 都 principal；
- PIP/SGP 更值得研究；
- short generators 的结构前提更透明；

你仍然还差至少两道新桥：

1. **抽取桥**：怎样从 noisy module instance 里 canonically 抽出一个真正相关的 ideal object？
2. **利用桥**：就算抽出来了，为什么那个 ideal 的 short generator 足以恢复 module secret、伪造签名、或破坏 KEM 安全游戏？

如果这两道桥没有被额外建立，那么“short generator 很危险”仍主要是 ideal world 的结论，而不是对当前标准实例的现成 break。

**Interface D：把“最后一步”写成一张小对照表**

| 问题 | ideal-lattice / trapdoor-flavored scheme | ML-KEM / ML-DSA-style module scheme |
| --- | --- | --- |
| 公开对象主要暴露什么 | 单 ideal、其 q-ary lattice，或与之紧耦合的环对象 | noisy matrix-vector relation over \(R_q^d\) |
| 秘密结构长什么样 | short generator / short basis 往往就是核心秘密 | short module vector、relation witness、scheme-specific encoding |
| 拿到 short generator 后会怎样 | 常可直接转成 good basis 或 trapdoor leverage | 仍需证明它与 \(s\) 或签名见证有可用映射 |
| 剩余障碍是什么 | 多半是几何质量与算法代价问题 | 首先是对象翻译问题，其次才谈几何攻击 |

这张表的教学意义在于：它把“攻击有没有路可走”拆成了两个互不相同的问题。

1. 数论结构是否已经足够透明？
2. 这种透明性是否真的落到了 scheme interface 上？

Section 4 帮我们无条件推进了第一个问题；而第二个问题，在今天的标准 module-lattice 语境里，依然需要额外的新论证。

**Interface E：本节最该带走的判断句**

最稳妥的总结不是：

> 只要 class number one 成立，short-generator 攻击就更接近打穿标准。

而是：

> class number one 让 ideal-lattice prerequisite 更干净；  
> 但从 prerequisite 到 standard-instance break，中间仍隔着一个 scheme interface problem。

把这句话记住，你就不容易把论文的“结构透明性提升”误读成“对 ML-KEM / ML-DSA 的现成直接攻击”。

#### 一个 attack-object ledger：ideal-lattice 攻击线真正围绕哪些对象转，哪些对象在 module world 里缺席

前面的 attack-flow notebook 解释了“链为什么会断”，scheme-interface notebook 解释了“短 generator 为什么还不等于 concrete break”。
但如果你想真正不再混淆 ideal world 和 module world，最好再把攻击线上最关键的对象单独列账。

因为很多误读都来自同一个问题：

> 讨论里明明一直在说 generator、short basis、trapdoor、module secret，  
> 但这些词到底是不是同一个对象的不同叫法？

答案是否定的。
它们是不同层次的对象，而且恰恰是这些对象之间的接口，决定了攻击线能不能闭环。

**先给出一张对象账本**

| 对象 | lives where | 谁通常把它交给攻击者 | PIP 能否直接处理 | SGP 能否直接处理 | 若拿到它，下一步最自然做什么 | 为什么它在标准 MLWE 里通常不现成出现 |
| --- | --- | --- | --- | --- | --- | --- |
| principal ideal \(I=(\alpha)\) | ideal world | ideal-lattice public object 或某个已抽出的 ideal | 是，PIP 正是从这里起步 | 只在知道其 principal 后继续追短代表 | 先恢复某个 generator | 标准实例公开的是 noisy module relation，不是单个 ideal |
| 某个 generator \(\alpha\) | ideal world | PIP 输出 | 是，PIP 的直接产物 | 还不够，SGP 继续筛单位轨道 | 问它是否足够短、足够平衡 | MLWE 不直接把 secret 写成单个 generator |
| short generator | ideal world + embedding geometry | SGP / unit search 的目标 | 否 | 是，它正是 SGP 的目标对象 | 展开成 short basis、trapdoor leverage、几何攻击接口 | 标准实例缺少“相关 ideal 已知且 generator 有用”的前提 |
| short basis / good basis | ideal-lattice trapdoor world | 常由 short generator 派生 | 否 | 间接，SGP 找到好 generator 后可导出 | inversion、decoding、trapdoor reconstruction | 标准 MLWE 的 secret 更像 module vector，不是天然 basis trapdoor |
| trapdoor-like leverage | scheme interface world | 由 short basis 或等价短描述导出 | 否 | 间接 | 接到 concrete break：恢复 secret、伪造、解码、采样 | 需要额外证明“这个短描述真的控制了 scheme secret” |
| module secret \(s\in R_q^d\) | module world | 标准实例隐藏对象 | 否 | 否 | 若直接恢复则攻击完成 | 它本来就不是 ideal/generator language 下的对象 |
| noisy relation \((A,b=As+e\bmod q)\) | module world | 标准公开输入 | 否 | 否 | 先解 search/decision MLWE | 这是 MLWE 自己的接口，不会自动变成 ideal attack 输入 |

这张表最重要的教学价值是：

> PIP、SGP、short basis、trapdoor leverage、module secret 不是“一个对象写成了五种语言”；  
> 它们是五个不同阶段的对象。

**再把对象之间的箭头画出来**

在 ideal-lattice 攻击线里，最常见的希望链条更像：

```text
principal ideal
    ->
some generator
    ->
short generator
    ->
short basis / good basis
    ->
trapdoor leverage
    ->
concrete break
```

而在标准 MLWE 里，攻击者一开始真正拿到的是：

```text
noisy module relation
    ->
?
    ->
module secret / break
```

两条链之间并不是只差一个“更强算法”，而是差了对象接口本身。

**哪几步是 PIP 真在做的，哪几步已经超出 PIP**

这也是一个非常值得单列的区分：

| 阶段 | PIP 负责吗 | 说明 |
| --- | --- | --- |
| ideal 是否 principal | 是 | PIP 的入口就是 principal ideal |
| 恢复某个 generator | 是 | PIP 的直接输出 |
| 在所有 generators 中找一个短的 | 否 | 这是 SGP / unit-orbit search 的问题 |
| 从 short generator 展开 good basis | 间接 | 依赖具体构造接口，不是 PIP 本身 |
| 从 good basis 变成 scheme break | 否 | 这是 cryptanalytic interface 的最后一跳 |

所以当你读到 “class number one 让 PIP 更透明” 时，应该自动在脑中补一句：

> 好，这最多是在推进账本前两行；  
> 后面几行仍然需要额外桥梁。

**为什么这张 ledger 能直接解释 module lift 的防护效果**

module lift 的关键不在于“它把一切攻击都挡掉”，而在于它让公开对象直接落在账本的最底部两行：

1. noisy relation；
2. hidden module secret。

而不是落在最上面的：

1. principal ideal；
2. generator；
3. short generator。

因此它最直接打断的并不是某个具体算法，而是 **攻击线的入口对象选择**。

换句话说：

> class number one 让 ideal-side ledger 更干净；  
> module lift 则让标准实例从一开始就不把“好攻击对象”直接交出来。

**把本节压成一句真正可记住的话**

如果你只带走一条判断句，最好记这个：

> ideal-lattice attack 能不能闭环，首先取决于你拿到的是哪一行对象；  
> 而不是只取决于某个 number-theoretic subroutine 本身有多强。

这就是为什么同样面对 class number one：

- ideal world 会说 “principal / generator / short basis 这条链更清楚了”；
- module world 却仍然可以说 “我公开给你的只是 noisy \(R_q^d\) relation”。

#### 一个 construct-interface notebook：为什么在 ideal world 里，short generator 往往真的能继续长成 short basis 或 trapdoor leverage

到这里，很多读者已经接受了：

1. PIP 只保证某个 generator 存在；
2. SGP 还要再追一个真正短的 generator；
3. 标准 MLWE 并不会天然把这种对象交出来。

但还会剩下一个很自然的问题：

> 好，就算我真的拿到了 short generator，  
> 它为什么常常会继续长成 short basis、decoding leverage、甚至 preimage sampling interface？

这一步如果不讲清楚，`short generator is dangerous` 仍容易像一句口号。

最稳妥的读法，是把它拆成三种最典型的 constructive 接口。

**Interface 1：从 short generator 到 short basis**

假设某个 ideal 确实是 principal：
\[
I=(\alpha).
\]

那从纯代数上说，你立刻有一个同构
\[
\mathcal{O}_K \xrightarrow{\ \cdot \alpha\ } I.
\]

如果你手里原本就有一组基
\[
\omega_1,\dots,\omega_n
\]
生成 \(\mathcal{O}_K\)，那么
\[
\alpha\omega_1,\dots,\alpha\omega_n
\]
就给出 \(I\) 的一组基。

问题不在“能不能写出一组基”，而在：

> 这组基在 canonical embedding 里几何上好不好用？

这正是 short generator 真正有价值的地方。
如果 \(\alpha\) 在 embedding 意义下足够平衡、足够短，那么它把 \(\mathcal{O}_K\) 那组参考基 transport 到 \(I\) 里之后，往往会给出一个更容易控制的 basis shape。

所以：

- generator 解决的是 “有没有一个单元素句柄”；
- short generator 进一步改善的是 “这个句柄把基拉成什么几何形状”。

这就是为什么 ideal world 常常把

```text
short generator
    ->
good basis candidate
```

看成一条自然箭头。

**Interface 2：从 good basis 到 decoding / inversion leverage**

一旦某个公开关系背后确实隐藏着一组比较好的 basis，trapdoor holder 通常就能做 outsider 做不到的几何操作。

最粗糙的直觉就是 Babai / nearest-plane 那一类：

1. 给定一个 target point 或某个 coset；
2. 持有好基的人可以逐层投影；
3. 因而更容易找到一个短代表，或一个足够近的格点。

在讲义其他地方我们已经见过这种桥：

```text
good basis
    ->
better control of a coset
    ->
decoding / inversion / representative recovery
```

这并不意味着“只要有好基就一定立刻破译所有方案”。
更准确的说法是：

> good basis 会把某些原本对外部攻击者困难的几何任务，  
> 变成对 trapdoor holder 更可计算、更可控制的任务。

所以 ideal-lattice/trapdoor 语境里，经常会把 short basis 视作一种 leverage-bearing secret structure，而不仅仅是“另一组坐标”。

**Interface 3：从 good basis 到 preimage sampling**

如果再往前走一步，就会遇到本地其他讲义里已经单独讲过的那条 GPV 线路：

```text
short basis
    ->
sample short representatives from a coset
    ->
preimage sampling interface
```

这条桥最重要的不是签名细节，而是它说明：

> good basis 的作用不只在 deterministic decoding；  
> 它还可能升级成 distribution-controlled short witness generation。

在 generic GPV-style language 里，公开关系长成
\[
A\mathbf{z}=\mathbf{u}\pmod q,
\]
而秘密不是“某个 magic scalar”，而是一组 hidden short basis，能让你在相应 coset 上采样短 preimage。

同样地，在更结构化的 NTRU / Falcon 风格里，公开对象不是一般矩阵，而是某个 ring map；但核心接口依旧类似：

1. 公开侧给出一个 relation family；
2. 秘密侧持有一组结构化短 basis；
3. trapdoor holder 因而能生成 outsiders 不能稳定生成的短 witness。

这说明 short basis → trapdoor leverage 不是一句比喻，而是在具体构造世界里确实反复出现的接口模式。

**把三类 leverage 压成一张表**

| 你已经拿到的对象 | 接下来最自然的 leverage | 在哪类构造里最常见 | 还缺什么才算真正闭环 |
| --- | --- | --- | --- |
| short generator | transport 出更可控的 basis shape | ideal-lattice / principal ideal world | 要证明这个 basis 对公开关系真的有用 |
| good basis | decoding / inversion / representative recovery | trapdoor-flavored lattice relations | 要把“几何上更好解”接回 scheme secret |
| hidden short basis on a public relation | preimage sampling / short witness generation | GPV-style、结构化 trapdoor 签名、NTRU/Falcon-style public maps | 要证明输出分布与安全接口匹配 |

这张表最重要的不是记住例子，而是记住方向：

> ideal world 里 “short generator 危险” 往往不是终点判断；  
> 它只是因为它有机会继续生长成更强的 secret structure。

**为什么这仍然没有自动打到标准 MLWE**

讲到这里，最容易再犯一次过快推理：

```text
short generator
    ->
good basis
    ->
decoding / sampling leverage
    ->
那标准也该危险
```

真正缺失的仍是同一件事：

> 这些 leverage-bearing structures 必须先和公开实例的接口对齐。

在 GPV / Falcon-like world 里，scheme 本来就故意把“公开 relation + 隐藏 short basis”设成一对接口。
而在 ML-KEM / ML-DSA 这边，公开数据更像
\[
(A,b=As+e\bmod q),
\qquad s\in R_q^d,
\]
其隐藏对象是 module secret、masking relation、或带噪声的 witness，而不是“某个公开 relation 背后的一组已抽象成 short basis 的 trapdoor”.

所以从本节视角看，标准 MLWE world 缺失的不是一句抽象 warning，而是两层具体接口：

1. **抽取层**：怎样从 noisy module relation 抽出真正相关的 ideal / basis object；
2. **利用层**：怎样证明那个 short basis 或 short generator 的 leverage 真能回到标准实例的 secret 或 forgery interface。

只要这两层没被补上，short generator → good basis → trapdoor leverage 这条链就仍主要属于 ideal-lattice / trapdoor-flavored world。

**本节最该带走的一句补充判断**

在 `attack-object ledger` 的基础上，再补一句最稳妥的话就是：

> class number one 让 ideal-side objects 更透明；  
> 但真正决定攻击能否闭环的，是这些对象能不能继续长成与公开 relation 对齐的 secret structure。

这正是为什么本论文能让 ideal-world prerequisite 更清楚，却仍不能被直接翻译成“标准 MLWE 因此自动失陷”。

#### 一个 public-relation card：哪些公开关系天然接受 short-basis leverage，哪些关系只是 noisy MLWE interface

如果前面的 `attack-object ledger` 和 `construct-interface notebook` 已经读顺了，下一步最值得再压实的一件事，就是把“公开关系到底长什么样”也单独列出来。

因为很多误读最后都会落到这一句：

> 既然 short basis / trapdoor leverage 这么有用，  
> 为什么它在某些 lattice constructions 里是自然接口，在 MLWE 标准里却不是？

最简洁的答案是：

> 公开关系的形状不同，决定了 secret structure 有没有天然落脚点。

下面先把三种最典型的 relation 摆在一张卡片上。

**Card A：generic GPV-style public relation**

最抽象的 trapdoor-signature 关系长成
\[
A\mathbf{z}=\mathbf{u}\pmod q.
\]

这里公开的是：

1. 一个矩阵或线性映射 \(A\)；
2. 一个 target \(\mathbf{u}\)；
3. 以及“找到短 preimage \(\mathbf{z}\)”这件事本身。

而隐藏结构是：

1. kernel lattice \(\Lambda_q^\perp(A)\) 的 short basis；
2. 或等价的 trapdoor sampling structure。

这类 relation 为什么天然吃 short-basis leverage？

因为它本来就在问：

> 给定一个公开 relation，能不能在相应 coset 里找到短 witness？

一旦你真的持有 short basis，这条接口就自然升级成：

```text
public relation A z = u mod q
    +
hidden short basis of the kernel
    ->
short-preimage sampling / signing leverage
```

也就是说，short basis 在这里不是“额外猜到的秘密”，而是 **正好控制这个 relation 的 hidden geometry**。

**Card B：NTRU / Falcon-like structured public relation**

更结构化一些的 relation 会长成
\[
(u,v)\mapsto u+vh \pmod q,
\]
或者等价地，对消息导出的 target \(c\)，寻找短对
\[
(s_1,s_2)\in R^2,\qquad s_1+s_2h\equiv c\pmod q.
\]

这里公开的是：

1. ring map 本身；
2. 公开键 \(h\)；
3. 以及相应 coset / kernel relation。

隐藏结构则是：

1. 某个结构化 kernel lattice 的 short basis；
2. 例如由 NTRU equation 诱导出来的短向量族。

所以这类 relation 和 generic GPV 的差别，不在“有没有 trapdoor”，而在：

- public relation 更结构化；
- short basis 也更结构化；
- sampling / decoding leverage 可以借助 FFT / ring arithmetic 压得更紧凑。

但接口本质仍然是同一类：

```text
public relation family
    +
hidden short basis adapted to that relation
    ->
controlled short witness generation
```

这就是为什么 Falcon-like world 会自然长出

- short basis；
- Gaussian sampling；
- preimage-style witness generation；
- trapdoor-sensitive implementation risks

这整条线。

**Card C：standard MLWE public relation**

而标准 MLWE / ML-KEM / ML-DSA 一开始公开的更像
\[
(A,b=As+e\bmod q),
\qquad
s\in R_q^d.
\]

这里公开的不是 “一个 relation + 要你找短 preimage”，而是：

1. 一个 noisy linear system；
2. 一个被噪声污染的 witness relation；
3. 一个目标是 search / decision MLWE 的 average-case instance。

隐藏对象更像：

1. module secret \(s\)；
2. error term \(e\)；
3. 或进一步的 masking / signing witness discipline。

这时最关键的区别是：

> hidden object 本身并不是 “这张公开 relation 的 short basis trapdoor”。

所以哪怕你在别处学到了

```text
short basis
    ->
short witness generation
```

也不能自动把它贴回这里，因为你还没有：

1. 一个与实例接口对齐的公开 coset relation；
2. 一组已经与该 relation 绑定的 hidden short basis。

**把三张卡压成一张总表**

| relation family | 公开接口首先长什么样 | 隐藏结构最自然长什么样 | short-basis leverage 自然吗 | 为什么 |
| --- | --- | --- | --- | --- |
| generic GPV-style | \(A\mathbf{z}=\mathbf{u}\pmod q\) | kernel lattice 的 short basis | 是 | relation 本来就在问短 preimage / short witness |
| NTRU / Falcon-like | \(u+vh\equiv c\pmod q\) 这类结构化 ring relation | 结构化 kernel lattice 的 short basis | 是 | public map 与 hidden basis 天然成对出现 |
| standard MLWE | \((A,b=As+e\bmod q)\) | module secret、noise、masking relation | 否，至少不现成 | 隐藏对象不是“公开 relation 的 short basis trapdoor” |

这张表最值得记住的一点是：

> 并不是所有 lattice-based public relations 都在问同一种 hidden object。

GPV / Falcon-like relation 公开的是“某个可被 short basis 控制的 relation family”；
而 MLWE relation 公开的是“某个带噪声的 average-case witness relation”。

**因此真正缺的不是算法名字，而是接口类型**

很多讨论会把问题说成：

> 有没有一个更强的 PIP / SGP / quantum algorithm？

但这还不是最底层的分界。
更底层的分界其实是：

> 这张公开 relation 到底是不是一张  
> “short basis holder 能自然控制 witness generation”的 relation？

如果答案是 `是`，那么 short basis / trapdoor leverage 往往会变成构造内生接口。
如果答案是 `不是`，那么你至少还要补：

1. relation extraction；
2. basis alignment；
3. leverage-to-secret mapping

这三层新桥，才能把 ideal-world 的 leverage 真正接到标准 MLWE instance 上。

**本页最该带走的一句卡片化总结**

把这一页压成一句话就是：

> short basis 之所以在 GPV- or Falcon-like world 里自然有用，  
> 不是因为“格都一样”，而是因为公开 relation 本来就被设计成接受这种 hidden geometry。  
> 标准 MLWE relation 没有天然给出这类接口。

#### 一个 break-step map：short generator → concrete break 中间到底还隔着哪几层构造接口

前面的 `attack-object ledger` 解释了攻击线围绕哪些对象转，`construct-interface notebook` 解释了为什么 short generator 往往能长成 short basis 或 trapdoor leverage，`public-relation card` 又解释了为什么有些 relation 天然接受这种 leverage、有些 relation 则不接受。

如果把这三页合起来，最后还剩一个最容易被一句口号糊过去的问题：

> 具体来说，所谓 “short generator 变危险”，  
> 危险到底是在哪一种 `concrete break` 里真正落地的？

这一步最好不要用一句“于是就能攻击某方案”带过。
更稳妥的做法，是把最后一跳拆成几类反复出现的构造原型。

**先给一张 break-step archetype map**

| break archetype | 公开接口首先在问什么 | 隐藏结构若被拿到，会给出什么能力 | 最终 break 更像什么 | 为什么 short generator / short basis 可能有用 | 为什么这不自动等于标准 MLWE break |
| --- | --- | --- | --- | --- | --- |
| inversion / decoding archetype | 给定某个公开 map、coset 或 q-ary lattice 关系，求一个短代表或近代表 | decoding、inversion、representative recovery | trapdoor inversion、解码、恢复某类隐藏代表 | short generator 若能展开成 good basis，就可能改善几何求解能力 | 标准 MLWE 公开的是 noisy average-case relation，不是“请在这个 coset 里解一个短 preimage” |
| signing / preimage-sampling archetype | 给定公开 relation，生成一个满足关系且分布受控的短 witness | coset sampling、short witness generation | 伪造签名、模拟 trapdoor-only witness generation | short basis 直接控制 kernel/coset 的几何，能把关系升级成 sampling interface | 标准 MLWE 的 hidden object 不是这张 relation 背后的 short-basis trapdoor |
| equivalent-secret reconstruction archetype | 从公开的结构化对象回推出一个与 secret key 等价的短描述 | 重建同等效力的 short basis、generator 或 trapdoor | key recovery、trapdoor cloning、secret-structure reconstruction | 若 scheme 的 secret 本来就是 principal / basis / structured trapdoor，一旦恢复等价短描述就可能直接闭环 | 标准 MLWE secret 更像 module vector 与 noise discipline，不是“一个等价 trapdoor 描述” |

这张表最重要的不是记住例子，而是记住：

> `short generator` 从来不是 break 的名字；  
> 它只是某些构造里可能继续长成 break-capability 的上游对象。

**Archetype A：inversion / decoding 型闭环**

这一类最容易从 q-ary lattice / trapdoor encryption 的直觉进入。
公开侧给出某个 map 或某个 coset，真正困难的问题是：

\[
\text{find a short or sufficiently close representative consistent with the public relation.}
\]

如果攻击者持有一组与该 relation 对齐的 good basis，那么很多 outsider 难做的几何步骤就会变得更可控，例如：

1. 更稳定地在 coset 上找近点；
2. 更容易做 bounded-distance style representative recovery；
3. 更容易把公开 map 反解回某类短 preimage 或短代表。

所以这一类闭环真正长得像：

```text
short generator
    ->
good basis candidate
    ->
relation-aligned decoding leverage
    ->
inversion / decryption / representative recovery
```

关键是中间这句 `relation-aligned`。
不是任何 short basis 都自动解任何公开关系；它必须先是 **这张公开关系背后的几何 secret structure**。

**Archetype B：signing / preimage-sampling 型闭环**

这一类在 GPV- or Falcon-like world 里最自然。
公开关系长得更像：

\[
A\mathbf{z}=\mathbf{u}\pmod q
\]

或者

\[
s_1+s_2h\equiv c\pmod q.
\]

这里 break 的核心不是“找一个近点就结束”，而是：

\[
\text{generate a short witness with the right relation and a controlled distribution.}
\]

一旦 hidden structure 真的是这张 relation 的 short basis trapdoor，那么攻击者就可能从

```text
short basis
    ->
coset sampling / short witness generation
    ->
forgery or signer-equivalent capability
```

这条线闭环。

所以这类场景里，short generator / short basis 危险，不是因为它抽象上“很短”，而是因为它正好接到了：

1. kernel geometry；
2. coset sampling；
3. witness distribution control；
4. signer-only interface。

这也是为什么 `public-relation card` 特别把 GPV-style relation 和 Falcon-like relation 单独列出来。

**Archetype C：equivalent-secret reconstruction 型闭环**

还有一类闭环更偏“结构恢复”。
某些构造或分析里，secret side 本来就可以被理解成：

1. 一个 structured principal ideal；
2. 一组 short basis；
3. 一个等价的 trapdoor description。

在这种语境下，攻击者若恢复出一个 **等价于 honest secret capability** 的短描述，即便它和实现里存储的字节串不完全一样，也可能已经足以宣布 break：

```text
short generator / short basis
    ->
equivalent secret structure
    ->
same inversion / sampling / signing power
    ->
effective key recovery
```

这一类闭环提醒我们：

> 有时 concrete break 的终点不是“还原出完全相同的 secret key encoding”；  
> 而是“恢复出一份具备同等攻击能力的 secret structure”。

但它同样依赖一个强前提：
scheme 的 secret interface 确实必须已经写成 principal / basis / trapdoor 这种语言。

**把最容易偷换的四根箭头单独拆开**

上面三类 archetype 虽然不同，但它们都共用同一条更抽象的箭头链：

```text
short generator
    ->
good basis candidate
    ->
relation-aligned hidden structure
    ->
concrete capability
    ->
concrete break
```

这里每一根箭头都不是废话，而是需要单独证明的接口命题：

| 箭头 | 真正要补的论证 | 在哪些世界里更自然 | 在标准 MLWE 上通常最先断在哪 |
| --- | --- | --- | --- |
| short generator -> good basis candidate | 这个 generator 是否真的足够短、足够平衡，能把参考基 transport 成好几何形状 | ideal-lattice / principal ideal world | 常常还没拿到 relevant generator object |
| good basis candidate -> relation-aligned hidden structure | 这组基是否真的控制了公开 relation 的 kernel/coset geometry | GPV、Falcon-like、trapdoor-flavored relation | 公开 relation 不是以 short-basis trapdoor 为中心组织的 |
| relation-aligned hidden structure -> concrete capability | 这组结构究竟给你 decoding、sampling 还是 equivalent-secret reconstruction | trapdoor encryption、trapdoor signatures、structured public maps | 隐藏对象首先是 module secret 与 noise，不是 capability-bearing basis |
| concrete capability -> concrete break | 这个能力是否真能翻译成 decrypt、forge、recover、clone trapdoor | 具体构造层 | 还没把 ideal-world leverage 映射回 standard-instance goal |

如果你把这张表记住，就不容易再把

> “number-theoretic prerequisite 更透明”

偷换成

> “标准 MLWE 因而自动有了 concrete break”.

因为两句话中间隔着的，正是这四根箭头。

**为什么这一页能把 ideal world 与 module world 的分界说得更硬**

从这一页视角看，标准 MLWE world 最核心的区别不是“攻击者算法还不够强”，而是：

1. 起点对象不同；
2. relation 组织方式不同；
3. honest secret 的语言不同；
4. concrete break target 也不同。

ideal world 往往从 `principal ideal / generator / good basis` 这些上游对象进入；
而标准 MLWE world 则从

\[
(A,b=As+e\bmod q),\qquad s\in R_q^d
\]

这类 noisy module relation 进入。

因此它最先暴露给攻击者的不是 “某个可继续打磨成 trapdoor 的短结构”，而是 “一个 average-case noisy instance”。
这就是为什么 class number one 虽然能显著清理 ideal-side prerequisite，却仍不足以直接宣布标准参数实例失陷。

**本页最该带走的一句最终判断**

最稳妥的说法不是：

> short generator 一旦出现，break 就近在眼前。

而是：

> short generator 是否危险，要看它能不能继续长成  
> **relation-aligned hidden structure -> concrete capability -> concrete break**  
> 这条链；  
> 而标准 MLWE 恰恰没有把这条链的前两步天然交给攻击者。

#### 一个 scheme-interface footnotes：把 GPV-style、Falcon-like、NTRU-like、ML-KEM-style、ML-DSA-style 放到同一张接口地图里

前面的 `break-step map` 已经把抽象箭头拆开了，但如果不把它们贴回熟悉构造，很多读者还是会觉得：

> 道理我懂了，可一到具体方案，  
> 我还是分不清“哪一类 secret structure 本来就在构造里”，  
> “哪一类只是攻击者希望从外部再抽出来的结构”。

这时最有效的做法，就是把几类最常见的 lattice-based interface 摆到同一张地图上。

注意下面这些脚注页不是在声称：

1. 本论文直接分析了这些方案；
2. 或者某个现成攻击已经完成闭环。

它们只是帮助你判断：

> 同样说“short basis”“trapdoor”“module secret”，  
> 不同构造到底把哪一层对象真的内生进了 public/secret interface。

**先压成一张 scheme-interface map**

| family | 公开接口主要长什么样 | honest side 真正持有什么 | 若攻击者恢复等价短结构，最自然拿到什么能力 | 哪几根箭头是构造本身已经铺好的 | 为什么这不由 Weber 结果自动给出 |
| --- | --- | --- | --- | --- | --- |
| GPV-style trapdoor signatures | \(A\mathbf{z}=\mathbf{u}\pmod q\) | kernel lattice 的 short basis | coset preimage sampling、签名能力 | good basis → relation-aligned hidden structure → concrete capability → forgery | 论文没有把公开 \(A\) 直接变成 short basis recovery 算法 |
| Falcon-like NTRU signatures | \(s_1+s_2h\equiv c\pmod q\) | NTRU lattice 的结构化 short basis | Gaussian sampling、等价 signer capability | structured short basis → sampling capability → forgery | class number one 不会自动把公开 \(h\) 变成 NTRU trapdoor |
| NTRU-like inversion / decryption world | 结构化 public map + ciphertext / target relation | 可做 inversion 的短结构或等价 trapdoor | decoding、inversion、代表元恢复 | good basis → decoding capability → decryption/key-recovery-style break | 论文没有给出从公开实例恢复 inversion trapdoor 的接口 |
| ML-KEM-style standard MLWE | noisy module relation，公钥核心像 \(t=A s+e\) | 小 module secret、noise discipline | 若真能恢复 secret 则是 decapsulation / search break | 构造没有内生 short basis trapdoor 这几根箭头 | Weber 结果只清理底层 arithmetic，不会把 \(t=A s+e\) 变成 principal-ideal/basis 接口 |
| ML-DSA-style FS-with-aborts | \(t=A\mathbf{s}_1+\mathbf{s}_2\) 加上 transcript / hint / abort interface | 小 module secret、masking、rejection discipline | transcript-level secret recovery 或 forgery | 构造同样没有内生 short basis → sampling trapdoor 接口 | Weber 结果不会把 FS-with-aborts 签名接口改写成 GPV/Falcon 式 trapdoor relation |

这张表最值得盯住的一列是：

> **哪几根箭头是构造本身已经铺好的**

因为它直接决定了 “一旦拿到等价短结构，攻击是否会立刻变成 concrete capability”。

**Footnote A：GPV-style 为什么最像“short basis 一到手，能力就直接落地”**

GPV-style relation 一开始就在问：

\[
A\mathbf{z}=\mathbf{u}\pmod q
\]

下的短 preimage。

honest secret 本来就是：

1. kernel lattice 的 short basis；
2. 或与之等价的 trapdoor sampling structure。

所以这里的接口是最整齐的：

```text
public relation
    +
hidden short basis
    ->
short-preimage sampling
    ->
signature capability
```

如果攻击者恢复出的正是一组等价 short basis，那么它拿到的并不是“也许以后有用的一条数论信息”，而是：

> 和 signer 几乎同类型的 sampling capability。

这就是为什么 GPV-style world 里，good basis → concrete capability 几乎是构造定义的一部分。

**Footnote B：Falcon-like 为什么仍然属于 trapdoor world，而不是一般 MLWE world**

Falcon-like interface 看上去更结构化，公开关系更像：

\[
s_1+s_2h\equiv c\pmod q,
\]

而 honest side 持有的不是一般矩阵 trapdoor，而是 NTRU lattice 的结构化 short basis。

这意味着它仍然保留了同一种核心接口：

1. public relation 已经给定；
2. hidden short basis 正好控制其 kernel/coset geometry；
3. signer capability 来自 Gaussian sampling，而不是从 noisy MLWE instance 里临时抽取出来。

所以 Falcon-like world 的正确归类不是“它也用了环，所以和标准 MLWE 差不多”，而是：

> 它是把 GPV-style trapdoor sampling 压进更结构化的 NTRU lattice。  
> 它的危险对象仍然是 relation-aligned short basis。

因此，若某个攻击真恢复出等价 NTRU trapdoor，它最自然长成的是：

```text
equivalent structured short basis
    ->
same sampling power
    ->
effective forgery
```

而不是先绕回一般 search-MLWE。

**Footnote C：NTRU-like inversion world 为什么强调的是 decoding capability，而不是 sampling capability**

再看 NTRU-like decryption / inversion 语境。
这里更自然的 break target 往往不是“模仿 signer 采样”，而是：

1. 对某类公开 map 做 inversion；
2. 对某个 ciphertext/coset 做 decoding；
3. 恢复一个正确代表或 message preimage。

也就是说，这一类构造里最关键的箭头通常是：

```text
good basis / equivalent trapdoor
    ->
decoding or inversion capability
    ->
concrete decryption-style break
```

它和 GPV/Falcon 的差别不在于“有没有短结构”，而在于：

> 最后闭环的 concrete capability 不是 sampling，而是 inversion / decoding。

这正是前面 `break-step map` 里 `inversion / decoding archetype` 的典型落点。

**Footnote D：ML-KEM-style 接口为什么没有把 trapdoor 世界那几根箭头天然交出来**

ML-KEM-style public side 的母语更像：

\[
t=A s+e,
\]

而 ciphertext/encapsulation side 仍然围绕 noisy module relations 组织。

honest secret 的母语不是：

1. a short basis of a public kernel lattice；
2. a structured inversion trapdoor；
3. an equivalent principal generator controlling the public relation。

而是：

1. small module secret；
2. noise terms；
3. decapsulation-specific reconciliation discipline。

所以对 ML-KEM-style world，最稳妥的话不是“它没有 secret structure”。
它当然有 secret structure，只是那种结构首先长成的是：

> module secret inside a noisy average-case relation，

而不是：

> public relation-aligned short basis trapdoor。

于是 `break-step map` 里最先断掉的，通常就是：

```text
good basis candidate
    ->
relation-aligned hidden structure
```

因为公开接口从一开始就没有把 “一组 basis 正在控制这张 public relation 的 kernel/coset” 这件事写进 scheme semantics。

**Footnote E：ML-DSA-style 为什么更接近“短秘密 + transcript discipline”，而不是 trapdoor sampling**

ML-DSA-style public side 更像：

\[
t=A\mathbf{s}_1+\mathbf{s}_2,
\]

随后签名接口又叠加：

1. masking vector；
2. Fiat-Shamir challenge；
3. rejection / abort discipline；
4. hint and norm checks。

这里 honest secret 的核心不是：

1. 一组能直接控制 public relation kernel 的 short basis；
2. 或一个用于 coset Gaussian sampling 的 trapdoor。

而是：

1. small module-lattice secrets；
2. transcript distribution control discipline。

所以如果你硬把 ideal-world 的 short generator → short basis → sampler 语言贴过来，就会立刻错位。
ML-DSA-style world 更自然的问题是：

> 你能否从 transcript、masking、abort 机制里恢复 module secret，  
> 或制造通过验证的响应？

这和 GPV/Falcon 的 trapdoor sampling 问题不是同一个接口类型。

**把五类接口再压成“哪类 hidden object 是 honest design 明写进去的”**

| family | honest design 明写进去的 hidden object | 攻击者若恢复等价对象，是否几乎立刻获得构造能力 |
| --- | --- | --- |
| GPV-style | short basis of a public-kernel lattice | 是，几乎直接得到 sampling / signing leverage |
| Falcon-like | structured NTRU trapdoor basis | 是，几乎直接得到 signer-equivalent sampling leverage |
| NTRU-like inversion | inversion / decoding trapdoor | 是，几乎直接得到 decryption-style leverage |
| ML-KEM-style | small module secret in noisy linear relation | 否，先得解决 average-case secret/noise recovery |
| ML-DSA-style | small module secrets plus transcript discipline | 否，先得解决 transcript-to-secret/forgery mapping |

这张表正是 ideal world / trapdoor world 和 standard MLWE world 的硬分界。

不是说后两类“更神秘”；
而是说它们从设计上就没有把

```text
public relation
    <-controlled by->
short basis trapdoor
```

这层接口内生进去。

**最后把这一页压成一句最该记住的话**

如果你要把这一页变成一条真正能带回原论文的判断句，最好记这个：

> Weber 结果最直接清理的是 arithmetic prerequisite；  
> 但一个具体方案会不会因为“等价短结构被恢复”而立刻产生 concrete break，  
> 取决于该方案是否从设计上就把这种短结构写成 honest secret capability。  
> GPV/Falcon/NTRU-like world 常常是；标准 MLWE/ML-KEM/ML-DSA world 通常不是。

#### 一个 public-key / secret-key / break-target ledger：五类接口到底把什么对象写进了 honest design

前面的 `scheme-interface footnotes` 已经把五类家族放到一张地图里了，但它还偏“分类学”。
如果你真的想判断：

> 某条 ideal-world attack line  
> 到底能不能自然贴到某个具体方案上，

还需要再压一层更硬的账本。

这张 ledger 不再主要问：

1. 公开接口长什么样；
2. honest side 手里拿着什么；

而是继续追问三件更直接的事：

1. 安全游戏里真正的 `break target` 是什么；
2. 若攻击者恢复出某种 `equivalent short structure`，它必须额外模拟出什么能力，才算真的闭环；
3. 如果你想把 `short generator / good basis` 语言硬贴进来，最先断掉的是哪根箭头。

**先给一张 public-key / secret-key / break-target 总账表**

| family | public-key surface 真正公开了什么 | honest secret 真正长什么样 | 安全游戏里的 first break target | 若恢复等价短结构，必须继续模拟什么能力 | 若把 ideal-world 攻击硬贴进来，最先断的箭头 |
| --- | --- | --- | --- | --- | --- |
| GPV-style | \(A\) 与 find short preimage for \(A\mathbf{z}=\mathbf{u}\pmod q\) 这类 relation family | public-kernel lattice 的 short basis | forge：为新 target 生成合法短 witness | short-preimage sampling，且分布仍与 signer/trapdoor interface 相容 | 基本不断在对象层；更常断在 sampling distribution matches the scheme |
| Falcon-like | 结构化 ring relation，例如 \(s_1+s_2h\equiv c\pmod q\) | NTRU lattice 的结构化 short trapdoor basis | forge：获得 signer-equivalent Gaussian witness generation | signer-equivalent sampling power，而不只是某个近似 basis | 常先断在 same object → same admissible sampling law |
| NTRU-like inversion | 结构化 public map、ciphertext/target relation、某个需被反演或解码的 coset | inversion / decoding trapdoor，或与之等价的 good basis | decrypt / invert：恢复 message preimage、近代表、或等价 capability | relation-aligned decoding / inversion capability | 常先断在 good basis → this public map is actually invertible by me |
| ML-KEM-style | noisy module relation，核心公开面更像 \(t=A s+e\) 与 encapsulation relation | 小 module secret、error、reconciliation discipline | recover challenge secret / decapsulation / solve search-style module-LWE | secret/noise recovery 或 challenge-consistent decapsulation power | 最先断在 good basis candidate → relation-aligned hidden structure |
| ML-DSA-style | \(t=A\mathbf{s}_1+\mathbf{s}_2\) 加 transcript / hint / abort interface | 小 module secrets、masking vectors、rejection discipline | forge 或 transcript-consistent secret recovery | transcript-valid witness generation under abort/hint rules | 最先断在 short structure → transcript-aligned hidden capability |

这张表里最值得盯住的不是前两列，而是最后三列。
因为它们决定了一个说法到底是在：

1. 讨论 arithmetic prerequisite；
2. 讨论 relation-aligned capability；
3. 还是已经真的讨论到 concrete break。

**为什么这里故意把 break target 单独列一列**

很多关于 lattice 攻击的误读，都可以压成一句：

> 我已经恢复出一个看起来很强的短结构，  
> 所以应该已经离 break 很近。

但“很强的短结构”与“当前安全游戏里的 break target”之间，往往还隔着一层 scheme-specific capability。

例如：

1. GPV-style 的 break target 更像 **为新消息采样一个合法且分布对的短 witness**；
2. NTRU-like inversion 的 break target 更像 **对公开 map 做 inversion / decoding**；
3. ML-KEM-style 的 break target 更像 **恢复 secret 或对 challenge 做正确 decapsulation**；
4. ML-DSA-style 的 break target 更像 **生成通过 transcript checks 的响应**。

所以教材式判断永远不应停在：

> 这东西像个 trapdoor。

而应继续追问：

> 这个 trapdoor-like object  
> 到底对应当前安全游戏里的哪一种可执行能力？

**把五行表格再拆成三类接口命题**

如果你不想死记五行，可以把上表压回三类命题：

| 你现在真正要问的命题 | 在 GPV/Falcon/NTRU-like world 里通常怎样 | 在 ML-KEM/ML-DSA-style world 里通常怎样 |
| --- | --- | --- |
| public relation 是否本来就接受 short-basis leverage | 是，构造本身常把 relation 与 hidden short structure 配成一对 | 否，公开对象首先是 noisy module relation |
| honest secret 是否本来就可被“等价短结构”描述 | 常是，或至少接近是 | 通常不是；更自然的语言是 small secret + noise / transcript discipline |
| concrete break target 是否正好由该短结构控制 | 常常是 sampling、decoding、inversion 这类 trapdoor capability | 通常不是；还需另证 secret recovery / forgery mapping |

这三类命题真正想纠正的误读是：

> “都是格，所以一个 short structure 的危险程度应该差不多。”

不是。
危险程度首先取决于：

> 该构造是否把 short structure 本身写成了 honest secret capability 的语言。

**Row A：为什么 GPV-style 最容易从“等价短结构”直落到 concrete break**

GPV-style world 的公开关系从第一天就在问：

\[
A\mathbf{z}=\mathbf{u}\pmod q
\]

下的短 witness。

因此 honest secret 与 break target 之间几乎是直接配对的：

```text
hidden short basis
    ->
short-preimage sampling
    ->
forgery capability
```

这里真正敏感的地方不是“有没有一个数论对象被恢复”，而是：

> 你恢复出的等价结构  
> 能不能给出 scheme-admissible 的 sampling law。

所以 GPV-style 行最容易断的，不是对象接口，而是 **分布接口**。

**Row B：为什么 Falcon-like 虽然更结构化，但本质仍是 trapdoor-sampling world**

Falcon-like world 往往让读者误以为：

> 既然它是 NTRU 风格、又用了环，  
> 那它是不是已经更像一般 MLWE 了？

不是。
它的 honest secret 仍然是 **relation-aligned structured short basis**，而不是 noisy module witness。

所以这行 ledger 的关键不是 “用了环”，而是：

> signer capability 本身就长在结构化 short basis 上。

攻击者若恢复等价结构，接下来必须补的不是 “这是不是 relevant object”，而是：

> 我能否用这份等价结构复现 signer 接口允许的 witness distribution？

**Row C：为什么 NTRU-like inversion 行的核心不是 sampling，而是 decoding**

NTRU-like decryption / inversion world 的 honest secret 更接近：

1. good basis；
2. decoding region control；
3. inversion capability。

因此即使你恢复出一个看起来很短的等价结构，它接下来的任务也不是“替代 signer 采样”，而是：

> 在当前 public map 上  
> 真正实现 outsider 本来做不到的 inversion / decoding。

所以这行最容易断的不是 “有没有 short structure”，而是：

> 这份结构是否真的对 **当前这张公开 map** 提供了可执行的 inversion leverage。

**Row D：为什么 ML-KEM-style 的 first break target 根本不在 trapdoor 语义里**

ML-KEM-style public key 的核心表面更像：

\[
t=A s+e.
\]

这意味着 honest secret 的自然语言是：

1. module secret；
2. noise；
3. encapsulation / decapsulation discipline。

所以如果有人把 ideal-world 里的
**short generator -> good basis -> concrete break**
硬贴过来，你现在应该立即问：

> 这份 short structure 到底怎样变成  
> challenge-consistent decapsulation power  
> 或 secret/noise recovery？

如果这一步说不清，那么它还停留在 “找到了某种也许有用的结构” 的层次，而没有进入 ML-KEM 的 break game。

这就是为什么这行最先断的通常是：

```text
good basis candidate
    ->
relation-aligned hidden structure
```

因为公开接口本来就不是用 basis trapdoor 来组织的。

**Row E：为什么 ML-DSA-style 还比 ML-KEM-style 多了一层 transcript discipline**

ML-DSA-style 更容易被误读成：

> 只要恢复了小秘密，或者某种短结构，应该就很接近伪造。

但这仍然太快。
因为它的 concrete break target 不是只要满足一个静态 relation，而是要同时穿过：

1. masking；
2. Fiat-Shamir challenge；
3. rejection / abort；
4. hint / norm checks。

所以这行 ledger 特别值得单列的一句是：

> 即使某种 short structure 与 secret 有关，  
> 你还得证明它能升级成 transcript-valid witness generation。

这比单纯的 secret recovery 还更具体，也更接近真实安全游戏。

**最后把这张 ledger 压成一个五问审计协议**

以后你读到任何“某类 ideal-world attack 也许威胁某个 lattice scheme”的说法，都可以先问五句：

1. public-key surface 公开的首先是 `relation family`，还是 `noisy module instance`？
2. honest secret 是 short basis / trapdoor，还是 small secret / noise / transcript discipline？
3. 当前安全游戏里的 first break target 到底是 `sample`、`decode`、`invert`、`recover secret`，还是 `forge transcript`？
4. 若我恢复出某种等价短结构，它还必须模拟出哪种 capability，才算真的碰到 break target？
5. 若作者没把第 4 句补出来，最先断的是对象对齐、分布对齐，还是 transcript / decapsulation 接口？

只要这五句问顺了，你就不会再把：

> **ideal-side arithmetic gets cleaner**

偷换成：

> `standard module-lattice scheme is now close to a break`.

**把这一页压成一句最该带走的话**

同样是“恢复等价短结构”，不同方案真正关心的不是这个对象本身，而是：

> 它能不能立刻长成当前安全游戏里的 first break target。  
> GPV/Falcon/NTRU-like world 往往可以更直接地长过去；  
> ML-KEM/ML-DSA-style world 通常还隔着 relation、distribution、甚至 transcript discipline 几层新桥。

#### 一个 security-game capability card：为什么“恢复了等价短结构”还不等于“已经赢下当前游戏”

前面的 `public-key / secret-key / break-target ledger` 已经把五类接口放到同一张账本里了。
但很多读者看到那里还是会多问一句：

> 好，我承认不同方案的 `first break target` 不一样。  
> 可如果攻击者真的恢复出一份很像 honest secret 的短结构，  
> 为什么还不能直接说 “这基本就赢了”？

这时最需要再补的一层，不是更多对象学，而是 **安全游戏视角**。

因为同一个“等价短结构”，只有在它能升级成游戏里真正可执行的能力时，才算接近获胜。
否则它还只是：

> 一个与 honest secret 看起来相关的结构性线索，

而不是当前 game 真正承认的 winning capability。

**先给一张 capability card 总表**

| family | 当前安全游戏首先在问什么 | 若攻击者想真正接近获胜，必须补出的 capability | 为什么“恢复等价短结构”本身还不够 | 最常缺的是哪一类桥 |
| --- | --- | --- | --- | --- |
| GPV-style | 能否对新 target 输出合法且分布合规的短 preimage | signer-equivalent short-preimage sampling | 你不只要找一个短 witness，还要匹配 trapdoor interface 允许的 sampling law | 分布对齐 |
| Falcon-like | 能否产生通过验证且分布像 honest signer 的响应 | Gaussian witness generation with signer-equivalent quality | 一个近似好基不自动给出 Falcon 接口要求的 admissible sampling behavior | 分布对齐 |
| NTRU-like inversion | 能否对当前 public map 真正做 inversion / decoding | relation-aligned decoding or inversion power | 结构短不等于它已足以在这张公开 map 上稳定恢复正确 preimage | 对象到能力的映射 |
| ML-KEM-style | 能否恢复 challenge secret、正确 decapsulate，或有效解 search-style module-LWE | challenge-consistent decapsulation or secret/noise recovery | 相关短结构若不能回到 noisy module relation 的 secret 语义，就还没碰到 KEM 游戏目标 | relation / decapsulation 接口 |
| ML-DSA-style | 能否生成通过 transcript、hint、norm、abort 规则的伪造响应 | transcript-valid witness generation under FS-with-aborts discipline | 就算结构与秘密有关，也还没证明它能穿过 challenge + abort + hint 这套在线约束 | transcript 接口 |

这张表最值得盯住的是中间两列：

1. `必须补出的 capability`；
2. `为什么恢复等价短结构本身还不够`。

因为它们回答的是：

> 你现在手里拿着的，  
> 到底是一个 game-relevant capability，  
> 还是一个仍需再翻译一次的结构性对象？

**先把“结构恢复”和“游戏获胜”严格拆开**

教材式最稳妥的拆法应该是：

```text
recover a structure
    ->
show it is capability-bearing for this public interface
    ->
show that capability meets the winning condition of this game
```

这三步里，第一步和第三步经常被读者误以为差不多。
其实它们隔着最关键的第二步：

> 这份结构是不是 **当前公开接口** 真正承认的 hidden capability？

如果这一步没补上，那么：

1. 你也许证明了某种 ideal-side object 更透明；
2. 甚至证明了某种 short basis 很接近 honest secret；
3. 但你仍没证明它能执行游戏真正要求的动作。

**Case A：GPV-style 为什么最像“结构一到手，就快碰到获胜能力”**

GPV-style world 的优势在于：

1. public relation 本来就在问短 preimage；
2. honest secret 本来就是 controlling that relation 的 short basis；
3. winning condition 也正好围绕短 witness generation。

因此它最接近：

```text
equivalent short basis
    ->
equivalent sampling capability
    ->
forgery
```

但即便在这里，也不能少写一句：

> 你恢复出的结构  
> 是否真的支持与 honest signer 足够一致的 sampling law？

这就是为什么 GPV-style world 虽然最接近“结构恢复即将闭环”，却依然常在 **分布对齐** 上承重。

**Case B：Falcon-like 为什么仍然要把“same structure”与“same sampling behavior”分开**

Falcon-like 容易给人一种错觉：

> 只要恢复等价 NTRU trapdoor，  
> 那当然就和 honest signer 一样了。

教材上更严谨的说法应该是：

1. 你也许恢复了某种等价结构化 short basis；
2. 但 signer 的能力不是“拥有一个结构化 short basis”这么静态；
3. 它还包括 `how to sample admissible responses` 这层动态接口。

因此 Falcon-like 的 capability card 最该强调的是：

> **same trapdoor object**
> 并不自动推出
> `same admissible witness distribution`.

这正是它和 generic GPV-style 相像、却又更细的地方。

**Case C：NTRU-like inversion 为什么一定要问“这能力是否作用在这张 map 上”**

NTRU-like inversion 的 winning condition 更像：

1. invert；
2. decode；
3. recover a correct representative or message preimage。

所以此时最关键的不是“结构是否短”，而是：

> 这份结构  
> 是否真的给了我对 **当前 public map** 的有效 inversion power？

若这一步没有明确论证，那么你至多得到了：

1. 一份看起来不错的几何描述；
2. 一种也许对某些相关格有用的 basis；

但还没有证明它已升级成安全游戏承认的“可反演能力”。

因此这一行最常缺的桥是：

```text
good structure
    ->
public-map-aligned inversion capability
```

**Case D：ML-KEM-style 为什么必须把 decapsulation 单独说出来**

在 ML-KEM-style world 里，很多过快推理都会停在：

> 如果某种 short structure 跟 \(s\) 很相关，  
> 那应该已经离 break 很近。

但 KEM 游戏里真正敏感的不是“你是不是看到了一个与 secret 有关的结构”，而是：

1. 你能否恢复 challenge 所需的 secret information；
2. 你能否对 challenge ciphertext 做正确 decapsulation；
3. 或你能否真正求解与游戏等价的 search problem。

这意味着一个非常重要的教材式分界：

> **secret-correlated structure**
> 并不自动等于
> `challenge-consistent decapsulation capability`.

所以 capability card 在 ML-KEM-style 这一行一定要把 `decapsulation` 单列出来。
否则读者太容易把

```text
some relevant short structure
```

误听成

```text
the KEM game is almost won
```

**Case E：ML-DSA-style 为什么必须再穿过 transcript-validity**

ML-DSA-style 比 ML-KEM-style 更容易多出一层错觉：

> 如果我已经摸到 secret 附近的结构，  
> 伪造应该只是时间问题。

但签名游戏在这里并不只是问静态 relation。
它还问：

1. 你的响应是否与 transcript 一致；
2. challenge 是否匹配；
3. abort / rejection 是否被正确满足；
4. hint / norm checks 是否通过。

因此 capability card 在这一行真正想纠正的是：

> **recover structure related to small secrets**
> 与
> **generate transcript-valid forged response**
> 之间，还隔着一整套 online discipline。

这也是为什么 ML-DSA-style 这一行最常缺的桥不是单纯的分布问题，而是：

> transcript interface 本身。

**把五类方案再压成三种常见“假闭环”**

为了防止读者把任何“恢复等价结构”的说法都误判成快要 break，可以把常见误读压成三类：

| 常见假闭环 | 真正遗漏了什么 | 最容易出现在谁身上 |
| --- | --- | --- |
| same structure → same sampling power | witness distribution / admissibility 仍未证明 | GPV-style、Falcon-like |
| good basis → same inversion power | basis 还没与当前 public map 的 inversion interface 对齐 | NTRU-like inversion |
| secret-related structure → game win | 还没穿过 decapsulation 或 transcript discipline | ML-KEM-style、ML-DSA-style |

这张小表最适合在读某些“看起来很吓人”的攻击 claim 时，先做第一轮降噪。

**最后给一个真正可重复使用的 game-audit protocol**

以后你读到任何一句：

> “如果恢复出某种等价短结构，某方案就危险了”

都可以先固定问下面五句：

1. 当前安全游戏真正承认的 `winning action` 是什么？
2. 这份结构能否直接执行那个 action，还是只能提供一部分相关信息？
3. 若不能直接执行，还差的是 sampling、inversion、decapsulation，还是 transcript-validity？
4. 这个缺口是对象没对齐、分布没对齐，还是在线接口没对齐？
5. 如果作者没把第 3 和第 4 句补出来，那他讨论的是 capability，还是只是在讨论 prerequisite？

只要这五句问顺了，你就不容易再把：

> “结构上更透明”

误译成：

> “当前游戏里已经几乎可赢”。

**把这一页压成一句最该带走的话**

安全游戏视角下，真正重要的不是某个短结构看起来多像 honest secret，而是：

> 它是否已经升级成当前 public interface 真正承认的 winning capability。  
> 在 trapdoor-flavored world，这一步常常更短；  
> 在 ML-KEM / ML-DSA-style world，这一步通常还隔着 decapsulation 或 transcript 这类更硬的接口。

#### 一个 relation-by-relation case card：同样说“等价短结构”，五类接口究竟卡在哪个 online step

前面的 `security-game capability card` 已经把问题压到：

> 这份结构是不是已经变成了 winning capability？

但如果你真的去读某篇攻击论文、某篇安全讨论，或者自己在脑中模拟一条 attack line，往往还会立刻遇到一个更具体的问题：

> 好，就算我要检查 capability，  
> 它到底会在 **哪一个 relation / 哪一个 online game step** 上第一次被当场拦住？

这就是这页 `case card` 的作用。

它不再停留在“这一类方案大概在做什么”，而是再往前压一步：

1. 给每类接口挑一个最典型的 relation 或 online step；
2. 明确说明“等价短结构”如果真 relevant，应该先在哪里经受测试；
3. 判断这一步更像 **对象问题**、**分布问题**，还是 **协议接口问题**。

**先给一张 relation-by-relation 总表**

| family | 最典型的 relation / online step | 如果等价短结构真 relevant，它首先必须通过哪一关 | 这一步为什么在 trapdoor world 与 standard MLWE world 表现不同 | 这一步更像哪类问题 |
| --- | --- | --- | --- | --- |
| GPV-style | 给新 target \(\mathbf{u}\) 采样短 preimage \(\mathbf{z}\) 使 \(A\mathbf{z}=\mathbf{u}\pmod q\) | 生成 **分布合规** 的短 witness，而不只是某个满足关系的解 | public relation 本来就围绕 short-preimage sampling 组织；接口天然等 short basis holder 来完成 | 分布问题 |
| Falcon-like | 对消息导出的 target \(c\) 生成满足 \(s_1+s_2h\equiv c\pmod q\) 的合法响应 | 给出 signer-admissible Gaussian witness，而不是仅给一个“足够短”的 pair | 结构化 trapdoor 的确对齐 relation，但 signer 还要求具体采样质量与接受分布 | 分布问题 |
| NTRU-like inversion | 对给定 ciphertext / coset 在当前 public map 下恢复正确代表或 preimage | 证明这份结构真的作用在 **当前这张 map** 的 inversion / decoding 上 | good basis 可能存在，但是否控制这张具体 map 的几何反演仍要单独证明 | 对象到能力映射 |
| ML-KEM-style | 对 challenge ciphertext 做 decapsulation，或对 \(t=A s+e\) 所定义的 noisy relation 做 search recovery | 把结构性信息升级成 challenge-consistent decapsulation / secret recovery | 公开接口首先是 noisy module relation，不是 trapdoor-controlled public relation；结构要先回到 secret/noise 语义 | 协议接口问题 |
| ML-DSA-style | 在给定 commitment / challenge / hint / norm checks 下输出 transcript-valid response | 让相关结构真正穿过 FS-with-aborts 的 online transcript discipline | 这里不是静态短关系，而是动态响应流程；结构必须兼容 challenge、abort、hint 与 norm gate | 协议接口问题 |

这张表最想压实的一件事是：

> “等价短结构有没有用”  
> 不是一个抽象 yes/no 问题；  
> 它会在某个非常具体的 online step 上第一次暴露真假。

**Case 1：GPV-style 被卡的不是“有没有解”，而是“这个解是否像 honest signer 一样被采出来”**

在 GPV-style world，攻击者最不缺的通常不是 relation：

\[
A\mathbf{z}=\mathbf{u}\pmod q.
\]

真正困难的是：

> 你能不能像 trapdoor holder 一样，  
> 按构造允许的方式采出一份短 witness？

所以这一类 case card 的最短版本可以直接写成：

```text
relation:
    A z = u mod q
gate:
    admissible short-preimage sampling
bottleneck:
    law, not mere existence
```

这能防止一个很常见的误读：

> 我已经找到一个短解了，  
> 所以 signer capability 应该差不多有了。

不够。
GPV-style 更常卡在：

> **same relation solution**
> 不自动推出
> `same sampling interface`.

**Case 2：Falcon-like 被卡在“Gaussian signer behavior”，而不只是“存在一个短对”**

Falcon-like 的 relation 看上去只是：

\[
s_1+s_2h\equiv c\pmod q,
\]

于是很多人会过快地想：

> 如果我有一份等价结构化 basis，  
> 那我大概能做出短对。

但这张 case card 要你继续问：

> 你做出的短对  
> 是否真的像 honest signer 那样穿过了它的 Gaussian / admissibility gate？

因此 Falcon-like 的 relation-level 读法应该更偏：

```text
relation:
    s1 + s2 h = c mod q
gate:
    signer-equivalent admissible Gaussian response
bottleneck:
    law of responses, not just shortness
```

所以它虽然和 GPV-style 一样仍属 trapdoor world，但被卡住的关口更像：

> **response law quality**

而不是单纯的“有没有找到 relation witness”。

**Case 3：NTRU-like inversion 被卡在“这份结构是否真能反这张公开 map”**

NTRU-like inversion 的 case card 最容易纠正一个看似合理但很危险的偷换：

> good basis 应该会帮助解码，  
> 所以离 decrypt / invert 不远了。

这里真正该写的是：

```text
relation:
    current public map / current ciphertext coset
gate:
    map-aligned inversion or decoding
bottleneck:
    this basis must control this map
```

也就是说，问题不在于：

1. 某处是否存在一份“还不错”的 basis；

而在于：

2. 这份 basis 是否已经与 **当前这张公开 map** 的几何反演任务对齐。

所以这一行最像 **对象到能力映射**，而不是纯分布或纯协议问题。

**Case 4：ML-KEM-style 被卡在 challenge decapsulation，不是在静态 relation 上卡住**

ML-KEM-style 最值得单独拿出来说的，是它第一次真正“露底”的地方，不在某个静态公式里，而是在 challenge 交互语境里。

教材式最短 case card 应该写成：

```text
relation:
    challenge ciphertext + decapsulation check
gate:
    challenge-consistent decapsulation / secret recovery
bottleneck:
    structure must re-enter the noisy secret semantics
```

这和 trapdoor world 的根本区别在于：

1. public key 不是“等你用 short basis 来控制”的 relation；
2. 它首先是一个 noisy average-case module instance；
3. 所以结构性线索若不先变成 decapsulation-capable information，就还没踩到真正游戏关口。

因此 ML-KEM-style 的 case card 最适合防的误读是：

> **some secret-correlated structure exists**
> 不自动推出
> `the challenge game is close to lost`.

**Case 5：ML-DSA-style 被卡在 transcript online discipline，而不是只卡在 secret relation**

ML-DSA-style 的 relation 级 case card 甚至比 ML-KEM-style 还更“在线”。
最有代表性的不是某个静态方程，而是：

1. commitment 已给出；
2. challenge 已派生；
3. response 必须过 norm / hint / abort；
4. transcript 最后还要整体一致。

所以这行可以压成：

```text
relation:
    online transcript step under FS-with-aborts
gate:
    transcript-valid forged response
bottleneck:
    structure must survive protocol discipline
```

这样一来，读者就不容易再把：

> “它似乎和小秘密有关”

偷换成：

> “那离伪造应该只差一点点”。

因为这里真正最硬的 gate 常常不是对象本身，而是：

> 这份对象能否穿过整套 online transcript discipline。

**把五类 case card 再压成三种“首个拦截点”**

如果你不想每次都从头想五类接口，可以先做一个更短的分类：

| 首个拦截点类型 | 典型代表 | 你最该先问什么 |
| --- | --- | --- |
| law gate | GPV-style、Falcon-like | 这份结构能否给出 **honest-admissible** 的采样行为？ |
| map-alignment gate | NTRU-like inversion | 这份结构是否真的控制了 **当前公开 map** 的反演几何？ |
| online-interface gate | ML-KEM-style、ML-DSA-style | 这份结构能否通过 decapsulation 或 transcript 这类在线协议关口？ |

这张短表最适合拿来做第一轮判断：

> 当前讨论里，最先需要怀疑的到底是 law、map alignment，还是 online interface？

**最后给一个真正可复用的 case-audit 模板**

以后读到任何具体 attack claim，你都可以先用下面五格做一张最小 case card：

```text
[relation case]
relation:
first gate:
needed capability:
bridge type: object / law / protocol
why not a win yet:
```

如果你能先把这五格填出来，再去判断“这个攻击是不是已经逼近具体方案”，误判率会低很多。

**把这一页压成一句最该带走的话**

同样一句“恢复了等价短结构”，在不同方案里第一次被真正检验的地方并不一样：

> GPV/Falcon-like world 常先卡在 witness law；  
> NTRU-like world 常先卡在 public-map alignment；  
> ML-KEM / ML-DSA-style world 常先卡在 decapsulation 或 transcript 这类 online interface。  
> 不把这个“首个拦截点”说清楚，就不能说攻击已经接近闭环。

#### 一个 single-step walkthrough card：把“结构说法第一次露馅”直接贴回最短 game snippet

前面的 `relation-by-relation case card` 已经告诉你：

> 不同接口第一次被真正检验的地方不一样。

但如果你还想把这种判断真正内化成读论文时的手感，就还需要再压一层：

> 不只说“它大概卡在 law / map / protocol”，  
> 而是直接写出一条最短的 game snippet，  
> 然后指出：结构 claim 到底在哪一步第一次露馅。

这页的目标非常具体：

1. 不再泛谈“某类方案通常怎样”；
2. 而是给出最短的 step-by-step 片段；
3. 并在片段里标出 **first failing gate**。

**先给一张 walkthrough 总表**

| family | 最短 game snippet 应该怎么写 | 结构 claim 第一次在哪一步被验收 | 这一步为什么最容易被忽略 | 这一步属于哪类 gate |
| --- | --- | --- | --- | --- |
| GPV-style | target arrives → output z → check relation/shortness → ask whether z came from the right sampling interface | 从“我能找短解”跳到“我能像 trapdoor holder 那样采样”这一步 | 很多人以为找到一个短 witness 就等于拿到了 signer capability | law gate |
| Falcon-like | message → hash/target c → output (s1,s2) → ask whether response is signer-admissible | 从“我能给出短对”跳到“我能给出 honest-signer-admissible response”这一步 | 关系式本身太醒目，容易遮住 response-law 这层要求 | law gate |
| NTRU-like inversion | ciphertext/target arrives → propose basis/trapdoor → try to invert this public map | 从“这份 basis 看起来不错”跳到“它真能反这张 map”这一步 | 读者常把 “good basis” 自动脑补成 “对当前 map 有用” | map-alignment gate |
| ML-KEM-style | challenge ciphertext arrives → use recovered structure → try to decapsulate consistently | 从“结构和 secret 相关”跳到“能完成 challenge-consistent decapsulation”这一步 | “相关”很容易被误听成“足够恢复游戏所需信息” | online-interface gate |
| ML-DSA-style | commitment → challenge → response → hint/norm/abort checks | 从“结构接近小秘密”跳到“能产出 transcript-valid response”这一步 | 很多讨论只盯静态 relation，忽略 online transcript discipline | online-interface gate |

这张表最想训练的是一种阅读动作：

> 每当你看到“恢复等价结构就接近 break”这类句子，  
> 立刻把它压成一个四步或五步的最短片段，  
> 再问：哪一步第一次真的会验它？

**Walkthrough A：GPV-style 不是在“找不找得到解”上第一次露馅，而是在“能不能像 signer 一样采样”上露馅**

最短版本可以写成：

```text
1. challenger gives a target u
2. attacker outputs z with A z = u mod q
3. z is short enough
4. claim: this already behaves like trapdoor signing power
```

真正该画红线的是 3 → 4。

因为 `1-3` 最多说明：

1. 你找到了一份 relation witness；
2. 它还足够短。

但这还没自动说明：

> 你已经具备了 honest trapdoor holder 的 sampling interface。

所以对 GPV-style，最短页边批注应该是：

```text
first failing gate:
short witness -> signer-equivalent sampling law
```

也就是说，GPV-style 真正第一次露馅的地方，不在 relation 是否成立，而在：

> 你是不是只会“找一份解”，  
> 还是已经能“按构造允许的方式采样解”。

**Walkthrough B：Falcon-like 真正露馅的也不是 relation，而是 response-admissibility**

最短片段可以压成：

```text
1. message determines target c
2. attacker outputs (s1, s2) with s1 + s2 h = c mod q
3. pair looks short
4. claim: this is effectively signer capability
```

这里最值得划线的仍是 3 → 4，但原因比 GPV-style 更细一层。

因为 Falcon-like 不只是：

1. relation 成立；
2. witness 很短；

而是还暗含：

> response 的生成方式  
> 足够像 honest signer 会给出的 admissible Gaussian behavior。

所以这行的最短批注更适合写成：

```text
first failing gate:
short pair -> signer-admissible response law
```

这样可以直接防住一句常见误读：

> 我已经能构造一个不错的短对，  
> 所以和真实 signer 大概差不多。

不够。
这句话第一次露馅的地方，就是 `response law`。

**Walkthrough C：NTRU-like inversion 第一次露馅在“这份结构到底是不是这张 map 的能力对象”**

最短片段可以写成：

```text
1. ciphertext or target coset arrives
2. attacker claims a recovered good basis / trapdoor-like structure
3. attacker tries to invert the current public map
4. claim: therefore decryption-style break is near
```

这里最该画线的是 2 → 3。

因为从

> “这份结构在某个相关格上看起来不错”

到

> “它已能反演当前公开 map”

中间还缺了一整层 map alignment。

所以这行的最短批注应该是：

```text
first failing gate:
good structure -> current-map inversion power
```

这比一句模糊的“有好基应该能解码”要硬得多，也更像真正的 game-level 审计。

**Walkthrough D：ML-KEM-style 第一次露馅在 challenge decapsulation，而不是 static secret relation**

最短片段可以压成：

```text
1. challenge ciphertext c* arrives
2. attacker uses some structure correlated with s or e
3. attacker must decapsulate c* consistently
4. claim: KEM game is close to lost
```

这里最该画线的是 2 → 3。

因为 `2` 顶多告诉你：

1. 某种结构与秘密有关；
2. 也许在理想化语境里有进一步价值。

但 KEM 游戏真正承认的是：

> 你能否对 challenge 实例做出正确、语义一致的 decapsulation。

因此这一行最短批注应写成：

```text
first failing gate:
secret-correlated structure -> challenge-consistent decapsulation
```

也正因为如此，ML-KEM-style 被卡住的地方更像：

> online challenge interface

而不是静态对象本身。

**Walkthrough E：ML-DSA-style 第一次露馅在 challenge 之后，而不是 challenge 之前**

最短片段可以压成：

```text
1. commitment is fixed
2. challenge is derived
3. attacker outputs response
4. hint / norm / abort discipline is checked
5. claim: this is now a valid forgery path
```

这里真正最该画线的是 2 → 4 整段，而不只是单独一条 relation。

因为你要穿过的不是：

1. 一个静态方程；

而是：

2. 一整套在 challenge 之后才真正生效的 online discipline。

所以这行最短批注更适合写成：

```text
first failing gate:
structure near secrets -> transcript-valid post-challenge response
```

这会强迫读者把注意力放到：

> 结构说法究竟能不能 survive the protocol after the challenge is fixed.

**把五个 walkthrough 再压成一句真正可操作的话**

如果你以后只想记一条最短规则，可以记这个：

| family class | 先别问什么 | 先问什么 |
| --- | --- | --- |
| GPV/Falcon-like | “它是不是已经差不多会签了？” | “它是不是已经过了 response law 这关？” |
| NTRU-like inversion | “有好基是不是就够了？” | “这份结构是不是已经对齐当前 map？” |
| ML-KEM/ML-DSA-style | “它是不是已经接近 secret 了？” | “它能不能穿过 challenge/decapsulation/transcript 这道在线关口？” |

**最后给一个最短的 walkthrough-audit 模板**

以后你读一条具体安全论断，可以先只写这四格：

```text
[single-step walkthrough]
snippet:
first failing gate:
gate type: law / map / protocol
why readers over-trust this step:
```

这张最小卡片的作用，是把：

> “这句话听起来很强”

重新压成：

> “它第一次真正被检查时，到底会在哪一步露馅？”

**把这一页压成一句最该带走的话**

结构 claim 真正危险不危险，不取决于它听起来多像 honest secret，而取决于：

> 在最短的一条 game snippet 里，  
> 它第一次被验收时能不能过 gate。  
> 过不了 law gate、map-alignment gate 或 online-interface gate 中的第一道，  
> 就还不能说它已经接近 concrete break。

#### 一个 paper-prose alignment card：把 Section 4 里听起来很强的句子直接对齐到 gate

前面的 `single-step walkthrough card` 已经把问题压到了：

> 一条最短的 game snippet 里，  
> 结构说法第一次会在哪一步露馅？

但读者真正回到论文时，看到的并不是我们这几页卡片，而是一句句看上去很自然、很顺的 prose。
真正的困难往往是：

> 论文原句听起来已经很强，  
> 我怎么快速判断  
> 它到底是在说 prerequisite、capability，  
> 还是已经快要说到 game win？

这页 `alignment card` 的目标，就是把“原文式句子”直接贴回本讲义已经搭好的 gate 地图。

注意这里不是在逐字摘录论文原句，而是在总结：

> 当你在 Section 4 一类讨论里读到这种风格的句子时，  
> 你的第一反应应该回跳到哪一张卡片。

**先给一张 prose-to-gate 总表**

| 你在论文里最可能看到的句型风格 | 这句话字面上最像在说什么 | 实际最该先对齐到哪一道 gate | 读者最容易过度解读成什么 | 最该回跳到本讲义哪一页 |
| --- | --- | --- | --- | --- |
| “recovering an equivalent short basis would allow one to sign/sample short witnesses” | 某个短结构似乎已足以给出 signer-like capability | law gate | “那基本已经会签了” | security-game capability card + single-step walkthrough |
| “such a basis would yield an attack on the corresponding inversion task” | 某份 basis 好像会直接变成 inversion/decryption leverage | map-alignment gate | “有好基就当然能反这张公开 map” | relation-by-relation case card |
| “a related short structure would threaten ML-KEM-style instances” | 某种与 secret 相关的结构似乎已足够让 KEM 危险 | online-interface gate，更具体地说是 challenge-consistent decapsulation | “只要和 secret 相关，就已经接近赢 challenge game” | security-game capability card 的 ML-KEM 行 |
| “similar considerations extend to ML-DSA-style signatures” | 某种结构风险似乎也能迁到 FS-with-aborts 世界 | online-interface gate，更具体地说是 post-challenge transcript validity | “和小秘密相关就差不多能伪造” | single-step walkthrough 的 ML-DSA 行 |
| “this makes the attack line more transparent / cleaner / more plausible” | 结构障碍被清掉了，论证路线更通顺 | 很可能仍只停在 prerequisite 层，不一定还没到 capability | “既然更透明了，就已经更接近 concrete break” | public-key / secret-key / break-target ledger |

这张表最重要的不是第一列，而是第三列：

> 你不要顺着句子的语气强度走，  
> 而要先把它丢回正确的 gate。

**Sentence Type A：看起来最像“拿到短结构就能签”的句子**

论文式 prose 往往会写得很顺，例如一种典型读法是：

> 如果恢复出等价 short basis，  
> 那就可以生成满足关系的短响应。

这类句子最危险的地方在于，它把两个层次压得太近了：

1. `recover a structure`；
2. `use it in a signing/sampling interface`。

你真正该做的不是立刻点头，而是先补一句：

```text
alignment:
which sampling law is being claimed here?
```

因为对 GPV-style、Falcon-like world 来说，最先真的会验你的，不是：

1. 你有没有找到某个短 witness；

而是：

2. 你给出的 witness 是否像 honest trapdoor holder 那样被产出。

所以这类 prose 一出现，最该自动回跳的是：

> **law gate**

而不是直接跳到 “forge is near”。

**Sentence Type B：看起来最像“有好基就能解密/反演”的句子**

另一类很常见的论文式句子是：

> 一旦攻击者获得某个等价的 good basis，  
> 相应的 inversion task 就会变得容易。

这类句子最容易骗过读者的地方是：

> “good basis” 这个词本身听起来就已经很像 capability。

但对 NTRU-like inversion world 来说，你最该先问的不是：

1. 这份 basis 好不好；

而是：

2. 它是不是 **当前这张公开 map** 的能力对象？

因此这种句子最该贴到：

```text
alignment:
does this basis already align with the current public map?
```

如果这句没被补出来，那么这类 prose 还停留在：

> “某种结构看起来可能有用”

而不是：

> “对当前 inversion game 已经有了可执行 leverage”。

**Sentence Type C：看起来最像“某种相关结构已经威胁标准 KEM”的句子**

最需要防的另一类句子是：

> 如果某种与 secret 相关的短结构可以被恢复，  
> 那么 ML-KEM-style 实例也会受到威胁。

这类句子的问题不在于它一定错，而在于它太容易跳过真正的在线关口：

1. challenge ciphertext；
2. decapsulation consistency；
3. search/decode equivalence in the actual game。

所以读到这类句子时，最短回跳问题应该是：

```text
alignment:
where exactly is challenge-consistent decapsulation being obtained?
```

若这句话答不出来，那么这类 prose 很可能只是：

> 把 secret-correlated structure  
> 说成了  
> challenge-game capability。

**Sentence Type D：看起来最像“同理也会威胁 ML-DSA-style”的句子**

还有一类句子会更抽象，长得像：

> 类似的结构恢复也会迁移到 module-lattice signature setting。

这类 prose 最容易被误听成：

> 既然和小秘密、短响应有关，  
> 那大概也就接近伪造了。

但 ML-DSA-style 的真正硬门槛不在 “有没有静态相关结构”，而在：

1. commitment 已固定；
2. challenge 已派生；
3. response 必须通过 hint / norm / abort；
4. transcript 要整体一致。

所以这类句子的最短对齐问句应该是：

```text
alignment:
which post-challenge transcript gate is being crossed here?
```

如果作者没有把这一点讲出来，那这句话更像是在讨论：

> structure relevance

而不是：

> transcript-valid forgery capability.

**Sentence Type E：最容易让人把 “cleaner prerequisite” 听成 “stronger attack” 的句子**

还有一类句子表面上很保守，但其实很容易被读者自己加戏。
典型风格像：

> 这使得相关攻击前提更加透明 / 更干净 / 更少条件化。

这类句子字面上其实常常只是在说：

1. arithmetic obstruction 少了；
2. interface 更容易被单独分析；
3. 某些 hidden assumptions 被清掉了。

它未必已经在说：

1. capability 变得可执行；
2. current game 更接近被赢下。

所以这类句子最该贴到的不是某个具体 gate，而是先回到这张判断线上：

```text
prerequisite
    vs
capability
    vs
game win
```

也就是说，它真正最容易被误读成的是：

> “前提更透明”
> 被偷听成
> “攻击更接近闭环”。

**把五类句子再压成一张最短速查卡**

如果你只想把这页带回论文页边，最短可以压成：

```text
short basis -> sign?
    ask: law gate在哪

good basis -> invert?
    ask: map alignment在哪

secret-related structure -> ML-KEM threat?
    ask: decapsulation gate在哪

secret-related structure -> ML-DSA threat?
    ask: post-challenge transcript gate在哪

cleaner attack prerequisite?
    ask: 这句还停在 prerequisite 吗
```

这张速查卡真正想给你的不是知识点，而是一种防误读节奏：

> 先找 gate，  
> 再决定这句话到底在证明什么。

**最后给一个最小 prose-audit 模板**

以后你真回到论文，可以先只写这五格：

```text
[paper prose audit]
prose style:
literal claim:
hidden gate:
bounce card:
what overreading should I resist:
```

如果这五格能先写出来，很多“听起来已经很强”的句子就会自动降到更精确的位置。

**把这一页压成一句最该带走的话**

读 Section 4 这类密码学翻译段落时，最危险的不是看不懂术语，而是听信句子的语气强度。
更稳妥的做法是：

> 先把句子对齐到 `law gate / map-alignment gate / online-interface gate`，  
> 再判断它是在讲 prerequisite、capability，还是 game win。  
> 没有先做这一步，就很容易把“说得像已经能攻破”误听成“真的已经接近攻破”。

#### 一个 source-sentence annotation page：把 Section 4.1 的句子直接批成 `P / C / G`

前面的 `paper-prose alignment card` 已经告诉你：

> 先找 gate，  
> 再决定一句 prose 到底在讲 prerequisite、capability，还是 game win。

但真正回到论文页边时，你往往没有空间再抄整张表。
所以这页只做一件更窄、但更实用的事：

1. 不再分析整类句型；
2. 只给五类接口各写一句更接近页边原句的 source sentence；
3. 直接告诉你页边最短该写成 `P / C / G` 的哪一种。

这里约定三种页边标记：

- `P = prerequisite`：句子只是在说结构障碍更清楚、攻击前提更透明，或某种相关对象更值得怀疑；它还没有给出可执行的 signer / inverter / decapsulator capability。
- `C = capability`：句子已经在暗示某种可执行能力，但还没有穿过当前安全游戏真正验收它的那一道 gate。
- `G = game gate`：句子已经显式碰到“当前 public map / challenge ciphertext / post-challenge transcript”这类真正的验收点。

如果你拿不准，就先问：

```text
这句话到底已经点名了哪个 current target？
```

若它连 `current map`、`challenge ciphertext`、`post-challenge transcript` 这类名词都没露出来，通常还不该急着标 `G`。

**先给一张五句批注总表**

| family | 更接近页边原句的 source sentence | 页边该标哪一层 | 为什么最容易被误读成更强 claim | 页边最短该补什么 | 最该回跳到哪一页 |
| --- | --- | --- | --- | --- | --- |
| GPV-style | “recovering an equivalent short basis should allow short-preimage sampling” | C | sampling 这个词听起来太像“已经会签” | [C] law? witness exists != honest sampling law | single-step walkthrough card |
| Falcon-like | “a related short structure should permit admissible short responses” | C | admissible response 很容易被读成“已经够伪造” | [C] response-law unstated; not yet forgery | security-game capability card |
| NTRU-like inversion | “such a basis would threaten the corresponding inversion task” | C | inversion task 很像已经点到 game target，但 corresponding 仍可能没对齐当前公开 map | [C] which map? alignment still missing | relation-by-relation case card |
| ML-KEM-style | “a secret-related short structure would threaten ML-KEM-style instances” | P | threaten instances 会把“和 secret 相关”偷听成“已能处理 challenge \(c^\*\)” | [P] relation only; no challenge-consistent decapsulation | security-game capability card 的 ML-KEM 行 |
| ML-DSA-style | “similar structure recovery should also endanger module-lattice signatures” | P | endanger signatures 太容易遮住 commitment → challenge → response → checks 这条在线链 | [P] pre-challenge only; no transcript-valid response | single-step walkthrough 的 ML-DSA 行 |

这张表最关键的不是第一列，而是第三列和第五列：

> 你不是在给句子“打情绪分”，  
> 而是在给它标明  
> “这句到底已经 claim 到哪一层，  
> 还缺哪个 noun 才能升到下一层”。

**为什么这里大多数句子只该标成 P 或 C**

Section 4.1 这类密码学翻译段落里，真正应该直接标 `G` 的句子其实很少。
原因不复杂：

1. 一句 prose 若只说 “short basis / related structure / cleaner attack line”；
2. 它通常还没有说明当前公开对象是谁；
3. 也还没有说明当前安全游戏里真正的 winning action 是什么。

所以很多句子听起来很强，实际仍只是在：

```text
P: attack prerequisite becomes cleaner
or
C: some scheme-facing capability is being hinted
```

而不是：

```text
G: the current target in the current game can now be crossed
```

**什么样的句子才更接近真的 G**

如果一句 sentence 真要逼近 `G`，它至少该显式露出下面这类名词：

| world | 若只停在 P/C，句子通常怎么写 | 若真要逼近 G，它至少还得补什么 |
| --- | --- | --- |
| GPV/Falcon-like | “short basis / short response should suffice” | 明写当前 signing / forgery interface 到底怎样验 honest-admissible law |
| NTRU-like inversion | “good basis threatens inversion” | 明写是 **哪张当前公开 map**、哪类 current inversion target |
| ML-KEM-style | “related structure threatens instances” | 明写 challenge ciphertext 与 consistent decapsulation |
| ML-DSA-style | “similar recovery endangers signatures” | 明写 post-challenge transcript、response checks、valid forgery path |

这张表想训练的不是记忆，而是一种批注习惯：

> 如果一句话没有把 `current target noun` 写出来，  
> 那你在页边就先别给它 `G`。

**再给一张真正可拿回论文页边的最小模板**

以后你真回到 Section 4.1，只需写这四格：

```text
[source sentence]
P/C/G:
missing noun before next level:
bounce card:
```

例如对一句

> “a related short structure would threaten ML-KEM-style instances”

你最短可以批成：

```text
P
missing noun: challenge ciphertext / consistent decapsulation
bounce: capability card (ML-KEM row)
```

这张最小模板真正想防的是：

> 句子一旦用了 threaten / allow / endanger 这类动词，  
> 读者就自动脑补成  
> “已经快到 concrete break 了”。

**把这一页压成一句最该带走的话**

Section 4.1 的很多强句子，真正该做的第一页边动作不是“赞同或反对”，而是先写一个字母：

> `P`、`C`、还是 `G`。  
> 然后立刻追问：  
> “它还缺哪个 current target noun，  
> 才配升到下一层？”

### 10.5 把整条链翻译成密码学语言

```text
Weber conjecture verified for k <= 12
    ->
relevant real cyclotomic subfields have class number one
    ->
all ideals principal, torsion-free modules free
    ->
codifferent / embedding / quotient-module structure simplify
    ->
RLWE-PLWE comparisons and MLWE reductions lose conditional baggage
    ->
for current NIST parameter families, these simplifications are unconditional
```

把前面 `10.2`、`10.3`、`10.4` 压成一张对照表，可以更清楚地看到“哪些东西被 class number one 清理了，哪些东西并没有被它自动解决”：

| 维度 | ideal world / RLWE-proof side | module world / MLWE-standard side | class number one 改变了什么 | class number one 没有自动改变什么 |
| --- | --- | --- | --- | --- |
| 基础对象 | single ideal、principal generator、dual ideal | \(R_q^d\) 上的向量、子模、线性系统 | 去掉 ideal-class twisting | 不会把 module instance 压成单个 ideal |
| 几何语言 | canonical embedding + ideal lattice + codifferent | free module coordinates + noisy linear algebra | codifferent / inverse ideal 更显式 | 不会消掉 LWE 噪声本身 |
| 结构障碍 | nonprincipal ideals、nonfree rank-one modules | module freeness、quotient-module cleanliness | principality 与 freeness 变成无条件事实 | 不会白送 secret recovery |
| 证明收益 | RLWE/PLWE bridge 更少条件假设 | MLWE reduction 的底层 module 假设更干净 | 清理 reduction 的地基层包袱 | 不等于直接增强具体安全界 |
| 攻击入口 | PIP / SGP / short generator | search / decision MLWE on \(R_q^d\) | 让 ideal attack prerequisite 更透明 | 没有自动给出从 MLWE 到 PIP 的 reduction |
| 对 NIST 参数的含义 | 相关 real subfield 的数论前提已无条件 | ML-KEM / ML-DSA 仍是 module-LWE 问题 | “结构前提更稳固” 变成现实结论 | 不等于“标准因此自动变弱” |

这就是为什么本文标题虽然是 “Module Lattice Security”，正文却大段在谈 class groups、cyclotomic units 与 Bernoulli numbers：它真正做的是在 reduction 的地基层清理假设。

## 11. 难点清单、学习检查与延伸阅读

真正会卡住读者的地方，不是 theorem 编号多，而是同一篇论文里同时混合了：

- algebraic objects；
- representation-theoretic slicing；
- explicit computation；
- cryptographic translation。

下面先把最难点清单写成学习路标。

### 11.1 最难的 10 个定义

1. fractional ideal：因为它不是“有分母的元素集合”这么简单，而是 ideal group 成群所必需的对象。
2. class group：因为它是 quotient of ideal groups，不是单个失败实例。
3. maximal real subfield：因为它既是 cyclotomic field 的子域，又是本文主结论真正发生的地方。
4. idempotent \(e_\chi\)：因为它来自群环，而不是普通线性代数投影矩阵。
5. eigenspace \(\mathrm{Cl}(K_k^+)[\ell]^{e_\chi}\)：因为它把 class-group torsion 与 Galois representation 绑在一起。
6. cyclotomic unit：因为它看似只是一类特殊单位，实际却是 Stage 1 sieve 的入口。
7. generalized Bernoulli number \(B_{1,\psi}\)：因为它不是经典实分析里的 Bernoulli number，而是 character-twisted 版本。
8. conductor of a character：因为它决定你在 tower 的哪一层工作。
9. codifferent：因为它在 RLWE 几何里自然出现，但很多密码学读者从未系统学过。
10. free module over \(\mathcal{O}_K\)：因为它和“有坐标基”密切相关，却比向量空间微妙得多。

### 11.2 最难的 10 个证明节点

1. 为什么 Theorem 3.1 的 Wieferich-style 条件足以排除 \(\ell \mid h_k^+\)。
2. 为什么 Proposition 3.1 可以沿 \(\mathbb{Z}_2\)-tower 消掉低阶 characters。
3. 为什么这个归纳不循环，尤其是 \(k=9 \to 10\) 的接力。
4. 为什么 full-order characters 才是 Section 3.2 之后唯一残余。
5. 为什么 Herbrand bridge 只需处理这些 full-order characters。
6. 为什么 \(\ell \equiv -1 \pmod{n}\) 时仍有合适的 orbit-norm 版本可用。
7. 为什么 Proposition 3.2 真能把所有 odd prime divisors 压进一个有限集合。
8. 为什么 Lemma 3.4 的 bounds 足够让 candidate search 变成现实 computation。
9. 为什么 `3.5 Optimizations` 可以把多次整数分解替换成一次分解或 GCD。
10. 为什么 Section 4 的 module freeness 叙事确实从 \(h_k^+ = 1\) 无条件推出。

### 11.3 最难的 10 个公式

1. \(\mathrm{Cl}(K) = I_K / P_K\)。
2. \(K_n^+ = \mathbb{Q}(\zeta_{2^{n+2}} + \zeta_{2^{n+2}}^{-1})\)。
3. \(\operatorname{Tr}_{L/K}(\alpha)\) 与 \(\operatorname{N}_{L/K}(\alpha)\) 的共轭定义。
4. eigenspace decomposition 相关的 \(e_\chi\) 投影公式。
5. Sinnott formula 把 \(h_k^+\) 接到 cyclotomic unit index 的那一步。
6. generalized Bernoulli number 的 character 求和定义。
7. Herbrand divisibility：\(\ell \mid \operatorname{N}(B_{1,\psi})\)。
8. Theorem 3.1 中 \(\ell \equiv \pm 1 \pmod{2^{k-1}}\) 的 congruence filter。
9. Algorithm 1 中 \(\xi_5^{(\ell-1)/2^{k-1}} \bmod \mathfrak{l}\) 的 residue test。
10. Section 4 中 \(R_q = \mathbb{Z}_q[x]/(x^{256}+1)\) 与 \(\mathcal{O}_{K_9}\) quotient 的对应。

### 11.4 最难的 10 个技巧

1. 不直接算 class number，而是先找“若 \(\ell \mid h_k^+\) 会发生什么”。
2. 先排小素数和错误同余类，再谈深层 eigenspace。
3. 用 tower norm-coherence 做 character-order 级别的消元。
4. 用 Herbrand theorem 把抽象 class-group 信息落到整数整除。
5. 用 orbit pairing 降低需要计算的 character 数量。
6. 用 bounds 控制 Bernoulli norms 的大小，而不是盲目大整数分解。
7. 用 GCD over \(\mathbb{F}_\ell\) 替换重复整数分解。
8. 把 ideal-lattice risk 与 module-lattice standards 分开讨论。
9. 把 class number one 同时翻译成 principal ideal / free module / PLWE equivalence 三种语言。
10. 始终把每一步证明问成“现在排除的是哪一类 \(\ell\)”。

### 11.5 难点学习法：不要把难点清单当作背诵表

上面的 `10 个定义 / 10 个证明 / 10 个公式 / 10 个技巧` 不是考试提纲。
它们更像一张 debugging map：
当你读原论文某一段突然断线时，先判断自己断在哪一类对象上，再回到对应 notebook 补证据链。

The brief’s wording is literal here: 10 hardest definitions / 10 hardest proofs / 10 hardest formulas / 10 hardest techniques.

最稳的做法是把每个难点都拆成三问：

1. `它为什么难`：是对象陌生、量词复杂、公式来源不明，还是 proof interface 断了？
2. `怎样理解它`：先把它压成 toy model、workflow、ledger，还是直接看 theorem dependency？
3. `回跳到哪一页`：回到数学补丁、Section 2 notebook、Section 3 proof notebook，还是 Section 4 reduction/attack ledger？

下面这张矩阵给一个具体读法。

| 难点类型 | 为什么难 | 最优理解方式 | 回跳位置 |
| --- | --- | --- | --- |
| definition difficulty | 名词本身来自代数数论，读者容易只背定义、不知道它解决什么失败 | 先问“它修复了什么失败”，再看 toy example，最后再读 formal definition | 5.2–5.17，尤其是 ideal / class group / codifferent / monogenicity / short generator |
| proof-node difficulty | 一步 proof 往往同时调用前一节 theorem、外部定理和当前归纳假设 | 把每个 proof node 写成 input → theorem used → output → next step | 8.7、8.8、8.9、8.10 |
| formula difficulty | 公式表面短，但背后可能藏着 quotient、norm、character twist 或 local-field branch | 先说清公式左右两边各住在哪个对象里，再问它是 definition、bridge 还是 certificate | 6 的 glossary，7.5 的 idempotent notebook，9.10 的 residue certificate pages |
| algorithm difficulty | Algorithm 1 混合了 candidate generation、candidate elimination 和优化 trick | 强制拆成 Phase A / Phase B；每相只写 input、work、output、failure mode | 7.10、9.1–9.4、9.10 |
| local-certificate difficulty | 一条 \(\mathfrak l\)-row 容易被误听成一个 prime verdict | 先写 branch、lift、\(g\)、\(E_{\mathfrak l}\)，再数 rows；没有全量 rows 就只能写 boundary record | 9.8、9.10、k=10,\ell=12289 boundary page |
| cryptographic-translation difficulty | 数论结论与 scheme security 中间隔着 reduction interface、distribution law 和 game capability | 不问“是不是更安全/更危险”，先问它清掉的是 prerequisite、capability 还是 game-win gate | 10.2–10.5，尤其是 reduction invariant 与 attack interface ledger |

**一条具体使用规则**

如果你在读原论文时卡住，不要先说“我不懂代数数论”。
先把卡点归类成下面四种之一：

```text
object unknown
    -> go back to Section 5 / 6

proof arrow unclear
    -> go back to Section 8 theorem notebooks

computation certificate unclear
    -> go back to Section 9 row / boundary pages

cryptographic meaning unclear
    -> go back to Section 10 reduction / attack ledgers
```

这条规则的作用是把“看不懂”变成一个可定位问题。
比如你看到
\[
\ell\mid \operatorname{N}(B_{1,\psi})
\]
时卡住，就不要立刻去翻完整 Herbrand 理论。
先问：

1. 这里的 \(\psi\) 是从哪个 \(\chi\) 扭出来的？
2. 这条式子是在生成 candidate，还是在 elimination prime？
3. 当前是否已经由 Corollary 3.1 把 low-order characters 清掉？

如果这三问能回答，说明你卡的不是公式本身，而是 theorem-chain bookkeeping。
回跳 `8.8` 会比重读整本 cyclotomic fields 教材更有效。

再比如你看到
\[
\xi_5^{E_{\mathfrak l}}\bmod \mathfrak l
\]
时卡住，不要先纠结 Sage 语法。
先问：

1. branch 是 `+` 还是 `-`？
2. lift 是 clean 还是 dirty？
3. \(g\) 有多少条 rows？
4. 当前写的是 complete counter、partial walkthrough，还是 boundary record？

如果第 `4` 问答不上来，就说明你卡的不是 algebra，而是 proof-state discipline。
这正是 `9.10` 后半与 `k=10,\ell=12289` 边界页要训练的东西。

**这张学习法矩阵最该防的误区**

不要把所有难点都压成“背景不够”。
在这篇论文里，至少有四种完全不同的难：

1. `background gap`：不知道 class group / codifferent 是什么。
2. `bridge gap`：知道对象，但不知道它怎样接到 theorem。
3. `certificate gap`：知道公式，但不知道它是否已经足够推出 prime verdict。
4. `translation gap`：知道数论结论，但不知道它在 reduction 或 security game 里清掉的是哪层 debt。

这四种 gap 的修复方式不同。
背景 gap 靠定义和 toy example；bridge gap 靠 theorem notebook；certificate gap 靠 row counter 和 boundary record；translation gap 靠 reduction invariant 与 attack interface ledger。
把它们混成一句“我代数不够”，会让学习路线重新变得不可执行。

### 11.6 学习检查：按章节做自测

**Chapter 1-4：导读与阅读路线**

1. 概念题：为什么本文真正证明的是 \(h_k^+=1\)，而不是 \(K_k\) 的 full class number？
2. 思考题：如果你只关心密码学应用，为什么仍不能完全跳过 class group language？

**Chapter 5-6：数学补丁与符号表**

1. 概念题：用你自己的话解释 ideal 与 principal ideal 的区别。
2. 推导题：计算 \(a+b\sqrt{2}\) 在 \(\mathbb{Q}(\sqrt{2})\) 中的 trace 和 norm。
3. 证明题：说明 \(h(K)=1\) 为什么等价于 every fractional ideal is principal。
4. 概念题：解释 codifferent、different 与 discriminant 各自在测什么，不要把它们混成同一个对象。
5. 思考题：为什么 monogenicity 解决的是 presentation 问题，而 class number one 解决的是 ideal-class 问题？
6. 思考题：为什么 “找到某个 generator” 和 “找到 short generator” 是两道不同的问题，units 在中间起了什么作用？
7. 推导题：在 \(K=\mathbb{Q}(\sqrt2)\) 中写出 basis \(\{1,\sqrt2\}\) 到 canonical embedding 的矩阵，并解释为什么这已经足以说明 “monogenic” 不等于 “RLWE 与 PLWE 自动同形”。
8. 思考题：把 \(f(\zeta_{512})\) 写成 coefficient vector 与 embedding vector 两种坐标，并说明为什么这已经足以看出 \(R_q\) 的多项式表示与 canonical embedding 不是同一套几何语言。

**Chapter 7：Section 2 精读**

1. 概念题：为什么 Section 2.5 的 eigenspace decomposition 不是可有可无的 representation theory 装饰？
2. 思考题：Section 2.6 与 2.7 分别给 Section 3.1 和 3.3 提供了什么接口？

**Chapter 8-9：Section 3 与算法**

1. 概念题：Theorem 3.1 排除的是哪两类风险？
2. 证明题：解释 Proposition 3.1 与 Corollary 3.1 如何把风险压到 full-order characters。
3. 推导题：把 Algorithm 1 拆成 “candidate generation” 与 “candidate elimination” 两相，并说明每相的输入输出。
4. 开放题：如果想把方法推到 \(k>12\)，现实瓶颈更可能出现在 bounds、整数分解，还是 residue tests？
5. 实战题：以 `k=10,\ell=12289` 为例，写一张 `computation boundary record`，要求只写 branch、lift、\(g\)、\(E_{\mathfrak l}\) 与阻塞点；同时明确列出哪一句 prime verdict 现在还不能写。
6. 实战题：为一个没有快速 row 输出的 `k=11` 或 `k=12` 候选 prime 填一张 `no-fast-output audit card`，要求至少写出 `candidate source / branch evidence / lift evidence / expected row count / exponent obligation / blocking step / allowed claim / forbidden claim` 八格。

**Chapter 10：密码学翻译层**

1. 概念题：为什么 module freeness 是 MLWE reduction 的技术前提？
2. 思考题：为什么 PIP-based 攻击线没有自动推出 ML-KEM / ML-DSA 失陷？
3. 概念题：为什么 codifferent 会进入 RLWE 的 dual / error / reduction 语言，而在 PLWE 里你首先看到的却是 coefficient basis？
4. 思考题：为什么 “class number one + monogenicity” 仍然不够推出 RLWE 与 PLWE 自动完全等价？
5. 思考题：PIP 与 SGP 的区别是什么，为什么后者会把 units 与 log-unit lattice 拉进攻击线？
6. 开放题：class number one 带来的“证明更干净”与“ideal-lattice 攻击条件更清楚”之间是否存在张力？
7. 推导题：把同一个 RLWE 样本分别翻译成 coefficient presentation、\(\mathcal{O}_K/q\mathcal{O}_K\)、canonical embedding、dual/codifferent 四套语言，并说明每一层最自然回答的是什么问题。
8. 推导题：试着把 Section 4.2 的 reduction 阅读流程拆成 instance interface → arithmetic lift → freeness legitimacy → embedding geometry → dual compatibility → executable shell 六层，并标明其中哪两层最直接受 \(h_k^+=1\) 影响。
9. 实战题：任选 Section 4.2 的一小段 prose，给它加上 `instance-writing / arithmetic-lift / freeness-legitimacy / geometry-lift / dual-compatibility / shell-return` 之一的页边标签，并解释你的判断。
10. 实战题：参照本节的 synthetic skeleton，自己写出一条六段式的 reduction prose 摘要，并给每段打上对应 tag。
11. 实战题：用本节的 worksheet 回看 Section 4.2 任意两段，至少写出主 tag、`this paragraph is for ...` 短语，以及它是否直接受 \(h_k^+=1\) 影响。
12. 概念题：在 principal ideal → generator → short generator → short basis → trapdoor leverage → concrete break 这条账本里，PIP、SGP、scheme interface 各自真正接管的是哪几步？
13. 思考题：为什么 short generator → good basis → preimage sampling / decoding leverage 在 GPV- or Falcon-like world 里是自然接口，而在标准 MLWE world 里却还缺两层桥？
14. 概念题：比较 `A\mathbf{z}=\mathbf{u}\pmod q`、`u+vh\equiv c\pmod q` 和 `(A,b=As+e\bmod q)` 这三类 relation，分别说明它们最自然绑定的 hidden object 是什么，以及为什么只有前两类天然吃 short-basis leverage。
15. 概念题：把 inversion / decoding、signing / preimage sampling、equivalent-secret reconstruction 这三类 break archetype 分开说明；每一类里 `short generator`、`good basis`、`concrete capability` 分别扮演什么角色？
16. 思考题：为什么 short generator → concrete break 不是一个定理，而是一串需要逐箭头补证明的接口命题？如果目标是标准 MLWE 实例，你认为最先断的是哪一根箭头，为什么？
17. 概念题：比较 GPV-style、Falcon-like、NTRU-like、ML-KEM-style、ML-DSA-style 这五类接口，分别说出 honest side 明写进去的 hidden object 是什么，以及为什么前 3 类比后 2 类更自然吃 relation-aligned hidden structure → concrete capability 这条链。
18. 思考题：如果某个论文声称“恢复等价短结构即可 break 某 lattice scheme”，你现在应该先检查哪三件事，才能判断它是在 trapdoor world 里闭环，还是在 standard MLWE world 里偷换了接口？
19. 概念题：对 `instance-writing / arithmetic-lift / freeness-legitimacy / geometry-lift / dual-compatibility / shell-return` 六个 tags，各写出一条“这一层真正背着的 proof obligation 是什么”，不要只重复它在表面上谈什么对象。
20. 实战题：任选 Section 4.2 的一段 prose，分别写出它的主 tag、它若失败 proof 最先会坏在哪、以及 \(h_k^+=1\) 是否直接替这段免掉了一层验证负担。
21. 概念题：区分 `oracle-family invariant` 与 `distribution-fidelity invariant`。为什么 “最后还能写成 \(A,b=As+e\bmod q\)” 仍不足以保证 reduction 可以合法调用 average-case solver？
22. 实战题：任选 Section 4.2 的两段 prose，各写出它们主要在保护哪条 invariant；若其中一条 invariant 在这两段之间断掉，最终更可能坏在 oracle 调用、坐标合法性、几何语义，还是 duality 兼容上？
23. 概念题：为什么 `object transport` 与 `distribution transport` 在 reduction 里必须分开审计？请结合 `arithmetic-lift`、`geometry-lift` 与 `shell-return` 各举一句最可能出错的地方。
24. 思考题：如果某一步经过 basis change 或 embedding reinterpretation 后，样本仍能写成 \(A,b=As+e\bmod q\)，你还需要额外确认哪两件事，才能说 `distribution-fidelity invariant` 没断？
25. 实战题：把 Section 4.2 的一段自然段归到 `五步审计卡` 里的某一步，并写出这一步的主 tag、主 invariant、若失败 proof 最先坏在哪。
26. 概念题：为什么 `Step 2` 和 `Step 4` 比 `Step 1/3/5` 更直接吃到 \(h_k^+=1\)？请用 “它替你免掉了哪类额外验证负担” 的语言回答，而不是只说 “因为 class number one”。
27. 实战题：从 Section 4.2 任意挑一句带有 `regard as / identify with / under the canonical embedding / via the trace pairing / thus given a solver` 风格的 prose，先用 phrase decoder 译成一句 hidden claim，再写出它对应的主 tag 与下一句你要追问的问题。
28. 概念题：为什么 reduction prose 里最像“过渡句”的句子，反而常常在偷偷搬运最重的 claim？请分别用一个 `identify with` 句型和一个 `thus solver` 句型说明。
29. 实战题：任选 Section 4.2 的一个自然段，只用 `literal / imports / bounce / H=1 / transport / watch` 六格给它做批注；要求每格都不超过一句话。
30. 概念题：为什么 `literal claim` 与 `hidden import` 必须分开写？请各用一个 `choose coordinates` 句型和一个 `thus given a solver` 句型说明，如果把两格混写，最容易漏掉什么。
31. 实战题：在 Section 4.2 中各找一段 `H=1: direct` 与 `H=1: indirect` 的 prose 原型，分别说明它们的区别到底在 “作者此处正在免掉哪层 debt” 还是 “作者只是在消费 earlier gain”。
32. 概念题：为什么 `transport: both` 往往是最危险的一类批注？请说明它与单纯的 `object transport` 相比，多承担了哪一层 oracle-facing burden。
33. 概念题：用 `public-key surface / honest secret / first break target` 三列，分别比较 GPV-style、Falcon-like、NTRU-like、ML-KEM-style、ML-DSA-style 五类接口；要求不要只写“trapdoor”或“secret”，而要写成当前安全游戏里真正相关的能力对象。
34. 思考题：为什么同样是“恢复等价短结构”，GPV-style 更自然通向 `short-preimage sampling`，而 ML-KEM-style 还必须补 `challenge-consistent decapsulation` 这一层？请用 ledger 里的两列分别说明。
35. 实战题：任选一篇声称 ideal-world attack 可能威胁标准 lattice scheme 的论文或说法，先用 `public-key / secret-key / break-target ledger` 填出五格，再指出最先缺失的桥是对象对齐、分布对齐，还是 transcript/decapsulation 接口。
36. 概念题：为什么 `Weber result cleans arithmetic prerequisite` 与 `scheme is close to a break` 之间不能直接画等号？请至少结合一行 trapdoor world 和一行 standard MLWE world 的 ledger 来回答。
37. 概念题：比较 `equivalent short structure`、`capability-bearing structure` 与 `winning capability` 这三个层次；请各给出一个 GPV/Falcon-like world 的例子和一个 ML-KEM/ML-DSA-style world 的例子。
38. 思考题：为什么 GPV-style 和 Falcon-like 最容易在 same structure → same admissible sampling law 这一步失手，而 ML-KEM-style 更常在 secret-related structure → challenge-consistent decapsulation 这一步失手？
39. 实战题：任选一个你熟悉的安全游戏，把“恢复出某种相关结构”的说法翻译成 `winning action / missing capability / missing bridge type` 三项表述；要求明确指出缺的是对象、分布，还是在线接口。
40. 概念题：为什么教材里必须把 `prerequisite`、`capability`、`game win` 这三层分开，而不能只用一句“攻击更接近闭环了”含混带过？
41. 实战题：分别给 GPV-style、NTRU-like inversion、ML-KEM-style 各写一张五格 `relation case` 小卡；要求明确写出它们各自的 `first gate`。
42. 概念题：为什么 Falcon-like 与 GPV-style 虽然都首先卡在分布层，但 Falcon-like 更适合写成“admissible Gaussian response gate”，而不是泛泛写成“有个 short witness 就行”？
43. 思考题：为什么 ML-DSA-style 的首个拦截点更像 transcript online discipline，而不是静态的 secret relation？请用 commitment → challenge → response → checks 这条链回答。
44. 实战题：任选一个“恢复结构即可威胁标准 lattice scheme”的说法，用 `relation / first gate / needed capability / bridge type / why not a win yet` 五格重写，并指出它缺的第一层到底是哪一格。
45. 实战题：给 GPV-style 写一个四步 `single-step walkthrough`，并明确指出为什么 short witness → signer-equivalent sampling law 才是第一次真正的 gate。
46. 实战题：给 ML-KEM-style 写一个 challenge ciphertext → structure claim → decapsulation 的最短片段，并说明为什么 `secret-correlated` 还不等于 `challenge-consistent`。
47. 概念题：为什么 NTRU-like inversion 比起分布问题，更像 good structure → current-map inversion power 的 map-alignment gate？
48. 思考题：如果某条攻击论断在你写出的 `single-step walkthrough` 里只能停在第 2 步，为什么它更像 prerequisite claim，而不是 capability 或 game-win claim？
49. 实战题：任选一类你在 Section 4 类讨论里最容易被吓到的句型，把它填进 `paper prose audit` 五格模板，并明确写出 `hidden gate`。
50. 概念题：为什么 “recovering an equivalent short basis would allow signing” 这类句子，首先应该回跳到 `law gate`，而不是直接回跳到 `forge` 结论？
51. 思考题：为什么 “this would threaten ML-KEM-style instances” 这类 prose 若没有说明 challenge-consistent decapsulation，是不够资格直接被听成 game-level claim 的？
52. 实战题：任选一条你认为“听起来像已经很强”的 prose 风格句子，把它分别重写成 `prerequisite claim`、`capability claim`、`game-win claim` 三版，说明区别在哪里。
53. 实战题：分别给 GPV-style、Falcon-like、NTRU-like inversion、ML-KEM-style、ML-DSA-style 各写一句更接近页边原句的 source sentence，并给每句标上 `P / C / G`；要求每句再补一条不超过一行的页边批注。
54. 概念题：为什么 “allow short-preimage sampling” 与 “already near forgery” 之间，仍然隔着一条 `honest-admissible law` 的门槛？请分别用 GPV-style 与 Falcon-like 各举一句说明。
55. 思考题：如果一句 ML-KEM-style prose 只说 “related short structure threatens instances”，你在页边至少应该补哪两个 noun，才可以认真讨论它是否逼近 `G`？
56. 实战题：任选一条你熟悉的 Section 4.1 风格句子，先按 `P` 批注一次；再把它改写成真正更接近 `C` 的版本；最后说明若要升到 `G`，还必须把哪一个 current target 明写出来。
57. 实战题：对 `worked paragraph-audit page` 里的五个原型段落任选四个，分别填一张六格卡，并说明为什么其中只有 geometry paragraph 最自然地写成 `transport: law`。
58. 概念题：为什么 arithmetic-lift paragraph 更适合标 `H=1: indirect`，而 freeness / dual-compatibility paragraphs 更适合标 `H=1: direct`？请用“作者此处还欠哪层 debt”来回答。
59. 实战题：自己写一句 `thus given a solver for Module-LWE ...` 风格的 reduction 收束句，然后分别给它写出 `literal`、`imports` 与 `watch`；要求明确指出为什么 `same shell` 还不等于 `same law`。
60. 实战题：把 `paragraph-chain notebook` 的六段 strip 各写一条 `received debt / paid here / passed next` 三格卡；要求至少指出其中两处“上一段只是把问题送来，并没有真正解决”的位置。
61. 概念题：为什么 arithmetic-lift → freeness-legitimacy 不能合并读成一句“重新解释对象并顺便选了基”？请分别说明这两段各自真正替 proof 还的是哪一层 debt。
62. 思考题：为什么 `shell-return` 最好反着读？请用 “它要 cash out 到哪个 solver promise、又必须继承前面哪两层 already-paid debt” 的语言回答。
63. 实战题：为 `reduction-certificate notebook` 里的六个 proof jobs 各写一句 `certificate:`，要求每句都不超过一行，并明确区分 `certificate 4` 与 `certificate 6` 不是同一件事。
64. 概念题：为什么 `certificate 3` 与 `certificate 5` 才是 \(h_k^+=1\) 两次最直接落地的位置？请分别用 “它替作者免掉了哪层 debt” 的语言回答。
65. 思考题：任选一段你觉得像“阶段收束”的 Section 4.2 prose，给它填 `certificate / why earned here / which next step now becomes legal / if missing, next step is fake because` 四格卡。
66. 实战题：用 `reduction I/O notebook` 的四格 `in / move / out / what must still be invariant`，分别给 `arithmetic realization`、`coordinate legitimization`、`oracle cash-out` 三步各写一张卡。
67. 概念题：为什么 `Step 2` 的 `out` 最多只是 “有 arithmetic parent 的对象”，而不能直接写成 “合法 free coordinates”？请用 I/O 语言而不是直觉语言回答。
68. 思考题：为什么后半段最该盯住 `out-object` 而不是句子表面语气？请分别比较 `geometry reinterpretation`、`dual alignment`、`oracle cash-out` 三步里最容易被偷换的 `out`。
69. 实战题：用 `reduction route sheet` 的七格 `route step / trying to obtain / before / after / not-yet / H=1 / main invariant`，分别给 `Step 2`、`Step 3`、`Step 5` 各写一张卡；要求至少指出一步“看起来像重新命名，实际上在还 H=1 债”。
70. 概念题：为什么 `free coordinates` 与 `geometric shortness` 之间必须保留整整一箭头，而不能压成一句 “选了基以后 small 语义就跟着来了”？请用 `after / not-yet` 语言回答。
71. 实战题：自己写三句 synthetic reduction prose，其中第二句必须扮演 `dual alignment`；然后分别标出这三句对应的 route arrow，并说明若删掉第二句，会导致哪一种 `duality-compatibility` 或 `distribution-fidelity` 偷换。
72. 实战题：对 `route-to-paragraph margin sheet` 的六个 synthetic paragraphs，分别写一句 `why not previous` 与一句 `why not next`；要求至少指出一处“这一段看上去像 object rewrite，实际上已经切到 law transport”的位置。
73. 概念题：在六段 strip 里，哪一段最接近 `coefficient-shell presentation`，哪一段是 freeness 的直接落点，哪一段是 dual/codifferent 的直接落点，哪一段是 distribution-fidelity 的最终 cash-out？请各用一句理由回答。
74. 实战题：自己写两段相邻的 synthetic reduction prose，其中第一段只能做到 `arithmetic hosting`，第二段必须完成 `freeness discharge`；然后分别写出这两段的 `failure mode`，说明为什么不能把它们并成一句“重新解释对象并顺便选了基”。
75. 概念题：为什么 `D` 更该染成 `distribution-fidelity` 的前置主战场，而不是 `object rewrite`？请用 “这一段在保哪种 law，而不是哪种壳” 的语言回答。
76. 实战题：对 `four-burden overlay sheet` 的六段 strip，分别写出 `main burden / secondary burden / definitely not / if miscolored, next break` 四格；要求至少指出一段“看起来像抽象术语补充，实际上在做 glue burden”的位置。
77. 思考题：为什么 `F` 若被染成 `presentation-main`，整条 reduction 会被误降格成“公式终于长得像 MLWE”？请明确写出这里丢掉的是哪一层 solver promise。
78. 实战题：对 `worked reduction-by-reduction audit page` 的 A-F 六段，各写一句 `first downstream collapse if false`；要求至少指出一处“真正先坏掉的不是当前句，而是下一段里第一句看起来很自然的合法过渡句”。
79. 概念题：为什么 `Paragraph C` 若失败，`Paragraph D` 最先失去合法性的不是 “Gaussian” 这个词本身，而是 “在 proof-legitimate coordinates 上解释 Gaussian” 这件事？请用 `incoming / outgoing state` 语言回答。
80. 思考题：为什么 `Paragraph E` 到 `Paragraph F` 的真正门槛不是 “对象终于又长得像 \(R_q^d\)” ，而是 “dual-compatible state 是否真的 cash out 回 solver promise” ？请明确说明这两句分别属于哪一种 main burden。
81. 实战题：对 `sentence-surgery notebook` 的三条 long sentence，各把 clause 拆成最少两刀，并分别写出 `what it legalizes / what it does NOT yet earn / which later clause consumes it`。
82. 概念题：为什么 `synthetic long sentence B` 若不在 `B2 | B3` 之间下刀，读者最容易把 `dual glue` 误听成 “继续在讲 embedding geometry”？请用 `route arrow` 语言回答。
83. 思考题：为什么 `synthetic long sentence C` 里最危险的不是 `solver` 这个词本身，而是把 `returned to the standard shell` 和 `solver applies to the promised family` 合并听成一句？请明确指出这里偷掉的是哪一层 law-side burden。
84. 实战题：对 `near-paper paragraph crosswalk` 的 `S1-S4` 四句，各写出 `earned state / still not justified / consumes / overclaim if read too fast` 四格；要求至少指出一处“真正危险的是前一句没赚到的 state，被后一句假装已经拿到了”。
85. 概念题：为什么 `S3` 不能只写成一句 “继续在讲 geometry” ？请分别说明它的前半句和后半句各自落在哪根 route arrow 上，以及两半句中间为什么不能偷并。
86. 思考题：为什么 `S4` 是该 paragraph 第一次真正拿到 `solver-legal average-case family` 的位置，而不是 `S2` 或 `S3`？请用 `earned state / still not justified` 语言回答。
87. 实战题：对 `micro actual-sentence crosswalk` 的 `T1-T3` 三句，各填一张 `literal claim / hidden import / bounce page / H=1 status / transport type / first failure` 六格卡；要求明确指出哪一句第一次真正从 `object transport` 切到 `distribution transport`。
88. 概念题：为什么 `T2` 里的 `carry the intended Gaussian/shortness law` 不能偷译成 “embedding preserves geometry” ？请明确说明这里保的是哪种 law、前面哪一句先赚到了合法 coordinates、以及为什么这一半 still 不能推出 solver applies。
89. 思考题：如果把 `T1` 粗写成一句 “under \(h_k^+=1\) the target family is standard” ，会一次性偷掉哪两层不同的 debt？请用 `host / coordinates / law / solver promise` 语言回答。
90. 实战题：对 `dependency-colored source strip` 的 `D1-D4` 四句，各写出 `tempting color / true burden color / H=1 status / first fake next sentence` 四格；要求至少指出一处“句面像 presentation，实际主 burden 是 freeness”的位置。
91. 概念题：为什么 `D3` 更该染成 `dual/codifferent glue`，而不是笼统写成一句 “继续解释 geometry” ？请明确说明若在这里误染，下一句最自然会偷跳成哪种假过渡。
92. 思考题：为什么 `D4` 的危险点不是 “又回到标准壳” 这几个词，而是 “`promised average-case law`” 这半句？请用 `shell return / distribution-fidelity cash-out` 语言分别说明两者差别。
93. 概念题：把 later-layer final residue elimination 拆成 `theorem statement / finite computation / layer closure` 三层，各写一句它真正要交出的结论；要求明确指出为什么 “factor 出 shared norm” 只够覆盖其中一层。
94. 实战题：为 later-layer theorem notebook 的 `R1-R6` 六行各写一句 incoming fact → outgoing fact；要求至少指出一处“单个 prime 的 verdict” 和 “整层 closure” 之间不能直接偷并的地方。
95. 思考题：为什么 `Branch +` 与 `Branch -` 的差别不只是指数公式长得不同，而是 local certificate 的宿主大小真的变了？请分别用 `N(\mathfrak l)`、`E_{\mathfrak l}` 与 `what this branch certifies` 三格回答。
96. 实战题：固定一层 `k=10`、`k=11` 或 `k=12`，分别写出该层的 \(n_k\)、`Branch +` 下的 local site 数量 \(g_+\)、`Branch -` 下的 local site 数量 \(g_-\)；要求明确说明为什么 `Branch -` 不只是 “指数里多了一个平方”。
97. 概念题：为什么 later-layer local certificate 里，先写 \(g=n_k/f\) 并不是可有可无的 bookkeeping，而是直接决定你到底要验多少个 \(\mathfrak l\)-rows？请用 “prime verdict 需要什么量词” 的语言回答。
98. 思考题：把 later-layer 公式级 audit 拆成 branch fixes → local rows → prime verdict → layer verdict 四层；请分别写出每层最常见的 false equality，并说明若在该层偷并，下一层会如何被假装已经完成。
99. 实战题：给 later-layer 的一条 \(\mathfrak l\)-row 填最短 `prime-ideal row card` 八格；要求最后一格必须明确写出 “still need how many more rows before prime verdict”。
100. 概念题：为什么一条 `Branch +` row 与一条 `Branch -` row 虽然住在不同大小的 residue field 里，但它们在 row verdict 层面说的话完全一样？请用 `one local site cleared` 语言回答。
101. 思考题：把 later-layer 的量词链写成 one row → one prime → whole finite suspect set 三层，并分别指出每层最常见的一句过快错话。
102. 实战题：把 `k=5,\ell=97` 的 concrete row counter 至少抄完前四条 rows，并在每一条后面写一句 `why still not prime verdict`。
103. 概念题：为什么 `R8` 的特殊性不在于它算出了一个“更强”的 residue，而只在于它补齐了哪一道量词？请直接用 `for all \(\mathfrak l\mid 97\)` 语言回答。
104. 思考题：把 `k=5,\ell=97` 的 8-row counter 与 later-layer 的 `Branch +` / `Branch -` row audit 对齐，分别说明“什么完全没变、什么只是在数量级上变大、什么仍然不该被误听成 prime verdict”。
105. 实战题：把 `k=9,\ell=257` 的前四条 rows 抄成最短 counter，并在每一条后面更新 `still need ... more rows`；要求明确写出为什么四条都 pass 仍然离 prime verdict 很远。
106. 概念题：为什么 `k=9,\ell=1279` 的 `M1` 一旦已经给出 \(r_{\mathfrak l}=1\)，后面的 `M2-M4` 就不再承担 “争取 no-certificate” 的功能？请用 `row verdict / prime unresolved` 语言回答。
107. 思考题：比较 `k=5,\ell=97`、`k=9,\ell=257`、`k=9,\ell=1279` 三页 training walkthrough，各写一句它们分别训练的是哪一种 row-reading reflex：`complete counter`、`long accumulation` 还是 `early obstruction`。
108. 实战题：对 `k=10`、`k=11`、`k=12` 的 clean lift 与 dirty lift，分别写出“前 `4` 条 rows 全都 pass”时还剩多少条 rows，外加各自完成比例；要求明确说明为什么 `branch` 只决定 exponent family，而 `lift` 才决定 row count。
109. 概念题：为什么本讲义在 `k=9,\ell=257`、`k=9,\ell=1279` 与 `k=10,\ell=7681` 这三条训练证据里，要同时写出 exponent branch 与 `mod 2^k` lift？请直接用 `dirty lift` 与 `real row count` 语言回答。
110. 思考题：给 later-layer 的 partial audit 各写一句最短合法状态话：一句对应 clean / dirty lift 的 long-counter 前缀，一句对应 `Branch -` 的 first-failure 情形；要求显式出现 `still need ... more rows` 与 `prime unresolved by this certificate`。
111. 实战题：把 `k=10,\ell=7681` 的前四条 rows 抄成最短 counter，并在每一条后面更新 `still need ... more rows`；要求明确写出为什么这已经是真正的 later-layer dirty-lift local-field page，而不是第九章的 toy walkthrough。
112. 思考题：对比 `k=9,\ell=7681` 的 clean-lift `128 rows` 与 `k=10,\ell=7681` 的 dirty-lift `128 rows`，各写一句它们为什么数字相同但理由不同；要求必须同时提到 `k`、`branch` 与 `lift class`。

### 11.7 延伸阅读地图

**初级：先补能读懂本文的最低代数数论**

1. Neukirch, *Algebraic Number Theory*。重点看 number field、ring of integers、ideals、class group。
2. Washington, *Introduction to Cyclotomic Fields*。重点看 cyclotomic fields、characters、Bernoulli、Herbrand-Ribet。
3. 任意一份关于 Dirichlet unit theorem 与 Dedekind domain 的讲义。目标是补对象感，不必先通读全书。

**中级：把本文三条技术线各自补实**

1. Fukuda-Komatsu 的相关论文。对应 Stage 1 sieve 的来源。
2. Miller 关于 Weber class number problem 的工作。对应归纳 base case 与旧结果版图。
3. Washington 中关于 Stickelberger / Herbrand / Ribet 的章节。对应 Stage 3 的理论背景。

**高级：回到密码学主线**

1. Lyubashevsky-Peikert-Regev：理解 RLWE 与 ideal lattices 的经典 reduction。
2. Langlois-Stehle：理解 Module-LWE reduction 与 freeness 要求。
3. NIST FIPS 203 / 204：把 \(K_9\)、\(R_q\)、module-over-\(R_q\) 这条链映射回现实标准。
4. 追 Ideal-LWE、Module-LWE、module-LIP 与 quantum PIP attack 的后续论文，建立“哪些攻击属于 ideal world，哪些进入 module world 会失效”的地图。

## 12. 当我们写到这里，我们已经做了什么？

到这一版为止，这份讲义已经不再只是提纲。它已经具备：

1. 论文导读与阅读路线。
2. 面向弱代数背景读者的最小数学补丁。
3. glossary、proof dependency graph 与算法入口。
4. Section 4 的密码学翻译层。
5. 难点清单、难点学习法矩阵、习题与阅读地图。
6. 一张把 `k=9,10,11,12` 的 theorem-scale verification 分解成“归纳输入 / shared norm / candidate filtering / residue elimination” 的分层核对表。
7. `Section 3.1`、`Section 3.2` 与 `Section 3.3` 三条最关键的桥，已经分别补成：
   - residue-side sieve 的 theorem notebook；
   - full-order reduction 的 layer-by-layer notebook；
   - bad prime why-it-enters-`S_k` 的 theorem notebook。
8. 一页面向 `k=10,11,12` later layers 的 Phase-B 工作单、逐公式 notebook 模板、一页真正把 every surviving prime → local no-certificate → exhaustive layer closure 写成 theorem-scale 量词链的 later-layer residue theorem notebook、一页把 `Branch + / Branch -` 真正摊成 exponent branch、clean/dirty lift、\(f,g,N(\mathfrak l),E_{\mathfrak l},r_{\mathfrak l}\) 的公式级 local-certificate verification page、一页把单个 \(\mathfrak l\)-row 能证明什么与绝对还不能证明什么压成量词审计的 `prime-ideal-row audit page`、一页把 `k=5,\ell=97` 的八条 rows 真抄成完整 counter 的 concrete local-field walkthrough、一页更接近 later-layer 的 `k=9` branch-aware training walkthrough，用 `257` 与 `1279` 两个人工训练 prime 展示 `long accumulation` 与 `early obstruction` 两种 row 命运、一页把这些 training reflex 真正翻译成 `k=10,11,12` 的 lift-aware row-budget sanity page，一页把 `k=10,\ell=7681` 真 rows 抄成 later-layer dirty-lift local-field walkthrough，一页专门说明 `k=10,\ell=12289` 这类高层数慢实验应如何写成 `computation boundary record` 而不是伪装成完成的 local certificate，以及一页把 `k=11/12` 无快速 row 输出时仍应怎样填写 branch modulus、lift modulus、row budget、exponent obligation、blocking step 与 allowed claim 的 `no-fast-output audit ledger`。
9. `Section 2.6` 与 `2.7` 之间最容易混淆的双账本分工，也已经补清楚：\(\xi_5\) 负责 prime elimination，\(B_{1,\psi}\) 负责 candidate generation。
10. `5.6/5.7` 的基础补丁已补上最小小例子与常见误区，不再只停留在“接口说明”，而开始更像真正的教材条目。
11. `Section 2` 到 `Algorithm 1` 的前置接口图也已经显式写出，读者现在可以更顺地从背景章进入 Phase A / Phase B 的算法闭环。
12. `2.5` 的 idempotent/eigenspace decomposition 也已经补成一页零跳步 notebook，读者可以更顺地理解为什么后文能按 \(\chi\)-块逐个清空 class-group 风险。
13. `2.4` 的 full-order / even-odd / conjugate-pair 计数也已经补成一页零跳步 notebook，后文的 candidate counting 与 orbit reduction 不再像黑箱。
14. Section 4 的密码学翻译层已经不再只是结论列表，而是补出了几页零跳步 notebook，分别解释了 module freeness、codifferent/dual lattice、PIP-vs-MLWE、RLWE-vs-PLWE 的对象边界、同一个 RLWE 实例在四套 proof language 里如何切换、Section 4.2 的 reduction 阅读流程该怎样分层、如何给 Section 4.2 段落做页边批注、一条 synthetic reduction skeleton 该怎样逐段贴标签、一张真正可回带到原文上的 worksheet、一张按 `proof obligation / failure mode / H=1 burden` 审计第二遍阅读的 ledger、一张把 `oracle-family / arithmetic-legitimacy / coordinate-legitimacy / geometry-of-shortness / duality-compatibility / distribution-fidelity` 六类 invariant 单独拎出来的 global map、一页专门解释为什么 `same shell` 不自动推出 `same law` 的 distribution-fidelity notebook、一张把 Section 4.2 常见五次对象搬运压成真正手持审计卡的 step-audit card、一页把 `regard as / identify with / under the canonical embedding / via the trace pairing / thus given a solver` 这类句型直接译成 hidden claim 的 phrase decoder、一张直接面向原文自然段的六格 `paragraph-audit template`、一页把五类 source-like reduction 段落原型真的填进六格的 `worked paragraph-audit page`、一页专门解释六段连续 reduction prose 之间 debt 如何接力的 `paragraph-chain notebook`、一页把每个 proof job 该交出的结果压成显式证书链的 `reduction-certificate notebook`、一页把关键 reduction step 真正压成 in → move → out 的 `reduction I/O notebook`、一张把连续六个 proof moves 压成 `earned state / not-yet debt / H=1 status` 路线页的 `reduction route sheet`、一张把连续六段 source-like prose 直接钉回 route arrow 并逐段写出 `why not previous / why not next` 的 `route-to-paragraph margin sheet`、一张把同一条 six-paragraph strip 重新按 `coefficient-shell presentation / freeness / dual-codifferent glue / distribution-fidelity cash-out` 着色的 `four-burden overlay sheet`、一张把同一条 A-F strip 真正逐段压成 `incoming state / outgoing state / main burden / downstream collapse` 的 `worked reduction-by-reduction audit page`、一页把单条长句继续切成 `host / legality / law / cash-out` clause 并逐刀标明哪一半只是在补许可证、哪一半才真的领出下一层 state 的 `sentence-surgery notebook`、一页把整段连续 four-sentence prose 逐句压成 `earned state / still not justified / consumes / overclaim if read too fast` 的 `near-paper paragraph crosswalk`、一页把更像真实 paper bridge 的三句短句压成 `literal claim / hidden import / bounce page / H=1 status / transport type / first failure` 审计账本的 `micro actual-sentence crosswalk`、一页把四句 source-like bridge 强制区分 `surface color` 与 `true burden color` 的 `dependency-colored source strip`、一张 attack-object ledger、short generator 为什么在某些 trapdoor-flavored constructions 里会继续长成 good basis / preimage-sampling leverage、哪类 public relation 天然接受这种 leverage、short generator → concrete break 中间还隔着哪几层构造接口、GPV/Falcon/NTRU-like 与 ML-KEM/ML-DSA-style interface 各自到底把什么 hidden structure 内生进了构造、一张更硬的 `public-key / secret-key / break-target ledger` 用来核对不同安全游戏里的 first break target、一页更细的 `security-game capability card` 用来区分 prerequisite、capability 与 winning action、一张更细的 `relation-by-relation case card` 用来定位五类接口各自的首个拦截点、一张直接把最短 game snippet 写出来的 `single-step walkthrough card`、一张把论文式强句子直接压回 gate 地图的 `paper-prose alignment card`、一张把 Section 4.1 source-like 句子直接标成 `P / C / G` 的 `source-sentence annotation page`，以及 short-generator attack 与 standard scheme interface 之间真正断在哪一层。
15. 数学补丁区已经补到 `5.13`–`5.17`，把 codifferent、monogenicity、different/discriminant、short generator 与 log-unit lattice 放进了可回查的基础层，而不再只在后文密码学段落里突然出现。
16. `10.5` 现在已经给出一张 `ideal world vs module world` 的并排对照表，读者可以把 proofs、reductions 与 attack interfaces 放到同一张地图里看。
17. `5.16` 现在已经补出一个显式的 \(\mathbb{Q}(\sqrt2)\) basis-change notebook，把 coefficient basis、canonical embedding、discriminant 与噪声椭圆化写成真正可算的 \(2\times2\) 矩阵例子。
18. 同一小节现在还把这个 toy model 显式桥接回 \(K_9\)、\(K_9^+\) 与 \(R_q\)，把 “power basis / polynomial coefficients / real-subfield class number / canonical embedding” 之间的关系重新放到了同一页图景里。

但若要达到“可以独立读懂原论文全部证明”的目标，下一轮仍应继续做三件更深的工作：

1. 把 Section 2 每个定义、命题、公式做成更细的逐段注释版。
2. 把 `Section 3.1/3.2/3.3` 现有 notebook 继续压到更细的公式级注释版，尤其是把外引结果与本论文内部结论的接口再标得更清楚。
3. 把当前已经补出的 `k=9,10,11,12` 分层 contract、later-layer Phase-B 工作单、residue theorem notebook、公式级 verification page、`prime-ideal-row audit page`、concrete local-field walkthrough、`k=9` branch-aware training walkthrough、`k=10,11,12` lift-aware row-budget sanity page、`k=10,\ell=7681` dirty-lift local-field walkthrough、`k=10,\ell=12289` computation-boundary page 与 `k=11/12` no-fast-output audit ledger 继续往下压到更像 later-layer 自身的局部域实例页。若有真实 rows，就补 clean-lift / other-lift case 与逐 row walkthrough；若实验不快速产出完整 rows，则优先补 boundary record 与 proof-obligation audit，不把未完成计算伪装成 local certificate。

若只看密码学翻译层，下一轮最值得继续深挖的则是：

1. 在现有 worksheet、synthetic walkthrough、obligation ledger、invariant map、distribution-fidelity notebook、step-audit card、phrase decoder、六格 `paragraph-audit template`、`worked paragraph-audit page`、`paragraph-chain notebook`、`reduction-certificate notebook`、`reduction I/O notebook`、`reduction route sheet`、`route-to-paragraph margin sheet`、`four-burden overlay sheet`、`worked reduction-by-reduction audit page`、`sentence-surgery notebook`、`near-paper paragraph crosswalk`、`micro actual-sentence crosswalk` 与 `dependency-colored source strip` 的基础上，继续把 Section 4.2 压到更细的 reduction-by-reduction 注释页，尤其把 Langlois-Stehle 这类 MLWE reduction 里哪些步骤依赖 freeness、哪些步骤依赖 dual/codifferent、哪些步骤只是在 coefficient presentation 上实现、哪些步骤主要在保 distribution-fidelity，再逐段标明；最好直接落成 `literal claim / hidden import / H=1 status / transport type / first failure` 级别的 source-like 审计页，并逐句指出 `surface color` 与 `true burden color` 是否一致。
2. `Section 4.1` 的接口层现在已经覆盖到 `scheme-interface footnotes`、`public-key / secret-key / break-target ledger`、`security-game capability card`、`relation-by-relation case card`、`single-step walkthrough card`、`paper-prose alignment card` 与 `source-sentence annotation page`。这层若继续深挖，优先级已经低于 `Section 4.2` 的 reduction-by-reduction 注释页；除非你能直接拿真实 prose 做更细的 actual-sentence crosswalk，否则不要再重复写一轮同义 gate 卡。

这一轮之后，计算附录已经不再只是工具入口，而是至少给出了五段高信号样例：

1. `k=5`：手算一个 primitive odd character 的 \(B_{1,\psi}\) 与范数。
2. `k=7`：展示一次真正的 “factor -> congruence filter -> cutoff filter -> candidate set clears” 流程。
3. `k=9`：不只给出单 character 的 \(55\) 位 Bernoulli norm，还实际扫过全部 `64` 个 primitive odd full-order characters，并验证 merged Stage-A raw set 经筛法后为空。
4. `k=5, \ell=97`：把 Phase B 的 \(\xi_5\) residue elimination 真正逐 prime-ideal 跑通一次。
5. `k=9` 的一页 notebook：把 base case、full-order reduction、merged Stage A、vacuous Stage B 与 \(h_9^+=1\) 的闭环真正拼起来；同时用 `k=10` 的单样例提示后续层数的计算代价跃迁。
6. later-layer 的一组真实 row-scale 样例：`k=9,\ell=257`、`k=9,\ell=1279` 与 `k=10,\ell=7681`，已经足够把 exponent branch、clean/dirty lift、real row count、`long accumulation` 与 `early obstruction` 这几层 bookkeeping 明确区分开；`k=10,\ell=12289` 则补成 computation-boundary 样例，`k=11/12` 也已经补出 no-fast-output audit ledger，用来说明高层数实验慢时如何完整记录 branch、lift、row budget、exponent obligation 与阻塞点，而不越界 claim prime verdict。

但要把这份草稿升级成“读者可以跟着它走完原论文主定理验证”的教材稿，仍然需要把当前这套 `k=9` merged recomputation、`3.1/3.2/3.3` theorem notebooks、分层 verification contract、later-layer residue theorem notebook、公式级 branch verification page、`prime-ideal-row audit page`、concrete local-field walkthrough、`k=9` branch-aware training walkthrough、`k=10,11,12` lift-aware row-budget sanity page、`k=10,\ell=7681` dirty-lift local-field walkthrough、`k=10,\ell=12289` boundary record 与 `k=11/12` no-fast-output audit ledger 继续往下压：尤其是把 later-layer 的 final residue elimination，从 theorem-scale、公式级、row-level、toy-instance、branch-aware training、row-budget bookkeeping 页面继续推进到更多真实 row walkthrough；若没有快速实验结果，则继续推进 boundary/audit 页面，先保证 proof-state 文档完整。

