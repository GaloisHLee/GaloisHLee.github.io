# Web3 语境下椭圆曲线密码学的系统分工


> Reading: this is a system map, not an ECC primer.

在 Web3 里谈椭圆曲线密码学，最常见的误读是把所有场景压成同一个问题: “既然都是 ECC，为什么不选一条最强的曲线统一掉？”这个问题本身就错了。账户签名、共识聚合签名、链上 zkSNARK 验证、EIP-4844 里的 KZG commitment，虽然都使用了椭圆曲线上的群对象，但它们消费的代数接口并不相同，因此几乎不会自然收敛到同一条曲线。

如果只看“安全位数”，现实会显得很奇怪。Bitcoin 和 Ethereum 的账户层长期绑定在 secp256k1；Ethereum 的链上 zk verifier 历史上却依赖 BN254/alt_bn128 预编译；到了 BLS 聚合签名与 KZG，又几乎总会碰到 BLS12-381。这里真正决定系统形态的，不只是密码学安全性，还包括预编译、gas 成本、标准化、已有库、证明系统接口，以及协议是否依赖 pairing。[^bip340] [^eip196] [^eip4844]

所以这一篇不重讲椭圆曲线的群律，也不把三条曲线写成百科词条。更有用的做法是先画出一张职责图: 哪些系统只需要普通离散对数群，哪些系统必须进入 pairing-friendly curve，哪些场景看似是“曲线选择”，其实首先是“链上可用性和 verifier 成本”的问题。到文末，再把这张图压成工程实现对接清单。

> Quick Note.
> This article explicitly distinguishes ordinary elliptic-curve groups from pairing-friendly curves. It also states the historical role of BN254/alt_bn128 in Ethereum and contrasts it with the modern role of BLS12-381. Finally, it treats trusted setup as a protocol-level constraint rather than an intrinsic ECC property.

<!--more-->

## 先回答工程问题: Web3 里并不是一个 ECC 接口

先把“ECC in Web3”拆成四类问题。

1. 账户与交易签名
2. 共识层或聚合场景中的多签名验证
3. 链上 zk 证明验证
4. 数据可用性或多项式承诺验证

这四类问题共享的只是“群对象出现在协议里”，不共享“验证器真正检查的代数关系”。

对账户签名，验证器通常只需要确认某个公开密钥对应的离散对数关系是否满足签名方程。以 Schnorr 为例，验证条件可以写成

$$
sG \stackrel{?}= R + eP.
$$

这里没有 pairing，也没有多项式 opening。验证器要做的事情，本质上仍然是普通椭圆曲线群上的标量关系检查。

但到了 pairing-friendly systems，验证器消费的对象变了。最小形式可以压成

$$
e(P_1, Q_1)\cdot e(P_2, Q_2)\cdots e(P_k, Q_k) \stackrel{?}= 1,
$$

或者某种等价的 pairing equation。这个接口天然适合 BLS 聚合签名、Groth16 verifier，以及 KZG opening verification，因为这些协议都希望把某个“高维关系”压缩成少数几个双线性检查。

所以第一条判断线已经很明确:

> 不是所有 Web3 协议都需要 pairing；只要验证目标还是普通离散对数群上的签名关系，普通曲线就够了。只有当协议验证本身需要双线性映射时，pairing-friendly curves 才成为必要对象。

换句话说，ordinary elliptic-curve groups 适合账户与普通签名验证；pairing-friendly curves 则服务于需要双线性验证结构的协议。这里讨论的不是哪一类曲线“更先进”，而是哪一类代数接口更匹配具体 protocol instance。

## secp256k1: 账户与交易签名层的保守核心

secp256k1 是账户层事实上的保守核心。它的参数来自 SEC 2 标准，长期服务于 Bitcoin 和 Ethereum 的账户签名语境。[^sec2]

这一点容易被说成“因为它最安全”或者“因为它最流行”，但更准确的说法是: 它足够安全、部署极深、库实现极成熟，而且账户层根本不需要 pairing 提供的那套结构。

### 同一条曲线，不同签名机制

账户层最熟悉的两套接口是 ECDSA 和 Schnorr。它们都工作在 secp256k1 上，但协议性质不同。

ECDSA 的验证关系可以写成

$$
r \stackrel{?}= x\big(u_1 G + u_2 P\big) \bmod n,
$$

其中

$$
u_1 = z s^{-1}, \qquad u_2 = r s^{-1}.
$$

Schnorr 的验证关系则更线性:

$$
sG \stackrel{?}= R + H(R \| P \| m)P.
$$

同一条曲线上，为什么会有两类签名？因为账户层需要的不是“最复杂的代数结构”，而是一个可靠的离散对数群接口，然后再在其上选择更适合协议组合、批量验证、抗可塑性的签名方案。BIP-340 迁移到 Schnorr，改变的是签名接口的协议性质，而不是底层曲线家族。[^bip340]

### 为什么账户层不需要 pairing

账户层验证的 instance 很简单: “这个公钥对应的私钥持有者，是否产生了这条签名？”它需要的是标准椭圆曲线群上的标量乘法和群加法，而不是双线性映射。

如果强行把 pairing-friendly curve 用到普通账户层，通常得不到明显收益，反而会引入更大的实现复杂度、不同的库生态、不同的参数与审计面。账户层的重心从来都不是“更适合压 verifier”，而是:

- 实现稳定
- 接口清晰
- 硬件和软件钱包生态成熟
- 常数时间实现和侧信道处理成熟

这也是为什么 `libsecp256k1` 这类库会成为事实标准。工程上真正被反复打磨的是标量算术、endomorphism 优化、常数时间分支控制、签名与验证 API 的边界，而不是把账户系统改造成 pairing consumer。[^libsecp]

## BN254/alt_bn128: Ethereum 链上 zk 验证的历史入口

如果从账户签名跳到链上 zk 验证，系统目标已经变了。

以 Groth16 为代表的链上 zkSNARK verifier，最关键的不是“如何签名”，而是“如何把证明验证压缩到智能合约可承受的 gas 范围里”。Ethereum 历史上选择 BN254/alt_bn128，不是因为它在抽象意义上“最现代”，而是因为执行层最终给了它 precompile。[^eip196] [^eip197]

### EIP-196 与 EIP-197 打开的是什么能力

EIP-196 为 alt_bn128 提供了加法和标量乘预编译，EIP-197 则进一步提供了 pairing check 预编译。结果是: 合约终于能在可接受的 gas 范围内验证配对方程。

这件事的意义不在“Ethereum 支持了一条新曲线”，而在“Ethereum 执行层首次把 pairing verification 变成了一个可以消费的链上接口”。从那之后，Groth16 一类 verifier 才真正具备实用空间。

换句话说，BN254/alt_bn128 的历史角色不是“Ethereum 最爱的曲线”，而是:

> 它是第一条被执行层以 precompile 形式认真暴露出来的 pairing-friendly curve，因此成为链上 zk verifier 的早期现实入口。

如果把历史角色和现代角色并排写，结论会更清楚: BN254/alt_bn128 的历史 role 是 Ethereum execution layer 上的早期 zkSNARK verifier curve；BLS12-381 的 modern role 则是 aggregate signatures 与 KZG/data-availability infrastructure curve。

### Gas 成本会反向塑造密码学可用性

只谈“数学上能不能验证”还不够，链上系统还要回答“gas 上能不能验证”。EIP-1108 下调了 alt_bn128 相关 precompile 的 gas 成本，这进一步说明曲线选择不是纯粹的数学决策，还是执行层经济模型的一部分。[^eip1108]

这也是 Web3 语境和教科书语境的第一个明显分叉。教科书会说 pairing-friendly curve 能支持某类 verifier；Web3 工程师必须再追问一句: 在这个执行环境里，pairing 相关操作是否被暴露成了真实可用的接口，且成本是否可承受？

## BLS12-381: 聚合签名与 KZG 的现代基础设施曲线

如果 BN254/alt_bn128 主要代表“早期链上 pairing verifier 的现实妥协”，那么 BLS12-381 更像现代基础设施曲线。它频繁出现在两个语境里:

- BLS aggregate signatures
- KZG polynomial commitments

这两者看起来相距很远，一个是签名，一个是多项式承诺；但它们共享的关键代数接口正是 pairing。

### BLS 聚合签名为什么需要 pairing

BLS 的吸引力在于聚合后仍能把多个签名的正确性压成紧凑验证关系。它依赖双线性映射把“多个签名对应多个消息和公钥”的关系挤到少数 pairing checks 中。[^bls]

这里要注意，pairing-friendly curve 不是“更先进的普通曲线”，而是为了支持这样的双线性验证接口而构造出来的曲线家族。它的价值不在一般性的签名速度神话，而在 verifier 压缩能力。

### KZG 为什么天然落在 pairing-friendly curves 上

KZG 更能说明“协议接口决定曲线类型”这件事。

对多项式 $f(X)$ 的 KZG commitment，本质上是在 structured reference string 上承诺 $f(\tau)$。当 prover 要证明 $f(z)=y$ 时，会构造商多项式

$$
q(X)=\frac{f(X)-y}{X-z},
$$

并把 opening claim 压成一个 pairing verification equation。最小形状可以写成

$$
e(C / g_1^y, g_2) \stackrel{?}= e(\pi, g_2^{\tau - z}).
$$

这意味着 KZG 不是“恰好常和 BLS12-381 一起出现”，而是它的 opening verification 本来就需要 pairing-friendly setting。[^kzg]

### 从 EIP-2537 到 EIP-4844

EIP-2537 试图把 BLS12-381 相关运算以 precompile 形式暴露给 Ethereum 执行层，说明生态已经明显感受到 BLS12-381 的基础设施压力。[^eip2537]

而 EIP-4844 则把 KZG commitments 放进 blob transaction 与数据可用性语境，进一步把 BLS12-381 从“共识层和密码学论文里的曲线”推进为现代 Ethereum 基础设施的组成部分。[^eip4844]

这里再强调一次: BLS12-381 的地位，不是来自“它比别的曲线更新”，而是来自两个非常具体的协议需求:

1. 需要 pairing 压缩验证关系
2. 需要一个现代、足够安全且生态上可维护的 pairing-friendly curve

## 一张职责图: 按系统边界选曲线，而不是按“先进程度”选曲线

把上面几节压缩成一张图，会比任何口号都更有用。

This is the mapping from curve family to system responsibility in Web3.

| 曲线/曲线族 | 关键代数接口 | 典型协议职责 | 链上可用性语境 | 主要约束 |
| --- | --- | --- | --- | --- |
| secp256k1 | 普通离散对数群 | 账户签名、交易签名 | Bitcoin / Ethereum 账户层 | 实现成熟度、签名接口、侧信道防护 |
| BN254 / alt_bn128 | pairing-friendly | 历史链上 zkSNARK verifier | Ethereum 执行层已有 precompile | 预编译接口、gas 成本、安全边际、历史包袱 |
| BLS12-381 | pairing-friendly | BLS 聚合签名、KZG、现代 DA/共识基础设施 | 共识层与新一代协议栈 | pairing 需求、库生态、预编译缺口、协议集成 |

这张图后面有三条判断线。

### 第一条: 你到底需不需要 pairing

如果系统验证目标只是账户签名或普通公钥关系，通常不需要 pairing。

如果系统要验证的是:

- 聚合签名
- pairing product equation
- KZG opening proof
- 某些 SNARK verifier relation

那 pairing-friendly curves 就不再是可选装饰，而是协议接口的一部分。

### 第二条: 链上可用性往往先于数学优雅

在 Ethereum 语境里，“有没有 precompile”常常比“哪条曲线更漂亮”更决定现实。

BN254/alt_bn128 并不是后见之明下最优雅的答案，但它拥有 precompile 这一点足以长期塑造系统实现。反过来，BLS12-381 即便在现代协议中非常关键，也会受到执行层可用接口的制约。

### 第三条: trusted setup 不是曲线属性，而是协议属性

Web3 讨论里经常把 “BLS12-381 / pairing-friendly curve / trusted setup” 混在一起说，这是不准确的。

需要 trusted setup 的是 KZG、Groth16 之类具体协议构造，不是“因为这条曲线天然有 setup 问题”。更直接地说，trusted setup is a protocol-level constraint rather than an intrinsic ECC property. 曲线提供的是代数舞台，协议决定是否引入 structured reference string、toxic waste 与 setup ceremony。

这一点必须说清楚，否则后面读 KZG 和 Groth16 时会把协议层约束误认为曲线层缺陷。

## 工程实现对接

如果只停在“职责图”，这篇文章还不够像工程文章。真正落地时，至少要把下面几件事分开。

### 1. 账户层实现

如果你做的是账户、钱包、交易签名或类似接口，优先考虑的是:

- secp256k1 相关成熟实现与审计历史
- ECDSA 与 Schnorr 的协议边界
- nonce 生成、常数时间、硬件实现与侧信道

这部分对应的工程世界，关键词是 `libsecp256k1`、RFC 6979、BIP-340、hardware wallet signing pipeline，而不是 pairing library。

### 2. 链上 verifier 设计

如果你做的是执行层合约中的 zk verifier，先问的不是“哪条曲线最先进”，而是:

- 目标链是否已有对应 precompile
- verifier 代价是否在 gas 预算内
- 现成 proving system 是否围绕该曲线与接口构建

这也是为什么 Ethereum 上历史 Groth16 verifier 长期与 alt_bn128 绑定。

### 3. 聚合签名与数据可用性系统

如果你做的是共识聚合签名、KZG、blob commitments 或 DA 相关验证，则需要正面进入 BLS12-381 生态:

- BLS signature libraries
- KZG setup and verification libraries
- consensus/client integration
- precompile 或 native implementation 边界

这里的风险不只是密码学正确性，还有协议集成、参数一致性、trusted setup 管理与跨实现兼容性。

### 4. 审计时该检查什么

不同语境下，审计关注点完全不同。

- 账户签名: nonce、side-channel、签名 API 复用、输入规范化
- 链上 verifier: precompile 调用、输入编码、gas 假设、边界条件
- KZG / BLS: 参数一致性、SRS 来源、聚合验证前提、域分离和消息绑定

如果工程团队把这些都统称为“ECC 模块审计”，通常说明系统边界还没有划清楚。

### 5. 本系列接下来怎么读

这一篇只负责画地图。后面的 5 篇分别补足地图上的关键地块:

1. `secp256k1 的协议定位与签名机制`
2. `椭圆曲线签名的安全性分析：从 ECDLP 到 HNP、EC-HNP 与 EHNP`
3. `配对友好曲线的数学结构与协议动机`
4. `Ethereum 中配对友好曲线的演化路径：从 BN254 到 BLS12-381`
5. `KZG 多项式承诺的曲线基础及其协议应用`

这也是本文刻意不展开 pairing 细节、Groth16 推导与 KZG 全证明链的原因。地图篇的任务不是替代后文，而是先把“为什么不是一条曲线包打天下”说清楚。

## 总结

Web3 里的 ECC 不是一套统一技术栈，而是一组由不同系统职责拼出来的曲线分工图。

- secp256k1 服务普通账户签名接口
- BN254/alt_bn128 代表历史链上 pairing verifier 的现实入口
- BLS12-381 服务聚合签名与 KZG 一类现代基础设施协议

所以曲线选择从来不是“哪条曲线更先进”的抽象竞赛，而是“这个系统到底消费什么代数接口、执行环境给了什么能力、现有生态能否把它安全地实现出来”的综合决策。

下一篇开始，我们先把账户层拉直: 同样是 secp256k1，ECDSA 与 Schnorr 到底改变了什么。

[^sec2]: SEC Group, *SEC 2: Recommended Elliptic Curve Domain Parameters*, Version 2.0. https://www.secg.org/sec2-v2.pdf
[^bip340]: Pieter Wuille et al., *BIP-340: Schnorr Signatures for secp256k1*. https://bips.xyz/0340
[^libsecp]: `bitcoin-core/secp256k1` repository. https://github.com/bitcoin-core/secp256k1
[^eip196]: *EIP-196: Precompiled contracts for addition and scalar multiplication on the elliptic curve alt_bn128*. https://eips.ethereum.org/EIPS/eip-196
[^eip197]: *EIP-197: Precompiled contracts for optimal ate pairing check on the elliptic curve alt_bn128*. https://eips.ethereum.org/EIPS/eip-197
[^eip1108]: *EIP-1108: Reduce alt_bn128 precompile gas costs*. https://eips.ethereum.org/EIPS/eip-1108
[^bls]: Dan Boneh, Ben Lynn, Hovav Shacham, *Short Signatures from the Weil Pairing*. https://hovav.net/ucsd/papers/bls04.html
[^eip2537]: *EIP-2537: Precompile for BLS12-381 curve operations*. https://eips.ethereum.org/EIPS/eip-2537
[^kzg]: Aniket Kate, Gregory Zaverucha, Ian Goldberg, *Constant-Size Commitments to Polynomials and Their Applications*. https://www.iacr.org/archive/asiacrypt2010/6477178/6477178.pdf
[^eip4844]: *EIP-4844: Shard Blob Transactions*. https://eips.ethereum.org/EIPS/eip-4844

