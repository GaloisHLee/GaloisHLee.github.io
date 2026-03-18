# 零知识证明的最小理论框架


> Reading: GMR 1989, Goldreich, and the standard zero-knowledge definition chain.

开篇先不讲洞穴故事，也不讲“验证者虽然信了但什么都没学到”这种直觉口号。后面整条 ZK 系列都会反复调用同一批对象: relation、language、instance、witness、prover、verifier、transcript、view、simulator、extractor。第一篇的任务，只是把这些对象之间的接口一次性钉死。

零知识证明真正容易混掉的，不是某个协议细节，而是哪些性质分别由 completeness、soundness、zero knowledge、knowledge soundness 约束；哪些结论只对 honest verifier 成立；哪些地方已经从 proof 退到了 argument。只要这些边界不先写清楚，后面看到 Sigma、Fiat-Shamir、SNARK、STARK 时就会不断把不同层次的结论混在一起。

本文给出一个最小理论框架: 从 $R(x, w)$ 定义语言，到交互证明的接受事件，再到 verifier view 与 simulator 的 indistinguishability，最后补上 argument/proof 与 knowledge soundness 的分界。它不是一篇历史综述，而是一份后续文章默认继承的接口说明书。[^gmr] [^goldreich]

<!--more-->

## First Principles: Relation, Language, Instance, Witness

零知识证明不是“证明一个命题”这么宽泛。它的最小对象是一个二元关系

$$
R \subseteq \mathcal{X} \times \mathcal{W},
$$

其中 $x \in \mathcal{X}$ 是 instance，$w \in \mathcal{W}$ 是 witness。由关系 $R$ 诱导出的语言是

$$
L_R = \{x \in \mathcal{X} \mid \exists w \in \mathcal{W},\ R(x, w) = 1 \}.
$$

这一步必须先固定，因为后面所有性质都围绕下面这个结构展开:

- prover 试图说服 verifier 某个 $x$ 属于 $L_R$
- witness $w$ 是 prover 的额外信息，使得 $R(x, w) = 1$
- verifier 通常不直接看到 $w$

> Formal Definition.
> 一个零知识协议讨论的对象不是“事实”本身，而是 relation membership: 给定公开实例 $x$，证明者知道某个见证 $w$，使得 $R(x, w)=1$。

例如，离散对数关系可以写成

$$
R_{\mathrm{DL}}((G, g, y), w) = 1 \iff y = g^w.
$$

这里公开实例是 $(G, g, y)$，见证是指数 $w$。这已经足够支撑后面 Schnorr 风格 proof of knowledge 的讨论，但本文先停在接口层，不展开具体协议。

### Transcript Versus Verifier View

进入交互之后，至少要区分两个对象:

1. transcript: 双方发送的消息序列
2. verifier view: verifier 在执行中“看到”的全部信息

如果协议轮数为 $t$，那么 transcript 可写成

$$
\tau = (m_1, m_2, \dots, m_t).
$$

但 verifier view 通常比 transcript 更大。对于一个 verifier $V^\*$，它的 view 可以写成

$$
\mathrm{View}_{V^\*}^{P}(x, w; z) = (x, z, r, m_1, \dots, m_t),
$$

其中 $r$ 是 verifier 的随机性，$z$ 是 auxiliary input。后面 zero knowledge 定义里，simulator 要模拟的是这个 view 的分布，而不只是消息串本身。

这是第一条必须守住的边界: transcript 是外层消息对象，view 是分布对象；只讨论 transcript 常常不足以表述 zero knowledge。

## Interactive Proof 的最小定义

现在引入两台交互机器: prover $P$ 与 verifier $V$。给定 instance $x$，若 prover 还持有见证 $w$，则运行记为

$$
\langle P(w), V \rangle(x).
$$

verifier 最终输出 accept 或 reject。交互证明的第一层只有两个性质: completeness 和 soundness。

### Completeness

当 $x \in L_R$ 且 prover 确实持有满足 $R(x, w)=1$ 的 witness 时，诚实双方运行应当接受。形式上可写成

$$
\Pr[\langle P(w), V \rangle(x) = 1] \ge 1 - \varepsilon_c(\lambda),
$$

其中 $\varepsilon_c(\lambda)$ 通常是 negligible，很多协议甚至要求 perfect completeness，即接受概率恰好为 $1$。

completeness 只回答一个问题: 真命题加真见证时，系统会不会把正确输入误杀。它不涉及“学到了什么”，也不涉及“伪造者能做到什么”。

### Soundness

soundness 约束的是假命题。若 $x \notin L_R$，则任意作弊 prover $P^\*$ 都不应高概率骗过 verifier:

$$
\Pr[\langle P^\*, V \rangle(x) = 1] \le \varepsilon_s(\lambda).
$$

这里最重要的不是公式，而是量词顺序:

- 先固定一个假的 instance $x \notin L_R$
- 再对任意作弊 prover $P^\*$ 取上界
- 最后要求接受概率足够小

很多口语化解释会把 soundness 说成“证明者不会撒谎”。这不准确。soundness 只是在说: 对不在语言里的实例，想让 verifier 接受会很难。

### Proof Versus Argument

这一层马上出现第一条硬边界。

如果 soundness 对任意甚至无限算力 prover 都成立，我们通常称其为 proof。若 soundness 只对 probabilistic polynomial-time cheating prover 成立，那么它是 argument。

所以：

- proof: 信息论或无条件地排除了伪造接受
- argument: 只在计算边界内排除了伪造接受

现代零知识系统里，大量对象其实是 argument，不是 proof。原因很简单: 一旦 soundness 依赖离散对数、pairing hardness、Fiat-Shamir random oracle、low-degree testing 假设之类的计算前提，它就已经不再是对无限算力 prover 的陈述。

这一点必须在第一篇就写清楚，否则后面很容易把“系统很安全”误读成“这是 proof，而不是 argument”。

## 从 Verifier View 到 Zero Knowledge

到目前为止，我们只约束了“真的要能过，假的不该过”。这还不是零知识。zero knowledge 讨论的是: 在接受性之外，verifier 究竟学到了什么。

最常见的误解是把它说成“验证者没有获得任何信息”。这句话几乎总是错误的，因为 verifier 至少学到了一个事实:

$$
x \in L_R.
$$

zero knowledge 的正确说法不是“没有信息”，而是“除了 statement validity 及其可从中高效推导出的内容外，没有获得额外知识”。而这句话若不通过 simulator 形式化，实际上仍然是空的。

### Honest Verifier 的 View

若先只看 honest verifier，定义会简单一些。诚实 verifier $V$ 按协议规定产生随机性和挑战，因而其 view 分布是确定的：

$$
\mathrm{View}_{V}^{P}(x, w) = (x, r, \tau).
$$

如果存在一个高效算法 $\mathrm{Sim}$，仅从公开 instance $x$ 就能生成一个分布，与真实交互中的 verifier view 近似不可区分，那么我们说协议是 zero knowledge 的某种形式。

形式上：

$$
\mathrm{Sim}(x) \approx \mathrm{View}_{V}^{P}(x, w).
$$

这里的 $\approx$ 需要具体化。

### Perfect, Statistical, Computational

根据“近似”的强弱不同，zero knowledge 分成三层:

1. perfect zero knowledge: 两个分布完全相同
2. statistical zero knowledge: 两个分布的统计距离可忽略
3. computational zero knowledge: 任意高效 distinguisher 都无法有效区分

强弱关系是

$$
\text{perfect} \Rightarrow \text{statistical} \Rightarrow \text{computational}.
$$

如果一个协议只对诚实 verifier 满足这个条件，通常称为 HVZK, honest-verifier zero knowledge。若对任意多项式时间恶意 verifier $V^\*$ 都存在 simulator，则才是更一般的 ZK:

$$
\forall V^\* \in \mathrm{PPT},\ \exists \mathrm{Sim}_{V^\*}\ \text{s.t.}\ \mathrm{Sim}_{V^\*}(x, z) \approx \mathrm{View}_{V^\*}^{P}(x, w; z).
$$

这比 HVZK 强得多，因为恶意 verifier 可以偏离协议、嵌入辅助输入、按自选策略生成挑战。

> Key Observation.
> zero knowledge 真正约束的对象是 real view distribution 和 simulated view distribution 之间的关系，而不是某句“泄露了多少”的直觉描述。

## 为什么直觉不够: 一条最小定义链

现在把前面的对象串起来。

给定 relation $R$，我们先得到语言

$$
L_R = \{x \mid \exists w,\ R(x,w)=1\}.
$$

随后定义一个交互协议 $(P, V)$，它必须先满足：

1. completeness: 真实例加真 witness 能让 verifier 接受
2. soundness or argument soundness: 假实例不能被高概率伪造为真

但如果要称其为 zero knowledge，还必须再补一条分布要求：

3. 对某类 verifier，真实 view 与 simulator 输出不可区分

用更压缩的写法，这条链是

$$
R(x,w)=1
\Longrightarrow x \in L_R
\Longrightarrow \Pr[\langle P(w),V\rangle(x)=1]\ \text{高}
$$

以及

$$
x \notin L_R
\Longrightarrow \Pr[\langle P^\*,V\rangle(x)=1]\ \text{低},
$$

同时

$$
\mathrm{View}_{V^\*}^{P}(x,w;z) \approx \mathrm{Sim}_{V^\*}(x,z).
$$

这三条不是一个性质的不同说法，而是三个不同维度:

- completeness 关心正确性
- soundness 关心伪造接受
- zero knowledge 关心 view distribution

如果不把 verifier view 写出来，“验证者没有获得额外信息”这句话其实没有参照对象。额外信息相对于什么定义？是 transcript，还是 verifier 的内部随机性，还是包含 auxiliary input 的完整 execution state？这些不写清楚，zero knowledge 就只剩修辞。

### HVZK 为什么不等于 Full ZK

HVZK 的 simulator 只需要重现诚实 verifier 的 view。很多经典协议在这一步并不难，因为诚实 verifier 的挑战或随机性是按固定分布来的，simulator 可以“先采 challenge，再倒推出 transcript”。

但一旦 verifier 恶意化，问题就变了:

- verifier 可以让消息依赖辅助输入 $z$
- verifier 可以选择异常挑战分布
- verifier 可以把协议嵌进更大的环境中联合执行

所以 HVZK 常常只是第一步，不是最终目标。后面看到 Fiat-Shamir 时，这个边界尤其关键: 某些协议的 HVZK 很自然，但转到非交互或更强环境之后，定义和证明义务都会变。

### 参数边界也属于定义的一部分

还需要避免一个常见偷懒动作: 只写“概率很小”而不写小到什么程度。

密码学里通常要求错误概率是 negligible function，即对任意多项式 $p(\lambda)$，在足够大安全参数 $\lambda$ 下都有

$$
\mu(\lambda) < \frac{1}{p(\lambda)}.
$$

不写 negligible，只说“很难”或“几乎不可能”，在理论上没有可组合性。因为后续规约、并行重复、知识提取、Fiat-Shamir 转换，都会显式消耗这些误差项。

## Knowledge Soundness 不是 Plain Soundness

plain soundness 只讨论假实例 $x \notin L_R$。它说的是: 伪造一个假命题的接受记录很难。

knowledge soundness 要求更强。它讨论的是另一件事:

如果某个 prover 能让 verifier 接受一个 instance，那么它是否“真的知道”某个 witness？

这就引入 extractor。非正式地说，若存在一个高效算法 $E$，能够通过访问 prover 的行为恢复出 witness，那么协议就有 proof of knowledge 风格的保证。一个常见接口写法是:

$$
\Pr[\text{accept}] - \Pr[R(x, E^{P^\*}(x)) = 1] \le \kappa(\lambda),
$$

其中 $\kappa(\lambda)$ 是 knowledge error。它表达的不是“每次接受都必然抽出 witness”，而是“接受概率与可抽取性之间的缺口受到控制”。

这里要特别注意两点。

### 第一，Soundness 与 Knowledge Soundness 的量词不同

plain soundness 只需要保证:

$$
x \notin L_R \Rightarrow \Pr[\text{accept}] \text{ 很低}.
$$

knowledge soundness 则要求:

$$
\text{若 } \Pr[\text{accept}] \text{ 足够高，则应能构造 extractor 恢复 witness。}
$$

前者只排除了“假命题骗过系统”，后者还试图把“成功证明”与“拥有见证”绑定起来。

### 第二，True Statement 也可能缺少 Knowledge Guarantee

即使 $x \in L_R$，plain soundness 仍然什么都没说。因为 soundness 根本不讨论真实例上的作弊 prover。理论上，可能存在一个 prover 能在真实例上制造接受 transcript，但我们却无法从它那里抽取 witness。

这就是为什么后面进入 Sigma 协议、proof of representation、SNARK knowledge soundness 时，extractor 会变成核心对象。只说 soundness 远远不够。

> Quick Note.
> “accepting transcript exists” 与 “prover knows a witness” 不是同义句。两者之间缺的正是 extractor 论证。

## 一个极小的接口视角

如果把本文压缩成一个后续可复用的模板，可以写成下面这样。

给定公开 instance $x$，我们先问：

1. 证明目标是什么语言成员性问题？
2. witness 的形式是什么？
3. verifier 接受的显式条件是什么？
4. 假实例上的伪造概率如何界定？
5. real view 与 simulated view 的距离如何界定？
6. 若要证明 knowledge soundness，extractor 的接口是什么？

后面几乎所有系统都只是把这六个问题搬到不同代数对象上：

- 在承诺与 Sigma 协议里，关系落在群元素与线性关系上
- 在 R1CS/QAP/AIR 里，关系落在向量与多项式恒等式上
- 在 KZG/FRI/PLONKish 里，acceptance condition 变成开点等式、低度性测试或 grand product identity

接口变了，定义链不变。

## Summary

零知识证明的最小理论框架并不复杂，但它必须分层:

- relation/language 固定“在证明什么”
- completeness/soundness 固定“何时接受、何时拒绝”
- simulator-based zero knowledge 固定“view 是否可被模拟”
- argument/proof 区分 soundness 的计算边界
- knowledge soundness 通过 extractor 把“接受”与“知道 witness”连接起来

如果没有这套分层，后面的协议分析很容易退化成模糊直觉: 有时把 HVZK 当成 full ZK，有时把 argument 当成 proof，有时把 soundness 当成 knowledge soundness。这些混淆都不是术语洁癖问题，而是会直接污染后续证明结构的问题。

下一篇会把这个接口落到有限域、循环群、离散对数与 Pedersen 承诺上。届时 relation 不再只是抽象谓词，而会变成具体的代数约束；simulator 与 extractor 也会开始有真正可写下来的形式。

## References

[^gmr]: Shafi Goldwasser, Silvio Micali, and Charles Rackoff, *The Knowledge Complexity of Interactive Proof Systems*, 1989.
[^goldreich]: Oded Goldreich, *Foundations of Cryptography, Volume 1*, zero-knowledge related chapters.


