# Sigma 协议的统一视角：Schnorr、表示证明与 Fiat-Shamir


> Reading: sigma protocols as the first reusable proof skeleton of the series.

上一篇把 Pedersen 承诺写成了一个 relation：

$$
C = g^m h^r.
$$

这一篇开始研究，怎样围绕这种 relation 构造一个既能 extraction、又能 simulation 的三步证明骨架。真正需要记住的不是 “Schnorr” 这个名字，而是同一个模板：first message 先把 witness 压进一个承诺式对象，verifier 给出一个公币 challenge，prover 再用线性响应把 witness 带回来。

Sigma 协议之所以重要，不是因为它是某个古典协议家族，而是因为它把三件后面一直会重复出现的东西压进了同一份 transcript：

- commitment / first message
- extractor 需要的两份 accepting transcripts
- simulator 需要重建的 transcript distribution

这也是它成为后续现代证明系统桥梁的原因。Schnorr 证明离散对数是最小例子，表示证明是直接推广，而 Fiat-Shamir 则显示这套骨架一旦被 hash 取代 challenge，交互虽然消失，模型边界却一起换掉了。[^schnorr] [^bs] [^fs]

<!--more-->

## Sigma 协议的最小接口

先把对象固定下来。

给定一个 relation

$$
R(x, w) = 1,
$$

Sigma 协议通常指一个三步 public-coin 协议：

1. prover 发送 first message，也常记作 commitment $a$
2. verifier 发送 challenge $e$
3. prover 发送 response $z$

verifier 最后检查某个验证方程是否成立。

如果把 transcript 写成

$$
\tau = (a, e, z),
$$

那么 Sigma 协议最核心的不是“只有三步”，而是这三步同时满足三条性质：

- completeness
- special soundness
- special honest-verifier zero knowledge, special HVZK

### Why “Public-Coin” Matters

public-coin 意味着 verifier 的 challenge 只是公开随机挑战，而不是夹带私有状态的复杂消息。这个约束非常关键，因为：

- extractor 需要比较两个不同 challenge 下的接受 transcript
- simulator 在 HVZK 里通常要先选 challenge 再倒推 first message

如果第二步不是 clean challenge，而是带隐藏状态的恶意消息，很多简洁的提取和模拟公式都不再成立。

### The Three Properties

completeness 这里不新鲜：诚实 prover 持有 witness 时，验证应通过。

special soundness 则更具体。它要求：

> 只要拿到两份使用同一个 first message $a$、但 challenge 不同的接受 transcript，就能高效恢复 witness。

注意这里的关键前提是 same first message。没有这个前提，提取公式通常不存在。

special HVZK 则是另一面：

> 存在一个高效 simulator，能够先采样 challenge 和 response，再反推出一个 first message，使得模拟 transcript 与诚实 verifier 下的真实 transcript 分布一致。

special soundness 和 special HVZK 看起来像两条不同性质，实际上它们都在利用同一件事：response 通常是 witness 与随机承诺变量的线性组合。这个线性结构让两份 transcript 可以相减做提取，也让 simulator 可以先定 response 再倒推 first message。

## Schnorr 作为规范例子

最经典的 sigma 协议是 Schnorr 证明离散对数。

取一个 prime-order 循环群 $G = \langle g \rangle$，阶为 $q$。statement 是一个群元素

$$
y = g^x,
$$

其中 $y$ 公开，$x \in \mathbb{Z}_q$ 是 witness。relation 写成

$$
R(y, x) = 1 \iff y = g^x.
$$

### Protocol Messages

诚实 prover 持有 $x$，按下面三步运行：

1. 采样随机数 $r \xleftarrow{\$} \mathbb{Z}_q$，发送

$$
a = g^r
$$

2. verifier 发送 challenge

$$
e \xleftarrow{\$} \mathcal{C}
$$

通常可把挑战空间看成 $\mathbb{Z}_q$ 或某个足够大的子集

3. prover 返回 response

$$
z = r + ex \pmod q
$$

verifier 检查

$$
g^z \stackrel{?}= a y^e.
$$

### Why The Verification Equation Works

这条检查看起来像记忆公式，其实只是把定义代回去。

因为

$$
z = r + ex,
$$

所以

$$
g^z = g^{r+ex} = g^r (g^x)^e = a y^e.
$$

这就是 completeness。

Schnorr 的关键不在于这条式子本身，而在于这条式子把 witness $x$ 放进了一个线性位置：$z = r + ex$。后面 special soundness 与 simulator 都在吃这条线性结构。

## Special Soundness：两份 Transcript 提取 Witness

现在进入第一条核心推导。

假设我们拿到两份接受 transcript：

$$
(a, e, z), \qquad (a, e', z'),
$$

它们具有：

- 同一个 first message $a$
- 不同 challenge，$e \ne e'$
- 都通过验证

对 Schnorr 来说，接受意味着

$$
g^z = a y^e,\qquad g^{z'} = a y^{e'}.
$$

把第二式从第一式消掉，得到

$$
g^{z-z'} = y^{e-e'}.
$$

再代入 $y = g^x$：

$$
g^{z-z'} = g^{x(e-e')}.
$$

由于群阶是素数 $q$，指数在 $\mathbb{Z}_q$ 中比较，可得

$$
z - z' = x(e - e') \pmod q.
$$

若 $e \ne e'$，则 $(e-e')^{-1}$ 存在，于是 extractor 立刻恢复

$$
x = (z - z')(e - e')^{-1} \pmod q.
$$

这就是 Schnorr 的 special soundness extractor 方程。

### What The Extractor Really Uses

这里 extractor 只用了两件事：

1. same first message $a$
2. response 对 witness 线性

如果 first message 不同，就无法把 $a$ 消掉；如果 response 不是线性的，也很难得到单步求逆式的 witness 恢复。

所以 special soundness 不是一句抽象标签，而是一个非常具体的 transcript algebra 条件。

### Soundness Error And Challenge Space

再补一个边界。

special soundness 说的是：若你已经拿到两份不同 challenge 的接受 transcript，就能提取 witness。它本身并没有保证你一定能轻易拿到这两份 transcript。

在单次交互里，作弊 prover 若没有 witness，通常最多只能提前准备一个对某个 challenge 有效的响应。因此 challenge 空间越大，撞上 verifier 实际 challenge 的概率越小。这个概率上界正是 sigma 协议 soundness error 的来源之一。

所以这里不要把 special soundness 误读成“只要接受一次就必然抽出 witness”。真正的提取叙事通常还要结合 rewinding、forking 或多次交互。

## Special HVZK：先选 Challenge 再倒推 Transcript

第二条核心推导是 simulator。

对 Schnorr，special HVZK simulator 可以这样构造：

1. 随机采样

$$
e \xleftarrow{\$} \mathcal{C}, \qquad z \xleftarrow{\$} \mathbb{Z}_q
$$

2. 定义

$$
a = g^z y^{-e}
$$

3. 输出 transcript

$$
(a, e, z)
$$

### Why The Simulated Transcript Verifies

直接代回去：

$$
a y^e = g^z y^{-e} y^e = g^z.
$$

所以 verifier 的检查

$$
g^z \stackrel{?}= a y^e
$$

一定成立。

### Why The Distribution Matches

验证通过还不够，special HVZK 需要的是分布匹配。

真实执行中：

- prover 先采样 $r \xleftarrow{\$} \mathbb{Z}_q$
- verifier 再采样 $e \xleftarrow{\$} \mathcal{C}$
- prover 计算 $z = r + ex$

所以对固定 $e$，$z$ 仍然在 $\mathbb{Z}_q$ 上均匀，因为它只是把均匀随机的 $r$ 平移了一个常数 $ex$。

同时

$$
a = g^r = g^{z-ex} = g^z y^{-e}.
$$

这与 simulator 产生 $a$ 的公式完全一样。因此真实 transcript 与模拟 transcript 的联合分布一致。

这就是 “special” HVZK 的含义：simulator 可以自己选 challenge，所以它只需要匹配 honest verifier 的随机挑战分布，而不是处理任意恶意 verifier 的自适应行为。

> Key Observation.
> 在 Schnorr 里，extractor 与 simulator 都来自同一条线性关系 $z = r + ex$。两份 transcript 相减得到 witness，先选 $(e,z)$ 再倒推 $a$ 得到模拟。

## 从 Schnorr 到表示证明

Schnorr 只是单个离散对数的最小例子。真正更一般、也更贴近上一篇 Pedersen 承诺的是 proof of representation。

设公开 statement 为

$$
Y = \prod_{i=1}^n g_i^{x_i},
$$

其中 $g_1, \dots, g_n \in G$ 为公开基，witness 是向量

$$
\mathbf{x} = (x_1, \dots, x_n) \in \mathbb{Z}_q^n.
$$

relation 写成

$$
R(Y, \mathbf{x}) = 1 \iff Y = \prod_{i=1}^n g_i^{x_i}.
$$

### The Same Template, Vectorized

诚实 prover 采样随机向量

$$
\mathbf{r} = (r_1, \dots, r_n) \xleftarrow{\$} \mathbb{Z}_q^n
$$

发送

$$
A = \prod_{i=1}^n g_i^{r_i}.
$$

verifier 发送 challenge $e$。然后 prover 返回

$$
z_i = r_i + e x_i \pmod q,\qquad i=1,\dots,n.
$$

verifier 检查

$$
\prod_{i=1}^n g_i^{z_i} \stackrel{?}= A Y^e.
$$

把定义代回去就得到 completeness：

$$
\prod g_i^{z_i}
= \prod g_i^{r_i + e x_i}
= \left(\prod g_i^{r_i}\right)\left(\prod g_i^{x_i}\right)^e
= A Y^e.
$$

### Extraction Is Component-Wise

若拿到两份接受 transcript

$$
(A, e, \mathbf{z}), \qquad (A, e', \mathbf{z}'),
$$

其中 $e \ne e'$，则对每个分量都有

$$
z_i - z_i' = (e - e')x_i \pmod q.
$$

所以

$$
x_i = (z_i - z_i')(e - e')^{-1} \pmod q.
$$

也就是说，Schnorr extractor 不是一个孤立技巧，而是表示证明的坐标级版本。

### Pedersen Opening Is Already A Representation Proof

上一篇的 relation

$$
C = g^m h^r
$$

其实就是二元表示证明，witness 是 $(m,r)$。所以若要证明“我知道 Pedersen 承诺的 opening”，并不需要发明新协议，只需把上面的向量模板套到

$$
Y = C,\qquad (g_1, g_2) = (g,h),\qquad (x_1, x_2) = (m,r)
$$

上即可。

这就是 sigma 协议统一视角真正有用的地方：不同对象看起来不同，但 transcript skeleton 与提取/模拟逻辑没有变。

## Fiat-Shamir：收益与边界一起到来

现在看最常见的非交互化方式。

在交互式 sigma 协议里，challenge 是 verifier 发来的随机值 $e$。Fiat-Shamir 的想法是：不用等待 verifier，直接把 challenge 设为某个哈希值，例如

$$
e = H(x, a)
$$

或者在签名场景下写成

$$
e = H(m, x, a).
$$

于是 prover 只需输出

$$
(a, z),
$$

verifier 自己重算 $e$ 再做原本的验证方程检查。

### What We Gain

这一步显然消掉了交互，也让 sigma 骨架可以进入：

- 非交互识别/证明
- Schnorr signatures
- 一系列 transcript-based 证明系统

所以 Fiat-Shamir 常被看成“把 sigma 协议编译成非交互对象”的标准手段。

### What Changes In The Security Proof

但这里必须立刻踩刹车。Fiat-Shamir 的收益不是免费的。

交互式 Schnorr 的 special soundness 叙事是：若我有两份同 $a$、不同 $e$ 的接受 transcript，就能提取 witness。可在非交互版本里，challenge 不再来自 verifier，而是由哈希决定：

$$
e = H(x,a).
$$

这意味着“从同一个 $a$ 得到两个不同 $e$”不再是原协议里的自然事件，而变成了对随机预言机进行 rewinding 或 forking 的证明技术问题。

也就是说，proof obligation 从

- 两份 transcript 的代数消元

变成了

- 如何在随机预言机模型里制造两次不同回答，从而喂给同一个 extractor 公式

代数部分没变，模型和技术变了。

### HVZK Does Not Automatically Become Full NIZK

同样，交互式 sigma 协议里的 special HVZK 也不能直接翻译成“于是 Fiat-Shamir 后得到 full NIZK”。

原因至少有三层：

1. HVZK 只讨论 honest verifier 的随机 challenge
2. Fiat-Shamir 里的 challenge 来自 hash oracle，不是一个独立 verifier
3. 非交互零知识若要做 simulation，往往需要对 oracle 进行编程或依赖更强模型

所以：

- 交互式 sigma 的 HVZK 是关于 transcript distribution 的陈述
- Fiat-Shamir 后的零知识/知识论证安全性，是关于随机预言机模型下模拟与提取能力的陈述

这两者有关联，但不是同一句话。

### The Right Caveat

Fiat-Shamir 最常见的误读是：

> “既然 sigma 协议有 special soundness 和 HVZK，那么哈希一下就得到一个标准模型里的非交互零知识知识证明。”

这句话一般是错的。更稳妥的说法是：

> Fiat-Shamir 把 sigma 协议变成了一个在随机预言机启发式或随机预言机模型中分析的非交互对象；提取和模拟仍然沿用 sigma 的代数骨架，但证明义务与模型边界已经变化。

这也是为什么后面到了更现代的证明系统，大家不只是“把 challenge 哈希掉”，而是要非常明确地写出 transcript、oracle、commitment binding、challenge domain 和 extraction argument。

## Summary

Sigma 协议真正统一的是一套 transcript skeleton：

1. 先发一个 witness-independent 的 first message
2. 再接受 public challenge
3. 最后用线性 response 把 witness 带回验证方程

Schnorr 是这套骨架的最小例子，因为它把所有关键对象都压成一条式子

$$
g^z = a y^e.
$$

从这条式子出发：

- 两份 same-commitment、different-challenge 的接受 transcript 导出 special soundness extractor

$$
x = (z-z')(e-e')^{-1}
$$

- 先选 challenge 与 response，再设

$$
a = g^z y^{-e}
$$

导出 special HVZK simulator

同一个模板推广后就是表示证明，而上一篇的 Pedersen opening relation 本来就是一个二元表示证明。

Fiat-Shamir 则说明，这个骨架可以被折叠成非交互对象，但不是无代价的“自动编译”：交互没了，随机预言机、forking、simulation boundary 这些证明义务会立刻补上。

下一层开始进入算术化时，commit-challenge-response 的骨架不会消失，只会被搬到更复杂的多项式与约束系统上。

## References

[^schnorr]: Claus-Peter Schnorr, *Efficient Signature Generation by Smart Cards*, 1991.
[^bs]: Dan Boneh and Victor Shoup, *A Graduate Course in Applied Cryptography*, sigma protocol related sections.
[^fs]: Amos Fiat and Adi Shamir, *How to Prove Yourself: Practical Solutions to Identification and Signature Problems*, 1986.


