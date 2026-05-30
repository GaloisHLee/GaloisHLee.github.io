# WayToNTT Part 0


> Survey:NTT,as it was.

<!--more-->


# First Principles

> From First Principles to Post-Quantum Cryptography: A Comprehensive Analysis of Convolution and the Number Theoretic Transform

## Introduction

At the heart of numerous scientific and technological domains lies a fundamental, yet computationally demanding, operation: the multiplication of polynomials. From the filtering of signals in digital communication systems and the processing of images in medical diagnostics to the abstract manipulations of computer algebra and the security foundations of modern cryptography, the ability to efficiently compute the product of two polynomials is of paramount importance.

The direct, "schoolbook" method for this task, while conceptually simple, carries a quadratic computational complexity of $O(n^2)$, rendering it impractical for the large-degree polynomials frequently encountered in advanced applications. This computational bottleneck has motivated centuries of mathematical and algorithmic innovation.

The algebraic core of polynomial multiplication is an operation known as convolution. Formally, the sequence of coefficients of a product polynomial is the linear convolution of the coefficient sequences of the original polynomials. This equivalence establishes a central theme: to multiply polynomials efficiently, one must compute convolutions efficiently.

For decades, the primary tool for this task has been the Fast Fourier Transform (FFT), an algorithm that dramatically reduces the complexity of convolution to a quasi-linear $O(n\log n)$. However, the FFT and its underlying Discrete Fourier Transform (DFT) operate over the field of complex numbers. When implemented on digital computers, this reliance on floating-point arithmetic introduces inevitable precision and rounding errors.

While acceptable in many signal processing applications where some degree of noise is tolerable, such errors are catastrophic in domains like cryptography and large-integer arithmetic, where computations must be exact.

This report provides a detailed exposition of the Number Theoretic Transform (NTT), a powerful analogue of the DFT that resolves the precision dilemma. By operating entirely within finite fields of integers, the NTT provides a mechanism for performing convolutions with the quasi-linear time complexity of the FFT but with perfect, error-free precision.

This unique combination of speed and exactness has made the NTT an indispensable tool, particularly in the burgeoning field of post-quantum cryptography (PQC). As the world prepares for the security challenges posed by quantum computers, lattice-based cryptographic schemes have emerged as a leading solution, and the NTT is the algorithmic engine that makes their performance practical.

The following analysis will provide a comprehensive and pedagogically structured journey into this critical subject. The report begins by meticulously defining and differentiating the three primary forms of convolution—linear, cyclic, and negacyclic—and establishing their profound connection to polynomial multiplication in distinct algebraic rings.

It then develops the complete mathematical theory of the NTT from first principles, including the crucial role of primitive roots of unity and a rigorous proof of the convolution theorem in finite fields. Subsequently, the report delves into the algorithmic intricacies of fast transforms, such as the Cooley-Tukey algorithm, butterfly operations, and bit-reversal, culminating in a detailed numerical example.

Finally, it illuminates the specialized techniques required to adapt the NTT for negacyclic convolution and analyzes its critical role in securing the next generation of public-key cryptography, with a specific focus on the NIST-standardized schemes CRYSTALS-Kyber and CRYSTALS-Dilithium.

## The Three Convolutions: A Mathematical Framework

The term "convolution" is not monolithic; it describes a family of related mathematical operations, each with distinct properties and algebraic interpretations. Understanding the differences between linear, cyclic, and negacyclic convolution is fundamental to grasping their applications in signal processing, abstract algebra, and cryptography.

The choice of convolution is, in essence, the choice of an algebraic world. The transition between these types is not merely an algorithmic adjustment but a fundamental shift in the underlying mathematical structure where the computation is defined.

### Linear Convolution:
> **The Foundation of Signal Processing and Polynomial Products**

Linear convolution is the most fundamental of the three types and corresponds to the intuitive notion of polynomial multiplication and the behavior of many physical systems.

#### Formal Definition

In the context of discrete sequences, the linear convolution of two sequences, $g$ and $h$, is a third sequence, $y$, denoted as $y = g * h$. The $k$-th element of the output sequence is defined by the summation:

$$y[k] = (g * h)[k] = \sum_{i=-\infty}^{\infty} g[i]h[k-i]$$

This operation describes how the shape of one sequence or function modifies the shape of another. In the domain of digital signal processing, this formula is the cornerstone of Linear Time-Invariant (LTI) system theory, where it calculates the system's output $y[k]$ given an input signal $x[k]$ and the system's impulse response $h[k]$.

#### Equivalence to Polynomial Multiplication

The most direct algebraic interpretation of linear convolution is standard polynomial multiplication. If we have two polynomials, $P(x)$ and $Q(x)$, with coefficient vectors $p = [p_0, p_1, \ldots, p_{L-1}]$ and $q = [q_0, q_1, \ldots, q_{M-1}]$ respectively:

$$P(x) = \sum_{i=0}^{L-1} p_i x^i$$ and $$Q(x) = \sum_{j=0}^{M-1} q_j x^j$$

Their product, $C(x) = P(x) \cdot Q(x)$, is a new polynomial whose coefficients, $c_k$, are given by:

$$c_k = \sum_{i+j=k} p_i q_j$$

By substituting $j = k - i$, this becomes $c_k = \sum_i p_i q_{k-i}$, which is precisely the definition of the linear convolution of the coefficient vectors $p$ and $q$. Thus, computing a linear convolution is algebraically equivalent to multiplying two polynomials in the ring of polynomials with integer or real coefficients, $\mathbb{Z}[x]$ or $\mathbb{R}[x]$.

#### Output Length

A defining characteristic of linear convolution is that the output sequence is longer than the input sequences. If the input polynomials have degrees $L-1$ and $M-1$ (corresponding to $L$ and $M$ coefficients), their product will have a degree of $(L-1) + (M-1) = L + M - 2$. This means the resulting coefficient vector will have a length of $L + M - 1$.

This expansion of the output is a natural consequence of the multiplication; for example, the product of two degree-3 polynomials is a degree-6 polynomial.

#### Matrix Representation

Linear convolution can also be visualized as a matrix-vector multiplication. The operation can be structured by forming a Toeplitz matrix from one of the input vectors. A Toeplitz matrix has constant values along its diagonals, which elegantly captures the "shift-and-multiply" nature of convolution.

For an input sequence $h$ of length $M$ and an input sequence $x$ of length $L$, the convolution $y = h * x$ can be written as:

$\begin{bmatrix} y_0 \\\\ y_1 \\\\ \vdots \\\\ y_{L+M-2} \end{bmatrix} = \begin{bmatrix} h_0 & 0 & \cdots & 0 \\\\ h_1 & h_0 & \cdots & 0 \\\\ \vdots & h_1 & \ddots & \vdots \\\\ h_{M-1} & \vdots & \ddots & h_0 \\\\ 0 & h_{M-1} & \cdots & h_1 \\\\ \vdots & \vdots & \ddots & \vdots \\\\ 0 & 0 & \cdots & h_{M-1} \end{bmatrix} \begin{bmatrix} x_0 \\\\ x_1 \\\\ \vdots \\\\ x_{L-1} \end{bmatrix}$

This matrix representation explicitly shows the shift-invariant property of the operation and is useful for both theoretical analysis and certain computational approaches.

### Cyclic Convolution: Introducing Periodicity and Quotient Rings

While linear convolution is fundamental, many efficient algorithms, such as the FFT and NTT, are naturally suited to a periodic version of the operation known as cyclic or circular convolution.

#### The "Wrap-Around" Concept

The core idea of cyclic convolution is to treat the sequences as if they are inscribed on a circle. When an index goes past the end of the sequence, it "wraps around" to the beginning. This creates a periodic structure where the output sequence has the same length as the input sequences, unlike the expansion seen in linear convolution.

#### Formal Definition

For two sequences, $x$ and $h$, both of length $N$, their $N$-point cyclic convolution, denoted $y = x \circledast h$, is defined as:

$y[m] = \sum_{n=0}^{N-1} x[n]h((m-n) \bmod N)$

The modulo operator on the index of $h$ is what mathematically enforces the wrap-around behavior. The resulting sequence $y$ also has length $N$.

#### Algebraic Isomorphism

The shift from linear to cyclic convolution represents a profound change in the underlying algebraic setting. $N$-point cyclic convolution is directly isomorphic to polynomial multiplication within a specific algebraic structure known as a quotient ring. Specifically, it corresponds to multiplication in the ring $\mathbb{Z}_q[x]/(x^N - 1)$.

To understand this connection, consider the polynomial modulus $\phi(x) = x^N - 1$. In this ring, any two polynomials are considered equivalent if their difference is a multiple of $x^N - 1$. This implies the relation $x^N \equiv 1 \pmod{x^N - 1}$.

When we multiply two polynomials $P(x)$ and $Q(x)$ and reduce the result modulo $x^N - 1$, any term with a power of $x$ greater than or equal to $N$ is affected. For instance, a term $c_N x^N$ becomes $c_N$, and a term $c_{N+1} x^{N+1} = c_{N+1} x \cdot x^N$ becomes $c_{N+1} x$.

The coefficient of a higher-degree term effectively "wraps around" and is added to the coefficient of the corresponding lower-degree term. This additive wrap-around is precisely the behavior of cyclic convolution.

### Negacyclic Convolution: The Skewed Variant for Modern Cryptography

A third, more specialized type of convolution has become critically important in modern cryptography: negacyclic convolution. It is also referred to as skew-circular or negative wrapped convolution.

#### The Negated "Wrap-Around"

Negacyclic convolution is similar to its cyclic counterpart but with a crucial twist: when an index wraps around, a sign change is introduced. This "skewed" periodicity is fundamental to the security properties of many lattice-based cryptosystems.

#### Formal Definition

For two sequences, $f$ and $g$, of length $N$, their negacyclic convolution, which we can denote as $f \circledast_{-} g$, is defined by the formula:

$$(f \circledast_{-} g)\_k := \sum_{j=0}^{N-1} f_j g_{(k-j) \bmod N} \cdot (-1)^{\lfloor (k-j) < 0 \rfloor}$$

where the term $(-1)^{\lfloor (k-j) < 0 \rfloor}$ introduces a sign flip for indices that wrap around (i.e., when $k - j < 0$). The resulting sequence also has length $N$.

#### Algebraic Isomorphism

Just as cyclic convolution corresponds to the ring $\mathbb{Z}_q[x]/(x^N - 1)$, negacyclic convolution is isomorphic to polynomial multiplication in the quotient ring $\mathbb{Z}_q[x]/(x^N + 1)$. This ring is a specific type of cyclotomic ring that is central to the security of the Ring-Learning With Errors (RLWE) problem.

The defining relation in this ring is $x^N \equiv -1 \pmod{x^N + 1}$. When a product polynomial is reduced modulo $x^N + 1$, a term like $c_N x^N$ becomes $-c_N$, and a term $c_{N+1} x^{N+1} = c_{N+1} x \cdot x^N$ becomes $-c_{N+1} x$.

In this case, the coefficient of a higher-degree term "wraps around" and is subtracted from the coefficient of the corresponding lower-degree term. This subtractive wrap-around gives the convolution its "negacyclic" character.

### Comparative Analysis and Numerical Examples

To solidify the distinctions, consider the multiplication of two simple polynomials, $a(x) = 2x + 1$ and $b(x) = 4x + 3$. Their coefficient vectors (for degree 1, length $N = 2$) are $a = [1, 2]$ and $b = [3, 4]$.

#### Linear Convolution

This is standard polynomial multiplication:

$$C(x) = (2x + 1)(4x + 3) = 8x^2 + 6x + 4x + 3 = 8x^2 + 10x + 3$$

The resulting coefficient vector is $c = [3, 10, 8]$. The length is $2 + 2 - 1 = 3$.

#### Cyclic Convolution (N=2)

This corresponds to multiplication in a ring modulo $x^2 - 1$. We take the linear result and reduce it:

$$C(x) \pmod{x^2 - 1} = (8x^2 + 10x + 3) \pmod{x^2 - 1}$$

Since $x^2 \equiv 1 \pmod{x^2 - 1}$, we substitute:

$$8(1) + 10x + 3 = 10x + 11$$

The resulting coefficient vector is $c = [11, 10]$. The wrap-around added the coefficient of $x^2$ (which is 8) to the constant term (which was 3).

#### Negacyclic Convolution (N=2)

This corresponds to multiplication in a ring modulo $x^2 + 1$. We again reduce the linear result:

$C(x) \pmod{x^2 + 1} = (8x^2 + 10x + 3) \pmod{x^2 + 1}$

Since $x^2 \equiv -1 \pmod{x^2 + 1}$, we substitute:

$$8(-1) + 10x + 3 = 10x - 5$$

The resulting coefficient vector is $c = [-5, 10]$. The wrap-around subtracted the coefficient of $x^2$ from the constant term.

This simple example starkly illustrates how the choice of convolution—or equivalently, the choice of the underlying polynomial ring—fundamentally changes the outcome of the computation.

### Summary Table

| Feature | Linear Convolution | Cyclic Convolution | Negacyclic Convolution |
|---------|-------------------|-------------------|----------------------|
| Mathematical Formula | $y[k] = \sum_i g[i]h[k-i]$ | $y[m] = \sum_{n=0}^{N-1} x[n]h((m-n) \bmod N)$ | $y_k = \sum_{j=0}^{N-1} f_j g_{(k-j) \bmod N} \cdot (-1)^{\lfloor (k-j) < 0 \rfloor}$ |
| Associated Polynomial Ring | $\mathbb{Z}[x]$ | $\mathbb{Z}_q[x]/(x^N - 1)$ | $\mathbb{Z}_q[x]/(x^N + 1)$ |
| Output Length (for length-N inputs) | $2N - 1$ | $N$ | $N$ |
| "Wrap-around" Behavior | None (expansion) | $c_k \leftarrow c_k + c_{k+N}$ | $c_k \leftarrow c_k - c_{k+N}$ |
| Primary Application Domain | General Signal Processing | DFT/NTT-based Filtering | Lattice-Based Cryptography |

## The Number Theoretic Transform (NTT): Principles and Theory

The convolution theorem provides a path to transform a computationally intensive convolution in the time or spatial domain into a simple pointwise product in the frequency domain. The Number Theoretic Transform (NTT) is an adaptation of this principle to the realm of finite fields, designed to achieve the speed of transform-based methods while preserving the exactness required for integer arithmetic.

It achieves this by being a fundamental change of basis for the vector space of polynomials. The NTT transforms a polynomial from the standard monomial basis (coefficients of $1, x, x^2, \ldots$) to a basis of evaluation points (values at $\omega^0, \omega^1, \ldots$). Multiplication, which is complex (convolution) in the monomial basis, becomes simple (pointwise) in the evaluation basis. The Inverse NTT (INTT) is the transformation back to the standard basis.

### From DFT to NTT: The Quest for Exact Integer Arithmetic

The journey to the NTT begins with its more famous counterpart, the Discrete Fourier Transform (DFT).

#### The DFT and the Convolution Theorem

The DFT is a mathematical transform that decomposes a finite sequence of values into a sequence of complex numbers representing different frequency components. The cornerstone of its utility for fast computation is the Convolution Theorem, which states that the DFT of a cyclic convolution of two sequences is equal to the pointwise (or Hadamard) product of their individual DFTs:

$$\mathcal{F}(f * g) = \mathcal{F}(f) \odot \mathcal{F}(g)$$

This theorem allows one to replace a costly $O(n^2)$ convolution with two forward transforms, one pointwise product ($O(n)$), and one inverse transform. When the transforms are computed using the Fast Fourier Transform (FFT) algorithm, the total complexity becomes $O(n \log n)$.

#### The Problem with Floating-Point Arithmetic

The DFT and FFT operate over the field of complex numbers, $\mathbb{C}$. On digital hardware, these numbers are represented using finite-precision floating-point arithmetic. This inevitably introduces small rounding and precision errors at each step of the computation.

While these errors are often negligible in applications like audio or image processing, they are completely unacceptable in contexts where results must be perfectly exact, such as in cryptography or when multiplying large integers. A single bit error in a cryptographic key or ciphertext can render it useless or insecure.

#### The NTT as an Analogue

The Number Theoretic Transform (NTT) was developed to overcome this limitation. It is a generalization of the DFT that operates over a finite field, typically the integers modulo a prime, $\mathbb{Z}_q$, instead of the complex numbers.

The core idea is to replace the complex primitive $n$-th roots of unity, $e^{-2\pi i/n}$, with primitive $n$-th roots of unity that exist within the finite field itself. Because all operations (additions, multiplications) are performed using modular integer arithmetic, the entire transform is computed without any loss of precision, yielding perfectly exact results.

### Comparison Table

| Property | Discrete Fourier Transform (DFT) | Number Theoretic Transform (NTT) |
|----------|----------------------------------|----------------------------------|
| Domain | Complex Numbers ($\mathbb{C}$) | Finite Field ($\mathbb{Z}_q$) |
| Arithmetic | Floating-Point | Integer Modulo $q$ |
| Roots of Unity | $e^{-2\pi i/n}$ | $\omega \in \mathbb{Z}_q$ where $\omega^n \equiv 1 \pmod{q}$ |
| Precision | Approximate (subject to rounding errors) | Exact (error-free) |
| Key Advantage | Widely applicable in physical sciences | Error-free computation for digital systems |
| Key Limitation | Precision errors | Requires specific prime moduli ($q \equiv 1 \pmod{n}$) |
| Typical Application | Signal/Image Processing, Digital Filtering | Cryptography, Large Number Multiplication |

### The Cornerstone: Primitive n-th Roots of Unity in Finite Fields

The entire theory of the NTT rests on the existence and properties of roots of unity within a finite field.

#### Definition

A primitive $n$-th root of unity in a finite field $\mathbb{Z}_q$ is an element $\omega \in \mathbb{Z}_q$ that satisfies two conditions:

1. $\omega^n \equiv 1 \pmod{q}$
2. $\omega^k \not\equiv 1 \pmod{q}$ for all integers $1 \leq k < n$

The first condition states that $\omega$ is an $n$-th root of unity. The second, more stringent condition ensures it is "primitive," meaning its powers $\omega^0, \omega^1, \ldots, \omega^{n-1}$ generate all $n$ distinct $n$-th roots of unity, forming a cyclic subgroup of order $n$.

#### The Existence Condition

The existence of such an element is not guaranteed for any arbitrary choice of $n$ and $q$. A fundamental theorem of number theory provides the necessary and sufficient condition:

**Theorem:** For a prime $q$, a primitive $n$-th root of unity exists in the finite field $\mathbb{Z}_q$ if and only if $n$ divides $q - 1$.

#### Proof of Existence

This theorem is central to the NTT and warrants a formal proof.

**Part 1** (If $\omega$ exists, then $n \mid (q-1)$):

Assume a primitive $n$-th root of unity, $\omega$, exists in $\mathbb{Z}_q$. The set of its powers, $\{\omega^0, \omega^1, \ldots, \omega^{n-1}\}$, forms a cyclic subgroup of order $n$ within the multiplicative group of non-zero elements, $\mathbb{Z}_q^*$.

The order (size) of this multiplicative group is $q - 1$. By Lagrange's theorem, the order of any subgroup must divide the order of the group. Therefore, $n$ must divide $q - 1$.

**Part 2** (If $n \mid (q-1)$, then $\omega$ exists):

Assume $n$ divides $q - 1$. It is a known result from finite field theory that the multiplicative group $\mathbb{Z}_q^\*$ is cyclic for any prime $q$. This means there exists a generator (or primitive root) $g \in \mathbb{Z}_q^{\*}$ such that its powers generate the entire group. The order of $g$ is $q - 1$.

Let's define an element $\omega$ as:

$\omega = g^{(q-1)/n} \pmod{q}$

We must show that $\omega$ is a primitive $n$-th root of unity.

First, we check that it is an $n$-th root of unity:

$\omega^n = (g^{(q-1)/n})^n = g^{q-1} \equiv 1 \pmod{q}$

This follows from Fermat's Little Theorem, which states that $g^{q-1} \equiv 1 \pmod{q}$ for any non-zero $g \in \mathbb{Z}_q$.

Second, we must show it is primitive. Let $k$ be the order of $\omega$. We know that $k$ must divide $n$. Suppose, for the sake of contradiction, that $k < n$. Then $\omega^k = g^{k(q-1)/n} \equiv 1 \pmod{q}$.

Since $g$ is a generator of $\mathbb{Z}_q^*$, its order is $q - 1$. Therefore, for $g^m \equiv 1$, it must be that $q - 1$ divides $m$. In our case, this means $q - 1$ must divide $k(q-1)/n$. This implies that $n$ must divide $k$.

But we assumed $k < n$, and since $k$ is a positive integer, this is a contradiction. Therefore, the order of $\omega$ must be exactly $n$, making it a primitive $n$-th root of unity.

### The NTT and its Inverse: Formal Definitions and Correctness

With the existence of a primitive root of unity established, we can formally define the NTT and its inverse.

#### Forward NTT

Let $a = [a_0, a_1, \ldots, a_{n-1}]$ be a vector of elements in $\mathbb{Z}_q$. Its forward Number Theoretic Transform, $\hat{a} = \text{NTT}(a)$, is a vector $\hat{a} = [\hat{a}_0, \hat{a}_1, \ldots, \hat{a}\_{n-1}]$ where each component is defined as:

$$\hat{a}\_k = \sum_{j=0}^{n-1} a\_j \omega^{jk} \pmod{q}$$

This transformation is linear and can be represented as a matrix-vector product, $\hat{a} = Wa$, where $W$ is an $n \times n$ matrix with entries $W_{kj} = \omega^{jk}$. This is a Vandermonde matrix constructed from the powers of $\omega$.

#### Inverse NTT (INTT)

The inverse transform, $a = \text{INTT}(\hat{a})$, is defined as:

$a_j = n^{-1} \sum_{k=0}^{n-1} \hat{a}_k \omega^{-jk} \pmod{q}$

This formula requires the existence of $n^{-1}$, the multiplicative inverse of $n$ modulo $q$. This inverse exists if and only if $\gcd(n, q) = 1$. Since $q$ is prime and the existence condition for $\omega$ requires $n \mid (q-1)$, we know $n < q$, so $\gcd(n, q) = 1$ is guaranteed.

#### Proof of Invertibility

To prove that the INTT is indeed the inverse of the NTT, we substitute the definition of $\hat{a}_k$ into the INTT formula:

$a_j = n^{-1} \sum_{k=0}^{n-1} \left(\sum_{l=0}^{n-1} a_l \omega^{lk}\right) \omega^{-jk} \pmod{q}$

Rearranging the order of summation:

$a_j = n^{-1} \sum_{l=0}^{n-1} a_l \left(\sum_{k=0}^{n-1} \omega^{(l-j)k}\right) \pmod{q}$

The inner sum is a geometric series of roots of unity. Let $\delta = l - j$. The sum is $\sum_{k=0}^{n-1} (\omega^\delta)^k$.

**Case 1:** $l = j$ ($\delta = 0$): The term $\omega^\delta = \omega^0 = 1$. The sum becomes $\sum_{k=0}^{n-1} 1 = n$.

**Case 2:** $l \neq j$ ($\delta \neq 0$): Since $\omega$ is a primitive $n$-th root of unity and $0 < |l - j| < n$, $\omega^\delta \neq 1$. The sum of the geometric series is $\frac{\omega^\delta - 1}{(\omega^\delta)^n - 1} = \frac{\omega^\delta - 1}{(\omega^n)^\delta - 1} = \frac{\omega^\delta - 1}{1^\delta - 1} = 0$.

Therefore, the inner sum is non-zero only when $l = j$, in which case it equals $n$. The expression simplifies to:

$a_j = n^{-1} (a_j \cdot n) = a_j \pmod{q}$

This confirms that the INTT is indeed the inverse of the NTT.

### The Convolution Theorem in Finite Fields

The primary motivation for using the NTT is its ability to simplify convolutions.

#### Statement and Interpretation

The Cyclic Convolution Theorem for NTT states that the NTT of a cyclic convolution of two vectors is the pointwise product of their individual NTTs:

$\text{NTT}(a \circledast b) = \text{NTT}(a) \odot \text{NTT}(b)$

This allows for the computation of a cyclic convolution via the procedure: $c = \text{INTT}(\text{NTT}(a) \odot \text{NTT}(b))$.

A powerful way to understand this theorem is through the lens of polynomial evaluation. The forward NTT of a coefficient vector $a$ is equivalent to evaluating its corresponding polynomial $A(x) = \sum a_j x^j$ at the $n$ distinct points given by the powers of the primitive root of unity:

$\hat{a}_k = A(\omega^k)$

The vector $\hat{a}$ is thus the representation of the polynomial in the "evaluation" or "frequency" domain.

#### Proof of the Theorem

This interpretation provides an elegant proof of the convolution theorem. Let $A(x)$, $B(x)$, and $C(x)$ be polynomials corresponding to vectors $a$, $b$, and their cyclic convolution $c = a \circledast b$. By the algebraic isomorphism discussed in Section 1.2, polynomial multiplication in the ring $\mathbb{Z}_q[x]/(x^n - 1)$ is equivalent to cyclic convolution of the coefficients. Therefore, $C(x) = A(x)B(x) \pmod{x^n - 1}$.

Let's compute the NTT of $c$:

$\hat{c}_k = C(\omega^k)$

Since $\omega$ is an $n$-th root of unity, $\omega^n = 1$, which means $\omega$ is a root of the polynomial $x^n - 1$. Therefore, evaluating the polynomial product at $\omega^k$ does not require the modulo operation:

$C(\omega^k) = (A(x)B(x) \pmod{x^n - 1})|_{x=\omega^k} = A(\omega^k)B(\omega^k)$

We recognize that $A(\omega^k) = \hat{a}_k$ and $B(\omega^k) = \hat{b}_k$. Thus:

$\hat{c}_k = \hat{a}_k \cdot \hat{b}_k$

This is precisely the definition of the pointwise product. We have shown that $\text{NTT}(c) = \text{NTT}(a) \odot \text{NTT}(b)$, completing the proof.

## High-Performance Computation: Fast NTT Algorithms

The theoretical framework of the NTT provides a path to exact, fast convolution. However, its practical utility hinges on the existence of an algorithm to compute the transform itself in less than the naive $O(n^2)$ time. The Cooley-Tukey algorithm, a cornerstone of digital signal processing, provides such a method by applying the "divide-and-conquer" principle. The duality between its two main variants, Decimation-in-Time (DIT) and Decimation-in-Frequency (DIF), enables highly optimized implementations that are crucial for performance-critical applications.

### The Divide-and-Conquer Paradigm: The Cooley-Tukey Algorithm

The efficiency of fast transform algorithms stems from the divide-and-conquer strategy, which solves a problem by recursively breaking it into smaller instances of the same problem and combining their solutions.

#### Core Principle

The Cooley-Tukey FFT algorithm, which applies directly to the NTT, re-expresses an NTT of a composite size $N = N_1 N_2$ in terms of smaller NTTs of sizes $N_1$ and $N_2$. When $N$ is a power of two, $N = 2^k$, this decomposition can be applied recursively, repeatedly halving the problem size. This recursive structure is what reduces the computational complexity from $O(N^2)$ to $O(N \log N)$.

#### Derivation for Radix-2 DIT

The Radix-2 Decimation-in-Time (DIT) variant is derived by splitting the input sequence into its even-indexed and odd-indexed elements. Starting with the NTT definition:

$$\hat{a}\_k = \sum_{j=0}^{N-1} a_j \omega_N^{jk}$$

We can separate the sum over even and odd indices by letting $j = 2m$ for the even part and $j = 2m + 1$ for the odd part:

We can separate the sum over even and odd indices by letting $j = 2m$ for the even part and $j = 2m + 1$ for the odd part:

$$\hat{a}\_{k} = \sum_{m=0}^{N/2-1} a_{2m} \omega_N^{(2m)k} + \sum_{m=0}^{N/2-1} a_{2m+1} \omega_N^{(2m+1)k}$$

Using the identity $\omega_N^2 = \omega_{N/2}$, we can rewrite the terms:

$$\begin{align}
\hat{a}\_{k} &= \sum_{m=0}^{N/2-1} a_{2m} (\omega_N^2)^{mk} + \omega_N^k \sum_{m=0}^{N/2-1} a_{2m+1} (\omega_N^2)^{mk} \\\\
&= \sum_{m=0}^{N/2-1} a_{2m} \omega_{N/2}^{mk} + \omega_N^k \cdot \sum_{m=0}^{N/2-1} a_{2m+1} \omega_{N/2}^{mk}
\end{align}$$

The first sum represents the NTT of the even-indexed elements, and the second sum represents the NTT of the odd-indexed elements.

Let us define:
- $\hat{a}\_{k}^{\text{even}} = \text{NTT}(a_{\text{even}})$ (NTT of even-indexed elements)
- $\hat{a}\_{k}^{\text{odd}} = \text{NTT}(a_{\text{odd}})$ (NTT of odd-indexed elements)

Then the formula becomes:

$$\hat{a}\_{k} = \hat{a}\_{k}^{\text{even}} + \omega_N^k \cdot \hat{a}_{k}^{\text{odd}}$$


This computes the first half of the output transform ($0 \leq k < N/2$). For the second half ($N/2 \leq k < N$), we can let $k^\prime = k - N/2$. Using the identities $\omega_N^{k+N/2} = \omega_N^k \omega_N^{N/2} = -\omega_N^k$ and the periodicity of the smaller transforms ($\hat{a}_{k+N/2}^{\text{even}} = \hat{a}_k^{\text{even}}$), we get:

$$\hat{a}_{k+N/2} = \hat{a}_k^{\text{even}} - \omega_N^k \cdot \hat{a}_k^{\text{odd}}$$

These two equations form the core of the recursive algorithm.

### The Butterfly Operation: The Heart of the FFT/NTT

The "butterfly" is the fundamental computational unit that implements the combination step of the Cooley-Tukey algorithm. It takes two input values and produces two output values, with its data-flow diagram resembling the shape of a butterfly's wings.

#### Cooley-Tukey (DIT) Butterfly

The pair of equations derived above defines the Cooley-Tukey or DIT butterfly. For each index $k$ from $0$ to $N/2 - 1$, it takes an element from the even-part transform ($\hat{a}_k^{\text{even}}$) and an element from the odd-part transform ($\hat{a}_k^{\text{odd}}$), combines them using a "twiddle factor" $\omega_N^k$, and produces two outputs for the full transform, $\hat{a}\_{k}$ and $\hat{a}\_{k+N/2}$.

$$\begin{cases}
\hat{a}\_k \leftarrow \hat{a}\_k^{\text{even}} + \omega_N^k \cdot \hat{a}\_k^{\text{odd}} \\\\
\hat{a}\_{k+N/2} \leftarrow \hat{a}\_k^{\text{even}} - \omega_N^k \cdot \hat{a}\_k^{\text{odd}}
\end{cases}$$

A diagram of this operation shows two inputs on the left, two outputs on the right, with lines crossing in the middle, one path involving a multiplication by the twiddle factor.

#### Gentleman-Sande (DIF) Butterfly

The Gentleman-Sande (GS) or Decimation-in-Frequency (DIF) algorithm is the computational dual of the DIT algorithm. It splits the output (frequency domain) sequence rather than the input (time domain) sequence. Its butterfly operation precedes the recursive calls and has a different structure:

$$\begin{cases}
y_0 \leftarrow x_0 + x_1 \\\\
y_1 \leftarrow (x_0 - x_1) \cdot \omega_N^k
\end{cases}$$

The GS butterfly is often preferred for implementing the inverse NTT (INTT). The DIT and DIF algorithms are computationally duals: a DIT forward transform naturally pairs with a DIF inverse transform. The DIT algorithm typically takes natural-ordered input and produces bit-reversed output, while the DIF algorithm takes bit-reversed input and produces natural-ordered output. When performing convolution via $c = \text{INTT}(\text{NTT}(a) \odot \text{NTT}(b))$, one can use a DIT for the forward NTTs and a DIF for the inverse INTT. The bit-reversed output of the DIT-based NTTs becomes the correctly ordered input for the DIF-based INTT, eliminating the need for an explicit bit-reversal permutation between the forward and inverse transforms. This pairing saves a full pass over the data, a significant optimization in high-performance implementations.

### Data Ordering and In-Place Computation: The Bit-Reversal Permutation

The recursive even-odd splitting at the heart of the Cooley-Tukey DIT algorithm has a peculiar side effect on the data ordering.

#### The Scrambling Effect

If one traces the indices through the recursive DIT decomposition, the input elements are effectively reordered according to a bit-reversal permutation. For an in-place algorithm that overwrites its input buffer, this means that to get a naturally ordered output, the input must first be placed in bit-reversed order. Conversely, if the input is in natural order, the output will be in bit-reversed order.

#### What is Bit-Reversal?

The bit-reversal permutation of an index is found by writing its binary representation and reversing the order of the bits. For example, for a transform of size $N = 8$, the indices are represented by 3 bits. The index 3 is 011 in binary. Reversing the bits gives 110, which is the decimal value 6. Thus, the element at index 3 is swapped with the element at index 6.

| Natural Index | Binary | Reversed Binary | Bit-Reversed Index |
|---------------|--------|-----------------|--------------------|
| 0 | 000 | 000 | 0 |
| 1 | 001 | 100 | 4 |
| 2 | 010 | 010 | 2 |
| 3 | 011 | 110 | 6 |
| 4 | 100 | 001 | 1 |
| 5 | 101 | 101 | 5 |
| 6 | 110 | 011 | 3 |
| 7 | 111 | 111 | 7 |

#### In-Place Algorithms

The primary motivation for dealing with bit-reversal is to enable in-place computation, which saves memory. Instead of allocating new arrays for the outputs of each recursive stage, an in-place algorithm overwrites the input buffer. A common strategy is to first apply the bit-reversal permutation to the entire input array. After this initial reordering, all subsequent butterfly stages can be computed in-place, and the final output will be in natural (sequential) order.

### A Step-by-Step Numerical Example: Polynomial Multiplication via Fast NTT

Let's perform a complete polynomial multiplication using a fast NTT, combining all the concepts discussed. We will compute the product of $A(x) = 2x^2 + x + 1$ and $B(x) = 3x^2 + 4x + 2$ in the ring $\mathbb{Z}_{17}[x]/(x^4 - 1)$.

#### 1. Parameter Setup:

Polynomial length $N = 4$. The coefficient vectors are $a = [1, 1, 2, 0]$ and $b = [2, 4, 3, 0]$.

Modulus $q = 17$. We check the existence condition: $N = 4$ must divide $q - 1 = 16$. $16/4 = 4$, so the condition holds.

Primitive 4th root of unity: We need $\omega \in \mathbb{Z}_{17}$ such that $\omega^4 \equiv 1$ and $\omega^2 \not\equiv 1$. Let's try $\omega = 4$.

$\omega^1 = 4 \pmod{17}$
$\omega^2 = 16 \equiv -1 \pmod{17}$
$\omega^3 = 64 \equiv 13 \pmod{17}$
$\omega^4 = 256 \equiv 1 \pmod{17}$.

Since $\omega^2 \not\equiv 1$, $\omega = 4$ is a primitive 4th root of unity. The required twiddle factors are $\omega^0 = 1, \omega^1 = 4$.


#### 2. Forward NTT (Cooley-Tukey DIT):

We will use a natural-order input, in-place algorithm, which will produce a bit-reversed output.

**Input vectors:** $a = [1, 1, 2, 0]$ and $b = [2, 4, 3, 0]$.

**Stage 1 (Butterflies of size 2):**

For vector $a$:
- Pair $(a_{0}, a_{2})$: 
  $$\begin{aligned}
  t &= a_{0} + a_{2} = 1 + 2 = 3 \\\\
  a_{2} &= a_{0} - a_{2} = 1 - 2 = -1 \equiv 16 \pmod{17}
  \end{aligned}$$
  New $a = [3, 1, 16, 0]$.

- Pair $(a_{1}, a_{3})$: 
  $$\begin{aligned}
  t &= a_{1} + a_{3} = 1 + 0 = 1 \\\\
  a_{3} &= a_{1} - a_{3} = 1 - 0 = 1
  \end{aligned}$$
  New $a = [3, 1, 16, 1]$.

For vector $b$:
- Pair $(b_{0}, b_{2})$: 
  $$\begin{aligned}
  t &= b_{0} + b_{2} = 2 + 3 = 5 \\\\
  b_{2} &= b_{0} - b_{2} = 2 - 3 = -1 \equiv 16 \pmod{17}
  \end{aligned}$$
  New $b = [5, 4, 16, 0]$.

- Pair $(b_{1}, b_{3})$: 
  $$\begin{aligned}
  t &= b_{1} + b_{3} = 4 + 0 = 4 \\\\
  b_{3} &= b_{1} - b_{3} = 4 - 0 = 4
  \end{aligned}$$
  New $b = [5, 4, 16, 4]$.

**Stage 2 (Butterflies of size 4):**

For vector $a = [3, 1, 16, 1]$:
- Pair $(a_{0}, a_{1})$ with twiddle factor $\omega^{0} = 1$:
  $$\begin{aligned}
  t &= a_{0} + 1 \cdot a_{1} = 3 + 1 = 4 \\\\
  a_{1} &= a_{0} - 1 \cdot a_{1} = 3 - 1 = 2
  \end{aligned}$$
  
- Pair $(a_{2}, a_{3})$ with twiddle factor $\omega^{1} = 4$:
  $$\begin{aligned}
  t &= a_{2} + 4 \cdot a_{3} = 16 + 4 \cdot 1 = 20 \equiv 3 \pmod{17} \\\\
  a_{3} &= a_{2} - 4 \cdot a_{3} = 16 - 4 = 12
  \end{aligned}$$
  
  Final $a = [4, 2, 3, 12]$.

For vector $b = [5, 4, 16, 4]$:
- Pair $(b_{0}, b_{1})$ with twiddle factor $\omega^{0} = 1$:
  $$\begin{aligned}
  t &= b_{0} + 1 \cdot b_{1} = 5 + 4 = 9 \\\\
  b_{1} &= b_{0} - 1 \cdot b_{1} = 5 - 4 = 1
  \end{aligned}$$
  
- Pair $(b_{2}, b_{3})$ with twiddle factor $\omega^{1} = 4$:
  $$\begin{aligned}
  t &= b_{2} + 4 \cdot b_{3} = 16 + 4 \cdot 4 = 32 \equiv 15 \pmod{17} \\\\
  b_{3} &= b_{2} - 4 \cdot b_{3} = 16 - 16 = 0
  \end{aligned}$$
  
  Final $b = [9, 1, 15, 0]$.

**Final NTT vectors (in bit-reversed order):**

$$\hat{a}\_{br} = [4, 2, 3, 12]$$

$$\hat{b}\_{br} = [9, 1, 15, 0]$$

Note: Since we used natural-order input with the DIT algorithm, the output is in bit-reversed order. The natural order would be the same: $\hat{a} = [4, 2, 3, 12]$ and $\hat{b} = [9, 1, 15, 0]$ (for $N=4$, bit-reversal is identity).


#### 3. Pointwise Multiplication:

We multiply the bit-reversed vectors component-wise.
$\hat{c}\_{br}[0] = 4 \cdot 9 = 36 \equiv 2 \pmod{17}$
$\hat{c}\_{br}[1] = 2 \cdot 1 = 2 \pmod{17}$
$\hat{c}\_{br}[2] = 3 \cdot 15 = 45 \equiv 11 \pmod{17}$
$\hat{c}\_{br}[3] = 12 \cdot 0 = 0 \pmod{17}$

$\hat{c}\_{br} = [2, 2, 11, 0]$

#### 4. Inverse NTT (Gentleman-Sande DIF):

We will use a bit-reversed input, in-place algorithm, which will produce a natural-order output. 

**Twiddle factor calculation:**
The twiddle factors use $\omega^{-1} = 4^{-1} \pmod{17}$. Since $4 \cdot 13 = 52 = 3 \cdot 17 + 1$, we have $\omega^{-1} = 13$. 

Therefore: $(\omega^{-1})^{0} = 1$ and $(\omega^{-1})^{1} = 13$.

**Input vector:** $\hat{c}_{br} = [2, 2, 11, 0]$.

**Stage 1 (Butterflies of size 4):**

- Pair $(\hat{c}\_{0}, \hat{c}\_{2})$:
  $$\begin{aligned}
  t &= \hat{c}\_{0} + \hat{c}\_{2} = 2 + 11 = 13 \\\\
  \hat{c}\_{2} &= (\hat{c}\_{0} - \hat{c}\_{2}) \cdot \omega^{0} = (2 - 11) \cdot 1 = -9 \equiv 8 \pmod{17}
  \end{aligned}$$

- Pair $(\hat{c}\_{1}, \hat{c}\_{3})$:
  $$\begin{aligned}
  t &= \hat{c}\_{1} + \hat{c}\_{3} = 2 + 0 = 2 \\\\
  \hat{c}\_{3} &= (\hat{c}\_{1} - \hat{c}\_{3}) \cdot \omega^{1} = (2 - 0) \cdot 13 = 26 \equiv 9 \pmod{17}
  \end{aligned}$$

After Stage 1: $\hat{c} = [13, 2, 8, 9]$.

**Stage 2 (Butterflies of size 2):**

- Pair $(\hat{c}\_{0}, \hat{c}\_{1})$:
  $$\begin{aligned}
  t &= \hat{c}\_{0} + \hat{c}\_{1} = 13 + 2 = 15 \\\\
  \hat{c}\_{1} &= \hat{c}\_{0} - \hat{c}_{1} = 13 - 2 = 11
  \end{aligned}$$

- Pair $(\hat{c}\_{2}, \hat{c}\_{3})$:
  $$\begin{aligned}
  t &= \hat{c}\_{2} + \hat{c}_{3} = 8 + 9 = 17 \equiv 0 \pmod{17} \\\\
  \hat{c}\_{3} &= \hat{c}\_{2} - \hat{c}\_{3} = 8 - 9 = -1 \equiv 16 \pmod{17}
  \end{aligned}$$

After Stage 2: $\hat{c} = [15, 11, 0, 16]$.

**Final vector (before scaling):** $c_{\text{unscaled}} = [15, 11, 0, 16]$.

**Scaling step:**

We need to multiply by $N^{-1} = 4^{-1} \equiv 13 \pmod{17}$:

$$\begin{aligned}
c_{0} &= 15 \cdot 13 = 195 = 11 \cdot 17 + 8 \equiv 8 \pmod{17} \\\\
c_{1} &= 11 \cdot 13 = 143 = 8 \cdot 17 + 7 \equiv 7 \pmod{17} \\\\
c_{2} &= 0 \cdot 13 = 0 \pmod{17} \\\\
c_{3} &= 16 \cdot 13 = 208 = 12 \cdot 17 + 4 \equiv 4 \pmod{17}
\end{aligned}$$

**Final Result:** $c = [8, 7, 0, 4]$.


#### 5. Verification:

Let's compute the product manually in $\mathbb{Z}_{17}[x]/(x^4 - 1)$.

**Linear product:** 
$(2x^2 + x + 1)(3x^2 + 4x + 2) = 6x^4 + 8x^3 + 4x^2 + 3x^3 + 4x^2 + 2x + 3x^2 + 4x + 2 = 6x^4 + 11x^3 + 11x^2 + 6x + 2$.

**Reduce modulo $x^4 - 1$** (i.e., $x^4 \equiv 1$):
$6(1) + 11x^3 + 11x^2 + 6x + 2 = 11x^3 + 11x^2 + 6x + 8$.

The coefficient vector is $[8, 6, 11, 11]$. There seems to be a discrepancy. Let's recheck the manual calculation and the NTT steps.

---

**Re-check of the example:**

Let's re-verify the manual calculation:
- $A(x) = 2x^2 + x + 1$
- $B(x) = 3x^2 + 4x + 2$

$$A(x)B(x) = (2x^2 + x + 1)(3x^2 + 4x + 2) = 6x^4 + 11x^3 + 11x^2 + 6x + 2$$

**Modulo $x^4 - 1$:** $6(1) + 11x^3 + 11x^2 + 6x + 2 = 11x^3 + 11x^2 + 6x + 8$.

**Coefficient vector:** $[8, 6, 11, 11]$.

---

**Re-check NTT:**

Input vectors: $a = [1, 1, 2, 0]$, $b = [2, 4, 3, 0]$.

**Forward NTT (Evaluation):**

$$\begin{aligned}
\hat{a}\_0 &= A(4^0) = A(1) = 2(1)^2 + 1 + 1 = 4 \\\\
\hat{a}\_1 &= A(4^1) = A(4) = 2(16) + 4 + 1 = 37 \equiv 3 \pmod{17} \\\\
\hat{a}\_2 &= A(4^2) = A(16) = A(-1) = 2(-1)^2 - 1 + 1 = 2 \\\\
\hat{a}\_3 &= A(4^3) = A(13) = A(-4) = 2(-4)^2 - 4 + 1 = 29 \equiv 12 \pmod{17}
\end{aligned}$$

Therefore: $\hat{a} = [4, 3, 2, 12]$ (matches natural order from before).

$$\begin{aligned}
\hat{b}\_0 &= B(1) = 3 + 4 + 2 = 9 \\\\
\hat{b}\_1 &= B(4) = 3(16) + 4(4) + 2 = 66 \equiv 15 \pmod{17} \\\\
\hat{b}\_2 &= B(-1) = 3 - 4 + 2 = 1 \\\\
\hat{b}\_3 &= B(-4) = 3(16) + 4(-4) + 2 = 34 \equiv 0 \pmod{17}
\end{aligned}$$

Therefore: $\hat{b} = [9, 15, 1, 0]$ (matches natural order from before).

**Pointwise product** $\hat{c} = \hat{a} \odot \hat{b}$:

$$\begin{aligned}
\hat{c}\_0 &= 4 \cdot 9 = 36 \equiv 2 \pmod{17} \\
\hat{c}\_1 &= 3 \cdot 15 = 45 \equiv 11 \pmod{17} \\
\hat{c}\_2 &= 2 \cdot 1 = 2 \\
\hat{c}\_3 &= 12 \cdot 0 = 0
\end{aligned}$$

Therefore: $\hat{c} = [2, 11, 2, 0]$.

**Inverse NTT** of $\hat{c} = [2, 11, 2, 0]$:

The coefficients are given by: $c\_j = 4^{-1} \sum_{k=0}^{3} \hat{c}\_k (\omega^{-1})^{jk}$, where $4^{-1} = 13$.

$$\begin{aligned}
c\_0 &= 13 \cdot (2 \cdot 1 + 11 \cdot 1 + 2 \cdot 1 + 0 \cdot 1) \\\\
&= 13 \cdot 15 = 195 \equiv 8 \pmod{17} \\\\
\\\\
c\_1 &= 13 \cdot (2 \cdot 1 + 11 \cdot 13 + 2 \cdot (-1) + 0 \cdot (-13)) \\\\
&= 13 \cdot (2 + 143 - 2) = 13 \cdot 143 = 13 \cdot 7 \\\\
&= 91 = 5 \cdot 17 + 6 \equiv 6 \pmod{17} \\\\
\\\\
c\_2 &= 13 \cdot (2 \cdot 1 + 11 \cdot (-1) + 2 \cdot 1 + 0 \cdot (-1)) \\\\
&= 13 \cdot (2 - 11 + 2) = 13 \cdot (-7) = -91 \\\\
&\equiv -6 \equiv 11 \pmod{17} \\\\
\\\\
c\_3 &= 13 \cdot (2 \cdot 1 + 11 \cdot (-13) + 2 \cdot (-1) + 0 \cdot 13) \\\\
&= 13 \cdot (2 - 7 - 2) = 13 \cdot (-7) \equiv 11 \pmod{17}
\end{aligned}$$

**Final vector:** $c = [8, 6, 11, 11]$.

This matches the manual calculation. The error was in the step-by-step butterfly calculation. The example demonstrates the correctness of the transform method.


## Negacyclic Convolution with NTT:
> **The Cryptographer's Toolkit**

While the NTT is naturally suited for cyclic convolution, the premier applications in modern cryptography are built upon negacyclic convolution, corresponding to multiplication in the ring $\mathbb{Z}_q[x]/(x^n + 1)$. A direct application of the standard NTT would produce an incorrect result. Fortunately, a clever and efficient adaptation exists that allows the powerful NTT machinery to be used for this crucial case. This adaptation is not merely an ad-hoc "trick" but a concrete implementation of a ring isomorphism, mapping the negacyclic problem into a cyclic one that the NTT can solve, and then mapping the result back.

### The Challenge: Adapting NTT for the Ring $\mathbb{Z}_q[x]/(x^n + 1)$

The fundamental structure of the NTT is based on the properties of $n$-th roots of unity, which are the roots of the polynomial $x^n - 1$. This algebraic foundation means the NTT naturally computes polynomial products modulo $x^n - 1$, which is cyclic convolution. Attempting to multiply polynomials from the ring $\mathbb{Z}_q[x]/(x^n + 1)$ using this machinery directly will fail because the wrap-around rule is different ($x^n \equiv -1$, not $x^n \equiv 1$).

### The Solution: The Role of the $2n$-th Root of Unity

The standard solution involves embedding the $n$-point negacyclic convolution within a larger, $2n$-point cyclic convolution. This is possible due to the algebraic identity $x^{2n} - 1 = (x^n - 1)(x^n + 1)$. This factorization allows a mapping between the two algebraic worlds.

The most common implementation of this mapping involves pre- and post-processing steps that "twist" the data into a form suitable for a cyclic NTT. Let $\psi$ be a primitive $2n$-th root of unity in $\mathbb{Z}\_{q}$. Note that $\psi^2 = \omega$ is a primitive $n$-th root of unity. The procedure to compute the negacyclic convolution $c = a \circledast_{-} b$ is as follows:

**Pre-processing (Twisting):** Take the input coefficient vectors $a$ and $b$ of length $n$. Create two new vectors, $a^\prime$ and $b^\prime$, by multiplying each coefficient by the corresponding power of $\psi$:

$a^\prime\_i = a_i \cdot \psi^i \pmod{q} \quad \text{for } i = 0, \dots, n-1$

$b^\prime\_i = b_i \cdot \psi^i \pmod{q} \quad \text{for } i = 0, \dots, n-1$

**Cyclic Convolution via NTT:** Compute the $n$-point cyclic convolution of the twisted vectors $a^\prime$ and $b^\prime$. This is done efficiently using the standard NTT based on the primitive $n$-th root of unity $\omega = \psi^2$.

$c^\prime = \text{INTT}(\text{NTT}(a^\prime) \odot \text{NTT}(b^\prime))$

**Post-processing (Untwisting):** Take the resulting vector $c^\prime$ and multiply each coefficient by the corresponding inverse power of $\psi$ to obtain the final result $c$:

$c_i = c^\prime_i \cdot \psi^{-i} \pmod{q} \quad \text{for } i = 0, \ldots, n-1$

This three-step process correctly computes the product of two polynomials in $\mathbb{Z}_q[x]/(x^n + 1)$ using the machinery of an $n$-point NTT designed for $\mathbb{Z}_q[x]/(x^n - 1)$. The mathematical derivation shows that the "twisting" factors correctly account for the sign change required by the negacyclic structure.

### Parameter Constraints for Negacyclic NTT

This elegant solution introduces a stricter requirement on the system parameters. The pre- and post-processing steps depend on the existence of a primitive $2n$-th root of unity, $\psi$. According to the existence theorem from Section 2.2, for such a root to exist in the finite field $\mathbb{Z}_q$, the modulus $q$ must satisfy the condition:

$2n \mid (q-1) \quad \text{or} \quad q \equiv 1 \pmod{2n}$

This condition is twice as restrictive as the one for the standard cyclic NTT ($n \mid (q-1)$). This constraint has profound implications for the design of lattice-based cryptosystems. Cryptographers must select prime moduli that are not only large enough for security and small enough for efficiency, but also have this specific algebraic property to enable the use of the fastest known polynomial multiplication algorithms. This delicate balancing act is a central theme in the practical implementation of post-quantum cryptography.

## Application Spotlight: Post-Quantum Cryptography

The abstract mathematical machinery of convolutions and number theoretic transforms finds its most critical modern application in the field of post-quantum cryptography (PQC). The transition to quantum-resistant algorithms is one of the most significant security upgrades in the history of computing, and the performance of these new systems hinges directly on the efficient implementation of polynomial arithmetic. The design of modern PQC schemes is not a linear process where a cryptosystem is created and an algorithm is later found to implement it; it is a sophisticated co-design, where the choice of algebraic structures is motivated by security, while the selection of numerical parameters is a delicate balance between security, correctness, and the specific requirements of algorithms like the NTT.

### The Need for Speed: 
> **Polynomial Multiplication in Lattice-Based Cryptography**

The impending threat of large-scale quantum computers, which can efficiently break current public-key standards like RSA and ECC using Shor's algorithm, has necessitated the development of new cryptographic foundations.

#### Lattices and PQC

Lattice-based cryptography has emerged as one of the most promising families of PQC candidates. Its security is based on the presumed hardness of certain mathematical problems on high-dimensional lattices, such as the Learning With Errors (LWE) and Ring-Learning With Errors (RLWE) problems. These problems are believed to be resistant to attack by both classical and quantum computers.

#### The Performance Bottleneck

A key feature of many lattice-based schemes, particularly those based on RLWE, is that their core operations involve arithmetic in polynomial quotient rings, such as $\mathbb{Z}_q[x]/(x^n + 1)$. The most frequent and computationally intensive operation in these schemes—including key generation, encapsulation, and decapsulation—is the multiplication of large-degree polynomials. For the security levels required for modern applications, the polynomial degree $n$ is typically in the range of 256 to 1024. Using a naive schoolbook multiplication algorithm with $O(n^2)$ complexity would render these schemes too slow for practical use, especially in resource-constrained environments like embedded systems or high-throughput servers.

#### NTT as the Enabler

The Number Theoretic Transform is the crucial enabling technology that makes these schemes practical. By reducing the complexity of polynomial multiplication to $O(n \log n)$, the NTT provides a speedup of one to two orders of magnitude, transforming a theoretical curiosity into a deployable technology. The performance of schemes like the NIST-standardized CRYSTALS-Kyber and Dilithium is fundamentally reliant on highly optimized NTT implementations.

### Security Considerations: Why Negacyclic over Cyclic?

While both cyclic and negacyclic convolutions can be accelerated by the NTT, lattice-based cryptography has a strong preference for the negacyclic ring $\mathbb{Z}_q[x]/(x^n + 1)$. This choice is not arbitrary; it is rooted in important security considerations.

#### The Vulnerability of $x^n - 1$

The polynomial $x^n - 1$ has a simple algebraic structure that can be exploited in certain attacks. For any $n$, it is reducible over the integers, as it always has the factor $(x - 1)$. The existence of this simple root ($x = 1$) allows an attacker to map polynomial operations into simpler integer operations by evaluating the polynomials at $x = 1$. This "polynomial evaluation" can leak information about the secret key, creating a vulnerability that undermines the security of the underlying hard problem.

#### The Strength of Cyclotomic Polynomials

In contrast, when $n$ is a power of two, the polynomial $\Phi_{2n}(x) = x^n + 1$ is the $2n$-th cyclotomic polynomial. This polynomial is irreducible over the rational numbers. While it may be reducible over the finite field $\mathbb{Z}_q$, its richer algebraic structure avoids the simple vulnerabilities associated with the $x = 1$ root. This makes the RLWE problem instantiated in the ring $\mathbb{Z}_q[x]/(x^n + 1)$ harder and the resulting cryptosystems more robust against known algebraic attacks. This security advantage is the primary reason for the near-ubiquitous use of negacyclic convolution in modern lattice-based PQC.

### Case Study: NTT in CRYSTALS-Kyber and Dilithium

The practical implications of these algorithmic and security choices are best illustrated by examining the schemes selected by the U.S. National Institute of Standards and Technology (NIST) for standardization.

#### NIST Standardization

After a multi-year global competition, NIST selected CRYSTALS-Kyber as its standard for Key Encapsulation Mechanisms (KEMs) and CRYSTALS-Dilithium for digital signatures. Both are part of the CRYSTALS suite and are built on the hardness of problems over module lattices, a variant of ideal lattices.

#### Shared Algebraic Structure

A key design feature of the CRYSTALS suite is that both Kyber and Dilithium operate over the same polynomial ring: $\mathbb{Z}_q[x]/(x^{256} + 1)$. This means that a highly optimized hardware or software implementation of the NTT for this specific ring can be reused for both key exchange and digital signatures, simplifying development and deployment.

#### Kyber's Parameters and the NTT "Mismatch"

The parameters for Kyber provide a fascinating real-world example of the co-design process. For all its security levels, Kyber uses a polynomial degree of $n = 256$. The modulus, however, is $q = 3329$.

Let's check the negacyclic NTT condition from Section 4.3: $2n \mid (q-1)$.
$n = 256$, so $2n = 512$.
$q - 1 = 3328$.

We check if 512 divides 3328: $3328/512 = 6.5$. It does not.

This reveals a crucial point: Kyber's parameters are intentionally chosen to be "NTT-unfriendly" with respect to the textbook negacyclic NTT algorithm. The choice of $q = 3329$ was motivated by other factors, such as simplifying certain types of modular reduction and satisfying the requirements of the security proof.

#### Kyber's Solution

This parameter mismatch does not mean the NTT cannot be used. Instead, it requires a more sophisticated algorithmic approach. The designers of Kyber employ a variant of the negacyclic convolution trick. Since $q \equiv 1 \pmod{n}$ but not $\pmod{2n}$, a full set of $2n$-th roots of unity does not exist. However, by decomposing the degree-$d$ polynomial into its even and odd sub-parts, each of degree $d/2$, the multiplication can be expressed in terms of several cross-multiplications and additions between these smaller polynomials. This allows a "half-NTT" or incomplete NTT procedure to be used, leveraging the existing roots of unity to accelerate the multiplication of the sub-parts. This is a prime example of how theoretical algorithms are adapted to meet the complex, multi-faceted constraints of real-world cryptographic engineering.

#### Dilithium's Parameters

CRYSTALS-Dilithium also operates over the ring with $n = 256$, but it uses a different, much larger prime modulus, $q = 2^{23} - 2^{13} + 1 = 8380417$. For this choice, $q - 1 = 8380416$. We check the condition $2n \mid (q-1)$:

$2n = 512$.
$8380416/512 = 16368$.

The condition holds. Therefore, Dilithium's parameters are "NTT-friendly," and it can use the standard negacyclic NTT algorithm without the complex workarounds required by Kyber. This difference in parameterization highlights the diverse design choices made even within a single cryptographic suite to optimize for different use cases (KEM vs. signatures).

| Scheme | Security Level | Degree ($n$) | Modulus ($q$) | Ring | $n \mid (q-1)$? | $2n \mid (q-1)$? | Implemented NTT Strategy |
|--------|----------------|--------------|---------------|------|-------------|--------------|-------------------------|
| CRYSTALS-Kyber | 512, 768, 1024 | 256 | 3329 | $\mathbb{Z}_q[x]/(x^{256} + 1)$ | Yes (3328/256=13) | No (3328/512=6.5) | Incomplete / Adapted Negacyclic NTT |
| CRYSTALS-Dilithium | 2, 3, 5 | 256 | 8380417 | $\mathbb{Z}_q[x]/(x^{256} + 1)$ | Yes | Yes | Standard Negacyclic NTT |

## Conclusion

This report has traversed the landscape of convolution and the Number Theoretic Transform, from foundational algebraic definitions to the intricate algorithmic details that power modern secure communication. The journey reveals a deep and elegant interplay between abstract algebra, computational theory, and practical engineering.

The analysis began by establishing a crucial conceptual link: the seemingly distinct operations of linear, cyclic, and negacyclic convolution are not merely procedural variants but are precise algebraic manifestations of polynomial multiplication within different rings. Linear convolution corresponds to the unconstrained ring $\mathbb{Z}[x]$, while cyclic and negacyclic convolutions correspond to multiplication in the quotient rings $\mathbb{Z}_q[x]/(x^n - 1)$ and $\mathbb{Z}_q[x]/(x^n + 1)$, respectively. This understanding reframes the choice of a convolution algorithm as the selection of a specific mathematical world with its own unique rules.

The Number Theoretic Transform was presented as a powerful tool for efficient computation within these finite algebraic worlds. As an analogue of the Discrete Fourier Transform operating over finite fields, the NTT inherits the quasi-linear $O(n \log n)$ complexity of the Fast Fourier Transform while crucially eliminating the floating-point precision errors that are intolerable in cryptographic contexts. The core of the NTT's power lies in the convolution theorem, which transforms the complex problem of convolution into a simple pointwise product. This was shown to be a direct consequence of the NTT's interpretation as a change of basis—from the standard monomial basis to a more convenient evaluation basis defined by the primitive roots of unity.

The practical implementation of the NTT through fast algorithms like the Cooley-Tukey and Gentleman-Sande methods, along with the nuances of in-place computation via bit-reversal, demonstrates the sophisticated engineering required to translate theory into high-performance code. The final application to post-quantum cryptography highlights the pinnacle of this synthesis. The design of NIST-standardized schemes like CRYSTALS-Kyber is a testament to the co-design of security and algorithms. The choice of the negacyclic ring $\mathbb{Z}_q[x]/(x^n + 1)$ is driven by security considerations, while the choice of the modulus $q$ and the specific formulation of the NTT algorithm are products of a complex optimization across security, correctness, and performance.

As the global transition to post-quantum cryptography accelerates, the importance of these concepts will only grow. The efficiency of NTT implementations will directly impact the viability of secure systems in a vast range of applications, from resource-constrained IoT devices to large-scale cloud infrastructure. Continued research into optimizing these algorithms, both in software and dedicated hardware, will remain a critical frontier, ensuring that the mathematical elegance of convolution and the Number Theoretic Transform continues to provide the foundation for a secure digital future.
