# Groth16：从 QAP 到配对检验方程


> Reading: Groth16 as a derivation, not a magic trick.

Groth16 最容易被记成一句营销话术：proof 只有三个群元素，验证很快，所以它很强。这个说法不假，但几乎没解释任何东西。真正该理解的是：这三个群元素到底在编码什么；它们为什么足以代表一个 QAP witness；以及最后那条 pairing product equation 为什么不是凭空出现的黑箱检查，而是 QAP identity 在秘密点评估之后的压缩形式。

前两篇已经把接口准备好了。第 4 篇给出

$$
A_{\mathbf{w}}(X)B_{\mathbf{w}}(X) - C_{\mathbf{w}}(X) = H(X)Z(X),
$$

第 5 篇解释了 KZG 风格的核心直觉：多项式关系可以通过秘密点 $\tau$ 上的群编码与 pairing check 来验证。Groth16 本质上就是把这条思路做到了 QAP satisfiability relation 上，但它还额外引入了若干 trapdoors，把 witness 可伪造空间压得非常窄，最终只留下三段 proof tuple。[^groth16] [^pot]

所以这篇不打算把 Groth16 写成一个“比别的 SNARK 更短”的结果，而是把这条推导链写出来：

$$
\text{QAP relation}
\longrightarrow
\text{CRS at secret point}
\longrightarrow
\text{proof tuple } (A,B,C)
\longrightarrow
\text{pairing product equation}.
$$

<!--more-->

## Groth16 消费的对象是什么

Groth16 并不直接处理电路。它处理的是已经算术化完成的 QAP statement。

回忆上一篇，给定 witness 向量 $\mathbf{w}$ 后，QAP 把约束满足性压成

$$
A_{\mathbf{w}}(X)B_{\mathbf{w}}(X) - C_{\mathbf{w}}(X) = H(X)Z(X),
$$

其中：

- $A_{\mathbf{w}}(X), B_{\mathbf{w}}(X), C_{\mathbf{w}}(X)$ 由 witness 系数组合得到
- $Z(X)$ 是约束点集合的 vanishing polynomial
- $H(X)$ 是 quotient polynomial

这条式子的含义很直接：若 witness 满足所有约束，则左边在所有约束点上为零，因此必然被 $Z(X)$ 整除。

### Instance, Witness, And Public Inputs

Groth16 里并不是所有 witness 都是私有的。通常我们把 witness 向量拆成：

$$
\mathbf{w} = (\mathbf{x}_{\text{pub}}, \mathbf{x}_{\text{priv}}),
$$

其中公开输入会被嵌进 verifier 可见的 statement 中，私有部分只由 prover 持有。

所以 Groth16 最终证明的 statement 不是“存在任意一个 witness”，而是：

> 对固定公开输入 $\mathbf{x}_{\text{pub}}$，存在某个私有 witness $\mathbf{x}_{\text{priv}}$，使得对应 QAP identity 成立。

这点很重要，因为 verifier 最后确实要在 pairing check 里显式接触公开输入相关项。

## CRS 不是一团黑箱

Groth16 的 setup 经常被压缩成一句 “需要 trusted setup”。这不够。更准确的说法是：Groth16 需要一个结构化参考串 CRS，它在秘密点上编码了 QAP 多项式，同时还注入了若干额外 trapdoors，用于限制伪造证明的自由度。

### The Role Of \(\tau\)

第一层 trapdoor 是

$$
\tau.
$$

它的角色和上一篇 KZG 里的秘密点评估一样：把多项式关系固定在某个隐藏点上检查。因为若

$$
A_{\mathbf{w}}(X)B_{\mathbf{w}}(X)-C_{\mathbf{w}}(X)=H(X)Z(X),
$$

作为多项式恒等式成立，那么在 $X=\tau$ 处也有

$$
A_{\mathbf{w}}(\tau)B_{\mathbf{w}}(\tau)-C_{\mathbf{w}}(\tau)=H(\tau)Z(\tau).
$$

Groth16 的 CRS 会给出一系列元素，使 prover 能在不知道 $\tau$ 的情况下，仍然构造这些多项式在 $\tau$ 上的群编码。

### Alpha, Beta, Delta

Groth16 还引入额外 trapdoors，例如

$$
\alpha,\ \beta,\ \delta.
$$

这些量的角色不是重复 $\tau$ 的功能，而是切断某些伪造空间。

粗略理解：

- $\alpha,\beta$ 把 proof 中的主元素绑到同一组 witness 线性组合上
- $\delta$ 控制 prover 可以自由加入的随机化空间，并把这些随机化限制在 verifier 可检查的结构里

如果只有 $\tau$，系统更接近“单纯把 QAP 多项式放到秘密点”的世界；而加入 $\alpha,\beta,\delta$ 后，proof 元素之间开始具备更强的 algebraic linkage，最终才有可能把证明压成很小的三元组。

### Toxic Waste And Ceremony Intuition

Groth16 的 trusted setup 危险点也必须拆开讲。所谓 toxic waste，不只是某一个秘密值，而是这些 trapdoors 的整体泄露。

若 setup 方保留了足够多的 trapdoor 信息，它就可能伪造满足 verifier pairing equation 的 proof，而不需要真实 witness。原因不是“配对群不安全”，而是 verifier 的整个检查本来就依赖这些隐藏结构参数保证某些表示不可伪造。

所以 trusted setup ceremony 的目标不是生成一个好看的参数文件，而是确保：

- 有人知道参数如何生成
- 但没有任何单一方保留完整 toxic waste

这也是 Powers of Tau 类 ceremony 存在的原因。

## Proof 三元组在编码什么

Groth16 proof 通常写成三个群元素：

$$
\pi = (A, B, C),
$$

其中 $A$ 落在 $G_1$，$B$ 落在 $G_2$，$C$ 落在 $G_1$。如果只记住“只有三个元素”，还是没有回答它们分别带了什么信息。

### A And B Elements

先看前两个元素。它们本质上是在秘密点上编码 witness 相关的线性组合，并加上随机化项。例如可以把它们的直觉写成

$$
A \sim g_1^{\alpha + A_{\mathbf{w}}(\tau) + r\delta},
$$

$$
B \sim g_2^{\beta + B_{\mathbf{w}}(\tau) + s\delta},
$$

其中 $r,s$ 是 prover 自选随机数。

这里不要把具体常数布局看成唯一重要的记忆点。更重要的是结构：

- $A$ 编码了与 $A_{\mathbf{w}}(\tau)$ 相关的 witness 信息
- $B$ 编码了与 $B_{\mathbf{w}}(\tau)$ 相关的 witness 信息
- 两者都掺入 trapdoor offsets 与随机化，防止 proof 直接暴露底层表示

### The C Element

第三个元素 $C$ 最容易被当成“剩下的那一部分”。更准确地说，$C$ 负责吸收：

- quotient polynomial $H(\tau)$ 相关项
- witness cross terms
- 与随机化变量 $r,s$ 相关、必须被验证方程平衡掉的项

直觉上，它是把 “左边乘起来之后会多出来但又必须被控制的那些项” 一并编码进 proof。

如果没有 $C$，verifier 只能知道某种粗糙的乘积关系；而有了 $C$，就可以把 QAP identity 的剩余部分与随机化补偿全部折进一个额外群元素里。

### Why Three Group Elements Are Enough

Groth16 的小 proof 并不来自“把所有东西都压平了”。它来自更精细的事实：

1. QAP 已经把 many constraints 压成一条 divisibility relation
2. CRS 已经把多项式族固定在秘密点上
3. $\alpha,\beta,\delta$ 把伪造自由度切到了一个很小的 algebraic subspace
4. pairing 可以在乘法层面同时检查多个编码关系

所以最后 verifier 不需要看很多开点证明，也不需要看完整 witness commitment，只需要看三个 carefully structured group elements。

> Key Observation.
> “Why three group elements are enough” 的答案不是压缩技巧，而是 QAP reduction + structured CRS + pairing bilinearity 共同减少了 verifier 必须显式看到的对象数量。

## 从 quotient polynomial 到 pairing 方程

现在进入最核心的一步：最终 pairing product equation 为什么成立。

### Evaluate The QAP Identity At The Secret Point

既然 witness 正确，就有

$$
A_{\mathbf{w}}(X)B_{\mathbf{w}}(X)-C_{\mathbf{w}}(X)=H(X)Z(X).
$$

在 $X=\tau$ 上评估：

$$
A_{\mathbf{w}}(\tau)B_{\mathbf{w}}(\tau)-C_{\mathbf{w}}(\tau)=H(\tau)Z(\tau).
$$

这就是 Groth16 想检查的代数核心。区别只是 verifier 既不知道 $\tau$，也不想直接看这些标量值，于是只能看它们的群编码。

### Move To Pairing Space

pairing 的双线性让我们可以把标量乘法转成群元素之间的可检验关系：

$$
e(g_1^a, g_2^b) = e(g_1, g_2)^{ab}.
$$

因此，只要 $A$ 和 $B$ 分别编码了与 $A_{\mathbf{w}}(\tau)$、$B_{\mathbf{w}}(\tau)$ 相关的量，那么

$$
e(A, B)
$$

自然会生成这些量的乘积结构。与此同时，$C$ 与 verifier 预先拿到的 CRS 元素一起，负责把

$$
C_{\mathbf{w}}(\tau),\ H(\tau)Z(\tau),
$$

以及公开输入项、随机化补偿项都移到等式另一边。

### The Pairing Product Check

Groth16 最终 verifier 检查的形式可理解为

$$
e(A, B)
=
e(\alpha g_1, \beta g_2)
\cdot
e(\text{public-input encoding}, \gamma g_2)
\cdot
e(C, \delta g_2),
$$

这里省略了具体符号布局，但结构很清楚：

- 左边来自 proof 中前两个主元素的乘积
- 右边第一项固定 CRS 中的 trapdoor anchors
- 右边第二项吸收公开输入
- 右边第三项吸收 quotient / witness cross terms / randomization terms

于是 verifier 实际上是在问：

> 这三个 proof elements 是否共同编码了一个满足 QAP identity 的 witness，并且它们之间的代数关系与 CRS trapdoors 一致？

这就是 pairing product equation 的本意。它不是在“神秘地验证三个点”，而是在 pairing 空间中重现 QAP relation 的平衡式。

### Completeness Intuition

诚实 prover 为什么会过？

因为它从真实 witness 出发，先构造正确的

$$
A_{\mathbf{w}}(\tau),\ B_{\mathbf{w}}(\tau),\ C_{\mathbf{w}}(\tau),\ H(\tau),
$$

再按 CRS 规定的线性组合和随机化规则生成 $A,B,C$。由于底层 QAP identity 已经成立，所有在 pairing 空间里展开的项最终会刚好相消，留下 verifier 检查所要求的平衡方程。

这本质上和前一篇 KZG 的 opening equation 是同一类思路：先有 polynomial identity，再把它搬到秘密点评估，再用双线性工具压成可检查关系。区别只是 Groth16 把这个过程做得更结构化，也更针对 QAP witness relation。

## Knowledge Soundness 与 Trusted Setup 边界

Groth16 的安全性不能只写 soundness。更准确地说，它要强调 knowledge soundness。

### Knowledge Soundness Assumptions

Groth16 不是纯信息论 proof。它的知识可靠性依赖 pairing-friendly groups 上的代数假设，以及 extractor 能从某类成功 prover 中恢复 witness 的叙事。

也就是说，正确表述不是：

> “没人能伪造证明。”

而是：

> “若有人能构造通过 verifier pairing check 的 proof，那么在相应代数假设下，应可将其视为知道某个满足 QAP relation 的 witness。”

这里的 `knowledge` 不是心理学意义，而是 extractor-based cryptographic meaning。

### What Toxic Waste Leakage Breaks

一旦 toxic waste 泄露，knowledge soundness 也会被破坏。因为攻击者可能直接利用 trapdoors 构造满足 pairing equation 的代数对象，而不需要真实 witness。

所以 trusted setup 风险不是“参数最好别泄露”这种软提醒，而是：

- setup 正确时，proof compression 与 verifier efficiency 成立
- setup 泄露时，伪造空间会被重新打开

这也解释了为什么 Groth16 虽然非常实用，但 ceremony 边界永远是它架构的一部分，而不是部署细节。

## Summary

Groth16 若从结果往回看，会像一个只有三个群元素的神秘 SNARK；但若从 QAP 往前推，它的结构其实很连贯。

第一步，QAP 已经把电路满足性压成

$$
A_{\mathbf{w}}(X)B_{\mathbf{w}}(X)-C_{\mathbf{w}}(X)=H(X)Z(X).
$$

第二步，CRS 用秘密点 $\tau$ 和额外 trapdoors $\alpha,\beta,\delta$ 把这条多项式关系搬进结构化群编码。

第三步，proof 三元组 $(A,B,C)$ 分别承载主 witness 组合、乘积结构和 quotient/randomization 补偿项。

第四步，verifier 通过 pairing product equation 检查这些对象是否共同平衡成 QAP relation 在秘密点评估后的等式。

所以 Groth16 的小 proof 不是魔法，而是：

- QAP reduction
- structured CRS
- bilinear pairings
- carefully constrained proof shape

共同作用的结果。

下一篇若转向 STARK，视角会正好形成对照：不再依赖 toxic waste 和 pairing，而改走 AIR、FRI 和 transparent low-degree testing 的路线。

## References

[^groth16]: Jens Groth, *On the Size of Pairing-based Non-interactive Arguments*, 2016.
[^pot]: Powers of Tau and structured setup ceremony references.


