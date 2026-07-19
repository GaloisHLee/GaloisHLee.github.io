# 分组密码工作模式安全性分析 Part 1：攻击者视角


> Threat Model: 这里不介绍 AES/SM4 的轮函数，也不讨论如何实现加密 API。默认底层分组密码 \(E_K\) 是强伪随机置换，攻击面来自 mode of operation 本身。

分组密码模式的安全性问题，本质是：如何把固定长度置换 \(E_K:\{0,1\}^n\to\{0,1\}^n\) 包装成多块消息加密、流式加密、磁盘加密或 AEAD，并且在攻击者能观察、选择、篡改、重放输入输出时仍保持边界清晰。

攻击者关心的不是“CBC 是什么”，而是：CBC 的哪个 XOR 可以被控制，CTR 的哪个 nonce 一旦重复就使两条消息相消，GCM 的哪个认证多项式会在 nonce reuse 后暴露方程，XTS 为什么解决磁盘 ECB 轮廓但仍不认证。

<!--more-->

## 0. 统一安全模型

把底层分组密码写成：

$$
\begin{aligned}
E_K &: \{0,1\}^n\to\{0,1\}^n,\\
D_K &: \{0,1\}^n\to\{0,1\}^n,\\
D_K(E_K(X))&=X.
\end{aligned}
$$

AES 与 SM4 的 block size 都是 \(n=128\)。本文故意不进入它们的内部结构，因为大多数 mode attack 不需要破坏 \(E_K\)。攻击者通常只利用 mode 方程：

$$
C=P\oplus S
$$

或者：

$$
P_i=D_K(C_i)\oplus C_{i-1}.
$$

这类攻击的目标通常是：

- 恢复明文，而不是恢复 key。
- 修改明文语义，而不是解密所有消息。
- 伪造 tag 或绕过认证，而不是求出 \(K\)。
- 利用错误信息、时间差、parser 行为，而不是直接调用理想解密 oracle。

## 1. 攻击者真正检查什么

一个 mode 是否安全，不能只问“用了 AES 吗”。至少要问：

| 问题 | 攻击者视角 |
| --- | --- |
| 是否隐藏相等关系 | 相同 block、相同字段、相同图片区域是否可识别 |
| 是否需要 IV/nonce | 它要求随机、唯一、不可预测，还是三者都有 |
| IV/nonce 误用后果 | 是泄漏第一块，还是全流 two-time pad，还是认证 key 暴露 |
| 是否可塑 | 改 ciphertext 是否能按目标改 plaintext |
| 是否认证 | 能否检测 bit flip、replay、reorder、truncate |
| 是否有 padding | padding validity 是否变成 oracle |
| 是否支持 random access | 随机访问接口是否变成 edit oracle |
| 错误是否可观察 | 错误内容、HTTP status、连接关闭、时间差是否可区分 |

可以把经典安全目标粗略分成：

| 安全目标 | 含义 | 对攻击者意味着什么 |
| --- | --- | --- |
| IND-CPA | chosen plaintext 下隐藏明文 | 只能加密查询时仍不能区分挑战消息 |
| IND-CCA | chosen ciphertext 下隐藏明文 | 能提交篡改密文解密时仍不能获益 |
| INT-CTXT | ciphertext integrity | 不能构造新的可接受密文 |
| AEAD | authenticated encryption with associated data | 同时保护保密性、完整性与 AAD 绑定 |

ECB/CBC/CFB/OFB/CTR 通常只可能讨论 IND-CPA，且还依赖正确 IV/nonce。它们默认不提供 IND-CCA 或 INT-CTXT。GCM/CCM/OCB 是 AEAD，但 nonce discipline 很硬。SIV/GCM-SIV 则牺牲一部分确定性泄漏，换取 nonce misuse resistance。

## 2. ECB：确定性泄漏不是小问题

ECB 的定义最简单：

$$
C_i=E_K(P_i).
$$

解密：

$$
P_i=D_K(C_i).
$$

没有 IV。没有 nonce。没有反馈。没有 tag。每个 block 独立。

### 2.1 等值关系直接泄漏

因为 \(E_K\) 是置换，所以有双向等价：

$$
P_i=P_j \Longleftrightarrow C_i=C_j.
$$

证明很短。若 \(P_i=P_j\)，显然：

$$
E_K(P_i)=E_K(P_j).
$$

反过来，若 \(C_i=C_j\)，则：

$$
E_K(P_i)=E_K(P_j).
$$

由于 \(E_K\) injective：

$$
P_i=P_j.
$$

所以 ECB 泄漏的不是“某个 bit 的值”，而是整个消息的 block partition：

$$
\Pi(P)=\{\{i:P_i=X\}:X\in\{P_1,\ldots,P_m\}\}.
$$

这就是企鹅图仍能看出轮廓的原因。图片中相同颜色区域经过分块后产生大量相同 \(P_i\)，ECB 把这些相等关系原样搬到 \(C_i\) 上。攻击者不需要知道颜色值，只要把相同密文块染成相同颜色，就能恢复结构。

### 2.2 ECB 不满足 IND-CPA

IND-CPA 挑战中，攻击者选择两条等长消息：

$$
M_0=X\|X,\quad M_1=X\|Y,\quad X\ne Y.
$$

挑战者返回：

$$
C^\star=\operatorname{Enc}_K(M_b).
$$

攻击者只检查两个 block 是否相等：

$$
C_1^\star=C_2^\star.
$$

若相等，猜 \(b=0\)；否则猜 \(b=1\)。因为 ECB 保留相等关系，攻击者成功率：

$$
\Pr[\operatorname{win}]=1.
$$

这说明 ECB 在最基本 chosen-plaintext 安全性下失败。注意这里没有恢复 key，也没有利用 AES 弱点。

### 2.3 Codebook 与数据库泄漏

如果明文字段来自小集合 \(\mathcal{D}\)，攻击者可以通过加密 oracle 建表：

$$
\mathsf{Book}[E_K(X)]=X,\quad X\in\mathcal{D}.
$$

之后看到某个数据库密文块 \(C_i\)，若：

$$
C_i\in\operatorname{dom}(\mathsf{Book}),
$$

则直接得到：

$$
P_i=\mathsf{Book}[C_i].
$$

危险字段包括布尔值、角色、状态码、邮编、日期、固定 JSON 片段、低熵枚举。ECB 对低熵结构化数据尤其致命。

### 2.4 ECB 的可塑性边界

ECB 的 bit flip 不可控。若攻击者把 \(C_i\) 改为 \(C_i'\)，解密为：

$$
P_i'=D_K(C_i').
$$

在强 PRP 假设下，\(P_i'\) 对攻击者近似随机。攻击者不能像 CTR 那样精确翻转某个 bit。

但 ECB 支持 block replay 和 block swap。若攻击者从合法密文中获得某个 block：

$$
C_j^{admin}=E_K(P_j^{admin}),
$$

就可以把它复制到另一条密文的 block 位置。解密端得到：

$$
D_K(C_j^{admin})=P_j^{admin}.
$$

所以 ECB 是“bit 层不可控，block 层高度可拼接”。

## 3. CBC：随机化保密与线性可塑并存

CBC 加密：

$$
\begin{aligned}
C_0&=IV,\\
C_i&=E_K(P_i\oplus C_{i-1}).
\end{aligned}
$$

解密：

$$
P_i=D_K(C_i)\oplus C_{i-1}.
$$

CBC 的设计目的，是让相同明文块在不同上下文中进入 \(E_K\) 前变成不同输入：

$$
X_i=P_i\oplus C_{i-1}.
$$

如果 \(IV\) 随机不可预测，第一块被随机化；后续块由前一密文随机化。

### 3.1 CBC 的 IV 为什么必须不可预测

固定 IV 会让第一块退化出 ECB 式等值泄漏：

$$
P_1=P_1'\Rightarrow E_K(P_1\oplus IV)=E_K(P_1'\oplus IV)\Rightarrow C_1=C_1'.
$$

仅仅“不重复”也不够。若 IV 可预测，chosen-plaintext 攻击者可以制造输入碰撞。设已有消息第一块为 \(P_1\)，IV 为 \(V\)，下一次 IV 可预测为 \(V'\)。攻击者选择：

$$
P_1'=P_1\oplus V\oplus V'.
$$

则：

$$
P_1'\oplus V'=P_1\oplus V.
$$

因此：

$$
C_1'=E_K(P_1'\oplus V')=E_K(P_1\oplus V)=C_1.
$$

这就是 CBC 和 CTR 的 nonce 要求不同之处。CTR 通常要求 counter block 唯一；CBC 的 IV 在 CPA 模型中还要不可预测。

### 3.2 CBC 的可塑性来自解密等式

定义 intermediate value：

$$
I_i=D_K(C_i).
$$

则：

$$
P_i=I_i\oplus C_{i-1}.
$$

攻击者不知道 \(I_i\)，但 \(C_{i-1}\) 是密文的一部分，通常可被修改。若攻击者令：

$$
C_{i-1}'=C_{i-1}\oplus\Delta,
$$

则目标块解密为：

$$
\begin{aligned}
P_i'&=I_i\oplus C_{i-1}'\\
&=I_i\oplus C_{i-1}\oplus\Delta\\
&=P_i\oplus\Delta.
\end{aligned}
$$

所以攻击者可以精确控制 \(P_i\) 的差分：

$$
\Delta=P_i\oplus P_i'.
$$

例如想把 `admin=false` 改成 `admin=true;`，只要知道字段位置，就对前一密文块对应字节加：

$$
\Delta=\texttt{false}\oplus\texttt{true;}.
$$

限制是：被修改的 \(C_{i-1}'\) 同时也是上一块的输入，因此：

$$
P_{i-1}'=D_K(C_{i-1}')\oplus C_{i-2}.
$$

这个值通常不可控并近似随机。因此 CBC bit flipping 常要求上一块是可牺牲区域、注释、padding-like 垃圾、parser 忽略字段，或者目标是第一块并且 IV 未认证。

### 3.3 IV manipulation 是第一块 bit flipping

第一块解密：

$$
P_1=D_K(C_1)\oplus IV.
$$

如果 IV 没有认证，攻击者令：

$$
IV'=IV\oplus P_1\oplus P_1'.
$$

则：

$$
\begin{aligned}
P_1'&=D_K(C_1)\oplus IV'\\
&=D_K(C_1)\oplus IV\oplus P_1\oplus P_1'\\
&=P_1\oplus P_1\oplus P_1'\\
&=P_1'.
\end{aligned}
$$

这说明 IV 不是“公开所以无所谓”。IV 可以公开，但必须被完整性保护。

### 3.4 Padding Oracle 的完整推导

CBC 加 PKCS#7 padding 时，最后一个 block 后缀合法形式为：

$$
\underbrace{t,t,\ldots,t}_{t\text{ bytes}},\quad 1\le t\le b.
$$

其中 \(b\) 是 block size bytes。Padding oracle 是一个谓词：

$$
\mathcal{O}(C)=
\begin{cases}
1,& \operatorname{Unpad}(\operatorname{Dec}_K(C))\text{ succeeds},\\
0,& \text{otherwise}.
\end{cases}
$$

考虑目标 block \(C_i\)。攻击者构造伪前块 \(G\)，提交：

$$
G\|C_i.
$$

解密目标块：

$$
P_i^G=D_K(C_i)\oplus G=I_i\oplus G.
$$

目标是恢复 \(I_i\)，因为真实明文为：

$$
P_i=I_i\oplus C_{i-1}.
$$

先恢复最后一个字节。希望 padding 为 `0x01`：

$$
P_i^G[b-1]=1.
$$

也就是：

$$
I_i[b-1]\oplus G[b-1]=1.
$$

枚举 \(G[b-1]=g\)。当 oracle 返回 1：

$$
I_i[b-1]=g\oplus 1.
$$

于是：

$$
P_i[b-1]=I_i[b-1]\oplus C_{i-1}[b-1].
$$

恢复倒数第 \(r\) 个字节时，已经知道后面 \(r-1\) 个 intermediate bytes。为了强制后缀成为 \(r\) 个 `0x r`，令：

$$
G[\ell]=I_i[\ell]\oplus r,\quad \ell>b-r.
$$

然后枚举当前位置 \(j=b-r\) 的 \(G[j]\)。当：

$$
\mathcal{O}(G\|C_i)=1
$$

成立时：

$$
I_i[j]=G[j]\oplus r.
$$

最终得到：

$$
P_i[j]=I_i[j]\oplus C_{i-1}[j].
$$

每个字节最多 256 次查询，一个 AES block 最坏：

$$
256\cdot16=4096
$$

次查询。现实攻击还要处理网络噪声、false positive、限速和时间差，但数学核心就是这个一字节谓词递推。

### 3.5 为什么 Padding Oracle 不恢复 key

Padding oracle 恢复的是某个目标密文块上的：

$$
I_i=D_K(C_i).
$$

这不是 \(D_K\) 的完整算法，也不是 \(K\)。换一个 \(C_j\)，攻击者需要重新查询恢复：

$$
I_j=D_K(C_j).
$$

所以它是 plaintext recovery attack，不是 key recovery attack。它破坏了 IND-CCA，因为解密错误接口把 \(D_K(C_i)\) 的信息一点点泄漏出来。

## 4. CFB：自同步流模式的可塑性

Full-block CFB 定义：

$$
\begin{aligned}
S_i&=E_K(C_{i-1}),\quad C_0=IV,\\
C_i&=P_i\oplus S_i.
\end{aligned}
$$

解密：

$$
P_i=C_i\oplus E_K(C_{i-1}).
$$

CFB 把分组密码变成 keystream generator，但反馈的是 ciphertext。

### 4.1 CFB 的 CPA 条件

如果 IV 随机不可预测，则第一块 keystream：

$$
S_1=E_K(IV)
$$

对攻击者不可预测。后续 keystream 由前一密文决定：

$$
S_i=E_K(C_{i-1}).
$$

在强 PRP 假设下，CFB 可以达到被动保密意义上的 IND-CPA。但这个结论不包含完整性。

### 4.2 CFB 的 bit flip

攻击者修改当前密文块：

$$
C_i'=C_i\oplus\Delta.
$$

当前明文变成：

$$
\begin{aligned}
P_i'&=C_i'\oplus E_K(C_{i-1})\\
&=C_i\oplus\Delta\oplus E_K(C_{i-1})\\
&=P_i\oplus\Delta.
\end{aligned}
$$

所以 CFB 当前块可控翻转。

但是下一块会被破坏：

$$
P_{i+1}'=C_{i+1}\oplus E_K(C_i').
$$

因为 \(C_i'\ne C_i\)，\(E_K(C_i')\) 对攻击者近似随机，所以 \(P_{i+1}'\) 通常不可控。full-block CFB 在再下一块重新同步，因为 \(C_{i+1}\) 没变。

### 4.3 CFB 的 IV 重用

若两条消息同 key 同 IV：

$$
S_1=E_K(IV)
$$

相同。于是第一块：

$$
\begin{aligned}
C_1\oplus C_1'&=(P_1\oplus S_1)\oplus(P_1'\oplus S_1)\\
&=P_1\oplus P_1'.
\end{aligned}
$$

第一块成为 two-time pad。第二块是否继续重用，取决于：

$$
C_1=C_1'.
$$

如果第一块密文不同，反馈状态分叉，后续 keystream 不再相同。CFB 的 IV 重用通常先破第一块，而不像 OFB/CTR 那样整流直接复用。

### 4.4 CFB 的安全边界

CFB 没有 padding，所以少了经典 padding oracle。但它仍不满足 IND-CCA，因为攻击者可以构造有意义的明文差分。如果系统在认证前解析明文，或者 parser 容忍一个损坏块，CFB 的 malleability 就能变成实际攻击。

## 5. OFB：状态独立导致 IV 重用灾难

OFB 定义：

$$
\begin{aligned}
S_0&=IV,\\
S_i&=E_K(S_{i-1}),\\
C_i&=P_i\oplus S_i.
\end{aligned}
$$

OFB 的状态不依赖明文，也不依赖密文。它先生成一条 keystream，再与明文 XOR。

### 5.1 OFB 本质是同步流密码

定义：

$$
\mathsf{KS}_K(IV,i)=E_K^{(i)}(IV).
$$

则：

$$
C_i=P_i\oplus \mathsf{KS}_K(IV,i).
$$

如果同一 key 下 IV 重复，则整条 keystream 重复：

$$
\mathsf{KS}_K(IV,i)=\mathsf{KS}_K(IV',i)
$$

其中 \(IV=IV'\)。

于是：

$$
\begin{aligned}
C_i\oplus C_i'&=(P_i\oplus S_i)\oplus(P_i'\oplus S_i)\\
&=P_i\oplus P_i'.
\end{aligned}
$$

这就是 two-time pad。

### 5.2 已知明文恢复另一条消息

如果攻击者知道一条明文 \(P_i\)，就能得到：

$$
S_i=C_i\oplus P_i.
$$

另一条同 IV 密文解密为：

$$
P_i'=C_i'\oplus S_i.
$$

代入：

$$
P_i'=C_i'\oplus C_i\oplus P_i.
$$

这说明 OFB 一旦 IV 重用，失败是全局的、确定的、直接的。

### 5.3 OFB 的 bit flip

若攻击者令：

$$
C_i'=C_i\oplus\Delta,
$$

则：

$$
P_i'=C_i'\oplus S_i=P_i\oplus\Delta.
$$

OFB 的密文不反馈进状态，因此这个修改不影响其他 block。传输错误只影响对应 bit；主动攻击者也能只改对应 bit。

### 5.4 OFB 的 forward/backward 边界

如果某一时刻状态 \(S_i\) 泄漏，攻击者可以计算未来：

$$
S_{i+1}=E_K(S_i),\quad S_{i+2}=E_K(S_{i+1}).
$$

但若不知道 key，不能从 \(S_i\) 反推 \(S_{i-1}\)，因为需要计算：

$$
D_K(S_i).
$$

所以 OFB 的状态泄漏具有 forward compromise 风险，但不自动给出过去流。工程上仍然不能把内部状态暴露或复用。

## 6. CTR：随机访问的代价是 nonce discipline

CTR 定义：

$$
\begin{aligned}
S_i&=E_K(N\|\operatorname{ctr}_i),\\
C_i&=P_i\oplus S_i.
\end{aligned}
$$

解密相同：

$$
P_i=C_i\oplus E_K(N\|\operatorname{ctr}_i).
$$

CTR 的安全条件不是“nonce 看起来随机”，而是同一 key 下所有 block-cipher input 唯一：

$$
(N,i)\ne(N',j)\Rightarrow N\|\operatorname{ctr}_i\ne N'\|\operatorname{ctr}_j.
$$

### 6.1 CTR 与流密码等价

定义：

$$
\mathsf{KS}_K(N,i)=E_K(N\|\operatorname{ctr}_i).
$$

则 CTR 就是：

$$
C=P\oplus\mathsf{KS}_K(N).
$$

因此所有同步流密码的 nonce reuse 风险都原样成立。

### 6.2 Nonce reuse 推导

两条消息使用相同 key、nonce、counter 初值：

$$
\begin{aligned}
C_i&=P_i\oplus E_K(N\|i),\\
C_i'&=P_i'\oplus E_K(N\|i).
\end{aligned}
$$

相 XOR：

$$
\begin{aligned}
C_i\oplus C_i'&=P_i\oplus E_K(N\|i)\oplus P_i'\oplus E_K(N\|i)\\
&=P_i\oplus P_i'.
\end{aligned}
$$

若知道 \(P_i\)，恢复：

$$
P_i'=C_i\oplus C_i'\oplus P_i.
$$

若攻击者有 chosen plaintext，并能强制 nonce 重用，提交：

$$
P_i=0^n.
$$

则：

$$
C_i=E_K(N\|i)=S_i.
$$

攻击者直接得到 keystream。

### 6.3 CTR 的 bit flip 与 edit oracle

CTR bit flip：

$$
C_i'=C_i\oplus\Delta.
$$

解密：

$$
P_i'=C_i'\oplus S_i=P_i\oplus\Delta.
$$

没有相邻块损坏。这使 CTR 在无认证时极易被改字段、金额、权限。

CTR 支持 random access，因为第 \(i\) 块只依赖：

$$
N\|\operatorname{ctr}_i.
$$

如果系统暴露 edit oracle：

$$
\operatorname{edit}(C,\operatorname{offset},P_{\text{new}})\to C_{\text{new}},
$$

攻击者把目标区间改成全零。返回密文在该区间等于 keystream：

$$
C_{\text{new}}=0^\ell\oplus S=S.
$$

于是原明文：

$$
P=C\oplus S.
$$

所以 CTR 的随机访问能力必须和认证、授权、上下文绑定一起设计。

### 6.4 Counter collision

CTR 不只怕 nonce 重复，也怕 counter block 碰撞。若：

$$
N\|\operatorname{ctr}_i=N'\|\operatorname{ctr}_j,
$$

则：

$$
S_i=S_j'.
$$

造成局部 two-time pad。常见来源：

- counter 位数太小导致 wrap。
- 多进程或重启后 nonce 状态回滚。
- 大端/小端编码不一致。
- user-controlled nonce 未查重。
- 随机 nonce 位数太短，生日碰撞。

若 nonce 是 \(r\)-bit 随机数，发送 \(Q\) 条消息，碰撞概率近似：

$$
\Pr[\operatorname{collision}]\approx1-\exp\left(-\frac{Q(Q-1)}{2^{r+1}}\right).
$$

这说明高吞吐系统不能用小 nonce 空间赌概率。

## 7. GCM：CTR 保密性加 GHASH 认证

GCM 可以粗略写成：

$$
\operatorname{GCM}=\operatorname{CTR encryption}+\operatorname{GHASH authentication}.
$$

加密部分：

$$
C_i=P_i\oplus E_K(J_i).
$$

认证部分：

$$
T=E_K(J_0)\oplus GHASH_H(A,C).
$$

其中：

$$
H=E_K(0^{128}).
$$

\(A\) 是 AAD，\(C\) 是 ciphertext，\(T\) 是 tag。

### 7.1 GCM nonce reuse 的两层失败

第一层继承 CTR。若 nonce 重复，则 keystream 重复：

$$
C\oplus C'=P\oplus P'.
$$

第二层更严重：认证也复用了同一个：

$$
E_K(J_0)
$$

和同一个 hash key：

$$
H=E_K(0^{128}).
$$

两条 tag 相 XOR：

$$
T\oplus T'=GHASH_H(A,C)\oplus GHASH_H(A',C').
$$

因为 \(GHASH_H\) 是 \(GF(2^{128})\) 上的多项式求值，右边会形成关于 \(H\) 的方程。攻击者若知道或控制足够多输入，就可能恢复 \(H\) 或缩小候选集合，从而伪造 tag。

这就是 GCM 的 Forbidden Attack 直觉：nonce reuse 不仅破坏保密性，还破坏认证结构。

### 7.2 GCM 为什么不能只说“CTR 加 MAC”

GCM 的认证不是普通黑盒 HMAC，而是 universal hash：

$$
GHASH_H(X_1,\ldots,X_m)=\sum_{i=1}^{m}X_iH^{m-i+1}\oplus L H.
$$

这里 \(L\) 编码 AAD 和 ciphertext 长度。tag：

$$
T=E_K(J_0)\oplus GHASH_H(A,C).
$$

只要 nonce 唯一，\(E_K(J_0)\) 类似一次性 pad，GHASH 的 universal property 给出强认证界。但 nonce 重复时，两个 tag 的 \(E_K(J_0)\) 抵消，攻击者直接看到 GHASH 差值：

$$
T\oplus T'=\Delta GHASH_H.
$$

这会把认证问题转成代数问题。

### 7.3 GCM 的安全边界

GCM 正确使用时是 AEAD，提供：

- plaintext confidentiality
- ciphertext integrity
- AAD integrity
- length binding

但前提是：

- 同 key 下 nonce 不重复。
- tag 必须验证失败即拒绝。
- AAD 覆盖协议上下文，例如方向、版本、序号、连接 ID。
- 不允许释放未认证明文。

GCM 的 nonce 可以公开，但不能重复。公开不是问题，重复才是问题。

## 8. CCM：CTR 加 CBC-MAC

CCM 组合：

$$
\operatorname{CCM}=\operatorname{CTR encryption}+\operatorname{CBC\text{-}MAC}.
$$

CTR 提供保密：

$$
C=P\oplus S.
$$

CBC-MAC 提供认证，覆盖 nonce、AAD、长度和明文相关数据。

### 8.1 CCM 的 nonce misuse

CCM 的保密部分仍是 CTR，因此 nonce 重复时：

$$
C\oplus C'=P\oplus P'.
$$

这已经足够严重。与 GCM 相比，CCM 不会以完全相同的 GHASH 多项式形式暴露 \(H\)，但 nonce misuse 仍破坏核心保密边界。

### 8.2 CCM 的工程脆弱点

CCM 的难点常在编码而不是公式：

- nonce 长度影响 message length encoding。
- AAD 长度编码必须唯一且无歧义。
- CBC-MAC 必须绑定 nonce 与长度。
- CTR counter block 不能与 MAC 初始块混淆。

如果实现者手写 CCM，很容易在域分离和长度编码上犯错。正确做法是使用经过验证的 AEAD API。

## 9. XTS：磁盘加密不是 AEAD

XTS 面向磁盘 sector。它是 tweakable encryption，不是普通消息 AEAD。

简化公式：

$$
C_i=E_{K_1}(P_i\oplus T_i)\oplus T_i.
$$

tweak：

$$
T_i=\alpha^iE_{K_2}(\operatorname{sector}).
$$

这里乘法发生在 \(GF(2^{128})\) 中。

### 9.1 XTS 解决什么

ECB 用在磁盘上会泄漏相同 sector offset 的相同 block。XTS 通过 sector number 和 block index 生成 tweak，使相同 plaintext block 在不同位置得到不同密文：

$$
(sector,i)\ne(sector',j)\Rightarrow T_i\ne T_j'.
$$

于是：

$$
E_{K_1}(P\oplus T_i)\oplus T_i
$$

不会因为 \(P\) 相同而直接相同。

### 9.2 XTS 不解决什么

XTS 不认证。攻击者仍可能：

- replay 旧 sector。
- 把某个位置的旧密文写回同一位置。
- 篡改密文导致对应明文随机损坏。
- 利用文件系统缺少完整性检测造成 rollback。

XTS 的目标是 confidentiality for storage under tweakable model，不是 INT-CTXT。磁盘防篡改需要认证树、MAC、fs-verity、dm-integrity 或上层完整性机制。

## 10. SIV：把 nonce misuse 变成确定性泄漏

SIV, Synthetic IV, 先计算 synthetic IV：

$$
S=\operatorname{PRF}_K(A,P).
$$

再用 \(S\) 做 CTR 类加密：

$$
C=P\oplus\mathsf{KS}_{K'}(S).
$$

### 10.1 SIV 的核心取舍

CTR/GCM 中，外部 nonce 重复会导致 keystream 重复。SIV 中，真正用于生成 keystream 的 \(S\) 依赖 \(A,P\)。如果外部 nonce 重复，但消息不同，synthetic IV 通常不同。

代价是确定性：

$$
(A,P)=(A',P')\Rightarrow (S,C)=(S',C').
$$

所以 SIV 泄漏消息相等性。但这比泄漏：

$$
P\oplus P'
$$

或破坏认证 key 要温和得多。

### 10.2 SIV 的攻击边界

SIV 不是“nonce 随便错也零代价”。它的安全退化通常是：

- 重复消息泄漏等值关系。
- 无法隐藏同一 \(A,P\) 重复出现。
- 性能需要两遍处理消息。

但它避免了 CTR/GCM 里 nonce reuse 的灾难性 two-time pad。

## 11. GCM-SIV：GCM 风格的误用抵抗

GCM-SIV 试图保留 GCM 的性能和工程形态，同时减轻 nonce reuse 风险。它先用 POLYVAL/GHASH 类 universal hash 从 nonce、AAD、plaintext 派生 synthetic IV：

$$
S=\operatorname{Hash}_{K_1}(N,A,P).
$$

再加密：

$$
C=P\oplus\mathsf{CTR}_{K_2}(S).
$$

### 11.1 和 GCM 的关键差异

GCM 直接使用外部 nonce 选择 \(J_0\) 和 CTR stream。nonce 重复时：

$$
S_i=S_i'
$$

并且 tag mask 也重复。

GCM-SIV 中，外部 nonce 重复不必然让 synthetic IV 重复，因为：

$$
S=\operatorname{Hash}_{K_1}(N,A,P).
$$

如果 \(P\) 或 \(A\) 不同，\(S\) 通常不同。于是不会直接 two-time pad。

### 11.2 GCM-SIV 的泄漏

若：

$$
(N,A,P)=(N',A',P'),
$$

则输出相同。这仍泄漏等值关系。GCM-SIV 的目标不是让 nonce misuse 完全无害，而是把灾难性认证崩塌降级为更可控的确定性泄漏。

## 12. OCB：高性能 AEAD 的 offset 思路

OCB 使用 per-block offset：

$$
C_i=E_K(P_i\oplus\Delta_i)\oplus\Delta_i.
$$

同时计算认证 tag，绑定 plaintext/ciphertext、AAD、长度和顺序。

### 12.1 OCB 为什么不是 XTS

OCB 和 XTS 都有 tweak/offset 形态：

$$
E_K(P_i\oplus\Delta_i)\oplus\Delta_i.
$$

但目标不同：

- XTS 面向磁盘 confidentiality，不认证。
- OCB 面向消息 AEAD，认证 AAD、长度、顺序和内容。

所以 XTS 不能替代 OCB，OCB 也不是磁盘 sector rewrite 场景的默认答案。

### 12.2 OCB 的安全边界

OCB 正确使用时提供 AEAD。攻击者不能有效 bit flip、reorder、truncate 或伪造新密文，因为 tag 会失败。

但 OCB 仍是 nonce-based AEAD。nonce 重复会破坏证明边界，只是具体后果不同于 GCM 的 GHASH forbidden attack。

## 13. 横向结论

| 模式 | 主要目标 | 是否 AEAD | 关键误用 | 误用后果 |
| --- | --- | --- | --- | --- |
| ECB | 多块直接加密 | 否 | 用于结构化多块数据 | 等值关系、轮廓、codebook |
| CBC | 随机化保密 | 否 | IV 固定/可预测，padding oracle | 第一块泄漏、bit flip、明文恢复 |
| CFB | 自同步流式加密 | 否 | IV 重用，无认证 | 第一块 two-time pad，可塑 |
| OFB | keystream 输出反馈 | 否 | IV 重用 | 全流 two-time pad |
| CTR | 并行随机访问流模式 | 否 | nonce/counter collision | 全流或局部 two-time pad |
| GCM | CTR + GHASH AEAD | 是 | nonce 重复 | 明文 XOR 泄漏，tag forgery 风险 |
| CCM | CTR + CBC-MAC AEAD | 是 | nonce 重复，编码错误 | 明文 XOR 泄漏，认证边界破坏 |
| XTS | 磁盘 tweak 加密 | 否 | 当作认证加密 | replay/rollback 不被检测 |
| SIV | misuse-resistant AEAD | 是 | 重复相同消息 | 等值关系泄漏 |
| GCM-SIV | GCM 风格 misuse resistance | 是 | 重复相同 nonce/AAD/plaintext | 等值关系泄漏 |
| OCB | 高性能 AEAD | 是 | nonce 重复 | 证明边界破坏 |

简化判断：

> ECB 从根上不满足多块语义安全。CBC/CFB/OFB/CTR 在正确 IV/nonce 条件下可以保护被动窃听，但不能抗主动篡改。GCM/CCM/OCB 提供 AEAD，但 nonce discipline 是硬条件。SIV/GCM-SIV 用确定性泄漏换取 nonce misuse resistance。XTS 是磁盘加密模式，不是认证加密。

## 14. 攻击者视角的最终心法

看到一个加密系统时，先不要问“是不是 AES”。应按顺序问：

1. 模式是否确定性。
2. IV/nonce 是否唯一、随机、不可预测，以及到底需要哪一种。
3. 解密等式中是否有攻击者可控 XOR。
4. ciphertext、IV、AAD、长度、序号是否被认证。
5. 错误路径是否可区分。
6. parser 是否会处理未认证明文。
7. 是否存在 replay、reorder、rollback。
8. nonce 状态是否跨进程、重启、分布式节点保持唯一。

大多数真实 mode 漏洞不是“密码算法被破译”，而是系统把一个只保证被动保密的模式当成了完整安全通道。攻击者只需要找到等式中能控制的那一项。

## References

- NIST SP 800-38A, [*Recommendation for Block Cipher Modes of Operation: Methods and Techniques*](https://csrc.nist.gov/pubs/sp/800/38/a/final)
- NIST SP 800-38D, [*Recommendation for Block Cipher Modes of Operation: Galois/Counter Mode (GCM) and GMAC*](https://csrc.nist.gov/pubs/sp/800/38/d/final)
- NIST SP 800-38E, [*Recommendation for Block Cipher Modes of Operation: XTS-AES Mode for Confidentiality on Storage Devices*](https://csrc.nist.gov/pubs/sp/800/38/e/final)
- RFC 3686, [*Using AES Counter Mode With IPsec ESP*](https://www.rfc-editor.org/rfc/rfc3686)
- RFC 5297, [*Synthetic Initialization Vector (SIV) Authenticated Encryption*](https://www.rfc-editor.org/rfc/rfc5297)
- RFC 8452, [*AES-GCM-SIV: Nonce Misuse-Resistant Authenticated Encryption*](https://www.rfc-editor.org/rfc/rfc8452)
- Serge Vaudenay, [*Security Flaws Induced by CBC Padding Applications to SSL, IPSEC, WTLS...*](https://www.iacr.org/archive/eurocrypt2002/23320530/cbc02_e02d.pdf)
- Nadhem AlFardan and Kenny Paterson, [*Lucky Thirteen: Breaking the TLS and DTLS Record Protocols*](https://www.isg.rhul.ac.uk/tls/Lucky13.html)
- Bodo Moeller, Thai Duong, Krzysztof Kotowicz, [*This POODLE Bites: Exploiting The SSL 3.0 Fallback*](https://www.openssl.org/~bodo/ssl-poodle.pdf)
- Poddebniak et al., [*EFAIL: Breaking S/MIME and OpenPGP Email Encryption using Exfiltration Channels*](https://efail.de/efail-attack-paper.pdf)

