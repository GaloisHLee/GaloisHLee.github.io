# Lattice Part 10: Implementation Fault Lines in Lattice Cryptography


Reading: Peikert's survey as the wide-angle frame[^peikert], NTRU Prime's attack-surface language[^ntruprime], NIST's standardized ML-KEM / ML-DSA interfaces[^fips203][^fips204], and the implementation-attack papers that target Kyber, BLISS, Falcon, and lattice decapsulation itself[^kyberslash][^mlkem-fault][^dfa-kyber][^bliss-cache][^bliss-branch][^falcon][^dilithium-fault]. Part 10 closes the lattice subseries at the only boundary that really matters in deployment: proof-level security is about an abstract construction, while deployed security is about the arithmetic, randomness, control flow, and fault behavior that actually reach silicon.

From Parts 0-3, read every deployed scheme as a concrete implementation of modular lattice relations over short coefficient vectors. Structured rings and modules buy compactness because multiplication becomes regular and sampling stays cheap enough to ship. The same structure also narrows the implementation surface into a few hot loops: NTT butterflies, modular reductions, coefficient compression, rejection tests, hint generation, and decapsulation compares. Those loops are exactly where timing, power, cache behavior, and induced faults start to expose data that the proof never modeled.

The useful split is therefore not "lattices have side channels too." That is too generic to help. The useful split is KEM versus signature. A KEM exposes a decapsulation oracle whose correctness margins and implicit rejection logic can be probed adaptively. A signature scheme exposes a signing loop whose sampler and abort logic must maintain the right distribution across many signatures under one long-term key. Same hardness family, different failure physics.

<!--more-->

## Reader State After Part 0-3

This final chapter assumes only the first lattice layer:

- Part 0 gave lattice points, bases, determinant, and distance.
- Part 1 explained why a basis representation can make the same lattice easy or hard to navigate.
- Part 2 introduced CVP and nearest-plane reasoning, which is the right mental model for decoding a noisy lattice relation.
- Part 3 introduced exact modular relations with short witnesses, the same syntax that later appears in KEM public keys, signatures, and verification equations.

Implementation security adds a different question to that geometry. The proof says that a certain relation is hard to solve or forge. The implementation executes that relation through integer arithmetic, memory accesses, randomness expansion, rejection loops, comparisons, and fault-prone hardware. The attacker may not solve the lattice problem directly; the attacker may instead observe or perturb the implementation until it exports an easier relation.

The reading discipline is therefore:

$$
\text{abstract lattice relation}
\Longrightarrow
\text{coefficient-vector program}
\Longrightarrow
\text{observable leakage or fault behavior}.
$$

### Threat Model And Notation

Use the following notation throughout the chapter:

- $R_q=\mathbb{Z}_q[x]/(x^n+1)$ is the polynomial ring used by ML-KEM / ML-DSA style arithmetic.
- $a(x)=\sum_i a_i x^i$ is stored as a coefficient vector $\mathbf{a}=(a_0,\ldots,a_{n-1})$.
- $\widehat{\mathbf{a}}=\operatorname{NTT}(\mathbf{a})$ denotes the NTT-domain representation.
- $\operatorname{Decaps}_{sk}(c)$ is the KEM decapsulation computation, including decrypt, re-encrypt, compare, and select.
- $\operatorname{Sign}_{sk}(\mu;\rho)$ is the signing computation, including mask expansion, challenge generation, rejection checks, and hint generation.
- $\mathcal{L}$ denotes an implementation leakage channel: timing, cache state, power, EM, branch trace, or intermediate buffer exposure.
- $\mathcal{F}$ denotes an injected fault that changes an intermediate arithmetic state or control-flow decision.

The proof model usually gives the adversary only the final transcript:

$$
\textsf{output}=\operatorname{Decaps}_{sk}(c)
\quad\text{or}\quad
\sigma=\operatorname{Sign}_{sk}(\mu;\rho).
$$

The implementation model may also give partial information of the form

$$
\mathcal{L}(\text{state}_0,\text{state}_1,\ldots)
\quad\text{or}\quad
\operatorname{output after }\mathcal{F}(\text{state}_i).
$$

That extra interface is where most real implementation attacks enter.

## Proof Security Versus Implemented Security

### What The Proof Model Sees

Peikert's survey is still the cleanest framing source for the whole subseries because it keeps the abstraction layers separate.[^peikert] Worst-case lattice hardness, average-case LWE / RLWE / MLWE assumptions, and concrete constructions do not live at the same level. Once we arrive at the scheme level, the proof is no longer reasoning about caches, integer dividers, or voltage glitches. It is reasoning about algebraic objects and distributions.

For the KEM line, the proof view is something like:

$$
\text{public relation} + \text{small noise} + \text{Fujisaki-Okamoto style check}
\Longrightarrow
\text{IND-CCA security in the stated model}.
$$

For the signature line, the proof view is something like:

$$
\text{short-response generation} + \text{Fiat-Shamir with aborts or GPV-style sampling}
\Longrightarrow
\text{EUF-CMA security in the stated model}.
$$

In both cases, the proof assumes that the implementation exports only the final interface:

- a shared key for decapsulation,
- or a signature for signing,
- and not the intermediate state that produced it.

That assumption is stronger than it first appears. For ML-KEM, the proof does not grant the adversary a cycle count for a division instruction or a bit telling whether the internal re-encryption compare almost failed. For ML-DSA or Falcon, the proof does not grant the adversary the iteration count of a rejection loop, the branch trace of a sampler, or a partially faulted ephemeral mask. These are not "mere engineering details." They are extra observables that live outside the security game.

> Implementation boundary: a proof certifies a transcript distribution and an input-output relation. An implementation executes a concrete state machine that may emit much more than that transcript.

### What The Implementation Adds

The implementation adds at least five security-relevant layers that the construction proof usually compresses away:

- a representation layer: coefficient form versus NTT form, signed versus centered residues, compressed versus decompressed coefficients;
- a reduction layer: Montgomery, Barrett, integer division, carry propagation, and rounding;
- a randomness layer: samplers, loop bounds, fault handling, and state reuse;
- a memory layer: where transformed secrets live, how long they persist, and which buffers are reused;
- a machine layer: variable-latency instructions, cache footprints, EM leakage, and injectable faults.

The standards themselves quietly admit this gap. FIPS 203 and FIPS 204 both impose requirements on destroying intermediate values and both forbid floating-point arithmetic in the standardized implementations.[^fips203][^fips204] FIPS 204 also explicitly notes that deterministic signing and some verification contexts need additional care against side-channel and fault attacks.[^fips204] That is already a signal that "provably secure scheme" and "secure implementation" are not interchangeable phrases.

NTRU Prime is useful here not because it replaced the MLWE line, but because its design criterion is explicit: attack surface is part of the design objective, not post-hoc cleanup.[^ntruprime] That is the right mindset for this last chapter. Once the mathematical construction is fixed, the next security question is not "can we optimize it?" but "which hot paths did the optimization force us to trust?"

## Arithmetic Fault Lines: NTT, Reduction, And Memory Layout

### NTT Kernels As Side-Channel Surfaces

Structured multiplication is an efficiency argument. In the implementation chapter, the same derivation has to be reread as an exposure argument.

Let

$$
R_q = \mathbb{Z}_q[x]/(x^n + 1),
$$

with the usual negacyclic choice used in ML-KEM and ML-DSA. Polynomial multiplication is implemented by moving into a transform domain:

$$
\begin{aligned}
c(x)
&= a(x)b(x) \\
&=
\operatorname{NTT}^{-1}\!\big(\hat a \odot \hat b\big),
\end{aligned}
$$

where

$$
\hat a = \operatorname{NTT}(a), \qquad
\hat b = \operatorname{NTT}(b).
$$

At the proof level, this is just one efficient realization of ring multiplication. At the implementation level, it means the most security-critical computations are concentrated in a small set of butterflies, pointwise multiplies, and modular reductions. FIPS 203 has a dedicated NTT section for ML-KEM, and FIPS 204 exposes `MultiplyNTT`, `MatrixVectorNTT`, and related transformed-domain primitives all the way up its algorithm ladder.[^fips203][^fips204]

That concentration is why the NTT becomes a side-channel surface. The address pattern may be fixed, the loop trip count may be fixed, and the source may contain no secret-dependent branches, yet the transform is still processing secret-bearing coefficients:

- in ML-KEM decapsulation, secret polynomials participate in the subtraction that recovers the message;
- in ML-DSA signing, secret key pieces and ephemeral masks pass through transformed-domain multiplies and decompositions;
- in Falcon-like designs, transform-domain arithmetic is even closer to the sampler itself.[^falcon]

So the right question is not "does the NTT branch on secrets?" The right question is "what analog or microarchitectural signal is emitted while secret-bearing coefficients pass through the transform and reduction stack?" Power, EM, fault sensitivity, and register-level timing can all answer that question long before the source code looks suspicious.

The memory argument is similar. A coefficient vector and its NTT image are mathematically equivalent, but operationally they are different secrets at different addresses with different lifetimes. Caching a public transformed matrix is fine; both standards allow reuse of public derived values.[^fips203][^fips204] Caching transformed secret state, or failing to wipe it, simply widens the number of points where the same algebraic object can leak.

### Reduction And Division Hazards

The proof says "reduce modulo $q$." The machine has to decide how.

That choice is not bookkeeping. It is part of the leakage model. The cleanest attack chain in this chapter is:

$$
\text{structured ring arithmetic}
\to
\text{NTT-domain products}
\to
\text{coefficient scaling / compression}
\to
\text{integer reduction or division}
\to
\text{timing oracle}.
$$

KyberSlash is the canonical example.[^kyberslash] The affected implementations were written with standard source-level constant-time discipline in mind: no obvious secret-dependent branches, no obvious secret-dependent table lookups. But the compiled code still used variable-time division instructions tied to arithmetic involving the Kyber modulus $q = 3329$. Once carefully chosen ciphertexts steer the numerators that reach those divisions during decapsulation, timing begins to reveal secret-dependent information anyway.

The important lesson is lattice-specific. The vulnerable instructions were not in an unrelated helper path. They sat in compression / decoding style arithmetic that exists because ML-KEM has to move noisy ring elements across a narrow byte-level interface. In other words:

$$
\text{proof-level ciphertext coefficient}
\not=
\text{implementation-level byte encoding}.
$$

The encoding layer has its own attack surface.

This is also why "integer-only" is not a complete defense condition. FIPS 203 and FIPS 204 forbid floating-point arithmetic for the standardized schemes, which removes one class of rounding surprises.[^fips203][^fips204] But integer arithmetic still contains variable-latency division, carry chains, and reduction tricks. Constant-time for lattice code therefore means more than branch-free code. It means secret-independent instruction selection and secret-independent instruction latency on the actual target machine.

## Sampling, Rejection, And Signature-Side Fragility

### Sampling Side Channels

Signature-side fragility is not just "another side-channel problem." The signer is a repeated distribution engine. Every signature is a new sample under the same long-term secret key, and the proof only works if that sample comes from the right distribution.

For ML-DSA, the central signing relation can be compressed to

$$
w = A y, \qquad
c = H(\mu \mathbin\Vert \operatorname{HighBits}(w)), \qquad
z = y + c s_1,
$$

with additional low-bit / hint machinery ensuring that the verifier can reconstruct the public commitment information without learning too much about the dropped parts.[^fips204][^dilithium]

The key point is that the proof does not want an arbitrary short $z$. It wants the distribution of accepted $(z,h,c)$ to look right. That is why ML-DSA signing is surrounded by explicit rejection logic, and why FIPS 204 spends real space on procedures such as `ExpandMask`, `RejNTTPoly`, `SampleInBall`, `HighBits`, `LowBits`, and `MakeHint`.[^fips204] The standard even gives a safe cap of 814 iterations for the `Sign_internal` rejection loop when bounding the probability of an overrun below $2^{-256}$.[^fips204]

That detail matters because it reveals what the implementation is really preserving: not merely correctness, but a distributional invariant. Timing leakage from that invariant is not a cosmetic side channel. It is leakage about how close an ephemeral sample was to the secret-dependent acceptance boundary.

This pressure becomes even sharper in Gaussian-sampling signatures. BLISS and Falcon compress signatures precisely by making the sampler more mathematically structured and more statistically delicate.[^bliss-cache][^bliss-branch][^falcon] In that family, the sampler is not peripheral. The sampler is the scheme. If the sampler leaks via cache lines, branch traces, or floating-point behavior, then the proof's distributional claim is being falsified at the point where it matters most.

### Rejection Logic As A Distribution Interface

Rejection logic in lattice signatures is easy to misread as "retry until the numbers are small enough." That is too weak. In the proof, rejection is the mechanism that detaches the published response distribution from the secret key strongly enough for Fiat-Shamir with aborts to work as intended.

For ML-DSA, the rejection conditions test whether the response and hint data remain inside carefully chosen bounds. Operationally, that means the implementation is evaluating secret-adjacent predicates such as:

$$
\lVert z\rVert_\infty < \gamma_1 - \beta,
$$

along with corresponding low-bit and hint-weight bounds.[^fips204]

If an implementation leaks:

- how many attempts were rejected,
- which rejection condition fired,
- or whether a fault skipped one of the norm or hint tests,

then it is leaking information about where the secret-shifted response landed relative to those boundaries.

The fault version is especially concrete. Suppose a fault causes reuse, or partial reuse, of an ephemeral signing value. Then for two signatures on different challenges,

$$
z_1 = y + c_1 s_1, \qquad
z_2 = y + c_2 s_1,
$$

so subtraction gives

$$
z_1 - z_2 = (c_1 - c_2) s_1.
$$

That is no longer a diffuse side-channel intuition. It is a direct algebraic handle on the secret. Differential fault attacks on deterministic lattice signatures exploit precisely this collapse from "fresh masked response" to "linear relation in the signing secret."[^dilithium-fault]

This is why signature-side fault lines do not look like KEM-side fault lines. The signature attacker is not primarily trying to cross a decryption boundary. The attacker is trying to learn, reuse, bias, or fault the ephemeral distribution that makes the public response safe to publish in the first place.

## Decryption Failure And KEM-Side Boundaries

### Where Failure Probability Enters

For the ML-KEM side, the decisive object is not a sampler output but a decapsulation margin.

Write the usual module-LWE style equations in compressed form, matching the Kyber / ML-KEM line from the original scheme paper and the standardized variant:[^kyber][^fips203]

$$
t = A s + e,
$$

and for ciphertext components,

$$
u = A^\top r + e_1, \qquad
v = t^\top r + e_2 + \operatorname{Encode}(m).
$$

Substituting the public key relation into decapsulation gives

$$
\begin{aligned}
v - s^\top u
&=
\operatorname{Encode}(m) + e^\top r + e_2 - s^\top e_1
\pmod q.
\end{aligned}
$$

If we name the aggregate noise term

$$
\nu = e^\top r + e_2 - s^\top e_1,
$$

then the correctness condition is simply that the coefficients of $\nu$ stay on the right side of the quantization boundary that separates one message representative from another.

This is the proof-level meaning of decapsulation failure. It is not an implementation bug. It is the tail probability that the honest aggregate noise crosses a rounding threshold. FIPS 203 lists these probabilities as roughly $2^{-138.8}$, $2^{-164.8}$, and $2^{-174.8}$ for the three standardized parameter sets.[^fips203] D'Anvers and Batsleer are useful here because they remind us that tiny failure probabilities are still security-relevant objects once attackers get many targets or many carefully crafted queries.[^dfa-kyber]

The engineering lesson is that a correctness margin is already an attack surface waiting for an observation channel. If an adversary can tell whether a ciphertext landed on one side of the margin or the other, then the adversary is no longer interacting with the idealized KEM proof interface. The adversary is interacting with a noisy threshold oracle whose threshold depends on secret-bearing arithmetic.

### Oracle And Fault Effects

FIPS 203 makes the ML-KEM decapsulation pipeline explicit.[^fips203] Internally it does four security-relevant things:

1. decrypts the ciphertext to obtain a candidate message $m'$;
2. derives candidate shared material and re-encryption randomness from $m'$;
3. re-encrypts to obtain $c'$;
4. compares $c$ and $c'$ and performs implicit rejection if they differ.

Written as a chain:

$$
c
\to
m' = \operatorname{Decrypt}(c)
\to
(K',r')
\to
c' = \operatorname{Encrypt}(m',r')
\to
[c \stackrel{?}{=} c'].
$$

The standard is careful on purpose: the implicit-reject flag is secret intermediate data and must not be exposed.[^fips203] But implementations do not execute "a secret flag." They execute a compare, a branchless select, some buffer handling, and a return path on real hardware. Every one of those stages can become observable through timing, power, EM leakage, cache traffic, or injected faults.

KyberSlash2 is the sharp example because it turns arithmetic leakage in re-encryption-style code into a plaintext-checking oracle during decapsulation.[^kyberslash] The proof assumes the ciphertext equality check leaks nothing except the final shared secret. The attack shows that if timing leaks enough about the re-encryption path, the attacker can recover more than the security game ever allowed.

Fault attacks on lattice KEMs make the same point from the opposite direction.[^mlkem-fault] There, the attacker does not passively listen to timing. The attacker perturbs the decapsulation computation and watches whether the perturbation was effective. Once "effective versus ineffective fault" becomes observable, the attacker again receives a richer oracle than the scheme proof modeled. The difference between a valid and invalid ciphertext, or between two nearby secret-dependent arithmetic states, stops being internal.

This is the KEM-side fault line in one sentence:

> proof-level ML-KEM security assumes one shared-key output; implementation-level ML-KEM security depends on whether decapsulation leaks any side information about how close the ciphertext was to the decryption and re-encryption boundaries.

## KEM Versus Signature Implementation Tradeoffs

### Transcript Differences

The same hardness family does not imply the same attacker workflow.

For KEMs, the attacker's natural interface is adaptive probing. The decapsulation device sees adversarial ciphertexts, performs secret-key arithmetic, and is asked to make a yes-or-no style decision about whether the transcript is consistent. The dangerous objects are:

- decryption-failure margins,
- re-encryption checks,
- implicit rejection,
- and decapsulation-time arithmetic on malformed inputs.

For signatures, the attacker interacts with a stateful distribution generator. The signer repeatedly maps messages into challenges and responses under one long-term key. The dangerous objects are:

- ephemeral samplers,
- rejection counts,
- hint-generation logic,
- and any fault that causes nonce reuse, partial nonce reuse, or biased acceptance.

The difference matters operationally. A KEM attack often needs an online oracle with chosen ciphertexts. A signature attack can accumulate information from published signatures, local traces during signing, or faulted outputs that remain valid enough to pass verification. The KEM attacker is pushing ciphertexts into a boundary test. The signature attacker is trying to learn how the signer keeps manufacturing admissible short witnesses.

### What Constant-Time Means In Each Family

This is the point where generic secure-coding rules become too weak to be useful. "Be constant-time" is not wrong, but it is incomplete unless we say which lattice-specific boundary must be flattened.

For ML-KEM-like decapsulation, constant-time really means:

- no secret-dependent branches, addresses, or variable-latency arithmetic in decrypt, re-encrypt, compare, and select paths;
- no observable distinction between valid and implicitly rejected ciphertexts;
- no preserved secret intermediate data such as compare flags or secret transforms after termination;
- no fault countermeasure whose failure signal becomes a new decapsulation oracle.

For ML-DSA-like or Falcon-like signing, constant-time really means:

- no leakage of sampler state, rejection count, or hint-generation path;
- no secret-dependent exposure through decompositions, norm checks, or retry loops;
- no nonce reuse or partial reuse under deterministic or faulted executions;
- no assumption that the final signature check is the only place where security lives.

The practical split between current standardized families is also clearer now. ML-KEM and ML-DSA deliberately stay in integer arithmetic and explicitly ban floating point.[^fips203][^fips204] That avoids one class of implementation fragility. Falcon chooses a different point in the design space: smaller signatures through a more delicate sampler stack.[^falcon] Neither choice makes implementation security automatic. They simply move the most dangerous surface to different places.

## Conclusion

The lattice subseries started from worst-case reductions and ended in hot loops. That is the correct ending.

The proof-level claim remains strong: module-LWE, NTRU-style structure, Fiat-Shamir with aborts, GPV-style sampling, and CCA transforms all explain why the abstract constructions should resist the attacks modeled by their proofs. But deployed trust is decided one layer lower, where those abstractions become NTT kernels, reductions, samplers, rejection logic, hint generation, decryption margins, compare flags, and fault behavior.

The KEM side and the signature side therefore have to be kept separate:

- KEM security breaks when decapsulation becomes a richer oracle than the proof allowed.
- Signature security breaks when the signer leaks or reuses the private distribution machinery that the proof needed to stay hidden.

That is the right line to close on. Theory-level confidence says the abstract relation is hard. Engineering-level trust says the implementation does not accidentally export a second, easier relation through timing, power, memory, or faults. In lattice cryptography, the two claims are connected, but they are never the same claim.

## References

Suggested reading order:

1. Peikert first, for the clean separation between lattice hardness assumptions, constructions, and implementation-independent security claims.
2. FIPS 203 and FIPS 204 next, with attention to the algorithms that become hot loops: NTT, compression, decapsulation, rejection, and hint generation.
3. NTRU Prime's attack-surface paper after that, to read design criteria as implementation-security choices rather than pure efficiency choices.
4. KyberSlash and ML-KEM fault papers next, for decapsulation as an oracle boundary.
5. BLISS, Falcon, and Dilithium implementation-attack papers last, for sampler, rejection, branch, cache, and fault behavior on the signature side.

[^peikert]: Chris Peikert, *A Decade of Lattice Cryptography*, *Foundations and Trends in Theoretical Computer Science*, 2016. DOI: 10.1561/0400000074.
[^fips203]: National Institute of Standards and Technology, *Module-Lattice-Based Key-Encapsulation Mechanism Standard*, FIPS 203, August 13, 2024. DOI: 10.6028/NIST.FIPS.203.
[^fips204]: National Institute of Standards and Technology, *Module-Lattice-Based Digital Signature Standard*, FIPS 204, August 13, 2024. DOI: 10.6028/NIST.FIPS.204.
[^ntruprime]: Daniel J. Bernstein, Chitchanok Chuengsatiansup, Tanja Lange, and Christine van Vredendaal, *NTRU Prime: reducing attack surface at low cost*, 2017.
[^kyberslash]: Daniel J. Bernstein, Karthikeyan Bhargavan, Shivam Bhasin, Anupam Chattopadhyay, Tee Kiah Chia, Matthias J. Kannwischer, Franziskus Kiefer, Thales B. Paiva, Prasanna Ravi, and Goutam Tamvada, *KyberSlash: Exploiting secret-dependent division timings in Kyber implementations*, 2025.
[^kyber]: Joppe Bos, Leo Ducas, Eike Kiltz, Tancrede Lepoint, Vadim Lyubashevsky, John M. Schanck, Peter Schwabe, Gregor Seiler, and Damien Stehle, *CRYSTALS-Kyber: a CCA-secure module-lattice-based KEM*, 2018.
[^dilithium]: Shi Bai, Leo Ducas, Eike Kiltz, Tancrede Lepoint, Vadim Lyubashevsky, Peter Schwabe, Gregor Seiler, and Damien Stehle, *CRYSTALS-Dilithium: A Lattice-Based Digital Signature Scheme*, 2018.
[^mlkem-fault]: Peter Pessl and Lukas Prokop, *Fault Attacks on CCA-secure Lattice KEMs*, *IACR Transactions on Cryptographic Hardware and Embedded Systems*, 2021.
[^dfa-kyber]: Jan-Pieter D'Anvers and Senne Batsleer, *Multitarget Decryption Failure Attacks and Their Application to Saber and Kyber*, *PKC 2022*.
[^bliss-cache]: Leon Groot Bruinderink, Andreas Huelsing, Tanja Lange, and Yuval Yarom, *Flush, Gauss, and Reload: A Cache Attack on the BLISS Lattice-Based Signature Scheme*, *CHES 2016*.
[^bliss-branch]: Thomas Espitau, Pierre-Alain Fouque, Benoit Gerard, and Mehdi Tibouchi, *Side-Channel Attacks on BLISS Lattice-Based Signatures: Exploiting Branch Tracing against strongSwan and Electromagnetic Emanations in Microcontrollers*, IACR ePrint 2017/505.
[^falcon]: Pierre-Alain Fouque, Jeffrey Hoffstein, Paul Kirchner, Vadim Lyubashevsky, Thomas Pornin, Thomas Prest, Thomas Ricosset, Gregor Seiler, William Whyte, and Zhenfei Zhang, *Falcon: Fast-Fourier Lattice-based Compact Signatures over NTRU*, 2020.
[^dilithium-fault]: Leon Groot Bruinderink and Peter Pessl, *Differential Fault Attacks on Deterministic Lattice Signatures*, *IACR Transactions on Cryptographic Hardware and Embedded Systems*, 2018.

