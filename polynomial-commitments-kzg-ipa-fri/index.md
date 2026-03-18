# 多项式承诺的统一看法：KZG、IPA、FRI


> Reading: one article, three schemes, one interface.

前一篇已经把 computation 变成了多项式对象：QAP 里有 $A_{\mathbf{w}}, B_{\mathbf{w}}, C_{\mathbf{w}}, H, Z$，AIR 里有 trace polynomials 与 composition polynomial。到了这里，问题不再是“如何把程序写成多项式”，而是“如何承诺这些多项式，并在不泄露全部系数的前提下证明某些点值或低度性声明”。

如果把 KZG、IPA、FRI 分别介绍，很容易写成三段协议百科。更有用的视角是：三者都在回答同一个问题，只是 machinery 不一样。这个问题可以先压成最小形式：

> 我已经对一个多项式 $f(X)$ 做了承诺。现在我要证明在点 $z$ 上，它的取值确实是 $y = f(z)$，或者至少证明它来自某个低度多项式家族。

在这个统一接口下：

- KZG 把 opening 变成 pairing verification equation
- IPA 把 opening 变成 coefficient vector 和 evaluation vector 的 inner-product proof
- FRI 把 low-degree claim 变成 repeated folding and query consistency checks

所以这一篇的重点不是“哪个更好”，而是它们各自把 evaluation proof 压成了什么对象，又因此带来了什么 setup、proof size 和 verifier cost。[^kzg] [^bulletproofs] [^fri]

<!--more-->

## Polynomial Commitment 的统一接口

先定义抽象接口。一个 polynomial commitment scheme 通常包含：

1. `Commit(f)`：对多项式 $f(X)$ 生成承诺 $C$
2. `Open(f, z)`：给出点 $z$ 处的 opening proof，证明 $f(z)=y$
3. `Verify(C, z, y, \pi)`：检查证明 $\pi$ 是否接受

我们真正关心的是 statement：

$$
\text{given } C \text{ and } z,\ \text{prove that } y = f(z).
$$

这里有两个最基本的性质。

### Correctness

若承诺确实来自某个多项式 $f$，且 prover 用正确 witness 生成 opening proof，那么 verifier 应当接受：

$$
\mathrm{Verify}(C, z, f(z), \pi) = 1.
$$

### Binding

若一个承诺 $C$ 可以被打开成同一点上的两个不同值

$$
f(z) = y \neq y' = f'(z),
$$

且两边都能通过验证，那么 scheme 的 binding 就崩了。不同方案对 binding 的来源不同：

- KZG 依赖 structured reference string 和代数假设
- IPA 依赖离散对数类假设与向量承诺 binding
- FRI 更偏向“oracle codeword 确实接近某个低度多项式”的查询式绑定

也就是说，三者都在提供“你不能随便把同一个 commitment 开成互相冲突的 claim”的保证，但对象和证明方法并不一样。

### Evaluation Proof Is The Real Interface

多项式承诺最容易被讲空的一点是：承诺本身不难，难的是 evaluation proof。

因为真正进入后续 SNARK / STARK 的不是“我有一个多项式”，而是：

- 它在某个点的值是什么
- 它是否满足某个商多项式关系
- 它是否低度
- 多个承诺对象之间是否在随机点上一致

所以这篇的主角不是 commitment string，而是 opening contract。

## KZG：把开点变成配对检验方程

KZG 的思路最紧凑。它利用一个隐藏点 $\tau$ 的 powers-of-tau 结构，把“承诺多项式”压成“承诺在秘密点 $\tau$ 的取值”。

### Setup And Commitment

设多项式次数上界为 $d$。structured reference string 提供

$$
g_1, g_1^\tau, g_1^{\tau^2}, \dots, g_1^{\tau^d}
$$

以及另一组配对群元素

$$
g_2, g_2^\tau.
$$

这里的 toxic waste 就是 $\tau$。若有人知道它，系统会出大问题。

对

$$
f(X) = \sum_{i=0}^d f_i X^i,
$$

KZG commitment 定义为

$$
C = g_1^{f(\tau)} = \prod_{i=0}^d \left(g_1^{\tau^i}\right)^{f_i}.
$$

所以承诺本质上是“在不知道 $\tau$ 的前提下，利用 SRS 对 $f(\tau)$ 做群编码”。

### Opening At \(z\)

若要证明

$$
f(z) = y,
$$

先构造 quotient polynomial

$$
q(X) = \frac{f(X) - y}{X - z}.
$$

这一步的意义非常关键。因为“在点 $z$ 上取值为 $y$”当且仅当

$$
X-z \mid f(X)-y.
$$

也就是商多项式 $q(X)$ 存在。

prover 然后给出 witness

$$
\pi = g_1^{q(\tau)}.
$$

### Verification Equation

现在把 evaluation claim 推到验证方程。

由

$$
f(X)-y = q(X)(X-z)
$$

在 $X=\tau$ 处代入，得

$$
f(\tau)-y = q(\tau)(\tau-z).
$$

转成群与 pairing 语言，就是

$$
e\!\left(C / g_1^y,\ g_2\right)
=
e\!\left(\pi,\ g_2^{\tau-z}\right).
$$

通常也写成

$$
e\!\left(C / g_1^y,\ g_2\right)
=
e\!\left(\pi,\ g_2^\tau / g_2^z\right).
$$

左边编码的是 $f(\tau)-y$，右边编码的是 $q(\tau)(\tau-z)$。两边相等，就等价于 quotient relation 成立。

这就是 KZG opening equation。

> Key Observation.
> KZG 的核心不是“proof 很短”，而是它把 evaluation claim 直接压成了一个关于秘密点 $\tau$ 的商多项式等式，并且这个等式可以用 pairing 一次检查。

### What KZG Buys

KZG 最吸引人的地方很明确：

- proof size 常数级
- verifier 工作量基本是常数个 pairings
- 非常适合 QAP / PLONKish 这类大量依赖多项式开点的系统

代价也同样明确：

- 需要 structured setup
- toxic waste $\tau$ 不能泄露
- 透明性差于 FRI

所以 KZG 不是“免费更快”，而是用强结构换开点简洁度。

## IPA：把开点变成内积证明

IPA 系 polynomial commitment 的出发点不同。它不依赖秘密点 $\tau$，而是把多项式看作系数向量，并把“在点 $z$ 求值”看成一个内积。

### Coefficient Vector View

设

$$
f(X) = \sum_{i=0}^{n-1} a_i X^i,
$$

把系数写成向量

$$
\mathbf{a} = (a_0, a_1, \dots, a_{n-1}).
$$

若用固定生成元向量 $\mathbf{G} = (G_0, \dots, G_{n-1})$ 构造系数承诺，常见形式可写成

$$
C = \langle \mathbf{a}, \mathbf{G} \rangle
$$

或在需要 hiding 时再加 blinding term。这里不展开 hiding 细节，先盯 evaluation contract。

### Evaluation Is An Inner Product

在点 $z$ 处，多项式取值可写成

$$
f(z) = \sum_{i=0}^{n-1} a_i z^i.
$$

定义 evaluation vector

$$
\mathbf{b}(z) = (1, z, z^2, \dots, z^{n-1}),
$$

则

$$
f(z) = \langle \mathbf{a}, \mathbf{b}(z) \rangle.
$$

因此“证明 $f(z)=y$”就变成了：

> 我已经承诺了向量 $\mathbf{a}$，现在证明它与公开向量 $\mathbf{b}(z)$ 的内积恰好是 $y$。

这一步就是 IPA-based evaluation proof 的核心直觉。

### Folding The Dimension

内积证明的强项在于它能递归折叠维度。把向量拆成左右两半：

$$
\mathbf{a} = (\mathbf{a}_L, \mathbf{a}_R),\qquad
\mathbf{b} = (\mathbf{b}_L, \mathbf{b}_R),
$$

然后通过一轮挑战把长度 $n$ 的内积证明压缩成长度 $n/2$ 的新内积证明。重复下去，最终把“高维内积正确”收束成若干轮 transcript 加一个很小的终点检查。

具体 Bulletproofs 风格 IPA 会在每轮发送左右交叉项承诺，然后用 verifier challenge 混合向量与基底，把维度对半折叠。这里不展开完整 transcript，只保留后续系统需要的接口结论：

- proof size 是对数级而不是常数级
- verifier 工作量也是对数级多标量运算
- 不需要像 KZG 那样的 toxic waste

### What IPA Buys

IPA 系方案更接近“transparent or hash-derived generators, logarithmic proof”这一侧。相比 KZG：

- proof 更长
- verifier 更重
- 但 setup 边界通常更干净，不需要 powers-of-tau 式秘密结构

这也是为什么 IPA 系方案在递归、无 pairings 场景下很有吸引力。

## FRI：把低度性变成折叠与查询

FRI 的思路又不一样。它最自然的对象不是“我承诺了系数向量”或“我承诺了 $f(\tau)$”，而是：

> 我给出某个 evaluation domain 上的 codeword，并声称它来自一个低度多项式。

所以 FRI 的主叙事是 low-degree testing contract，而不是单点开值方程。

### Codeword View

设域上有评估域 $D$，prover 持有某个函数表

$$
\{f(x)\}_{x \in D}.
$$

它会先把这份 codeword 通过 Merkle tree 之类的 oracle commitment 固定下来。verifier 接着要检查：

- 这份 codeword 是否接近某个低于给定次数界的多项式
- 若还附带 evaluation claim，则这些查询是否与目标点值或 composition relation 一致

也就是说，FRI 的 commitment 更像“承诺一张评估表”，然后通过查询证明“这张表确实来自低度对象”。

### Even/Odd Decomposition And Folding

FRI 的直觉核心是 even/odd decomposition。任意多项式都可写成

$$
f(X) = f_0(X^2) + X f_1(X^2),
$$

其中 $f_0$ 收集偶次项，$f_1$ 收集奇次项。

取一个随机挑战 $\beta$，定义新多项式

$$
g(X) = f_0(X) + \beta f_1(X).
$$

于是如果知道 $f(x)$ 和 $f(-x)$，就能把它们折叠成关于 $g(x^2)$ 的信息。更显式地，常见写法是

$$
g(x^2)
=
\frac{f(x)+f(-x)}{2}
+
\beta \cdot \frac{f(x)-f(-x)}{2x}.
$$

这一轮 folding 会把“度数上界 $d$ 的低度性问题”压到“大约度数上界 $d/2$ 的新问题”。

重复很多轮之后，原本的大对象被压成一个非常小的最终多项式，verifier 再通过少量随机查询检查每一轮的局部一致性。

### What FRI Is Really Proving

FRI 不是像 KZG 那样直接给出一个 quotient witness，也不是像 IPA 那样给出一条内积证明 transcript。它证明的是：

- codeword 在各轮 folding 下保持一致
- 最终缩小后的对象确实低度
- 因而原始 codeword 也应接近某个低度多项式

所以 FRI 更像“query-based oracle proof of low degree”。

如果在 STARK 场景里还要证明某个 evaluation claim，通常会先把 claim 合并进 composition polynomial 或 quotient-style low-degree object，然后再交给 FRI 去证明低度与一致性。

### What FRI Buys

FRI 的优势也很明确：

- transparent setup
- 不需要 toxic waste
- 非常适合 AIR / trace polynomial / low-degree testing 场景

代价则是：

- proof 通常比 KZG、IPA 更大
- verifier 需要做多轮查询和哈希验证
- statement 更偏向“低度 oracle 是否一致”，而不是单个 pairing equation

所以 FRI 的“透明”不是免费标签，而是用 oracle/query machinery 换来的。

## 统一比较：KZG、IPA、FRI

现在可以把三者拉回同一表面。

### 统一问题

三者都在回答：

$$
\text{What evidence convinces the verifier that a committed polynomial object satisfies the claimed evaluation or low-degree property?}
$$

不同点在于“证据”长什么样。

### KZG

- 对象：$f(\tau)$ 的结构化承诺
- opening 证据：quotient polynomial at $\tau$
- verifier machinery：pairing equation
- 典型特征：常数大小 proof，常数级验证，trusted setup

### IPA

- 对象：系数向量承诺
- opening 证据：inner-product relation transcript
- verifier machinery：对数轮折叠与 MSM 检查
- 典型特征：对数 proof，无 toxic waste，pairing-free

### FRI

- 对象：评估域上的 codeword oracle
- opening / low-degree 证据：folding transcript + query consistency
- verifier machinery：随机查询、Merkle 验证、低度检查
- 典型特征：transparent，proof 较大，验证偏 query-based

### Trusted Setup Versus Transparency

这里最容易空谈。更准确地说：

- KZG 需要结构化参考串，因为 verifier 最终要利用 $\tau$ 的代数结构
- IPA 通常只需公开 generators，可由 hash-to-curve 派生，不需要 toxic waste
- FRI 本质上靠公开评估域、随机挑战和 oracle commitment 工作，因此是 transparent

所以“transparent”不是一句价值判断，而是“安全性不依赖某个隐藏结构参数没有泄露”。

### What They Naturally Pair With

这三类方案也自然对应不同的证明系统接口：

- KZG 很自然地配 QAP、PLONKish 与 pairing-based SNARK
- IPA 很自然地配向量系数承诺、递归友好和 pairing-free 系统
- FRI 很自然地配 AIR、STARK、低度测试型协议

这不是历史偶然，而是由它们各自消费的 polynomial surface 决定的。

## Summary

把 KZG、IPA、FRI 放在一起看，最重要的不是记住三套术语，而是记住同一个 interface：

1. 先承诺一个多项式对象
2. 再证明它满足 evaluation 或 low-degree claim
3. verifier 只通过简化后的代数或查询对象完成检查

KZG 用的是 quotient polynomial 和 pairing equation：

$$
e(C/g_1^y, g_2) = e(\pi, g_2^{\tau-z})
$$

IPA 用的是

$$
f(z) = \langle \mathbf{a}, \mathbf{b}(z) \rangle
$$

再加 inner-product folding transcript。

FRI 用的是 even/odd decomposition 与 repeated folding，把低度性压成 query-based 一致性检查。

所以：

- 若你想要极短开点证明并接受 trusted setup，KZG 很强
- 若你想要 pairing-free、对数级证明和更干净的 setup 边界，IPA 很自然
- 若你要 transparent low-degree testing，并且能接受更大的 proofs 与查询式验证，FRI 是自然答案

下一篇进入 Groth16 时，真正会被消费的就是这里已经写清楚的 KZG/QAP 那条接口：商多项式如何被承诺，evaluation claim 如何被压成最终的 pairing check。

## References

[^kzg]: Aniket Kate, Gregory M. Zaverucha, and Ian Goldberg, *Constant-Size Commitments to Polynomials and Their Applications*, 2010.
[^bulletproofs]: Benedikt Bünz et al., *Bulletproofs: Short Proofs for Confidential Transactions and More*, 2018. Inner-product argument sections.
[^fri]: Eli Ben-Sasson et al., FRI / STARK related references on low-degree testing and transparent proof systems.


