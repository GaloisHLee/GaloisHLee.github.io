# 分组密码工作模式攻击讲义 Part 2：AEAD 的构造逻辑与认证边界


> Note: Vol.1 的核心结论是，保密性模式只要还保留一个“对任意篡改密文都返回某个明文”的解密接口，攻击者就仍然能沿着那条接口做线性利用。Vol.2 要补上的，不是又一个模式名词，而是这个接口何时会从总函数变成部分函数。

AEAD 的关键不在于“密文后面多挂了一个 tag”，而在于系统把 nonce、associated data、payload、长度和顺序一起压进一个验证方程，并把解密语义改写成只允许输出 \(P\) 或 \(\bot\)。一旦这个改写成立，Vol.1 里最常见的 bit flip、reorder、splice、truncate 利用链就不再天然拥有一个可调用的明文 oracle。

这一卷只讲原理和推导，不讲 API、不讲 PoC、不讲题目套路。主线只有三条: GCM 的 \(CTR+GHASH\)，CCM 的 \(CTR+CBC\text{-}MAC\)，以及 OCB 的 offset/checksum。它们都能做 AEAD，但它们的认证对象、状态对象和 nonce 进入系统的方式并不相同，因此 misuse 时的退化方式也并不相同。

<!--more-->

## 0. Threat Model 与记号

本卷默认底层分组密码 \(E_K\) 是一个强伪随机置换族：

$$
E_K : \{0,1\}^{128} \to \{0,1\}^{128}, \qquad D_K(E_K(X)) = X.
$$

这里的 \(E_K\) 可以是 AES，也可以是 SM4。只要它表现得像理想的 128-bit block cipher，本卷关注的主要风险就不在轮函数内部，而在 mode of operation 怎样组织状态、怎样绑定输入，以及怎样暴露接口。

统一把 AEAD 接口写成：

$$
\mathcal{E}_K(N, A, P) = (C, T), \qquad \mathcal{D}_K(N, A, C, T) \in \{P, \bot\}.
$$

其中：

- \(N\) 是 nonce
- \(A\) 是 associated data
- \(P\) 是 plaintext
- \(C\) 是 ciphertext
- \(T\) 是 authentication tag

攻击者最终观察到的是一个 transcript：

$$
\tau = (N, A, C, T).
$$

本卷反复追问的不是“用了没用 AES”，而是下面四件事：

1. \(N\) 进入了哪一层状态。
2. \(A\) 到底被绑定到了哪里。
3. \(\mathcal{D}_K\) 的 \(\bot\) 语义是否真的统一而不可区分。
4. nonce 一旦重复，攻击者能从两份 transcript 中消去什么。

还需要预先固定一个经常被轻描淡写、但其实是证明前提的条件：`nonce-respecting adversary`。标准 nonce-based 安全游戏默认同一密钥下加密查询满足

$$
N_i \neq N_j \qquad \text{for all } i \neq j.
$$

如果这条前提被打破，很多安全定理不是“变弱一点”，而是直接不适用。

## 1. 从 Vol.1 到 AEAD：为什么保密性不足以结束问题

Vol.1 反复出现过两条攻击方程。

流式模式里，攻击面通常可压成

$$
P = C \oplus S.
$$

CBC 类模式里，攻击面通常可压成

$$
P_i = D_K(C_i) \oplus C_{i-1}.
$$

这两条式子的共同点不是“都有 XOR”，而是它们都把解密接口保留成了一个总函数。只要攻击者提交某个被篡改过的 \(\widetilde{C}\)，系统仍会返回某个 \(\widetilde{P}\)。于是：

- 在 CTR / OFB / CFB 里，攻击者可以直接在线性层操纵 \(\widetilde{P}\)。
- 在 CBC 里，攻击者虽然不能直接控制每一位 \(D_K(C_i)\)，但可以通过控制 \(C_{i-1}\) 精确控制下一块的 XOR 后果。
- 在任何没有认证的模式里，replay、cut-and-paste、truncate 只要语法上还能过，就不会被密码层主动拦住。

所以 Vol.1 的真正结论不是“CBC 不好，CTR 也危险”，而是：

> 只保密的 mode，无论在 passive setting 下多么像随机加密，只要 decryption oracle 对任意修改都吐出某个明文，它就仍然保留着主动攻击面。

AEAD 引入的 \(\bot\) 不是装饰。它把解密接口从

$$
\operatorname{Dec}_K : (N, C) \mapsto P
$$

改成了

$$
\mathcal{D}_K : (N, A, C, T) \mapsto \{P, \bot\}.
$$

从攻击者视角看，最重要的变化不是多了一个输入 \(T\)，而是新鲜的、未见过的 tuple \((N,A,C,T)\) 只有在极小概率下才会落进 \(P\) 分支。

## 2. AEAD 的形式化接口与安全目标

### 2.1 syntax：认证的是 tuple，不是某段孤立字节串

AEAD 的语义对象不是“payload 加一段 tag”，而是一个四元组认证问题：

$$
(N, A, P) \xmapsto{\mathcal{E}_K} (N, A, C, T).
$$

其中 \(A\) 不需要保密，但它必须被认证，因为协议上下文往往就藏在 \(A\) 里，例如：

- record type
- direction bit
- sequence number
- algorithm identifier
- version
- domain separator

如果这些上下文信息没有进入认证对象，那么攻击者即便无法改写 payload，也可能把一条在语义域 \(\mathsf{ctx}_1\) 下有效的消息搬运到 \(\mathsf{ctx}_2\) 下解释。

更严格地说，AAD 的编码必须是单射。若一个协议把若干逻辑字段 \((x_1,\ldots,x_s)\) 编成 \(A\)，则至少要满足：

$$
\operatorname{enc}_A(x_1,\ldots,x_s) = \operatorname{enc}_A(x_1',\ldots,x_s') \Longrightarrow (x_1,\ldots,x_s) = (x_1',\ldots,x_s').
$$

否则密码层认证的只是某个有歧义的 bitstring，而不是唯一语义对象。

### 2.2 reject 语义为何改变攻击模型

对 AEAD 来说，完整性目标的核心可以写成：攻击者即便自适应访问加密 oracle，也不能输出一个新鲜且可接受的 tuple。

若记加密查询历史为

$$
\mathcal{Q}_E = \{(N_i, A_i, C_i, T_i)\},
$$

那么一个典型的 `INT-CTXT` 风格目标是要求

$$
\Pr\Big[(N,A,C,T)\notin \mathcal{Q}_E \ \land\ \mathcal{D}_K(N,A,C,T)\neq \bot\Big]
$$

是可忽略的。

这一定义和 Vol.1 的世界有根本差别。Vol.1 里的攻击常常依赖这样一个事实：任意篡改后的输入都会落在某个明文上。现在这个条件被打断了。攻击者若想继续利用解密 oracle，第一步必须先完成一个 forgery，也就是先让系统接受一个从未出现过的 \((N,A,C,T)\)。

### 2.3 confidentiality 与 integrity 不再能混成一句话

`IND-CPA` 关心的是，攻击者区分不了挑战明文来自哪一个候选。

`INT-CTXT` 关心的是，攻击者造不出新的可接受密文。

`IND-CCA` 关心的是，即便攻击者除了加密 oracle 还有解密 oracle，仍然不能在挑战上获得区分优势。

在 nonce-based AEAD 里，这三者的关系可以直观理解为：

- 只有 IND-CPA，没有 INT-CTXT：Vol.1 里的很多可塑性攻击仍然能活着。
- 只有 INT-CTXT，没有 IND-CPA：你挡住了伪造，但仍可能泄漏消息内容。
- 两者同时成立：攻击者既不能看穿挑战，也不能把解密 oracle 重新变回一个明文计算器。

这也是为什么 AEAD 不是“加密之后顺手再拼一个校验值”。它要求认证项参与定义解密语义本身，而不是只在日志里记一笔。

### 2.4 \(\bot\) 必须是一个符号，而不是一簇可观察分支

理论里只写一个 \(\bot\)，但实现里很容易悄悄变成很多可区分的失败分支：

- `bad length`
- `bad padding`
- `bad tag`
- `bad aad`
- `bad nonce`
- `bad parse`

如果这些失败分支在响应码、时间、连接行为或日志路径上可区分，那么密码层虽然名义上引入了 \(\bot\)，系统层却又把 \(\bot\) 拆回了 oracle。此时安全性证明仍然成立于理想接口，但真实系统已经回到了 Vol.1 风格的利用世界。

### 2.5 integrity 不是 freshness

AEAD 可以拒绝被篡改的 tuple，但它不会自动拒绝“原封不动重放”的旧 tuple。

如果 \((N,A,C,T)\) 曾经是合法输出，那么再次提交同一 tuple 时，解密通常仍会返回同一 \(P\)。因此：

$$
\text{integrity} \neq \text{anti-replay}.
$$

freshness 仍需要协议单独跟踪 sequence number、nonce set、epoch 或 state machine。

## 3. GCM：CTR 与有限域多项式认证如何耦合

GCM 是 Vol.2 的核心，因为它把两件事情叠在了一起：

1. 用 CTR 提供高吞吐的保密层。
2. 用 \(GF(2^{128})\) 上的多项式求值提供认证层。

### 3.1 GCM 的最小方程组

GCM 先定义一个 hash subkey：

$$
H = E_K(0^{128}).
$$

然后从 nonce 构造 pre-counter block \(J_0\)。

若 \(|N| = 96\)，则

$$
J_0 = N \,\|\, 0^{31} \,\|\, 1.
$$

若 \(|N| \neq 96\)，则

$$
J_0 = GHASH_H\Big(N \,\|\, 0^s \,\|\, [|N|]_{64}\Big),
$$

其中 \(s\) 用来把输入补到 128-bit block 边界。

接着用 \(J_0\) 生成 CTR 流：

$$
C = GCTR_K(\operatorname{inc32}(J_0), P).
$$

最后定义 tag：

$$
T = \operatorname{MSB}_t\Big(E_K(J_0) \oplus S\Big),
$$

其中

$$
S = GHASH_H\Big(A \,\|\, 0^v \,\|\, C \,\|\, 0^u \,\|\, [|A|]_{64} \,\|\, [|C|]_{64}\Big).
$$

这里的

$$
v = 128\left\lceil \frac{|A|}{128} \right\rceil - |A|,
\qquad
u = 128\left\lceil \frac{|C|}{128} \right\rceil - |C|.
$$

所以 GCM 不是“CTR 后面再哈一下”，而是一个非常具体的分层结构：

- \(J_0\) 决定 keystream 与 tag mask
- \(H\) 决定 GHASH 的多项式求值点
- \(A\) 与 \(C\) 先被唯一编码，再喂进 GHASH

### 3.2 GHASH 不是黑盒哈希，而是 \(GF(2^{128})\) 上的 Horner 递推

把每个 128-bit block 看作 \(GF(2^{128})\) 的元素。其乘法在约化多项式

$$
u^{128} + u^7 + u^2 + u + 1
$$

下进行。

若格式化后的输入块为

$$
X_1, X_2, \ldots, X_m,
$$

则 GHASH 定义为

$$
Y_0 = 0^{128},
$$

$$
Y_i = (Y_{i-1} \oplus X_i)\cdot H \qquad \text{for } 1\le i \le m,
$$

最终输出

$$
GHASH_H(X_1,\ldots,X_m) = Y_m.
$$

把递推展开，就得到一个公开系数、秘密求值点的多项式：

$$
Y_m = X_1 H^m \oplus X_2 H^{m-1} \oplus \cdots \oplus X_m H.
$$

这一步极其关键。GCM 的认证强度并不来自“一个普通 MAC 黑盒”，而是来自：

1. \(H\) 对攻击者未知。
2. 对唯一 nonce 而言，\(E_K(J_0)\) 提供了 one-time mask。
3. GHASH 对不同输入向量的碰撞概率可被 `universal-hash` 边界控制。

### 3.3 为什么长度块必须进入 GHASH

GCM 不是在认证原始串 \(A\|C\)，而是在认证一个唯一编码后的 pair \((A,C)\)。

没有长度块时，攻击者会立刻尝试问这样的问题：

$$
(A,C) \stackrel{?}{\neq} (A',C')
\quad \text{but} \quad
\operatorname{fmt}(A,C) = \operatorname{fmt}(A',C').
$$

如果认证输入只是一段模糊拼接后的比特串，那么以下几类歧义都会出现：

- 哪些比特属于 associated data，哪些属于 ciphertext
- 最后一块前面的零是 padding 还是消息内容
- 不同长度但同一 block decomposition 的对象是否会被同一 GHASH 解释

GCM 通过

$$
\Phi(A,C) = A \,\|\, 0^v \,\|\, C \,\|\, 0^u \,\|\, [|A|]_{64} \,\|\, [|C|]_{64}
$$

强制把 pair \((A,C)\) 编成唯一对象。换句话说，GHASH 认证的不是某段“长得像”消息的串，而是这对对象的唯一语法树。

### 3.4 为什么 \(\operatorname{MSB}_t(E_K(J_0)\oplus S)\) 这个结构成立

tag 写成

$$
T = \operatorname{MSB}_t(E_K(J_0)\oplus S)
$$

而不是直接写成 \(GHASH_H(A,C)\)，原因是 GHASH 本身是线性的。线性对象若不再加一个 nonce-dependent one-time mask，很容易被重复查询积累信息。

因此 GCM 的认证层有两部分：

- \(S\)：把 \(A\) 和 \(C\) 压成一个有限域元素
- \(E_K(J_0)\)：对该元素施加一次性遮罩

唯一 nonce 的作用并不只在保密层。它同样保证了这个 tag mask 不会跨消息复用。

### 3.5 nonce reuse：先坏保密，再坏认证，而且是可写成方程的坏

若同一 key 下两次使用同一 nonce \(N\)，则 \(J_0\) 相同，CTR keystream 相同，于是

$$
C \oplus C' = P \oplus P'.
$$

这是 CTR 继承来的第一层崩塌。

第二层崩塌发生在 tag 上。为避免被 tag truncation 干扰，先假设 \(t=128\)。此时

$$
T = E_K(J_0)\oplus GHASH_H(\Phi(A,C)),
$$

$$
T' = E_K(J_0)\oplus GHASH_H(\Phi(A',C')).
$$

两式相异或，nonce-dependent mask 被完全消去：

$$
T \oplus T' = GHASH_H(\Phi(A,C)) \oplus GHASH_H(\Phi(A',C')).
$$

设两条 transcript 对应的格式化块向量分别为

$$
\mathbf{X} = (X_1,\ldots,X_m), \qquad \mathbf{X}' = (X_1',\ldots,X_m').
$$

则有

$$
T \oplus T' = \sum_{i=1}^{m} (X_i \oplus X_i') H^{m-i+1}.
$$

也就是说，攻击者观察到的是一个关于未知点 \(H\) 的多项式值：

$$
\Delta T = M_{\Delta \mathbf{X}}(H).
$$

把它再写得更形式化一点。对每一对 repeated-nonce transcript，都可以定义一个公开系数多项式

$$
F_{\Delta \mathbf{X},\Delta T}(Z) = M_{\Delta \mathbf{X}}(Z) \oplus \Delta T.
$$

真正的 hash subkey \(H\) 满足

$$
F_{\Delta \mathbf{X},\Delta T}(H) = 0.
$$

因此一旦 nonce 重复，认证问题就从“猜一个 128-bit tag”退化成“求一个 128-bit 域元素 \(H\) 作为若干公开多项式的公共根”。如果 \(F\) 不是零多项式，且其次数为 \(d\)，那么在域 \(GF(2^{128})\) 上：

$$
\big|\{z \in GF(2^{128}) : F_{\Delta \mathbf{X},\Delta T}(z)=0\}\big| \le d.
$$

也就是说，一条 collision transcript 未必立刻唯一确定 \(H\)，但它已经把 \(H\) 的候选集合从 \(2^{128}\) 个点压缩到了至多 \(d\) 个根。多组 repeated-nonce transcript 则继续做根集合求交：

$$
H \in \bigcap_{j=1}^{q} \{z : F_j(z)=0\}.
$$

这就是 GCM nonce reuse 特别危险的地方。它不仅破坏保密性，而且把认证问题转成了一个公开系数、秘密求值点的代数问题。

### 3.6 Forbidden Attack 的原理入口

上式并不意味着攻击者立刻恢复 \(K\)。这里要分清三件事：

1. \(H = E_K(0^{128})\) 不是 AES key 本身。
2. 恢复 \(H\) 并不等于反演 block cipher。
3. 但恢复 \(H\) 已经足以破坏认证。

更具体地说，Forbidden Attack 的根本机制不是“神秘地猜中 tag”，而是先把 repeated nonce 变成 \(H\) 的根求解问题，再把认证退化成一个可计算函数。

先看一个最简单、但已经足够说明问题的情形：两条消息都满足

$$
A = A' = \varnothing, \qquad |C| = |C'| = 128.
$$

此时格式化输入都是两块：

$$
\Phi(\varnothing,C) = (C, L), \qquad \Phi(\varnothing,C') = (C', L),
$$

其中

$$
L = [0]_{64} \,\|\, [128]_{64}
$$

是相同的长度块。于是

$$
GHASH_H(\Phi(\varnothing,C)) = C H^2 \oplus L H,
$$

$$
GHASH_H(\Phi(\varnothing,C')) = C' H^2 \oplus L H.
$$

在 repeated-nonce 下相减，长度项完全抵消：

$$
T \oplus T' = (C \oplus C') H^2.
$$

只要 \(C \neq C'\)，就得到

$$
H^2 = (T \oplus T') (C \oplus C')^{-1}.
$$

而在 \(GF(2^{128})\) 中，Frobenius 映射 \(x \mapsto x^2\) 是一个双射，所以 \(H\) 被唯一确定。也就是说，在 full-tag 视角下，哪怕只发生一次这样的 nonce collision，攻击者都已经不再面对 \(2^{128}\) 的盲猜空间，而是在直接解出 hash subkey。

更一般地，若重复 nonce 的 transcript 对应一个低次数多项式

$$
F(H)=0,
$$

则攻击者得到的是一个小候选集合；若再拿到额外 collision transcript，就继续做根集合求交，直到 \(H\) 唯一。tag truncation 会提高样本需求，因为攻击者只看到 \(T\) 的前 \(t\) 位，但这个变化只影响约束强度，不改变攻击本质：nonce reuse 仍然把“认证”改写成关于 \(H\) 的代数恢复问题。

原因很直接。若攻击者在 repeated-nonce transcript 中通过足够多的约束恢复了 \(H\)，那么它就可以从任意一条合法 transcript 中解出该 nonce 对应的 mask：

$$
\Pi_N = E_K(J_0) = T \oplus GHASH_H(\Phi(A,C)).
$$

随后，对同一 nonce \(N\) 下任意伪造目标 \((A^\star, C^\star)\)，它都能计算

$$
T^\star = \Pi_N \oplus GHASH_H(\Phi(A^\star,C^\star)).
$$

这样 forged tuple \((N, A^\star, C^\star, T^\star)\) 就会通过验证。

攻击者需要恢复的不是 \(K\)，而只是：

- hash subkey \(H\)
- 某个已重复 nonce 的一次性 tag mask \(\Pi_N\)

对 GCM 来说，这已经足够致命。

### 3.7 正常使用时，攻击者看到了什么；没有看到什么

在 nonce 唯一、且 decryption 只返回 \(P\) 或 \(\bot\) 时，单条 transcript

$$
(N,A,C,T)
$$

给攻击者留下的是：

- 可见的 nonce \(N\)
- 可见的 associated data \(A\)
- 经 CTR 掩蔽后的 \(C\)
- 经 one-time mask 掩蔽后的 \(GHASH_H\) 值

但它看不到：

- keystream \(E_K(J_i)\)
- hash subkey \(H\)
- pre-counter mask \(E_K(J_0)\)

一旦 nonce 重复，前两类不可见对象开始通过消元间接显露结构；这就是 GCM 的真正硬边界。

## 4. CCM：CTR 加 CBC-MAC 为什么比口号更复杂

CCM 常被一句话概括成

$$
\text{CCM} = \text{CTR} + \text{CBC-MAC}.
$$

这句话没有错，但远远不够。因为 CCM 的安全性很大程度上依赖于：CBC-MAC 认证的到底是什么、怎样编码、怎样与 CTR 域分离。

### 4.1 先格式化，再谈 MAC

CCM 首先把 \((N,A,P)\) 格式化成 block sequence

$$
B_0, B_1, \ldots, B_r.
$$

其中最关键的起点块是

$$
B_0 = \operatorname{Flags} \,\|\, N \,\|\, [|P|]_q.
$$

这里 \(q\) 是 payload length field 的字节数，满足 nonce 长度与 \(q\) 一起决定 128-bit block 的剩余布局。`Flags` 本身又编码了三件事情：

- 是否存在 associated data
- tag length 参数
- \(q\) 的大小

associated data 则被编码成一个长度前缀唯一的串，再和 payload blocks 一起进入 \(B_1,\ldots,B_r\)。

所以 CCM 里真正被 CBC-MAC 认证的对象不是一句松散的“nonce + AAD + plaintext”，而是一个精确格式化后的结构化串：

$$
\operatorname{Fmt}(N,A,P,t,q) = B_0 \,\|\, B_1 \,\|\, \cdots \,\|\, B_r.
$$

### 4.2 CBC-MAC 链真正绑定了什么

设

$$
Y_0 = 0^{128},
$$

然后对 \(0\le i\le r\) 递推

$$
Y_{i+1} = E_K(Y_i \oplus B_i).
$$

raw authentication value 定义为

$$
T_\star = \operatorname{MSB}_t(Y_{r+1}).
$$

这条链意味着：任何对 \(\operatorname{Fmt}(N,A,P,t,q)\) 的修改，都会被传递到最终 \(Y_{r+1}\) 中。于是：

- 改 payload，不可能不改 tag
- 改 AAD，不可能不改 tag
- 改 payload length，不可能不改 tag
- 改 tag length 参数本身，也不可能不改 tag

这正是 CCM 比“CTR 后挂一个普通 MAC”更细的地方。它把解释消息所必需的元数据一起拉进了认证对象。

### 4.3 为什么长度编码与 parser branch 都必须进入 \(B_0\)

若没有 \([|P|]_q\)，则同一 block sequence 可能对应多种“在哪截断”的 payload 解释，truncation 就无法被密码层唯一识别。

若没有 tag length 参数，验证方甚至可能在两套不同的 `accept` 语义下解释同一 tuple。

若没有 `Adata` bit，解析器也无法知道自己是否应当读取 associated data length field。

因此 \(B_0\) 的意义不是“凑够 128 bit 方便实现”，而是：

> 把验证语义所依赖的控制信息放进认证对象本身。

### 4.4 associated data 长度编码为什么是证明前提，而不是实现细节

这里最好把要求写成一个正式的编码性质。CCM 的证明并不是直接面对 tuple 空间

$$
(N,A,P,t,q),
$$

而是先面对由 formatter 生成的 block language

$$
\mathcal{L}_{\mathrm{CCM}} = \{\operatorname{Fmt}(N,A,P,t,q)\}.
$$

CBC-MAC 真正作用的对象是 \(\mathcal{L}_{\mathrm{CCM}}\) 里的字符串，而不是原始 tuple 本身。因此证明所需要的不是模糊的“编码看起来合理”，而是：

$$
\operatorname{Fmt}(N,A,P,t,q) = \operatorname{Fmt}(N',A',P',t',q') \Longrightarrow (N,A,P,t,q)=(N',A',P',t',q').
$$

也就是 formatter 必须是单射。把注意力只放在 AAD 这一层，则至少要有：

$$
\operatorname{enc}_A(A) = \operatorname{enc}_A(A') \Longrightarrow A = A'.
$$

如果这一步不满足，那么即使 CBC-MAC 本身无懈可击，系统也只是在认证一个有歧义的序列。更糟的是，这里根本不需要概率型 forgery。若存在

$$
A \neq A' \qquad \text{but} \qquad \operatorname{enc}_A(A)=\operatorname{enc}_A(A'),
$$

并固定同一个 \(N,P,t,q\)，就立刻有

$$
\operatorname{Fmt}(N,A,P,t,q)=\operatorname{Fmt}(N,A',P,t,q).
$$

于是 CBC-MAC 链逐块相同：

$$
Y_i = Y_i' \qquad \text{for all } i,
$$

进而

$$
T_\star = T_\star', \qquad U = U'.
$$

也就是说，

$$
\mathcal{E}_K(N,A,P)=\mathcal{E}_K(N,A',P)
$$

会以概率 1 成立。攻击者此时做的不是“伪造一个新 tag”，而是把一份完全合法的 transcript 重新解释到另一个语义域里。这不是密码强度下降，而是 message representation 已经从根上失去唯一性。

可以用一个抽象反例看得更直。假设某个错误实现把 AAD 编码成

$$
\widetilde{\operatorname{enc}}_A(A)=A \,\|\, 0^{v(A)},
$$

也就是只补零到 block 边界，却不显式写入长度。那么只要 \(1 \le k \le v(A)\)，就有

$$
\widetilde{\operatorname{enc}}_A(A)=\widetilde{\operatorname{enc}}_A(A \,\|\, 0^k),
$$

因为 trailing zeros 到底是“真实 AAD 内容”还是“对齐 padding”，解析器已经分不出来。于是 \(A\) 与 \(A\|0^k\) 会产生相同的 MAC 输入。这里失败的不是 AES，也不是 CBC-MAC，而是 `parse` 不再是函数。

从这个角度看，CCM 里的 associated-data length encoding、\(B_0\) 里的 `Adata` bit、payload length field 和 tag-length/\(q\) 参数，本质上都在服务同一个条件：

$$
\operatorname{Parse}(\operatorname{Fmt}(N,A,P,t,q)) = (N,A,P,t,q).
$$

也就是验证方必须能从被认证的 bitstring 中唯一恢复出它自己正在认证什么。

这一点和 GCM 里的 length block 是同一类原则：

- GCM 用显式长度块避免 \((A,C)\) 的歧义拼接
- CCM 用 prefix-free 编码避免 associated data 的歧义解析

### 4.5 CTR 封装 payload 与 raw tag

CCM 的计数器块写成

$$
\operatorname{Ctr}_i = \operatorname{Flags}' \,\|\, N \,\|\, [i]_q,
$$

其中 \(\operatorname{Flags}'\) 与 \(B_0\) 的 `Flags` 明确区分，用于域分离。

定义 keystream blocks：

$$
S_i = E_K(\operatorname{Ctr}_i).
$$

payload ciphertext 为

$$
C = P \oplus \operatorname{MSB}_{|P|}(S_1 \,\|\, S_2 \,\|\, \cdots).
$$

而对 tag 的封装则是

$$
U = T_\star \oplus \operatorname{MSB}_t(S_0).
$$

最终输出可写成

$$
\mathcal{E}_K(N,A,P) = (C, U).
$$

这里再次体现出 nonce 的双重角色：

- 它决定 payload 的 CTR keystream
- 它也决定 raw tag \(T_\star\) 的一次性 mask

### 4.6 为什么 counter domain 必须与 MAC domain 分离

如果某个 \(\operatorname{Ctr}_i\) 能与某个 \(B_j\) 相撞，那么同一个 block-cipher input 就会同时出现在：

- CBC-MAC 链内部状态更新
- CTR keystream 生成

这会把本应分开的“认证域”和“加密域”缠在一起。CCM 通过让 \(B_0\) 与 \(\operatorname{Ctr}_i\) 使用不同控制位布局，来保证这两个命名空间不会相撞。

因此，`Flags` 不是为了协议美观，而是为了让

$$
\{B_0,B_1,\ldots,B_r\} \cap \{\operatorname{Ctr}_0,\operatorname{Ctr}_1,\ldots\}
$$

在结构上被强行隔开。

### 4.7 nonce reuse：为什么 CCM 不像 GCM 那样“线性优雅”，但仍然会坏

若 nonce 重复，则所有 \(\operatorname{Ctr}_i\) 重复，因而 payload keystream 重复：

$$
C \oplus C' = P \oplus P'.
$$

这是与 CTR 完全一致的第一层崩塌。

对 tag 而言，由于

$$
U = T_\star \oplus \operatorname{MSB}_t(S_0),
$$

同一 nonce 还会复用同一个 \(\operatorname{MSB}_t(S_0)\)。于是

$$
U \oplus U' = T_\star \oplus T_\star'.
$$

和 GCM 不同，右边不是公开多项式在未知点 \(H\) 上的值，而是两个 CBC-MAC 输出前缀的异或。这使得 CCM 的 misuse 后果不如 GCM 那样能立刻写成一个干净的有限域代数方程，但这并不意味着它安全得多。真实情况是：

- 保密性仍然立刻退化成 two-time pad
- tag 的一次性封装也被撤销
- 认证边界回退到“直接比较 raw CBC-MAC 输出差分”的世界

所以 CCM 的 repeated-nonce 崩塌虽然没有 GCM 那样漂亮的 `forbidden attack polynomial`，却同样足以破坏其核心安全前提。

### 4.8 \(\bot\) 在 CCM 里尤其不能被拆开

CCM 的 decryption-verification 流程里，理论上只应暴露一个结论：

$$
\text{accept} \quad \text{or} \quad \bot.
$$

如果实现把失败原因细分为“associated data 长度不对”“payload 长度不对”“tag 比对失败”“格式块不合法”，并通过时序或错误码泄露出来，那么理论上的 authenticated decryption 又被拆成了多个可观察 oracle。

因此 CCM 给出的安全接口不是“先解再慢慢报错”，而是：

> 所有验证失败在可观察层面都必须坍缩成同一个 \(\bot\)。

## 5. OCB：offset、checksum 与单遍 AEAD 的第三条路线

GCM 用 universal hash。CCM 用 CBC-MAC。OCB 走的是第三条路：为每个 block 生成位置相关的 offset，把保密与认证在同一遍流里缠起来。

### 5.1 从 \(L_\ast\) 到 \(L_i\)：先构造 offset 常量族

OCB 先定义：

$$
L_\ast = E_K(0^{128}),
$$

$$
L_\$ = \operatorname{dbl}(L_\ast),
$$

$$
L_0 = \operatorname{dbl}(L_\$),
$$

$$
L_i = \operatorname{dbl}(L_{i-1}) \qquad \text{for } i\ge 1.
$$

这里的 \(\operatorname{dbl}\) 是 \(GF(2^{128})\) 中乘以 \(x\) 的操作，也就是一次“左移加条件约化”。这套常量族的作用，是用很低代价为不同 block index 准备 tweak 素材。

### 5.2 nonce 如何进入 \(Offset_0\)

OCB 不直接把 nonce 当作某个 block index，而是先把它加工成初始 offset。设格式化后的 Nonce 最低 6 位为 `bottom`，再令

$$
Ktop = E_K(\operatorname{Nonce}[1..122] \,\|\, 0^6),
$$

$$
\operatorname{Stretch} = Ktop \,\|\, \big(Ktop[1..64] \oplus Ktop[9..72]\big).
$$

然后取一段滑动窗口：

$$
Offset_0 = \operatorname{Stretch}[1+\operatorname{bottom} \, .. \, 128+\operatorname{bottom}].
$$

这一步的意义不是“实现 trick”，而是：

> 用 nonce 选择一条新的 offset 轨道，使后续所有 block tweak 都锚定在这条轨道上。

### 5.3 为什么相同明文块不会再像 ECB 那样暴露相等关系

对 full blocks，OCB 的主加密式是

$$
Offset_i = Offset_{i-1} \oplus L_{\operatorname{ntz}(i)},
$$

$$
C_i = Offset_i \oplus E_K(P_i \oplus Offset_i),
$$

其中 \(\operatorname{ntz}(i)\) 是 \(i\) 的二进制末尾零个数。

如果出现两个相同明文块 \(P_i=P_j\)，只要 \(Offset_i \neq Offset_j\)，那么它们进入 block cipher 之前的输入就是：

$$
P_i \oplus Offset_i \neq P_j \oplus Offset_j.
$$

于是 \(E_K\) 的输入点被分离开来，ECB 那种“相同 block 映到相同密文”的等值泄漏不再成立。

所以 OCB 的 offset 不是附属 tweak，而是保密层本身的一部分：它让“同一明文块”在不同位置、不同 nonce 下都落到不同的 enciphering point 上。

### 5.4 checksum 为什么是认证的真正汇合点

仅有 per-block offset 还不够，因为攻击者仍可能尝试替换、重排或丢弃 block。OCB 因此维护一个 plaintext checksum：

$$
Checksum_0 = 0^{128},
$$

$$
Checksum_i = Checksum_{i-1} \oplus P_i.
$$

在 full-block 情形下，最终 tag 写成：

$$
Tag = E_K(Checksum_m \oplus Offset_m \oplus L_\$) \oplus HASH_K(A).
$$

这说明 OCB 的 tag 实际上把三类对象合并到了一起：

- 所有 plaintext full blocks 的全局 XOR 汇总 \(Checksum_m\)
- 由 nonce 与 block index 共同决定的 offset 轨道
- associated data 的单独 hash 路径 \(HASH_K(A)\)

与 GCM/CCM 的差别在于：OCB 不是先“整串算 MAC，再另做加密”，而是让每块 plaintext 的保密处理与最终认证汇入同一条 offset/checksum 结构。

### 5.5 associated data 在 OCB 中走的是另一条 HASH 路

OCB 对 AAD 使用独立的 HASH 过程。设 associated-data blocks 为 \(A_1,\ldots,A_a\)，可写成：

$$
\Sigma_0 = 0^{128}, \qquad \Delta_0 = 0^{128},
$$

$$
\Delta_i = \Delta_{i-1} \oplus L_{\operatorname{ntz}(i)},
$$

$$
\Sigma_i = \Sigma_{i-1} \oplus E_K(A_i \oplus \Delta_i).
$$

最后把

$$
HASH_K(A) = \Sigma_a
$$

并入 tag。

因此 OCB 的 AAD 绑定不是“顺手附加一个字段”，而是显式并入最终认证值的组成项。攻击者若只改 \(A\) 不改 \(C\)，tag 同样会失效。

### 5.6 为什么 OCB 不是 GCM 或 CCM 的小变体

GCM 的核心是：

- 公开系数
- 秘密求值点 \(H\)
- one-time tag mask

CCM 的核心是：

- 结构化格式化
- CBC-MAC 链
- CTR 对 payload 和 raw tag 的封装

OCB 的核心则是：

- nonce-derived offset 轨道
- block-index dependent tweak 递推
- plaintext checksum
- AAD 的独立 HASH 路线

所以 OCB 的设计哲学不是“也给 CTR 加个 tag”，而是：

> 让每个 block 自带位置相关 tweak，同时让整条消息在最后汇入一个 global authentication equation。

### 5.7 nonce reuse：为什么同一 offset 轨道被重用会很危险

若 nonce 重复，则 \(Offset_0\) 重复，进而整条

$$
Offset_1, Offset_2, \ldots
$$

轨道都重复。

这时，两个同位置 block 的加密分别是

$$
C_i = Offset_i \oplus E_K(P_i \oplus Offset_i),
$$

$$
C_i' = Offset_i \oplus E_K(P_i' \oplus Offset_i).
$$

相异或后得到

$$
C_i \oplus C_i' = E_K(P_i \oplus Offset_i) \oplus E_K(P_i' \oplus Offset_i).
$$

这不像 CTR 那样直接等于 \(P_i\oplus P_i'\)，但危险点在别处：攻击者现在拿到的是同一组 tweak 下的多次 enciphering 结果。也就是说，nonce 本应提供的一次性轨道被重用了。

这种重用会同时伤到两层：

- 保密层：相同位置上的 block 不再享有新的 offset 轨道
- 认证层：checksum/tag 组合复用同一 nonce 相关结构

与 GCM 相比，OCB 的 misuse 后果不那么容易压成一个公开多项式攻击；但从原理上看，破坏点是同一类：nonce 导出的 one-time structure 不再 one-time。

## 6. 统一比较：三条认证路线到底绑定了什么

把三种模式并排看，问题会比“谁更快”“谁更常见”更清楚。

| 模式 | 认证核心 | nonce 进入哪里 | 明文绑定对象 | AAD 绑定对象 | reuse 后首先被消去的东西 |
| --- | --- | --- | --- | --- | --- |
| GCM | \(GHASH_H\) | \(J_0\)、CTR 流、tag mask | ciphertext 经 GHASH 进入 tag | 同一 GHASH 输入 | \(E_K(J_0)\) |
| CCM | CBC-MAC | \(B_0\)、counter blocks、tag mask | 格式化后的 \(B_i\) 链 | 同一格式化串的一部分 | \(\operatorname{MSB}_t(S_0)\) |
| OCB | offset + checksum | \(Offset_0\) 轨道 | per-block offset 加 checksum | 独立的 \(HASH_K(A)\) | 整条 offset 轨道 |

### 6.1 从攻击者能消去什么，看模式的真正边界

对攻击者来说，最有价值的问题是：给我两条同 key transcript，我能消去什么？

GCM 下，重复 nonce 直接消去 tag mask：

$$
T \oplus T' = M_{\Delta \mathbf{X}}(H).
$$

CCM 下，重复 nonce 消去 raw tag 的 CTR 封装：

$$
U \oplus U' = T_\star \oplus T_\star'.
$$

OCB 下，重复 nonce 消去的是“本应一次性使用的 offset 轨道更新来源”。虽然它不像 GCM 那样给出一个公开的有限域多项式，但它同样撤掉了 nonce 提供的新鲜性结构。

### 6.2 三者都不能自动解决 replay

三种模式都能拒绝被篡改的 tuple，但都不会自动拒绝精确重放的旧 tuple。因为若攻击者提交的是一份已经合法的

$$
(N,A,C,T),
$$

那么密码层并没有理由把它判成非法。

因此 replay 防护需要上层额外确保：

$$
N \notin \mathcal{S}_{\text{seen}}
$$

或通过 sequence number、epoch、session transcript 来实现 freshness。

### 6.3 攻击者能推出什么；不能推出什么

在 nonce 正确使用时：

- GCM 攻击者看不到 \(H\) 与 \(E_K(J_0)\)
- CCM 攻击者看不到 raw CBC-MAC 链状态
- OCB 攻击者看不到 offset 常量族与 checksum 的秘密耦合结果

但在 misuse 时，攻击者并不需要恢复 \(K\) 才算赢。它只需要恢复某种足以生成有效 tuple 的中间对象，例如：

- GCM 的 \(H\) 与某个 nonce 的 tag mask
- CCM 的 repeated-nonce 下可复用的 keystream 和 raw tag 差分信息
- OCB 的 repeated-offset 轨道下可复用关系

这正是 mode attack 和“恢复 AES 主密钥”之间最容易混淆的地方。很多时候，攻击者从头到尾都没有逼近 \(K\)，但系统已经失守。

## 7. 走向 Vol.3：为什么 AEAD 仍不等于 misuse-safe

Vol.2 的三种模式都属于 `nonce-based AEAD`。它们的安全定理默认：

$$
N_i \neq N_j \qquad \text{for all encryption queries under one key.}
$$

因此它们真正提供的是：

> 在 nonce discipline 成立时，同时给出 confidentiality 与 integrity。

它们没有承诺的是：

> 即使 nonce discipline 失守，系统仍只发生温和退化。

GCM 告诉我们，reuse 可以把认证问题直接改写成关于 \(H\) 的代数方程。

CCM 告诉我们，reuse 即使不形成那么漂亮的公开多项式，也仍会同时撤掉 payload keystream 的新鲜性与 raw tag 封装。

OCB 告诉我们，reuse 会把本应一次性的 offset 轨道重放给攻击者。

所以 Vol.3 要回答的问题就自然出现了：

1. 能不能把“外部 nonce 直接决定一切”的结构换掉。
2. 能不能让 repeated nonce 的代价从“灾难性崩塌”降级为“确定性泄漏”。
3. 能不能在不要求完美 nonce discipline 的前提下，仍给出有意义的安全定义。

这正是 SIV 与 GCM-SIV 出场的地方。它们的重要性不在于“nonce 用错也完全没事”，而在于它们试图把失败模式从

$$
\text{two-time pad} + \text{tag algebra collapse}
$$

降级为

$$
\text{equality leakage} \quad \text{or similarly bounded degradation}.
$$

Vol.1 讲的是保密模式为什么挡不住主动攻击。Vol.2 讲的是认证方程如何切断这些主动攻击。Vol.3 则会继续问：如果 one-time 前提自己坏了，该怎样重新定义“还算安全”。

## References

- RFC 5116, [*An Interface and Algorithms for Authenticated Encryption*](https://www.rfc-editor.org/info/rfc5116)
- NIST SP 800-38D, [*Recommendation for Block Cipher Modes of Operation: Galois/Counter Mode (GCM) and GMAC*](https://csrc.nist.gov/pubs/sp/800/38/d/final)
- NIST SP 800-38C, [*Recommendation for Block Cipher Modes of Operation: The CCM Mode for Authentication and Confidentiality*](https://csrc.nist.gov/pubs/sp/800/38/c/upd1/final)
- RFC 7253, [*The OCB Authenticated-Encryption Algorithm*](https://www.rfc-editor.org/info/rfc7253)
- Phillip Rogaway, [*Authenticated-Encryption with Associated-Data*](https://www.cs.ucdavis.edu/~rogaway/papers/ad.html)
- Mihir Bellare and Chanathip Namprempre, [*Authenticated Encryption: Relations among Notions and Analysis of the Generic Composition Paradigm*](https://doi.org/10.1007/s00145-008-9026-x)

