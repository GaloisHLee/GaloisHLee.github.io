# Lattice Part 9: Concrete Security of Lattice Schemes — Primal, Dual, BKZ


Reading: Peikert's survey as the wide-angle frame[^peikert-survey], Albrecht-Player-Scott for concrete-LWE attack modeling[^aps], Chen-Nguyen for BKZ quality heuristics[^cn11], the Homomorphic Encryption Standard for published parameter-table practice[^he-standard], and the LWE Estimator for the operational interface between these papers and actual numbers[^estimator].

Parts 0-3 fixed the geometric vocabulary: bases, reduced bases, SVP/CVP, cosets, and short modular witnesses. A deployed lattice scheme then adds concrete parameters such as dimension, modulus, secret distribution, error distribution, and sample count. None of those symbols, by itself, proves that one parameter set costs an attacker $2^{128}$ steps. That last sentence is not a theorem output. It is an attacker model layered on top of the theorem.

So this chapter stays on the attacker side of the interface. The real objects are primal attacks, dual attacks, BKZ block size, root-Hermite factor, and the estimator-style chain that turns $(n,q,\chi,k,m)$ into a work factor only after a long list of modeling decisions has been fixed.

This chapter therefore separates asymptotic hardness claims from concrete parameter-setting practice. It defines primal and dual attack viewpoints clearly, then uses concrete security estimation and attack-cost modeling to map parameter choices to attack-cost estimates.

In that exact sense, the goal is to connect BKZ quality assumptions to concrete lattice-scheme security reasoning rather than to repeat a security badge from the reduction side.

<!--more-->

## Reader State After Part 0-3

This chapter assumes only the first lattice layer:

- Part 0 gave the lattice object $\Lambda$ and the distinction between a lattice point and an arbitrary ambient vector.
- Part 1 explained why basis quality matters, via Gram-Schmidt lengths and approximate shortest vectors.
- Part 2 turned CVP into an algorithmic question: given a target near a lattice, can one recover the nearby lattice point or coset representative?
- Part 3 introduced exact modular relations and short witnesses, the same shape that later becomes SIS, ISIS, and LWE-style public data.

Concrete security estimation is the attacker-side continuation of those four facts. An LWE sample says that a public vector is close, modulo $q$, to a structured lattice point. A primal attack asks whether that closeness can be decoded. A dual attack asks whether a short modular relation can cancel the secret and leave a detectable error bias. BKZ is the basis-reduction engine used to make either route plausible.

So the central object is not a new cryptographic category. It is the same Part 0-3 geometry with numerical parameters attached:

$$
\text{lattice dimension, determinant, target norm, and basis quality}
\Longrightarrow
\text{attack success condition}.
$$

### Notation For Concrete Estimates

The notation in this article is deliberately close to estimator input syntax:

- $n$ is the plain-LWE dimension.
- $m$ is the number of available LWE samples.
- $q$ is the modulus.
- $\chi$ is the error distribution; $\sigma$ denotes its standard deviation when a Gaussian approximation is used.
- $\alpha=\sigma/q$ is the normalized error rate used in many dual estimates.
- $\mathbf{s}$ is the secret vector and $\mathbf{e}$ is the error vector.
- $k$ is the module rank in Module-LWE; after coefficient expansion the effective dimension is often written $N=kn$.
- $d$ is the dimension of the attack lattice, not necessarily the same as $n$ or $N$.
- $\beta$ is the BKZ block size.
- $\delta_0(\beta)$ is the root-Hermite factor heuristic attached to BKZ-$\beta$.

The IACR-style discipline is to keep four layers separate:

1. theorem-level reduction;
2. concrete attack family;
3. lattice-reduction quality heuristic;
4. classical or quantum cost model.

A published security claim is meaningful only after all four layers are named.

## Why Reductions Do Not Set Parameters For You

### What The Reduction Gives

Use the standard LWE sample notation

$$
A \in \mathbb{Z}_q^{m\times n},\qquad
\mathbf{s}\in \mathbb{Z}_q^n,\qquad
\mathbf{e}\leftarrow \chi^m,\qquad
\mathbf{b}=A\mathbf{s}+\mathbf{e}\pmod q.
$$

What the reduction gives is structural confidence. In the plain-LWE line, Regev's worst-case-to-average-case theorem says that an efficient solver for average random LWE instances would imply efficient algorithms for worst-case approximate lattice problems in the corresponding dimension regime. Peikert's survey is still the cleanest reminder of what this means and what it does not mean: the theorem certifies that the assumption is connected to genuinely hard lattice geometry, not that a concrete table entry has already been priced out[^peikert-survey].

Write that distinction explicitly:

$$
\text{worst-case lattice hardness}
\Longrightarrow
\text{average-case LWE hardness}
$$

is a reduction claim, while

$$
(n,q,\chi,k,m)\Longrightarrow \text{ ``128-bit security'' }
$$

is not.

The same separation persists for RLWE or MLWE. A module or ring reduction tells you why the structured assumption is not ad hoc, but it still does not tell you the cheapest known attack on one exact Kyber-like or Dilithium-like parameter tuple. Signature assumptions have the same issue: SIS or Module-SIS style hardness explains why short forgeries should be difficult, yet the concrete gap between a norm bound and an attack cost still has to be modeled.

> Reduction boundary: reductions transfer hardness meaning; they do not publish an attacker runtime table.

### What The Reduction Does Not Give

Concrete parameter setting needs attacker-side choices that the reduction never fixes for you:

- Which attack family is being optimized: primal, dual, hybrid, BKW-style, Arora-Ge style, or something structure-specific.
- Which objective is being measured: secret recovery, decision advantage, decoding, key recovery, or existential forgery.
- Which cost model is used for the lattice-reduction backend: classical sieve, quantum sieve, enumeration, or some calibrated hybrid.
- Which secret and noise distributions are assumed: uniform secret, small secret, sparse secret, centered binomial, discrete Gaussian, rounded ciphertext noise.
- How many samples are available, and whether the scheme leaks more effective equations through compression, reconciliation, failure behavior, or transcript structure.

This is why imprecise language is too weak. Saying "the reduction proves 128-bit security" collapses several different layers:

1. asymptotic reduction,
2. attack selection,
3. lattice-quality heuristic,
4. runtime or memory model,
5. implementation-facing parameter choice.

Concrete security lives in layers 2-5. The reduction mostly justifies layer 1.

## Primal And Dual Attack Viewpoints

To keep the rest of the chapter grounded, fix the LWE sample relation again:

$$
\mathbf{b}=A\mathbf{s}+\mathbf{e}\pmod q.
$$

For Module-LWE, read this in flattened form: a ring degree $n$ and module rank $k$ behave like an effective ambient dimension $N=kn$ once one expands coefficients. The attacker will often reason on that flattened object even if the scheme is implemented in $R_q^k$.

### Primal Attacks

The primal viewpoint is search-flavored. Instead of trying to distinguish distributions directly, it turns the samples into a lattice problem whose hidden short object encodes the secret or the error pattern. Depending on the exact reduction, the resulting task is described as bounded-distance decoding (BDD), unique shortest vector (uSVP), closest vector (CVP), or an embedding variant of one of these[^aps].

The mental picture is:

- build a $q$-ary lattice from the public samples,
- interpret the LWE instance as a target point displaced by a short vector,
- use lattice reduction to make that short vector visible enough to decode.

What is short here is not an abstract existential witness. It is tied to the concrete scheme parameters. If the coefficients of $\mathbf{e}$ have standard deviation $\sigma$, then the error norm is typically on the order of

$$
\lVert\mathbf{e}\rVert \approx \sigma \sqrt{m}.
$$

If the secret is also small, its norm contributes too. So the primal attack is governed by a comparison of two scales:

1. the norm of the hidden target vector that the attacker hopes to expose or decode;
2. the quality of the reduced basis produced by BKZ.

The attacker-side view is the same Part 0-3 geometry with a noisy modular relation attached: the attacker tries to convert public noisy algebra back into a geometric decoding problem.

The CVP/BDD lift is literal. Define the $q$-ary lattice generated by the sample matrix:

$$
\Lambda_q(A)=\{A\mathbf{z}+q\mathbf{y}:\mathbf{z}\in\mathbb{Z}^n,\ \mathbf{y}\in\mathbb{Z}^m\}\subset\mathbb{Z}^m.
$$

Because $\mathbf{b}=A\mathbf{s}+\mathbf{e}\pmod q$, there exists an integer vector $\mathbf{y}$ such that

$$
\mathbf{b}=A\mathbf{s}+q\mathbf{y}+\mathbf{e}.
$$

Thus $\mathbf{b}$ is within distance $\lVert\mathbf{e}\rVert$ of the lattice point $A\mathbf{s}+q\mathbf{y}\in\Lambda_q(A)$. Search-LWE security, in this simplified primal picture, is exactly the claim that this near-lattice target is still too hard to decode at the chosen parameters.

### Dual Attacks

The dual viewpoint is distinguishing-flavored. It looks for a short relation among the public samples that annihilates the secret term modulo $q$.

Take a vector $\mathbf{w}\in \mathbb{Z}^m$ such that

$$
\mathbf{w}^\top A \equiv \mathbf{0}^\top \pmod q.
$$

Then on a genuine LWE sample,

$$
\begin{aligned}
\mathbf{w}^\top \mathbf{b}
&=
\mathbf{w}^\top A\mathbf{s}+\mathbf{w}^\top \mathbf{e} \\
&\equiv
\mathbf{w}^\top \mathbf{e}
\pmod q.
\end{aligned}
$$

If the error is narrow enough and $\mathbf{w}$ is short enough, then $\mathbf{w}^\top \mathbf{e}$ is measurably biased toward small values instead of being uniform mod $q$. On a uniform fake sample, the same statistic is uniform. So the dual attack reduces decision-LWE to the problem of producing a sufficiently short dual relation and then accumulating enough statistical evidence[^aps].

This is why the primal and dual labels should not be blurred:

- primal attacks want a decoding event or a short hidden witness,
- dual attacks want a short annihilator that turns structure into bias.

In KEM analysis, both may be relevant because decision and search are both operationally meaningful. In signature analysis, the situation can diverge further because the concrete objective may be short-forgery search, transcript leakage, or secret-key recovery under repeated signing, not only plain LWE distinguishing.

## BKZ And Root-Hermite Factor

### What BKZ Is Controlling

BKZ is not itself an attack on LWE. It is the reduction engine that most lattice attacks call internally. Given a basis $B=(\mathbf{b}_1,\ldots,\mathbf{b}_d)$ of a $d$-dimensional lattice $\Lambda$, BKZ with block size $\beta$ tries to improve the basis by solving or approximating SVP inside projected blocks of dimension about $\beta$. Larger $\beta$ usually means better reduction quality and sharply higher cost[^cn11].

For concrete-security reasoning, that sentence matters more than the acronym expansion. The attacker is not asking "can I run BKZ?" The attacker is asking:

1. what quality would BKZ-$\beta$ heuristically achieve on the attack lattice,
2. is that quality enough for the primal or dual success condition,
3. how expensive is that $\beta$ under a chosen backend model.

Chen-Nguyen's BKZ 2.0 paper is the standard reference because it does not treat block size as a symbolic knob only; it ties block size to basis quality and simulated attack cost in a way that later concrete-LWE work reuses[^cn11].

### Why Root-Hermite Factor Appears Everywhere

The reason root-Hermite factor keeps showing up is that it compresses basis quality into one scalar. A common heuristic writes the BKZ quality achieved at block size $\beta$ as

$$
\delta_0(\beta)
\approx
\left(
\frac{\beta}{2\pi e}
(\pi\beta)^{1/\beta}
\right)^{\frac{1}{2(\beta-1)}}.
$$

This is not a theorem one should quote as an exact identity for every lattice. It is the kind of heuristic quality proxy that concrete-security work uses because it turns "good enough basis" into algebra.

Under the geometric-series-assumption style picture, the Gram-Schmidt lengths behave approximately like

$$
\lVert\mathbf{b}_i^*\rVert
\approx
\delta_0(\beta)^{\,d-1-2(i-1)} \det(\Lambda)^{1/d},
$$

and therefore the leading basis scale is roughly

$$
\lVert\mathbf{b}_1\rVert
\approx
\delta_0(\beta)^{\,d-1}\det(\Lambda)^{1/d}.
$$

That is why root-Hermite factor becomes the bridge variable between scheme parameters and attack cost. Determinants come from the scheme-induced lattice. Success thresholds come from the primal or dual objective. $\delta_0(\beta)$ is the compression layer that connects the two.

Equivalently, many concrete estimates silently follow the same template:

$$
\text{scheme parameters}
\Longrightarrow
\det(\Lambda),\ d,\ \text{target norm or bias}
\Longrightarrow
\delta_0^\star
\Longrightarrow
\beta^\star
\Longrightarrow
\text{cost}.
$$

## From Reduction Quality To Cost Estimates

### Block Size To Work Factor

Now write the estimator-style chain explicitly.

For a primal attack, one first derives a target norm $R_{\mathrm{primal}}$ from the secret and error geometry. In a simplified embedding picture,

$$
R_{\mathrm{primal}}
\approx
\sqrt{\lVert\mathbf{e}\rVert^2 + \lVert\mathbf{s}\rVert^2 + M^2},
$$

where $M$ is the embedding parameter and the dimension is some $d\approx m+n$ or $d+1$, depending on the exact formulation. The attack asks for a basis good enough that the hidden short vector is no longer lost in the lattice bulk. A minimal heuristic success inequality is therefore

$$
\delta_0(\beta)^{\,d-1}\det(\Lambda)^{1/d}
\lesssim
R_{\mathrm{primal}}.
$$

One does not usually stop at this line, because it still contains the unknown $\beta$. The next step is to solve the inequality for the smallest block size $\beta^\star$ that makes the attack plausible. Only then does one attach a cost model such as

$$
\log_2 T_{\mathrm{sieve}}(\beta^\star)
\approx
c_{\mathrm{sieve}}\beta^\star
$$

or an enumeration-style model with a different dependence on $\beta^\star$. The constant is not universal. It depends on which sieve variant, which memory assumptions, and whether the estimate is classical or quantum.

For a dual attack the chain is different, but the logic is the same. If BKZ on the dual lattice yields a short vector $\mathbf{w}$, then the decision bias is controlled by the Fourier decay of the error distribution. In the common Gaussian-style normalization one gets a bias of the rough form

$$
\varepsilon_{\mathrm{dual}}
\approx
\exp\!\big(-2\pi^2 \alpha^2 \lVert\mathbf{w}\rVert^2\big),
\qquad
\alpha = \sigma/q.
$$

So dual success means two things at once:

1. BKZ must make $\lVert\mathbf{w}\rVert$ small enough.
2. The adversary must have enough samples for that bias to become statistically visible.

If the dual lattice has determinant $\det(\Lambda^\ast)$ and dimension $d$, then one again substitutes a BKZ-quality heuristic such as

$$
\lVert\mathbf{w}\rVert
\approx
\delta_0(\beta)^{\,d}\det(\Lambda^\ast)^{1/d},
$$

and solves backward from the bias threshold to the required block size. In the roughest sample-complexity view, the distinguisher only becomes meaningful once the accumulated evidence crosses a threshold such as

$$
\sqrt{m}\,\varepsilon_{\mathrm{dual}} \gtrsim 1
$$

or an equivalent likelihood-ratio criterion, so the same attack family factors into a geometric question and a statistical question.

This is the cleanest way to separate asymptotic and concrete language. The theorem says the distributions are computationally hard to distinguish in an asymptotic sense. The concrete estimate says: under a BKZ-$\beta$ quality heuristic and a chosen cost model, the shortest useful dual relation should cost about so many bits of work.

A useful attacker-side runbook is therefore:

1. flatten the scheme into the attacker's parameter set, often with an effective dimension like $N=kn$ for MLWE;
2. derive the attack lattice dimension $d$, determinant scale, and sample count actually available;
3. translate the scheme's secret and noise distributions into either a target norm $R_{\mathrm{primal}}$ or a bias parameter $\varepsilon_{\mathrm{dual}}$;
4. solve for the smallest $\beta$ whose heuristic $\delta_0(\beta)$ meets that target;
5. price that $\beta$ under an explicit classical or quantum backend model.

That is the concrete-security pipeline which estimator tools compress into one output table.

### Scheme Parameters To Attack Inputs

Estimator-style tools are useful because they automate the bookkeeping, not because they remove the modeling burden. The bookkeeping itself is already nontrivial:

- $n$ is the plain-LWE dimension or the ring degree.
- $k$ is the module rank, so MLWE often contributes an effective dimension $N=kn$ after coefficient expansion.
- $q$ controls lattice determinant scales, spacing in the dual, and the normalization of the noise rate.
- $\chi$ or $\sigma$ controls the error norm and therefore the decoding radius or dual bias.
- $m$ controls how many equations the attacker can use and often enters dimension-optimization choices directly.
- The secret distribution matters. Small or sparse secrets can shift which attack dominates.

That last point is easy to understate. A paper or estimator input that assumes uniform $\mathbf{s}$ is not modeling the same attack landscape as one that assumes centered-binomial or sparse ternary secrets. The primal embedding geometry changes. The dual advantage changes. Hybrids that guess part of the secret become more or less attractive.

Structured ring and module schemes add another layer. A Kyber-like MLWE instance is not just "LWE with different constants." The attacker may flatten the module structure into a larger plain lattice, but the input still carries module rank, ring degree, coefficient distribution, rounding rules, and sometimes decryption-failure or reconciliation behavior that affect how one translates scheme syntax into attack syntax[^he-standard].

Lattice signatures add a different complication. Falcon-like and Dilithium-like systems do not reduce to one single "LWE security number." The attacker objective may be:

- recovering a signing key,
- finding a short forgery for a Module-SIS style relation,
- exploiting transcript structure or rejection-sampling leakage,
- or attacking an NTRU-style key equation.

So even when the same BKZ language reappears, the concrete target condition is no longer identical to a KEM key-recovery condition. Trapdoors, Gaussian sampling, rejection sampling, and short-witness generation change the attack objective. The attacker sees those differences too.

## Reading Estimator Outputs As Models

### What An Estimator Assumes

The LWE Estimator is valuable precisely because it exposes, in executable form, the chain sketched above[^estimator]. But the right way to read its output is:

> this is the cheapest modeled attack under the current assumptions,

not:

> this is the theorem-certified cost of breaking the scheme.

When an estimator reports that `dual`, `bdd`, `usvp`, or a hybrid variant is the winner, it has already fixed several hidden choices:

- the noise normalization,
- the secret distribution,
- the number of available samples,
- the lattice-reduction quality heuristic,
- the backend cost model,
- the classical versus quantum setting,
- and the success threshold used to declare an attack plausible.

The Homomorphic Encryption Standard is a good secondary reference here because it presents concrete parameter tables only after fixing exactly these kinds of modeling choices, often with separate rows for classical or quantum estimates and with explicit caveats about secret distributions or noise growth assumptions[^he-standard].

So an estimator output is best read as a model summary:

$$
\text{chosen assumptions}
\Longrightarrow
\text{best modeled attack}
\Longrightarrow
\text{estimated cost}.
$$

If you change the assumptions, you may change the winning attack. That is not a bug. It is the whole point.

### What Changes Between KEMs And Signatures

The KEM/signature split makes this distinction unavoidable.

For KEMs or encryption systems based on LWE/RLWE/MLWE, the main concrete questions are usually variants of:

- can the attacker recover the secret key,
- can the attacker distinguish structured samples from random,
- can compression or failure behavior amplify the attack surface.

For lattice signatures, the main concrete questions often involve different interfaces:

- can the attacker produce a new short witness satisfying a public Module-SIS or NTRU-style relation,
- can repeated transcripts leak enough information to support key recovery,
- can implementation details in Gaussian or rejection sampling shift the concrete margin.

This is why it is technically wrong to quote one naked "128-bit" number across both families without saying which objective it belongs to. A Module-LWE KEM and a Module-SIS signature may both invoke BKZ in their security discussion, yet the corresponding attack lattices, success conditions, and sample models are not the same.

Peikert's survey framed lattice cryptography from the construction side. APS and the estimator line framed it from the cost-model side. Putting those together gives the right reading discipline:

1. use reductions to justify why the assumption class matters,
2. use primal or dual attack models to define a concrete adversary,
3. use BKZ and root-Hermite-factor heuristics to turn success conditions into block sizes,
4. use an explicit runtime model before attaching a security label.

Any shortcut that skips step 2 or step 4 is usually where the modeling error enters.

## Conclusion

The introductory geometry gave the lattice, coset, and short-witness language. Concrete lattice cryptography adds noisy linear relations, structured scheme families, and signing interfaces that need more than plain hardness. This chapter adds the attacker-side lens needed to price those objects.

Concrete security for lattice schemes is not the direct output of a worst-case reduction. It is a modeling discipline. One picks a primal or dual viewpoint, derives the lattice geometry or the distinguishing bias, translates the needed success condition into a BKZ quality requirement, compresses that quality through a root-Hermite-factor heuristic, and only then maps block size to a runtime estimate.

That is the reason careful papers and standards do not stop at "128-bit secure." They say which problem, which attack family, which BKZ model, which secret distribution, which sample regime, and which cost assumptions are carrying that number.

## References

Suggested reading order:

1. Peikert first, for the reduction-level lattice-cryptography frame and the distinction between worst-case and average-case hardness.
2. Albrecht-Player-Scott next, because it is the clean bridge from LWE parameters to concrete primal and dual attacks.
3. Chen-Nguyen after that, for BKZ quality, root-Hermite factors, and block-size reasoning.
4. The Homomorphic Encryption Standard parameter guidance next, to see how concrete estimates are reported in public parameter tables.
5. The LWE Estimator documentation last, as an executable version of the attack-model bookkeeping rather than as an oracle.

[^peikert-survey]: Chris Peikert (2016), *A Decade of Lattice Cryptography*.
[^aps]: Martin R. Albrecht, Rachel Player, and Sam Scott (2015), *On the Concrete Hardness of Learning with Errors*.
[^cn11]: Yuanmi Chen and Phong Q. Nguyen (2011), *BKZ 2.0: Better Lattice Security Estimates*.
[^he-standard]: HomomorphicEncryption.org, *Homomorphic Encryption Standard*, current public standard and parameter guidance documents.
[^estimator]: Martin R. Albrecht et al., *The LWE Estimator* project and documentation, <https://github.com/malb/lwe-estimator> and <https://lattice-estimator.readthedocs.io/>.

