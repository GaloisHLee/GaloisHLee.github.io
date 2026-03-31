# 椭圆曲线签名的安全性分析：从 ECDLP 到 HNP、EC-HNP 与 EHNP


> Reading: ECDLP is the baseline, not the whole risk surface.

前两篇已经把 ECC 子系列里最容易混淆的两件事拆开了: 哪些协议只消费普通离散对数群，哪些协议需要 pairing；以及为什么账户层长期绑定 secp256k1。这一篇继续沿着账户层往下走，但焦点不再是“签名接口长什么样”，而是“签名接口在哪里失守”。

对椭圆曲线签名来说，曲线层首先给出的安全基线是 `ECDLP security assumption`。如果敌手只能看到公钥 $P=dG$，却拿不到关于私钥 $d$ 和临时标量 $k$ 的额外泄漏，那么攻击目标仍然是离散对数问题本身。但真实系统并不总是在这个理想模型里运行。wallet and signing implementations 会暴露 nonce 生成、缓存访问、功耗、分支、fault handling 和接口复用等边界，而这些边界往往把问题从“解曲线上的离散对数”改写成“恢复带泄漏的 secret scalar”。[^howgrave] [^ryan]

ECDSA 的危险点尤其集中在 nonce。只要签名方程里的 nonce 有重复、偏置、部分泄漏，或者可由 side-channel trace 提供不完整信息，攻击者面对的就不再是 black-box ECDLP，而是某种 hidden-number style relation。也正是因此，HNP / EC-HNP / EHNP 不是只存在于格论教科书里的标签，而是现实账户系统在弱随机数、硬件钱包、远程 signer 和实现缺陷下会真正碰到的攻击视角。[^biasednonce] [^hlavac]

> Quick Note.
> This article connects ECDSA nonce leakage to hidden-number style linear constraints. It also distinguishes HNP, EC-HNP, and EHNP rather than flattening them into a generic lattice-attack label, covers nonce bias, partial leakage, and side-channel trace models relevant to wallet and signing implementations, and separates curve-level security from implementation leakage and protocol misuse.

<!--more-->

## Threat Model：曲线层安全不是签名实现的全部安全

### `ECDLP security assumption` 给出的基线

把基线写清很重要。对阶为 $n$ 的群，给定生成元 $G$ 与公钥

$$
P = dG,
$$

如果敌手只能把公钥当作黑盒群元素来观察，那么它需要恢复私钥 $d$。这就是 ECDLP 的基本形状。曲线参数选择、群阶、实现是否避免小子群或 invalid-curve 问题，这些都属于 curve-level security。

但对 ECDSA 系统来说，仅有这条基线远远不够。真实敌手常常不是只看见 $P$，而是还能收集大量签名 $(r_i, s_i)$、消息哈希 $z_i$、签名时间、功耗轨迹、窗口访存模式或故障注入后的异常执行结果。此时问题已经不是“能否直接求解 $d$ 的离散对数”，而是“能否把带泄漏的 nonce 约束整理成可恢复 $d$ 的线性关系”。

### 实现泄漏与协议误用如何改变问题类型

这里至少要分三层：

- 曲线层：ECDLP、群阶、cofactor、点验证、invalid-curve / twist 安全
- 实现层：nonce 生成、partial nonce leakage、wNAF trace、cache timing、power trace、fault injection
- 协议层：签名接口复用、跨域消息处理、错误的 deterministic nonce 上下文、重复签名状态

这三层经常被混着说，结果是攻击报告一旦写到“格攻击恢复 ECDSA 私钥”，读者会误以为是曲线本身失守。更准确的说法是：many practical breaks come from implementation leakage and protocol misuse, not from a direct failure of the curve-level ECDLP assumption.

## 从 ECDSA signing equation 到 hidden-number style relation

### 最小 ECDSA signing equation

设私钥为 $d$，公钥为 $P=dG$。对消息哈希 $z$，签名者取 nonce $k$，计算

$$
R = kG, \qquad r = x(R) \bmod n,
$$

再定义

$$
s = k^{-1}(z + rd) \bmod n.
$$

把这个式子乘回去，可得

$$
sk \equiv z + rd \pmod n.
$$

或者写成

$$
rd - sk + z \equiv 0 \pmod n.
$$

这一步已经暴露了 ECDSA 的脆弱面：nonce $k$ 和私钥 $d$ 在同一个模方程里线性耦合。

### 把 nonce 泄漏改写成线性约束

This section derives the hidden-number style relation from the ECDSA signing equation.

如果攻击者完全知道某次签名的 $k$，那么

$$
d \equiv (sk - z) r^{-1} \pmod n,
$$

私钥会被立即恢复。真实攻击更常见的不是完整泄漏，而是 `nonce bias and partial nonce leakage`。于是可以把 nonce 写成

$$
k_i = \hat{k}_i + \delta_i,
$$

其中 $\hat{k}_i$ 是攻击者已知或可猜测的部分，$\delta_i$ 是较小的未知误差项。代回签名方程得

$$
s_i(\hat{k}_i + \delta_i) \equiv z_i + r_i d \pmod n,
$$

整理为

$$
r_i d - s_i \delta_i \equiv s_i \hat{k}_i - z_i \pmod n.
$$

一旦 $\delta_i$ 的范围足够小，或者样本足够多，这组关系就会变成典型的 hidden-number style linear constraints。攻击者不再直接求解离散对数，而是利用部分已知的 nonce 结构，把未知量压进格约化或最近向量类问题。

这个转化解释了为什么 ECDSA 对 nonce 异常敏感：签名等式本身就把 secret key 和 ephemeral nonce 放进了同一条可线性化的约束里。

## HNP、EC-HNP 与 EHNP 的区别

### HNP：带近似信息的 hidden number

HNP 的抽象轮廓是：有一个隐藏标量 $\alpha$，攻击者观察到若干形如

$$
t_i \alpha + b_i \approx a_i
$$

的约束，其中“近似”来自高位、低位、区间或某种可控误差。HNP 本身不要求这些量来自椭圆曲线签名；它只是说，若同一个 hidden number 在多条近似线性关系里反复出现，就可能被恢复。

### EC-HNP：把约束放回椭圆曲线签名接口

EC-HNP 的意义，是把 HNP 的抽象关系落回 ECDSA 或相关 elliptic-curve 签名接口。此时 hidden number 不再是抽象变量，而是和签名方程直接耦合的私钥或 nonce 相关量。攻击者观测到的是：

- 签名样本 $(r_i, s_i)$
- 消息哈希 $z_i$
- 某种关于 nonce 的偏置信息或部分位信息

于是 HNP 的“近似线性关系”不再是外部给定，而是从 ECDSA signing equation 中系统地产生出来。In this sense, EC-HNP is the elliptic-curve instantiation of the hidden-number viewpoint.

### EHNP：泄漏模型变丰富时会发生什么

EHNP 常用来表示 extended hidden number problem 视角，也就是泄漏模型不再只是“某些高位或低位已知”，而是允许更复杂的不规则信息，例如：

- windowed multiplication 泄漏出的局部窗口信息
- `wNAF trace` 暗示的 signed-digit 模式
- 带噪声的 side-channel 分类结果
- 非连续 bit positions 的偏置信息

This article shows how leakage assumptions change the attack model from HNP to EC-HNP and EHNP.

从写作上更重要的一点是，不要把 HNP、EC-HNP 与 EHNP 压成一个宽泛的“lattice attack”标签。差异不只在名词，而在攻击者能看到什么、误差边界有多小、样本需求有多少，以及恢复步骤如何建模。

## 把攻击前提映射到真实的 Web3 签名环境

### nonce bias 与弱随机数

最经典的一类问题是 nonce 分布不均匀。假设 signer 的随机数生成器有偏置，或者某些高位在统计上更容易落到固定模式，那么攻击者就可能从链上或日志里收集大批签名样本，进而构造 HNP 类约束。Biased Nonce Sense 之所以有杀伤力，正因为这类攻击不要求侧信道探针，只要求大规模可观测签名样本。[^biasednonce]

这类风险更像运维和实现事故：

- 弱 RNG
- 随机源复用
- 虚拟化环境里熵不足
- 多租户 signer 共享状态

### `partial nonce leakage` 与 side-channel trace

另一类攻击更贴近硬件钱包、HSM 和远程 signer。此时攻击者未必能完整读出 nonce，但可以从 cache timing、power analysis、EM trace 或分支行为中得到部分信息。只要这些信息足以把 $k_i$ 压成“已知部分 + 小误差项”的形式，就会自然进入 EC-HNP 或 EHNP。

典型泄漏接口包括：

- 标量乘中的窗口访存模式
- 非 constant-time 条件分支
- `wNAF trace` 暴露的 signed-digit 模式
- 异常处理路径泄漏出的重试或归约信息

Return of the Hidden Number Problem 这一脉络的重要性就在于，它把这些 seemingly low-level side-channel observations 和高层私钥恢复直接连了起来。[^ryan]

### fault injection、重放与接口误用

fault injection 不一定总会落到 HNP 类问题，但它经常会扩大泄漏面。例如：

- 让某些乘法或模约减步骤跳过
- 让 deterministic nonce 的输入上下文不完整
- 诱导 signer 重复使用部分内部状态

这些情况有些更像协议误用，有些更像实现缺陷。区分这两类很重要，因为“如何修补”完全不同。修曲线参数解决不了 deterministic nonce 上下文拼接错误；加更多样本也不一定能修复一个会在故障下注入重复状态的签名服务。

This article maps attack preconditions to realistic Web3 signing environments.

在 Web3 语境里，可以把这些环境粗分为三类：

- 本地钱包：风险集中在设备侧信道、浏览器扩展接口和签名请求隔离
- 硬件钱包 / HSM：风险集中在物理侧信道、fault model 和标量乘实现
- 后端 signer / 托管服务：风险集中在随机数、并发、状态复用、日志和 API 域分离

## 曲线层安全、实现泄漏与协议误用要分开看

把这三层放在一张表里，会比口号更清楚。

| 层级 | 核心问题 | 典型失败模式 | 解决路径 |
| --- | --- | --- | --- |
| 曲线层 | ECDLP 是否足够强、点验证是否完备 | 小子群、invalid-curve、参数选择错误 | 选对曲线与群参数，做输入点验证 |
| 实现层 | nonce 和标量乘是否泄漏 | nonce bias、partial nonce leakage、cache/power/EM leakage、`wNAF trace` | constant-time、signer isolation、侧信道加固 |
| 协议层 | 接口是否在正确上下文中使用 | 域分离错误、deterministic nonce 上下文缺失、重放与状态复用 | API 边界、域分离、状态管理与审计 |

This section separates curve-level security from implementation leakage and protocol misuse.

很多争论卡住，是因为有人把所有问题都归为“曲线不安全”，也有人把所有问题都归为“实现写烂了”。更准确的看法是：curve-level security defines the algebraic baseline, while implementation leakage and protocol misuse define the practical attack surface.

## 工程实现对接

### RFC6979 与 deterministic nonce

RFC6979 的价值，在于把 ECDSA nonce 生成从“依赖外部随机源每次都不出错”收紧到一个 deterministic procedure。它显著降低了弱随机数和重复 nonce 的风险，但不意味着攻击面彻底消失。

RFC6979 解决的是：

- 低质量随机数导致的 nonce bias
- 签名服务偶发重复 nonce
- 由熵源质量波动带来的灾难性故障

它不直接解决的是：

- side-channel 泄漏出的 partial nonce 信息
- fault injection 打断正常执行
- 跨协议或跨上下文错误复用签名输入

### constant-time、`wNAF` 与 signer isolation

如果实现仍在标量乘、窗口选择、条件分支和异常路径上泄漏行为模式，那么 deterministic nonce 依然可能被 EHNP 类模型利用。工程上更实在的要求是：

- 标量乘和模运算保持 constant-time
- 尽量避免将 `wNAF`、窗口表访问和异常分支暴露成可测 side-channel
- 将 signer 放在最小可见接口后面，减少外部对签名过程的观测面
- 对硬件设备单独考虑 power / EM / fault model，而不是套用软件系统假设

### 审计时该检查什么

如果把这篇压成一份 audit checklist，我会优先看下面几项：

- nonce 生成是否严格符合 RFC6979 或对应规范
- 是否存在重复状态、并发复用或错误缓存导致的 nonce correlation
- 标量乘实现是否 constant-time，窗口访存是否可被外部观测
- 是否存在可形成 `partial nonce leakage` 的日志、debug、telemetry 或错误处理路径
- wallet and signing implementations 是否把消息域、链 ID、账户上下文与 deterministic nonce 输入正确绑定
- 是否有硬件钱包、HSM、远程 signer 各自独立的 threat model 与 side-channel 假设

真正值得警惕的不是某篇论文里的格参数有多花，而是系统是否在持续制造“同一个 secret scalar 出现在许多条近似线性关系里”的条件。

## 总结

ECDLP 仍然是椭圆曲线签名安全性的基线，但它不是现实风险面的全部。对 ECDSA 而言，一旦 nonce 偏置、部分泄漏或 side-channel 进入系统，攻击者面对的就不再是纯粹的离散对数，而是 hidden-number style relation。

所以第 3 篇真正想固定下来的判断框架只有一句：先问曲线层有没有给出足够强的代数基线，再问 signer 实现是否持续泄漏了把基线绕开的信息。前一个问题决定“理论上难不难解”，后一个问题决定“现实里会不会被攻破”。

下一篇进入 pairing-friendly curves 时，这个框架仍然有效。曲线类型会改变协议接口，但不会取消一个事实：实际安全性永远是 algebraic assumption、implementation boundary 和 protocol discipline 的交集。

[^howgrave]: Nick Howgrave-Graham and Nigel Smart, *Lattice Attacks on Digital Signature Schemes*.
[^hlavac]: Miroslav Hlavac and Tomas Rosa, *Extended Hidden Number Problem and Its Cryptanalytic Applications*.
[^ryan]: Keegan Ryan, *Return of the Hidden Number Problem*.
[^biasednonce]: Joachim Breitner and Nadia Heninger, *Biased Nonce Sense*.

