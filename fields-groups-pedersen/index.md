# 有限域、循环群、离散对数与 Pedersen 承诺


> Reading: Pedersen commitments as the first concrete algebraic interface in the ZK series.

上一篇把 relation、instance、witness、soundness、zero knowledge、knowledge soundness 的接口钉住了。这一篇开始把这些抽象对象落到具体代数对象上，但只选最小够用的一组: 标量域 $\mathbb{Z}_q$、一个 prime-order 循环群 $G$、两个生成元 $g,h$，以及一个承诺式子

$$
C = g^m h^r.
$$

这个式子看起来很短，但后面几篇会不断回到它。消息 $m$ 和随机数 $r$ 都是标量；群元素 $C$ 是公开 instance；opening 则是 witness $(m,r)$。如果把这些对象先写清楚，后面再看表示证明、Schnorr、Fiat-Shamir 时，很多结构就不再像“新协议”，而只是同一个关系换了一个验证方式。

Pedersen 承诺最值得写的不是“它同时 hiding 和 binding”这句口号，而是两条具体推导: 第一，为什么对固定消息，随机 $r$ 会把 $g^m h^r$ 推成群上的均匀分布，因此得到 perfect hiding；第二，为什么一旦有人给出两组不同的 opening，就能从

$$
g^m h^r = g^{m'} h^{r'}
$$

直接解出 $\log_g h$，因此 binding 只能是 computational 的。[^pedersen] [^bs]

<!--more-->

## Scalar Field And Prime-Order Group

先固定最小代数环境。

取一个素数 $q$。记

$$
\mathbb{F}_q \cong \mathbb{Z}_q = \mathbb{Z}/q\mathbb{Z},
$$

它既是一个有限域，也是后文的标量空间。然后取一个阶为 $q$ 的循环群 $G$，写作 multiplicative notation。群中的指数都按模 $q$ 运算，所以消息、随机数、挑战、响应这些标量对象都自然落在 $\mathbb{Z}_q$ 上。

这里的“有限域”和“循环群”不是两段独立背景，而是一对绑定对象:

- $\mathbb{Z}_q$ 提供标量运算
- $G$ 提供指数编码后的公开对象

如果 $g \in G$ 是一个生成元，那么每个群元素都可写成 $g^a$，其中 $a \in \mathbb{Z}_q$。这就是后面所有基于离散对数的协议能够把“线性方程”搬到群里的根本原因。

### Why Messages Live In \(\mathbb{Z}_q\)

Pedersen 承诺不是对任意大整数直接工作。它的自然消息空间是 $\mathbb{Z}_q$。原因不复杂:

1. 群的指数本来就按模 $q$ 计算
2. 若把消息视为整数，真正生效的仍然只是其模 $q$ 的剩余类
3. binding 与 opening 的等式推导也都发生在 $\mathbb{Z}_q$

因此，写

$$
C = g^m h^r
$$

时，默认应当读作

$$
m, r \in \mathbb{Z}_q,\quad C \in G.
$$

这一步不先说清楚，后面看“双开封推出一个线性方程”就会失焦，因为那个线性方程不是在整数环里，而是在模 $q$ 的域里。

### Two Generators, One Hidden Relation

再取另一个生成元 $h \in G$。由于 $G$ 是循环群，数学上必然存在某个

$$
\alpha \in \mathbb{Z}_q^\*
$$

使得

$$
h = g^\alpha.
$$

所以很多资料说的“generator independence”若字面理解成“$g$ 与 $h$ 没有离散对数关系”，那是错的。在循环群里，它们总有关系。真正需要的是:

> setup 不应暴露这个关系，也就是 adversary 不应知道 $\alpha = \log_g h$。

这是 Pedersen 承诺里最容易被说糊的地方。不是“随机挑两个生成元就好了”，而是“挑完之后，没人知道它们之间的离散对数关系”。

若有人知道 $\alpha$，绑定性会直接塌掉。这个结论后面会显式算出来。

## Discrete Log Assumption

既然 $h=g^\alpha$，那么一个自然计算问题是:

给定 $(G, g, h)$，求出 $\alpha$。

这就是离散对数问题。对应的 hardness assumption 是，对所选群族与安全参数而言，没有高效算法能以非忽略概率恢复这个 $\alpha$。

Pedersen 承诺的 binding 恰好站在这里。它不是信息论 binding，而是“如果你能制造双开封，我就能拿你的算法去解离散对数”。

所以这篇一定要把两类安全性分开:

- hiding 是信息论的
- binding 是计算的

这两者不是对称性质。

### Setup Requirement

Pedersen setup 常被写成“choose generators $g,h$”。更准确地说，应当是:

1. 选定一个 prime-order 群 $G$
2. 取两个生成元 $g,h \in G$
3. 保证没有参与者知道 $\alpha = \log_g h$

如果 setup 方知道 $\alpha$，它就拿到了 equivocation trapdoor。因为

$$
C = g^m h^r = g^m g^{\alpha r} = g^{m+\alpha r}.
$$

一旦知道 $\alpha$，就可以在保持 $C$ 不变的前提下，把 opening 从 $(m,r)$ 改写成任意别的消息。这个推导后面会正式写。

## Pedersen Commitment As A Linear Encoding

现在给出构造。

给定公开参数 $(G, q, g, h)$，对消息 $m \in \mathbb{Z}_q$：

1. 随机采样 $r \xleftarrow{\$} \mathbb{Z}_q$
2. 输出承诺

$$
\mathrm{Com}(m; r) = g^m h^r \in G.
$$

opening 就是二元组 $(m,r)$。验证则只需检查

$$
C \stackrel{?}= g^m h^r.
$$

### Instance And Witness View

如果用上一篇的关系语言来写，可以定义

$$
R_{\mathrm{Ped}}(C, (m,r)) = 1 \iff C = g^m h^r.
$$

于是：

- instance 是承诺值 $C$
- witness 是 opening $(m,r)$

这已经是一个非常标准的“知道某个表示”的关系。下一篇写 Sigma 协议时，本质上就是围绕这个 relation 去做 proof of representation。

### Commitment Equation Decomposition

Pedersen 承诺最值得盯住的不是结果式子，而是它背后的线性分解。

若设

$$
h = g^\alpha,
$$

则

$$
C = g^m h^r = g^m g^{\alpha r} = g^{m + \alpha r}.
$$

这说明承诺实质上把标量对 $(m,r)$ 压成了同一个群元素里的线性组合

$$
m + \alpha r \pmod q.
$$

如果 $\alpha$ 未知，这个线性关系仍然存在，但外部无法把 $C$ 反解成 $(m,r)$。这正是承诺同时具备 hiding 与 binding 的来源:

- 随机数 $r$ 提供遮蔽
- 未知的 $\alpha$ 阻止随意重写 opening

换句话说，Pedersen 不是“在消息上再乘个随机数”这么粗糙。它是在群里编码了一个带隐藏系数的线性关系。

## Perfect Hiding: A Distribution Argument

先看 hiding。

Pedersen 的 hiding 不是“感觉上随机数很多”，而是严格的分布命题:

> 对任意固定消息 $m \in \mathbb{Z}_q$，当 $r \xleftarrow{\$} \mathbb{Z}_q$ 时，$\mathrm{Com}(m;r)$ 在群 $G$ 上均匀分布。

只要这个命题成立，perfect hiding 就立刻跟着成立，因为不同消息导出的承诺分布完全相同。

### Step 1: Fix The Message

固定某个 $m$。考虑映射

$$
\phi_m : \mathbb{Z}_q \to G,\qquad r \mapsto g^m h^r.
$$

因为 $h$ 是 $G$ 的生成元，而 $G$ 的阶是 $q$，映射

$$
r \mapsto h^r
$$

是从 $\mathbb{Z}_q$ 到 $G$ 的双射。再左乘一个固定群元素 $g^m$，仍然是双射。因此 $\phi_m$ 是双射。

既然 $r$ 在 $\mathbb{Z}_q$ 上均匀，那么 $\phi_m(r)$ 就在 $G$ 上均匀。

### Step 2: Compare Two Messages

设 $m_0, m_1 \in \mathbb{Z}_q$ 是任意两条消息。因为对每个固定消息，承诺分布都等于 $G$ 上的均匀分布，所以

$$
\mathrm{Com}(m_0; r) \equiv U_G \equiv \mathrm{Com}(m_1; r),
$$

其中 $U_G$ 表示群上的均匀分布。

因此，哪怕攻击者算力无限，只看见 $C$ 也无法区分它来自 $m_0$ 还是 $m_1$。这就是 perfect hiding。

### Why This Is Stronger Than Computational Hiding

这里没有使用离散对数假设，也没有限制攻击者算力。分布完全相同，所以 distinguisher 的优势恰好为 $0$。

这就是“perfect”这个词真正表示的内容。它不是“大概很强”，而是：

$$
\forall m_0, m_1,\quad \mathsf{Dist}(\mathrm{Com}(m_0; r), \mathrm{Com}(m_1; r)) = 0.
$$

很多承诺方案只有 computational hiding，而 Pedersen 在这点上更强。

## Computational Binding: Two Openings Break Discrete Log

现在看 binding。

Pedersen 绝不可能是 perfect binding。因为同一个群元素 $C$ 背后本来就有许多可能的 $(m,r)$ 线性组合。真正阻止 adversary 找到第二组 opening 的，不是信息论唯一性，而是它不知道 $\alpha = \log_g h$。

所以 binding 的正确表述是：

> 如果存在高效 adversary 能以非忽略概率输出同一承诺的两组不同 opening，那么就可用它来高效求解离散对数。

### Step 1: Assume A Double Opening

假设 adversary 输出

$$
(C, m, r, m', r')
$$

满足

$$
C = g^m h^r = g^{m'} h^{r'}
$$

且

$$
m \ne m'.
$$

由两式相等可得

$$
g^m h^r = g^{m'} h^{r'}
\Longrightarrow g^{m-m'} = h^{r'-r}.
$$

再代入 $h = g^\alpha$：

$$
g^{m-m'} = g^{\alpha(r'-r)}.
$$

由于群阶是素数 $q$，指数在 $\mathbb{Z}_q$ 中比较，于是得到

$$
m - m' = \alpha (r' - r) \pmod q.
$$

### Step 2: Why \(r' - r\) Must Be Invertible

若 $r' = r$，上式会推出 $m=m'$，这与双开封要求不同消息矛盾。因此

$$
r' - r \ne 0 \pmod q.
$$

而在域 $\mathbb{Z}_q$ 里，每个非零元素都可逆，所以

$$
(r' - r)^{-1}
$$

存在。于是可以直接解出

$$
\alpha = (m - m')(r' - r)^{-1} \pmod q.
$$

也就是

$$
\alpha = \log_g h.
$$

这正是 binding 规约的核心。

### The Reduction View

把上面的等式链改写成规约语言就是：

1. 规约者接收一个离散对数实例 $(G, q, g, h)$
2. 它把这组参数直接交给双开封 adversary
3. 若 adversary 返回同一承诺的两组不同 opening
4. 规约者就按上式恢复 $\alpha = \log_g h$

因此，只要离散对数在该群上难解，高效双开封也应当难以发生。这就是 computational binding。

> Key Observation.
> Pedersen 的 binding 不是“一个承诺只对应一个 opening”，而是“想构造两个不同 opening，等价于求出隐藏的离散对数关系”。

## If \(\alpha\) Is Known, Binding Dies Immediately

前面已经说过 setup trapdoor 是危险点，这里把它正式算出来。

仍然写

$$
h = g^\alpha.
$$

如果某人知道 $\alpha$，并且手里已经有一个合法 opening $(m,r)$，对应承诺

$$
C = g^m h^r = g^{m+\alpha r},
$$

那么它想把同一个 $C$ 改开为任意另一个消息 $m^\*$，只需取

$$
r^\* = r + (m - m^\*)\alpha^{-1} \pmod q.
$$

验证一下：

$$
g^{m^\*} h^{r^\*}
= g^{m^\*} g^{\alpha r^\*}
= g^{m^\* + \alpha r + m - m^\*}
= g^{m + \alpha r}
= C.
$$

所以知道 $\alpha$ 就能把一个承诺开成任意消息。这说明 setup 的真实要求不是“参数公开即可”，而是“参数公开，但生成元之间的离散对数关系不被任何一方掌握”。

这也是为什么很多场景会通过 hash-to-group 或多方生成参数来避免单方持有 trapdoor。

## Linear Relation View: Why This Matters For ZK

到这里，Pedersen 承诺已经不只是一个“藏消息”的工具了。更重要的是，它给了我们一个非常适合做零知识证明的 relation：

$$
C = g^m h^r.
$$

这个 relation 有三个好处。

### First, The Witness Structure Is Explicit

witness 不是一个黑箱秘密，而是一个向量式对象 $(m,r)$。这会让后面“证明知道 opening”自然变成“证明知道某个线性表示”。

### Second, Verification Is Algebraic

验证条件就是一个显式群等式。Sigma 协议最喜欢这样的关系，因为：

- 可以围绕它构造 commitment-challenge-response
- 可以从两份 transcript 做 special soundness 提取
- 可以构造 honest-verifier simulator

换句话说，Pedersen 承诺不是零知识系统的外围组件，而是后续很多协议的 statement layer。

### Third, Additivity Comes For Free

虽然本文先只写单消息承诺，但它已经显露出线性结构：

$$
\mathrm{Com}(m_1; r_1)\mathrm{Com}(m_2; r_2)
= g^{m_1+m_2} h^{r_1+r_2}
= \mathrm{Com}(m_1+m_2; r_1+r_2).
$$

这意味着群乘法对应标量加法。后面做表示证明、范围证明、向量承诺接口时，这条线性结构会反复出现。

这里先不展开多消息版本，只保留一个判断：Pedersen 的价值不只是 commitment，而是它把“线性关系 + 随机掩码 + 群验证”等式打包成了一个极小接口。

## Summary

这一篇其实只固定了四件事。

第一，标量在 $\mathbb{Z}_q$ 上，群元素在 prime-order 循环群 $G$ 中，消息与随机数都应被视作模 $q$ 的对象。

第二，Pedersen 承诺

$$
C = g^m h^r
$$

不是随意拼起来的式子，而是把 $(m,r)$ 编码成群里的线性组合。

第三，perfect hiding 来自一个严格的分布事实：对固定 $m$，映射

$$
r \mapsto g^m h^r
$$

是到群 $G$ 的双射，因此承诺分布与消息无关。

第四，binding 只能是 computational 的。因为若存在两组不同 opening，就能从

$$
g^m h^r = g^{m'} h^{r'}
$$

推出

$$
\alpha = (m-m')(r'-r)^{-1} = \log_g h.
$$

所以 Pedersen 的 binding 不是唯一开封，而是“双开封会泄露隐藏的离散对数关系”。

下一篇进入 Sigma 协议时，我们就会把这个 relation 直接当作 statement 来证明：不是再介绍一个全新的世界，而是开始研究如何围绕这样的代数关系构造 transcript、simulator 与 extractor。

## References

[^pedersen]: Torben Pryds Pedersen, *Non-Interactive and Information-Theoretic Secure Verifiable Secret Sharing*, 1991. The commitment construction and its hiding/binding split are standardly attributed to Pedersen commitments.
[^bs]: Dan Boneh and Victor Shoup, *A Graduate Course in Applied Cryptography*, commitment-related chapters.


