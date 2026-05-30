# secp256k1 的协议定位与签名机制


> Reading: same curve, different signature interfaces.

上一篇已经把 Web3 里的曲线职责图画出来了。这一篇只把账户层单独拎出来：为什么 Bitcoin 和 Ethereum 长期把 secp256k1 当作账户与交易签名曲线，而不是换到 pairing-friendly curve；以及为什么在同一条曲线上，ECDSA 和 Schnorr 会导向不同的协议接口和工程边界。

账户层消费的不是 pairing，也不是多项式 opening proof，而是普通离散对数群上的签名验证关系。这件事决定了 secp256k1 的核心价值并不在“它是哪条曲线”，而在“它背后的实现生态、签名接口和安全边界是否足够稳定”。因此这篇文章真正要比较的对象不是 secp256k1 vs BLS12-381，而是 secp256k1 上的 ECDSA vs Schnorr。[^sec2] [^bip340]

如果把账户层写成“曲线教程”，重点会跑偏。更有用的顺序是：先说明 secp256k1 为什么留在账户层，再写 ECDSA 的最小验证关系和它对 nonce 的敏感性，再写 Schnorr 如何在同一条曲线上改写签名接口，最后把这些差异落到 Bitcoin、Ethereum、`libsecp256k1`、RFC 6979 与钱包实现的现实接口上。[^rfc6979] [^libsecp]

> Quick Note.
> This article separates ECDSA and Schnorr verification logic while keeping both on the same curve family. It also explains protocol fit for Bitcoin and Ethereum account usage, and connects implementation constraints to constant-time scalar arithmetic.

<!--more-->

## 为什么账户层长期绑定 secp256k1

在 Web3 语境里，账户层首先要解决的问题不是“如何压 verifier”，而是“如何让某个公开账户标识稳定地绑定一个可验证的签名接口”。这意味着账户层消费的是普通离散对数群上的签名关系，而不是 pairing-friendly curve 上的双线性验证结构。

secp256k1 在这里之所以长期稳定，不是因为它抽象上“最先进”，而是因为它同时满足了几件现实事情：

- 它有稳定的标准参数来源
- 它在 Bitcoin 和 Ethereum 里长期部署
- 它拥有高质量实现和成熟审计历史
- 它足以支撑账户层所需的签名验证接口

换句话说，账户层没有动机为了不存在的 verifier 压缩需求去切换到 pairing-friendly curve。账户层真正需要的是一个实现成熟、常数时间、接口边界清晰的 prime-order group setting，而 secp256k1 恰好已经满足这一点。[^sec2]

## ECDSA：历史主力接口

在 secp256k1 上，账户层最熟悉的历史接口是 ECDSA。它不是因为曲线本身特别“像 ECDSA”才绑定在一起，而是因为长期系统实现首先围绕它组织起来。

### 最小签名与验证关系

设私钥为 $d$，公钥为

$$
P = dG.
$$

对消息哈希 $z$，签名者选取 nonce $k$，并计算

$$
R = kG, \qquad r = x(R) \bmod n,
$$

再定义

$$
s = k^{-1}(z + rd) \bmod n.
$$

这就是 ECDSA 的最小签名关系。验证时，给定 $(r,s)$，验证者计算

$$
u_1 = z s^{-1}, \qquad u_2 = r s^{-1},
$$

然后检查

$$
r \stackrel{?}= x(u_1 G + u_2 P) \bmod n.
$$

This is the minimal ECDSA verification equation.

这个形式已经足够说明账户层的接口特征：验证者只需要普通群上的标量乘法和点加法，不需要 pairing，也不需要额外的 verifier algebra。

### 为什么 ECDSA 对 nonce 异常敏感

ECDSA 的工程负担，很大一部分都来自 nonce。

从

$$
s = k^{-1}(z + rd)
$$

出发，可改写为

$$
k = s^{-1}(z + rd) \bmod n.
$$

这意味着只要 $k$ 有重复、偏置或部分泄漏，私钥 $d$ 就可能被直接恢复或转化为后续的 hidden-number style 问题。也就是说，ECDSA 的安全边界不只取决于离散对数假设，还强烈依赖 nonce 生成和实现侧信道。[^rfc6979]

所以 RFC 6979 的意义并不是“给 ECDSA 一个可选优化”，而是在工程上把随机 nonce 生成这个脆弱点尽量收紧为 deterministic procedure。它不改变 ECDSA verifier 的数学形式，但明显改变了实现风险面。

### 可塑性与 low-s normalization

ECDSA 还有一个工程上很重要、但在数学笔记里经常被一笔带过的点：可塑性。

因为在群阶模 $n$ 下，若 $(r,s)$ 是有效签名，那么 $(r, n-s)$ 也会对应同一验证关系。这意味着如果协议层不加额外约束，同一个签名语义可能有两个编码表示。

账户系统因此通常需要额外规则来约束签名规范化，最常见的就是 low-s normalization。这里的重点不是“数学上这很优雅”，而是账户层需要稳定的 transaction identity 和签名编码接口，而不是把可塑性问题留给上层业务来兜底。

## Schnorr：同一条曲线上的接口升级

Schnorr 并没有把 secp256k1 换掉。它做的是另一件事：在同一条曲线上，把签名接口改写成更线性的形式。BIP-340 的意义，正是在这里。[^bip340]

### 最小验证关系

Schnorr 签名在最小形式下可写成：

$$
R = kG, \qquad e = H(R \| P \| m), \qquad s = k + ed \bmod n.
$$

验证时检查

$$
sG \stackrel{?}= R + eP.
$$

This is the minimal Schnorr verification equation.

和 ECDSA 相比，这个关系最大的特征不是“更短”或者“更现代”，而是它更线性。验证器看到的是一个直接的群等式，而不是包含模逆和 $x$ 坐标投影的间接关系。

### 线性、批量验证与多签组合

一旦验证关系变成

$$
sG = R + eP,
$$

很多高层协议就自然多了。线性结构意味着多个公钥和多个 nonce 的组合可以更直接地进入多签和批量验证逻辑。BIP-340 里就明确把 batch verification 作为正式接口的一部分，而这正是账户层协议升级最重要的收益之一。[^bip340]

这里要注意，Schnorr 的价值不是“把 secp256k1 改造成另一条曲线”，而是让同一条曲线更适合：

- 多签和门限签名的上层构造
- 更直接的 batch verification
- 更清晰的编码和验证接口

所以这一节最准确的标题不是“新曲线替代旧曲线”，而是“同一条曲线上的协议接口升级”。

The contrast on malleability and composability is the real protocol difference here: ECDSA carries historical malleability pressure and weaker composition properties, while Schnorr is built around a linear relation that is far friendlier to composability, batch verification, and multisignature constructions.

### BIP-340 的 nonce 与 domain separation

Schnorr 也不是“公式线性了，所以实现自然更简单”。BIP-340 明确给了自己的 nonce 生成和 domain separation 约束。

尤其要注意两点：

1. 它使用的是 synthetic nonce 设计，而不是简单照搬 RFC 6979
2. 它把 challenge 和消息处理放进更明确的 tagged hash / domain separation 语境

这说明同样在 secp256k1 上，ECDSA 和 Schnorr 不能共享一套含糊的“签名实现常识”。曲线相同，不代表 nonce、编码、消息预处理和验证接口都能共用。

## Bitcoin 与 Ethereum 如何不同地消费 secp256k1

如果只写 ECDSA 和 Schnorr 的公式，这篇文章还不够工程。更关键的是：相同曲线，在 Bitcoin 和 Ethereum 里被消费成了不同的账户层接口。

### Bitcoin：曲线不变，接口升级

Bitcoin 的演化可以概括成一句话：底层仍是 secp256k1，但签名接口从历史上的 ECDSA 走向了 BIP-340 Schnorr。

这意味着 Bitcoin 并没有说“账户层不再需要 secp256k1”，而是说“账户层仍然需要这条曲线，但更适合在其上采用更线性的签名接口”。因此，曲线 continuity 和 protocol upgrade 在这里是同时成立的。

### Ethereum：EOA、ECDSA 与 `ecrecover`

Ethereum 的账户层消费方式不同。对 externally owned accounts 来说，执行层长期围绕 ECDSA/secp256k1 组织，并通过 `ecrecover` 暴露公钥恢复相关接口。Yellow Paper 在相关附录里把 `ECDSARECOVER` 作为执行环境的一部分来定义。[^yellowpaper]

这意味着 Ethereum 对 secp256k1 的消费更偏“账户恢复与交易签名接口”，而不是像 Bitcoin 那样把 Schnorr 作为新的主签名协议引入链上主流程。

所以“Bitcoin 和 Ethereum 都使用 secp256k1”这句话虽然没错，但工程上太粗。更准确的说法是：

- Bitcoin：同一曲线下，主签名接口在升级
- Ethereum：同一曲线下，账户恢复和 ECDSA 接口仍是主路径

曲线相同，protocol consumption 不同。

## 工程实现对接

到这里，真正需要落地的是实现边界，而不是再多记几条公式。

### `libsecp256k1` 的工程哲学

账户层最怕的不是“曲线名写错”，而是实现层在标量算术、常数时间、nonce、编码和 API 边界上出问题。

For secp256k1 systems, implementation constraints connect directly to constant-time scalar arithmetic.

`libsecp256k1` 的长期价值就在于，它把账户层真正脆弱的部分当作一等问题来处理：

- 常数时间标量运算
- 明确的 signing / verification / key handling 边界
- 对 malformed inputs 和边界条件的严格处理
- 为上层协议保留稳定接口，而不是把细节泄漏给业务代码

因此在账户系统里，安全性常常首先取决于“是不是在消费一个高质量 secp256k1 implementation”，其次才是你对曲线名称有没有背熟。

### RFC 6979、synthetic nonce 与 side-channel

这一节最容易被写糊。更准确的区分是：

- ECDSA 工程上常依赖 RFC 6979 把 nonce 生成收紧
- BIP-340 Schnorr 有自己的 synthetic nonce 处理语境

二者都在回答同一个工程问题：nonce 不应成为可恢复私钥的脆弱入口。但它们的具体处理方式并不相同，不能把“deterministic nonce”当成一个无差别总标签。

此外，即使 nonce 方案设计正确，side-channel 和 fault injection 仍然可能在真实设备上打开额外泄漏面。这也是为什么账户层实现一定要把 constant-time 和 signing environment isolation 当作一等公民。

### 审计时该检查什么

如果要把这一篇压成审计清单，我会优先看下面几项：

- nonce 生成是否和协议规范一致
- 是否存在重复签名状态、弱随机数或跨上下文复用
- 是否有 low-s normalization 和签名编码约束
- 是否错误混用了 ECDSA 与 Schnorr 的消息处理和 domain separation
- 是否直接暴露了不必要的底层 secp256k1 接口给业务代码

对账户层来说，这些问题比“曲线参数有没有抄错”更现实。

## 总结

secp256k1 在 Web3 里最重要的角色，不是“某条著名曲线”，而是账户层长期消费的那条曲线。

在这条曲线上：

- ECDSA 是历史主力接口
- Schnorr 是同一曲线上的协议接口升级
- Bitcoin 和 Ethereum 使用了相同曲线，但并没有消费成同一种签名协议形态

所以这一篇真正想说明的不是“secp256k1 比谁更强”，而是：

> 账户层首先绑定的是一个普通离散对数群接口；曲线家族稳定之后，真正决定协议性质和工程边界的是签名机制、nonce 处理、实现质量和执行环境接口。

下一篇再沿这条线往下走，就会自然进入安全性分析：一旦 nonce、side-channel 或 trace 泄漏进入系统，ECDSA 为什么会落到 HNP、EC-HNP 与 EHNP 这类问题。

[^sec2]: SEC Group, *SEC 2: Recommended Elliptic Curve Domain Parameters*, Version 2.0. https://www.secg.org/sec2-v2.pdf
[^bip340]: Pieter Wuille et al., *BIP-340: Schnorr Signatures for secp256k1*. https://bips.xyz/0340
[^rfc6979]: Thomas Pornin, *RFC 6979: Deterministic Usage of the Digital Signature Algorithm (DSA) and Elliptic Curve Digital Signature Algorithm (ECDSA)*. https://www.rfc-editor.org/rfc/rfc6979.html
[^libsecp]: `bitcoin-core/secp256k1` repository. https://github.com/bitcoin-core/secp256k1
[^yellowpaper]: Gavin Wood, *Ethereum: A Secure Decentralised Generalised Transaction Ledger (Yellow Paper)*. https://ethereum.github.io/yellowpaper/paper.pdf

