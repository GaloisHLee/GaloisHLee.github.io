# NIP



> Note:Noisy Polynomial Interpolation Problem

<!--more-->


# Polynomial Interpolation Problem and Lattice

---

## 1. Understanding the NPI Problem

The Noisy Polynomial Interpolation (NPI) problem is a computational challenge that was once believed to be hard, forming the security basis for several cryptographic protocols. This analysis explores its definition, its origins, and why it turned out to be much weaker than anticipated.[^1]

### 1.1 A Formal Definition

Let's start with a formal definition. Imagine we have a polynomial $P(X)$ of degree $k$ over a finite field $F$.

> **Quick Note:** A finite field $F$ is just a set of numbers with a finite number of elements, where you can perform addition, subtraction, multiplication, and division, and the result stays within the set. A common example is the set of integers modulo a prime number.

The input to the NPI problem consists of:

* $n$ distinct points from the field $F$, which we'll call $x_1, x_2, \dots, x_n$.
* $n$ corresponding sets, $S_1, S_2, \dots, S_n$.

Each set $S_i$ contains $m$ elements. One of these elements is the true value of the polynomial at that point, $P(x_i)$, while the other $m-1$ elements are random "noise" values from the field. In other words, for every $i$, we know that $P(x_i) \in S_i$, but we don't know which one it is.

**The Goal:** Given the points $x_i$ and the sets of candidates $S_i$, the challenge is to recover the original secret polynomial $P(X)$.

For the problem to have a unique solution, the parameters must satisfy a certain condition. There are $m^n$ possible combinations of points to form a polynomial. To ensure that only the correct combination results in a polynomial of degree $k$, we need a heuristic condition:

$$
m^n \ll |F|^{n-(k+1)}
$$

This intuitively means that if the noise level ($m$) is low enough or the field ($F$) is large enough, a random combination of points is highly unlikely to form a low-degree polynomial, making the true solution stand out.

### 1.2 Cryptographic Origins

The NPI problem wasn't just an abstract mathematical puzzle. It was introduced by Naor and Pinkas in their pioneering work on **Oblivious Polynomial Evaluation (OPE)**. In an OPE protocol, a server holds a secret polynomial $P(X)$ and a client holds a secret input $\alpha$. The goal is to allow the client to learn $P(\alpha)$ without revealing $\alpha$ to the server, and without the client learning anything more about $P(X)$ than the result $P(\alpha)$. The security of the client's input in this protocol relied directly on the presumed hardness of the NPI problem.

### 1.3 A Related Problem: Polynomial Reconstruction

To understand NPI's complexity, it's useful to compare it to a well-studied problem called **Polynomial Reconstruction (PR)**, also known in coding theory as the list-decoding problem for Reed-Solomon codes.

> **PR Problem:** Given $n$ points $(x_1, y_1), \dots, (x_n, y_n)$ and integers $k$ and $t$, find all polynomials $P(X)$ of degree at most $k$ that pass through at least $t$ of these points.

There is a simple reduction from NPI to PR. By taking all $nm$ possible points $(x_i, y_{i,j})$ where $y_{i,j} \in S_i$, we can frame the NPI problem as a PR instance where we are looking for a polynomial of degree $k$ that passes through exactly $t=n$ of these points. The best algorithms for PR, like the Guruswami-Sudan (GS) algorithm, can solve this if $n > \sqrt{k(nm)}$, which simplifies to $m < n/k$.

This reduction led to the initial belief that NPI was as hard as PR. However, this turned out to be a flawed assumption. A reduction only proves that NPI is *no harder than* PR, not that it is equally hard. The PR instances generated from NPI have a special structure that, as we will see, makes them much easier to solve.

---

## 2. Traditional Attacks on NPI

Before lattice-based methods shattered the NPI landscape, cryptanalysts relied on a few classical approaches.

### 2.1 From Error Correction to List Decoding

The most direct attack, as mentioned, is to reduce NPI to PR and use a list-decoding algorithm like Guruswami-Sudan (GS). This works, but only under the condition that $m < n/k$. This provides a clear benchmark for when NPI is weak: if the number of noisy candidates ($m$) is small relative to the ratio of points to degree ($n/k$), the problem is broken. However, for cryptographic applications with a larger $m$, this attack is ineffective.

### 2.2 The Algebraic Approach: Gröbner Bases

Another method is to translate NPI into a system of multivariate polynomial equations. We can express the unknown polynomial as $P(X) = \sum_{i=0}^{k} a_i X^i$, where the coefficients $a_i$ are our variables. For each point $x_i$, the condition $P(x_i) \in S_i$ can be written as a single equation:

$$
\prod_{j=1}^{m} (P(x_i) - y_{i,j}) = 0
$$

This gives us a system of $n$ equations in $k+1$ variables ($a_0, \dots, a_k$). Such systems can be solved using **Gröbner basis** algorithms. The fatal flaw of this approach is its complexity, which is typically super-exponential in the number of variables ($k$). For any cryptographically interesting parameters, this method is computationally infeasible.

### 2.3 A Divide-and-Conquer Strategy: Meet-in-the-Middle

The meet-in-the-middle attack is a classic divide-and-conquer algorithm that is far more efficient than brute force. Its core idea is to leverage the linearity of the Lagrange interpolation formula.

> **Lagrange Interpolation:** This is a method to find the unique polynomial of degree at most $n'-1$ that passes through a given set of $n'$ points. The formula is:
> $$
> P(X) = \sum_{i=1}^{n'} P(x_i) L_i(X) \quad \text{where} \quad L_i(X) = \prod_{j=1, j \neq i}^{n'} \frac{X - x_j}{x_i - x_j}
> $$

The attack works by splitting the sum into two halves. It generates a list of possible polynomial parts from the first half and another list from the second half. It then searches for a pair (one from each list) that, when added together, cancels out the high-degree terms, resulting in a polynomial of the correct low degree $k$.

The complexity is roughly $O(m^{n'/2})$, a huge improvement over the $O(m^{n'})$ of brute force. However, it's still exponential. More importantly, this attack was the first to exploit the unique structure of NPI—the fact that the evaluation points $x_i$ are fixed and known. This hinted that NPI's structure was its Achilles' heel.

---



## 3. The Game Changer: Lattice-Based Cryptanalysis

While traditional methods like algebraic solvers or meet-in-the-middle attacks showed limited success, the advent of lattice-based cryptanalysis completely changed the game. This powerful approach transforms the algebraic NPI problem into a geometric one. By representing potential solutions as vectors in a high-dimensional grid, or **lattice**, the problem becomes one of finding an unusually short vector in that grid—a task for which efficient algorithms exist.

> **What is a Lattice?** In simple terms, a lattice is a regular, repeating grid of points in a high-dimensional space. Think of the corners of a grid of cubes that extends infinitely in all directions. Lattice-based cryptanalysis often boils down to solving the Shortest Vector Problem (SVP): finding the non-zero point in this grid that is closest to the origin. While finding the absolute shortest vector is computationally hard, algorithms like LLL and BKZ are remarkably good at finding very short vectors, which is often enough to break a scheme.

### 3.1 From Polynomials to Lattices: The Linearization Framework

The first step is to linearize the NPI problem. The non-linearity comes from having to *choose* the correct $P(x_i)$ from each set $S_i$. We can represent this choice using integer indicator variables $\delta_{i,j}$:

$$
\delta_{i,j} = \begin{cases} 1 & \text{if } P(x_i) = y_{i,j} \\\\ 0 & \text{otherwise} \end{cases}
$$

Using these variables, the Lagrange formula becomes a linear combination:

$$
P(X) = \sum_{i=1}^{n} \sum_{j=1}^{m} \delta_{i,j} y_{i,j} L_i(X)
$$

The core constraint of NPI is that $P(X)$ must have a degree at most $k$. This means the coefficients of all terms with a power higher than $k$ (i.e., $X^{k+1}, \dots, X^{n-1}$) must be zero. This gives us a system of linear equations in the variables $\delta_{i,j}$, which defines our lattice.

> **Lemma 1.** Let $A \in M_{n,e}(\mathbb{Z}_q)$. Then the volume of $L(A)> $ divides $q^e$. It is exactly $q^e$ if and only if $\{\mathbf{x}A :
> \mathbf{x} \in \mathbb{Z}_q^n\}$ is entirely $\mathbb{Z}_q^e$.
> 
> **Proof:** By definition, $L(A)$ is the kernel of the group homomorphism $\phi$ that maps any $\mathbf{x} \in \mathbb{Z}^n$ to $(\mathbf{x}A \pmod q) \in \mathbb{Z}_q^e$. Therefore the group quotient $\mathbb{Z}^n / L(A)$ is isomorphic to the image of $\phi$. But since $L(A)$ is a full-dimensional lattice in $\mathbb{Z}^n$, its volume is simply the index $[\mathbb{Z}^n : L(A)]$ of $L(A)$ in $\mathbb{Z}^n$, from which both statements follow. $\square$
>
> Letting $L_i(x) = \sum_{w=0}^{n-1} \ell_{i,w}x^w$, the lattice $L$ of Section 3.1 is equal to $L(A)$, where $\mathbb{F} = \mathbb{Z}\_q$ and $A$ is the following matrix of dimension $nm \times n-1-k$.
> $$ A = \begin{pmatrix}
> y_{1,1} \ell_{1, k+1} & \cdots & y_{1,1} \ell_{1, n-1} \\\\
> \vdots & \ddots & \vdots \\\\
> y_{i,j} \ell_{i, k+1} & \cdots & y_{i,j} \ell_{i, n-1} \\\\
> \vdots & \ddots & \vdots \\\\
> y_{n,m} \ell_{n, k+1} & \cdots & y_{n,m} \ell_{n, n-1}
> \end{pmatrix}
> $$



### 3.2 The First Attempt: Defining a Lattice

Let's define a lattice $L$ as the set of all integer vectors $(d_{1,1}, \dots, d_{n,m})$ that satisfy these degree constraints. The solution to our NPI problem corresponds to a specific vector $\delta$ in this lattice.

This target vector $\delta$ has two special properties:

1.  All its entries are either 0 or 1.
2.  It contains exactly $n$ ones and the rest are zeros.

From this, we can calculate its Euclidean length:

$$
\|\delta\| = \sqrt{\sum_{i,j} \delta_{i,j}^2} = \sqrt{n}
$$

In a space with $nm$ dimensions, a vector of length $\sqrt{n}$ is exceptionally short. This is the central insight of the attack: **if our target vector is the shortest non-zero vector in the lattice, we can find it using lattice reduction algorithms.** Finding $\delta$ is equivalent to solving the NPI problem.

### 3.3 Why This Might Work: Short Vector Heuristics

Heuristically, the shortest vector in a "random" lattice of this volume is expected to be much longer than $\sqrt{n}$. This gives us good reason to hope that our target vector $\delta$ is unique and findable. However, this relies on the assumption that our lattice is "random," which it isn't. Its structure is determined entirely by the NPI instance.

### 3.4 A Crucial Refinement: A Better Lattice

To move from a heuristic to a provable attack, a key refinement is needed. We have not yet used a crucial piece of information: for each $i$, exactly one of the $\delta_{i,j}$ values (for $j=1, \dots, m$) is 1. This means $\sum_{j=1}^{m} \delta_{i,j} = 1$ for all $i$.

We can encode this powerful structural information by defining a new, smaller lattice (a sublattice), which we'll call A. This lattice A contains only the vectors from our original lattice $L$ that also satisfy the property that the sum of entries for each block $i$ is equal. The target vector $\delta$ is clearly in this sublattice A.

This refinement is profound. By adding more constraints, we drastically shrink the search space and build a lattice that is no longer "random-like," allowing for a rigorous analysis.

### 3.5 From Heuristics to Proof: Reducing NPI to SVP

With the refined lattice A, it's possible to prove that, under certain conditions, NPI can be reduced to the **Shortest Vector Problem (SVP)**.

The analysis shows that if the problem parameters are chosen correctly, the expected number of "bad" vectors (short vectors in the lattice that are not our target solution) is less than 1. This implies that with high probability, the target vector $\delta$ is the unique shortest non-zero vector in the lattice A.

Therefore, an algorithm that can solve SVP (in practice, approximated by algorithms like LLL and BKZ) can solve the NPI problem. This constitutes a formal, probabilistic reduction from NPI to SVP. The attack works by encoding enough of the problem's unique structure into the lattice to eliminate other "short" but incorrect vectors, allowing the true solution to be isolated.

### 3.6 Putting Theory into Practice: Experimental Results

The theoretical attack was backed by powerful experimental results. The researchers implemented the attack and tested it against NPI instances with cryptographically significant parameters.

For an instance with $n=160, m=2$ over an 80-bit field:

* The GS algorithm attack works only for degree $k \le 80$.
* The lattice attack was theoretically predicted to work for $k \le 155$.
* **In practice, the attack successfully solved instances with degrees up to $k=154$**, running in just a few hours.

This was a stunning result. The lattice attack was not just a theoretical curiosity; it was a practical tool that could break NPI instances with parameters far beyond the reach of any other known method.

**Table 1: Performance of the Refined Lattice Attack on NPI Instances**

| n    | m    | $\log_2(q)$ | k (Theoretical Limit) | k (Achieved in Practice) | Algorithm Used   | Runtime   |
| ---- | ---- | ----------- | --------------------- | ------------------------ | ---------------- | --------- |
| 160  | 2    | 80          | 155                   | $\le 152$                | BKZ-20           | < 4 hours |
| 160  | 2    | 80          | 155                   | 154                      | BKZ-20 + Pruning | 8 hours   |
| 115  | 3    | 80          | 110                   | 101                      | BKZ-20           | < 1 day   |
| 105  | 4    | 80          | 100                   | 80                       | BKZ-20           | < 1 day   |

> To put the theory to the test, I set up a specific NPI instance with $n=95, m=3$, and a polynomial degree of $k=76$. The underlying field was defined by a 40-bit prime modulus $( q \approx 2^{40})$ I then ran the lattice attack using BKZ-20 to solve it.[^2]
---

## 4. Re-evaluating NPI's Security

The discovery of this devastating attack had immediate and serious implications for any cryptographic protocol whose security relied on the NPI problem.

### 4.1 The Fallen Hardness Assumption

The core conclusion is undeniable: **The NPI problem is far easier to solve than originally believed and is not a suitable foundation for cryptographic security.**

The attack fundamentally disproved the conjecture that NPI was as hard as the general Polynomial Reconstruction (PR) problem. The lattice attack is highly customized; it specifically exploits the unique structure of NPI, which is absent in the general PR problem. This makes NPI a demonstrably weaker problem.

This serves as a crucial cautionary tale for cryptographers: simply reducing a new problem to a known hard problem is not enough to guarantee its security. The new problem's specific structure might introduce vulnerabilities that can be exploited by powerful, specialized tools like lattice reduction.

### 4.2 Recommendations for Secure Protocols

For protocols built on NPI, modifications are essential to restore security. Two primary recommendations emerged:

1.  **Switch to a Stronger Foundation (PR):** Since PR is still considered hard, protocols can be modified to rely on it instead. For example, instead of having a fixed set of evaluation points $x_i$, the protocol could be changed so the points are not structured in a way that allows the NPI attack.
2.  **Change the Algebraic Setting (DLP):** Another effective defense is to move the protocol into an algebraic group where the Discrete Logarithm Problem (DLP) is hard. Instead of exchanging values $y_{i,j}$ directly, participants would exchange $g^{y_{i,j}}$, where $g$ is a generator of the group. The lattice attack relies on linear relationships between the $y_{i,j}$ values. Hiding them with a one-way function $x \mapsto g^x$ conceals these relationships, thwarting the attack.

---

## 5. A Number Theory Analogy: The NCR Problem

There is a beautiful analogy between algebra and number theory, and the NPI problem has a direct counterpart called the **Noisy Chinese Remaindering (NCR) problem**.

### 5.1 Definition and Applications

The setup is similar: given an unknown integer $N$ and a set of moduli $p_1, \dots, p_n$, we are given for each modulus a small set of candidates $S_i$ that contains the true remainder $N \pmod{p_i}$. The goal is to find $N$. This problem appears in applications like counting points on elliptic curves.

### 5.2 Solving the NCR Problem

Methods for solving NCR are strikingly similar to those for NPI, including a powerful lattice-based technique known as the Coppersmith method. This leads to a natural question: can our direct lattice attack on NPI also break NCR?

### 5.3 Why Lattice Attacks Fail Against NCR

The answer, surprisingly, is **no**. While one can construct a similar lattice for the NCR problem, it typically fails in practice.

The reason lies in the arithmetic structure of the problem. When the NCR instance involves many small moduli (which is common in its applications), it's very likely that short linear dependencies exist among the candidate remainders (e.g., $r_{i,1} + r_{i,2} - r_{i,3} \equiv 0 \pmod{p_i}$). Each of these dependencies creates an unrelated, "parasitic" short vector in the lattice.

These parasitic vectors effectively flood the lattice with noise, making it impossible for lattice reduction algorithms to isolate the true target vector. In essence, **the number-theoretic structure of NCR creates too many false signals in the lattice, hiding the very solution the attack needs to find.** This highlights a deep condition for a successful lattice attack: not only do you need a short target vector, but the lattice itself must be "well-behaved" and not full of other short vectors arising from its own internal structure.

---

## 6. Conclusion and Future Work

This analysis tells the story of a cryptographic problem that went from a promising hardness assumption to being completely broken by a highly effective, specialized attack. The NPI problem is far weaker than anticipated due to its unique algebraic structure, which makes it fatally vulnerable to lattice cryptanalysis.

This serves as a critical lesson in protocol design: a problem's specific structure must be scrutinized against the most powerful analytical tools available. For NPI, this structure was not a feature but a bug. Any protocols still relying on it must be updated.

The contrast with the NCR problem further deepens our understanding, showing that the success of a lattice attack depends heavily on the underlying "geometric quality" of the lattice, which is dictated by the problem's algebraic or number-theoretic properties. This continues to be a rich area of research, pushing the boundaries of what is possible in the world of lattice-based cryptanalysis.

# Reference:

[^1]: [Bleichenbacher, D., & Nguyen, P. Q. (2000). Noisy Polynomial Interpolation and Noisy Chinese Remaindering. In International Conference on the Theory and Application of Cryptographic Techniques (EUROCRYPT 2000). Springer.](https://www.iacr.org/archive/eurocrypt2000/1807/18070053-new.pdf)
[^2]: [Experimental Code](https://github.com/GaloisHLee/NoisyInterpolation)
