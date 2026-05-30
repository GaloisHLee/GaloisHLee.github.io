# PLONKish 体系：置换论证、lookup 与自定义门


> Reading: PLONKish as a design space, not a single protocol.

到了这一阶段，证明系统讨论的重点已经不再是“有没有多项式承诺”或者“有没有低度测试”，而是你把 computation 摆成什么 witness surface。PLONKish 的意义就在这里：它把现代 proving systems 的证明面固定成了 rows、columns、selectors、copy constraints、lookup tables、custom gates 这些对象，然后允许设计者在这个面上不断加表达力。

如果把 permutation、lookup、custom gates 分开讲，很容易误以为它们是三种后来加上的 feature。更准确的理解是：它们都在回答同一个问题，

> 当 witness 被排成列多项式之后，系统如何表达“这些列之间哪些值必须相等、哪些值必须属于某张表、以及每一行到底允许执行什么语义”？

PLONKish 的主角因此不是某篇单独论文，而是 grand-product machinery 和基于 selector 的约束设计。copy constraints 被重写成 permutation relation；lookup 被重写成 multiset relation；custom gates 则不断给 quotient degree、selector count、column layout 和 prover cost 施压。

这一篇的主线因此也不是 feature list，而是同一个 proving surface 上的三类约束重写。[^plonk] [^plookup]

<!--more-->

## PLONKish 证明面长什么样

PLONKish 最先改变的，是 witness 的组织方式。

不同于 QAP 那种更“全局插值后一次性消费”的感觉，PLONKish 更像是把 witness 放进若干列，然后在每一行上施加局部代数条件。设评估域为

$$
H = \{\omega^0, \omega^1, \dots, \omega^{n-1}\}.
$$

每一列 witness 被插值成列多项式，例如

$$
a(X),\ b(X),\ c(X),
$$

它们在点 $\omega^i$ 上的值，就是第 $i$ 行对应列的 witness 值。

### Rows, Columns, And Selectors

最经典的门约束可以写成

$$
q_M(X)a(X)b(X)+q_L(X)a(X)+q_R(X)b(X)+q_O(X)c(X)+q_C(X)=0
$$

在所有 $X \in H$ 上成立。

这里：

- $a,b,c$ 是 witness columns
- $q_M,q_L,q_R,q_O,q_C$ 是 selector polynomials

selector 的作用是：不是每一行都执行同一条语义，而是用多项式系数选择某一行上哪些项激活、哪些项关闭。

所以 PLONKish 的 proving surface 不是“一个庞大的约束矩阵”，而是：

- 列多项式承载 witness
- selector 多项式承载 gate semantics
- 域点逐行承载局部约束

这比纯 QAP 更适合表达现代系统常见的 copy constraints 和 richer gate semantics。

## 从 copy constraints 到 permutation argument

PLONKish 真正变聪明的地方，是它没有把“两个位置的值必须相等”继续硬编码成大量显式约束，而是把这些 copy constraints 改写成 permutation consistency。

### What Copy Constraints Actually Say

在电路或行列布局里，copy constraint 的含义很朴素：

- 某一行某一列的值，应当等于另一行另一列的值
- 一个逻辑变量在不同 gate 位置复用时，所有出现点必须一致

如果直接为每对位置写显式相等约束，系统会很笨重。PLONK 的做法是：不逐对检查，而是把整组“应该相等的位置”编码进一个排列。

### Permutation Encoding

设每个位置都带有一个身份标签 id，例如列编号和行编号的组合。copy constraint 指定了一组置换 $\sigma$，把“原始位置标签”映射到“应该与之相等的位置标签”。

于是问题从

> 这些位置两两相等吗？

改写成

> witness values 加上位置标签后，原始序列和置换后序列是否表示同一个 multiset？

这就是 permutation argument 的核心转换。

### Grand Product Polynomial

PLONK 不会直接逐点比较两个 multiset，而是引入一个 grand-product polynomial $Z(X)$，递推地累计两边比值。直觉上，可写成

$$
Z(\omega^{i+1})
=
Z(\omega^i)
\cdot
\frac{
(a_i + \beta \cdot \mathrm{id}_{a,i} + \gamma)
(b_i + \beta \cdot \mathrm{id}_{b,i} + \gamma)
(c_i + \beta \cdot \mathrm{id}_{c,i} + \gamma)

}{
(a_i + \beta \cdot \sigma_{a,i} + \gamma)
(b_i + \beta \cdot \sigma_{b,i} + \gamma)
(c_i + \beta \cdot \sigma_{c,i} + \gamma)
}.
$$

这里 $\beta,\gamma$ 是随机挑战，用来把“值 + 位置标签”压成单个 field 元素，避免攻击者在不同分量上做巧妙抵消。

如果 copy constraints 全都满足，那么分子和分母描述的是同一个 multiset，累计乘积最终应当回到 1；于是 $Z(X)$ 作为一个 witness polynomial，可以在边界条件和递推关系下被 verifier 检查。

> Key Observation.
> grand product 不是附加技巧，而是 PLONKish 把全局相等约束压成单一 witness object 的核心 machinery。

### Why This Matters

有了 permutation argument，系统不再需要把变量复用翻译成大量手写等式。copy constraints 被收缩成：

- 一个排列编码
- 一个 grand-product witness
- 一组 boundary / transition-like 约束

这使得列式 witness layout 和复用变量之间的关系变得非常可扩展。

## Lookup：把表成员关系压成 multiset relation

仅仅有 permutation argument 还不够。很多工程约束并不是“两个位置相等”，而是“某个 witness 值必须属于某张合法表”。典型例子是：

- range check
- byte decomposition
- 固定 opcode table
- 某种预定义函数关系表

如果不用 lookup，而只靠普通门去表达这些关系，row count 往往会爆炸。

### Why Plain Gates Hurt

以 range check 为例。若想证明某个值 $x$ 在 $[0, 2^8)$ 内，仅用普通低次数门去展开，通常需要很多 bit-decomposition 或 carry constraints。逻辑没错，但门数和列布局压力会很大。

lookup 的思路是换问法：

> 与其证明这个值逐步满足一堆基础位约束，不如直接证明它出现在一张合法表里。

### Table Membership As Multiset Equality

设 witness 中需要查表的一组值为

$$
f_1,\dots,f_m,
$$

固定表中的值为

$$
t_1,\dots,t_N.
$$

lookup 的核心不在“查”，而在把

$$
\{f_i\}
$$

属于表的关系，重写成某种 multiset equality / inclusion relation。不同方案实现细节不同，但直觉一致：

- 要么把 witness lookups 与 table entries 做排序拼接，证明排序后相邻差分满足结构关系
- 要么构造类似 grand-product 的累计对象，证明两边压缩后的 multiset 一致

所以 lookup 本质上不是数组访问，而是：

> table membership to multiset relation.

### Lookup Relation As A First-Class Constraint

这一步一旦有了，系统表达力会明显增强。比如某些非线性或离散约束，本来需要很多普通门，现在只要：

1. 把候选值放进 lookup witness 列
2. 指定固定表列
3. 证明 lookup relation 成立

代价当然不是零。lookup 会引入额外列、额外排序对象或额外 grand-product 结构，但在很多场景下，这比硬把逻辑拆成基础门更便宜。

## Custom Gates 不是免费表达力

PLONKish 的第三个核心对象是 custom gates。它们的直觉很简单：别让每一行只表达最基础的乘加关系，而是允许一行承载更多语义。

### Packing More Semantics Per Row

比如一个 custom gate 可以在一行里表达：

- 某种特定线性组合
- 一次 range chunk relation
- 一次椭圆曲线加法局部约束
- 一小段 hash round 的局部语义

这会减少总 row 数，因为同样的 computation 被压进更丰富的每行语义里。

### Degree, Selectors, And Prover Cost

但 custom gates 从来不是白来的。表达力越强，系统通常要支付以下几类代价：

- quotient polynomial degree 上升
- selector 多项式更复杂
- witness 列布局更紧张
- prover 在构造和开点这些对象时成本更高

换句话说，custom gate 不是“多加一个 feature”，而是直接改变 proving surface 的工程平衡点。

如果一个 gate 设计得太激进，虽然 row count 下降了，但 quotient degree 可能上升，最终导致证明大小、开点复杂度或者实现难度反而更糟。

### Why This Becomes A System Design Problem

这就是为什么到了 Halo2 一类实现里，讨论重点经常变成：

- advice / fixed / instance columns 怎么排
- 哪些关系值得做 custom gate
- 哪些关系更适合做 lookup
- copy constraints 和 lookup constraints 会不会互相挤压布局

也就是说，现代 proving system 已经不是“选一个协议然后填 witness”这么简单，而是在设计一个约束 DSL。

## 统一视角：一切都在重写约束接口

现在把这三者放回同一个表面。

### Permutation Argument

它处理的是：

> 不同位置上的值必须是同一个逻辑变量。

实现方式是 permutation encoding + grand product。

### Lookup

它处理的是：

> witness 值必须属于某个合法表。

实现方式是 multiset relation，常常也会借用 grand-product or sorting machinery。

### Custom Gates

它处理的是：

> 每一行到底允许表达多丰富的局部语义。

实现方式是 selector-polynomial design，加上对 degree / columns / prover cost 的控制。

所以 PLONKish 真正统一的，不是某个单一协议公式，而是一套 row/column based proving surface：

- witness 被摆进列
- gate semantics 由 selectors 决定
- copy constraints 由 permutation argument 管
- table membership 由 lookup relation 管
- 系统表达力与 proving 成本由 custom gate 设计压平衡

> Quick Note.
> PLONKish 不是“PLONK 加几个补丁”，而是一整套现代 proving system 的约束设计语言。

## Summary

如果只记住 PLONKish 支持 permutation、lookup、custom gates，这还只是 feature list。更有用的总结是：

1. witness surface 被固定成 rows、columns 和 selector polynomials
2. copy constraints 被重写成 permutation relation
3. permutation relation 被压成 grand-product polynomial
4. table membership 被重写成 lookup multiset relation
5. custom gates 决定每一行到底能塞多少语义，并同时改变 degree、布局和 prover cost

所以 PLONKish 的核心不是某条单独方程，而是“如何设计约束接口，使现代 proving system 既能表达复杂语义，又不把 proving cost 推爆”。

下一篇进入 IPA / Halo / Nova / folding 时，这种“系统设计空间”的感觉还会更强，因为递归和 accumulation 会进一步把 proving surface 从静态约束推向跨 proof-state 的接口设计。

## References

[^plonk]: Ariel Gabizon, Zachary J. Williamson, and Oana Ciobotaru, *PLONK: Permutations over Lagrange-bases for Oecumenical Noninteractive arguments of Knowledge*, 2019.
[^plookup]: Ariel Gabizon and Zachary J. Williamson, *plookup: A simplified polynomial protocol for lookup tables*, 2020.

