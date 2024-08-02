# WayToFFT Part 1


从拉格朗日插值法到 FFT. Part-1.

>  It was listed by the Science magazine as one of the ten greatest algorithms in the 20th century

从 FT 开始加速多项式乘法。主要记录了笔者学习傅里叶变换这一特殊线性变换时的笔记。

<!--more-->

<br>
<br>

## Fourier Transform

> 傅里叶变换（Fourier Transform）是一种分析信号的方法，它可分析信号的成分，也可用这些成分合成信号。许多波形可作为信号的成分，傅里叶变换用正弦波作为信号的成分。

> 对于时序逻辑可以写作关于时间 $t$ 的函数 $f(t)$ , 傅里叶变换可以检测频率 $\omega$ 的周期在 $f(t)$ 出现的程度。
> $$
> F(\omega)=\mathbb{F}[f(t)]=\int_{-\infty}^\infty f(t)\mathrm{e}^{-\mathrm{i}\omega t}dt
> $$
> 



其作用也可以表示为：**时域映射到频域**
$$
\hat{f} \left ( \xi \right ) = \int _{- \infty }^{\infty }f( x) e^{- 2\pi ix\xi }dx 
$$

$\xi$为任意实数

很多时候，这里的 $\hat{f}\left(\xi\right)$ 会写成 $F(w)$ 或 $F(f)$ 表示角速度或者频率，当然后面的公式的量纲也需要对应的修改；后面的自变量 $x$ 大多数时候都是写成 $t$ 表示时间。


<br>
<br>



## Discrete Fourier Transform

DFT, 离散傅里叶变换。DFT 在工程中是将离散信号从时域转为频域的过程。其表达式可以用于多项式求值，点值固定设计为单位根。



DFT 将一个长度为 $N$ 的复数序列 $x_0,x_1,\dots,x_{N-1}$ 通过如下公式转化为另一等长度复数序列$X_0,X_1,\dots,X_{N-1}$:
$$
X_k = \sum_{n=0}^{N-1} x_n e^{-\frac{2\pi i}{N}kn} \tag1
$$
结合单位根 ([root of unit](https://en.wikipedia.org/wiki/Root_of_unity)) 的概念可得
$$
X_k=\sum_{n=0}^{N-1}x_n\omega_N^{-kn} \tag2
$$
那么，对于已知目标多项式 (系数向量为待变换序列)
$$
f(x)=\sum_{n=0}^{N-1}x_nx^i\tag3
$$
代入得
$$
X_k=\sum_{n=0}^{N-1} x_n (\omega_N^{-k})^n=f(\omega_N^{-k})
$$
说明，所构造多项式 $(3)$ 进行在单位根处求值，即可完成一次离散傅里叶变换。


<br>
<br>



## Fast Fourier Transform

快速傅里叶变换

用于加速多项式乘法。


<br>




### Core idea.

> 我们的问题是，如何用更小的开销实现多项式求值。



不难证明，对于任意一多项式 $f(x)$ ，可分解为奇函数 $f_o(x)$ 和 偶函数 $f_e(x)$ 之和。则：
$$
\begin{cases} f(x) &= f_e(x) + f_o(x) \\\\ f(-x) &= f_e (x) - f_o(x) \end{cases}
$$
注意到 $f_e(x),f_o(x)$ 项数均为 $f(x)$ 的一半，且 $f_o(x) = xf^{\prime}_e(x)$. 又由换元思想，可将元函数二分处理。



即：
$$
f(x)=(a_0+a_2x^2+a_4x^4+\cdots+a_{n-2}x^{n-2})+(a_1x+a_3x^3+a_5x^5+\cdots+a_{n-1}x^{n-1})
$$

令
$$
\begin{cases}f_e(x)=a_0+a_2x+a_4x^2+\cdots+a_{n-2}x^{\frac n2-1} \\\\ f_o(x)=a_1+a_3x+a_5x^2+\cdots+a_{n-1}x^{\frac n2-1} \end{cases}
$$
则有

$$
f(x)=f_e(x^2)+xf_o(x^2)
$$

假设$k<\frac n2$ , 现在要求$f(\omega_n^k)$

$$
\begin{aligned}f(\omega_{n}^{k})&=f_e(\omega_n^2)+\omega_n^k f_o(\omega_n^{2k}) \\\\ &=f_e(\omega_{\frac n2}^k)+\omega_n^kf_o(\omega_{\frac n2}^k)\end{aligned}
$$

这一步转化利用了单位根的性质。

考虑 $f(\omega_n^{k+\frac{1}{2}n})$, 由循环群的性质不难计算：
$$
f(\omega_n^{k+\frac{1}{2}n})=f_e(\omega_{n/2}^k) - \omega_n^k f_o(\omega_{n/2}^k)
$$
令 $k$ 遍历 $[0,n/2 -1]$, 则 $k+n/2$ 遍历 $[0,n-1]$.



即，若已知 $f_e(x),f_o(x)$ 在 $\omega$ 在 $\omega_{\frac n2}^0,\omega_{\frac n2}^1,\ldots,\omega_{\frac n2}^{\frac n2-1}$ 处的点值，就可以在 $O(n)$ 的时间内求得 $f(x)$ 在 $\omega_n^0,\omega_n^1,\ldots,\omega_n^{n-1}$ 处的取值。而关于 $f_e(x)$ 和 $f_o(x)$ 的问题都是相对于原问题规模缩小了一半的子问题，分治的边界为一个常数项$a_{0}$。

根据主定理，该分治算法的时间复杂度为

$$
T(n)=2T(\frac{n}{2})+O(n)=O(n\log n)
$$

最常用的 FFT 算法—— Cooley-Tukey 算法。

> 递归实现的 FFT 效率不高，实际中一般采用迭代实现。




<br>
<br>





##  Inverse Discrete Fourier Transform, IDFT



离散傅里叶逆变换 (Inverse Discrete Fourier Transform, IDFT) 可以视为单位根处**插值**的过程。即给出$n=2^w$个在所有$n$次单位根处的点值

$P_k=(\omega_n^k,f(\omega_n^k))(0\leq k<n)$, 要求还原$f$的各项系数，其中 $f$ 的次数不大于 $n-1$ 。 IDFT和IFFT 之间也存在一些类似差异。



根据前文所提，不难知道拉格朗日插值可以视作范德蒙方阵求逆过程。对于多项式 $f$ 经过 FFT 之后， 再进行 快速傅里叶变换， 仍得到 $f$.

那么对 FFT 的矩阵求逆，可以得到 IFFT 的变换矩阵。
$$
\mathcal{F}=\begin{bmatrix}(\omega^0)^0&(\omega^0)^1&\cdots&(\omega^0)^{n-1} \\\\ (\omega^1)^0&(\omega^1)^1&\cdots&(\omega^1)^{n-1} \\\\ \vdots&\vdots&\ddots&\vdots \\\\ (\omega^{n-1})^0&(\omega^{n-1})^1&\cdots&(\omega^{n-1})^{n-1}\end{bmatrix}
$$

则 $(\mathcal{F_{ij}}^{-1})= [x^i] \prod_{k\neq j} \frac{x-\omega^k}{\omega^j-\omega^k}$ .

进一步分析：
$$
\prod_{0 \le k \lt n} (x-\omega^k) = x^n -1
$$
考虑辅助函数
$$
\begin{aligned} g(x) &= \frac{\prod_{0 \le k \lt n} (x-\omega^k)}{x - \omega^j} \\\\ &= \sum_{0 \le i \le n-1} (\omega^{j})^{i}x^{n-1-i} \end{aligned}
$$
故

$$
\begin{aligned} \mathcal{F_{ij}}^{-1} &= [x^i]\prod_{k\neq j}\frac{x-\omega^k}{\omega^j-\omega^k} \\\\
&= \frac{[x^i]g(x)}{g(\omega^j)} \\\\
&= \frac{(\omega^{-j})^i\omega^{-j}}{n\omega^{-j}}=\frac{\omega^{-ij}}n
\end{aligned}
$$

展开写作
$$
\begin{gathered}\mathcal{F}^{-1}=\frac1n\begin{bmatrix}(\omega^{-0})^0&(\omega^{-0})^1&\cdots&(\omega^{-0})^{n-1} \\\\ (\omega^{-1})^0&(\omega^{-1})^1&\cdots&(\omega^{-1})^{n-1} \\\\ \vdots&\vdots&\ddots&\vdots \\\\ (\omega^{-(n-1)})^0&(\omega^{-(n-1)})^1&\cdots&(\omega^{-(n-1)})^{n-1}\end{bmatrix}\end{gathered}
$$
这就完成了对比分析过程。



上述多少有些 Abuse of notation ，应稍作总结。


<br>



### Summary

<br>



#### 1. Definition

- **Fourier Transform Matrix $\mathcal{F}$**:
  $$
  \mathcal{F}_{N} = \begin{bmatrix}
  \omega_N^{0 \cdot 0} & \omega_N^{0 \cdot 1} & \cdots & \omega_N^{0 \cdot (N-1)} \\\\
  \omega_N^{1 \cdot 0} & \omega_N^{1 \cdot 1} & \cdots & \omega_N^{1 \cdot (N-1)} \\\\
  \vdots & \vdots & \ddots & \vdots \\\\
  \omega_N^{(N-1) \cdot 0} & \omega_N^{(N-1) \cdot 1} & \cdots & \omega_N^{(N-1) \cdot (N-1)}
  \end{bmatrix}
  $$
  where $\omega_N = e^{-\frac{2\pi i}{N}}$ is the $N$-th root of unity.

  

- **Inverse Fourier Transform Matrix \(\mathcal{F}^{-1}\)**:
  $$
  \mathcal{F}^{-1}_{N} = \frac{1}{N} \begin{bmatrix}
  \omega_N^{0 \cdot 0} & \omega_N^{0 \cdot 1} & \cdots & \omega_N^{0 \cdot (N-1)} \\\\
  \omega_N^{1 \cdot 0} & \omega_N^{1 \cdot 1} & \cdots & \omega_N^{1 \cdot (N-1)} \\\\
  \vdots & \vdots & \ddots & \vdots \\\\
  \omega_N^{(N-1) \cdot 0} & \omega_N^{(N-1) \cdot 1} & \cdots & \omega_N^{(N-1) \cdot (N-1)}
  \end{bmatrix}
  $$

一类利用单位根，来做到优化时间复杂度。


<br>



#### 2. FFT and IFFT 

- **FFT**: For a sequence $X_k$, the FFT is defined as:
  $$
  \text{FFT}(x) = X_k = \sum_{n=0}^{N-1} x_n \omega _N^{-kn}
  $$
  
  or using matrix notation:
  
  $$
  X = \mathcal{F}_{N} x
  $$
  
  where $\mathcal{F}_{N}$ is the Fourier transform matrix.

  
  
- **IFFT**: For the sequence $X_k$, the IFFT is given by:
  $$
  x_n = \frac{1}{N} \sum_{k=0}^{N-1} X_k \omega_N^{kn}
  $$

  or using matrix notation:
  
  $$
  x = \mathcal{F}^{-1}_{N} X
  $$

  where $\mathcal{F}^{-1}_{N}$ is the inverse Fourier transform matrix.



<br>



#### 3. Compact Formula for FFT

Using the uniform symbols and subscripts, the FFT formula is:

$$
X_k = \sum_{n=0}^{N-1} x_n \omega _N^{-kn}
$$

In matrix form:

$$
X = \mathcal{F}_{N} x
$$

where $\mathcal{F}_{N}$ is:

$$
\mathcal{F}_{N} = \begin{bmatrix}
\omega_N^{0 \cdot 0} & \omega_N^{0 \cdot 1} & \cdots & \omega_N^{0 \cdot (N-1)} \\\\
\omega_N^{1 \cdot 0} & \omega_N^{1 \cdot 1} & \cdots & \omega_N^{1 \cdot (N-1)} \\\\
\vdots & \vdots & \ddots & \vdots \\\\
\omega_N^{(N-1) \cdot 0} & \omega_N^{(N-1) \cdot 1} & \cdots & \omega_N^{(N-1) \cdot (N-1)}
\end{bmatrix}
$$


<br>



#### 4. Compact Formula for IFFT

The IFFT formula is:

$$
x_n = \frac{1}{N} \sum_{k=0}^{N-1} X_k \omega_N^{kn}
$$

In matrix form:

$$
x = \mathcal{F}^{-1}_{N} X
$$

where $\mathcal{F}^{-1}_{N}$ is:

$$
\mathcal{F}^{-1}_{N} = \frac{1}{N} \begin{bmatrix}
\omega_N^{0 \cdot 0} & \omega_N^{0 \cdot 1} & \cdots & \omega_N^{0 \cdot (N-1)} \\\\
\omega_N^{1 \cdot 0} & \omega_N^{1 \cdot 1} & \cdots & \omega_N^{1 \cdot (N-1)} \\\\
\vdots & \vdots & \ddots & \vdots \\\\
\omega_N^{(N-1) \cdot 0} & \omega_N^{(N-1) \cdot 1} & \cdots & \omega_N^{(N-1) \cdot (N-1)}
\end{bmatrix}
$$


<br>



#### Conclusion

- **FFT**: $X = \mathcal{F}_{N} x$
- **IFFT**: $x = \mathcal{F_{N}}^{-1} X = \frac{1}{N} \mathcal{F_{N}}^{\dagger} X$

Where $\mathcal{F_{N}^{\dagger}}$ denotes the conjugate transpose of $\mathcal{F_{N}}$.



Note that where:

- $X = [X_0,X_1,...,X_{N-1}]^T$ is the frequency domain representation.
- $x = [x_0,x_1,...,x_{N-1}]^T$ is the time domain sequence.
- $\omega_N = e^{- \frac{2\pi i}{N}}$ is  the $N$-th root of unity.
- $\mathcal{F}_N$ is the $N\times N$ Fourier transform matrix, whose conjugate transpose used for IFFT.


<br>
<br>



## Fast polynomial multiplication

Fast polynomial multiplication of two polynomials.

For two polynomials:

$$
\begin{align}p(x)&= a_0+a_1x+\cdots+a_{n-1}x^{n-1},\\\\ q(x)&= b_0+b_1x+\cdots+b_{n-1}x^{n-1} \end{align}
$$

Their product

$$
(p\cdot q)(x)=p(x)\cdot q(x)=c_0+c_1x+\cdots c_{2n-2}x^{2n-2}
$$

where
$$
c_i = \sum_{\max \set{0,i-(n-1)}\le k \le \min \set {i,n-1}} a_k b_{i-k}
$$


<br>



### Algorithm



1. Evaluate $p(x)$ and $q(x)$ at $2n$ points $\omega_{2n}^0,...,\omega_{2n}^{2n-1}$ using DFT.   

2. Obtain the values of $p(x)q(x)$ at these $2n$ points through pointwise multiplication

$$
\begin{aligned}
(p\cdot q)(\omega_{2n}^{0})& =\quad p(\omega_{2n}^0)\cdot q(\omega_{2n}^0), \\\\
(p\cdot q)(\omega_{2n}^1)& =\quad p(\omega_{2n}^1)\cdot q(\omega_{2n}^1), \\\\
&... \\\\
(p\cdot q)(\omega_{2n}^{2n-1})& =\quad p(\omega_{2n}^{2n-1})\cdot q(\omega_{2n}^{2n-1}). 
\end{aligned}
$$

3. Interpolate the polynomial $p \cdot q$ at the product values using IDFT to obtain $c_0,..,c_{2n-2}$.

步骤1, 2, 3 对应时间复杂度为 $\Theta(n\log n),\Theta(n\log n).\Theta(n).$

代数角度来看，是利用单位根元素构成的特殊代数结构及其性质完成的策略。

不难联想到，利用 FFT 也可以计算 两向量卷积。

卷积（Convolution）:
$$
a = (a_0,...,a_{n-1}) \quad and\quad b = (b_0,...,b_{n-1})
$$
定义卷积所得向量 $c$
$$
c_j = \sum_{k=0}^j a_k b_{j-k}, j = 0,...,n-1 
$$
时间复杂度也为 $\Theta(n\log n)$.



<br>
<br>
<br>
<br>





# Reference

- [Fast polynomial multiplication](https://course.ccs.neu.edu/csu690/notes/fast-poly-mult.html)
- [Polynomial: From FT to NTT](https://www.cnblogs.com/alex-wei/p/Polynomial___Lagrange_Interpolation_and_Fast_Fourier_Transform.html)
- [Polynomial Multiplication and Fast Fourier Transform (Com S 477/577 Notes) Yan-Bin Jia. Sep 20, 2022](https://faculty.sites.iastate.edu/jia/files/inline-files/polymultiply.pdf)
- [Number Theoretic Transform - CCRMA, Stanford](https://ccrma.stanford.edu/~jos/st/Number_Theoretic_Transform.html)
- [A Complete Beginner Guide to the Number Theoretic Transform (NTT)](https://eprint.iacr.org/2024/585.pdf)


