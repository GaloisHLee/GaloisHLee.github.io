# KZG 多项式承诺的曲线基础及其协议应用


> Reading: KZG turns polynomial objects into deployable commitments.

到 ECC 子系列的最后一篇，问题已经收缩得很具体了。前面几篇分别回答了 why not one curve、账户层为什么停在 secp256k1、pairing-friendly curves 为什么进入 verifier、以及 Ethereum 为什么在 BN254 与 BLS12-381 之间形成长期工程张力。到了 EIP-4844，pairing-friendly curve 不再只是 verifier 的数学背景，而是直接进入 data-availability commitment workflow。

这里真正要理解的，不是“BLS12-381 为什么常出现”，而是“KZG commitments 为什么会自然进入 blob workflow”。如果一个协议只需要对数据做普通哈希承诺，那么 pairing-friendly curve 完全可以不出现；但如果协议既想对多项式对象做常数大小承诺，又想对某个 evaluation claim 给出紧凑 opening proof，那么 verifier 最终就会落到 pairing equation 上，而这也是 KZG commitments and opening proofs rely on pairing-friendly curves 的根本原因。[^kzg] [^eip4844]

所以这一篇的顺序会比“直接讲 blob”更慢半步：先定义 polynomial commitment object，再写 minimal KZG opening verification equation，随后把这个对象映射到 `EIP-4844 blobs` 的 commitment / proof / verification workflow，最后把 `trusted setup` 从尾注抬成 first-class protocol constraint，并在文末给出工程实现对接。 

> Quick Note.
> This article explains why KZG commitments and opening proofs rely on pairing-friendly curves. It also gives the minimal KZG opening verification equation, shows the role of BLS12-381 in modern Ethereum data-availability commitments, maps the polynomial commitment object to the blob-commitment workflow in EIP-4844, and makes trusted setup a first-class protocol constraint rather than an afterthought.

<!--more-->

## 为什么 KZG 会进入 EIP-4844

如果把 `EIP-4844 blobs` 理解成“链上多了一种大数据对象”，会漏掉关键点。EIP-4844 真正要解决的不是把字节串丢进链里，而是让协议能够围绕这些数据对象建立可验证、可传递、可压缩的承诺接口。

这就是为什么 blob 不是普通哈希承诺。哈希可以证明“这个字节串对应这个 digest”，但它不直接提供多项式 evaluation claim 的紧凑 opening structure。KZG 则不同：它把一个多项式对象压成常数大小 commitment，同时允许 prover 对“在点 $z$ 处，值是否为 $y$”这类 claim 给出紧凑证明。

于是 BLS12-381 在 modern Ethereum data-availability commitments 里的角色就变得具体了：它不是“某条大家现在更喜欢的曲线”，而是 KZG verifier 需要的 pairing-friendly setting 的工作曲线。

## Formal Object：`KZG commitments`

### structured reference string

KZG 的起点不是消息，而是多项式 $f(X)$。为了对它做 commitment，协议首先需要一个 structured reference string。最小语境下，可以把它理解成某个隐藏标量 $\tau$ 诱导出来的一组群元素，例如

$$
g_1, g_1^{\tau}, g_1^{\tau^2}, \ldots
$$

以及配套的目标群侧对象。这里已经出现了本文必须单独抬出来的约束：`trusted setup`。因为 commitment object 不是从空气中长出来的，它依赖一个结构化参考字符串，而这正是 KZG 和普通无结构承诺的根本区别。

### 多项式承诺与 opening claim

对多项式 $f(X)$，KZG commitment 可以压成

$$
C = g_1^{f(\tau)}.
$$

这就是 `KZG commitments` 的最小对象。接下来，如果 prover 想证明

$$
f(z) = y,
$$

那么它需要给出一个 opening proof $\pi$，使 verifier 不必知道整个 $f(X)$，也不必看到 $f(\tau)$ 的秘密结构，就能检查这个 claim 是否成立。

## minimal KZG opening verification equation

### 商多项式视角

KZG opening proof 的关键在于商多项式

$$
q(X)=\frac{f(X)-y}{X-z}.
$$

这个对象存在的前提正是 $f(z)=y$。如果 claim 为真，那么 $f(X)-y$ 可以被 $(X-z)$ 整除，于是 prover 可以围绕 $q(X)$ 构造证明对象。

### pairing verification 形状

This section gives the minimal KZG opening verification equation.

在最小形状下，verifier 要检查的是

$$
e(C / g_1^y, g_2) \stackrel{?}= e(\pi, g_2^{\tau-z}).
$$

这就是 `minimal KZG opening verification equation`。

一旦这条式子出现，为什么 `KZG commitments and opening proofs rely on pairing-friendly curves` 就已经不需要再靠口号解释了。左边编码的是 commitment 与 claimed value 的差，右边编码的是商多项式与结构化参考字符串的关系。verifier 真正比较的，不再是普通群里的签名等式，而是 pairing equation。

## 从 polynomial commitment object 到 blob workflow

### blob commitment、proof 与 verifier

This section gives the mapping from polynomial commitment object to blob-commitment workflow in EIP-4844.

把抽象对象压到 `blob-commitment workflow in EIP-4844`，可以得到下面这张表：

| KZG 对象 | 协议角色 | EIP-4844 语境 |
| --- | --- | --- |
| 多项式 $f(X)$ | 被承诺的数据对象 | blob 对应的数据多项式表示 |
| commitment $C$ | 常数大小承诺 | blob commitment |
| evaluation claim $(z, y)$ | 想验证的点值关系 | 某个 opening query |
| proof $\pi$ | opening proof | blob verification proof |
| pairing verifier | 检查 claim 是否成立 | 共识/验证流程中的 KZG verification |

这张表后面的关键结论是：EIP-4844 不是“先发明了 blob，再随手挑了 KZG”；而是 data-availability commitment workflow 需要一个既能紧凑承诺多项式对象、又能紧凑验证 opening claim 的接口，而 KZG 正好提供了这组对象。

### `BLS12-381` 在工作流里的角色

`BLS12-381` 在这里的角色非常直接：它提供 KZG verifier 所依赖的 pairing-friendly setting。换句话说，curve choice 不是外围实现细节，而是 commitment object 能否被实际验证的一部分。

这也是为什么上一章的 Ethereum 曲线演化不能单独看。`BLS12-381` 在 modern Ethereum data-availability commitments 中的重要性，不是因为它“流行”，而是因为 blob commitment 和 opening proof 的 verifier 最终消费的就是这一类 pairing-friendly curve interface。

## `trusted setup` 不是尾注

### 为什么 setup 在 KZG 里是结构性约束

很多工程叙事会把 trusted setup 写成“当然还有个 ceremony”。这种写法太轻。更准确的说法是：在 KZG 里，setup assumptions 是 commitment object 的一部分。如果结构化参考字符串的生成过程不可信，那么整个安全边界都会被污染。

This article makes trusted setup a first-class protocol constraint rather than an afterthought.

也就是说，`trusted setup` 不是 KZG 旁边的行政手续，而是协议本体的一部分。它和 pairing-friendly curve 一样，都是 verifier interface 的组成条件。

### 工程师该如何理解 toxic waste 风险

如果用工程语言重写这个问题，真正需要问的是：

- setup ceremony 的假设是什么
- 哪些对象必须被销毁或不可恢复
- library 和 integration 是否默认了 setup 输入永远可信

只要这些前提没有被明确写进协议和实现边界里，KZG integration 就是不完整的。

## 工程实现对接

### library boundary 与 serialization

现实系统里最先会出问题的，通常不是 pairing 公式写错，而是 object boundary 不清：

- commitment、proof 和 evaluation input 的 serialization 是否一致
- library 是否把 BLS12-381 群对象与 KZG object 清晰分开
- verifier 调用方是否明确知道哪些输入已经做 subgroup checks

这些看起来像实现细节，但实际上直接决定 blob verification integration 是否可靠。

### `blob-verification integration`

对协议工程师来说，`blob-verification integration` 真正要关心的是：在哪一层消费 commitment、在哪一层消费 opening proof、哪一段系统承担 verification，以及 setup assumptions 是否被正确继承到运行环境里。

如果这些接线位置不清，系统很容易退化成“数学对象是对的，部署对象是糊的”。这也是为什么 KZG 的工程实现对接必须同时包含：

- 曲线与 pairing library 边界
- setup ceremony 假设
- serialization 与 proof object 约束
- verifier workflow 的责任划分

### 作为 ECC 子系列的收尾

到这里，ECC 子系列实际上收束成了一条很稳定的判断线：

- 账户层问题先问普通离散对数群接口
- verifier compression 问 pairing-friendly curve
- 进入 Ethereum 工程现实后，再问 precompile、gas、deployment friction
- 进入 KZG / blob workflow 后，再把 `trusted setup` 和 object boundary 拉回主轴

也就是说，曲线从来不是孤立选型，而是协议接口、实现边界和部署现实的交点。

## 总结

KZG 之所以会成为 EIP-4844 blob workflow 的核心，不是因为它“先进”，而是因为它把 polynomial commitment object 和 opening proof 压成了实际可消费的 verifier interface。于是 pairing-friendly curve 在这里不再是背景知识，而是 commitment / proof / verification 整条链上的工作曲线。

因此这一篇最想固定下来的判断线是：如果协议要对多项式对象做紧凑承诺，并对 evaluation claim 做紧凑验证，那么 KZG 会自然进入画面；而一旦 KZG 进入画面，`BLS12-381` 与 `trusted setup` 也就不再是可选注释，而是协议主轴本身。

作为 ECC 子系列收尾，这也把前面 6 篇连成了一条线：不同曲线不是被“偏好”出来的，而是被账户接口、verifier compression、Ethereum execution surface 和 deployment constraints 一起推出来的。

[^kzg]: Aniket Kate, Gregory M. Zaverucha, and Ian Goldberg, *Constant-Size Commitments to Polynomials and Their Applications*.
[^eip4844]: EIP-4844, *Shard Blob Transactions*.

