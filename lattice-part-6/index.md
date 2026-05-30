# Lattice Part 6: Lattice Trapdoors and Signatures — GPV, Falcon, Dilithium


Reading path: Micciancio-Regev 的 survey / lecture-note 主线[^micciancio-regev]，再接 GPV[^gpv]、Lyubashevsky 的 trapdoor-free signature line[^lyu12]、Falcon[^falcon] 与 Dilithium[^dilithium]。这一篇只讨论结构，不写成 NIST 结果综述。

Part 4 和 Part 5 讲的是 hardness 如何支撑 encryption 与 KEM：SIS / LWE 给出难解关系，RLWE / MLWE / NTRU 给出更适合实现的结构。Part 6 反过来问另一个问题。假设已经有困难问题了，signer 到底靠什么持续生成“可验证而且足够短”的 witness？

这是签名线和加密线分叉的地方。加密只要求 honest party 能解码；签名要求 honest signer 主动制造一个受控分布的短向量，而且这个过程不能把 trapdoor 本身逐次泄露出去。于是短基、陪集、预像采样、Gaussian sampling、Fiat-Shamir with aborts 会在这一篇同时出现。

Working thesis: lattice signatures need more than encryption hardness. They need a mechanism for distribution-controlled short witness generation.

<!--more-->

## Reader State After Part 0-3

The only geometric tools assumed here are the ones already used earlier in the series:

- Part 1: a basis can be good or bad; Gram-Schmidt lengths measure how usable that basis is.
- Part 2: CVP asks for a short displacement from a target to a lattice or a coset; Babai gives a deterministic nearest-plane intuition.
- Part 3: SIS / ISIS ask for short integer witnesses satisfying public modular relations.

The signature setting combines these three objects. A verifier wants to check an ISIS-like relation:

$$
A\mathbf{z}=\mathbf{u}\pmod q,
$$

but the signer must produce such a short $\mathbf{z}$ repeatedly for many message-derived targets $\mathbf{u}=H(\mu)$. One short solution is not enough; the output distribution over many signatures is part of the security claim.

The guiding translation from Part 0-3 is:

$$
\text{coset-CVP with a hidden good basis} \quad\leadsto\quad \text{signature preimage sampling}.
$$

This is why the chapter keeps returning to short bases, cosets, and sampling distributions rather than only listing signature schemes.

## 为什么签名需要的不只是 Hardness

### Hardness Is Necessary But Not Sufficient

Part 4 中最典型的 LWE 关系是

$$
\mathbf{b}=A\mathbf{s}+\mathbf{e}\pmod q.
$$

这里的困难点在于：给你公开的 $A,\mathbf{b}$，想恢复短秘密 $\mathbf{s}$ 或把噪声从线性关系里剥出来是难的。这个困难性很适合做 encryption，因为合法接收方只需要借助额外信息完成 decoding。

签名不是这样。签名系统要求 signer 在每条消息上输出一个新的可验证对象，并且 verifier 能检查：

1. 它满足某个公开关系；
2. 它足够短；
3. 它的分布不会把 signer 的 hidden structure 逐次暴露出来。

所以“存在一个难问题”只是必要条件，不是完整接口。更准确地说，签名需要的是：

- public relation；
- private trapdoor or signing structure；
- controlled short-solution generation；
- distribution-hiding mechanism。

如果只有 hardness 而没有后两项，那么 signer 和 attacker 面对的是同一个难题，系统根本无法正常签名。

因此，签名接口需要的不只是一个难解关系。Signer 必须拥有 verifier 看不到的结构，才能稳定地生成受控短响应。

### Notation For The Signing Relation

本篇先固定一个抽象签名关系，后面再分别实例化为 GPV、Falcon 和 Dilithium。

- $q$ 是模数，所有公开线性关系先在 $\mathbb{Z}_q$ 或 $R_q$ 上检查。
- $A$ 是公开矩阵；在 module-lattice 语境下，$A$ 的条目属于 $R_q=\mathbb{Z}_q[x]/(x^n+1)$。
- $\mathbf{u}=H(\mu)$ 是由消息 $\mu$ 导出的 target，实际方案会加入 domain separation、salt 或 transcript binding。
- $\mathbf{z}$ 是签名响应或 witness。
- $\beta$ 是 verifier 接受的范数界。
- $\|\cdot\|$ 默认表示把模 $q$ 系数取 centered representative 后的 Euclidean norm 或方案指定范数；具体方案也可能使用 $\ell_\infty$ 型界检查。

抽象关系写成：

$$
R(A,\mathbf{u},\mathbf{z})=1\quad\Longleftrightarrow\quad A\mathbf{z}=\mathbf{u}\pmod q\ \text{and}\ \|\mathbf{z}\|\le\beta.
$$

这不是安全定义本身，只是 verifier 能检查的语法关系。安全性还要求 $\mathbf{z}$ 的分布不能泄露 signing structure。

### What A Signer Must Actually Produce

从 Part 3 的 ISIS 视角看，签名最自然的原型是：

给定公开矩阵 $A\in\mathbb{Z}_q^{n\times m}$，给定消息导出的 target $\mathbf{u}$，要求生成一个短向量 $\mathbf{z}$ 使得

$$
A\mathbf{z}=\mathbf{u}\pmod q,\qquad \|\mathbf{z}\|\le \beta.
$$

验证者只看公开关系和长度界：

$$
\textsf{Verify}(A,\mathbf{u},\mathbf{z})=1
\quad\Longleftrightarrow\quad
A\mathbf{z}=\mathbf{u}\pmod q\ \text{and}\ \|\mathbf{z}\|\le \beta.
$$

这时 signer 要输出的不是“一个密文的逆像”，而是某个公开陪集里的短代表元。这个表述会直接把我们带回 Part 2 的 CVP / coset 语言。

> Signature-specific constraint: lattice signatures are not about hiding a secret relation once. They are about repeatedly sampling short witnesses for public relations without leaking the hidden sampling structure.

## 陷门、短基与陪集

### 从 Part 2 的陪集语言重新看签名

对公开矩阵 $A$，定义对应的 $q$-ary lattice

$$
\Lambda_q^\perp(A)=\{\mathbf{x}\in\mathbb{Z}^m: A\mathbf{x}=0\pmod q\}.
$$

再定义非齐次陪集

$$
\Lambda_q^{\mathbf{u}}(A)=\{\mathbf{x}\in\mathbb{Z}^m: A\mathbf{x}=\mathbf{u}\pmod q\}.
$$

如果 $\mathbf{x}_0$ 是一个特解，那么

$$
\Lambda_q^{\mathbf{u}}(A)=\mathbf{x}_0+\Lambda_q^\perp(A).
$$

这和 Part 3 的 ISIS、Part 2 的 coset 语言是同一件事：所有合法签名都落在一个公开陪集里，而 signer 需要的是这个陪集中的短向量。

The coset identity is worth spelling out because it is the exact bridge from Part 2. If $\mathbf{x}_0$ is one solution and $\mathbf{v}\in\Lambda_q^\perp(A)$, then

$$
A(\mathbf{x}_0+\mathbf{v})=A\mathbf{x}_0+A\mathbf{v}\equiv \mathbf{u}\pmod q.
$$

Conversely, if $\mathbf{x}$ and $\mathbf{x}_0$ both solve the same target equation, then

$$
A(\mathbf{x}-\mathbf{x}_0)\equiv \mathbf{0}\pmod q,
$$

so $\mathbf{x}-\mathbf{x}_0$ lies in the kernel lattice. Thus the solution set is not an arbitrary cloud; it is one translated copy of the kernel lattice.

于是签名问题可以写成

$$
\text{sample a short point from } \mathbf{x}_0+\Lambda_q^\perp(A).
$$

这就是 Part 2 的最近向量直觉进入现代构造的地方。区别在于这里不能满足于“找一个近的点”，还必须控制输出分布。

### Short Basis As Hidden Structure

为什么短基是 trapdoor？

因为对 outsider 来说，$\Lambda_q^\perp(A)$ 只是一个高维格；对 trapdoor holder 来说，它还带着一组“好基” $T=(\mathbf{t}_1,\ldots,\mathbf{t}_m)$，其 Gram-Schmidt 长度比较小：

$$
\|\widetilde{T}\|=\max_i \|\widetilde{\mathbf{t}}_i\|
$$

处在可控范围内。

这意味着几件事：

1. 持有者可以高效近似解 coset-CVP；
2. 持有者可以把一个特解逐步投影到较短的代表元；
3. 持有者可以围绕该 coset 做 randomized sampling，而不是只做 deterministic decoding。

这正是 Part 1 的 short basis intuition 在密码构造中的落点。LLL 或 BKZ 给我们的，是“某个格可能存在较短基”的算法视角；trapdoor constructions 给我们的，是“scheme designer 先天植入一个只有 signer 知道的短基”。

### 为什么 Babai 还不够

只从几何上说，持有好基后可以做类似 Babai nearest plane 的事情：把目标点逐层投影，得到某个比较近的格点。这解释了为什么 trapdoor holder 比 outsider 强。

但签名不能停在这里。

如果 signer 每次都用几乎 deterministic 的 decoding 方式，从陪集里选取“同一类”近点，那么长期输出会带上可识别的 basis fingerprint。攻击者未必一次恢复 trapdoor，但可以逐步收集分布偏差。

用符号写，若每条消息都输出

$$
\mathbf{z}_i=\operatorname{NearestPlane}_T(H(\mu_i)),
$$

那么 $\mathbf{z}_i$ 的统计形状会受到 hidden basis $T$ 的影响。Verifier 只检查短和正确，但 attacker 可以观察很多 $\mathbf{z}_i$。因此签名需要的不只是“能找到短点”，而是“以不暴露 $T$ 的方式生成短点”。

所以签名需要的是：

$$
\text{short solution generation}
\quad\Longrightarrow\quad
\text{distribution-controlled short solution generation}.
$$

这就是 GPV 要解决的问题。

本章后面反复使用的桥就是：short basis 提供 preimage sampling 能力，但签名安全还要求该 sampling 的输出分布可控。

## GPV Sampling 作为桥梁

### From ISIS-Style Relations To Preimage Sampling

Gentry, Peikert, Vaikuntanathan 的核心贡献之一，就是把“有 trapdoor 的短基”变成“可采样的预像接口”[^gpv]。

仍然从关系

$$
A\mathbf{z}=\mathbf{u}\pmod q
$$

出发。若已知 $\Lambda_q^\perp(A)$ 的短基 $T$，则可以不只找一个短向量，而是从整个陪集

$$
\Lambda_q^{\mathbf{u}}(A)=\mathbf{x}_0+\Lambda_q^\perp(A)
$$

中按某种平滑分布采样。

这时签名接口已经非常接近最终形式：

1. hash 消息得到 target $\mathbf{u}=H(m)$；
2. 用 trapdoor 在 $\Lambda_q^{\mathbf{u}}(A)$ 上采样短向量 $\mathbf{z}$；
3. 输出 $\mathbf{z}$ 作为 signature；
4. verifier 检查 $A\mathbf{z}=H(m)\pmod q$ 与长度界。

这条链路把 Part 3 的 ISIS 从“攻击或 hardness 视角”转成了“签名或 constructive 视角”。

GPV sampling 是从 hardness relation 走向 signing interface 的标准桥。

For a Part 2 reader, the change from Babai to GPV can be summarized as follows. Babai gives an algorithmic point estimate:

$$
\mathbf{z}\approx \operatorname{argmin}_{\mathbf{x}\in\Lambda_q^{\mathbf{u}}(A)}\|\mathbf{x}\|.
$$

GPV instead asks for a randomized draw from the whole coset, biased toward short vectors:

$$
\mathbf{z}\leftarrow D_{\Lambda_q^{\mathbf{u}}(A),s}.
$$

The first line is a decoding algorithm. The second line is a signing distribution.

### 为什么 Gaussian Sampling 会出现

在 GPV 里，目标分布通常写成某个离散 Gaussian：

$$
D_{\Lambda+\mathbf{c},s}(\mathbf{x})
\propto
\exp\left(-\pi\frac{\|\mathbf{x}\|^2}{s^2}\right),
\qquad \mathbf{x}\in \Lambda+\mathbf{c}.
$$

更完整地写，令 $\rho_s(\mathbf{x})=\exp(-\pi\|\mathbf{x}\|^2/s^2)$，则

$$
D_{\Lambda+\mathbf{c},s}(\mathbf{x})=\frac{\rho_s(\mathbf{x})}{\rho_s(\Lambda+\mathbf{c})},\qquad \rho_s(\Lambda+\mathbf{c})=\sum_{\mathbf{y}\in\Lambda+\mathbf{c}}\rho_s(\mathbf{y}).
$$

这里的意义不是概率论装饰，而是它同时解决了三件事：

1. 长度控制：大向量概率指数下降；
2. 平滑性：输出不会黏在几个特殊近点上；
3. trapdoor hiding：当 $s$ 足够大时，分布对具体 trapdoor basis 的依赖变弱。

这就是本章需要的 discrete Gaussian sampling discussion：它不是装饰性的概率论，而是长度控制、分布隐藏与签名可证明性的共同接口。

GPV 视角下典型的参数条件长成

$$
s \gtrsim \|\widetilde{T}\|\cdot \omega(\sqrt{\log m}),
$$

也就是离散 Gaussian 的宽度要压过 trapdoor basis 的 Gram-Schmidt 长度。直觉上，这是为了让采样过程“看到的是整个陪集的平滑轮廓”，而不是被某几个 basis direction 卡住。

这和 Part 0 里提到的 smoothing 其实是一条线：一旦 Gaussian 宽度越过某个阈值，格的局部毛刺会被抹平，很多统计性质会开始变得好用[^micciancio-regev].

### Generic GPV Signature Skeleton

把这些对象压缩成一个最小签名原型，可以写成：

> Public key: a matrix $A$.
>
> Secret key: a short basis $T$ of $\Lambda_q^\perp(A)$.
>
> Message digest: $\mathbf{u}=H(m)$.
>
> Signature: sample $\mathbf{z}\leftarrow D_{\Lambda_q^{\mathbf{u}}(A),s}$ using $T$.
>
> Verification: check $A\mathbf{z}=H(m)\pmod q$ and $\|\mathbf{z}\|\le \beta$.

这已经把“hardness -> signing”桥梁写完整了：

$$
\text{SIS-style short relation}
\longrightarrow
\text{trapdoor short basis}
\longrightarrow
\text{preimage sampling on a coset}
\longrightarrow
\text{signature distribution}.
$$

但 GPV 也不是工程终点。Generic trapdoor signatures 往往同时承担三种成本：

1. unstructured matrix public key 往往偏大；
2. trapdoor basis generation 与存储开销不小；
3. 高质量 discrete Gaussian sampling 本身就是实现难点。

所以 GPV 更像一个 architecture-level answer。它给出“签名为什么可能”的抽象接口，也说明“短基与采样怎样接上 hardness”；但部署方案还必须把这条线压到更结构化、更紧凑、或更易实现的对象上。

之后的 Falcon 与 Dilithium，可以看成沿着这座桥分叉的两条工程路线：

- Falcon 保留 “short basis + Gaussian sampling” 这条线，并把它极限压缩到 NTRU lattice 上；
- Dilithium 则尽量绕开高精度 trapdoor Gaussian sampling，改走 Fiat-Shamir with aborts 这条更整数化的路径。

所以如果要用一句话概括后文：GPV-style sampling bridges hardness assumptions and signing, while Falcon and Dilithium implement that bridge in sharply different ways.

## Falcon：把 GPV 压进 NTRU Lattice

### NTRU Trapdoor And Signing Relation

Part 5 已经说过，NTRU 不是 RLWE / MLWE 的别名，但它确实提供了一个自然的 structured lattice 背景。Falcon 使用的就是这个位置。

Falcon 保留 trapdoor / short-basis / Gaussian sampling 这条线，但把它压缩到一个高度结构化的 NTRU lattice 上。

在 NTRU setting 中，公开键通常写成

$$
h \equiv g f^{-1}\pmod q,
$$

其中 $f,g$ 是短多项式。与此同时，secret side 还持有一组满足 NTRU equation 的短对象

$$
fG-gF=q.
$$

这个关系保证可以构造出一个对应 NTRU lattice 的短基。把公开关系写成线性映射的形式，可以看成

$$
(u,v)\mapsto u+vh \pmod q.
$$

于是 kernel lattice 是

$$
\Lambda_h=\{(u,v)\in R^2: u+vh\equiv 0 \pmod q\},
$$

并且由

$$
g-fh\equiv 0\pmod q,\qquad G-Fh\equiv 0\pmod q
$$

可以看出 $(g,-f)$ 与 $(G,-F)$ 正好给出这类 kernel relation 的短向量来源。也就是说，Falcon secret key 的实质并不只是“知道某几个短多项式”，而是“知道这张公开映射背后的一组结构化短基”。

The two congruences come directly from the public key equation and the NTRU equation. First,

$$
g-fh\equiv g-f(gf^{-1})\equiv 0\pmod q.
$$

Second, $fG-gF=q$ gives $fG\equiv gF\pmod q$; multiplying by $f^{-1}$ gives

$$
G-Fh\equiv 0\pmod q.
$$

Thus the secret polynomials are not just short. They certify short kernel vectors for the public map $(u,v)\mapsto u+vh$.

而签名要做的是：对消息导出的 target $c$，找到短对偶

$$
(s_1,s_2)\in R^2,\qquad s_1+s_2h\equiv c\pmod q.
$$

这和 GPV 的 coset-preimage 语言完全一致。差别在于：

- public relation 不再是一般矩阵 $A$，而是 NTRU-style ring map；
- trapdoor 不再表现为一般 $q$-ary lattice 的短基，而是 NTRU lattice 的结构化短基；
- 整个算法可以借助 FFT / NTT-like ring arithmetic 压到非常小的签名尺寸。

### Fast Fourier Sampling

Falcon 的关键不是“它也采样 Gaussian”，而是“它必须把 Gaussian sampling 做得极准，而且做在 NTRU lattice 的递归结构上”[^falcon]。

核心思路可以概括为：

1. 用 secret basis 构造目标 lattice 的 Gram matrix；
2. 在 Fourier domain 里做分解，例如 LDL 型递归分解；
3. 对每层的一维或低维条件分布做离散 Gaussian sampling；
4. 再把结果合成为最终的短 preimage。

如果用一句更几何的话来描述，Falcon 做的是：

$$
\text{sample from a high-dimensional Gaussian on a coset}
\quad\text{by recursively sampling conditionals in a basis-adapted coordinate system.}
$$

这件事结构紧凑，但也说明它为什么实现敏感。算法不是普通的“模整数加减乘”，而是对数值精度、舍入方式、递归采样误差都有严格要求。

### Why Falcon Is Elegant And Fragile

Falcon 的优点很明显：

- 签名短；
- 公钥也比较紧凑；
- 继承了 GPV / Gaussian sampling 这条理论线的整洁结构。

但它的脆弱点也很集中：

1. 浮点实现面。Fast Fourier sampling 常依赖浮点近似、舍入和数值稳定性；精度管理失手会直接改变输出分布。
2. 定时与控制流。若采样路径、拒绝次数或数值修正与秘密相关，就可能形成 side-channel。
3. 精确分布问题。Falcon 的安全故事不只是“向量要短”，而是“输出要足够接近目标 Gaussian”。有偏 sampler 会动到根上。
4. 实现可审计性。相比纯整数路径，浮点 Gaussian sampler 更难给出强直觉和强工程保证。

所以 Falcon 给人的正确印象不该只是“compact signatures”。更准确地说：

> Falcon is a structured GPV descendant whose main engineering problem is distribution fidelity under tight numerical control.

## Dilithium：绕开 Trapdoor Gaussian 的另一条线

### Why Dilithium Does Not Start From GPV Sampling

Dilithium 不是把 Falcon 的 sampler 换一组参数。它使用另一条构造路线：

- 继续保留 Part 5 的 module-lattice arithmetic；
- 继续基于短秘密与公开线性关系；
- 但尽量避免高精度 trapdoor Gaussian sampling；
- 改为 identification-to-signature 的 Fiat-Shamir with aborts 路线[^lyu12][^dilithium].

Dilithium 不把 trapdoor Gaussian 当成核心接口，而是把 module-lattice public relation、short masking、Fiat-Shamir with aborts 和 rejection discipline 组合成一个更整数化的 signing construction。

从 public key 关系开始，Dilithium 的骨架更像

$$
\mathbf{t}=A\mathbf{s}_1+\mathbf{s}_2,
$$

其中

$$
A\in R_q^{k\times \ell},
\qquad
\mathbf{s}_1\in R^\ell,\ \mathbf{s}_2\in R^k
$$

都是小元素。

这里你已经能看出它与 Part 5 的连续性：同样是 module lattice，同样是环上矩阵乘法，同样依赖 fast polynomial arithmetic。变化不在“代数平台”，而在“签名接口”。

### Fiat-Shamir With Aborts

先看一个理想化的 identification skeleton。signer 先采样一个短随机掩码 $\mathbf{y}$，然后计算

$$
\mathbf{w}=A\mathbf{y}.
$$

挑战由消息和 commitment 导出：

$$
c = H(m,\mathbf{w}_1),
$$

其中 $\mathbf{w}_1$ 表示只暴露某种经 rounding / decomposition 处理后的高位信息。随后 signer 返回

$$
\mathbf{z}=\mathbf{y}+c\mathbf{s}_1.
$$

如果只做到这里，问题立刻出现：$\mathbf{z}$ 的分布会围绕 $c\mathbf{s}_1$ 发生偏移。只要攻击者收集足够多的 transcript，就可能从分布偏差里读出秘密信息。

所以 Lyubashevsky line 的核心不是单纯的 Fiat-Shamir，而是

$$
\text{Fiat-Shamir} + \text{rejection / abort discipline}.
$$

这就是 Fiat-Shamir with aborts 的重点：abort 不是异常分支，而是主动修正响应分布的协议机制。

也就是：当 $\mathbf{z}$ 或若干关联量越界时，signer 直接丢弃本轮并重采样。目标是让“被接受的 $\mathbf{z}$”看起来更像从一个与秘密弱相关的目标分布中抽到，而不是赤裸裸地暴露 $\mathbf{y}+c\mathbf{s}_1$ 的中心漂移。

最小化地写，若 $\mathbf{y}$ 从盒子 $[-\gamma_1,\gamma_1]$ 中采样，而 $\mathbf{z}=\mathbf{y}+c\mathbf{s}_1$，则签名方只在

$$
\|\mathbf{z}\|_\infty < \gamma_1-\beta
$$

等条件成立时接受本轮。这个余量 $\beta$ 用来吸收 $c\mathbf{s}_1$ 的偏移，使被接受响应的边界效应不明显依赖秘密。真实 Dilithium 还会对 low bits、hint 相关量和 challenge weight 做更多约束，但核心逻辑仍然是用 rejection 把 transcript 分布从秘密中心偏移中拉回来。

这正是 Dilithium 与 Falcon 的主要分叉：

- Falcon 靠 trapdoor Gaussian 直接采样“对的分布”；
- Dilithium 靠 simpler sampling + abort 机制把“原本有偏的分布”修正到可接受范围。

### What The Verifier Is Actually Checking

理想化地忽略 rounding 和 hint 细节时，验证关系可以写成

$$
\begin{aligned}
A\mathbf{z}-c\mathbf{t}
&=
A(\mathbf{y}+c\mathbf{s}_1)-c(A\mathbf{s}_1+\mathbf{s}_2) \\
&=
A\mathbf{y}-c\mathbf{s}_2.
\end{aligned}
$$

也就是说，verifier 看到的是：

1. $\mathbf{z}$ 足够短；
2. 用公开键 $\mathbf{t}$ 和挑战 $c$ 回推出的量，与 signer 当初提交的 high bits 一致；
3. 这些对象共同说明 signer 确实知道短秘密，而不是临时伪造了一个响应。

The cancellation should be read in the same way as the encryption cancellation in Part 4 and Part 5. The large public term $A c\mathbf{s}_1$ cancels against $cA\mathbf{s}_1$ inside $c\mathbf{t}$. What remains is the original commitment $A\mathbf{y}$ plus the small correction $-c\mathbf{s}_2$. The verifier cannot see $\mathbf{y}$ directly, but it can check that the reconstructed high bits agree with the challenge transcript.

真实的 Dilithium 还要处理：

- $\mathbf{t}$ 的高低位分解；
- challenge space 的离散化；
- hints 用于帮助 verifier 重建 rounding 后的高位；
- 对 $\mathbf{z}$、低位噪声项及相关对象的范数界检查。

如果把这一步再写得更显式一点，可以把 public key decomposition 记作

$$
\mathbf{t}=\mathbf{t}_1\cdot 2^d+\mathbf{t}_0.
$$

那么 verifier 实际不会恢复 signer 内部看到的全部 $\mathbf{w}=A\mathbf{y}$，而是只恢复足以重建 challenge 的那部分高位信息。于是 hint 的角色就清楚了：

- 它不是额外泄露一个新秘密；
- 它是帮助 verifier 在不知道 $\mathbf{s}_2,\mathbf{t}_0$ 全貌的情况下，对 rounding 后的 high bits 做一致性恢复；
- 它让 scheme 可以把签名大小、公开键大小与验证复杂度一起压住。

这也解释了为什么 Dilithium 的实现不能把 rounding / decomposition 当成无关紧要的编码细节。它们属于协议语义本身。

核心抽象并不复杂：

$$
\text{short random mask}
\longrightarrow
\text{public commitment}
\longrightarrow
\text{hash challenge}
\longrightarrow
\text{short response with aborts}.
$$

这不是 trapdoor preimage sampling，而是更像一个 lattice-flavored Schnorr / Fiat-Shamir 族，只不过为了压住分布泄露，不得不把 abort mechanism 做到协议核心里。

## 两条签名路线的并列对照

到这里可以把 Falcon 与 Dilithium 放在同一张图里看：

| Scheme | Public relation | Hidden structure | Sampling model | Main implementation pain |
| --- | --- | --- | --- | --- |
| GPV-style trapdoor signatures | $A\mathbf{z}=\mathbf{u}\pmod q$ | short basis of $\Lambda_q^\perp(A)$ | discrete Gaussian on a coset | basis quality, Gaussian sampler correctness |
| Falcon | $s_1+s_2h\equiv c\pmod q$ | NTRU trapdoor basis | fast Fourier Gaussian sampling | floating point, constant-time, numerical fidelity |
| Dilithium | $\mathbf{t}=A\mathbf{s}_1+\mathbf{s}_2$ plus FS transcript | small module-lattice secret | uniform-ish masking plus rejection / aborts | abort logic, hint handling, norm checks, constant-time control flow |

所以两者的 tradeoff 不能概括成一句“Falcon 小，Dilithium 好实现”就结束。更准确的分界应该是：

- Falcon 继承了更直接的 trapdoor-sampling 理论线；
- Dilithium 继承了更容易整数化、工程化的 FS-with-aborts 线；
- 两者都建立在 lattice short relation 上，但“如何生成安全分布的响应”完全不同。

## 为什么签名实现更敏感

### Falcon: Sampling, Timing, Floating Point

Falcon 的危险不只在 side-channel 这个词本身，而在具体接口：

1. FFT / floating-point pipeline 是否数据相关；
2. recursive Gaussian sampler 的路径是否泄露秘密相关分支；
3. 舍入误差是否使输出分布系统性偏离理论目标；
4. rejection 或 resampling 次数是否可被观测；
5. 压缩编码是否引入额外的 timing surface。

这些风险之所以严重，是因为 Falcon 把安全性大量押在“sampled signature 的统计形状”上。实现若把这个形状扭坏了，问题就不是常规的常数项退化，而是可能让攻击者直接看到 trapdoor structure 的影子。

### Dilithium: Abort Logic, Hints, Constant-Time Discipline

Dilithium 避开了浮点 Gaussian，但不等于没有分布风险。它的脆弱点更多集中在协议控制面：

1. abort condition 是否 constant-time；
2. hint generation 和 use-hint 逻辑是否严格匹配规范；
3. decomposition / rounding 的边界是否实现一致；
4. norm checks 是否存在 early exit 或数据相关访问；
5. randomness domain separation 是否清晰，是否把不同轮次或不同消息的掩码隔离开。

从工程上看，Dilithium 的优势是它更接近“整数化的模块算术 + 明确的界检查”。但这并不意味着可以把 rejection logic 当成普通异常分支。对它来说，abort 不是边角料，而是分布修正器本身。

### 这和 Part 5 的 KEM 视角有什么不同

这一点值得单独强调。Part 5 里谈 Kyber 时，我们也关心 constant-time、NTT、噪声分布、压缩接口。但签名比 KEM 更敏感，原因在于：

- signer 会重复发布与同一把秘密钥匙相关的 many-message transcript；
- 攻击者直接观察到签名分布，而不只是解密失败概率；
- sampling / rejection 的每一点偏差，都有可能跨样本累计成 secret leakage。

所以在 lattice signatures 里，“implementation detail” 往往不是边角细节。它们是 scheme security argument 的一部分。

这些就是本章关心的 implementation sensitivity and deployment tradeoffs：Falcon 把风险集中到高精度采样与数值控制，Dilithium 则把风险集中到 rejection logic、hint handling 与 constant-time discipline。

## 总结

Part 0 到 Part 3 准备了四件东西：

1. 格与短向量语言；
2. short basis 与 Gram-Schmidt 质量；
3. coset / CVP / nearest-plane 直觉；
4. SIS / ISIS 这类公开短关系。

Part 4 和 Part 5 又把这条线推到了现代 encryption / KEM：LWE、RLWE、MLWE、NTRU、Kyber。

Part 6 则把地图再补完整一块：

- 如果目标是 encryption，那么 hardness + decoding 结构已经足够重要；
- 如果目标是 signatures，那么还必须回答“谁来生成短 witness、怎么生成、输出分布为何不泄露 hidden structure”；
- GPV 给出了 trapdoor preimage sampling 这条统一桥；
- Falcon 站在这座桥的 structured Gaussian 端；
- Dilithium 站在 identification + Fiat-Shamir with aborts 的工程化端。

所以这一篇要读懂的，不是哪一个方案进入标准化，而是：

$$
\text{short basis}
\longrightarrow
\text{trapdoor or signing structure}
\longrightarrow
\text{distribution-controlled response generation}
\longrightarrow
\text{signature security}.
$$

如果把这个链条看清，Falcon 与 Dilithium 的差异就不再只是方案选择，而是 lattice-based signatures 本身的结构分叉。

## References

Suggested reading order:

1. Micciancio-Regev first, for the q-ary lattice, short basis, and smoothing vocabulary used throughout the article.
2. GPV next, for the trapdoor/preimage-sampling interface and the discrete Gaussian distributional requirement.
3. Lyubashevsky 2012, for the Fiat-Shamir-with-aborts alternative to trapdoor Gaussian sampling.
4. Falcon and Dilithium primary specifications last, comparing the two implementation surfaces after the abstract signing relation is clear.

[^micciancio-regev]: Daniele Micciancio and Oded Regev, *Lattice-based Cryptography*, in *Post-Quantum Cryptography*.
[^gpv]: Craig Gentry, Chris Peikert, and Vinod Vaikuntanathan (2008), *Trapdoors for Hard Lattices and New Cryptographic Constructions*.
[^lyu12]: Vadim Lyubashevsky (2012), *Lattice Signatures without Trapdoors*.
[^falcon]: Pierre-Alain Fouque, Vadim Lyubashevsky, Gregor Seiler, et al. (2018), *Falcon: Fast-Fourier Lattice-Based Compact Signatures over NTRU*.
[^dilithium]: Léo Ducas, Eike Kiltz, Tancrède Lepoint, Vadim Lyubashevsky, Peter Schwabe, Gregor Seiler, and Damien Stehlé (2018), *CRYSTALS-Dilithium: A Lattice-Based Digital Signature Scheme*.

