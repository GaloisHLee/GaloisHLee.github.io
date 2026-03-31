# IPA 系证明与递归：Bulletproofs、Halo、Nova/Folding


> Reading: recursion through the IPA lens.

这一篇最容易写错的方式，是把 Bulletproofs、Halo、Nova 分成三段协议摘要。那样会丢掉真正重要的主线：inner-product machinery 为什么天然会把系统推向 recursion；以及“可递归的对象”到底在不同系统里发生了什么变化。

若只从结果看：

- Bulletproofs 给出对数级压缩证明
- Halo 给出递归验证路线
- Nova 给出 folding / IVC

这三句都不假，但还是太散。更有用的看法是：它们都在处理“证明对象如何被继续压缩”这个问题，只是压缩的对象不同。

在这条线上：

- Bulletproofs 压缩的是高维 relation 本身
- Halo 累积的是 verification obligations
- Nova 折叠的是 instance / witness state

所以这一篇的主角不是 chronology，而是对象变化：从证明一个 instance claim，到维护一个 accumulator，再到维护一个可继续折叠的证明状态。[^bulletproofs] [^halo] [^nova]

<!--more-->

## 为什么 IPA 天然通向递归

inner-product argument, IPA，最核心的不是某个特定应用，而是这样一种压缩原理：

> 高维代数关系若能被重写成内积声明，就可以通过 repeated halving 在对数轮里压缩。

设有向量

$$
\mathbf{a}, \mathbf{b} \in \mathbb{F}^n,
$$

要证明

$$
\langle \mathbf{a}, \mathbf{b} \rangle = t.
$$

若把向量拆成左右两半

$$
\mathbf{a} = (\mathbf{a}_L, \mathbf{a}_R),\qquad
\mathbf{b} = (\mathbf{b}_L, \mathbf{b}_R),
$$

则内积可以拆成

$$
\langle \mathbf{a}, \mathbf{b} \rangle
=
\langle \mathbf{a}_L, \mathbf{b}_L \rangle
+
\langle \mathbf{a}_R, \mathbf{b}_R \rangle.
$$

再用一轮挑战把左右部分线性混合，就能把长度 $n$ 的证明变成长度 $n/2$ 的新证明。反复进行后，最终只剩一个很小的终点检查。

### What “Logarithmic” Really Means

所谓 IPA 的“对数级证明”不是平白出现的 succinctness，而是每一轮都做了维度减半：

$$
n \to n/2 \to n/4 \to \cdots \to 1.
$$

所以它本质上是一条 compression chain。后面 recursion 之所以自然接上来，就是因为：

- 你已经在逐轮压缩一个证明对象
- 你开始关心这种压缩是否还能继续嵌套、继续延迟、继续累积

这就是从 IPA 到 recursion 的第一座桥。

## Bulletproofs：压缩 skeleton

Bulletproofs 在这条线上最该保留的，不是 confidential transactions 的应用背景，而是它展示了 IPA 如何成为一个通用压缩 skeleton。

### From Relation To Inner Product

很多高维约束最终可以改写成一个内积声明，再加上一些承诺一致性条件。Bulletproofs 的 insight 是：

若 verifier 不需要逐分量看到向量，而只需被说服某个承诺向量满足内积关系，那么 IPA 正好给出一个很自然的对数级证明。

因此，Bulletproofs 的真正贡献可以用一句更抽象的话概括：

> 把“高维 witness relation”变成“可递归折叠的内积验证对象”。

### Repeated Folding

每一轮 IPA transcript 都会发送左右交叉项承诺，然后根据 verifier challenge，把：

- witness vector
- evaluation vector
- 基底向量

一起折叠成更短的新对象。

于是证明对象不再是静态 instance claim，而是“一个正在被压缩的 relation state”。这点非常关键，因为它已经在心理上把我们从“验证一份最终证明”带向“维护一个可以继续压缩的证明过程”。

## 为什么递归验证不是简单套娃

到了 recursion，最直觉的想法是：

> 那就在证明里再跑一次 verifier。

这当然可以作为定义起点，但工程上通常很糟。因为 verifier 本身要做：

- 承诺检查
- 多项式开点检查
- transcript 吸收
- 有时还要做 pairings 或复杂 MSM

若每一层都完整重跑上一层 verifier，递归成本会很快变成新的瓶颈。

所以 recursion 里真正的问题不是“能不能嵌套 verifier”，而是：

> 能否把越来越多的 verification obligations 压成一个更小、更可继续处理的对象？

这就引出了 accumulation。

## Halo：累积 verification obligations

Halo 的关键转向在于，它不把 recursion 理解成“每次都 fully verify 前一份证明”，而是把多份待验证义务压缩成一个 accumulator。

### Accumulator As New Instance Claim

accumulation 的视角是：

- 原本 verifier 有很多独立验证义务
- 现在把这些义务组合成一个新的 instance claim
- 之后只需证明“这个 accumulator 的验证义务是合法累积来的”

所以递归对象不再是“旧证明全文”，而是“压缩后的验证义务”。

这和 Bulletproofs 的压缩 skeleton 一脉相承，但对象已经变化：

- Bulletproofs 压 relation dimension
- Halo 压 verification obligations

### Deferred Verification

Halo 路线的直觉可以理解为 deferred verification。也就是：

我不必在当前层把所有低层证明彻底验完，只要把“尚未验完但已被正确打包”的义务转交给下一个 accumulator 即可。

这正是 recursion 真正可扩展的原因之一。否则，递归只是在复制 verifier，而不是在压缩验证工作。

> Key Observation.
> accumulation 不是“合并很多 proofs”这么含糊，它更准确地是在合并 many verification claims into one compressed obligation。

## Folding 不是 Accumulation

接下来必须把一个很常见的混淆拆开：accumulation 和 folding 不是同义词。

### Accumulation Merges Claims To Verify

在 accumulation 视角里，主角是 verification obligations。你关心的是：

- 哪些证明还没完全验完
- 如何把这些待验证条件压到更小对象里
- verifier 以后只需继续处理这个更小对象

也就是说，被压缩的是“验证任务”。

### Folding Merges Instances Themselves

在 folding 视角里，主角则是 instance / witness 对象本身。你不只是把验证义务打包，而是把两个实例直接合成一个新实例，使得：

> 新实例可满足，当且仅当旧实例对某种组合意义下可满足。

也就是说，被压缩的是“问题对象”。

这一区别很重要，因为它决定了系统究竟更像：

- 递归 verifier pipeline

还是

- incremental proving state machine

Halo 更偏前者，Nova 更偏后者。这就是全文需要保留的 accumulation vs folding distinction：前者压的是 verification obligations，后者折叠的是 instance / witness 对象本身。

## Nova：面向 IVC 的 folded instance

Nova 代表的路线，是把 folding 做成 first-class object。

### Why A Relaxed Instance Appears

如果想把两个实例直接折叠成一个新实例，通常需要允许某种“残差”或“松弛”存在。否则，两份独立约束很难无损地塞进一个同类型实例里。

因此 Nova 类系统会引入 relaxed instance 的想法：新的实例不仅包含原始 instance 信息，还允许带着一个受控的 residual term，一起进入下一轮。

这一步的意义不是数学装饰，而是为了让“实例可继续被折叠”这件事成为闭合操作。

### Folded Instance Construction

高层直觉上，若有两个实例

$$
I_1,\ I_2
$$

及其 witness

$$
w_1,\ w_2,
$$

folding scheme 会用随机挑战 $\rho$ 构造一个新实例

$$
I^\* = I_1 + \rho I_2
$$

以及对应 folded witness

$$
w^\* = w_1 + \rho w_2,
$$

再把残差项一起吸收到 relaxed relation 中。

具体公式取决于所折叠的约束系统，但关键直觉不变：

- 不是把两个 verifier 任务合并
- 而是把两个 instance/witness 对直接线性组合成一个新对象

这就是 folded instance construction at a high level：两个实例及其 witness 经过随机线性组合，连同 relaxed residual 一起形成可继续递推的新实例。

### IVC-Oriented Reasoning

Nova 路线最自然的系统目标是 IVC, Incrementally Verifiable Computation。也就是说，计算不是一次性全部证明完，而是：

1. 当前步骤生成一个新的 computation state
2. 把旧 state proof 和新 step relation 折叠进一个新证明状态
3. 这个新状态继续进入下一步

这样，证明系统的主对象就变成了一个持续演化的 folded state，而不只是某份静态终稿证明。

所以 Nova 一系真正改变的，不只是 proof compression，而是把 proving 本身改写成 stateful process。

## 统一视角：对象在逐步变化

现在把三条路线放回同一主线里。

### Bulletproofs

它告诉我们：高维 relation 可以通过 inner-product folding 压到对数大小。

### Halo

它告诉我们：递归对象不必是完整 proof，可以是被压缩后的 verification obligation，也就是 accumulator。

### Nova / Folding

它告诉我们：甚至连 verification obligation 都不必是唯一主角，instance 本身也可以被折叠成一个持续演化的新对象。

于是，“递归证明”这个词在现代语境下至少包含三种不同层次：

- relation compression
- obligation accumulation
- instance/state folding

如果不把这三层拆开，很多系统会被误看成只是同一个想法的不同命名。

## Summary

这条 IPA-to-recursion 主线，其实是在不断改变“被压缩的对象”。

先是 Bulletproofs：

- 内积关系被对数级折叠

然后是 Halo：

- 多个 verification claims 被压成 accumulator

再到 Nova：

- instance / witness state 本身被 folded into a relaxed state

所以 recursion 不是简单把 verifier 套进 verifier，而是在寻找一个可以跨轮维护、持续压缩、持续组合的证明对象。

如果把现代 proving system 设计看成一条连续演化线，那么这篇的关键词应该是：

- compression
- accumulator
- folded instance
- incremental state

下一篇进入 zkEVM 与工程约束时，这条线会落到更具体的问题上：真正的大系统里，到底哪些约束设计会让 soundness 漏掉，哪些状态对象必须被严格维护，哪些“看起来只是工程实现”的选择会直接变成证明语义的一部分。

## References

[^bulletproofs]: Benedikt Bünz et al., *Bulletproofs: Short Proofs for Confidential Transactions and More*, 2018.
[^halo]: Sean Bowe, Jack Grigg, and Daira Hopwood, *Halo: Recursive Proof Composition without a Trusted Setup*, 2019.
[^nova]: Srinath Setty et al., *Nova: Recursive Zero-Knowledge Arguments from Folding Schemes*, 2021/2022 line of work.

