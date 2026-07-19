# 分组密码工作模式攻击讲义 Part 4：XTS、Tweakable Enciphering 与存储机密性的边界


> Note: 前三卷讨论的主角都是 message encryption。Vol.4 换了问题模型。攻击者不再主要围着 transcript、tag 与 decrypt-oracle 打转，而是围着 sector number、block offset、旧密文镜像与版本回滚做文章。

XTS 的价值不在于把磁盘加密变成 AEAD，而在于给每个位置引入一个 tweak，使“同一明文块在不同逻辑位置直接映成同一密文块”这件事不再发生。它解决的是结构化存储上的 confidentiality 问题，而不是 data origin authentication、freshness 或 rollback detection。

所以这一卷真正要回答的是：tweakable enciphering 到底改变了什么，没有改变什么；为什么 XTS 能修补 ECB 在磁盘上的轮廓泄漏，却仍然挡不住 replay、rollback 和旧状态回放；以及为什么 storage confidentiality 与 authenticated storage 根本不是同一层保证。

<!--more-->

## 0. Threat Model：消息加密与存储加密根本不是同一个游戏

前三卷的默认对象都是消息。一个抽象接口通常写成：

$$
\mathcal{E}_K(N,A,P) = (C,T), \qquad \mathcal{D}_K(N,A,C,T)\in\{P,\bot\}.
$$

这里攻击者看到的是：

- message transcript
- nonce
- tag
- decrypt-or-reject interface

而存储场景里，攻击者更常见的接口是：

- 读取某个逻辑位置上的 ciphertext block
- 覆写某个逻辑位置上的旧 ciphertext block
- 整体回滚到旧 snapshot
- 交换 metadata 与 data-unit mapping

因此 storage model 的基本单位不是一次 message transcript，而是一个 `data unit`，例如：

- sector
- page
- block group
- logical block address

这也是为什么把消息 AEAD 语言直接套到磁盘上会失焦。这里真正的问题不是“这条记录有没有 tag”，而是：

1. 相同明文块在不同位置是否还会暴露相等关系。
2. 旧密文镜像写回原位置时能否被检测。
3. 整体 snapshot rollback 是否会被发现。
4. 存储层是否知道自己正在读取的是“当前版本”还是“过去某次合法版本”。

换句话说，存储加密面对的是一个比 message confidentiality 更接近 `state continuity` 的问题。

## 1. 从 ECB 到 tweakable enciphering：为什么存储模型需要位置参数

### 1.1 ECB 用在磁盘上为什么天然暴露轮廓

若直接对 sector 中每个 block 用 ECB：

$$
C_i = E_K(P_i),
$$

则有

$$
P_i = P_j \Longleftrightarrow C_i = C_j.
$$

在磁盘或数据库页上，这意味着：

- 相同的全零块仍然映到相同密文块
- 相同文件头、相同空洞、相同元数据模板仍然映到相同密文块
- 不同 logical positions 上的 identical contents 仍会留下 identical ciphertext patterns

这正是为什么“磁盘上的 ECB”会保留结构轮廓。

### 1.2 加一个随机 IV 为什么不够

对消息加密来说，我们习惯用随机 IV 或 nonce 去打散相同首块。但磁盘加密不适合为每个 block 单独附一个 message-style IV，原因至少有三层：

1. block 是可随机访问的，不能要求读取第 \(i\) 块前先流式处理前面所有块。
2. 同一逻辑位置需要稳定地重新解密，不适合依赖外存中另存一堆 per-block randomness。
3. 磁盘位置本身就是最自然的 domain separator。

因此 storage model 更自然的接口不是

$$
E_K(P;R),
$$

而是

$$
\widetilde{E}_K(T,P),
$$

其中 \(T\) 是 tweak，通常由位置或数据单元编号导出。

### 1.3 tweakable enciphering 的最小语义

把 tweakable enciphering 抽象写成：

$$
\widetilde{E}_K : \mathcal{T}\times\{0,1\}^{n}\to\{0,1\}^{n},
$$

$$
\widetilde{D}_K : \mathcal{T}\times\{0,1\}^{n}\to\{0,1\}^{n}.
$$

对每个固定 tweak \(T\)，映射

$$
P \mapsto \widetilde{E}_K(T,P)
$$

仍然是一族置换；但不同 tweak 代表不同位置上下文。

这意味着 tweak 的真正作用不是“再给一段随机数”，而是：

> 把同一个 block cipher 的作用点切分成许多按位置索引的独立域。

所以 tweak 改变的是 `where this block lives`，而不是 `whether this block is authenticated`。

## 2. XTS：每块方程与 tweak 递推

### 2.1 XEX skeleton

XTS 的核心方程可以先写成 XEX 结构：

$$
C_i = E_{K_1}(P_i \oplus T_i)\oplus T_i.
$$

解密则是

$$
P_i = D_{K_1}(C_i \oplus T_i)\oplus T_i.
$$

如果只看这一式，XTS 和 OCB / XEX / 其他 tweakable enciphering 看起来很像：都是“进 AES 前异或 tweak，出 AES 后再异或回来”。

但这里千万不要自动联想到 AEAD。这个结构的第一目标只有一个：

$$
P_i \text{ 在不同位置时，不应再进入同一个 } E_{K_1}\text{ 输入点。}
$$

### 2.2 tweak 来自数据单元编号与块偏移

XTS 把一个 data unit 的编号记作 \(j\)，例如 sector number。先计算该数据单元的 base tweak：

$$
T_0 = E_{K_2}(j).
$$

然后对同一数据单元内部的第 \(i\) 个 block，使用

$$
T_i = \alpha^i \cdot T_0,
$$

其中乘法发生在 \(GF(2^{128})\) 中，\(\alpha\) 通常对应域中的 \(x\)。

因此完整写法是：

$$
C_{j,i} = E_{K_1}(P_{j,i}\oplus \alpha^i E_{K_2}(j)) \oplus \alpha^i E_{K_2}(j).
$$

这条式子说明了两层位置参数：

- 不同数据单元 \(j\) 有不同 base tweak
- 同一数据单元中不同块 \(i\) 再通过 \(\alpha^i\) 分离

### 2.3 为什么这能打掉 ECB 式相等关系

设有两个相同明文块：

$$
P_{j,i} = P_{j',i'}.
$$

若对应 tweak 不同：

$$
T_{j,i} \neq T_{j',i'},
$$

则进入 \(E_{K_1}\) 前的输入分别为

$$
P_{j,i}\oplus T_{j,i}, \qquad P_{j',i'}\oplus T_{j',i'}.
$$

因为 tweak 不同，这两点通常不同。于是：

$$
E_{K_1}(P_{j,i}\oplus T_{j,i}) \neq E_{K_1}(P_{j',i'}\oplus T_{j',i'})
$$

不会再因为明文块本身相同而直接落到同一密文块。

这就是 XTS 相对 ECB 的真正修正：

> 它没有让明文块消失，而是让“同一明文块”不再总在同一坐标系里被加密。

### 2.4 tweak 不是认证标签

这里很容易发生一个误解：既然 tweak 编码了位置，XTS 是不是就“绑定了位置”，因此也就提供某种完整性？

答案是否定的。tweak 只是让

$$
\widetilde{E}_K(T,\cdot)
$$

在不同 \(T\) 下成为不同置换族，它并不附带一个

$$
\mathcal{D}_K(T,C)\in\{P,\bot\}
$$

这样的 reject 语义。XTS 的解密总是输出某个 block：

$$
P_i = D_{K_1}(C_i \oplus T_i)\oplus T_i.
$$

若把

$$
\tau_T(X)=X\oplus T
$$

记成一个按 tweak 平移的 involution，则 XTS 在固定 tweak \(T\) 下就是共轭置换族

$$
\pi_T = \tau_T \circ E_{K_1}\circ \tau_T,
\qquad
\pi_T^{-1} = \tau_T \circ D_{K_1}\circ \tau_T.
$$

因此对每个固定 \(T\)，\(\pi_T\) 都是整个 \(\{0,1\}^{128}\) 上的双射：

$$
\forall C\in\{0,1\}^{128},\quad \exists! P\in\{0,1\}^{128}\text{ such that } \pi_T(P)=C.
$$

这句量词形式已经把边界说死了：每个 128-bit 串都是某个合法明文在该 tweak 下的像，所以 XTS 没有“非法 ciphertext 集”，也就没有 tag-based reject surface。

因此它仍然属于：

- confidentiality mode
- tweakable enciphering mode

而不是：

- authenticated encryption mode

## 3. XTS 解决了什么；没有解决什么

### 3.1 它解决的是位置相关结构泄漏

XTS 对 ECB 的改进可以精确概括成：

- 相同位置不同数据单元的相同块，不再直接映成同一密文
- 同一数据单元内不同偏移上的相同块，不再直接映成同一密文
- 磁盘映像中的大块重复模式被显著削弱

因此 XTS 真正提升的是 `storage confidentiality under position tweaks`。

### 3.2 它没有给出 reject 语义

对 AEAD 来说，最重要的安全边界之一是：

$$
\mathcal{D}_K(N,A,C,T)\in\{P,\bot\}.
$$

XTS 没有这层接口。对任意覆写后的 \(C_i'\)，系统只会得到某个

$$
P_i' = D_{K_1}(C_i' \oplus T_i)\oplus T_i.
$$

所以攻击者虽然通常不能精确控制 \(P_i'\) 的内容，但它可以稳定造成：

- corruption
- silent misdecode
- parser confusion
- application-level state damage

这是“没有 authentication”最直接的含义。

### 3.3 它没有给出 freshness

若某个位置 \((j,i)\) 在时间 \(t_0\) 上的合法密文为

$$
C_{j,i}^{(0)},
$$

在时间 \(t_1\) 上更新为

$$
C_{j,i}^{(1)},
$$

攻击者只要把旧值

$$
C_{j,i}^{(0)}
$$

重新写回相同位置，解密方仍会得到对应的旧明文

$$
P_{j,i}^{(0)}.
$$

因为 tweak 没变：

$$
T_{j,i}^{(0)} = T_{j,i}^{(1)}.
$$

更严格地写，若

$$
C_{j,i}^{(0)} = E_{K_1}(P_{j,i}^{(0)}\oplus T_{j,i})\oplus T_{j,i},
$$

那么攻击者在时间 \(t_1\) 把同一串 \(C_{j,i}^{(0)}\) 写回相同位置后，解密结果满足

$$
D_{K_1}(C_{j,i}^{(0)}\oplus T_{j,i})\oplus T_{j,i}
\;=\; D_{K_1}(E_{K_1}(P_{j,i}^{(0)}\oplus T_{j,i}))\oplus T_{j,i}
\;=\; P_{j,i}^{(0)}.
$$

这不是“有概率回到旧值”，而是一个逐块恒等式。也就是说，同位置 replay 在密码层面的效果正是把过去的合法状态精确恢复为当前读出的明文。

而 XTS 没有任何额外 tag 或 version binding 去区分“这是不是当前版本”。

所以：

$$
\text{XTS confidentiality} \neq \text{freshness}.
$$

## 4. 主攻击面：replay、rollback、sector relocation

### 4.1 replay：同一逻辑位置上的旧值回放

这是 XTS 最基本、也最本质的缺口。

攻击者若保存某个 sector 在过去时刻的合法密文镜像

$$
C_j^{\text{old}},
$$

之后再把它整段写回同一 sector number \(j\)，则 base tweak 仍然是

$$
T_0 = E_{K_2}(j).
$$

于是整段数据会被正常解密成旧明文。这里不存在任何密码层面的 \(\bot\)。

因此 replay 在 XTS 里不是 corner case，而是模式设计没有试图解决的问题。

### 4.2 rollback：整盘或子树级别的旧快照回放

rollback 本质上是 replay 的系统级版本。若攻击者能够恢复一组旧 ciphertext image 与相应 metadata mapping，则：

$$
\text{current state} \longrightarrow \text{old but valid encrypted state}
$$

不会被 XTS 检测。

这一点必须和 confidentiality 区分开。rollback 不一定泄漏新的明文信息，但它破坏了：

- state continuity
- freshness
- version monotonicity
- update authenticity

因此 rollback 的失败不是“XTS 没藏住数据内容”，而是“XTS 从未承诺它能区分现在和过去”。

### 4.3 sector relocation：什么会被 tweak 挡住，什么不会

如果攻击者把一个 ciphertext block 从 \((j,i)\) 搬到另一个不同位置 \((j',i')\)，则解密使用的是不同 tweak：

$$
T_{j',i'} \neq T_{j,i}.
$$

因此得到的明文通常是

$$
P'_{j',i'} = D_{K_1}(C_{j,i}\oplus T_{j',i'})\oplus T_{j',i'},
$$

它通常不会等于原始

$$
P_{j,i}.
$$

把原式

$$
C_{j,i} = E_{K_1}(P_{j,i}\oplus T_{j,i})\oplus T_{j,i}
$$

代回去，并记

$$
\Delta = T_{j,i}\oplus T_{j',i'},
$$

则

$$
P'_{j',i'}
\;=\; D_{K_1}(E_{K_1}(P_{j,i}\oplus T_{j,i})\oplus \Delta)\oplus T_{j',i'}.
$$

若 \(\Delta=0\)，这当然退化回同位置 replay；但当 \(\Delta\neq 0\) 时，攻击者面对的是

$$
D_{K_1}(X\oplus \Delta)
$$

而不是

$$
D_{K_1}(X)\oplus \Delta.
$$

对一般分组密码置换并不存在

$$
D_{K_1}(X\oplus \Delta)=D_{K_1}(X)\oplus \Delta
$$

这样的交换律，所以跨 tweak 的 cut-and-paste 不会保持原始明文语义。换句话说，XTS 真正打掉的是“同一密文块可被无损搬运到别的位置继续解释成同一内容”。

所以 pure cut-and-paste across different logical positions 往往不能像 ECB 那样语义保持。这就是 tweak 真正挡住的东西。

但这里不能推出“XTS 阻止了 relocation attack”这种过强结论。更准确的说法是：

- 它阻止了不同 tweak 位置之间的明文语义保持式直接搬运。
- 它没有提供 source authentication。
- 它也无法阻止攻击者把某个旧 sector image 重新绑定回它原本所属的 logical position。

因此 XTS 抵抗的是 `cross-position pattern equality`，不是所有形式的 state substitution。

### 4.4 bit corruption 与 targeted forgery 的区别

在 CTR/CBC 一类 message mode 里，我们关心的是 bit flip 是否可控。XTS 里更常见的是另一种风险：攻击者未必能让明文精确变成某个目标值，但它能让某个 block 落成近似随机值，从而：

- 损坏文件系统元数据
- 损坏 inode / extent / allocation bitmap
- 损坏应用层数据库页

因此对存储来说，“攻击者不能精确控制目标明文”并不等于“攻击没用”。silent corruption 本身就是严重破坏。

## 5. 为什么 XTS 不等于 authenticated storage

### 5.1 authenticated storage 至少还缺什么

若想从 storage confidentiality 走到 authenticated storage，至少还需要附加某种机制去绑定：

- 当前版本
- 数据来源
- 完整性校验
- tree/path consistency

抽象地说，需要的不只是

$$
\widetilde{E}_K(T,P)=C,
$$

还需要某种验证对象 \(V\)，使读出时能判断：

$$
\text{is this ciphertext the current legitimate one for this logical position?}
$$

若进一步把“当前版本”也显式建模成一个状态变量 \(\nu\)，那么 authenticated storage 至少需要某个验证谓词

$$
\mathcal{V}_K(\nu,T,C,\sigma)\in\{0,1\},
$$

其中 \(\sigma\) 可以是 tag、Merkle path、版本计数器绑定或其他完整性元数据，并且希望满足类似

$$
\mathcal{V}_K(\nu_0,T,C,\sigma)=1
\not\Rightarrow
\mathcal{V}_K(\nu_1,T,C,\sigma)=1
\qquad (\nu_0\neq \nu_1)
$$

的 freshness 约束。XTS 单独一层既不给 \(\sigma\)，也不给 \(\nu\) 的验证入口，所以它无法表达“过去合法，但现在不合法”这件事。

而 XTS 没有提供这个 \(V\)。

### 5.2 消息 AEAD 也不能直接替代 XTS

这里还要避免另一个对称误解：既然 XTS 不认证，那是不是直接用 AEAD 包每个 sector 就完了？

这同样不能一句话带过。原因在于 message AEAD 的自然接口是 transcript-oriented，而 storage encryption 需要：

- fixed-size data units
- random access
- 局部写入
- 与地址空间绑定的 tweak/position handling

也就是说，XTS 和 message AEAD 不只是“一个有 tag，一个没 tag”的关系，它们面对的是不同的访问模型。authenticated storage 往往需要的是：

$$
\text{tweakable confidentiality}
\quad + \quad
\text{integrity/freshness composition}.
$$

这已经是一个组合问题，而不是单个 mode 名称能独立解决的问题。

### 5.3 storage confidentiality 与 authenticated storage 的区别

可以把这两类目标写成两个完全不同的问题。

第一类：

> 给定当前位置标识 \(T\)，如何阻止不同位置上的相同明文块继续暴露相等关系？

第二类：

> 给定当前位置标识 \(T\)，如何阻止旧状态、错误来源或被替换的数据在未来仍被当作合法当前状态接受？

XTS 回答的是第一类，不回答第二类。

因此最危险的误用不是“XTS 的数学错了”，而是：

> 系统把一个只承诺 storage confidentiality 的模式，当成了 authenticated storage 的全部答案。

## 6. 统一收束：Vol.1 到 Vol.4 的边界图

这条讲义线到 Vol.4 为止，可以压成四层边界：

### 6.1 Vol.1：malleability boundary

没有认证的消息模式会在解密方程里留下可被攻击者利用的结构。

### 6.2 Vol.2：authentication boundary

AEAD 通过 reject 语义与认证方程把这些主动攻击面切掉，但前提是 unique nonce。

### 6.3 Vol.3：misuse boundary

SIV / GCM-SIV 把 repeated nonce 的灾难性崩塌降级成可界定退化，例如 equality leakage。

### 6.4 Vol.4：storage-model boundary

XTS 说明，即使消息加密里的认证与 misuse 边界都讨论完了，存储模型里仍然有一层独立问题：

- 位置是否被编码
- 旧状态是否会被接受
- 来源与版本是否被认证

也就是说，消息 AEAD 的世界和存储机密性的世界并不是简单包含关系。

若继续向后扩展，这条线自然会走向更高一层的主题：

- robust AE
- release of unverified plaintext
- key commitment
- authenticated storage composition

但在 XTS 这里，最重要的结论已经够明确：

> tweakable enciphering 解决的是“同一内容在不同位置不应长得一样”，不是“旧内容永远不能回来”，更不是“存储状态天然被认证”。

## References

- NIST SP 800-38E, [*Recommendation for Block Cipher Modes of Operation: The XTS-AES Mode for Confidentiality on Storage Devices*](https://csrc.nist.gov/pubs/sp/800/38/e/final)
- Phillip Rogaway, [*Efficient Instantiations of Tweakable Blockciphers and Refinements to Modes OCB and PMAC*](https://www.cs.ucdavis.edu/~rogaway/papers/offsets.pdf)
- Phillip Rogaway and Mark Wooding, [*Evaluation of Some Blockcipher Modes of Operation*](https://www.cs.ucdavis.edu/~rogaway/papers/modes.pdf)

