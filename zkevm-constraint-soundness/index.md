# zkEVM 与 ZK 系统安全性：约束设计、内存一致性与 soundness footguns


> Reading: soundness is only as strong as the constrained surface.

zkEVM 文章最容易写歪的地方，是把“有 EVM opcode、能出 proof、链上 verifier 接受”误当成安全性本身。真正需要抓住的是：proof 只覆盖那些被约束、被 commitment 绑定、再被 transcript 吸收进去的对象。任何没有被这条链路真正绑住的语义，都不在证明系统的 soundness boundary 里。

所以这篇不讨论产品对比，而讨论 engineering attack surface。zkEVM 不等于“把客户端代码翻译成电路”；更准确地说，它是把 opcode 语义、memory read/write、storage queue、状态根更新、lookup 表、递归聚合这些对象拆成一组 constraint obligations，再证明这些 obligations 一起闭合。[^polygon-main] [^zksync-circuits] [^scroll-proof]

也因此，最典型的 failure mode 不是某条大公式明显写错，而是 linkage 丢了。某个 selector 没把实例绑定到正确路径，某个 queue payload 没和前一层结果相等，某个 lookup 只检查 limb 合法却没检查重组，某个 challenge 没吸收所有 commitments。只要缺一条边，prover 就可能构造一个“局部都像样、全局却不是你想证明的执行”的 witness。

<!--more-->

## 证明系统到底在验证什么

zkEVM 从来不是直接“执行 EVM”。它做的是 opcode semantics to constraint obligation mapping：把每条 opcode 想表达的语义，拆成电路真正能检查的局部约束与跨模块一致性义务。

以一条简化的 `SSTORE` 风格语义为例，程序员脑中的语义大致是：

1. 从当前上下文读出 address、slot、value
2. 根据当前 state 计算写入前后的状态差异
3. 更新 storage、refund、access list 或其他相关状态
4. 把新的全局状态承诺到下一步

但在约束系统里，这些语义会被分解成：

1. opcode decoder 约束当前 row 的 selector 是否对应 `SSTORE`
2. witness columns 提供 address、slot、value 等局部对象
3. lookup 和 range checks 保证字节分解、字段范围、编码格式合法
4. memory / storage queues 记录读写事件
5. 排序或去重电路检查这些事件满足时间顺序与键一致性
6. 递归或聚合层最终只验证若干 commitments 与 accumulator 关系

这里真正危险的点在于：EVM 语义在工程上已经被拆成很多局部 obligations。只要其中一条 linkage 没被电路表达出来，系统证明的就不再是“这条 opcode 的真实语义”，而只是“某些局部对象分别长得像合法片段”。

### One VM, Many Auxiliary Consistency Tasks

这也是为什么现代 zkEVM 通常不是单电路。Polygon zkEVM 文档把主状态机与 ROM、memory、binary、hash、storage 等状态机拆开来描述；zkSync Era 的电路文档则明确区分 Main VM、storage sorter、code decommitter、RAM permutation 等辅助电路；Scroll 的证明栈也强调把 execution trace、证明生成、递归聚合拆成多层。[^polygon-main] [^zksync-mainvm] [^zksync-storage] [^scroll-proof]

拆分本身不是问题。问题在于拆分之后，安全性不再只看某一个局部 gate，而要看这些模块之间是不是形成了闭环。

> Key Observation.
> proof 证明的不是“解释器看起来像在跑”，而是“所有被拆开的局部 obligations 共同锁死了同一条 execution trace”。

## Memory And State Consistency Are The Real Backbone

很多人第一次看 zkEVM，会把注意力放在 opcode arithmetic、hash gadget、precompile gadget 这些显眼模块上。但从 soundness 角度看，真正更像 backbone 的是 memory and state consistency constraints。

原因很直接。大系统里，单步局部语义通常不难写；难的是保证：

- 这一步读到的值，真的是前面某一步写进去的值
- 这次 storage update，真的是对同一个 key 做的前后更新
- rollback、refund、access list、nonce、balance 这些状态不是各自漂移
- 递归聚合前后的 commitments 的确绑定的是同一份历史

这就是 memory and state consistency constraints 的核心。

### Memory Consistency

把内存读写写成事件流之后，最朴素的正确性条件其实只有一句：

> 对同一个地址和时刻，最后一次写入必须决定之后读出的值。

但为了把它变成可证明对象，系统通常会把读写事件编码成某种 queue 或 multiset，再借助排序、permutation、timestamp 检查等机制去表达：

$$
\text{read}(addr, t) \Rightarrow \text{value} = \text{last-write-before}(addr, t).
$$

一旦这条关系没有被完整表达，局部电路即便都成立，prover 仍可能让“当前 opcode 看到的 value”和“全局 memory history 里的 value”发生偏离。

### Queues, Sorting, And State Roots

这也是 auxiliary circuits 存在的主要原因。它们并不是“给主电路加速”的配件，而是在负责 global consistency：

- memory queue 负责把局部读写事件送入全局历史
- sorter 负责把同一 key 或地址的事件排到可比较顺序
- state root / storage root 更新负责把局部变化压成 commitment 边界
- recursion / aggregation 层负责把很多局部 obligations 再压成更小的 verifier surface

zkSync 的 storage sorter 文档就很适合拿来看这个问题：主 VM 不会独立承担所有全局一致性，很多正确性其实被转移到 queue、sorting 与后续连接电路去消费。[^zksync-storage]

所以 memory consistency 不是“另一个模块”，而是 zkEVM 里所有模块最终都要返回去满足的 global invariant。

## Under-Constrained Circuits Usually Mean Missing Linkage

under-constrained circuits 这个词很常见，但很多时候说得还是太抽象。更直接一点：

> under-constrained circuit risk 的本质，不是 prover 找到了一条违反公式的路径，而是系统根本没有把预期语义完整写成公式。

换句话说，某些自由度本来应该被锁死，却仍留在 witness 里。

### Local Checks Can All Pass

这类 bug 的可怕之处在于，它们经常满足下面的表象：

- selector 检查通过了
- lookup 表查到了
- range check 也过了
- queue 事件格式合法
- 最终 quotient 看上去没有问题

但这些只说明“局部片段各自合法”，不说明“它们描述的是同一个 execution fact”。

Trail of Bits 在 Halo2 相关案例里指出过类似教训：如果约束系统没有把本该相等的对象真正连起来，证明系统会接受大量并非设计者预期的 witness。[^tob-under]

### A Toy Soundness Bug From Missing Constraint Linkage

看一个极简 toy bug。设某条 opcode 需要把 ALU 结果 `r` 写入 memory queue，并且系统分别做了两类检查：

1. `r` 经过 lookup and range assumptions，证明它是合法的 32-byte decomposition
2. memory queue append 约束了写入事件 `q = (addr, ts, v)` 的格式

如果设计者忘了加上

$$
v = r
$$

这条 linkage，那么 prover 可以这样做：

1. 提供一个局部合法的 `r`，让 ALU 与 lookup 检查全部通过
2. 再提供另一个同样格式合法的 `v`，把它写进 memory queue
3. 后续 memory consistency 只看到 queue 里的 `v`

于是电路实际证明的是：

- “我算出了某个合法的 `r`”
- “我也写入了某个合法的 `v`”、

却没有证明“被写入的就是算出来的那个值”。这就是一个 example of a soundness bug from missing constraint linkage。

这个例子故意很小，但它足够表达现实里的 bug 形态：缺的往往不是大结构，而是一条把两个局部合法对象绑成同一个事实的等式、selector 条件或 permutation relation。

## Lookup、Range 与 Transcript 都会扩大攻击面

写到这里，危险边界已经比较清楚了。下面两类对象尤其容易让人产生“已经检查过了”的错觉。

### Lookup And Range Assumptions

lookup and range assumptions 确实能显著降低电路复杂度，但它们也引入了额外信任边界：

- table 本身是否真表达了你想要的语义
- limb 合法是否真的推出原对象合法
- decomposition 与 recomposition 是否双向绑定
- 不同列之间是不是只做了局部 membership，却没做全局一致性

例如一个 256-bit word 被拆成若干 limb，每个 limb 都通过 range check，并不自动推出原 word 被正确使用。若缺少

$$
x = \sum_{i=0}^{k-1} 2^{w i} x_i
$$

这样的 recomposition 约束，系统可能只证明“这些 limb 分别在范围内”，却没有证明“它们就是这个 word 的真实分解”。

这类问题和上面的 queue linkage 是同一类 bug：局部合法，不等于全局语义闭合。

### Fiat-Shamir And Transcript Misuse Pitfalls

同样，Fiat-Shamir and transcript misuse pitfalls 往往不是“哈希函数坏了”，而是 transcript 没把应该绑定的对象全都吸收进去。

如果 challenge 没有绑定：

- 某个 auxiliary queue commitment
- 某个 lookup table commitment
- 某个 recursive layer 的 accumulator state
- 某个 instance boundary 上的公开承诺

那么 prover 可能在 challenge 生成后仍保留额外自由度，等于把一部分应该固定的 witness 留到了后面。

从工程视角看，transcript 的职责不是“把协议串起来”，而是把所有必须被 challenge 绑定的 commitments 关进同一个不可回退的顺序里。只要顺序、覆盖面或域分离做错，soundness 就可能从系统边界漏出去。

## System-Level Soundness Boundary

现在可以回到开头那句话：proof 只覆盖被约束并被 commitment 绑定的对象。

所以 system-level soundness boundary 必须明确区分两件事：

1. 电路层是否完整表达了预期语义
2. 系统运行模式是否真的在消费这些被证明的语义

如果某个 dev/test mode 允许 mock finalization、跳过真实 proof、或把某些 bridge / settlement 步骤交给外部治理逻辑，那它们都不属于密码学证明自动覆盖的范围。Scroll 的文档就明确把 proof generation、aggregation 与不同部署模式区分开来。[^scroll-proof]

这不是在贬低系统，而是在明确边界。工程上最危险的误解，恰恰是把“某个模块有 proof”误当成“整个系统没有额外 trust assumption”。

### Engineering Attack Surface Checklist

如果把这篇压成一份审计清单，我会先查这几类点：

1. opcode semantics to constraint obligation mapping 是否有缺边
2. memory and state consistency constraints 是否真的闭合到队列、排序与状态承诺
3. 是否存在 under-constrained circuits，尤其是 selector、equality linkage、recomposition linkage 漏掉的地方
4. lookup and range assumptions 是否只检查了局部合法性，而没有检查全局绑定
5. Fiat-Shamir and transcript misuse pitfalls 是否可能让 challenge 在少吸收对象的情况下生成
6. recursive aggregation 有没有遗漏某个 accumulator / commitment 边界
7. 系统部署与运维模式是否超出了 proof 真正覆盖的 system-level soundness boundary

这份清单也是本文真正想给出的 engineering attack surface。

## Summary

zkEVM 的安全性不能只看“有没有证明系统”，而要看证明系统到底绑定了什么。

这条线可以压成四句话：

1. zkEVM 的核心是 opcode semantics to constraint obligation mapping，而不是把解释器原样搬进电路
2. memory consistency 与更一般的 memory and state consistency constraints 是全局 backbone
3. under-constrained circuits 往往来自 missing constraint linkage，而不是单条公式的显眼错误
4. lookup and range assumptions、Fiat-Shamir transcript、recursive aggregation 与部署模式一起定义了 system-level soundness boundary

所以真正的工程问题不是“proof 跑出来没有”，而是“系统有没有把它声称在证明的那件事，完整而闭合地写进约束、commitment 与 transcript”。

## 工程实现对接

如果把这篇落回工程动作，最值得保留的是三件事：

- 设计阶段先画 dependency graph，明确每个 opcode 结果最终被哪些 queue、state root、accumulator 消费
- 审计阶段优先找 missing linkage，而不是先找大公式里的显眼代数错误
- 测试阶段构造 adversarial witness，用“局部都合法、全局不一致”的方式去撞 under-constrained circuits

这比泛泛地说“加强测试”和“提高覆盖率”更接近 zk 系统真正的失效路径。

## References

[^polygon-main]: Polygon zkEVM, *Main state machine*, <https://docs.polygon.technology/zkEVM/architecture/zkprover/main-state-machine/>.
[^zksync-circuits]: ZKsync Era Docs, *Circuits overview*, <https://docs.zksync.io/zksync-protocol/circuits/circuits>.
[^zksync-mainvm]: ZKsync Era Docs, *Main VM*, <https://docs.zksync.io/zksync-protocol/circuits/circuits/main-vm>.
[^zksync-storage]: ZKsync Era Docs, *Storage sorter*, <https://docs.zksync.io/zksync-protocol/circuits/circuits/storage-sorter>.
[^scroll-proof]: Scroll Docs, *Proof generation*, <https://docs.scroll.io/en/technology/prover/proof-generation/>.
[^tob-under]: Trail of Bits, *A Riel Dilemma in Halo2 CE*, <https://blog.trailofbits.com/2024/02/20/a-riel-dilemma-in-halo2-ce/>.

