# 基于 Reed–Solomon 码的公钥密码系统的代数结构攻击


> Reading: 对编码密码来说，“存在高效译码算法”和“公开码看起来像随机码”是两件不同的事。

广义 Reed–Solomon（Generalized Reed–Solomon, GRS）码把低次多项式求值写成线性码，因此同时拥有 MDS 参数、显式生成矩阵和高效译码算法。问题也出在这里：行扰乱矩阵只能更换公开码的基，置换矩阵只能重排坐标；它们没有消除“这是一个求值码”这一事实。

这篇笔记沿一条结构泄漏链展开：先证明 GRS 码的 Schur square 只有 \(2k-1\) 维，再用系统形生成矩阵的 generalized-Cauchy 结构解释等价私钥恢复，最后分析随机列为何能抬高平方码维数、又为何会付出二次量级的密钥膨胀。原始 Sidelnikov–Shestakov 攻击、2014 年的 square-code 方法和 2016 年后的 RLCE 不会被混成同一个算法。

<!--more-->

## Threat Model：攻击者真正拿到了什么

设秘密码 \(C_{\mathrm{sec}}\subseteq \mathbb F_q^n\) 有生成矩阵 \(G_{\mathrm{sec}}\in\mathbb F_q^{k\times n}\)。McEliece 型公钥通常写成

$$
G_{\mathrm{pub}}=S G_{\mathrm{sec}}P,
$$

其中 \(S\in\operatorname{GL}_k(\mathbb F_q)\) 是可逆行扰乱矩阵，\(P\) 是置换矩阵。加密者计算

$$
\boldsymbol c=\boldsymbol mG_{\mathrm{pub}}+\boldsymbol e,
$$

其中 \(\boldsymbol e\) 是低 Hamming weight 的错误向量。私钥持有者撤销坐标置换，调用秘密码译码器恢复码字，再撤销 \(S\) 对消息基的变换。

攻击者看到的并不只是一个矩阵文件，而是两个对象：

- 行空间 \(C_{\mathrm{pub}}=\operatorname{rowspan}(G_{\mathrm{pub}})\)；
- 每一列对应的公开坐标位置。

第一点立刻说明 \(S\) 的能力边界：

$$
\operatorname{rowspan}(S G_{\mathrm{sec}})=\operatorname{rowspan}(G_{\mathrm{sec}}).
$$

因此 \(S\) 没有隐藏码，只是换了一组基。\(P\) 确实隐藏了原坐标顺序，但坐标置换会把码的不变量一起置换，而不会把它们变成随机码的不变量。

> **攻击成功条件**
>
> 攻击者不必唯一恢复密钥生成时抽到的 \(S\)、\(P\)、评价点与列乘因子。只要构造出一个与公开码等价、能够译码的 GRS 表示，就已经得到足以解密的 equivalent decoding trapdoor。

全文的攻击链可以压成：

```text
公开生成矩阵
  -> 公开码的行空间
  -> Schur / puncture / shorten 不变量
  -> 识别或恢复隐藏的求值坐标
  -> 构造等价 GRS 译码陷门
```

## RS 与 GRS：Vandermonde 结构从哪里来

### Reed–Solomon 码

取互异评价点

$$
\boldsymbol a=(a_1,\ldots,a_n)\in\mathbb F_q^n.
$$

对 \(k\le n\)，Reed–Solomon 码是所有次数小于 \(k\) 的多项式在这些点上的求值：

$$
\operatorname{RS}_k(\boldsymbol a)=\left\{(f(a_1),\ldots,f(a_n)):\ f\in\mathbb F_q[X],\ \deg f\lt k\right\}.
$$

按基 \(1,X,\ldots,X^{k-1}\) 展开，得到 Vandermonde 生成矩阵

$$
V_k(\boldsymbol a)=
\begin{pmatrix}
1&1&\cdots&1\\
a_1&a_2&\cdots&a_n\\
a_1^2&a_2^2&\cdots&a_n^2\\
\vdots&\vdots&&\vdots\\
a_1^{k-1}&a_2^{k-1}&\cdots&a_n^{k-1}
\end{pmatrix}.
$$

任意 \(k\) 个互异评价点给出的 \(k\times k\) Vandermonde 子矩阵都可逆，所以 RS 码是参数为 \([n,k,n-k+1]\) 的 maximum-distance separable（MDS）码。

### 广义 Reed–Solomon 码

再取非零列乘因子

$$
\boldsymbol v=(v_1,\ldots,v_n)\in(\mathbb F_q^\times)^n.
$$

定义

$$
\operatorname{GRS}_k(\boldsymbol a,\boldsymbol v)=\left\{(v_1f(a_1),\ldots,v_nf(a_n)):\deg f\lt k\right\}.
$$

其生成矩阵是

$$
G_{\mathrm{GRS}}=V_k(\boldsymbol a)\operatorname{diag}(\boldsymbol v).
$$

列缩放不改变非零位置，因此 GRS 仍是 \([n,k,n-k+1]\) MDS 码。高效译码所利用的正是“一个码字来自同一个低次多项式”的全局约束。

### 参数为什么不唯一

GRS 表示天然存在等价自由度：

- 重排 \((a_i,v_i)\) 只会置换码坐标；
- 同时把所有 \(v_i\) 乘同一个非零标量不会改变码；
- 对支撑点施加射影线性变换 \(x\mapsto(\alpha x+\beta)/(\gamma x+\delta)\)，并相应调整列乘因子，会得到 monomially equivalent 的 GRS 表示。

所以“唯一恢复原始评价点”不是合理的安全目标。密码分析只需要恢复一个等价表示。

## Schur 积：GRS 留下的低维指纹

### 定义

对 \(\boldsymbol x,\boldsymbol y\in\mathbb F_q^n\)，定义坐标乘积（Schur product）

$$
\boldsymbol x\star\boldsymbol y=(x_1y_1,\ldots,x_ny_n).
$$

对两个长度相同的线性码 \(A,B\subseteq\mathbb F_q^n\)，定义

$$
A\star B=\operatorname{span}_{\mathbb F_q}
\{\boldsymbol a\star\boldsymbol b:\boldsymbol a\in A,\boldsymbol b\in B\}.
$$

平方码（square code）是

$$
C^{\star2}=C\star C.
$$

若 \(G\) 的行是 \(\boldsymbol g_1,\ldots,\boldsymbol g_k\)，那么只需对 \(1\le i\le j\le k\) 计算 \(\boldsymbol g_i\star\boldsymbol g_j\)，再做一次 Gaussian elimination，就能求出 \(C^{\star2}\)。候选生成向量最多有

$$
N=\binom{k+1}{2}
$$

个，因此任何 \([n,k]\) 码都满足

$$
\dim C^{\star2}\le \min\left\{n,\binom{k+1}{2}\right\}.
$$

### 定理：同支撑 GRS 码的 Schur 积仍是 GRS 码

> **定理**
>
> 设 \(C=\operatorname{GRS}_k(\boldsymbol a,\boldsymbol v)\)，
> \(D=\operatorname{GRS}_\ell(\boldsymbol a,\boldsymbol w)\)，且二者使用同一组互异评价点，则
>
> $$
> C\star D=
> \operatorname{GRS}_{\min\{n,k+\ell-1\}}
> (\boldsymbol a,\boldsymbol v\star\boldsymbol w).
> $$

**证明。** 取 \(\deg f\lt k\) 与 \(\deg g\lt\ell\)。对应码字的坐标乘积是

$$
(v_if(a_i))_{i=1}^n\star(w_ig(a_i))_{i=1}^n=\bigl(v_iw_i(fg)(a_i)\bigr)_{i=1}^n.
$$

因为 \(\deg(fg)\le k+\ell-2\)，左侧生成的每个向量都落在右侧，故有包含关系“\(\subseteq\)”。

反方向需要说明所有次数不超过 \(k+\ell-2\) 的单项式都由允许的乘积生成。对任意
\(0\le d\le k+\ell-2\)，可选

$$
i=\min\{d,k-1\},\qquad j=d-i.
$$

此时 \(0\le i\lt k\)、\(0\le j\lt\ell\)，且 \(X^d=X^iX^j\)。所以

$$
\operatorname{span}\{fg:\deg f\lt k,\deg g\lt\ell\}=\mathbb F_q[X]_{\lt k+\ell-1}.
$$

当 \(k+\ell-1\le n\) 时，求值映射在该多项式空间上是单射，维数为 \(k+\ell-1\)；超过 \(n\) 后，长度为 \(n\) 的 GRS 码饱和到整个 \(\mathbb F_q^n\)。定理得证。\(\square\)

令 \(C=D\)，立即得到

$$
\dim C^{\star2}=\min\{n,2k-1\}.
$$

### 与随机线性码的差距

对一般位置的随机 \([n,k]\) 码，\(N=\binom{k+1}{2}\) 个对称行乘积通常没有额外代数关系，因此以高概率有[^randomsquare]

$$
\dim C_{\mathrm{rand}}^{\star2}=\min\left\{n,\binom{k+1}{2}\right\}.
$$

这里必须说“通常”或“以高概率”，而不是对每一个随机样本写等号。GRS 的结论则是确定的定理。

若 \(2k-1\lt n\) 且 \(k\ge3\)，两个维数会出现明显缺口：

| 码族 | 平方码维数 |
| --- | --- |
| \(\operatorname{GRS}_k\) | \(\min\{n,2k-1\}\) |
| 一般位置随机 \([n,k]\) 码 | \(\min\{n,\binom{k+1}{2}\}\) |
| GRS 加 \(t\) 个一般位置随机列 | 至多 \(\min\{n+t,2k-1+t,\binom{k+1}{2}\}\) |

### 为什么 \(S\) 与 \(P\) 消不掉这个指纹

\(S\) 不改变码，所以当然不改变平方码。对坐标置换 \(P\)，有

$$
(CP)^{\star2}=C^{\star2}P.
$$

因此平方码的坐标顺序可能改变，维数却保持不变。攻击者完全可以从任意公开生成矩阵出发，计算一个与行基无关、与坐标顺序无关的维数统计量。

## Sidelnikov–Shestakov：从公开码到等价译码陷门

Sidelnikov 与 Shestakov 在 1992 年证明，基于 GRS 码的公开码表示可在多项式时间内恢复出足以译码的结构参数。[^ss] 后续文献常从系统形、最小重量码字、缩短码或支撑关系重述这类恢复。下面不给出原论文的逐行复刻，而给出一个便于手算、与其恢复结论等价的现代代数视图：把公开 GRS 码写成系统形，识别 generalized-Cauchy block，再由 cross-ratio 恢复支撑。

这个区分很重要：本节不是把 1992 年攻击改名为 square-code attack；平方码将在下一节作为后来形成的区分与 filtration 工具出现。Couvreur 等人的论文也明确把自己的 filtration recovery 称为 Sidelnikov–Shestakov 的 alternative，而不是同一算法。[^couvreur]

### 第一步：选信息集并行约化

因为 GRS 是 MDS 码，任意 \(k\) 个坐标都能构成信息集。对公开生成矩阵作坐标重排与行消元，可得

$$
G_{\mathrm{sys}}=[I_k\mid A],
$$

其中 \(A\in\mathbb F_q^{k\times(n-k)}\)。

把信息集对应的支撑记为

$$
X=(x_1,\ldots,x_k),
$$

其余支撑记为

$$
Y=(y_1,\ldots,y_{n-k}).
$$

对信息集使用 Lagrange 基多项式

$$
L_i(Z)=\prod_{m\ne i}\frac{Z-x_m}{x_i-x_m},
$$

则第 \(i\) 个系统形基码字在信息集上等于单位向量，在第 \(j\) 个非信息坐标上的值是

$$
A_{ij}=\frac{v_{k+j}}{v_i}L_i(y_j).
$$

### 第二步：把 \(A\) 化成 generalized-Cauchy form

令

$$
\pi_X(Z)=\prod_{m=1}^k(Z-x_m).
$$

注意

$$
\begin{aligned}
\prod_{m\ne i}(y_j-x_m)
&=\frac{\pi_X(y_j)}{y_j-x_i}\\
&=-\frac{\pi_X(y_j)}{x_i-y_j},
\end{aligned}
$$

且

$$
\prod_{m\ne i}(x_i-x_m)=\pi_X'(x_i).
$$

于是

$$
A_{ij}=\frac{r_i c_j}{x_i-y_j},
$$

其中可以取

$$
r_i=-\frac{1}{v_i\pi_X'(x_i)},\qquad
c_j=v_{k+j}\pi_X(y_j).
$$

这就是 generalized-Cauchy form。未知量分成三类：信息支撑 \(x_i\)、其余支撑 \(y_j\)，以及只依赖行或列的非零因子 \(r_i,c_j\)。

### 第三步：四项比值消去乘因子

取两行 \(i,i'\) 和两列 \(j,j'\)，计算

$$
\rho_{i,i';j,j'}=\frac{A_{ij}A_{i'j'}}{A_{ij'}A_{i'j}}.
$$

代入 Cauchy 形式，所有 \(r_i,r_{i'},c_j,c_{j'}\) 都消失：

$$
\rho_{i,i';j,j'}=\frac{(x_i-y_{j'})(x_{i'}-y_j)}{(x_i-y_j)(x_{i'}-y_{j'})}.
$$

右侧只依赖四个支撑点，是一个 cross-ratio 型射影不变量。这个式子解释了“随机列缩放为何隐藏不了支撑”：缩放只进入 \(r_i,c_j\)，而四项比值专门把它们约掉。

### 第四步：固定射影自由度并解支撑

\(\operatorname{PGL}_2(\mathbb F_q)\) 在射影直线三元组上传递，因此可把三个支撑点规范化为

$$
x_1=0,\qquad x_2=1,\qquad y_1=\infty.
$$

对有限的 \(y_j\)，取 \(i=1\)、\(i'=2\)、\(j'=1\)，按射影极限解释上式，得到

$$
\rho_{1,2;j,1}=\frac{1-y_j}{-y_j}=1-\frac1{y_j}.
$$

只要 \(\rho_{1,2;j,1}\ne1\)，就有

$$
y_j=\frac1{1-\rho_{1,2;j,1}}.
$$

恢复所有 \(y_j\) 后，再固定两个已知 \(y\) 值，使用同类 cross-ratio 方程求每个剩余 \(x_i\)。退化分母可以通过改选行列四元组避免。每一步只是有限域上的加减乘除和线性或低次方程求解。

### 第五步：恢复列乘因子

支撑恢复后，方程

$$
A_{ij}=\frac{r_ic_j}{x_i-y_j}
$$

对行列因子是 rank-one factorization。任选规范 \(r_1=1\)，由第一行求全部 \(c_j\)，再由任一列求其余 \(r_i\)。结合 Lagrange 表达式即可构造一组等价列乘因子。

这里得到的 \((\boldsymbol a',\boldsymbol v')\) 可能不是密钥生成时的原值，但它定义的 GRS 码与公开码 monomially equivalent，已经足以调用 GRS 译码器。

### 攻击伪代码

```text
Algorithm RecoverEquivalentGRS(G_pub)
Input:
    full-rank public generator G_pub over F_q
Output:
    an equivalent GRS support/multiplier pair, or failure

1. Choose a k-coordinate information set and permute it to the front.
2. Row-reduce G_pub to systematic form [I_k | A].
3. For many 2-by-2 submatrices of A, compute
       rho = A[i,j] A[i',j'] / (A[i,j'] A[i',j]).
4. Fix three support points by PGL_2 normalization and solve the
   cross-ratio equations for all remaining support coordinates.
5. Factor A[i,j](x_i-y_j) as r_i c_j and reconstruct equivalent
   column multipliers.
6. Rebuild GRS_k(a',v'), compare its systematic form with G_pub,
   and return the parameters if the codes are equivalent.
```

行约化、四项比值计算、有限域方程求解和最终等价性检验都是多项式时间操作。攻击的致命点不是“猜中了原始随机数”，而是公开码本身已经决定了足够多的代数关系。

## \(\mathbb F_{11}\) 上的 \([6,3]\) 例子

下面的数值全部由独立有限域脚本复算。取

$$
\boldsymbol a=(0,1,2,3,4,5),\qquad
\boldsymbol v=(1,1,1,1,1,1).
$$

Vandermonde 生成矩阵为

$$
G=
\begin{pmatrix}
1&1&1&1&1&1\\
0&1&2&3&4&5\\
0&1&4&9&5&3
\end{pmatrix}\in\mathbb F_{11}^{3\times6}.
$$

选择满秩行扰乱矩阵

$$
S=
\begin{pmatrix}
1&2&0\\
0&1&1\\
1&0&1
\end{pmatrix},
$$

并按索引

$$
(2,5,1,4,0,3)
$$

重排列，得到公开矩阵

$$
G_{\mathrm{pub}}=
\begin{pmatrix}
5&0&3&9&1&7\\
6&8&2&9&0&1\\
5&4&2&6&1&10
\end{pmatrix}.
$$

对前三列行约化：

$$
G_{\mathrm{sys}}=
\left(
\begin{array}{ccc|ccc}
1&0&0&1&2&5\\
0&1&0&6&2&2\\
0&0&1&5&8&5
\end{array}
\right).
$$

此时信息支撑和其余支撑分别是

$$
X=(2,5,1),\qquad Y=(4,0,3).
$$

脚本得到

$$
\boldsymbol r=(4,10,8),\qquad
\boldsymbol c=(5,1,7),
$$

并逐项验证

$$
A_{ij}=\frac{r_ic_j}{x_i-y_j}.
$$

取 \(i=1\)、\(i'=2\)、\(j=1\)、\(j'=2\)。矩阵侧四项比值为

$$
\frac{A_{11}A_{22}}{A_{12}A_{21}}=\frac{1\cdot2}{2\cdot6}=2\pmod{11}.
$$

支撑侧为

$$
\frac{(2-0)(5-4)}{(2-4)(5-0)}=2\pmod{11}.
$$

两者一致，且行列因子完全消失。

最后计算六个对称行乘积

$$
\boldsymbol g_1\star\boldsymbol g_1,\quad
\boldsymbol g_1\star\boldsymbol g_2,\ \ldots,\
\boldsymbol g_3\star\boldsymbol g_3,
$$

其秩为

$$
\dim C^{\star2}=5=2k-1.
$$

而一般位置随机 \([6,3]\) 码的典型平方码维数是

$$
\min\left\{6,\binom42\right\}=6.
$$

这个例子只有一个维度差，但已经把“低维平方码”和“cross-ratio 恢复”两个机制同时显露出来。

## Square Code 区分器：从低维异常到坐标定位

### 最小区分器

给定公开 \([n,k]\) 码 \(C\)，最简单的统计检验是：

1. 计算 \(d_\square=\dim C^{\star2}\)；
2. 计算随机码基线 \(d_{\mathrm{rand}}=\min\{n,\binom{k+1}{2}\}\)；
3. 若 \(d_\square\) 显著低于 \(d_{\mathrm{rand}}\)，输出“存在结构”。

若理想化结构分布总满足 \(d_\square\le d_s\lt d_{\mathrm{rand}}\)，而随机码以至少 \(1-\varepsilon\) 的概率达到 \(d_{\mathrm{rand}}\)，则这个阈值区分器的优势至少是

$$
\begin{aligned}
\operatorname{Adv}_{\mathrm{dist}}
&=\left|\Pr[D(C_{\mathrm{struct}})=1]-\Pr[D(C_{\mathrm{rand}})=1]\right|\\
&\ge1-\varepsilon.
\end{aligned}
$$

低维不是“略微可疑”，而可能给出接近 1 的区分优势。

### 区分器不等于密钥恢复

只知道 \(C\) 不是随机码，还没有得到译码器。密钥恢复还需要回答：

- 哪些坐标来自 GRS 支撑？
- 哪些坐标是随机插入或局部混合的？
- 去除这些坐标后，剩余码是否真的是 GRS？

puncturing 和 shortening 提供了定位接口。

对坐标集合 \(I\)，puncturing \(P_I(C)\) 删除这些坐标；shortening \(S_I(C)\) 先限制码字在 \(I\) 上为零，再删除 \(I\)。攻击者比较

$$
\dim P_I(C)^{\star2}
\quad\text{或}\quad
\dim S_I(C)^{\star2}
$$

随 \(I\) 改变的差分。若删除一个随机列会消除一个额外平方方向，而删除普通 GRS 列只表现为 GRS 长度变化，那么维数差分就成为坐标分类器。

Couvreur 等人在 2014 年系统化了这种思路：先用 component-wise products 构造 distinguisher，再对 punctured/shortened codes 建 filtration，最终恢复支撑与非零乘因子。[^couvreur]

## 随机列掩码：维数缺口需要多少熵

### 理想化追加列模型

先考虑比 RLCE 更简单的模型。令 \(G_{\mathrm{GRS}}\) 是 \(k\times n\) GRS 生成矩阵，追加 \(t\) 个一般位置随机列：

$$
\widetilde G=[G_{\mathrm{GRS}}\mid R]\in\mathbb F_q^{k\times(n+t)}.
$$

从对称平方空间

$$
\operatorname{Sym}^2(\mathbb F_q^k)
$$

看，每个公开坐标都对应一个二次求值泛函。原来 \(n\) 个 GRS 坐标的像空间维数最多是 \(2k-1\)；每增加一个输出坐标，线性映射的秩最多增加 1。因此

$$
\dim \widetilde C^{\star2}
\le
\min\left\{
n+t,\ 2k-1+t,\ \binom{k+1}{2}
\right\}.
$$

要让这个全局维数至少有机会等于随机 \([n+t,k]\) 码的典型值，必须满足

$$
2k-1+t
\ge
\min\left\{
n+t,\binom{k+1}{2}
\right\}.
$$

若 \(n>2k-1\) 且 \(n+t\lt\binom{k+1}{2}\)，两侧之差是

$$
(n+t)-(2k-1+t)=n-(2k-1),
$$

与 \(t\) 无关。也就是说，在随机平方码尚未饱和到对称平方维数之前，单纯追加列无法抹平原有缺口。

当长度已经满足

$$
n+t\ge\binom{k+1}{2},
$$

随机平方码饱和到 \(\binom{k+1}{2}\)。此时必要条件变成

$$
t\ge\binom{k+1}{2}-(2k-1)=\frac{(k-1)(k-2)}2.
$$

> **参数含义**
>
> 对这个理想化模型，只为消除一个全局维数区分器，就可能需要 \(\Theta(k^2)\) 个独立随机坐标。公钥长度和存储随之显著增长。该不等式只是必要下界，不是任意 RLCE 参数的充分安全定理。

当 \(k=3\) 时，下界只有 1；当 \(k=100\) 时，它已经是 \(4851\)。这正是“安全性—密钥尺寸”妥协的代数来源。

### Wieschebrink 型随机列

Wieschebrink 提出的修补方向是在 GRS 生成矩阵中插入少量随机列。[^wies] 2014 年的 square-code 工作直接分析了这类结构：puncture 一个候选随机坐标和 puncture 一个 GRS 坐标，会对平方码维数产生不同影响；定位随机位置后，将其删除，再对剩余 GRS 部分运行结构恢复。

这时攻击路径是：

```text
低维 square distinguisher
  -> puncture / shorten 维数差分
  -> 找到随机列位置
  -> 删除随机列
  -> 对剩余 GRS 码恢复等价私钥
```

### RLCE 不是“只追加几列”

Random Linear Code Encryption（RLCE）确实从随机列开始，但还会把每个秘密码坐标与若干随机坐标组成小块，再右乘局部可逆矩阵，最后全局置换。对 \(r=1\) 的示意块，

$$
[\,\boldsymbol g_i\mid\boldsymbol r_i\,]A_i,
\qquad A_i\in\operatorname{GL}_2(\mathbb F_q).
$$

所有 \(A_i\) 组成 block-diagonal matrix。这个局部混合意味着公开坐标不再分成肉眼可见的“GRS 列”和“随机列”。

RLCE 原论文在 \(n=560,k=380,r=1\) 的实验中，对删除任一列后的公开码计算 product code，报告维数为 \(1119\)，等于相同长度随机码的预期值。[^rlce] 这说明最朴素的全局 square-dimension test 在该实验参数上没有区分力，不能把 Wieschebrink 的攻击结论机械外推为“全局平方码直接击破所有 RLCE”。

但“全局维数饱和”也不等于“结构已经信息论消失”。局部块混合仍规定了哪些坐标来自同一小块；短密钥设置还会压缩可选结构的熵。2019 年，Couvreur、Lequesne 与 Tillich 给出了对 RLCE 短密钥参数的多项式时间 key-recovery attack，并明确声称覆盖作者提出的全部 short-key 参数。[^rlceattack]

因此更准确的结论是：

- square-code 维数是强大的结构诊断工具，但某一全局统计量可以被维数饱和掩盖；
- puncture、shorten、filtration 或块关系仍可能把局部结构重新显露出来；
- “增加随机列”与“增加足够的独立结构熵”不是同一句话；
- 对实际 RLCE 的结论必须绑定具体参数与攻击模型。

## 两类攻击揭示的共同问题

### 简单扰乱只作用于表示，不作用于码

\(S\) 更换行基，\(P\) 重排坐标。只要攻击量是行空间不变量或坐标置换下的协变量，简单扰乱就没有触及安全问题的核心。

### 低熵或局部掩码会留下可枚举边界

少量随机列把平方码维数从 \(2k-1\) 抬高到至多 \(2k-1+t\)，但随机码基线可能是 \(\Theta(k^2)\)。局部 \(2\times2\) 混合虽然打散单列语义，却也引入固定的小块关系。攻击者会寻找的不是原始矩阵，而是任何低复杂度统计偏差。

### “随机外观”必须成为论证，而不是实验印象

设计者至少需要回答：

1. 公钥分布与随机线性码分布之间采用什么 indistinguishability assumption？
2. 已知的 Schur powers、dual、puncture、shorten 和 conductor 操作下，分布是否仍接近随机？
3. 掩码引入的块、秩、准循环或子域结构是否可被局部化？
4. 为消除结构统计量增加的熵，是否已经抵消了紧凑密钥的收益？

## 如何更谨慎地使用代数码

### 使用结构痕迹更弱的码族

经典 McEliece 使用 binary Goppa codes，而不是把 GRS 生成矩阵直接公开。Goppa/alternant 码仍有代数结构，也持续受到 distinguisher 与 key-recovery 研究，但对提交参数的最好已知攻击并没有退化成 plain-GRS 的多项式时间恢复。

MDPC/QC 方案把安全性放在 syndrome decoding、quasi-cyclic decoding 和译码失败概率等问题上。它们不是“无结构随机码”：准循环压缩本身就是结构，BIKE 的 decoder failure rate 与 weak-key 分析正说明结构和译码器仍需共同审计。

秩度量码同样不能作为自动安全答案。Gabidulin/GPT 一类方案曾遭遇 Overbeck 型结构攻击；只有在明确攻击面和掩码效果后，rank metric 才是不同的设计空间，而不是免疫证明。

### 使用更彻底、可证明的再随机化

如果仍要使用求值码，掩码应避免只增加少量列、低秩项或固定小块。可能的方向包括：

- 让公开对象来自更大的随机超码或高余维子码，并证明其分布边界；
- 对评价表示做非局部再随机化，使简单 puncture/shorten 无法隔离结构坐标；
- 让安全规约落在明确的 decoding assumption 上，而不是依赖“攻击者应该认不出来”的直觉；
- 把 Schur powers、dual、conductor 和 filtration 测试纳入参数选择流程。

这些方向各有代价。掩码越彻底，密钥、解密复杂度和失败概率分析通常越难控制。

## NIST PQC：现实影响不能写成错误因果

下面的历史表把不同结论放回正确位置。

| 时间 | 工作 | 可以得出的结论 |
| --- | --- | --- |
| 1992 | Sidelnikov–Shestakov | plain GRS 公钥可在多项式时间恢复等价译码结构 |
| 2010 | Wieschebrink | GRS subcode / 随机列修补仍需结构密码分析 |
| 2014 | Couvreur 等 | square code、puncture、shorten 与 filtration 可把异常维数推进到 key recovery |
| 2016 | RLCE | 随机列加局部块混合能让某些全局 product-code 实验达到随机维数 |
| 2019 | Couvreur–Lequesne–Tillich | 作者提出的 RLCE short-key 参数可被多项式时间恢复 |
| 2025 | NIST IR 8545 | 第四轮只选择 HQC；Classic McEliece 未被选中，但 NIST 仍对其提交参数安全性有信心 |

NIST IR 8545 的表述很克制：Classic McEliece 使用 binary Goppa codes；NIST 认为针对其提交参数的结构攻击仍比 information-set decoding 更昂贵。NIST 没有选择它，主要理由是公钥极大、部署场景有限，而不是 Sidelnikov–Shestakov 攻击已经击破其参数。[^nist]

同一报告选择 HQC，强调其 quasi-cyclic syndrome decoding 假设、稳定的 decryption-failure-rate analysis，以及作为 ML-KEM 补充方案的工程权衡。编码密码没有因为 GRS 失败而退出后量子标准化；真正被淘汰的是“公开结构足够强，却只靠廉价线性掩码隐藏”的设计逻辑。

## 课后思考与练习

### 练习 1：GRS 的 Schur 积

设

$$
C=\operatorname{GRS}_k(\boldsymbol a,\boldsymbol v),\qquad
D=\operatorname{GRS}_\ell(\boldsymbol a,\boldsymbol w).
$$

证明 \(C\star D\) 的公式，并单独讨论 \(k+\ell-1>n\) 时为什么维数饱和到 \(n\)。

> **提示：** 先证明每个次数 \(0\le d\le k+\ell-2\) 的单项式都可写成两个允许次数单项式的乘积，再使用互异点上的求值映射。

### 练习 2：手算一次支撑恢复

使用本文 \(\mathbb F_{11}\) 的系统形矩阵。把三个支撑点规范化为 \(0,1,\infty\)，选择一个非退化四项比值，解出一个剩余支撑坐标。

> **提示：** 使用 \(\rho=1-1/y\)，并记得所有除法都是乘模逆。

### 练习 3：随机列下界

在理想化追加列模型中，从

$$
\dim \widetilde C^{\star2}
\le\min\left\{n+t,2k-1+t,\binom{k+1}{2}\right\}
$$

推出消除全局维数缺口所需的 \(t\) 的必要条件。分别分析 \(n+t\lt\binom{k+1}{2}\) 和 \(n+t\ge\binom{k+1}{2}\)。

> **提示：** 先检查随机码基线是否已经饱和。最后得到的 \((k-1)(k-2)/2\) 依赖“一般位置随机列每列至多贡献一个独立平方方向”等假设。

### 练习 4：区分器如何走向密钥恢复

构造一个小参数实验：在 GRS 生成矩阵中插入两个随机列，逐个 puncture 坐标并记录平方码维数。说明为什么维数异常只给出分类信号，还需要删除随机位置并运行结构恢复才能得到译码器。

> **提示：** 对比 puncture 一个 GRS 坐标与 puncture 一个随机坐标时，平方码秩的变化。

## 总结

GRS 码的安全弱点不是“Vandermonde 矩阵长得整齐”，而是这种结构在换基、置换、列缩放乃至某些局部掩码之后仍留下可计算关系：

$$
\text{低次多项式求值}
\Longrightarrow
\text{低维 Schur powers}
\Longrightarrow
\text{可定位的支撑关系}.
$$

Sidelnikov–Shestakov 告诉我们：隐藏生成矩阵的外观不等于隐藏码。Square-code 方法进一步说明：即使公开码已经不是严格的 GRS，只要少量随机化没有消除 product-code 异常，攻击者仍能由 distinguisher 走向 coordinate recovery。RLCE 的历史则补上另一半：让一个全局统计量达到随机基线并不自动构成安全证明，参数熵和局部块结构仍需独立分析。

编码密码可以安全使用强结构，但前提是结构泄漏被当成一等安全目标，而不是交给 \(S\)、\(P\) 或少量随机列去“遮住”。

## 参考文献

[^ss]: V. M. Sidelnikov and S. O. Shestakov, “On insecurity of cryptosystems based on generalized Reed-Solomon codes,” *Discrete Mathematics and Applications*, 2(4), 1992. <https://doi.org/10.1515/dma.1992.2.4.439>

[^wies]: Christian Wieschebrink, “Cryptanalysis of the Niederreiter Public Key Scheme Based on GRS Subcodes,” *PQCrypto 2010*, LNCS 6061, pp. 61–72. <https://doi.org/10.1007/978-3-642-12929-2_5>

[^couvreur]: Alain Couvreur, Philippe Gaborit, Valérie Gauthier-Umaña, Ayoub Otmani, and Jean-Pierre Tillich, “Distinguisher-based attacks on public-key cryptosystems using Reed–Solomon codes,” *Designs, Codes and Cryptography*, 73(2), pp. 641–666, 2014. <https://doi.org/10.1007/s10623-014-9967-z>; preprint: <https://arxiv.org/abs/1307.6458>

[^randomsquare]: Ignacio Cascudo, Ronald Cramer, Diego Mirandola, and Gilles Zémor, “Squares of Random Linear Codes,” *IEEE Transactions on Information Theory*, 61(3), pp. 1159–1173, 2015. <https://doi.org/10.1109/TIT.2015.2393251>

[^rlce]: Yongge Wang, “Quantum resistant random linear code based public key encryption scheme RLCE,” *IEEE ISIT 2016*, pp. 2519–2523. <https://doi.org/10.1109/ISIT.2016.7541753>; preprint: <https://arxiv.org/abs/1512.08454>

[^rlceattack]: Alain Couvreur, Matthieu Lequesne, and Jean-Pierre Tillich, “Recovering Short Secret Keys of RLCE in Polynomial Time,” *PQCrypto 2019*, pp. 133–152. <https://doi.org/10.1007/978-3-030-25510-7_8>; repository record: <https://inria.hal.science/hal-01959617>

[^nist]: Gorjan Alagic et al., *Status Report on the Fourth Round of the NIST Post-Quantum Cryptography Standardization Process*, NIST IR 8545, March 2025. <https://doi.org/10.6028/NIST.IR.8545>

