# Ethereum 中配对友好曲线的演化路径：从 BN254 到 BLS12-381


> Reading: Ethereum curve history is interface history first.

上一篇已经把 pairing-friendly curves 的数学接口说清了，但 Ethereum 的工程现实从来不只是“知道 pairing 有用”这么简单。真正决定协议能不能落地的，往往不是抽象上的 verifier compression，而是 execution layer 到底暴露了什么接口、这些接口的 gas 怎么定价、以及库和工具链是否能稳定消费这些接口。

所以 Ethereum 中配对友好曲线的演化路径，首先是一段接口史。BN254/alt_bn128 之所以先进入链上主流，不是因为今天回头看它最优雅，而是因为它先被 precompile 暴露成了真实可调用能力。相反，BLS12-381 虽然在安全 margin 和现代生态上形成了明显的 upgrade pressure，但 deployment friction 也更真实，执行层并不会因为“它看起来更好”就自动完成迁移。[^eip196] [^eip197] [^eip2537]

这一篇的重点因此不是“比较两条曲线谁更先进”，而是按时间线回答三件事：为什么 Ethereum adopted BN254/alt_bn128 precompiles early，gas-cost considerations 怎样直接改变 protocol feasibility，以及 BLS12-381 为什么持续构成升级压力却又没有简单替换掉 BN254。文末再把这条时间线压成工程实现对接清单。[^eip1108]

> Quick Note.
> This article explains why Ethereum adopted BN254/alt_bn128 precompiles early. It also contrasts BN254 and BLS12-381 in security margin, ecosystem maturity, and deployment friction, maps Ethereum precompile history to cryptographic capability changes, relates curve choice to verifier cost and deployability, and ties gas pricing and precompile availability to protocol feasibility.

<!--more-->

## 为什么 Ethereum 的曲线演化首先是接口史

如果只站在密码学教科书里看，问题很容易被写成“哪条 pairing-friendly curve 更好”。但在 Ethereum 里，真正更接近一线工程的问题是：EVM 有没有把相关运算暴露成可调用接口？如果没有，那么再漂亮的配对友好曲线也只是 paper design，不是 deployable protocol surface。

这就是为什么 execution layer availability 必须比抽象优雅先进入讨论。对链上 verifier 来说，pairing 不是“最好有”，而是“没有接口就根本做不了”。而一旦接口真的存在，gas pricing 和 precompile availability 就会直接决定哪类证明系统、哪类 verifier 和哪类 curve choice 变得可行。

This article ties gas pricing and precompile availability to protocol feasibility.

## BN254 / alt_bn128 为什么先落地

### EIP-196 与 EIP-197

Ethereum 早期给 pairing-friendly curve 打开的入口，是 `alt_bn128 precompiles`。EIP-196 提供了椭圆曲线加法和标量乘预编译，EIP-197 则把 pairing check 也带进了执行层。到这里，链上合约第一次拥有了一个现实意义上的 pairing verifier surface。[^eip196] [^eip197]

从 capability 角度看，这一步的变化很具体：

- 没有 precompile 之前，链上几乎不可能承受 pairing 运算
- 有了 add / mul / pairing check 之后，证明系统的 verifier 才具备进入合约的机会

This section maps Ethereum precompile history to cryptographic capability changes.

### 为什么链上 verifier 因此变得可行

对 zk verifier 来说，问题从来不是“pairing 在数学上能不能定义”，而是“pairing equation 能不能在智能合约里以可承受成本被执行”。当 EIP-196 和 EIP-197 把 BN254/alt_bn128 相关操作标准化为 precompile 之后，早期链上 verifier 的 deployability 才真正成立。

换句话说，Ethereum adopted BN254/alt_bn128 precompiles early 不是因为它预言了未来几十年的最优曲线，而是因为当时 execution-layer capability 需要一个现实可用、实现边界相对清楚的 pairing-friendly interface，而 BN254 先被做成了这个接口。

## EIP-1108：gas repricing 如何改变协议现实

### 为什么 gas 调整会改变 verifier choice

很多工程讨论会把 gas repricing 写成“参数微调”，但对密码学协议来说，它常常不是次要变量。EIP-1108 下调了 alt_bn128 precompiles 的 gas 成本，这意味着同一种 verifier 结构，在 repricing 前后会呈现完全不同的部署吸引力。[^eip1108]

`gas-cost considerations` 在这里不是附注，而是协议 feasibility 的一部分。链上 verifier 需要的不是“理论上能算”，而是“在用户和应用能接受的成本窗口里算”。

### verifier cost 与 deployability 的直接关系

This section relates curve choice to verifier cost and deployability.

更直接地看，curve choice 在 Ethereum 里总是和 verifier cost 绑定出现。因为链上协议实际消费的不是“曲线名字”，而是：

- 某条曲线对应的 precompile 是否存在
- 这些 precompile 的 gas 价格是否让 verifier 可承受
- 开发者和 prover 是否愿意围绕这一成本结构组织协议

一旦把这三个条件放回去，就会发现 EIP-1108 实际上是在重新定义“哪些 zk verifier 是可部署的”。

## BLS12-381 为什么形成升级压力

### security margin 与 ecosystem maturity

随着协议栈继续前进，BN254 的现实短板逐渐变得更明显。这里不必把它写成危言耸听，但有两个判断很难绕开：

- BN254 的安全 margin 相比现代预期更紧
- 围绕 BLS signatures、现代 pairing 库和新一代协议接口，`BLS12-381 upgrade pressure` 持续增加

这就是为什么 BLS12-381 在工程讨论里反复出现。它并不只是“另一条 pairing-friendly curve”，而是一个在现代安全边界、生态工具链和协议复用上更有吸引力的目标。

This section contrasts BN254 and BLS12-381 in security margin, ecosystem maturity, and deployment friction.

### EIP-2537 与 deployment friction

但升级压力不等于自动落地。EIP-2537 试图把 BLS12-381 运算也暴露成 precompile，正说明生态已经感受到执行层的接口缺口。与此同时，这也暴露了 deployment friction 的现实：增加新的 precompile、修改客户端实现、统一测试向量、处理 serialization 与 subgroup checks，都不是一句“换曲线”能解决的。[^eip2537]

所以 BLS12-381 的故事更像这样：

- 数学和安全性层面，它给出更强的现代吸引力
- 协议和生态层面，它和 BLS 聚合签名、KZG 等接口天然对接
- 执行层层面，它仍然受到 precompile adoption 和兼容性路径的制约

这也是为什么“BN254 已经过时，所以很快会被完全替代”这种说法在工程上太粗糙。

## 一张工程对照表

把 BN254 和 BLS12-381 压成一张 Ethereum 语境下的表，会更容易扫读。

| 维度 | BN254 / alt_bn128 | BLS12-381 |
| --- | --- | --- |
| 历史角色 | 早期链上 zk verifier 的现实入口 | 现代 pairing 生态与新协议的升级目标 |
| 执行层可用性 | 已有 `alt_bn128 precompiles` | 需要额外 precompile / client support |
| gas 现实 | EIP-1108 后更可部署 | 执行层缺口让 deployability 更复杂 |
| 安全与生态 | 历史包袱更重，安全 margin 偏紧 | 现代库、现代协议和更强安全预期 |
| 工程压力 | 已部署、已消费、难以轻易抛弃 | upgrade pressure 强，但 deployment friction 也强 |

这张表真正想说的是：Ethereum 中的曲线演化从来不是线性替代，而是历史可用性和现代压力长期并存。

## 工程实现对接

### precompile boundary 与 library support

现实部署里最容易踩坑的，通常不是“理论上理解了 BN254 和 BLS12-381 的差异”，而是没有把 precompile boundary 和 userland library boundary 分开。

工程上至少要问：

- 协议运行在哪一层，execution 还是 consensus
- verifier 依赖的 curve operation 是走 precompile 还是走用户态库
- library 是否对 subgroup checks、serialization 和 invalid input 做了明确边界

这些边界不清，curve choice 很容易在实现阶段变成 deployability 问题。

### serialization、subgroup checks 与 protocol integration

pairing-friendly curves 进入协议后，serialization 和 subgroup checks 永远不是收尾工作，而是第一层 integration boundary。尤其当系统一部分运行在链上、一部分运行在 prover 或 off-chain service 里时，编码规则和 subgroup discipline 是否一致，直接决定 protocol integration 是否可靠。

这也是为什么 deployment friction 不该只理解成“还没上链”。更准确的说法是：它包含了客户端支持、测试向量、编码约束、gas 成本、接口一致性和运维成熟度。

### 为 KZG / EIP-4844 做 handoff

到这里，第 5 篇真正做完的事只有一件：把 Ethereum pairing curve story 从“曲线优劣比较”收束成“execution-layer capability、gas pricing 和 deployment surface 的历史”。下一篇进入 KZG / EIP-4844 时，主线会继续收缩到更具体的问题：为什么 modern Ethereum data availability commitments 会自然落到 BLS12-381，以及 trusted setup 怎样重新回到协议主轴。

## 总结

Ethereum 中配对友好曲线的演化路径，本质上是 capability、cost 和 deployment surface 的共同演化。BN254 先落地，是因为它先被 execution layer 做成了真实接口；BLS12-381 形成升级压力，是因为现代协议、安全 margin 和生态成熟度都在把系统往那个方向推。

因此这一篇最想固定下来的判断线是：在 Ethereum 里，curve choice 从来不是抽象偏好，而是 precompile availability、gas-cost considerations、library maturity 和 protocol demand 的交叉结果。

下一篇进入 KZG / EIP-4844 时，这条线会更具体：pairing-friendly curve 不再只是 verifier 友好，而是直接变成数据可用性承诺工作流的一部分。

[^eip196]: EIP-196, *Precompiled contracts for addition and scalar multiplication on the alt_bn128 curve*.
[^eip197]: EIP-197, *Precompiled contracts for optimal ate pairing check on the alt_bn128 curve*.
[^eip1108]: EIP-1108, *Reduce alt_bn128 precompile gas costs*.
[^eip2537]: EIP-2537, *Precompile for BLS12-381 curve operations*.

