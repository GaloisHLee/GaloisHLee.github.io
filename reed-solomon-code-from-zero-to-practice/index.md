# Reed–Solomon 码：从零基础到原理、译码与工程实践


> Note: Reed–Solomon 码不是“把数据多复制几份”。它把有限域上的 \(k\) 个符号解释成低次多项式，再用多点求值或等价的生成多项式计算代数冗余。

这篇笔记从本科生真正需要的最小起点展开：先区分 error 与 erasure，再建立 Hamming distance、有限域和 Vandermonde 生成矩阵，随后推导 RS 的 MDS 距离并进入 syndrome、错误定位多项式、Chien search 与 Forney algorithm。中间所有小参数数值都由独立脚本复算。

工程部分固定 PyPI 稳定版 `reedsolo==1.7.0`。我们会主动破坏字节，分别测试未知错误、已知擦除、联合纠错边界和越界失败；同时说明一个更重要的边界：纠错码不是哈希、认证码或加密算法。

<!--more-->

## 先看损坏模型：位置未知和位置已知差一倍

设发送端产生码字 \(\boldsymbol c\)，信道或存储介质返回 \(\boldsymbol r\)。最简单的符号模型写成

$$
\boldsymbol r=\boldsymbol c+\boldsymbol e,
$$

其中 \(\boldsymbol e\) 是错误向量。加法发生在有限域中，不一定是普通整数加法。

真正决定译码成本的，不只是“坏了几个符号”，还有坏符号的位置是否已知：

| 模型 | 译码器已知什么 | 需要解决的问题 | 一个坏符号消耗的冗余预算 |
| --- | --- | --- | ---: |
| error | 不知道位置，也不知道原值 | 先定位，再恢复数值 | 2 |
| erasure | 知道位置，不知道原值 | 只恢复数值 | 1 |

例如，磁盘控制器若明确报告第 7 块不可读，这个位置就是 erasure。若收到一个看起来正常、实际已被静默改写的字节，译码器连位置都不知道，它才是 error。二维码上的污损常先经过版面定位和模块采样，再落到符号错误或擦除模型；光盘划痕则通常还要靠交织把连续 burst damage 分散到多个码字。

全文的对象关系可以先压成：

```text
物理损坏 / 传输噪声
  -> symbol errors / erasures
  -> Hamming distance 给出可恢复预算
  -> 有限域上的低次多项式产生 RS 码字
  -> syndrome 消去正确码字
  -> locator 找位置，evaluator 找数值
  -> 恢复后再用 hash / MAC 检查语义完整性
```

### 纠错、检错与认证不是同一件事

| 工具 | 主要目标 | 能否自动恢复 | 是否抵抗主动篡改 | 是否隐藏内容 |
| --- | --- | ---: | ---: | ---: |
| Reed–Solomon | 在给定符号预算内恢复码字 | 是 | 否 | 否 |
| CRC | 低成本检测随机传输错误 | 否 | 否 | 否 |
| cryptographic hash | 绑定内容摘要 | 否 | 单独使用时否 | 否 |
| MAC / digital signature | 验证完整性与来源 | 否 | 是 | 否 |
| encryption | 隐藏内容 | 否 | 视模式而定 | 是 |

> **边界先行**
>
> RS 译码器只问“附近是否有一个合法码字”。它不知道这个码字是不是发送者真正想表达的内容。超过保证半径时，译码器可能抛异常，也可能落到另一个合法码字；需要 hash 或 MAC 才能检查更高层语义。

## 线性码的最小工具箱

### 从消息到码字

取有限域 \(\mathbb F_q\)。一个 \(q\)-元线性 \([n,k]\) 码是 \(k\) 维线性子空间

$$
C\subseteq\mathbb F_q^n.
$$

编码器把消息

$$
\boldsymbol m\in\mathbb F_q^k
$$

映射为长度 \(n\) 的码字。若 \(G\in\mathbb F_q^{k\times n}\) 是满行秩生成矩阵，则

$$
\operatorname{Enc}(\boldsymbol m)=\boldsymbol mG.
$$

参数的含义是：

- \(n\)：code length，一个码字有多少 symbols；
- \(k\)：dimension，也是消息 symbols 数；
- \(n-k\)：redundancy / parity symbols 数；
- \(R=k/n\)：code rate，越高表示冗余比例越低。

这里的 symbol 是域元素。对 \(\operatorname{GF}(2^8)\)，一个 symbol 正好能装进一个 byte；对 \(\mathbb F_{11}\)，一个 symbol 是 \(0,\ldots,10\) 中的剩余类。

### Hamming weight 与 distance

向量 \(\boldsymbol x\) 的 Hamming weight 是非零坐标个数：

$$
\operatorname{wt}(\boldsymbol x)
=\left|\{i:x_i\ne0\}\right|.
$$

两个向量的 Hamming distance 是不同坐标个数：

$$
d_H(\boldsymbol x,\boldsymbol y)
=\operatorname{wt}(\boldsymbol x-\boldsymbol y).
$$

码的 minimum distance 定义为不同码字的最小距离：

$$
d(C)=\min_{\boldsymbol c\ne\boldsymbol c'}
d_H(\boldsymbol c,\boldsymbol c').
$$

因为线性码中 \(\boldsymbol c-\boldsymbol c'\) 仍是码字，所以也可写成

$$
d(C)=\min_{\boldsymbol 0\ne\boldsymbol c\in C}
\operatorname{wt}(\boldsymbol c).
$$

### 为什么最小距离控制检错和纠错

若错误向量满足

$$
\operatorname{wt}(\boldsymbol e)\le d-1,
$$

则 \(\boldsymbol c+\boldsymbol e\) 不可能是另一个码字：否则两个码字的距离至多是 \(d-1\)，与 minimum distance 矛盾。因此距离 \(d\) 能保证检测最多 \(d-1\) 个符号错误。

唯一纠错需要更强条件。若接收词 \(\boldsymbol r\) 同时位于两个不同码字 \(\boldsymbol c,\boldsymbol c'\) 的半径 \(t\) 球内，则三角不等式给出

$$
\begin{aligned}
d_H(\boldsymbol c,\boldsymbol c')
&\le d_H(\boldsymbol c,\boldsymbol r)
 +d_H(\boldsymbol r,\boldsymbol c')\\
&\le2t.
\end{aligned}
$$

只要 \(2t\lt d\)，两个球就不可能相交。于是唯一译码半径是

$$
t=\left\lfloor\frac{d-1}{2}\right\rfloor.
$$

擦除位置已知时，可以先删去这 \(s\) 个坐标。若两个码字在剩余坐标完全相同，则它们的距离至多为 \(s\)。因此 \(s\lt d\) 时仍能唯一恢复。把 \(s\) 个擦除坐标先 puncture 掉，剩余码的距离至少是 \(d-s\)；再纠正 \(e\) 个未知错误，需要

$$
2e\lt d-s.
$$

合并得

$$
2e+s\lt d.
$$

这就是“一个 error 消耗两个预算、一个 erasure 消耗一个预算”的距离解释。

### Singleton bound：冗余最多能换来多少距离

> **Singleton bound**
>
> 任意 \(q\)-元 \([n,k,d]\) 码都满足 \(d\le n-k+1\)。

证明很短。任意删去 \(d-1\) 个坐标；这个 puncturing 映射在 \(C\) 上必须是单射。否则两个不同码字删去后相同，它们原来只可能在被删的 \(d-1\) 个位置不同，与最小距离 \(d\) 矛盾。

删完只剩 \(n-d+1\) 个坐标，最多容纳 \(q^{n-d+1}\) 个不同向量。线性 \([n,k]\) 码有 \(q^k\) 个码字，所以

$$
q^k\le q^{n-d+1}.
$$

比较指数：

$$
k\le n-d+1,
$$

即

$$
d\le n-k+1.
$$

达到等号的码称为 Maximum Distance Separable（MDS）码。它在固定 \(n,k\) 下把最小距离推到了可能的最大值。Reed–Solomon 码正是 MDS 码。[^huffman-pless]

## 有限域：为什么一个 symbol 可以是一整个字节

### 素域 \(\mathbb F_p\)

当 \(p\) 是素数时，整数模 \(p\) 构成域 \(\mathbb F_p\)。加法和乘法都取模；每个非零元素都有乘法逆元。

例如在 \(\mathbb F_{11}\) 中，

$$
4^{-1}=3,
$$

因为

$$
4\cdot3=12\equiv1\pmod {11}.
$$

于是“除以 4”就是“乘以 3”。后面的插值、Gaussian elimination 和 Forney 公式都依赖这种可除性。

若模数不是素数，事情会坏掉。例如在整数模 256 中，2 没有乘法逆元，因为它和 256 不互素。故

$$
\mathbb Z/256\mathbb Z
$$

不是域。

### 扩域 \(\mathbb F_{p^m}\)

要得到 \(p^m\) 个元素，取 \(\mathbb F_p[u]\) 上一个次数 \(m\) 的不可约多项式 \(P(u)\)，构造商环

$$
\mathbb F_{p^m}
\cong
\mathbb F_p[u]/(P(u)).
$$

每个域元素由次数小于 \(m\) 的多项式表示。加法逐系数进行；乘法先做多项式乘法，再模 \(P(u)\) 约化。不可约性保证每个非零剩余类都有逆元。

对 \(\operatorname{GF}(2^8)\)，系数只有 0 和 1，一个元素

$$
b_7u^7+\cdots+b_1u+b_0
$$

恰好对应 8-bit byte \(b_7\cdots b_0\)。加法就是逐位 XOR，但乘法不是普通整数乘法，也不是模 256 乘法。

本文的复算脚本使用本原多项式

$$
P(u)=u^8+u^4+u^3+u^2+1,
$$

其十六进制表示是 `0x11d`。脚本实际得到

```text
0x57 + 0x83 = 0xd4    # XOR
0x57 * 0x83 = 0x31    # polynomial product reduced by 0x11d
```

“不可约多项式”和“本原元”是两个对象：前者定义扩域表示；后者是乘法群

$$
\mathbb F_{2^8}^{\times}
$$

的生成元。选用本原多项式时，\(u\) 的剩余类 \(\alpha=[u]\) 可以有阶 \(2^8-1=255\)，于是每个非零元素都能写成 \(\alpha^i\)。指数表 / 对数表正是许多 RS 实现加速乘除法的基础。

### bit error 与 symbol error

在 \(\operatorname{GF}(2^8)\) codec 中，一个 symbol 是一个 byte。某个 byte 内翻转 1 bit 或 8 bits，都只占一个 symbol-error 位置；相反，连续损坏 20 个 bytes 通常占 20 个符号位置。RS 擅长处理有限数量的 symbol errors；若物理 burst 太长，系统会用 interleaving 把相邻 bytes 分散到不同码字。

## RS 码就是低次多项式的多点求值

取有限域 \(\mathbb F_q\) 和 \(n\) 个互异评价点

$$
\boldsymbol a=(a_1,\ldots,a_n)\in\mathbb F_q^n,
$$

其中 \(n\le q\)。对 \(k\le n\)，定义 Reed–Solomon 码[^reed-solomon]：

> **Reed–Solomon code**
>
> 这是所有次数小于 \(k\) 的多项式在固定互异点上的评价向量。

$$
\operatorname{Ev}_{\boldsymbol a}(f)
=\bigl(f(a_1),\ldots,f(a_n)\bigr).
$$

于是

$$
\operatorname{RS}_k(\boldsymbol a)
=\left\{
\operatorname{Ev}_{\boldsymbol a}(f):
\deg f\lt k
\right\},
$$

其中 \(f\in\mathbb F_q[x]\)。

### Vandermonde 生成矩阵

把消息 \(\boldsymbol m=(m_0,\ldots,m_{k-1})\) 解释为系数，令

$$
f(x)=\sum_{j=0}^{k-1}m_jx^j.
$$

逐点评价等价于矩阵乘法

$$
\boldsymbol c=\boldsymbol mG,
$$

其中

$$
G=
\begin{pmatrix}
1&1&\cdots&1\\
a_1&a_2&\cdots&a_n\\
a_1^2&a_2^2&\cdots&a_n^2\\
\vdots&\vdots&\ddots&\vdots\\
a_1^{k-1}&a_2^{k-1}&\cdots&a_n^{k-1}
\end{pmatrix}.
$$

这是 Vandermonde 型生成矩阵。评价点互异时，任取前 \(k\) 列得到的方阵行列式为

$$
\prod_{1\le i\lt j\le k}(a_j-a_i)\ne0,
$$

所以矩阵满行秩，码的维数确实是 \(k\)。等价地，次数小于 \(k\) 的两个多项式若在 \(k\) 个互异点取值相同，它们之差有至少 \(k\) 个根，只能是零多项式。

### 为什么 \(d=n-k+1\)

取任意非零 \(f\) 且 \(\deg f\lt k\)。域上的非零多项式至多有 \(\deg f\) 个根，因此在 \(n\) 个评价点中最多有 \(k-1\) 个位置取零。对应码字至少有

$$
n-(k-1)=n-k+1
$$

个非零坐标，故

$$
d\ge n-k+1.
$$

这个下界可以达到。任选 \(k-1\) 个评价点，取

$$
f_*(x)=\prod_{j=1}^{k-1}(x-a_j).
$$

它的次数小于 \(k\)，并且恰好在选中的 \(k-1\) 个评价点为零，故对应码字 weight 为 \(n-k+1\)。于是

$$
d=n-k+1.
$$

结合 Singleton bound，RS 是 \([n,k,n-k+1]\) MDS 码。

若记冗余符号数

$$
r=n-k,
$$

则 RS 的保证范围变成

$$
2e+s\le r.
$$

只含未知错误时可纠正

$$
e\le\left\lfloor\frac r2\right\rfloor;
$$

只含已知擦除时可恢复

$$
s\le r.
$$

## 同一个 RS 码，三种消息接口

“RS 编码后还能不能直接看到原消息？”这个问题没有只靠 \([n,k]\) 就能给出的答案。RS 定义规定了码字集合，却没有规定应用如何把 \(k\) 个消息符号映射到这组码字。

### 系数消息：最直接，但通常非系统

定义中的自然接口是把消息直接当作多项式系数：

$$
\boldsymbol m=(m_0,\ldots,m_{k-1}).
$$

对应的多项式是

$$
f(x)=\sum_{j=0}^{k-1}m_jx^j.
$$

然后输出

$$
\bigl(f(a_1),\ldots,f(a_n)\bigr).
$$

这种写法最适合证明线性、维数和最小距离，但前 \(k\) 个输出通常不等于 \(\boldsymbol m\)。它是 non-systematic encoding。

### 值消息：先插值，再得到系统评价码

也可以规定消息符号就是前 \(k\) 个评价点上的值：

$$
f(a_i)=m_i,\qquad 1\le i\le k.
$$

由 Lagrange interpolation，唯一的次数小于 \(k\) 的多项式是

$$
f(x)=\sum_{i=1}^{k}m_iL_i(x),
$$

其中

$$
L_i(x)=
\prod_{\substack{1\le j\le k\\j\ne i}}
\frac{x-a_j}{a_i-a_j}.
$$

再计算剩余 \(n-k\) 个评价值，所得码字前 \(k\) 个坐标恰好是原消息。这是 systematic evaluation encoding。它与系数消息接口生成同一个 RS 码，只是选择了不同的消息基。

### 生成多项式：工程 codec 常见的系统形式

当码长与评价点选择允许把 RS 写成 cyclic / BCH-style code 时，常取本原元 \(\alpha\)，令冗余 \(r=n-k\)，并选连续根

$$
\alpha^b,\alpha^{b+1},\ldots,\alpha^{b+r-1}.
$$

生成多项式为

$$
g(z)=\prod_{j=0}^{r-1}
\left(z-\alpha^{b+j}\right).
$$

把消息序列解释为消息多项式 \(m(z)\)，先空出 \(r\) 个校验符号，再做 polynomial division：

$$
c(z)=z^rm(z)-
\operatorname{rem}_{g(z)}\bigl(z^rm(z)\bigr).
$$

因为右侧可被 \(g(z)\) 整除，\(c(z)\) 是合法码字；低 \(r\) 次部分由余数抵消，高次消息部分保留。具体到 byte array 时，系数按高次到低次还是反向存储、消息出现在前缀还是后缀，取决于实现约定。

三种接口可放在一张表里：

| 视角 | 消息符号表示什么 | 码字是否直接包含消息 | 最适合解释什么 |
| --- | --- | ---: | --- |
| coefficient evaluation | \(f\) 的系数 | 通常否 | 定义、Vandermonde、MDS 证明 |
| value evaluation | 指定 \(k\) 点上的函数值 | 是 | 插值、系统评价码 |
| cyclic generator polynomial | 序列多项式 \(m(z)\) 的系数 | 通常是 | syndrome 与工程 codec |

> **Convention Warning**
>
> 上述视角描述等价码族，但不能在一行公式中无说明地混用。`reedsolo` 的 `RSCodec` 输出是系统字节序列，默认 `fcr=0`，并有明确的系数方向；它不是把用户 bytes 直接当作 \(f(x)\) 的低次系数，再返回某组公开评价点上的值。

## 代数译码：先定位，再求错误值

距离告诉我们“在什么范围内唯一解存在”，代数译码回答“如何把这个解算出来”。下面固定一个教学约定，避免指数和位置因子来回变化：

- 使用 cyclic RS 表示；
- 码字多项式在 \(\alpha,\alpha^2,\ldots,\alpha^r\) 上为零，即首根 \(b=1\)；
- 错误发生在系数位置 \(i_1,\ldots,i_e\)；
- 定义错误位置数 \(X_\ell=\alpha^{i_\ell}\)，错误值为 \(Y_\ell\)。

不同教材和库可能采用 \(b=0\)、反向坐标或 reciprocal locator。算法结构不变，但 Forney 公式会多出一个位置因子。公式必须跟约定一起搬运。

### Syndrome：把正确码字消掉

接收多项式为

$$
r(z)=c(z)+E(z).
$$

因为合法码字满足 \(c(\alpha^j)=0\)，定义 syndrome

$$
S_j=r(\alpha^j),\qquad 1\le j\le r,
$$

就有

$$
S_j=E(\alpha^j).
$$

若错误多项式写成

$$
E(z)=\sum_{\ell=1}^{e}Y_\ell z^{i_\ell},
$$

则

$$
S_j=
\sum_{\ell=1}^{e}Y_\ell X_\ell^j.
$$

正确码字已经完全消失，剩下的是少量指数序列之和。若所有 \(S_j=0\)，接收词本身是一个合法码字；这不保证它一定是发送端原来的那个码字。

### 错误定位多项式

定义 error-locator polynomial

$$
\Lambda(z)=
\prod_{\ell=1}^{e}(1-X_\ell z).
$$

把乘积展开可写成

$$
\Lambda(z)=
1+\lambda_1z+\cdots+\lambda_ez^e.
$$

它的根是错误位置数的逆：

$$
\Lambda(X_\ell^{-1})=0.
$$

把等式乘以 \(X_\ell^e\)，得到

$$
X_\ell^e+
\lambda_1X_\ell^{e-1}+
\cdots+
\lambda_e=0.
$$

再乘 \(Y_\ell X_\ell^j\) 并对所有错误求和：

$$
S_{j+e}+
\lambda_1S_{j+e-1}+
\cdots+
\lambda_eS_j=0.
$$

因此 syndrome 序列满足阶数至多为 \(e\) 的线性递推。Berlekamp–Massey algorithm 的任务，就是从有限段 syndrome 中找出满足它的最短 connection polynomial；在这里，它正对应 \(\Lambda(z)\)。Massey 的 1969 年论文把 shift-register synthesis 与 BCH decoding 明确连接起来。[^massey]

### Berlekamp–Massey 在更新什么

算法逐个读入 syndrome，维护当前候选 \(\Lambda(z)\)。对第 \(N\) 个 syndrome 计算 discrepancy

$$
\Delta_N=S_N+
\sum_{i=1}^{L}\lambda_iS_{N-i},
$$

其中 \(L=\deg\Lambda\)。

- 若 \(Delta_N=0\)，当前递推仍解释新 syndrome；
- 若 \(Delta_N\ne0\)，用上一次有效候选的适当移位与缩放修正 \(\Lambda\)；
- 当现有阶数不足以解释 discrepancy 时，提高 \(L\)。

完整实现还要维护旧候选、多项式移位和上次非零 discrepancy。这里最重要的不是背更新式，而是理解它的输入输出：

```text
S_1, S_2, ..., S_r
  -> shortest linear recurrence
  -> locator Lambda(z)
  -> its inverse roots encode error positions
```

### Chien search：从根回到坐标

得到 \(\Lambda(z)\) 后，可以直接遍历所有可能位置 \(i\)，检查

$$
\Lambda(\alpha^{-i})=0.
$$

每个命中都对应一个错误坐标。有限域只有 \(n\) 个候选位置，这种枚举比调用一般多项式求根器更简单，也适合硬件流水线。该步骤称为 Chien search。[^chien]

若找到的根数不等于 \(\deg\Lambda\)，候选 locator 与实际错误模式不一致。`reedsolo` 的越界异常示例正会报告 Chien search 找到的错误数不对。

### Key equation 与错误求值多项式

把 syndrome 装入形式幂级数

$$
S(z)=S_1+S_2z+\cdots+S_rz^{r-1}.
$$

由几何级数，模 \(z^r\) 有

$$
S(z)\equiv
\sum_{\ell=1}^{e}
\frac{Y_\ell X_\ell}{1-X_\ell z}
\pmod {z^r}.
$$

乘以 locator 后，分母被约掉：

$$
\Lambda(z)S(z)
\equiv\Omega(z)
\pmod {z^r},
$$

其中 error-evaluator polynomial 是

$$
\Omega(z)=
\sum_{\ell=1}^{e}
Y_\ell X_\ell
\prod_{h\ne\ell}(1-X_hz),
$$

并满足 \(\deg\Omega\lt e\)。这就是 key equation。Berlekamp–Massey 先求 \(\Lambda\)，随后可由截断乘积求 \(\Omega\)；扩展 Euclidean algorithm 也能直接求出满足 key equation 的一对多项式。

### Forney algorithm：由位置求错误值

在 \(z=X_\ell^{-1}\) 处，\(\Omega\) 的求和中除第 \(\ell\) 项外全部为零：

$$
\Omega(X_\ell^{-1})=
Y_\ell X_\ell
\prod_{h\ne\ell}
\left(1-X_hX_\ell^{-1}\right).
$$

另一方面，对 locator 求导并代入同一点：

$$
\Lambda'(X_\ell^{-1})=
-X_\ell
\prod_{h\ne\ell}
\left(1-X_hX_\ell^{-1}\right).
$$

两式相除，得到当前 \(b=1\) 约定下的 Forney magnitude formula：

$$
Y_\ell=
-\frac{\Omega(X_\ell^{-1})}
{\Lambda'(X_\ell^{-1})}.
$$

在特征 2 中负号与正号相同。若首根、位置指数或 syndrome 起点改变，公式会出现额外的 \(X_\ell\) 幂次；这不是算法冲突，而是约定变换。Forney 的原始论文讨论了 BCH 类码的这种代数译码。[^forney]

### 加入 erasure

若已知擦除位置集合 \(J\)，先构造 erasure-locator polynomial

$$
\Gamma(z)=\prod_{j\in J}(1-X_jz).
$$

可以把 \(\Gamma\) 合入 locator，或先用它修改 syndromes，再让 Berlekamp–Massey 只求未知错误部分。已知擦除已经给出了位置，每个擦除只剩一个 magnitude 未知；一个未知 error 则同时包含位置和 magnitude 两类未知量。

从 syndrome 约束数量看，\(r=n-k\) 个冗余符号最多承担

$$
2e+s\le r.
$$

这是直观的未知量计数；前面的 minimum-distance 论证给出了严格的唯一译码保证。

完整 bounded-distance decoder 可以概括为：

```text
Input: received word r, optional erasure positions J

1. Evaluate syndromes S_1, ..., S_r.
2. Build the erasure locator Gamma(z) when J is known.
3. Run Berlekamp-Massey (or extended Euclid) for Lambda(z).
4. Run Chien search to recover error locations.
5. Compute Omega(z) from the key equation.
6. Use Forney's formula to recover magnitudes.
7. Subtract the errata and verify that the corrected word is a codeword.

Guarantee: 2 * unknown_errors + erasures <= n - k.
```

最后一步的“是码字”检查仍不是内容认证：越界接收词可能更靠近另一个合法码字。

## \(\mathbb F_{11}\) 上的 \([7,3,5]\) 贯穿例

现在把抽象对象压到一个可手算实例。取

在 \(\mathbb F_{11}\) 上令 \(k=3\)，并取评价点

$$
\boldsymbol a=(0,1,2,3,4,5,6).
$$

参数是

$$
[n,k,d]=[7,3,5],
$$

所以冗余 \(r=4\)，能保证纠正 2 个未知错误或 4 个已知擦除。

Vandermonde 生成矩阵为

$$
G=
\begin{pmatrix}
1&1&1&1&1&1&1\\
0&1&2&3&4&5&6\\
0&1&4&9&5&3&3
\end{pmatrix}.
$$

独立脚本在模 11 下验证 \(\operatorname{rank}(G)=3\)。

### 系数消息的非系统评价

取消息系数

$$
\boldsymbol m=(3,2,4),
$$

对应

$$
f(x)=3+2x+4x^2.
$$

逐点计算：

| \(x\) | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
| ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| \(f(x)\bmod 11\) | 3 | 9 | 1 | 1 | 9 | 3 | 5 |

所以

$$
\boldsymbol c=(3,9,1,1,9,3,5).
$$

前三个符号 \((3,9,1)\) 并不是消息系数 \((3,2,4)\)。这不是编码失败，而是 coefficient-evaluation 接口本来就不系统。

### 值消息的系统评价

改用值消息

$$
(f(0),f(1),f(2))=(4,7,2).
$$

令

$$
f(x)=c_0+c_1x+c_2x^2.
$$

由 \(f(0)=4\) 得 \(c_0=4\)。另外两式是

$$
c_1+c_2=3,
$$

以及

$$
2c_1+4c_2=9
\pmod {11}.
$$

第二式减去第一式的两倍，得到

$$
2c_2=3.
$$

在 \(\mathbb F_{11}\) 中 \(2^{-1}=6\)，故

$$
c_2=18\equiv7\pmod {11},
$$

进而 \(c_1=7\)。插值多项式是

$$
f(x)=4+7x+7x^2.
$$

在七个点上评价：

$$
\boldsymbol c_{\mathrm{sys}}
=(4,7,2,0,1,5,1).
$$

这次前三个坐标原样保留了值消息。

### 枚举验证距离与边界

下面是复算脚本中最小的核心：

```python
from itertools import product

P = 11
support = list(range(7))

def evaluate(coefficients, x):
    value = 0
    for coefficient in reversed(coefficients):
        value = (value * x + coefficient) % P
    return value

def encode(coefficients):
    return [evaluate(coefficients, x) for x in support]

codewords = [encode(list(m)) for m in product(range(P), repeat=3)]
minimum_weight = min(
    sum(symbol != 0 for symbol in word)
    for word in codewords[1:]
)

print(len(codewords), minimum_weight)
```

实际输出是：

```text
1331 5
```

脚本枚举了全部 \(11^3=1331\) 个消息，而不是随机抽样。最小非零 weight 确实为 5，与 \(n-k+1=5\) 一致。

再把系数码字的第 1、5 号位置改坏，得到接收词

$$
\boldsymbol r_2=(3,10,1,1,9,5,5).
$$

枚举所有码字后，距离不超过 2 的候选只有一个，其消息系数就是 \((3,2,4)\)。这与唯一译码半径 2 一致。

距离 3 时保证已经消失。考虑零码字和

$$
f_1(x)=x^2-x,
$$

对应码字

$$
\boldsymbol c_1=(0,0,2,6,1,9,8),
$$

其 weight 为 5。接收词

$$
\boldsymbol r_3=(0,0,2,6,0,0,0)
$$

满足

$$
d_H(\boldsymbol r_3,\boldsymbol 0)=2,
$$

同时

$$
d_H(\boldsymbol r_3,\boldsymbol c_1)=3.
$$

因此半径 3 的球已经重叠：若 \(\boldsymbol c_1\) 才是发送码字，发生 3 个错误后，nearest-codeword decoder 反而会选择更近的零码字。这就是“超过界限可能误纠”的具体几何图像。

## Python 实验：用 `reedsolo` 破坏并修复字节

### 版本与接口边界

本文在隔离环境中固定 PyPI 稳定版：

```bash
python -m pip install "reedsolo==1.7.0"
```

截至本文复核时间，PyPI 稳定版是 1.7.0；官方 GitHub `master` 已包含面向 v2 的迁移说明，但不能把尚未发布的 master 文档当作稳定版行为。API 资料已通过 Context7 的 `/tomerfiliba-org/reedsolomon` 索引与官方 README 交叉核对。[^reedsolo]

高层接口只有两个核心对象：

```python
from reedsolo import RSCodec, ReedSolomonError

rsc = RSCodec(10)  # append 10 error-correction symbols per chunk
encoded = rsc.encode(b"Reed-Solomon")
decoded, corrected_msgecc, errata_pos = rsc.decode(encoded)
```

`decode()` 返回三个值：

1. 修复后的原消息；
2. 修复后的“消息 + ECC”完整码字；
3. 被修复的 error / erasure 位置。

### 第一个码字

```python
from importlib.metadata import version
from reedsolo import RSCodec

message = b"Reed-Solomon"
rsc = RSCodec(10)
encoded = rsc.encode(message)

print(version("reedsolo"))
print(len(message), len(encoded))
print(encoded.hex(" "))
```

真实输出：

```text
1.7.0
12 22
52 65 65 64 2d 53 6f 6c 6f 6d 6f 6e af 3b a7 2e be 2c e9 ce da 3a
```

前 12 bytes 正是 ASCII 消息，后 10 bytes 是 parity symbols。这是系统编码接口。

### 五个未知错误：恰好用满十个冗余符号

为了让实验可重复，固定损坏位置和非零 XOR 差值：

```python
from reedsolo import RSCodec

def corrupt(data, positions, salt):
    damaged = bytearray(data)
    for offset, position in enumerate(positions, start=1):
        delta = ((salt + 37 * offset) % 255) + 1
        damaged[position] ^= delta
    return damaged

message = b"Reed-Solomon"
rsc = RSCodec(10)
encoded = rsc.encode(message)

positions = [0, 3, 7, 12, 20]
damaged = corrupt(encoded, positions, salt=11)
decoded, corrected, errata_pos = rsc.decode(damaged)

assert bytes(decoded) == message
assert bytes(corrected) == bytes(encoded)
assert sorted(errata_pos) == positions

print(bytes(decoded))
print(list(errata_pos))
print(rsc.check(corrected))
```

真实输出：

```text
b'Reed-Solomon'
[20, 12, 7, 3, 0]
[True]
```

位置返回顺序不承诺升序，所以断言比较排序后的集合。`check()` 对每个 chunk 返回一个布尔值；这里完整修复后的单个码字通过校验。

十个 parity symbols 能保证纠正的未知错误数是

$$
\left\lfloor\frac{10}{2}\right\rfloor=5.
$$

### 十二个已知擦除

已知位置时，一个 erasure 只消耗一个预算。换用 12 个 parity symbols：

```python
from reedsolo import RSCodec

message = b"Reed-Solomon"
rsc = RSCodec(12)
encoded = rsc.encode(message)  # 12 + 12 = 24 bytes

erase_pos = list(range(0, 24, 2))
damaged = corrupt(encoded, erase_pos, salt=23)
decoded, corrected, errata_pos = rsc.decode(
    damaged,
    erase_pos=erase_pos,
)

assert bytes(decoded) == message
assert sorted(errata_pos) == erase_pos

print(bytes(decoded))
print(sorted(errata_pos))
print(rsc.check(corrected))
```

真实输出：

```text
b'Reed-Solomon'
[0, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22]
[True]
```

这里破坏了整整 12 个位置；因为位置通过 `erase_pos` 提供，仍在保证边界内。

### 联合错误与擦除：边界取等

取 `nsym=10`，设置 3 个未知错误和 4 个已知擦除：

$$
2\cdot3+4=10.
$$

```python
from reedsolo import RSCodec

message = b"Reed-Solomon"
rsc = RSCodec(10)
encoded = rsc.encode(message)

erase_pos = [1, 5, 13, 18]
unknown_error_pos = [2, 9, 21]
damaged = corrupt(
    encoded,
    erase_pos + unknown_error_pos,
    salt=41,
)

decoded, corrected, errata_pos = rsc.decode(
    damaged,
    erase_pos=erase_pos,
)

assert bytes(decoded) == message
assert sorted(errata_pos) == sorted(erase_pos + unknown_error_pos)

print(bytes(decoded))
print(sorted(errata_pos))
print(rsc.check(corrected))
```

真实输出：

```text
b'Reed-Solomon'
[1, 2, 5, 9, 13, 18, 21]
[True]
```

这个实验把 \(2e+s\le n-k\) 用满，但没有超过。

### 超出能力：一个稳定异常案例

官方 README 给出 `RSCodec(10)` 下 6 个未知错误的失败例。下面用同一组位置从编码结果构造：

```python
from reedsolo import RSCodec, ReedSolomonError

rsc = RSCodec(10)
encoded = rsc.encode(b"hello world")
damaged = bytearray(encoded)

for position in [1, 2, 3, 9, 15, 16]:
    damaged[position] = ord("X")

try:
    rsc.decode(damaged)
except ReedSolomonError as error:
    print(type(error).__name__)
    print(error)
```

真实输出：

```text
ReedSolomonError
Too many (or few) errors found by Chien Search for the errata locator polynomial!
```

> **不要把这个异常误写成保证**
>
> 6 个未知错误已经超出 \(10/2=5\) 的保证半径。这个固定样例会抛异常，但官方 README 明确提醒：其他越界模式也可能被译成另一个合法码字而不抛异常。`check()` 只能检查“结果是不是码字”，不能证明“结果是不是原消息”。可靠系统应额外保存 cryptographic hash；存在主动攻击者时应使用 MAC 或 digital signature。

### `nsym` 如何改变长度与纠错能力

```python
from reedsolo import RSCodec

message = b"parameter-check"

for nsym in [8, 16]:
    rsc = RSCodec(nsym)
    encoded = rsc.encode(message)
    print(nsym, len(encoded), rsc.maxerrata())
```

真实输出：

```text
8 23 (4, 8)
16 31 (8, 16)
```

增加 `nsym` 会降低码率，但提高每个 chunk 的错误/擦除预算。`maxerrata()` 返回“只含 errors 时的最大数”和“只含 erasures 时的最大数”；联合情形仍使用 \(2e+s\le\texttt{nsym}\)。

### 长消息：自动 chunking，不是一个超长码字

默认 \(\operatorname{GF}(2^8)\) 下 `nsize=255`。当 `nsym=10` 时，每个 chunk 最多携带

$$
255-10=245
$$

个消息 bytes。`RSCodec` 会自动把更长输入分块：

```python
from reedsolo import RSCodec

rsc = RSCodec(10)
message = bytes(index % 251 for index in range(600))
encoded = rsc.encode(message)
decoded, corrected, _ = rsc.decode(encoded)

print(len(message), len(encoded))
print(rsc.nsize - rsc.nsym)
print(rsc.check(corrected))
print(bytes(decoded) == message)
```

真实输出：

```text
600 630
245
[True, True, True]
True
```

600 bytes 被切成 245、245、110 三块，每块各追加 10 个 parity bytes，因此总长度是 630。三块各自有独立纠错预算；不能把 30 个 parity bytes 当作可集中纠正任意一个位置的统一预算。

`RSCodec` 也允许通过 `nsize`、`c_exp`、`prim`、`fcr` 等参数选择更大域或兼容其他实现。除非协议已经固定这些参数，否则不要随意改：编码器和译码器必须在域表示、首根、生成元和符号顺序上完全一致。

## 历史注记：译码流水线不是一次写成的

| 时间 | 工作 | 留下的接口 |
| ---: | --- | --- |
| 1960 | Reed 与 Solomon 提出有限域上的 polynomial codes | 低次多项式、多点求值、最大距离结构 |
| 1964 | Chien 给出 BCH 类码的循环求根过程 | 由 locator roots 定位错误坐标 |
| 1965 | Forney 研究 BCH 代数译码 | error evaluator 与 magnitude 计算 |
| 1968--1969 | Berlekamp 的代数译码与 Massey 的 shift-register synthesis | 从 syndrome 求最短递推 / locator |

今天常说的“Berlekamp–Massey + Chien + Forney”不是四个互不相干的黑盒。它们的中间接口恰好首尾相接：

1. syndromes；
2. locator \(\Lambda\)；
3. error positions；
4. evaluator \(\Omega\)；
5. error magnitudes。

## 应用：RS 解决的是哪一层问题

### QR Code：版面污损最终落到 codewords

DENSO WAVE 的官方说明明确写道，QR Code 的 error correction 通过加入 Reed–Solomon code 实现；提高纠错等级会提升恢复能力，也会增加码字开销和二维码尺寸。[^qr]

这并不表示“污损面积百分比”能直接等同于 RS 的 symbol-error 数。QR decoder 还要完成定位图形识别、采样、mask 撤销、block 重组等步骤。到了 RS 层，输入才是分块后的 codewords 与可能的 erasure information。

### CD：RS 与交织共同对付 burst damage

音乐 CD 的 Cross-Interleaved Reed–Solomon Code（CIRC）把 RS 与跨帧交织组合。划痕造成连续物理损坏；交织先把相邻坏 symbols 分散到多个码字，使每个码字内的错误数落回 RS 的纠错预算。[^blahut]

这里真正有效的系统链是

```text
continuous scratch
  -> deinterleave into sparse symbol errors
  -> Reed-Solomon decode each codeword
  -> conceal or report data that still cannot be recovered
```

单独把一个长 burst 直接塞进一个短 RS 码字，可能迅速用完全部符号预算。

### CCSDS：信道编码是协议栈的一层

CCSDS 131.0-B-5 *TM Synchronization and Channel Coding* 把 Reed–Solomon coding 作为遥测信道编码选项之一。[^ccsds] 实际航天链路还包含同步、分帧、交织与其他信道码；RS 负责其中的符号级块码接口，不应被描述成完整链路协议。

### RAID 与分布式存储：丢失位置往往已知

在 RAID-like storage 中，控制器通常知道哪块磁盘或哪个 shard 不可用，因此主要面对 erasures。Plank 的教程把 RS coding 用于这类 fault-tolerance 模型。[^plank]

若用 \(k\) 个 data shards 生成 \(r\) 个 parity shards，任意至多 \(r\) 个已知 shard erasures 可以恢复，这正对应 MDS 性质。网络传输中的 silent corruption 则更接近 unknown errors，每个坏 symbol 要消耗两倍预算。

| 场景 | RS 层常见模型 | 还需要的外层机制 |
| --- | --- | --- |
| QR Code | 分块后的 symbol errors / erasures | 定位、采样、mask、block layout |
| CD / optical media | 交织后的 sparse errors | interleaving、frame logic、concealment |
| CCSDS telemetry | channel symbol errors | synchronization、framing、其他 channel coding |
| RAID / distributed storage | known shard erasures | placement、metadata、checksum、repair policy |

### 它仍然不提供真实性和保密性

RS 可以把“附近的损坏序列”投回码空间，但码空间中每个码字在数学上都同样合法。若攻击者有意把数据改成另一个合法码字，syndrome 会全部为零；若损坏越过唯一译码边界，decoder 也可能误纠。

工程上常见的组合是：

```text
plaintext
  -> optional encryption / authentication
  -> framing and checksum
  -> Reed-Solomon encoding
  -> channel or storage
  -> Reed-Solomon decoding
  -> verify checksum / hash / MAC
```

顺序取决于协议，但职责不能混淆。

## 从 RS 到 GRS：强结构的另一面

广义 Reed–Solomon（Generalized Reed–Solomon, GRS）码在每个评价坐标再乘一个非零因子。先定义带权评价

$$
\operatorname{Ev}_{\boldsymbol a,\boldsymbol v}(f)
=\bigl(v_1f(a_1),\ldots,v_nf(a_n)\bigr).
$$

其中 \(v_i\in\mathbb F_q^\times\)。当 \(f\) 遍历所有次数小于 \(k\) 的多项式时，得到

$$
\operatorname{GRS}_k(\boldsymbol a,\boldsymbol v)
=\left\{
\operatorname{Ev}_{\boldsymbol a,\boldsymbol v}(f)
\right\},
$$

其中 \(f\in\mathbb F_q[x]\)。它仍是 \([n,k,n-k+1]\) MDS 码。列乘因子没有破坏“低次多项式评价”这一核心。

定义向量的 coordinate-wise / Schur product：

$$
\boldsymbol x\star\boldsymbol y
=(x_1y_1,\ldots,x_ny_n).
$$

对两个线性码，先收集全部向量乘积：

$$
\mathcal P(C,D)=
\left\{
\boldsymbol c\star\boldsymbol d:
\boldsymbol c\in C,\ \boldsymbol d\in D
\right\}.
$$

code-level Schur product 是这个集合的线性张成：

$$
C\star D=
\operatorname{span}_{\mathbb F_q}\mathcal P(C,D).
$$

若 \(\boldsymbol c\in C\) 和 \(\boldsymbol d\in D\) 分别对应多项式 \(f,g\)，

则

$$
\boldsymbol c\star\boldsymbol d
=\bigl(v_iw_i(fg)(a_i)\bigr)_{i=1}^n.
$$

因为 \(\deg f\le k-1\) 且 \(\deg g\le\ell-1\)，乘积满足

$$
\deg(fg)\le k+\ell-2.
$$

不同单项式乘积张成所有允许次数。把两个 GRS 码简记为 \(C,D\)，并令

$$
m=\min\{n,k+\ell-1\}.
$$

于是

$$
C\star D=
\operatorname{GRS}_m
(\boldsymbol a,\boldsymbol v\star\boldsymbol w).
$$

特别地，若 \(C=\operatorname{GRS}_k(\boldsymbol a,\boldsymbol v)\)，则 square code 的维数是

$$
\dim C^{\star2}=\min\{n,2k-1\}.
$$

这个低维现象来自与高效译码相同的根：低次多项式乘积仍然低次。它是结构优势，也是把 GRS 直接放进公钥时的可见痕迹。

下一篇 [《基于 Reed–Solomon 码的公钥密码系统的代数结构攻击》](/reed-solomon-structural-attacks/) 将从这里继续：为什么换行基、置换坐标或插入少量随机列，仍可能留下 square-code、puncture、shorten 与 cross-ratio 可利用的关系。

## 课后思考与练习

### 练习 1：从参数读出保证

给定一个 \([255,223]\) RS 码，计算 minimum distance、纯 errors 半径、纯 erasures 上限。若已有 10 个 unknown errors，最多还能容纳多少 erasures？

> **答案：** \(r=32\)，所以 \(d=33\)，最多纠正 16 errors 或 32 erasures。联合条件 \(2\cdot10+s\le32\)，故 \(s\le12\)。

### 练习 2：为什么模 256 不够

说明 \(\mathbb Z/256\mathbb Z\) 中的元素 2 没有乘法逆元；再解释 \(\operatorname{GF}(2^8)\) 如何通过不可约多项式避免这个问题。

> **提示：** 若 \(2x\equiv1\pmod {256}\)，左边必为偶数，右边为奇数。扩域中的非零元素是多项式剩余类，不是模 256 整数；不可约性使商环成为域。

### 练习 3：手算一次评价编码

在本文 \(\mathbb F_{11}\) 的七个评价点上，取

$$
f(x)=1+3x+2x^2.
$$

计算完整码字，并验证它至少有 5 个非零坐标。

> **提示：** 使用 Horner rule，并在每一步取模 11。若计算得到的 weight 小于 5，应先检查模运算，因为这会违反已经证明的距离下界。

### 练习 4：系统评价编码

要求 \(f(0)=2,f(1)=6,f(2)=3\)，且 \(\deg f\lt3\)。求 \(f\)，再计算 \(f(3),\ldots,f(6)\)。

> **提示：** 设 \(f(x)=c_0+c_1x+c_2x^2\)，解一个 \(3\times3\) Vandermonde system。检查最终码字前三项是否仍是 \((2,6,3)\)。

### 练习 5：联合纠错预算

`RSCodec(12)` 收到 4 个 unknown errors。最多还能同时处理多少个 erasures？若 erasures 数是 5，距离定理还能保证恢复吗？

> **答案：** \(2\cdot4+s\le12\)，所以最多 4 个 erasures。5 个时预算为 13，超过保证；某次运行成功不能把定理边界改成 13。

### 练习 6：观察越界行为

把文中的 Python 例子改成 `RSCodec(6)`，固定消息后分别注入 3、4、5 个 unknown symbol errors；对多组位置和差值重复实验，并用 SHA-256 比较译码结果与原消息。

> **提示：** 3 errors 在保证内；4、5 errors 越界。记录三类结果：正确恢复、`ReedSolomonError`、返回错误但合法的码字。不要假设三类都一定在小样本中出现。

### 练习 7：GRS 的 Schur product

令

$$
C=\operatorname{GRS}_k(\boldsymbol a,\boldsymbol v).
$$

再令

$$
D=\operatorname{GRS}_\ell(\boldsymbol a,\boldsymbol w).
$$

并令 \(m=\min\{n,k+\ell-1\}\)。证明

$$
C\star D=
\operatorname{GRS}_m
(\boldsymbol a,\boldsymbol v\star\boldsymbol w).
$$

> **提示：** 包含关系来自 \(\deg(fg)\le k+\ell-2\)。反向张成关系可对每个单项式 \(x^d\) 选择 \(x^a\cdot x^b\)，其中 \(a\le k-1\)、\(b\le\ell-1\)、\(a+b=d\)。当次数达到 \(n-1\) 后，评价空间维数饱和为 \(n\)。

## 总结

Reed–Solomon 码的主线来自两个短推导。低次多项式的根数界给出

$$
\deg f\lt k
\Longrightarrow
d=n-k+1.
$$

距离再给出联合保证

$$
2e+s\le n-k.
$$

评价码视角解释“为什么距离最优”，循环码视角解释“工程译码怎样计算”。Berlekamp–Massey 从 syndrome 找递推，Chien search 把根变回位置，Forney algorithm 再恢复错误值。三者串成 syndrome、locator、evaluator 的恢复链，并共同依赖一个事实：错误模式是有限域上少量指数项的和。

`reedsolo` 实验把边界变成了可观察行为：十个 parity symbols 能保证五个未知 errors、十个 erasures，或满足 \(2e+s\le10\) 的组合；长消息则被拆成多个独立码字。越界以后没有正确恢复保证，也没有可靠异常保证，因此 RS 必须与 hash、MAC、framing 和 interleaving 等机制按职责组合。

最后，低次多项式结构并不会在编码完成后消失。它既产生 MDS 距离和高效译码，也产生低维 Schur powers。理解这一点，才真正接上 RS 从“纠错工具”到“密码结构攻击对象”的下一层问题。

## 参考文献

[^reed-solomon]: I. S. Reed and G. Solomon, “Polynomial Codes Over Certain Finite Fields,” *Journal of the Society for Industrial and Applied Mathematics*, 8(2), pp. 300–304, 1960. <https://doi.org/10.1137/0108018>

[^huffman-pless]: W. Cary Huffman and Vera Pless, *Fundamentals of Error-Correcting Codes*, Cambridge University Press, 2003. <https://doi.org/10.1017/CBO9780511807077>

[^blahut]: Richard E. Blahut, *Algebraic Codes for Data Transmission*, Cambridge University Press, 2003. <https://doi.org/10.1017/CBO9780511800467>

[^chien]: R. T. Chien, “Cyclic decoding procedures for Bose-Chaudhuri-Hocquenghem codes,” *IEEE Transactions on Information Theory*, 10(4), pp. 357–363, 1964. <https://doi.org/10.1109/TIT.1964.1053699>

[^forney]: G. D. Forney, “On decoding BCH codes,” *IEEE Transactions on Information Theory*, 11(4), pp. 549–557, 1965. <https://doi.org/10.1109/TIT.1965.1053825>

[^massey]: J. L. Massey, “Shift-register synthesis and BCH decoding,” *IEEE Transactions on Information Theory*, 15(1), pp. 122–127, 1969. <https://doi.org/10.1109/TIT.1969.1054260>

[^reedsolo]: Tomer Filiba et al., `reedsolo`, official repository and stable package documentation. <https://github.com/tomerfiliba-org/reedsolomon>; <https://pypi.org/project/reedsolo/>

[^qr]: DENSO WAVE, “Error Correction Feature,” QRcode.com. <https://www.qrcode.com/en/about/error_correction.html>

[^ccsds]: Consultative Committee for Space Data Systems, *TM Synchronization and Channel Coding*, CCSDS 131.0-B-5. <https://ccsds.org/Pubs/131x0b5.pdf>

[^plank]: James S. Plank, “A tutorial on Reed-Solomon coding for fault-tolerance in RAID-like systems,” *Software: Practice and Experience*, 27(9), pp. 995–1012, 1997. <https://doi.org/10.1002/(SICI)1097-024X(199709)27:9%3C995::AID-SPE111%3E3.0.CO;2-6>

