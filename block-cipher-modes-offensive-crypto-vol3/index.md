# 分组密码工作模式攻击讲义 Part 3：Nonce Misuse Resistance 与 Synthetic IV


> Note: Vol.2 已经把 AEAD 的认证方程和 nonce-based 边界讲清了。Vol.3 的问题不是再追加两个模式名词，而是继续追问：如果 “nonce never repeats” 这条前提自己坏了，安全定义应该怎样升级，失败模式又该怎样被重新组织。

SIV 与 GCM-SIV 的价值，不在于它们让 misuse 没有代价，而在于它们改变了失败的几何形状。对 GCM 而言，repeated nonce 会直接复用 keystream 和 tag mask，把攻击者送进关于 \(H\) 的代数恢复问题。对 SIV / GCM-SIV 而言，外部 nonce 不再直接选中 keystream，而是先进入一个 synthetic IV 的计算，于是 repeated nonce 不再自动等于 two-time pad。

这一卷真正要区分的，不是“谁更先进”，而是四种完全不同的退化：keystream reuse、tag collapse、equality leakage、deterministic repetition。把这些退化混成一句“nonce misuse 很危险”，反而会掩掉攻击者到底得到了什么。

<!--more-->

## 0. Threat Model 与目标切换

本卷默认读者已经接受 Vol.2 的接口：

$$
\mathcal{E}_K(N, A, P) = (C, T), \qquad \mathcal{D}_K(N, A, C, T)\in\{P,\bot\}.
$$

Vol.2 已经说明，这样的 `nonce-based AEAD` 在 nonce 唯一时可以同时提供：

- confidentiality
- ciphertext integrity
- associated-data binding

但这一类安全定理把一个条件埋得很深：

$$
N_i \neq N_j \qquad \text{for all encryption queries under one key.}
$$

这不是一个实现小贴士，而是定义的一部分。Vol.3 的目标就是考察当该条件被破坏时，安全性如何退化。

这里先固定三种不同的失败形态：

1. `catastrophic break`
   攻击者一旦得到 repeated nonce transcript，就能直接把保密性或认证性改写成显式可利用的代数对象。
2. `bounded degradation`
   重复 nonce 仍然导致泄漏，但泄漏被约束成某种可描述对象，例如等值关系。
3. `deterministic repetition`
   对相同输入得到相同输出，这是一种设计允许的泄漏，而不是 misuse 才触发的崩塌。

把这三者分开，是理解 SIV / GCM-SIV 的前提。

## 1. 从 Vol.2 到 Vol.3：nonce-based AEAD 的边界在哪里结束

Vol.2 中最硬的一章其实是 GCM。因为 repeated nonce 不只是让

$$
C \oplus C' = P \oplus P'
$$

成立，而且还让认证项退化成关于

$$
H = E_K(0^{128})
$$

的多项式根问题。这就是典型的 `catastrophic break`：攻击者不需要恢复 AES key，只需要恢复足以伪造 tag 的中间对象。

如果把这一点抽象掉，Vol.2 真正留下的问题其实是：

> 有没有一种构造，使外部 nonce 的重复不再直接等于“同一明文异或流”和“同一 tag mask 消元”？

SIV 与 GCM-SIV 的回答都是否定了“外部 nonce 直接驱动 keystream”这个做法。它们先计算一个依赖消息与上下文的 synthetic IV，再用该 IV 驱动流层。

这一步的含义非常大。它不是微调一个 counter format，而是把安全边界从

$$
N \mapsto \text{keystream selector}
$$

改成

$$
(N, A, P) \mapsto S \mapsto \text{keystream selector}.
$$

于是攻击者若只重复外部 nonce，而不能让 \((A,P)\) 也重复，就不再自动复用内部状态。

## 2. 从 AEAD 到 DAE：安全定义为什么必须升级

### 2.1 标准 AEAD 的接口依赖 unique nonce

标准 AEAD 的对象是带 nonce 的随机化加密：

$$
\mathcal{E}_K(N,A,P) = (C,T).
$$

在这个接口里，攻击者通常把 \(N\) 当作：

- public value
- unique value
- not necessarily unpredictable value

也就是说，标准 AEAD 接受 `nonce can be public`，但不接受 `nonce can repeat`。

因此标准 AEAD 的安全断言本质上是一个条件句：

$$
\text{if nonce discipline holds, then confidentiality and integrity hold.}
$$

### 2.2 deterministic authenticated encryption 的接口

若完全移除外部 nonce，一个自然对象是 deterministic authenticated encryption。它的接口更接近：

$$
\mathcal{E}_K(A,P) = Z, \qquad \mathcal{D}_K(A,Z)\in\{P,\bot\}.
$$

这里没有随机化输入，所以对同一 \((A,P)\)：

$$
\mathcal{E}_K(A,P)=\mathcal{E}_K(A,P).
$$

这句看起来像废话，但它意味着一个 unavoidable leakage：

$$
(A,P)=(A',P') \Longrightarrow Z=Z'.
$$

也就是说，deterministic construction 必然泄漏等值关系。攻击者无法从 deterministic security 中被承诺“相同输入得到不同输出”。

因此 deterministic AE 从来就不是“完美隐藏所有结构”，而是：

> 在承认 equality leakage 的前提下，继续保持认证与对不等输入的高层隐藏性。

### 2.3 misuse-resistant AE 的目标

misuse-resistant AE 想实现的是一种中间态：接口看起来仍可接受外部 nonce，但 repeated nonce 不应立即造成 GCM 式灾难。

它真正追求的是：

- 若 \((N,A,P)\) 与 \((N',A',P')\) 完全相同，则允许泄漏相等性。
- 若只重复 nonce，但 \(A\) 或 \(P\) 不同，则不应直接复用 keystream。
- 对新鲜 tuple 的伪造能力仍应被压到可忽略范围。

因此 Vol.3 的核心比较不是“随机 AE vs deterministic AE”，而是：

| 退化类型 | 攻击者得到什么 |
| --- | --- |
| GCM repeated nonce | \(P \oplus P'\) 以及 tag-related algebraic collapse |
| deterministic repetition | 只知道 two transcripts 是否完全相等 |
| misuse-resistant AE under repeated nonce | 目标是尽量只退化到上面第二种 |

### 2.4 等值泄漏和 two-time pad 泄漏不是一个量级

这点必须单独强调。若攻击者只得到

$$
\mathbf{1}\big[(A,P)=(A',P')\big],
$$

它知道的是一位 equality information。若攻击者得到

$$
P \oplus P',
$$

它得到的是整条消息差分结构。后者能被已知明文、格式约束、低熵字段、自然语言冗余或 parser 结构继续放大，前者则不能直接还原线性差分。

因此把 SIV 的 equality leakage 和 GCM 的 two-time pad 泄漏放在同一句“都会泄漏信息”里，是严重淡化了失败程度差异。

## 3. SIV：synthetic IV 如何改变失败模式

### 3.1 SIV 的最小结构

SIV 的核心接口可以先抽象写成：

$$
S = \operatorname{PRF}_{K_1}(A_1,\ldots,A_d,P),
$$

$$
C = P \oplus \mathsf{KS}_{K_2}(S),
$$

最终输出为

$$
Z = S \,\|\, C.
$$

这里 \(A_1,\ldots,A_d\) 是一组 associated data components，plaintext \(P\) 作为最后一个输入进入 PRF。RFC 5297 中的具体 PRF 构造叫做 `S2V`，使用 CMAC 和 doubling 链实现。

这条方程已经体现了与 Vol.2 的根本差异：

- GCM 中，外部 nonce 直接决定 \(J_0\)
- SIV 中，真正驱动 CTR 流的是 synthetic IV \(S\)

于是 repeated external nonce 不再自动意味着 repeated internal counter origin。

### 3.2 S2V：为什么 synthetic IV 是一个“消息摘要式”的状态，而不是随机 nonce

RFC 5297 的 S2V 可以抽象成下面的链。设

$$
D_0 = \operatorname{CMAC}_{K_1}(0^{128}).
$$

对前 \(d\) 个 associated-data components 递推：

$$
D_i = \operatorname{dbl}(D_{i-1}) \oplus \operatorname{CMAC}_{K_1}(A_i).
$$

然后把 plaintext 作为最后一个输入。如果 \(|P| \ge 128\)，则做尾部异或：

$$
S = \operatorname{CMAC}_{K_1}\big(\operatorname{xorend}(P, D_d)\big).
$$

若 \(|P| < 128\)，则做一次额外 doubling 再与 padding 后的 \(P\) 异或：

$$
S = \operatorname{CMAC}_{K_1}\big(\operatorname{dbl}(D_d) \oplus \operatorname{pad}(P)\big).
$$

所以 \(S\) 不是“外部给的一个 number used once”，而是：

> 把所有需要认证的对象一起压缩出来的 synthetic state。

这也意味着，只要 \(A\) 或 \(P\) 改变，\(S\) 通常就会改变。

### 3.3 nonce-based SIV 与 deterministic SIV

SIV 同时支持两种视角：

1. deterministic AE
   把 associated data vector 写成 \((A_1,\ldots,A_d)\)，不额外给 nonce。
2. nonce-based misuse-resistant AE
   把外部 nonce \(N\) 也作为一个 associated-data component 放进向量。

后一种做法可写成：

$$
S = \operatorname{PRF}_{K_1}(N, A, P).
$$

这一步至关重要。外部 nonce 在 SIV 里不是直接用来选中 keystream，而是被吸收到 PRF 输入里。因此：

$$
N=N' \not\Rightarrow S=S'.
$$

只有当

$$
(N,A,P)=(N',A',P')
$$

时，才会必然有

$$
S=S'.
$$

### 3.4 为什么 repeated external nonce 不等于 repeated keystream

比较一下 GCM 与 SIV：

GCM 中，

$$
N \Longrightarrow J_0 \Longrightarrow \text{counter stream}.
$$

SIV 中，

$$
(N,A,P) \Longrightarrow S \Longrightarrow \text{counter stream}.
$$

因此 repeated nonce 但不同消息时：

$$
N=N', \quad (A,P)\neq(A',P')
$$

在 GCM 下会直接重用 keystream；而在 SIV 下通常导致

$$
S \neq S',
$$

所以不会自动出现

$$
C \oplus C' = P \oplus P'.
$$

这正是 SIV 把 repeated nonce 的灾难性失败降级掉的机制。

### 3.5 SIV 仍然会泄漏什么

SIV 并不是零代价方案。它会显式泄漏：

$$
(N,A,P)=(N',A',P') \Longrightarrow Z=Z'.
$$

也就是说，攻击者知道两次输入是否完全相同。这就是 equality leakage。

从 deterministic design 角度看，这个泄漏几乎是不可避免的：没有外部随机性，完全相同的输入只能给出完全相同的输出，除非额外引入状态或随机源，而那又会回到 standard AEAD 的世界。

因此 SIV 的代价可以概括成两条：

- `equality leakage`
- `two-pass processing`

第二条是工程代价，第一条是安全代价。本卷只关心第一条。

### 3.6 SIV 没有解决什么

把外部 nonce 变成 associated-data input，并不意味着一切攻击面都消失。SIV 仍然没有自动解决：

- replay
- application-level state confusion
- side-channel visible \(\bot\) 分支
- message equality as a privacy signal

因此更准确的说法不是 “SIV is safe under nonce reuse”，而是：

> repeated nonce 不再把系统推入 two-time pad 或 tag-collapse，而只把它推入 deterministic equality leakage 的边界。

## 4. GCM-SIV：保留 GCM 轮廓下的 bounded degradation

### 4.1 目标不是“修补 GCM 一个 corner case”

GCM-SIV 的定位非常精确。它不是在 GCM 外面补一个告警，也不是简单把 GCM 的 nonce 换个格式。它要做到三件事：

1. 尽量保留 GCM 类实现与硬件优势。
2. 避免 repeated nonce 直接复用 keystream。
3. 保持输出仍然是 AEAD 风格，而不是回到纯 deterministic wrapping。

RFC 8452 对这一点说得很清楚：其目标是在 nonce 重复时“不发生 catastrophic failure”，而退化到是否相等这一最小信息。

### 4.2 GCM-SIV 的最小方程组

GCM-SIV encryption 接受：

- key-generating key \(K\)
- 96-bit nonce \(N\)
- associated data \(A\)
- plaintext \(P\)

它先用 \(K\) 与 \(N\) 派生一组 per-nonce keys：

$$
(K_{\mathrm{auth}}, K_{\mathrm{enc}}) = \operatorname{Derive}_K(N).
$$

然后定义长度块

$$
L = [|A|]_{64}^{\mathrm{le}} \,\|\, [|P|]_{64}^{\mathrm{le}},
$$

并对 padded associated data 与 padded plaintext 做 POLYVAL：

$$
S_s = POLYVAL_{K_{\mathrm{auth}}}(A^\star \,\|\, P^\star \,\|\, L).
$$

接着把 nonce 混入 \(S_s\)，并清掉最后一个字节的最高位，得到一个 pre-tag state：

$$
S' = \operatorname{mix}(S_s, N).
$$

最后用 AES 在 \(K_{\mathrm{enc}}\) 下加密 \(S'\) 得到 tag：

$$
T = E_{K_{\mathrm{enc}}}(S').
$$

再把 tag 改成 counter origin，对 plaintext 做 CTR：

$$
C = P \oplus \mathsf{KS}_{K_{\mathrm{enc}}}(\operatorname{ctr}(T)).
$$

输出为

$$
Z = C \,\|\, T.
$$

这条结构和 GCM 的最重要差异是：

- GCM 认证的是 \(A\) 与 ciphertext \(C\)
- GCM-SIV 认证的是 \(A\) 与 plaintext \(P\)

也就是说，GCM-SIV 先通过认证层生成 synthetic IV/tag，再让该 tag 反过来驱动加密层。

### 4.3 POLYVAL 与 GHASH：相似，但角色被重新安排了

POLYVAL 与 GHASH 都是 \(GF(2^{128})\) 上的多项式认证器。RFC 8452 甚至明确说明二者可通过 byte order 和域元素变换联系起来。

但在安全边界上，真正关键的不是 “POLYVAL 和 GHASH 都是 polynomial hash”，而是：

- GCM 中，counter origin 先由 nonce 确定，然后再去认证 ciphertext
- GCM-SIV 中，tag 先由 \((N,A,P)\) 的认证计算确定，然后再去驱动 CTR

因此 repeated nonce 时，即使 per-nonce keys 会重复，攻击者也不会仅凭 nonce 相等就得到相同 counter origin。还需要 \((A,P)\) 的认证结果一起相等。

### 4.4 为什么 repeated nonce 不再直接重用 keystream

GCM-SIV 中的 keystream origin 本质上是

$$
\operatorname{ctr}(T),
$$

而 \(T\) 本身来自

$$
T = E_{K_{\mathrm{enc}}}(\operatorname{mix}(POLYVAL_{K_{\mathrm{auth}}}(A^\star \,\|\, P^\star \,\|\, L), N)).
$$

所以 repeated nonce 只给出：

$$
N=N' \Longrightarrow (K_{\mathrm{auth}},K_{\mathrm{enc}})=(K'_{\mathrm{auth}},K'_{\mathrm{enc}}).
$$

但这并不推出

$$
T=T'.
$$

因为还需要

$$
POLYVAL_{K_{\mathrm{auth}}}(A^\star \,\|\, P^\star \,\|\, L)
= POLYVAL_{K_{\mathrm{auth}}}(A'^\star \,\|\, P'^\star \,\|\, L').
$$

在正常安全边界下，这通常要求 \((A,P)\) 本身也相同或发生极罕见碰撞。因此 GCM-SIV 避免了 GCM 中那种

$$
N=N' \Longrightarrow \text{same keystream}
$$

的直接灾难。

### 4.5 GCM-SIV 仍然泄漏什么

RFC 8452 对 repeated nonce 的目标非常克制：若同一 nonce 下两条消息重复加密，泄漏的应该主要是它们是否相等。

形式化地说，若

$$
(N,A,P)=(N',A',P'),
$$

则必然有

$$
Z=Z'.
$$

因此 GCM-SIV 的退化边界并不是“啥也不泄漏”，而是：

$$
\text{same input} \Longrightarrow \text{same output}.
$$

这和 SIV 一样，是 deterministic component 不可避免的痕迹；不同的是 GCM-SIV 仍然保留显式的外部 nonce 接口，并尽量贴近 GCM 的性能轮廓。

### 4.6 为什么未认证明文绝不能提前释放

RFC 8452 对解密流程有一句非常硬的要求：plaintext 在 tag confirmation 完成前 `MUST NOT be output`。

这不是实现洁癖，而是理论边界的组成部分。原因在于 GCM-SIV 的 decrypt 先会基于 tag 驱动 CTR 还原一个候选明文：

$$
\widetilde{P} = C \oplus \mathsf{KS}_{K_{\mathrm{enc}}}(\operatorname{ctr}(T)).
$$

然后再检查该 \(\widetilde{P}\) 与 \(A\) 是否重新生成相同 tag。若系统在这一步确认前就把 \(\widetilde{P}\) 暴露给 parser、日志或上层应用，那么攻击者又重新获得了一个“先解密、后认证”的 side channel 接口。

因此 GCM-SIV 的安全性不只依赖 synthetic IV 结构，也依赖 decrypt semantics 真正保持：

$$
\mathcal{D}_K(N,A,C,T)\in\{P,\bot\},
$$

而不是

$$
\widetilde{P} \to \text{parse} \to \text{later decide whether to reject}.
$$

## 5. 统一比较：GCM、SIV、GCM-SIV 在 misuse 下分别退化成什么

现在把三者放到同一个框架里。

| 构造 | repeated nonce / repeated input 后首先复用的对象 | 直接退化 |
| --- | --- | --- |
| GCM | keystream 与 tag mask | \(P \oplus P'\) 与认证崩塌 |
| SIV | synthetic IV 仅在完整输入相等时复用 | equality leakage |
| GCM-SIV | per-nonce keys 可重复，但 counter origin 仍受 \((A,P)\) 约束 | bounded equality-style degradation |

换一种问法：攻击者究竟能推出什么？

### 5.1 GCM

若 nonce 重复，攻击者很快得到：

$$
C \oplus C' = P \oplus P'
$$

并进一步把 tag 关系压成关于 hash subkey 的多项式约束。它得到的是整条线性差分和认证代数入口。

### 5.2 SIV

攻击者看到的不是 \(P \oplus P'\)，而是：

$$
Z=Z' \Longleftrightarrow (A,P)=(A',P')
$$

或在 nonce-based 用法下

$$
Z=Z' \Longleftrightarrow (N,A,P)=(N',A',P').
$$

这是一种 equality oracle，而不是 linear-difference oracle。

### 5.3 GCM-SIV

GCM-SIV 的目标与 SIV 类似：repeated nonce 时，攻击者至多获得近似 equality-style 的退化，而不是 GCM 式 catastrophe。与 SIV 相比，它额外保留了显式 nonce 接口和 GCM-like performance profile；与 GCM 相比，它牺牲的是“单遍高效随机化 AE 的简洁性”。

### 5.4 认证保留与保密退化必须分开看

很多介绍在这里会说“GCM-SIV 更安全”。更准确的说法应该是：

- 在 standard nonce discipline 下，三者都可以给出很强的 AE 边界。
- 在 repeated nonce 下，GCM 会出现 catastrophic confidentiality/integrity failure。
- SIV / GCM-SIV 试图保留认证边界，同时把保密退化限制到等值泄漏级别。

这比“更安全”这种空泛判断更接近攻击者真正关心的问题。

## 6. 三卷收束：nonce discipline、认证边界、misuse 退化

到这里，这条讲义线的前三卷其实已经形成三层结构：

### 6.1 Vol.1

Vol.1 讲的是：如果模式根本不认证，那么攻击者总能在解密方程里找到一个可控项。

### 6.2 Vol.2

Vol.2 讲的是：AEAD 通过 reject 语义和认证方程切断这些主动攻击面，但代价是安全边界严重依赖 unique nonce。

### 6.3 Vol.3

Vol.3 讲的是：若必须面对 nonce misuse，那么唯一可行的方向不是继续假装 repeated nonce 不会发生，而是把外部 nonce 从“直接选流”改成“先进入 synthetic IV 认证层”，从而把灾难性崩塌降级为 deterministic equality leakage 或其他 bounded degradation。

所以这三卷真正建立起来的，不是一个“模式名词表”，而是三条连续边界：

1. `malleability boundary`
2. `authentication boundary`
3. `misuse boundary`

而这三条边界都还没有碰到 storage encryption 的问题。消息加密里的 misuse-resistant AE 依然不等于 authenticated storage，也不等于 tweakable enciphering。若继续往后走，下一卷自然就该切到：

- tweakable block ciphers
- XTS
- replay / rollback / sector relocation
- storage confidentiality vs authenticated storage

## References

- RFC 5116, [*An Interface and Algorithms for Authenticated Encryption*](https://www.rfc-editor.org/info/rfc5116)
- RFC 5297, [*Synthetic Initialization Vector (SIV) Authenticated Encryption Using the Advanced Encryption Standard (AES)*](https://www.rfc-editor.org/info/rfc5297)
- RFC 8452, [*AES-GCM-SIV: Nonce Misuse-Resistant Authenticated Encryption*](https://www.rfc-editor.org/info/rfc8452)
- Phillip Rogaway and Thomas Shrimpton, [*Deterministic Authenticated-Encryption: A Provable-Security Treatment of the Key-Wrap Problem*](https://doi.org/10.1007/11818175_8)

