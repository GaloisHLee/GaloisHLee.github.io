# Hash To Curve


> Reading: RFC 9380, *Hashing to Elliptic Curves*.

Core problem: given an arbitrary byte string, produce a point in the correct elliptic-curve subgroup, with the distribution and side-channel properties a protocol actually needs. This is not “hash to an $x$ and try to solve for $y$”.

The RFC answer is a pipeline, not a trick:

$$
\text{bytes} \rightarrow \text{field elements} \rightarrow \text{curve points} \rightarrow \text{subgroup points}
$$

plus domain separation and constant-time constraints.[^rfc9380]

This note only tracks that structure: what object the RFC is defining, why `encode_to_curve` and `hash_to_curve` differ, and why suites such as secp256k1 and BLS12-381 are built the way they are.

<!--more-->

## First Principles: Bytes To Subgroup

### What Problem Is RFC 9380 Solving?

RFC 9380 specifies algorithms for encoding or hashing an arbitrary string to a point on an elliptic curve.[^rfc9380] The sentence sounds small. Protocol-wise, it is a real primitive.

Suppose we want to turn a message `msg` into a group element:

$$
\text{msg} \in \{0,1\}^\* \longmapsto P \in G
$$

where $G$ is the prime-order subgroup used by the protocol. This shows up everywhere:

- BLS signatures hash messages into a curve group before scalar multiplication.
- VOPRF/OPRF-style protocols need a deterministic encoding of client input into a group element.
- PAKE and password-based designs often need a carefully domain-separated group encoding.

“Map bytes to a curve point” is not enough. A protocol usually wants:

- deterministic behavior
- subgroup correctness
- constant-time implementation
- random-oracle style security
- domain separation across protocol instances

That already kills the naive constructions.

### Why Try-And-Increment Had To Go

Historically, a common instinct was rejection sampling: hash the input, interpret it as a candidate $x$, test whether $x^3 + Ax + B$ is a square, and if not, try again with another counter. RFC 9380 explicitly does **not** standardize these probabilistic “try-and-increment” or “hunt-and-peck” methods, and says their use is **NOT RECOMMENDED** because they have repeatedly caused side-channel vulnerabilities.[^try]

> Key Observation: hash-to-curve is not hard because points are rare. It is hard because protocols need a point generation method whose control flow and output distribution are compatible with security proofs and implementations.

So the RFC target is deterministic, straight-line, plausibly constant-time code. That is why the standard prefers a layered interface over ad hoc rejection loops.

## The RFC Object Model

RFC 9380 decomposes the problem into three basic functions:[^iface]

1. `hash_to_field`
2. `map_to_curve`
3. `clear_cofactor`

This decomposition is the whole mental model.

### Formal Definition

The high-level ladder is:

$$
\text{bytes} \xrightarrow{\text{hash\_to\_field}} F
\xrightarrow{\text{map\_to\_curve}} E(F)
\xrightarrow{\text{clear\_cofactor}} G
$$

where:

- $F$ is the base field or extension field of the curve,
- $E(F)$ is the full set of curve points over $F$,
- $G$ is the target subgroup used by the protocol.

If this ladder is not explicit in the implementation, the bug usually reappears later as “wrong subgroup”, “wrong DST”, or “distribution not matching the proof”.

### Layer 1: `hash_to_field`

The first stage does **not** hash directly to a curve point. It hashes the message into one or more field elements.

RFC 9380 defines:

```text
hash_to_field(msg, count)
```

and the implementation route is:

1. expand the input with `expand_message`,
2. slice the resulting byte string into chunks,
3. interpret each chunk as an integer,
4. reduce modulo $p$ to obtain field elements.[^hash2field]

At a high level:

$$
\text{uniform\_bytes} \xrightarrow{\text{chunk + OS2IP + mod } p}
(u_0, \dots, u_{count-1}) \in F^{count}
$$

The `expand_message` stage matters because the RFC wants a clean source of pseudorandom-looking bytes, parameterized by a domain separation tag `DST`.

#### Why `L` Looks Like This

This is the first place where the RFC is doing real cryptographic engineering rather than mere serialization.

Let

$$
n = \lceil \log_2 p \rceil.
$$

If we sample only an $n$-bit integer $X$ and reduce modulo $p$, the distribution can be badly biased when $p$ is not close to a power of two. The RFC gives a concrete warning: if

$$
p \approx \frac{3}{4} \cdot 2^{256},
$$

then reducing a random $256$-bit value mod $p$ lands in

$$
[0, p/3]
$$

with probability roughly $1/2$.[^hash2field]

That is already far from uniform.

So the RFC adds a security slack of $k$ bits and samples an integer of length at least

$$
n + k
$$

bits. Then the reduction bias is at most $2^{-k}$.[^hash2field]

This is where

$$
L = \left\lceil \frac{\lceil \log_2 p \rceil + k}{8} \right\rceil
$$

comes from.

So `hash_to_field` is not just:

$$
H(\text{msg}) \bmod p.
$$

It is:

$$
\text{OS2IP}(\text{expand\_message}(\text{msg}, \text{DST}, L)) \bmod p,
$$

with $L$ chosen large enough that the reduction step does not introduce unacceptable bias.

> Key Observation: `hash_to_field` is where RFC 9380 pays the statistical cost required to make the later map-to-curve stage meaningful.

#### The Extension-Field Case

If the target field is $F = GF(p^m)$, then one field element is represented as

$$
u_i = (e_0, \dots, e_{m-1}),
$$

where each coordinate comes from its own `L`-byte chunk:

$$
e_j = \text{OS2IP}(\text{tv}_j) \bmod p.
$$

So for `count` outputs, the RFC requests

$$
\text{len\_in\_bytes} = \text{count} \cdot m \cdot L.
$$

This is why BLS12-381 G2 feels heavier than a prime-field suite: the top-level interface stays the same, but `m = 2` means the field layer already doubles the coordinate work.[^hash2field]

### Layer 2: `map_to_curve`

Once we have field elements, the next step is a deterministic map:

```text
map_to_curve(u)
```

This takes $u \in F$ and outputs a point $Q \in E(F)$. The goal here is not to sample uniformly from the subgroup in one step. The goal is to use a deterministic algebraic map with no rejection loop and a curve-family-specific formula.

This is why the RFC spends so much space on specific mapping families:

- Simplified SWU for Weierstrass curves
- Elligator 2 for Montgomery and twisted Edwards settings
- rational maps and isogenies when direct mapping to the target curve is inconvenient[^maps]

This section is formula-heavy, but the idea is short: `map_to_curve` is the curve-specific deterministic map that replaces “keep trying random candidates”.

#### A First Look At Simplified SWU

For a Weierstrass curve

$$
E: y^2 = g(x) = x^3 + Ax + B,
$$

the problem is to turn an arbitrary field element $u$ into some $x$ such that $g(x)$ is a square.

The naive method is “keep trying candidates until one works”. Simplified SWU instead manufactures a small set of algebraically related candidates.

For the $A \neq 0, B \neq 0$ case, RFC 9380 presents the mapping in terms of a carefully chosen non-square $Z$ and a few temporary values.[^maps] At the algebraic level, the core move is:

$$
\text{tv1} = \text{inv0}(Z^2 u^4 + Z u^2),
$$

$$
x_1 = -\frac{B}{A}(1 + \text{tv1}),
$$

and, in the exceptional case $\text{tv1} = 0$,

$$
x_1 = \frac{B}{ZA}.
$$

Then one computes

$$
g(x_1) = x_1^3 + A x_1 + B.
$$

If $g(x_1)$ is square, the job is done. If not, the construction derives another candidate $x_2$ from the same algebraic state and tests that branch instead.[^maps]

The point is not that $x_1$ itself is magical. The point is that the formulas force the candidate set to be structured enough that the algorithm can stay deterministic and branch only on square testing, not on an unbounded retry loop.

The RFC's optimized straight-line implementation hides this behind `tv1`, `tv2`, `tv3`, `sqrt_ratio`, and `CMOV`. But the design principle is simple:

1. construct a tiny algebraic family of candidate $x$ values from $u$;
2. guarantee that the exceptional case is handled without looping;
3. select the valid branch in constant-time style.

This is the real conceptual jump from “hunt for a valid point” to “use a deterministic map”.

#### What `sqrt_ratio` Is Actually Doing

At first sight, the optimized straight-line pseudocode hides the algebra almost too well:

```text
(is_gx1_square, y1) = sqrt_ratio(tv2, tv6)
```

The key point is that `sqrt_ratio(u, v)` does **not** merely ask “is $u/v$ square?” It returns a pair

$$
(b, y)
$$

such that:[^sqrt_ratio]

$$
b = \text{True}, \quad y = \sqrt{u/v}
$$

if $u/v$ is square in the field, and otherwise

$$
b = \text{False}, \quad y = \sqrt{Z \cdot (u/v)}.
$$

So `sqrt_ratio` is simultaneously:

1. a quadratic-residuosity test,
2. a square-root extractor for the successful branch,
3. a constant-time fallback mechanism for the unsuccessful branch.

This is why the RFC insists that `sqrt_ratio` and `map_to_curve_simple_swu` use the **same** non-square constant $Z$.[^sqrt_ratio]

In the optimized SSWU code, the pair `(tv2, tv6)` is constructed so that

$$
\frac{\text{tv2}}{\text{tv6}} = g(x_1)
$$

for one candidate branch, while the fallback value

$$
Z \cdot \frac{\text{tv2}}{\text{tv6}}
$$

lands on the other branch in the algebraic family. So `sqrt_ratio` is where the map compresses

- “test whether $g(x_1)$ is square” and
- “if not, move to the alternative candidate”

into one constant-time interface.

That is why the surrounding pseudocode can do:

```text
x = CMOV(x, tv3, is_gx1_square)
y = CMOV(y, y1, is_gx1_square)
```

instead of looping over candidate points. The branch exists, but the retry loop is gone.

#### The `AB == 0` Detour

Curves such as secp256k1 or BLS12-381 G1 do not fit the basic Simplified SWU precondition $A \neq 0$ and $B \neq 0$ directly. RFC 9380 handles this by moving to an **isogenous** curve

$$
E': y'^2 = x'^3 + A' x' + B'
$$

where the mapping is easier, then transporting the result back to the target curve by a rational isogeny map.[^ab0]

So the actual path for these suites is:

$$
u \xrightarrow{\text{SSWU on } E'} Q' \xrightarrow{\text{iso\_map}} Q \in E.
$$

This is exactly the kind of detail one misses if one thinks hash-to-curve means “find a valid point somehow”.

### Layer 3: `clear_cofactor`

Even after `map_to_curve`, the point may live on the full curve rather than inside the prime-order subgroup the protocol actually uses.

So the third stage is:

```text
clear_cofactor(Q)
```

which sends a curve point into the subgroup $G$.[^iface]

This step is easy to underestimate. On pairing-friendly curves or curves with nontrivial cofactors, forgetting subgroup handling is not a cosmetic bug. It is a protocol bug.

In some suites, cofactor clearing is scalar multiplication by an effective cofactor $h_{\text{eff}}$. In others, the RFC allows a faster equivalent method. The equivalence matters: a “fast” method may match $h_{\text{eff}}$ while not matching naive multiplication by the raw cofactor $h$.[^cofactor]

### RO Versus NU: `encode_to_curve` And `hash_to_curve`

RFC 9380 defines two related but different encodings:

- `encode_to_curve(msg)`
- `hash_to_curve(msg)`

The difference is output distribution.

`encode_to_curve` is **nonuniform**: its image covers only a fraction of subgroup points, and some points occur more frequently than others.[^encode]

`hash_to_curve` is designed to be statistically close to **uniform** over $G$.[^hash]

Operationally, the RFC gives:

```text
encode_to_curve(msg):
  u = hash_to_field(msg, 1)
  Q = map_to_curve(u[0])
  P = clear_cofactor(Q)
  return P
```

while

```text
hash_to_curve(msg):
  u = hash_to_field(msg, 2)
  Q0 = map_to_curve(u[0])
  Q1 = map_to_curve(u[1])
  R = Q0 + Q1
  P = clear_cofactor(R)
  return P
```

The extra sample and point addition are not decoration. They are part of how the uniform random-oracle style behavior is obtained.[^hash]

> Quick Note: if a security proof wants a random oracle returning group elements, the safe default is `hash_to_curve`, not `encode_to_curve`.

#### Why `Q0 + Q1` Helps

This is the right place to think in terms of distributions on the subgroup $G$.

Let the output distribution of

$$
\text{clear\_cofactor}(\text{map\_to\_curve}(u))
$$

be $\mu$. Then `encode_to_curve` samples from $\mu$, which is generally nonuniform.

`hash_to_curve` instead samples two independent field elements

$$
u_0, u_1 \in F
$$

via `hash_to_field(msg, 2)`, maps them separately, and returns

$$
P = \text{clear\_cofactor}(Q_0 + Q_1).
$$

At the distribution level, this replaces $\mu$ with the group convolution

$$
\mu * \mu,
$$

where

$$
(\mu * \mu)(R) = \sum_{Q \in G} \mu(Q)\mu(R-Q).
$$

The intuition is the same as elsewhere in additive cryptography: adding two independent samples smooths the bias far more effectively than using one sample alone.

But the safe statement is stronger and more precise than that heuristic. RFC 9380 does **not** merely say “adding two points looks more random”. It says:

- `encode_to_curve` is nonuniform,
- `hash_to_curve` is statistically close to uniform in $G$,
- and when instantiated with a `hash_to_field` that is indifferentiable from a random oracle, `hash_to_curve` is itself indifferentiable from a random oracle.[^uniform][^roproof]

So the real takeaway is:

> Key Observation: `Q0 + Q1` is not an implementation flourish. It is the step that turns a one-sample deterministic encoding into the RFC's random-oracle-style encoding discipline.

Also note what the RFC does **not** say: it does not say that point addition alone magically uniformizes any arbitrary map. The security statement is about the standardized composition

$$
\text{hash\_to\_field} \rightarrow \text{map\_to\_curve} \rightarrow (+) \rightarrow \text{clear\_cofactor}
$$

under the assumptions in Sections 5 and 10.1.[^roproof]

## The Domain Boundary

### DST

RFC 9380 requires all uses of the encoding functions to include domain separation.[^dst] The mechanism is a **domain separation tag**, `DST`.

This is not optional hygiene. It is part of the primitive.

If an application instantiates multiple independent encodings, even on the same curve, the tags must be different. The RFC recommends nonempty tags, and recommends a minimum length of 16 bytes to reduce collision risk with other applications.[^dst]

So one should think of the real primitive as:

$$
P = \text{hash\_to\_curve}(\text{msg}, \text{DST}, \text{suite})
$$

not just `hash_to_curve(msg)`.

If the DST discipline is weak, distinct protocol components may silently share the same internal hash-to-field behavior. That is exactly the failure domain separation is supposed to prevent.

### Constant-Time Or It Does Not Count

The RFC repeatedly pushes the design toward constant-time implementation.

At the utility-function level, it says implementations should be constant time, meaning running time and memory access patterns should not depend on secret inputs, intermediate values, or outputs.[^ct1]

At the security-considerations level, the document says constant-time implementations are strongly recommended for all uses, and required when the encoding input is secret, because otherwise timing leakage can become an attack surface.[^ct2]

This explains several design decisions:

- no rejection-sampling based `hash_to_field`
- explicit handling of exceptional cases
- constant-time utility operations such as `CMOV`, `inv0`, and field square-root selection

The standard is not only solving a math problem. It is solving an implementation problem under side-channel constraints.

### When Uniformity Actually Matters

RFC 9380 says that applications whose security requires a random oracle returning uniformly random subgroup points **must** use a suite whose encoding type is `hash_to_curve`.[^uniform]

The rule is short:

- if nonuniformity is acceptable and analyzed, `encode_to_curve` may be enough;
- if the proof wants a random oracle into the group, use `hash_to_curve`.

And when one is unsure, the RFC says to use a uniform encoding.[^encprop]

## Suite Layer

### Why SSWU, Elligator 2, And Isogenies Appear

The mapping stage is where curve family matters.

For short Weierstrass curves, the RFC standardizes Shallue-van de Woestijne variants, especially Simplified SWU. For Montgomery and twisted Edwards settings, it standardizes Elligator 2 based routes.[^maps]

At first sight this looks fragmented. The interface is still unified:

- `hash_to_field` gives field elements,
- the suite chooses a mapping family,
- `clear_cofactor` lands in the target subgroup.

So “hash to curve” is not one formula. It is a suite framework with curve-specific algebra in the middle.

### Case Study: secp256k1

For secp256k1, RFC 9380 defines `secp256k1_XMD:SHA-256_SSWU_RO_` and the corresponding nonuniform variant.[^k1]

The suite uses:

- `expand_message_xmd`
- `SHA-256`
- Simplified SWU for the `AB == 0` case
- parameter $Z = -11$
- a 3-isogeny map from an auxiliary curve $E'$ to the target curve $E$
- effective cofactor $h_{\text{eff}} = 1$[^k1]

This is a good example of the “suite, not trick” mindset. Even for a curve as familiar as secp256k1, the standard route is not “hash to x-coordinate”. It is:

1. hash to field,
2. map to an isogenous auxiliary curve using SSWU,
3. transport via isogeny,
4. apply the suite's subgroup rule.

The interesting part is that `h_eff = 1` here.[^k1] So subgroup clearing is trivial in this suite, but the mapping layer is not. The complexity moved upward into the isogenous-curve construction.

#### Why The 3-Isogeny Matters

The RFC does not merely say “there exists some isogeny”. Appendix E.1 gives its shape explicitly as a rational map:[^iso]

$$
x = \frac{x_{\mathrm{num}}(x')}{x_{\mathrm{den}}(x')}, \qquad
y = y' \cdot \frac{y_{\mathrm{num}}(x')}{y_{\mathrm{den}}(x')},
$$

where:

$$
x_{\mathrm{num}}(x') = k_{1,3}x'^3 + k_{1,2}x'^2 + k_{1,1}x' + k_{1,0},
$$

$$
x_{\mathrm{den}}(x') = x'^2 + k_{2,1}x' + k_{2,0},
$$

and similarly

$$
y_{\mathrm{num}}(x') = k_{3,3}x'^3 + k_{3,2}x'^2 + k_{3,1}x' + k_{3,0},
$$

$$
y_{\mathrm{den}}(x') = x'^3 + k_{4,2}x'^2 + k_{4,1}x' + k_{4,0}.
$$

So the 3-isogeny is not just a conceptual bridge. It is an actual degree-3 rational map from the easier auxiliary curve

$$
E': y'^2 = x'^3 + A'x' + B'
$$

to the target curve

$$
E: y^2 = x^3 + 7.
$$

That matters for implementation:

- the SSWU logic runs on $E'$ where the map is convenient,
- the final point on secp256k1 is obtained by one rational map evaluation,
- and because `h_eff = 1`, there is no extra subgroup-clearing cost after the isogeny.[^k1]

This is a nice inversion of the usual intuition. For secp256k1, the expensive-looking part is not cofactor handling. It is the algebra needed to land on the right curve without using rejection.

### Case Study: BLS12-381

For BLS12-381, the RFC defines suites for both G1 and G2.[^bls]

For G1, the suite is again built around Simplified SWU plus an isogeny map. For G2, the field is an extension field with $m = 2$, and the suite parameters live over $GF(p^2)$.[^bls]

This is where the decomposition pays off. The top-level interface does not change, even though:

- the field representation changes,
- the mapping constants change,
- the cofactor-clearing details change.

The RFC also notes that the chosen $h_{\text{eff}}$ values for the BLS12-381 suites are selected for compatibility with a fast cofactor-clearing method.[^bls]

That line is easy to skip, but it matters. On pairing-friendly curves, subgroup handling is not an afterthought.

#### G1: `h_eff` Is Not The Raw Cofactor

For BLS12-381 G1, the RFC sets

$$
h_{\mathrm{eff}} = \texttt{0xd201000000010001}.
$$

The same section notes that this value is chosen for compatibility with Scott's fast cofactor-clearing method.[^bls]

So the intended mental model is **not**

$$
\text{clear\_cofactor}(P) = h \cdot P
$$

using the raw cofactor from the curve equation. The intended model is:

$$
\text{clear\_cofactor}(P) = h_{\mathrm{eff}} \cdot P,
$$

where `h_eff` is the scalar equivalent of the fast subgroup projection method.

This distinction matters because RFC 9380 says it explicitly: when a curve admits a fast cofactor-clearing method, scalar multiplication by the raw cofactor $h$ does **not** generally give the same result and must not be used.[^fastcofactor]

For G1, the story is relatively clean because the equivalent scalar is still small enough to write compactly.

#### G2: The Endomorphism View

G2 is the more interesting case.

Here the RFC does not merely list a huge scalar and move on. Appendix G.3 gives an explicit fast cofactor-clearing routine using the efficiently computable endomorphisms

$$
\psi \quad \text{and} \quad \psi^2
$$

on the twist curve over $GF(p^2)$.[^g2clear]

The pseudocode starts with the BLS parameter

$$
x = -\texttt{0xd201000000010000},
$$

stored as `c1`, and computes:

```text
t1 = x * P
t2 = psi(P)
t3 = psi2(2P)
t3 = t3 - t2
t2 = x * (t1 + t2)
t3 = t3 + t2
t3 = t3 - t1
Q  = t3 - P
```

If we expand the group operations, this is

$$
Q = \psi^2(2P) + (x-1)\psi(P) + (x^2 - x - 1)P.
$$

This is the useful formula hiding inside the pseudocode.

So BLS12-381 G2 cofactor clearing is not “multiply by an enormous scalar because the cofactor is large”. It is “use the curve's endomorphism structure to realize the equivalent projection much faster”.

The RFC then says the result of this routine is equal to

$$
h_{\mathrm{eff}} \cdot P
$$

for the suite's published `h_eff` value.[^g2clear]

That is the right way to read the giant hexadecimal `h_eff` in Section 8.8.2:

- not as the algorithm you should literally implement first,
- but as the scalar that is equivalent to the fast subgroup-projection formula.

> Key Observation: on BLS12-381, especially G2, `h_eff` is best understood as the scalar shadow of an endomorphism-based projector.

## Protocol View

### Why Signatures, VOPRFs, And Password Protocols Care

If one only sees elliptic curves through ECDSA or Schnorr, hash-to-curve can feel exotic. But many protocols need group elements derived from arbitrary messages rather than arbitrary scalars.

Examples:

- BLS signatures hash the message into a curve group before exponentiation by the signing key.
- OPRF/VOPRF constructions often encode client input into a group element before the blind evaluation step.
- PAKE-like designs can need carefully domain-separated group encodings to avoid cross-protocol interference.

In all of these settings, “just find some curve point” is weaker than what the protocol actually wants.

### What Can Still Go Wrong

Even with RFC 9380 in hand, there are several easy failure modes:

1. **Wrong DST**  
   Reusing a DST across logically distinct encodings destroys domain separation.

2. **Wrong encoding type**  
   Using `encode_to_curve` where the proof wants `hash_to_curve`.

3. **Wrong subgroup handling**  
   Returning a curve point without proper cofactor clearing, or confusing the raw cofactor with the suite's effective method.

4. **Timing leaks**  
   Reintroducing try-and-increment logic or data-dependent exceptional-case handling.

5. **Rolling a custom suite too casually**  
   RFC 9380 gives a recipe for defining new suites, but that does not mean “pick a hash and improvise the rest”.[^newsuite]

So I prefer to think of hash-to-curve as a protocol primitive with a standard instantiation discipline, not as a local helper.

## Summary

### Mental Model

The mental model to keep is:

$$
\text{message bytes}
\rightarrow \text{field elements}
\rightarrow \text{curve points}
\rightarrow \text{subgroup points}
$$

with two constraints always carried along:

- domain separation
- constant-time implementation

Once this is clear, the RFC becomes easier to read. The formulas and suites are all instantiations of the same skeleton.

### Implementation Checklist

Compressed into a checklist:

1. Start from an existing RFC 9380 suite if possible.
2. Choose `hash_to_curve` unless nonuniform encoding is explicitly acceptable.
3. Treat `DST` as part of the interface, not an optional string parameter.
4. Do not use try-and-increment or rejection sampling.
5. Respect the suite's mapping and cofactor-clearing rules exactly.
6. Keep every layer compatible with constant-time implementation.

> Key Observation: hash-to-curve is a suite interface with security conditions, not a trick for finding a valid point.

## References

- RFC 9380, *Hashing to Elliptic Curves*: https://www.ietf.org/rfc/rfc9380.html
- RFC Editor entry for RFC 9380: https://www.rfc-editor.org/info/rfc9380
- For the core interface, read Sections 3, 5, 6, and 7 first.
- For security properties, read Section 10.1 and Section 10.3.
- For secp256k1 suite details, read Section 8.7 and Appendix E.1.
- For BLS12-381 suite details, read Section 8.8 and Appendix G.3.

[^rfc9380]: RFC 9380, Abstract and status page. See Sections 1 and 3 for the standard interface.
[^try]: RFC 9380 says probabilistic rejection-sampling methods such as “try-and-increment” are not specified and are not recommended due to side-channel risk; see Introduction and Appendix A discussion. Source: https://www.ietf.org/rfc/rfc9380.html
[^iface]: RFC 9380, Section 3, “Encoding Byte Strings to Elliptic Curves.” Source: https://www.ietf.org/rfc/rfc9380.html
[^hash2field]: RFC 9380, Section 5.2 and the `hash_to_field` interface. Source: https://www.ietf.org/rfc/rfc9380.html
[^maps]: RFC 9380, Section 6. Source: https://www.ietf.org/rfc/rfc9380.html
[^cofactor]: RFC 9380, Section 7 and suite-specific `h_eff` parameters. Source: https://www.ietf.org/rfc/rfc9380.html
[^ab0]: RFC 9380, Section 6.6.3, “Simplified SWU for AB == 0”. Source: https://www.ietf.org/rfc/rfc9380.html
[^encode]: RFC 9380, Section 3, definition of `encode_to_curve` as a nonuniform encoding. Source: https://www.ietf.org/rfc/rfc9380.html
[^hash]: RFC 9380, Section 3, definition of `hash_to_curve` and its two-sample composition. Source: https://www.ietf.org/rfc/rfc9380.html
[^dst]: RFC 9380, Section 3.1, domain separation requirements. Source: https://www.ietf.org/rfc/rfc9380.html
[^ct1]: RFC 9380, Section 4, utility functions and constant-time guidance. Source: https://www.ietf.org/rfc/rfc9380.html
[^ct2]: RFC 9380, Section 10.3, constant-time requirements. Source: https://www.ietf.org/rfc/rfc9380.html
[^uniform]: RFC 9380, Section 8 and suite guidance for applications requiring a uniform random oracle. Source: https://www.ietf.org/rfc/rfc9380.html
[^encprop]: RFC 9380, Section 10.1, properties of encodings. Source: https://www.ietf.org/rfc/rfc9380.html
[^sqrt_ratio]: RFC 9380, Section 5, `sqrt_ratio` interface and Section 6 optimized mappings using the same non-square constant `Z`. Source: https://www.ietf.org/rfc/rfc9380.html
[^roproof]: RFC 9380, Section 10.1, indifferentiability from a random oracle for `hash_to_curve` under the stated assumptions. Source: https://www.ietf.org/rfc/rfc9380.html
[^k1]: RFC 9380, Section 8.7, secp256k1 suites. Source: https://www.ietf.org/rfc/rfc9380.html
[^bls]: RFC 9380, Section 8.8, BLS12-381 G1 and G2 suites. Source: https://www.ietf.org/rfc/rfc9380.html
[^iso]: RFC 9380, Appendix E.1, 3-isogeny map for secp256k1 written as rational functions in $x'$ and $y'$. Source: https://www.ietf.org/rfc/rfc9380.html
[^fastcofactor]: RFC 9380, Section 7, warning that scalar multiplication by the raw cofactor is generally not equivalent to the fast cofactor-clearing method when such a method exists. Source: https://www.ietf.org/rfc/rfc9380.html
[^g2clear]: RFC 9380, Appendix G.3, fast cofactor clearing for BLS12-381 G2 using $\psi$ and $\psi^2$, and its equivalence to multiplication by the suite's `h_eff`. Source: https://www.ietf.org/rfc/rfc9380.html
[^newsuite]: RFC 9380, Section 8.9, defining a new hash-to-curve suite. Source: https://www.ietf.org/rfc/rfc9380.html

