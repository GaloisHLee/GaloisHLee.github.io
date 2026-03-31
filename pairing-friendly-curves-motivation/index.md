# Pairing-Friendly Curves 的数学结构与协议动机


> Reading: pairing is a verifier interface, not a prestige upgrade.

前 3 篇已经把 ECC 子系列的前半段压实了：Web3 为什么不会只用一条曲线，账户层为什么长期停在 secp256k1，以及 signer-side 风险如何从 ECDLP 走向 nonce leakage。到这里，新的问题自然出现：既然账户层不需要 pairing，为什么 BLS signatures、SNARK verifier 和 KZG 却总会把 pairing-friendly curves 拉进来？

关键不在“这类曲线更先进”，而在“这类协议的 verifier 从一开始就需要另一种代数接口”。普通离散对数群擅长表达签名关系；pairing-friendly setting 则擅长把分散在多个群元素、多个约束甚至多个消息上的关系压缩成少数几个 pairing checks。也就是说，这里发生的不是曲线选型审美，而是 protocol verification compression。[^bls] [^kzg]

所以这一篇不会把 pairing 写成完整教材。更有用的顺序是：先定义最小 bilinear map properties，再给出一个 minimal pairing equation pattern，随后分别说明 BLS signatures、SNARK verifier intuition 和 KZG opening verification 为什么会消费同一类接口。最后再把这些数学对象压成工程实现对接边界。 

> Quick Note.
> This article defines the minimal bilinear map properties needed for protocol use. It also explains why pairings enable aggregate signature verification and polynomial opening verification, gives the minimal pairing equation pattern, and shows the mapping from bilinearity to protocol verification compression. It deliberately avoids expanding into full Miller-loop or final-exponentiation implementation details.

<!--more-->

## 为什么某些协议一开始就需要 pairing

先把前 3 篇的结论拿回来。对账户签名、交易签名或普通公钥关系，verifier 要检查的通常只是：

$$
sG \stackrel{?}= R + eP
$$

或者 ECDSA 那种等价的标量关系。这里的对象仍然是单一群里的标量乘法和点加法。ordinary elliptic-curve groups 已经足够，不需要 pairing。

但有些协议要做的是另一件事：它们希望把原本分散的多个关系压成紧凑验证。这种压缩不只是“少做几次乘法”，而是把验证目标改写到一个新的目标群里，让多个输入关系在双线性映射下发生对齐。此时，普通离散对数群接口就不够了。

更直接地说，this is why pairings enable aggregate signature verification and polynomial opening verification. 协议不是先选择了一条很炫的曲线，再去找应用；而是 verifier 想要一个能够把高维关系压缩成少数检查的接口，结果自然落到了 pairing-friendly curves。

## Formal Definition：最小 bilinear map properties

### 群对象与映射

设 $G_1$、$G_2$、$G_T$ 是阶为素数 $r$ 的群。pairing 的最小对象定义是一个映射

$$
e: G_1 \times G_2 \rightarrow G_T.
$$

如果只从协议实现角度出发，这已经是第一步分叉：输入不再是“同一个群里的两个点”，而是两个源群中的元素，输出落到另一个目标群。

### `bilinearity`、non-degeneracy 与 computability

This section defines the minimal bilinear map properties needed for protocol use.

协议层真正关心的最小性质通常只有三条。

第一条是 bilinearity。对任意 $P \in G_1$、$Q \in G_2$ 和标量 $a,b \in \mathbb{F}_r$，有

$$
e(aP, bQ) = e(P, Q)^{ab}.
$$

第二条是 non-degeneracy，也就是 pairing 不是把所有输入都压成单位元。存在某些 $P,Q$ 使得

$$
e(P,Q) \neq 1.
$$

第三条是 efficient computability。协议当然还要求这个映射是可计算的，否则双线性只停在抽象定义里，没有 verifier 价值。

把这三条放一起看，真正重要的是：pairing 允许 verifier 在目标群里看到源群标量关系的乘法结构。这就是双线性映射成为协议工具，而不是纯数学装饰的原因。

## 从双线性到 verifier compression

### minimal pairing equation pattern

This section gives the minimal pairing equation pattern.

很多协议最后都会收束到同一种形状：

$$
e(P_1, Q_1)\cdot e(P_2, Q_2)\cdots e(P_k, Q_k) \stackrel{?}= 1_{G_T}.
$$

或者等价地，把若干项移到等式两边：

$$
e(A, B) \stackrel{?}= e(C, D).
$$

这两个式子的重要性不在于它们“长得统一”，而在于大量不同来源的约束最终都能被搬到目标群里比较。verifier 不再逐条重做所有底层关系，而是检查一个被压缩后的 pairing product relation。

### 为什么双线性可以压缩验证关系

This section explains the mapping from bilinearity to protocol verification compression.

双线性的核心收益，在于标量关系可以被带进目标群指数上。如果协议原本需要验证很多条“这个标量乘积、那个多项式商、那组签名公钥关系”是否同时成立，那么 pairing 允许这些关系先在源群中被编码，再在目标群中被合并。

这就是 verifier compression 的直觉版本：

- 协议先把原始关系编码到群元素中
- 双线性把多个源群关系搬运到目标群乘法里
- verifier 只需要检查少数几个 pairing equation

这并不意味着 pairing 免费。它只是把“很多普通关系”换成“少数昂贵但结构更强的关系”。为什么这仍然值得做，取决于协议是更在乎 prover 端结构，还是更在乎 verifier 端紧凑性。

## BLS signatures、SNARK verifier 与 KZG 的三种消费方式

### `BLS signatures`

`BLS signatures` 是 pairing-friendly curves 最直接的协议例子之一。设私钥为 $x$，公钥为 $X=g_2^x$，消息映射到群元素 $H(m)\in G_1$，签名为

$$
\sigma = H(m)^x.
$$

验证关系写成

$$
e(\sigma, g_2) \stackrel{?}= e(H(m), X).
$$

一旦进入聚合语境，多个签名和多个公钥仍然可以被压成少数 pairing checks。这里真正被消费的是 pairing 的 bilinearity，而不是“某条曲线更适合签名”这种笼统说法。

### `SNARK verifier intuition`

`SNARK verifier intuition` 不要求本文把 Groth16 或别的 pairing-based SNARK 全推一遍。更有用的直觉是：证明系统把原本高维、跨多项式、跨约束系统的关系编码进若干群元素，最后 verifier 只需要检查一个 pairing product equation 是否成立。

也就是说，SNARK verifier 之所以喜欢 pairing，不是因为 pairing 本身神秘，而是因为它能把“很多约束同时成立”压缩成“少数群关系在目标群里对齐”。这和账户层签名完全不是一个接口层次。

### KZG opening verification

KZG 更能把 pairing-friendly setting 的必要性说清。设 commitment 为 $C$，声称多项式在点 $z$ 处取值 $y$，proof 为 $\pi$。当 prover 使用商多项式

$$
q(X)=\frac{f(X)-y}{X-z}
$$

构造 opening proof 时，verifier 最终要检查的可以压成

$$
e(C / g_1^y, g_2) \stackrel{?}= e(\pi, g_2^{\tau-z}).
$$

这里已经不是普通离散对数群上的“签名正确性”，而是 polynomial opening verification 被压成 pairing equation。于是 pairing-friendly curves 成为协议接口的一部分，而不是实现选配。

## 一张协议职责表

如果把三类协议并排写，会更容易看清 pairing-friendly curves 的角色。

| 协议 | verifier 想检查什么 | pairing 在做什么 | 为什么普通群不够 |
| --- | --- | --- | --- |
| BLS signatures | 签名与公钥、消息的一致性 | 把签名关系搬到目标群里比较 | 需要紧凑聚合验证 |
| SNARK verifier | 多项式/约束系统编码后的证明关系 | 压缩很多约束为 pairing product checks | verifier 需要高密度压缩 |
| KZG | opening claim 是否与 commitment 一致 | 把商多项式关系压成双线性检查 | opening verification 天然依赖 pairing |

这张表背后的共同点只有一个：protocol motivation 不是“想用更新的曲线”，而是“想要一个能压 verifier 的双线性接口”。

## 工程实现对接

### subgroup checks 与 serialization

pairing-friendly curves 真正进入工程世界后，第一层会出问题的通常不是 bilinearity 定义本身，而是输入边界：

- 点是否在正确子群里
- serialization / deserialization 是否严格
- cofactor clearing 是否在正确位置发生
- API 是否允许上层错误混用 G1、G2 和 GT 对象

这些问题看起来像“实现细节”，但实际上直接决定 pairing verifier 是否在错误对象上工作。

### curve library、precompile 与 deployment surface

在真实 Web3 系统里，curve library 和 deployment surface 也会反向塑造协议可用性：

- 链上有没有相关 precompile
- 共识层和执行层是否消费同一套曲线接口
- library 是否对 subgroup checks、serialization 和 constant-time 做了明确边界

这也是下一篇要继续进入 BN254 和 BLS12-381 演化路径的原因。协议上需要 pairing，不等于系统里已经有可部署、可维护、可审计的 pairing surface。

### 本文故意不展开什么

This article avoids expanding into full Miller-loop or final-exponentiation implementation details.

原因很简单：这一篇的目标是固定 pairing-friendly curves 的协议动机，而不是写一本 pairing 算法教程。到这个阶段，读者需要先搞清：

- bilinearity 为什么能压 verifier
- 为什么 BLS、SNARK verifier 和 KZG 会共用这一类接口
- 为什么 pairing-friendly curves 不是普通曲线的升级版

在此之前直接冲进 Miller loop 或 final exponentiation，会把“为什么需要它”这条主线打散。

## 总结

pairing-friendly curves 不是更先进的普通曲线，而是另一类协议接口的代数舞台。只要协议要验证的还是普通签名关系，普通离散对数群就够了；一旦 verifier 想要把分散的关系压缩成少数检查，pairing 就会自然进入系统设计。

所以这一篇最想固定下来的判断线只有一句：pairing-friendly curves 的价值，不在曲线名称本身，而在 bilinearity 如何把协议验证从“逐条重做关系”改写成“检查被压缩后的目标群等式”。

下一篇进入 Ethereum 中配对友好曲线的演化路径时，焦点就会从数学接口继续落到工程现实：为什么 BN254 先落地，为什么 BLS12-381 形成升级压力，以及 precompile 与 gas 成本如何反向决定密码学方案的可部署性。

[^bls]: Dan Boneh, Ben Lynn, and Hovav Shacham, *Short Signatures from the Weil Pairing*.
[^kzg]: Aniket Kate, Gregory M. Zaverucha, and Ian Goldberg, *Constant-Size Commitments to Polynomials and Their Applications*.

