# STARK：AIR、低度测试与透明性


> Reading: STARK as a proof pipeline, not a brand label.

如果说 Groth16 的关键词是 `QAP + CRS + pairing equation`，那么 STARK 的关键词就是 `trace + AIR + FRI`。把它说成“没有 trusted setup 的 SNARK”不算错，但也没抓住结构。真正要理解的是：STARK 证明的对象不是一个静态约束向量，而是一整张 execution trace；AIR 不是一个术语包装，而是把这张 trace 的合法性写成代数关系；FRI 则不是附在最后的 appendix，而是整个 soundness contract 的后半段。

前一篇已经把 FRI 放到 polynomial commitment / low-degree testing 的统一接口里了。这一篇要把它放回系统级视角：trace 怎样变成列多项式，边界条件和转移条件怎样被压成 composition polynomial，以及为什么“低度”在这里不是顺带属性，而是 verifier 最终抓住 cheating trace 的主要证据。

所以这一篇的主链不是：

$$
\text{AIR}\quad\text{and}\quad\text{FRI}
$$

而是：

$$
\text{trace}
\longrightarrow
\text{AIR constraints}
\longrightarrow
\text{composition polynomial}
\longrightarrow
\text{FRI low-degree test}.
$$

<!--more-->

## STARK 证明的对象是什么

STARK 的 witness 不是一个一次性列出的向量，而是某个 computation 的 execution trace。

设一个计算被展开成 $N$ 个时间步，且每步有 $k$ 个寄存器列。则 trace table 可写成

$$
T \in \mathbb{F}^{N \times k}.
$$

第 $i$ 行是第 $i$ 步状态，第 $j$ 列是一条寄存器轨迹。和 R1CS/QAP 那种“静态约束列表”不同，这里 prover 真正声称的是：

> 这整张 trace 是某个合法计算的执行结果。

### AIR Constraint System

AIR, Algebraic Intermediate Representation，把这句“trace 合法”拆成两类约束。

第一类是 boundary constraints。它们固定某些特定位置的值，例如初始状态、公开输入、终止输出：

$$
T_{0,1} = x,\qquad T_{N-1,2} = y.
$$

第二类是 transition constraints。它们描述相邻若干行之间必须满足的递推关系。例如某个单寄存器递推可以写成

$$
T_{i+1,1} - T_{i,1}^2 - c = 0.
$$

更一般地，若一组列在窗口大小 $\ell$ 上满足某个多变量多项式关系，则写成

$$
\Phi\big(T(i), T(i+1), \dots, T(i+\ell)\big)=0
$$

对所有有效步成立。

这就是 AIR 的最小接口：不是“程序源码”，而是“trace 上的代数约束系统”。

## 从 AIR 到 Composition Polynomial

仅仅列出很多条 AIR constraints 还不够。和 QAP 一样，证明系统更喜欢一个被合成后的单一对象。

### Trace Columns As Polynomials

首先把每一列 trace 在某个评估域 $D$ 上插值成列多项式：

$$
t_j(X),\qquad j=1,\dots,k.
$$

这样，原来“第 $i$ 步第 $j$ 列的值”就被编码成“在域点 $\omega_i$ 上的多项式取值”。

于是 boundary constraints 可以改写成某些固定点评值条件，transition constraints 可以改写成这些列多项式在相邻点上的多变量关系。

### Constraint Combination

STARK 不会把每一条约束都单独证明。常见做法是用随机系数把多条约束组合成一个 composition polynomial。形式上，可以把若干归一化后的约束项写成

$$
C_1(X), C_2(X), \dots, C_m(X),
$$

然后取随机挑战系数 $\alpha_1,\dots,\alpha_m$，构造

$$
\mathrm{Comp}(X) = \sum_{j=1}^m \alpha_j C_j(X).
$$

这里每个 $C_j(X)$ 往往已经除掉了相应的 vanishing factors，使其在合法 trace 上应当保持低度。

所以 composition polynomial 的作用不是“把公式糊在一起”，而是：

- 把很多局部约束压成一个统一对象
- 让 prover 无法分别对不同约束作弊
- 让 verifier 只需对一个组合对象做低度性和一致性检查

> Key Observation.
> STARK 里的 composition polynomial，扮演的是和 QAP 中 quotient/divisibility claim 类似的接口角色：把很多局部约束压成 verifier 真正要消费的单一代数对象。

## 低度为什么是 soundness certificate

STARK 最容易被误解的一点，是把 low degree 当成“为了 FRI 方便”才引入的技术条件。更准确的理解是：低度本身就是 soundness certificate。

### Low-Degree Extension

trace 和 composition object 不会只在原始执行域上使用。prover 通常会把它们扩展到更大的评估域，也就是 low-degree extension, LDE。直觉上，这一步让 verifier 可以在更大的域上随机抽查对象行为，而不是只在原始 trace 点上看局部值。

如果 trace 真正来自一个满足 AIR 的合法计算，那么对应列多项式和 composition polynomial 都应该具有受控的低度界。

### What A Cheating Trace Breaks

若 prover 伪造了一张不合法的 trace，会发生什么？

边界条件或转移条件一旦被破坏，组合约束对象就不再与某个低度多项式一致。也就是说，作弊者也许能在少量点上伪装局部值，但它很难让整张 oracle table 在全域上继续看起来像一个满足度数界的对象。

这就是“低度 acts as a soundness certificate”的真正含义：

- 合法计算诱导低度结构
- 非法计算往往只能伪造局部一致性
- verifier 通过随机查询低度结构，捕捉这种全局不一致

这和 Groth16 的思路正好形成对照。Groth16 的 soundness certificate 更接近“pairing 空间里的单一代数平衡式”；STARK 的则更接近“oracle object 必须 globally look low-degree and consistent”。

## FRI 在 STARK 里扮演什么角色

现在把 FRI 放回主流水线。

STARK prover 会把 trace / composition objects 的评值表承诺成 oracle，通常再用 Merkle tree 固定。verifier 拿到的是：

- 一个承诺好的 evaluation oracle
- 若干随机挑战
- 若干查询路径与一致性证明

FRI 的任务就是回答：

> 这个 oracle 是否确实接近某个低度多项式？

### Folding And Query Consistency

FRI 的核心仍然是 folding。对一个多项式

$$
f(X)=f_0(X^2)+Xf_1(X^2),
$$

取随机挑战 $\beta$，构造

$$
g(X)=f_0(X)+\beta f_1(X).
$$

这会把原来的低度测试问题压缩到更小域、更低度的新对象上。反复多轮后，verifier 只需检查：

- 每轮 folding 前后的局部值是否一致
- 最终压缩到很小的对象是否满足显式低度界

所以 FRI 在 STARK 里不是“再加一个 commitment trick”，而是证明 pipeline 里专门负责 low-degree certification 的模块。

### From FRI Back To AIR

这一步必须反过来看。FRI 验的不是任意低度多项式，而是前面 composition polynomial pipeline 产出的那个对象。

因此 STARK 的 soundness 叙事是两段连在一起的：

1. AIR 把 trace 合法性压成 composition polynomial 应满足的结构
2. FRI 证明 prover 承诺的那个组合对象确实具有相应的低度与一致性

少了前半段，FRI 只是在测一个多项式是否低度；少了后半段，AIR 只是纸面上的约束描述。只有两者接起来，才是完整 STARK。

## 透明性与 post-quantum 定位

STARK 最常被拿来宣传的两个词是 transparent 和 post-quantum。这两个词都该拆开讲。

### Transparent Setup

transparent 的含义不是“完全没有 setup”，而是：

- 不需要像 Groth16/KZG 那样的 toxic waste ceremony
- verifier 和 prover 只依赖公开域、公开哈希、公开随机挑战流程

也就是说，系统不依赖某个秘密结构参数没有泄露。这和前面的 pairings + structured CRS 路线形成明确对照。

### Why People Call It Post-Quantum

STARK 常被称为 post-quantum，主要因为其核心安全叙事更接近：

- 哈希函数安全性
- 信息论式或组合式 low-degree testing soundness
- 不依赖 pairing 或传统离散对数那类结构化代数假设

这并不意味着“只要是 STARK 就自动在任何量子模型里完美安全”。更准确的说法是：它的主要安全基础通常被认为比 pairing-based SNARK 更适合 post-quantum setting。

所以这里也要避免口号化。transparent 说的是 ceremony boundary；post-quantum 说的是主要依赖的 hardness surface。

## Summary

STARK 不是 “AIR + FRI” 两个词的拼接，而是一条连续的证明流水线：

1. witness 被展开成 execution trace
2. AIR 把合法执行写成 boundary constraints 和 transition constraints
3. 这些约束被组合成 composition polynomial
4. prover 对相应 oracle objects 做承诺
5. FRI 证明这些对象确实保持预期低度与 folding 一致性

所以 verifier 最终相信的，不是某个神秘 transcript，而是：

- 这张 trace 的约束组合对象看起来像合法低度多项式
- 少量随机查询足以高概率发现伪造 trace 的全局不一致

如果把 Groth16 看成 `QAP + structured CRS + pairing equation`，那么 STARK 可以看成

$$
\text{AIR} + \text{oracle commitment} + \text{FRI low-degree test}.
$$

这条路线换来了 transparent setup、更大的 proof、以及更 query-based 的 verifier 工作方式。下一层进入更现代系统时，PLONKish、递归和工程约束讨论的，其实都是在这些不同 algebraic surfaces 和 proof pipelines 上继续做设计。

## References

[^stark]: Eli Ben-Sasson et al., *Scalable, Transparent, and Post-Quantum Secure Computational Integrity*, 2018.
[^fri]: FRI and related low-degree testing references.

