# WayToFFT Part 2


# Way To NTT......

> Note

The Number Theoretic Transform (NTT) is a powerful mathematical tool that has become increasingly important in developing Post Quantum Cryptography (PQC) and Homomorphic Encryption (HE).

<!--more-->

![image-20250808154030770](https://s2.loli.net/2025/08/08/SgVeCWoRObTpEJr.png)



## Review

Maybe those are all you need [link1](https://halois.top/waytofft-part-0/),[link2](https://halois.top/waytofft-part-1/).

### Notation 

> - Textbook-style Convolution
> - linear convolution
> - cyclic convolution
> - Negacyclic Convolution

### Review - Linear Convolution 



#### Converting Coefficients Multiplication into Vector Multiplication.

**Polynomial multiplication:**

> Having $G(x)$ and $H(x)$ as polynomials of degree $n-1$ in the ring $\mathbb{Z_q} \[x\]$ where q $\in \mathbb{Z}$ and $x$ is the polynomial variable. Generally speaking, a **Polynomial Multiplication** of $G(x)$ and $H(x)$ is defined as:
> $$Y(x) = G(x)\cdot H(x) = \sum_{k=0}^{2(n-1)}y_kx^k$$
> where $y_k = \sum_{i=0}^{k} g_i h_{k-i} \mod q$, $g$ and $h$ are the polynomial coefficients of $G(x)$ and $H(x)$ respectively.

**About $y_k = \sum_{i=0}^{k} g_i h_{k-i} \mod q$**

Simply This is merely **a symbolic expression** used to describe the final result. 

Think about what happens when you manually multiply two polynomials, $G(x)$ and $H(x)$. How do you get the final term for $x^k$?

You get it by finding every pair of terms—one from $G(x)$ and one from $H(x)$—whose exponents add up to exactly $k$.

Specifically, to get an $x^k$ term, you must multiply a term like $g_ix^i$ from G(x) with a term like $h_jx^j$ from $H(x)$ where $i+j=k$.

The formula is simply a systematic way of finding and adding up all these pairs.
$$
y_k = \sum_{i=0}^{k} g_i h_{k-i}
$$
This formula iterates through all the possibilities:

- **When $i=0$**, it pairs the constant term from $G(x)$ (coefficient $g_{0}$) with the $x^k$ term from $H(x)$ (coefficient $h^k$). This gives you the product $g_{0}h^{k}$.
- **When $i=1$**, it pairs the $x^1$ term from $G(x)$ (coefficient $g_{1}$) with the $x^{k−1}$ term from $H(x)$ (coefficient $h^{k−1}$). This gives you $g_{1}h^{k−1}$.
- ...and so on.
- This continues until $i=k$, pairing the $x^{k}$ term from $G(x)$ (coefficient $g_{k}$) with the constant term from $H(x)$ (coefficient $h_{0}$), which gives you $g_{k}h^{0}$.

The final coefficient, $y_k$, is just the sum of all these products. The formula is the compact, formal way of writing this exact process, which is also known as a **discrete convolution**.

So, just import some notation about coefficient vector. We can convert **Coefficients Multiplication into Vector Multiplication.**

$$
y[k]= \boldsymbol{g} \star \boldsymbol{h}[k] = \sum_{i=0}^{k}g[i]h[k-i]
$$

Of course. The formula you've shown is a **discrete linear convolution**. While it's not a simple one-to-one vector multiplication, it can be understood as a form of vector multiplication in three main ways.

---
**Understood as a Series of Dot Products**

We can think of the calculation of **each** output element `y[k]` as a single vector **dot product**.

The formula is: $y[k] = \sum_{i=0}^{k} g[i]h[k-i] = g[0]h[k] + g[1]h[k-1] + \dots + g[k]h[0]$

To calculate `y[k]`, we need two vectors:
1.  The first `k+1` elements of vector `g`: `[g[0], g[1], ..., g[k]]`
2.  The first `k+1` elements of vector `h`, but **reversed**: `[h[k], h[k-1], ..., h[0]]`

Then, `y[k]` is simply the dot product of these two vectors.

**Intuition:**
This is known as the "Flip and Slide" method. Imagine flipping the vector `h` and then sliding it across the vector `g` one step at a time. At each step, the dot product of the overlapping parts is calculated, which gives one element of the resulting vector `y`.



**Understood as Matrix-Vector Multiplication**

This is the most direct algebraic representation. We can expand one of the vectors (e.g., `g`) into a special matrix called a **Toeplitz Matrix** or **Convolution Matrix**. The entire convolution operation then becomes a single **matrix-vector multiplication**.

Let's assume `g = [g₀, g₁, g₂]` and `h = [h₀, h₁, h₂]`.
The result of the linear convolution, `y`, will have a length of (3+3-1) = 5. We can construct the following matrix-vector multiplication:



$$
\begin{pmatrix}
y_0 \\\\ y_1 \\\\ y_2 \\\\ y_3 \\\\ y_4 
\end{pmatrix} =
\begin{pmatrix}
g_0 & 0 & 0 \\\\
g_1 & g_0 & 0 \\\\
g_2 & g_1 & g_0 \\\\
0 & g_2 & g_1 \\\\
0 & 0 & g_2
\end{pmatrix}
\begin{pmatrix}
h_0 \\\\
h_1 \\\\
h_2
\end{pmatrix}
$$

If you work out the matrix multiplication, you will find that the results match the convolution definition exactly:
- $y_0 = g_0 h_0$
- $y_1 = g_1 h_0 + g_0 h_1$
- $y_2 = g_2 h_0 + g_1 h_1 + g_0 h_2$
...and so on.

This method packages the entire convolution process into a single linear transformation, which is the clearest "vector multiplication" representation from a linear algebra perspective.

# Review - Cyclic Convolution

Positive Wrapped Convolution as $PWC(x)$ 

> Suppose that $G(x)$ and $H(x)$ are polynomials of degree $n-1$ in the quotient ring $\mathbb{Z_{q}}[x]/(x^{n}-1)$ where $q \in \mathbb{Z}$.
> A **Cyclic Convolution or positive wrapped convolution is defined as:**
> $$PWC(x) = \sum_{k=0}^{n-1} c_{k}x^{k}$$
> where $c_{k}= \sum_{i=0}^{k}g_{i}h_{k-i} + \sum_{i=k+1}^{n-1} g_{i}h_{k+n-i}$. If $Y(x)$ is the result of their linear convolution in the ring $\mathbb{Z}_{q}[x]$, it also can be defined as:
> $$PWC(x) = Y(x) \mod (x^{n} - 1)$$

### Review - Negacyclic Convolution 

>**Definition 2.3.** Suppose that $G(x)$ and $H(x)$ are polynomials of degree $n-1$ in the quotient ring $\mathbb{Z}[x]/(x^{n}+1)$ where $q \in \mathbb{Z}$. A **negacyclic convolution** or **negative wrapped convolution**, $\operatorname{NWC}(x)$ is defined as:  
>$$
 \operatorname{NWC}(x)=\sum_{k=0}^{n-1}c_{k}x^{k} $$
 where $c_{k}=\sum_{i=0}^{k}g_{i}h_{k-i}-\sum_{i=k+1}^{n-1}g_{i}h_{k+n-i} \mod q$. If $Y(x)$ is the result of their linear convolution in the ring $\mathbb{Z}[x]$, it also can be defined as 
 > $$ \operatorname{NWC}(x)=Y(x) \bmod (x^{n}+1)$$

**Key Observation**

![image.png](https://s2.loli.net/2025/08/09/pfsojKguXwvbd57.png)

 

  The above is a brief description of polynomial multiplication.



### Review - NTT-Based Convolution.

Number Theoretic Transform Based on $\omega$


### Review -  Fast NTT







## summary

FFT-style NTT algorithm or Fast-NTT is particularly useful in lattice-based cryptography.

:)







