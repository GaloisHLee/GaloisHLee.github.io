# ContinuedFraction


**Continued Fraction**

Be continued fraction,hence there is a limitation.

<!--more-->

# 1.Expression form

$$
x=a_0+\frac{1}{a_1+\frac{1}{a_2+\frac{1}{a_3+\frac{1}{\ddots}}}}
$$


$$
a_0 ∈\mathbb{Z},a_n∈\mathbb{Z}/\{0\} ~(n\ne0)
$$

If **partial numerators** and **partial denominators** are allowed to assume arbitrary values, which may include [functions](https://en.wikipedia.org/wiki/Function_(mathematics)), the resulting expression is called a [generalized continued fraction](https://en.wikipedia.org/wiki/Generalized_continued_fraction). When it is necessary to distinguish this standard form from generalized continued fractions, it is called a **simple** or **regular continued fraction**, or simply in **canonical form**.



# 2.Motive

## 2.1intro

> ·Different from decimal decimal notation.
>
> ·Infinite approximation to irrational numbers

Mathematically, it can be proven that the convergents obtained from (simple) continued fractions provide the best approximations to the exact value, whose numerator or denominator is less than the corresponding fraction of the next convergent.

## 2.2 Properties

In contrast to decimal expressions, continued fraction expressions have some properties:

- The continued fraction representation of a rational number is finite.
- The continued fraction representation of a "simple" rational number is short.
- Any rational number has a unique continued fraction representation, if it does not have a trailing 1. $([a_{0};a_{1},\ldots,a_{n},1]=[a_{0};a_{1},\ldots,a_{n}+1])$
- The continued fraction representation of an irrational number is unique.
- The terms of a continued fraction will be periodic if and only if it is the continued fraction representation of a quadratic irrational (i.e. the real solution to a quadratic equation with integer coefficients) [[1\]](https://zh.wikipedia.org/wiki/连分数#cite_note-1)[[2\]](https://zh.wikipedia.org/wiki/连分数#cite_note-2).
- The truncated continued fraction representation of a number *x* provides the "best possible" rational number approximation to *x* in a certain sense (see Theorem 5, Corollary 1 below).

**Diofantische Approximation**

## 2.3 Continued Fractions and Approximation Theory

### 2.3.1 Approximation Theorems

#### Theorem 1

Let $x$ be a real number, and let $a_0,a_1,a_2,\ldots,a_n$ be the coefficients of its truncated continued fraction. Then we have

$$
\left|x-\frac{p_n}{q_n}\right|<\frac{1}{q_n^2a_{n+1}}
$$

where $\frac{p_n}{q_n}$ is the simplest rational approximation of $x$.

#### Theorem 2

Let $x$ be a real number, and let $a_0,a_1,a_2,\ldots$ be the coefficients of its continued fraction. Then we have

$$
\left|x-\frac{p_n}{q_n}\right|<\frac{1}{q_nq_{n+1}}
$$

where $\frac{p_n}{q_n}$ is the simplest rational approximation of $x$.

### 2.3.2 Best Approximation

#### Theorem 3

Let $x$ be a real number, and let $a_0,a_1,a_2,\ldots$ be the coefficients of its continued fraction. Let $\frac{p_n}{q_n}$ be the simplest rational approximation of $x$. Then $\frac{p_n}{q_n}$ is the best rational approximation of $x$.

#### Theorem 4

Let $x$ be a real number, and let $a_0,a_1,a_2,\ldots$ be the coefficients of its continued fraction. Then the best rational approximation of $x$ is a simplest rational number of some truncated continued fraction of $x$.

#### Theorem 5

Let $x$ be a real number, and let $a_0,a_1,a_2,\ldots$ be the coefficients of its continued fraction. Let $\frac{p_n}{q_n}$ be the simplest rational approximation of $x$. Then we have:

1. $\frac{p_n}{q_n}$ is the best rational approximation of $x$.
2. For any positive rational number $\epsilon$, there exists an integer $n$ such that for all $n\geq m\geq 0$, we have

$$
\left|x-\frac{p_m}{q_m}\right|<\frac{1}{q_m^2}\epsilon
$$

#### Corollary 1

Let $x$ be a real number, and let $a_0,a_1,a_2,\ldots$ be the coefficients of its continued fraction. For any positive rational number $\epsilon$, there exists an integer $n$ such that for all $n\geq m\geq 0$, we have

$$
\left|x-\frac{p_m}{q_m}\right|<\frac{1}{q_m^2}\epsilon
$$

where $\frac{p_m}{q_m}$ is the simplest rational approximation of $x$ truncated at the $m$-th term.

### 2.3.3 Simplest Periodic Continued Fractions

#### Theorem 6

Let $x$ be a quadratic irrational. Then its simplest periodic continued fraction can be written as

$$
[a_0;\overline{a_1,a_2,\ldots,a_n}]
$$

where $a_0,a_1,a_2,\ldots,a_n$ are the coefficients of its truncated continued fraction, and $n$ is its smallest positive period.

#### Theorem 7

Let $x$ be a quadratic irrational. Then the denominator $q_n$ of its simplest periodic continued fraction satisfies

$$
q_n\leq\sqrt{D}n
$$

where $D$ is the discriminant of the irreducible quadratic equation of $x$.

# 3. Algorithm for Continued Fractions

## 3.1 Conversion of Continued Fractions

Consider a real number $r$. Let $i$ be its integer part and $f$ be its fractional part. Then the continued fraction representation of $r$ is $[i;...]$, where "..." is the continued fraction representation of $\frac{1}{f}$. It is customary to use a semicolon instead of the **first** comma.

To compute the continued fraction representation of a real number $r$, first write down the integer part (floor) of $r$, and then subtract this integer part from $r$. If the difference is zero, stop. Otherwise, find the reciprocal of this difference and repeat the process. This process is recursive and terminates if and only if $r$ is a rational number.

This algorithm is suitable for real numbers, but if implemented using floating-point numbers, it may cause numerical disasters. As an alternative, any floating-point number is an exact rational number (on modern computers, the denominator is usually a power of 2, and on electronic calculators, it is usually a power of 10), so a variant of the [Euclidean algorithm](https://en.wikipedia.org/wiki/Euclidean_algorithm) can be used to give an accurate result.

## 3.2 Notation

Continued fractions can be abbreviated as:
$$
x=[a_{0};a_{1},a_{2},a_{3}] 
$$
Or, written in Pringsheim's notation:
$$
x=a_{0}+\frac{1|}{|a_{1}}+\frac{1|}{|a_{2}}+\frac{1|}{|a_{3}} 
$$
There is also another notation:
$$
x=a_{0}+\frac{1}{a_{1}+}\frac{1}{a_{2}+}\frac{1}{a_{3}+}
$$
Sometimes angle brackets are used, such as:
$$
x\,=\,\langle a_{0}\!\colon a_{1},a_{2},a_{3}\rangle 
$$
The semicolon is optional when using angle brackets.
In addition, an infinite simple continued fraction can be defined as the limit:
$$
[a_{0};a_{1},a_{2},a_{3},\ldots]=\operatorname*{lim}_{n\to\infty}[a_{0};a_{1},a_{2},\ldots,a_{n}] 
$$
For any positive integers ${\hat{a}}_{1},\ {\hat{a}}_{2},\ {\hat{a}}_{3}$ ..., there exists this limit.
Or Gauss's notation can be used:
$$
x=a_{0}+\frac{3}{i=1}\ \frac{1}{a_{i}}
$$

## 3.3 Reciprocal of Continued Fractions

The continued fraction representation of a rational number and its reciprocal are the same except that if the number is less than or greater than 1, it is shifted left or right one place, respectively. In other words, $\left[a_0 ; a_1, a_2, a_3, \ldots, a_n\right]$ and $\left[0 ; a_0, a_1, a_2, \ldots, a_n\right]$ are reciprocals of each other. This is because if $a$ is an integer, then if $x<1$, $x=0+\frac{1}{a+\frac{1}{b}}$ and $\frac{1}{x}=a+\frac{1}{b}$, and if $x>1$, $x=a+\frac{1}{b}$ and $\frac{1}{x}=0+\frac{1}{a+\frac{1}{b}}$ with the last number generating the same continued fraction residues for $x$ and its reciprocal.
For example:
$$
\begin{aligned}
& 2.25=\frac{9}{4}=[2 ; 4] \\
& \frac{1}{2.25}=\frac{4}{9}=[0 ; 2,4]
\end{aligned}
$$

## 3.4 Infinite Continued Fractions

**Convergents:** The infinite continued fraction representation of an irrational number is very useful because its initial segments provide excellent rational approximations to the number. These rational numbers can be called the [convergents](https://en.wikipedia.org/wiki/Convergent_(continued_fraction)) of the continued fraction. All even-numbered convergents are less than the original number, while odd-numbered convergents are greater than it.

**Infinite Continued Fractions:**
All infinite continued fractions are irrational numbers, and all irrational numbers can be represented as infinite continued fractions in a unique way. The convergents are all less than the original number if they are even-numbered, and greater than it if they are odd-numbered.
$$
\frac{a_0}{1}, \quad \frac{a_1 a_0+1}{a_1}, \quad \frac{a_2\left(a_1 a_0+1\right)+a_0}{a_2 a_1+1}, \quad \frac{a_3\left[a_2\left(a_1 a_0+1\right)+a_0\right]+\left(a_1 a_0+1\right)}{a_3\left(a_2 a_1+1\right)+a_1}
$$
If the consecutive convergents are found, with numerators $h_1, h_2, \ldots$ and denominators $k_1, k_2, \ldots$, then the following recursive formulas hold:
$$
h_n=a_n h_{n-1}+h_{n-2}, \quad k_n-a_n k_{n-1}+k_{n-2}
$$
The continued fraction is given by the following formula:
$$
\frac{h_n}{k_n}=\frac{a_n h_{n-1}+h_{n-2}}{a_n k_{n-1}+k_{n-2}}
$$

$$
\frac{h_n}{k_n} = \frac{a_n h_{n-1}+h_{n-2}}{a_n k_{n-1} + k_{n-2}}
$$

The above equation is related to **ExGcd**, and it has important significance in subsequent attacks.

# 4.Theorem and concept

## 4.1 Theorem

### 4.1.1

**Theorem.1:**

for  $x \in \mathbb{R^+}$
$$
\left[a_0 ; a_1, \ldots, a_{n-1}, x\right]=\frac{x h_{n-1}+h_{n-2}}{x k_{n-1}+k_{n-2}}
$$
This expression is a formula for the $n$th convergent of the continued fraction expansion of $\alpha=[a_0;a_1,\ldots,a_{n-1},x]$. Here, $h_i$ and $k_i$ are the numerators and denominators of the $i$th convergent of $\alpha$, respectively.

The formula states that the $n$th convergent of $\alpha$ can be expressed as a fraction with numerator $x h_{n-1}+h_{n-2}$ and denominator $x k_{n-1}+k_{n-2}$. This formula is useful for approximating irrational numbers with rational numbers, since the convergents of a continued fraction are the best rational approximations to the original number.

For example, consider the continued fraction expansion of $\sqrt{2}$:

$$
\sqrt{2}=[1;2,2,2,\ldots]
$$

The convergents of $\sqrt{2}$ are:

$$
\frac{1}{1}, \frac{3}{2}, \frac{7}{5}, \frac{17}{12}, \frac{41}{29}, \frac{99}{70}, \frac{239}{169}, \frac{577}{408}, \frac{1393}{985}, \frac{3363}{2378}, \ldots
$$

Using the formula, we can compute the next convergent of $\sqrt{2}$:

$$
\frac{x h_{10}+h_{9}}{x k_{10}+k_{9}}=\frac{x \cdot 3363+1393}{x \cdot 2378+985}=\frac{3363 x+1393}{2378 x+985}
$$

This is the 11th convergent of $\sqrt{2}$. As $x$ gets larger, this fraction gets closer and closer to $\sqrt{2}$.

### 4.1.2

**Theorem.2**:

$\left[a_0, a_1, a_2, \ldots\right]$ whose convergence subsequence is
$$
\left[a_0 ; a_1, \ldots, a_n\right]=\frac{h_n}{k_n}, n \in \mathbb{N}
$$
constituent series, which converges to its limit  $\left[a_0, a_1, a_2, \ldots\right]$ 。

### 4.1.3

Theorem 3:
If the $m$-th convergent of a continued fraction is $h_n/k_n$, then
$$
k_n h_{n-1}-k_{n-1} h_n=(-1)^n
$$
Corollary 1: Each convergent is in its lowest terms (if $h_n$ and $k_n$ have an unusual common divisor, it can be divided by $k_n h_{n-1}-k_{n-1} h_n$, which is impossible).
Corollary 2: The difference between consecutive convergents is a unit fraction:
$$
\left|\frac{h_n}{k_n}-\frac{h_{n-1}}{k_{n-1}}\right|=\left|\frac{h_n k_{n-1}-k_n h_{n-1}}{k_n k_{n-1}}\right|=\frac{1}{k_n k_{n-1}}
$$
Corollary 3: The continued fraction is equivalent to the alternating series of terms:
$$
a_0+\sum_{n=0}^{\infty} \frac{(-1)^n}{k_{n+1} k_n}
$$
Corollary 4: The determinant of the matrix
$$
\left[\begin{array}{ll}
h_n & h_{n-1} \\
k_n & k_{n-1}
\end{array}\right]
$$
is either 1 or -1, so it belongs to the group of $2 \times 2$ unimodular matrices $S^* L(2, \mathbb{Z})$.



A brief explanation of how each corollary can be proved:

Corollary 1: If $h_n$ and $k_n$ have a common divisor, then we can divide both by their greatest common divisor to obtain a new pair of integers $(h_n', k_n')$ that represent the same fraction. Then, by Theorem 3, we have $k_n' h_{n-1}' - k_{n-1}' h_n' = (-1)^n$. But since $h_n'$ and $k_n'$ are relatively prime, their greatest common divisor is 1, so $k_n' h_{n-1}' - k_{n-1}' h_n'$ is also relatively prime to $h_n'$ and $k_n'$. Thus, it cannot be a divisor of both $h_n$ and $k_n$, which contradicts the assumption that they have a common divisor.

Corollary 2: The difference between consecutive convergents is given by $|h_n/k_n - h_{n-1}/k_{n-1}| = |(h_n k_{n-1} - k_n h_{n-1})/(k_n k_{n-1})|$. But by Theorem 3, we have $h_n k_{n-1} - k_n h_{n-1} = (-1)^n$, so the difference between consecutive convergents is $1/(k_n k_{n-1})$, which is a unit fraction.

Corollary 3: To prove this corollary, we can use the fact that the continued fraction is equivalent to the sequence of convergents. Then, we can use the formula for the difference between consecutive convergents (as in Corollary 2) to express the continued fraction as an alternating series of unit fractions.

Corollary 4: The determinant of the matrix $\begin{bmatrix} h_n & h_{n-1} \\ k_n & k_{n-1} \end{bmatrix}$ is $h_n k_{n-1} - h_{n-1} k_n$, which is $\pm 1$ by Theorem 3. The group $S^* L(2, \mathbb{Z})$ consists of $2 \times 2$ matrices with integer entries and determinant $\pm 1$, so this matrix belongs to the group.

## 4.1 Some Theorems

### 4.1.4 Theorem 4:

Each (nth) convergent is closer to subsequent (nth+1) convergents than to any of the preceding (rth) convergents. Symbolically, if the nth convergent is $\left[a_0 ; a_1, a_2, \ldots a_n\right]=x_n$, then
$$
\left|x_r-x_n\right|>\left|x_s-x_n\right|
$$
for all $r<s<n$:

Corollary 1: Odd convergents (before the nth) are monotonically increasing and always less than $x_n$.
Corollary 2: Even convergents (before the nth) are monotonically decreasing and always greater than $x_n$.



### 4.1.5 Theorem 5:

Theorem 5
$$
\frac{1}{k_n\left(k_{n+1}+k_n\right)}<\left|x-\frac{h_n}{k_n}\right|<\frac{1}{k_n k_{n+1}}
$$
Corollary 1: Any convergent is closer to the continued fraction than any other fraction with a denominator less than that of the convergent.
Corollary 2: Any convergent immediately preceding a large quotient is a best approximation to the continued fraction.

## 4.2 Some Concepts

### 4.2.1 Convergents and Semi-Convergents

**Convergents:**

> Excellent rational approximations to the number. These rationals are called the convergents (also known as "convergent fractions" or "convergent quotients") of the continued fraction.

**Semi-convergents:**

If $\frac{h_{n-1}}{k_{n-1}}$ and $\frac{h_n}{k_n}$ are consecutive convergents, then any fraction of the form
$$
\frac{h_{n-1}+a h_n}{k_{n-1}+a k_n}
$$
where $a$ is a non-negative integer and the numerator and denominator lie between the nth and (n+1)th terms (inclusive), is called a "semi-convergent", a sub-convergent, or a medi-ant. This term often implies that a possibility of being a convergent is excluded, and refers to a kind of half-convergent. The semi-convergents to the continued fraction of a real number $x$ include all rational approximations that are better than any with smaller denominators. Another useful property is that consecutive semi-convergents $\frac{a}{b}$ and $\frac{c}{d}$ satisfy $a d-b c= \pm 1$.

**The semi-convergents to the continued fraction of a real number $x$ include all rational approximations that are better than any with smaller denominators.**

### 4.2.2 Best Rational Approximations

Continued fractions theory plays a foundational role in the field of Diophantine approximation, and can solve the problem of best approximations to real numbers.

# 5. Summary of Development

- 300 BC - Euclid, "Elements" - An algorithm for the greatest common divisor generates a continued fraction as a by-product.
- 1579 - Rafael Bombelli, "L'Algebra Opera" - A method for extracting square roots is related to continued fractions
- 1613 - Pietro Cataldi, "Trattato del modo brevissimo di trovar la radice quadra delli numeri" - The first notation for continued fractions. Cataldi represented a continued fraction as $a_0$ $.\& \frac{n_1}{d_1} . $${\&} \frac{n_2}{d_2} .$ $ {\&} \frac{n_3}{d_3}$ with dots indicating where the continued fraction is to be continued.
- 1695 - John Wallis, "Opera Mathematica" - Introduces the term "continued fraction"
- c. 1780 - Joseph-Louis Lagrange - Provides a general solution to Pell's equation using continued fractions similar to Bombelli's.
- 1748 - Leonhard Euler, "Introductio in analysin infinitorum". Vol. I, Chapter 18 - Proves the equivalence of certain continued fractions and generalized infinite series.
- 1813 - Carl Friedrich Gauss, "Werke", Vol. III, pp. 134-138 - Derives very general continued fractions for complex-valued functions by using an identity involving hypergeometric series.

# References

- [wikipedia](https://zh.wikipedia.org/wiki/%E8%BF%9E%E5%88%86%E6%95%B0#%E5%8A%A8%E6%9C%BA)
- [zhihu](https://zhuanlan.zhihu.com/p/400818185)

