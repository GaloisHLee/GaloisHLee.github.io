# 从算术电路到 R1CS、QAP 与 AIR


> Reading: arithmetization as interface design, not notation churn.

前面三篇一直在准备同一件事：证明系统不会直接消费“某个程序跑对了”这种语义句子。它们消费的是某种 algebraic surface，最好能被承诺、挑战、开点、低度测试或者 pairing equation 直接处理。到了这一篇，主角不再是 transcript skeleton，而是 computation 本身如何被翻译成代数约束。

如果只用一句话概括算术化，它就是把“计算正确”改写成“某个 witness 满足一组 field 上的代数关系”。但这句话还不够，因为后面的证明系统并不直接吃“很多条关系”。它们更喜欢的是：

- 向量上的 rank-1 constraints
- 多项式上的恒等式或整除关系
- execution trace 上的 transition constraints

所以这一篇真正要写清楚的不是名词，而是两条链：

$$
\text{circuit} \longrightarrow \text{R1CS} \longrightarrow \text{QAP}
$$

以及

$$
\text{trace} \longrightarrow \text{AIR}.
$$

两条链的共同目标都是同一个：把 computation 变成一个证明系统能消费的 polynomial object。[^thaler] [^groth16] [^stark]

<!--more-->

## 算术化到底在改什么

先固定 statement 形式。

给定公开输入 $x$ 和私有 witness $w$，我们想证明某个计算关系

$$
\mathcal{R}(x, w) = 1.
$$

这里的“计算关系”可以来自程序、电路、状态机或执行轨迹。问题是，大多数现代证明系统并不擅长直接验证程序语义。它们擅长的是在有限域 $\mathbb{F}$ 上检查某种代数条件。

所以算术化做的事不是优化实现细节，而是更换 statement surface。它把原来的问题：

> 是否存在 witness，使某个 computation 正确执行？

翻译成：

> 是否存在 witness，使某个向量约束系统、某个多项式恒等式，或某个 trace 约束系统成立？

这一步一做，后面的证明系统才有抓手。因为承诺方案、开点协议、低度测试、sumcheck、pairing 检查，这些工具本来就是围绕代数对象设计的。

> Key Observation.
> arithmetization 改变的不是 witness 的真假，而是 verifier 最终消费 statement 的接口类型。

## 从算术电路到 R1CS

最常见的起点是 arithmetic circuit。

设所有值都落在某个有限域 $\mathbb{F}$ 上。电路中的门不再是布尔门，而是 field 上的加法门和乘法门。这样做的原因很直接：后面约束系统和多项式都要在同一个 field 上工作。

### 一个极小例子

考虑一个玩具计算：

$$
z = x \cdot y + x.
$$

若把 $x, y$ 当作输入，电路可以拆成两步：

1. 计算中间线

$$
t = x \cdot y
$$

2. 计算输出

$$
z = t + x
$$

这是一个非常小的电路，但已经足够展示 R1CS 与 QAP 的接口。

### Witness Vector Semantics

为了把门级计算收缩成统一约束，我们把所有 wire values 排成一个 witness vector。常见做法是把常数 1 放在最前面，以便把线性项和常数项一起编码。

对上面的例子，可以取

$$
\mathbf{w} = (1, x, y, t, z).
$$

这里每个坐标都在 $\mathbb{F}$ 中。R1CS 之后所有约束都会只看到这个向量，而不再关心“这条值是来自哪根 wire”这种电路图语义。

### Rank-1 Constraint System

R1CS 的基本约束形式是

$$
\langle \mathbf{A}_i, \mathbf{w} \rangle \cdot \langle \mathbf{B}_i, \mathbf{w} \rangle = \langle \mathbf{C}_i, \mathbf{w} \rangle.
$$

也就是：两个线性形式的乘积等于第三个线性形式。

对乘法门 $t = x \cdot y$，可以直接写成：

$$
\langle (0,1,0,0,0), \mathbf{w} \rangle \cdot
\langle (0,0,1,0,0), \mathbf{w} \rangle =
\langle (0,0,0,1,0), \mathbf{w} \rangle.
$$

因为这就是

$$
x \cdot y = t.
$$

对加法关系 $z = t + x$，R1CS 需要把它也写成 rank-1 形式。一个简单写法是

$$
(t + x) \cdot 1 = z,
$$

也就是

$$
\langle (0,1,0,1,0), \mathbf{w} \rangle \cdot
\langle (1,0,0,0,0), \mathbf{w} \rangle =
\langle (0,0,0,0,1), \mathbf{w} \rangle.
$$

因为第一项取出 $x+t$，第二项取出常数 $1$，右边取出 $z$。

### Why R1CS Is Useful

电路原本是 heterogeneous 的：不同门有不同角色，不同 wire 有图结构。R1CS 则把它统一成一张“约束矩阵表”：

- 每一行是一条 rank-1 constraint
- witness vector 是全局共享对象
- 所有约束都只是线性形式与乘积关系

也就是说，电路的图结构在这里被抹平成矩阵与向量接口。

如果共有 $m$ 条约束，记三组矩阵为

$$
A, B, C \in \mathbb{F}^{m \times n},
$$

其中 $n$ 是 witness vector 长度。那么“$\mathbf{w}$ 满足整个 R1CS”就等价于：

$$
\forall i \in \{1,\dots,m\},\quad
\langle A_i, \mathbf{w} \rangle \cdot \langle B_i, \mathbf{w} \rangle = \langle C_i, \mathbf{w} \rangle.
$$

这就是 circuit to R1CS translation 的本质。

## 从 R1CS 到 QAP

R1CS 已经把电路变成了代数约束，但它仍然是“很多行约束”。SNARK 更喜欢的是单一的多项式声明。QAP 做的就是把这些行约束插值成多项式恒等式。

### Rows Become Evaluation Points

设一共有 $m$ 条 R1CS 约束。取域中的 $m$ 个互异点

$$
r_1, r_2, \dots, r_m \in \mathbb{F}.
$$

把第 $i$ 行约束绑定到点 $r_i$。然后对 witness vector 的每一列系数做插值，得到多项式基。

对每个坐标 $j$，构造多项式 $A_j(X), B_j(X), C_j(X)$，使得

$$
A_j(r_i) = A_{i,j},\qquad
B_j(r_i) = B_{i,j},\qquad
C_j(r_i) = C_{i,j}.
$$

于是，对固定 witness $\mathbf{w}$，定义组合多项式

$$
A_{\mathbf{w}}(X) = \sum_{j=1}^n w_j A_j(X),
$$

$$
B_{\mathbf{w}}(X) = \sum_{j=1}^n w_j B_j(X),
$$

$$
C_{\mathbf{w}}(X) = \sum_{j=1}^n w_j C_j(X).
$$

那么在每个约束点 $r_i$ 上，都会有

$$
A_{\mathbf{w}}(r_i) = \langle A_i, \mathbf{w} \rangle,
$$

$$
B_{\mathbf{w}}(r_i) = \langle B_i, \mathbf{w} \rangle,
$$

$$
C_{\mathbf{w}}(r_i) = \langle C_i, \mathbf{w} \rangle.
$$

因此，R1CS 满足性等价于

$$
A_{\mathbf{w}}(r_i) B_{\mathbf{w}}(r_i) - C_{\mathbf{w}}(r_i) = 0
\qquad \text{for all } i.
$$

### Vanishing Polynomial

现在引入 vanishing polynomial：

$$
Z(X) = \prod_{i=1}^m (X - r_i).
$$

一个多项式若在所有 $r_i$ 上都为零，就等价于它可被 $Z(X)$ 整除。所以定义

$$
P_{\mathbf{w}}(X) = A_{\mathbf{w}}(X)B_{\mathbf{w}}(X) - C_{\mathbf{w}}(X).
$$

则“$\mathbf{w}$ 满足所有 R1CS 约束”当且仅当

$$
Z(X) \mid P_{\mathbf{w}}(X).
$$

也就是存在一个 quotient polynomial $H(X)$，使得

$$
A_{\mathbf{w}}(X)B_{\mathbf{w}}(X) - C_{\mathbf{w}}(X) = H(X) Z(X).
$$

这就是 QAP 的核心等式。

### Why The Quotient Polynomial Matters

R1CS 里 verifier 面对的是 $m$ 条约束。QAP 里 verifier 面对的是一个 divisibility claim：某个多项式差值被 vanishing polynomial 整除。

这一步非常重要，因为它把“逐行检查很多约束”压缩成了“检查一个多项式关系”。一旦后面再引入多项式承诺，证明者就不必把整个多项式显式展开给 verifier，而可以承诺它、在随机点开它、证明整除关系成立。

> Formal Definition.
> QAP 不是单纯把矩阵换成多项式；它把 many-row constraint satisfaction 变成 a single polynomial divisibility claim。

这就是为什么 QAP 改变了证明系统接口。它让 SNARK 不再直接消费电路图或矩阵行，而是消费一个可被承诺和开点的多项式对象。

## 一个玩具例子的 QAP 直觉

回到上面的两条约束例子。若我们把两行约束放在点 $r_1, r_2$ 上，那么

$$
Z(X) = (X-r_1)(X-r_2).
$$

第一个约束对应乘法门，第二个约束对应加法门。插值之后，$A_{\mathbf{w}}(X), B_{\mathbf{w}}(X), C_{\mathbf{w}}(X)$ 在 $X=r_1$ 时恢复第一行内积，在 $X=r_2$ 时恢复第二行内积。

于是只要 witness 正确，

$$
P_{\mathbf{w}}(r_1)=0,\qquad P_{\mathbf{w}}(r_2)=0,
$$

自然就得到

$$
P_{\mathbf{w}}(X)=H(X)Z(X).
$$

这里值得记住的不是某个显式插值系数，而是接口变化：

- 电路里我们想的是 gate correctness
- R1CS 里我们想的是 row-wise bilinear constraints
- QAP 里我们想的是 one polynomial identity plus one quotient polynomial

后面 Groth16 正是继续吃这条接口。

## 从执行轨迹到 AIR

QAP 不是唯一算术化路线。STARK 系证明系统走的是另一条主线：不是从静态 constraint matrix 出发，而是从 execution trace 出发。

### Trace Table

设某个计算在离散时间步上展开。可以把每一步的寄存器状态记成一行，把不同寄存器记成列，得到 trace table：

$$
T \in \mathbb{F}^{N \times k},
$$

其中：

- $N$ 是时间步数
- $k$ 是寄存器列数

第 $i$ 行表示第 $i$ 步的系统状态。

与电路视角不同，AIR 直接把 computation 看成“状态如何从一行过渡到下一行”。

### Boundary Constraints

首先需要固定边界条件，也就是某些特定行上的值。例如：

$$
T_{0,1} = x,\qquad T_{N-1,2} = y.
$$

这些约束编码了初始状态、公开输入、终止状态或公开输出。

boundary constraints 的作用类似于电路里“某些 wire 是输入/输出”，但对象已经从静态线值变成了 trace 某几行的列值。

### Transition Constraints

AIR 的核心是 transition constraints。它们描述相邻行之间必须满足的递推关系。

例如，若单列 trace 满足 Fibonacci 递推，可写成

$$
T_{i+2} - T_{i+1} - T_i = 0.
$$

更一般地，若每一步状态由若干列共同更新，transition constraint 会是某个多变量多项式

$$
\Phi\big(T(i), T(i+1), \dots, T(i+\ell)\big)=0
$$

在所有有效步上成立。

这说明 AIR 的 statement surface 不是“某一行满足某个 rank-1 条件”，而是“整张表在相邻行之间满足某些低度递推关系”。

### Polynomial View Of The Trace

和 QAP 一样，AIR 最终也会进入多项式世界。常见做法是把每一列 trace 在某个评估域上插值成列多项式：

$$
t_j(X),\qquad j=1,\dots,k.
$$

然后把 boundary 和 transition constraints 改写成这些列多项式在评估域上的恒等式。为了把多条约束收束成单一低度检查对象，通常还会构造 composition polynomial，把各条约束按随机系数组合在一起。

这里先不深入 FRI 细节，只保留接口：

- trace table 被插值成列多项式
- transition / boundary constraints 被组合成低度约束声明
- 后续 STARK 证明系统验证的是这些多项式对象的低度性和一致性

### Why AIR Feels Different

R1CS/QAP 更像“静态约束系统”：一次性写下所有门级关系。

AIR 更像“时序约束系统”：写下初始条件与状态转移规则，让整段计算轨迹自己展开。

但两者在更高层的目标是一致的：都在把 computation 变成 polynomial constraints。

## QAP 与 AIR 的统一视角

现在可以把两条路线并排看。

### R1CS / QAP

- 从算术电路出发
- 把 witness 编成单个向量
- 把每条门约束压成 rank-1 关系
- 再把很多行约束插值成一个 divisibility claim

### AIR

- 从 execution trace 出发
- 把 witness 编成整张状态表
- 把计算语义写成边界条件与转移条件
- 再把这些条件插值成列多项式与 composition polynomial

从接口角度说，QAP 更接近“constraint system -> polynomial identity”，AIR 更接近“trace system -> low-degree trace constraints”。

> Quick Note.
> SNARK / STARK 的关键差异不只是 commitment scheme 或 trusted setup；更前面的一层差异其实已经出现在它们吃进去的 arithmetization interface 上。

### Why This Changes The Next Article

下一篇讲多项式承诺时，真正的输入就不再是“一个程序”或“一个电路”，而是：

- QAP 里的 $A_{\mathbf{w}}, B_{\mathbf{w}}, C_{\mathbf{w}}, H, Z$
- 或 AIR 里的 trace polynomials 与 composition polynomial

也就是说，多项式承诺一篇并不是突然引入全新对象，而是在消费这一篇已经制造好的 algebraic surface。

## Summary

算术化这一步做的，不是把 computation 换个说法，而是把它翻译成证明系统能消费的代数接口。

对电路路线来说：

1. 先把电路 wire 排成 witness vector
2. 把门级计算写成 R1CS：

$$
\langle A_i,\mathbf{w}\rangle \cdot \langle B_i,\mathbf{w}\rangle = \langle C_i,\mathbf{w}\rangle
$$

3. 再插值成 QAP：

$$
A_{\mathbf{w}}(X)B_{\mathbf{w}}(X) - C_{\mathbf{w}}(X) = H(X)Z(X)
$$

其中 $Z(X)$ 是约束点集合的 vanishing polynomial。

对 trace 路线来说：

1. 先把计算展开成 trace table
2. 用 boundary constraints 和 transition constraints 描述合法执行
3. 再把这些约束插值成列多项式与 composition polynomial

两条路线都在做同一件事：把 computation 改写成 polynomial identity 或 low-degree constraint。后面的 SNARK 与 STARK，不是直接证明“程序跑对了”，而是在证明这些算术化对象满足相应的代数声明。

## References

[^thaler]: Justin Thaler, *Proofs, Arguments, and Zero-Knowledge*, arithmetization related chapters.
[^groth16]: Jens Groth, *On the Size of Pairing-based Non-interactive Arguments*, 2016. QAP-based zkSNARK interface background.
[^stark]: Eli Ben-Sasson et al., *Scalable, Transparent, and Post-Quantum Secure Computational Integrity*, 2018. AIR and low-degree testing background.


