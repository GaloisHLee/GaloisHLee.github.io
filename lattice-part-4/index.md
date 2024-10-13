# Lattice Part 4




**NTRU & Attack in CTF, usually with Lattice Reduction**



<!--more-->

# NTRU

Based on SVP, NTRU public key cryptosytsem was developed in 1996 by Hoffstein, Piper and Silverman. The NTRU public key cryptosystem is one of the fastest known public key cryptosystem. NTRU works in the ring of truncated polynomials (Quotient Ring, an algebra structure you should know):

$$
R_q=\frac{\mathbb{Z_q[X]}}{X^N-1},
$$

Where $N$ is a fixed positive integer and $\mathbb{Z_q}$ denotes the ring of integers modulo $q$. In this ring a polynomial $f$ can be written as 

$$
f = (f_0,\dots,f_{N-1})= \sum_{k = 0}^{N-1} f_k X^k
$$
It is worthy to recall the definition of Ring, and Quotient Ring in abstract algebra.

> Ring: 
>
> 1. $R$ is an abelian group under addition.
> 2. $R$ is a monoid under multiplication. (That means the inverse under multiplication isn't defined in Ring, also we have zero element for addition and identical element for multiplication)
> 3. Multiplication is distributive with respect to addition.



> Quotient Ring:
>
> A quotient ring (also called a residue-class ring) is a [ring](https://mathworld.wolfram.com/Ring.html)   that is the quotient of a [ring](https://mathworld.wolfram.com/Ring.html)  $A$ and one of its [ideals](https://mathworld.wolfram.com/Ideal.html) $a$ , denoted  $A / a$. 



To simplify, we can just see the polynomials in $R_q=\frac{\mathbb{Z_q[X]}}{X^N-1}$ as a new polynomial mod $X^N-1$ with coefficients in $\mathbb{Z}_q$.










## References

- [NTRU parameter setting](https://eprint.iacr.org/2015/708.pdf)
- [ntru.org](https://ntru.org/)
- [The NTRU Cryptosystem](https://shrek.unideb.hu/~tengely/crypto/section-8.html) 
- [Mathworld Wolfram](https://mathworld.wolfram.com/QuotientRing.html#:~:text=A%20quotient%20ring%20%28also%20called%20a%20residue-class%20ring%29,classes%20where%20%5Bx%5D%3D%20%5By%5D%20iff%20x-y%20in%20a.)

